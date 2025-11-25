# 🔐 SETUP ACCESSO SSH - DealerCloud Deployment Server

**Data:** 25 Novembre 2025  
**Server:** Windows Server 2016 Standard (192.168.19.216)  
**Utente:** sysadmin  
**Scopo:** Accesso SSH per analisi gestionale DealerCloud

---

## 📋 INFORMAZIONI SERVER

- **Hostname:** VPN-SERVER
- **IP:** 192.168.19.216
- **OS:** Windows Server 2016 Standard
- **Utente SSH:** sysadmin
- **Gruppo:** Administrators

---

## 🔧 CONFIGURAZIONE COMPLETATA

### 1. CREAZIONE UTENTE

**Utente creato:** `sysadmin`

```powershell
$securePassword = Read-Host "Inserisci password complessa per utente sysadmin" -AsSecureString
New-LocalUser -Name "sysadmin" -Password $securePassword -Description "DealerCloud Audit - Multi-Server"
Add-LocalGroupMember -Group "Administrators" -Member "sysadmin"
```

**Gruppi:**
- Administrators ✅
- Remote Management Users (non disponibile su Windows Server 2016)

---

### 2. INSTALLAZIONE OPENSSH SERVER

**Metodo:** Installazione manuale (Windows Server 2016 non supporta OpenSSH come capability)

**Download:**
```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest -Uri "https://github.com/PowerShell/Win32-OpenSSH/releases/download/v9.2.2.0p1-Beta/OpenSSH-Win64.zip" -OutFile "$env:TEMP\OpenSSH-Win64.zip"
```

**Installazione:**
```powershell
Expand-Archive -Path "$env:TEMP\OpenSSH-Win64.zip" -DestinationPath "$env:TEMP\OpenSSH" -Force
Copy-Item -Path "$env:TEMP\OpenSSH\OpenSSH-Win64\*" -Destination "C:\Program Files\OpenSSH" -Recurse -Force
Set-Location "C:\Program Files\OpenSSH"
.\install-sshd.ps1
```

**Servizio:**
- Nome: `sshd`
- Account: `LocalSystem`
- Path: `C:\Program Files\OpenSSH`
- Porta: 22

---

### 3. CONFIGURAZIONE FIREWALL

**Regola creata:**
```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

**Verifica:**
```powershell
netstat -an | findstr ":22"
# Output atteso:
# TCP 0.0.0.0:22 0.0.0.0:0 LISTENING
# TCP [::]:22 [::]:0 LISTENING
```

---

### 4. CONFIGURAZIONE SSH

#### 4.1 Abilitazione Autenticazione Chiave Pubblica

**File config:** `C:\ProgramData\ssh\sshd_config`

**Modifiche applicate:**
```powershell
# Abilitato PubkeyAuthentication
(Get-Content "C:\ProgramData\ssh\sshd_config") -replace '#PubkeyAuthentication yes', 'PubkeyAuthentication yes' | Set-Content "C:\ProgramData\ssh\sshd_config"

# Abilitato PasswordAuthentication (per test)
(Get-Content "C:\ProgramData\ssh\sshd_config") -replace '#PasswordAuthentication yes', 'PasswordAuthentication yes' | Set-Content "C:\ProgramData\ssh\sshd_config"
```

#### 4.2 Configurazione AuthorizedKeysFile

**IMPORTANTE:** Su Windows OpenSSH, per utenti nel gruppo Administrators, il file authorized_keys è diverso!

**Configurazione trovata:**
```
Match Group administrators
AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

**File utilizzato:** `C:\ProgramData\ssh\administrators_authorized_keys` (NON `C:\Users\sysadmin\.ssh\authorized_keys`)

---

### 5. CREAZIONE CHIAVI SSH

#### 5.1 Generazione Chiave sul Mac

**Chiave generata:**
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_dealercloud_sysadmin -C "dealercloud-sysadmin-key" -N ""
```

**File creati:**
- Privata: `~/.ssh/id_dealercloud_sysadmin`
- Pubblica: `~/.ssh/id_dealercloud_sysadmin.pub`

**Fingerprint:**
```
SHA256:QY+0XI3Sy3cY8oH/AjiWCHwGr0PPmZJsEGJLXmKiwTU
```

**Chiave pubblica:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPx28ohSztkL5HcqlDAXeg4yQfGjTCbMOeHMjfuk0c60 dealercloud-sysadmin-key
```

#### 5.2 Configurazione sul Server Windows

**File:** `C:\ProgramData\ssh\administrators_authorized_keys`

**Comando:**
```powershell
"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPx28ohSztkL5HcqlDAXeg4yQfGjTCbMOeHMjfuk0c60 dealercloud-sysadmin-key" | Add-Content -Path "C:\ProgramData\ssh\administrators_authorized_keys" -Encoding ASCII
```

**Permessi:**
```powershell
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "SYSTEM:F" /grant "Administrators:R"
```

---

### 6. PERMESSI FILE/DIRECTORY

#### 6.1 Directory .ssh (creata ma non utilizzata per Administrators)

```powershell
New-Item -ItemType Directory -Path "C:\Users\sysadmin\.ssh" -Force
icacls "C:\Users\sysadmin\.ssh" /grant "SYSTEM:RX"
```

#### 6.2 File authorized_keys (backup nella home directory)

```powershell
icacls "C:\Users\sysadmin\.ssh\authorized_keys" /inheritance:r /grant "sysadmin:F" /grant "SYSTEM:R"
```

**NOTA:** Questo file NON viene utilizzato per utenti Administrators, ma è stato creato come backup.

---

## 🚨 PROBLEMI RISOLTI

### Problema 1: PowerShell Core su macOS non supporta WinRM
**Soluzione:** Installato OpenSSH Server invece di usare WinRM

### Problema 2: OpenSSH non disponibile come Windows Capability su Server 2016
**Soluzione:** Installazione manuale da GitHub releases

### Problema 3: Autenticazione chiave pubblica falliva
**Causa:** Per utenti Administrators, OpenSSH usa `C:\ProgramData\ssh\administrators_authorized_keys` invece della home directory
**Soluzione:** Aggiunta chiave nel file corretto

### Problema 4: Servizio sshd non in ascolto
**Causa:** Servizio fermato dopo modifiche configurazione
**Soluzione:** Riavvio servizio dopo ogni modifica a `sshd_config`

---

## ✅ VERIFICA CONNESSIONE

**Comando test:**
```bash
ssh -i ~/.ssh/id_dealercloud_sysadmin sysadmin@192.168.19.216 "hostname; whoami"
```

**Output atteso:**
```
VPN-SERVER
sysadmin
```

---

## 📝 FILE DI CONFIGURAZIONE

### Server Windows

**File principali:**
- `C:\ProgramData\ssh\sshd_config` - Configurazione OpenSSH Server
- `C:\ProgramData\ssh\administrators_authorized_keys` - Chiavi SSH per Administrators
- `C:\Users\sysadmin\.ssh\authorized_keys` - Backup (non utilizzato per Administrators)

### Mac Client

**File chiavi:**
- `~/.ssh/id_dealercloud_sysadmin` - Chiave privata SSH
- `~/.ssh/id_dealercloud_sysadmin.pub` - Chiave pubblica SSH

**Configurazione SSH (opzionale):**
Aggiungere a `~/.ssh/config`:
```
Host dealercloud-deployment
    HostName 192.168.19.216
    User sysadmin
    IdentityFile ~/.ssh/id_dealercloud_sysadmin
    StrictHostKeyChecking no
```

---

## 🔄 COMANDI UTILI

### Riavvio servizio SSH
```powershell
Restart-Service sshd
```

### Verifica stato servizio
```powershell
Get-Service sshd
```

### Verifica porta in ascolto
```powershell
netstat -an | findstr ":22"
```

### Verifica configurazione
```powershell
Get-Content "C:\ProgramData\ssh\sshd_config" | Select-String -Pattern "PubkeyAuthentication|PasswordAuthentication|AuthorizedKeysFile"
```

---

## 🎯 PROSSIMI SERVER DA CONFIGURARE

Stesso utente `sysadmin` e stessa chiave SSH possono essere utilizzati per:

1. **DealerCloud Produzione** (critico)
2. **DealerWeb Aruba** (critico)
3. **DealerWeb Gilbarco**

**Procedura per altri server:**
1. Creare utente `sysadmin` con stessa password
2. Aggiungere a gruppo Administrators
3. Installare OpenSSH Server (stesso metodo)
4. Aggiungere stessa chiave pubblica a `C:\ProgramData\ssh\administrators_authorized_keys`
5. Configurare firewall porta 22

---

## 📌 NOTE IMPORTANTI

1. **Sicurezza:** La chiave privata `~/.ssh/id_dealercloud_sysadmin` NON deve essere condivisa o committata su Git
2. **Password:** La password dell'utente sysadmin è salvata solo nella memoria dell'utente, mai esposta in chat/log
3. **Permessi:** Il file `administrators_authorized_keys` deve essere leggibile da SYSTEM
4. **Configurazione:** Dopo ogni modifica a `sshd_config`, riavviare il servizio sshd

---

**Ultimo aggiornamento:** 25 Novembre 2025  
**Status:** ✅ Configurazione completata e testata

