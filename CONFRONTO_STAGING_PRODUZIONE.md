# CONFRONTO STAGING vs PRODUZIONE - DEALERCLOUD

**Data:** 2025-11-24  
**Server Staging:** 192.168.19.216 (Windows Server 2016)  
**Scopo:** Verifica differenze tra ambiente deployment/staging e produzione

---

## 🎯 OBIETTIVO

Confrontare l'ambiente di deployment/staging con quello di produzione per:
- Identificare differenze di configurazione
- Verificare versioni codice
- Analizzare differenze database
- Validare che staging sia allineato a produzione

---

## 📋 SERVER STAGING (192.168.19.216)

### Informazioni Server
- **Hostname:** VPN-SERVER
- **IP:** 192.168.19.216
- **OS:** Windows Server 2016 Standard
- **IIS:** Microsoft-IIS/10.0
- **PHP:** 8.3.19
- **Database:** SQL Server (VPN-SERVER\SQLSERVER)

### Siti Web Identificati
1. **`/dealercloud/`** - DealerCloud (applicazione principale)
2. **`/rangereport/`** - RangeReport (redirect)
3. **`/app/`** - DealerCloudMobile

### Accesso
- **SSH:** sysadmin@192.168.19.216 (chiave SSH configurata)
- **HTTP:** http://192.168.19.216/dealercloud/
- **Database:** VPN-SERVER\SQLSERVER (DealerCloud_dev)

---

## 🔍 AREE DI CONFRONTO

### 1. CODICE APPLICATIVO

**File da confrontare:**
- Struttura directory
- Versioni file PHP
- Configurazioni (`cn.php`, `functions.php`, ecc.)
- File di configurazione IIS
- `.htaccess` o `web.config`

**Metodo:**
```bash
# Confronto struttura directory
diff -r staging/ produzione/

# Confronto file specifici
diff staging/login.php produzione/login.php
diff staging/cn.php produzione/cn.php
```

---

### 2. DATABASE

**Elementi da confrontare:**
- Schema database (tabelle, colonne, indici)
- Stored procedures
- Views
- Trigger
- Dati di configurazione
- Versioni dati

**Query SQL per confronto:**
```sql
-- Confronto schema tabelle
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
ORDER BY TABLE_NAME, ORDINAL_POSITION;

-- Confronto stored procedures
SELECT ROUTINE_NAME, ROUTINE_DEFINITION
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_TYPE = 'PROCEDURE';

-- Confronto indici
SELECT 
    t.name AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType
FROM sys.indexes i
INNER JOIN sys.tables t ON i.object_id = t.object_id
WHERE i.name IS NOT NULL
ORDER BY t.name, i.name;
```

---

### 3. CONFIGURAZIONE SERVER

**Elementi da confrontare:**
- Versione PHP
- Estensioni PHP abilitate
- Configurazione IIS (application pools, bindings)
- Permessi file/directory
- Variabili ambiente
- Certificati SSL

**Comandi PowerShell:**
```powershell
# Versione PHP
php -v

# Estensioni PHP
php -m

# Application Pools IIS
Get-IISAppPool | Select-Object Name, State, ManagedRuntimeVersion

# Siti IIS
Get-WebSite | Select-Object Name, State, PhysicalPath, Bindings
```

---

### 4. SICUREZZA

**Elementi da confrontare:**
- Credenziali database (hardcoded vs config file)
- Password hashing (algoritmi utilizzati)
- Session configuration
- HTTPS/SSL configuration
- Firewall rules
- File permissions

---

### 5. PERFORMANCE

**Elementi da confrontare:**
- Indici database
- Query performance
- Caching configuration
- Connection pooling
- Log files size

---

## 📊 CHECKLIST CONFRONTO

### Codice
- [ ] Struttura directory identica
- [ ] Versioni file PHP allineate
- [ ] Configurazioni identiche
- [ ] Nessun file mancante/extra

### Database
- [ ] Schema identico (tabelle, colonne)
- [ ] Stored procedures identiche
- [ ] Indici allineati
- [ ] Dati configurazione sincronizzati

### Server
- [ ] Versione PHP identica
- [ ] Estensioni PHP allineate
- [ ] IIS configuration identica
- [ ] Permessi file corretti

### Sicurezza
- [ ] Credenziali gestite correttamente
- [ ] Password hashing allineato
- [ ] Session config identica
- [ ] SSL/HTTPS configurato

### Performance
- [ ] Indici database ottimizzati
- [ ] Query performance accettabile
- [ ] Caching configurato

---

## 🚨 DIFFERENZE CRITICHE DA VERIFICARE

1. **Credenziali hardcoded:** Verificare che staging non abbia credenziali diverse da produzione
2. **Versioni codice:** Assicurarsi che staging sia aggiornato come produzione
3. **Schema database:** Verificare che non ci siano migrazioni mancanti
4. **Configurazioni:** Verificare che tutte le configurazioni siano allineate

---

## 📝 NOTE IMPORTANTI

- **Staging è ambiente di test:** Può contenere dati di test non presenti in produzione
- **Confronto bidirezionale:** Verificare sia staging→produzione che produzione→staging
- **Documentare differenze:** Ogni differenza trovata deve essere documentata con motivo

---

## 🔄 PROSSIMI PASSI

1. Identificare server produzione (IP/hostname)
2. Configurare accesso sicuro a produzione
3. Eseguire confronto sistematico
4. Documentare tutte le differenze
5. Allineare staging a produzione (se necessario)

---

**Documento creato:** 2025-11-24  
**Versione:** 1.0  
**Autore:** CTO DigitalBrain

