
---
---
---

#### INDICE
- [[#Introduzione]]
- [[#Enumerazione]]
	1. [[#HTTP Routing]]
- [[#Abilitare i Logs]]
	1. [[#PostgresSQL]]
- [[#Escapes]]
	1. [[#PostgreSQL]]
- [[#REGEX]]

---
---
---
### Introduzione
Una **SQL Injection** è una vulnerabilità che emerge quando il codice applicativo costruisce query SQL utilizzando input controllabile dall’utente senza un’adeguata validazione o parametrizzazione. Spesso si necessita di accesso al database come utente amministratore ed accesso ai logs (in alcuni casi è necessario abilitarli).

---
### Enumerazione
Per trovare SQL queries nel codice possiamo utilizzare le REGEX:

| **Pattern**          | **Spiegazione**              |
| -------------------- | ---------------------------- |
| `^.*?query.*?select` | Cerchiamo `query` e `select` |
#### HTTP Routing
Tra gli script cerchiamo comandi che dirigono il flusso di dati come, poiché spesso gli input degli utenti vengono utilizzati per query al DB:

**Java**

```
doGet
doPost
doPut
doDelete
doCopy
doOptions
```

📚​ Writeups
- *Windows:*[[Windows#Manageengine | Manageengine]], 
---
### Abilitare i Logs
E' sempre consigliato abilitare i logs del database in modo da catturare gli errori e vedere perché il tentativo di *SQLi* non va a buon fine:
#### PostgresSQL
In `C:\Program Files (x86)\ManageEngine\AppManager12\working\pgsql\data\amdb\postgresql.conf`: 

```
log_statement = 'all'			# none, ddl, mod, all
```

I logs sono in `C:\Program Files (x86)\ManageEngine\AppManager12\working\pgsql\data\amdb\pgsql_log\`

E poi riavviare il servizio da testare con: 

```
services.msc NAME_SERVICE_TEST
```

#### MariaDB
Abilitiamo i logs decommentando le righe in `/etc/mysql/my.cnf`:

```
[mysqld]
...
general_log_file        = /var/log/mysql/mysql.log
general_log             = 1
```

Riavviando il servizio:

```
sudo systemctl restart mysql
```

Adesso i logs saranno visibili su `/var/log/mysql/mysql.log`

---
### Escapes
Per *escapes* intediamo modi per rappresentare un simbolo o carattere non utilizzando il classico ASCII. Gli escapes sono spesso utilizzate per bypassare i controlli o la sanitizzazione dei caratteri.

#### PostgreSQL
##### Punto e virgola `;`
Possiamo riprodurre il `;` con i seguenti comandi:

```
select concat('1337',' h@x0r')
```

```
select concat(0x31333337,0x206840783072)
```
##### CHR e String Concatenation
Utilizzando `CHR()` possiamo inserire caratteri all'interno del database poiché ogni numero corrisponde ad un carattere ASCII

```
amdb=# CREATE TABLE AWAE (offsec text); INSERT INTO AWAE(offsec) VALUES (CHR(65)||CHR(87)||CHR(65)||CHR(69));
CREATE TABLE
INSERT 0 1

amdb=# SELECT * from AWAE;
 offsec
--------
 AWAE
```

##### Apostrofo `'`
Possiamo evitare di usare gli apici `'` sostituendoli con `$$TEXT$$` o `$TAG$TEXT$TAG`.
Di seguito un esempio (tutte le righe sono equivalenti):

```
SELECT 'AWAE';
SELECT $$AWAE$$;
SELECT $TAG$AWAE$TAG$;
```

---
### REGEX

| **Pattern**          | **Spiegazione**              |
| -------------------- | ---------------------------- |
| `^.*?query.*?select` | Cerchiamo `query` e `select` |
|                      |                              |
