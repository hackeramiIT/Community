###### Runbook
Un runbook è un documento operativo che descrive passo dopo passo le attività da svolgere per completare un processo tecnico. Nel contesto del penetration testing, un runbook guida il tester attraverso le varie fasi del test, garantendo coerenza, copertura e ripetibilità.

---
###### Sottodomini CDN
Una CDN (Content Delivery Network) è una rete di server distribuiti che velocizza la consegna di contenuti web statici (immagini, video, script, ecc.).

---
###### UDP (User Datagram Protocol)
Il UDP, al contrario, è un protocollo non orientato alla connessione e senza garanzia di consegna. In pratica, i pacchetti vengono inviati al destinatario senza sapere se sono arrivati, senza negoziare la connessione e senza ritrasmissioni in caso di errore. È molto più leggero e veloce, ma non affidabile.

---
###### TCP (Transmission Control Protocol)
Il TCP è un protocollo di trasporto affidabile e orientato alla connessione. Prima che due dispositivi possano scambiarsi dati, devono stabilire una connessione stabile attraverso un processo chiamato handshake a tre vie.

---
###### Porta aperta
Una porta aperta indica che un'applicazione o un servizio sta attivamente ascoltando su quella porta ed è pronto a ricevere connessioni. Questo significa che, se invii una richiesta, otterrai una risposta dal servizio sottostante.

---
###### Porta chiusa
Una porta chiusa significa che non c’è nessun servizio in ascolto su quella porta. Il sistema risponde comunque al tuo tentativo di connessione.

---
###### Porta filtrata
Una porta filtrata è una porta per cui non ricevi alcuna risposta: né un'accettazione, né un rifiuto. Questo accade quando c’è un firewall o un dispositivo di filtraggio che blocca i tuoi pacchetti in ingresso o in uscita.

---
###### Reverse shell
Una reverse shell è una tecnica attraverso la quale un sistema compromesso (target) avvia una connessione in uscita verso l'attaccante, aprendo una shell remota controllata da quest’ultimo. Questa connessione permette all’attaccante di eseguire comandi da remoto. 
A tal proposito si consiglia l'utilizzo di questo [link](https://www.revshells.com/) per la creazione di reverse shell. 

---
###### Web Shell
Una web shell è uno script malevolo (solitamente scritto in linguaggi come PHP, ASP, JSP, Python o altri linguaggi lato server) che viene caricato e eseguito su un server web compromesso per fornire all’attaccante una shell remota via browser.

---
###### CVE
CVE è l’acronimo di Common Vulnerabilities and Exposures, un sistema di identificazione standardizzato per le vulnerabilità di sicurezza note nei software, firmware e sistemi informatici. Ogni CVE rappresenta una singola vulnerabilità, alla quale viene assegnato un codice univoco nel formato: CVE-AAAA-NNNN.

---
###### Burp Suite
Burp Suite è una suite integrata di strumenti per il test di sicurezza delle applicazioni web, ampiamente utilizzata da penetration tester, red teamer e bug hunter. Sviluppata da PortSwigger, è considerata uno degli strumenti più potenti e completi per l’analisi, l’intercettazione e la manipolazione del traffico HTTP/S.

---
###### Crawler
Un crawler, nel contesto della sicurezza informatica e del penetration testing, è uno strumento automatico che esplora in profondità una web application seguendo i link interni, le risorse collegate (JS, CSS, immagini) e spesso anche i parametri dinamici, con lo scopo di mappare tutte le pagine e funzionalità accessibili.

---
###### Stack Trace
Uno stack trace è un messaggio di errore generato da un’applicazione (solitamente web o software) che mostra l’elenco delle funzioni o metodi eseguiti dal programma fino al punto in cui si è verificato un errore. Viene chiamato “stack” perché riflette la pila delle chiamate (call stack) attive in quel momento. Generalmente questo tipo di errore è può portare ad attacchi di tipo[[SQL Injection|SQL Injection]].

---
###### Zone Transfer
Il zone transfer è un meccanismo del protocollo DNS utilizzato per replicare i dati di una zona DNS da un server primario (master) a uno o più server secondari (slave). Serve a mantenere la coerenza dei record DNS tra più nameserver che gestiscono lo stesso dominio.

---
###### HTTP Status Code

| **Codice** | **Categoria** | **Significato**                              | **Cosa indica per un pentester**                                                  |
| ---------- | ------------- | -------------------------------------------- | --------------------------------------------------------------------------------- |
| **1xx**    | Informativo   | Richiesta ricevuta, il client deve attendere | Raramente visibili, usati per protocolli avanzati (es. `101 Switching Protocols`) |
| **200**    | Successo      | OK                                           | Pagina/risorsa esistente, contenuto disponibile                                   |
| **201**    | Successo      | Created                                      | Una nuova risorsa è stata creata                                                  |
| **204**    | Successo      | No Content                                   | Nessun contenuto restituito, ma richiesta valida                                  |
| **301**    | Redirect      | Moved Permanently                            | URL spostato in modo permanente → da seguire                                      |
| **302**    | Redirect      | Found (Moved Temporarily)                    | Redirezione temporanea, utile per identificare login o auth                       |
| **307**    | Redirect      | Temporary Redirect                           | Come 302, ma mantiene metodo HTTP                                                 |
| **308**    | Redirect      | Permanent Redirect                           | Come 301, ma mantiene metodo HTTP                                                 |
| **400**    | Client Error  | Bad Request                                  | Sintassi errata o richiesta malformata                                            |
| **401**    | Client Error  | Unauthorized                                 | Richiesta authentication → bruteforce possibile                                   |
| **403**    | Client Error  | Forbidden                                    | Accesso negato → ma la risorsa esiste                                             |
| **404**    | Client Error  | Not Found                                    | Risorsa non trovata                                                               |
| **405**    | Client Error  | Method Not Allowed                           | Metodo HTTP non supportato → testabili per bypass                                 |
| **429**    | Client Error  | Too Many Requests                            | Rate-limit attivo → attenzione a bruteforce/fuzz                                  |
| **500**    | Server Error  | Internal Server Error                        | Possibile malfunzionamento → può indicare bug sfruttabile                         |
| **502**    | Server Error  | Bad Gateway                                  | Problemi con proxy o upstream                                                     |
| **503**    | Server Error  | Service Unavailable                          | Servizio momentaneamente offline                                                  |
| **504**    | Server Error  | Gateway Timeout                              | Timeout tra proxy e backend → può indicare DoS                                    |

---
###### Port Forwading / Tunneling
Tunneling, o port forwarding, è una tecnica che consente di redirezionare il traffico di rete attraverso una connessione intermediaria, aggirando restrizioni di rete, firewall o NAT.  
Si sfrutta una sessione autenticata (es: [[22 - SSH | SSH]]) per creare un canale cifrato attraverso il quale far passare dati diretti a porte e servizi interni non accessibili dall’esterno.

---
###### Passphrase
Una passphrase è una stringa segreta usata per proteggere chiavi crittografiche, come le chiavi private SSH (`id_rsa`), file PGP/GPG, certificati SSL o archivi crittografati.  
A differenza delle password, le passphrase possono (e dovrebbero) essere più lunghe e complesse, perché servono a proteggere chiavi già molto sensibili.

---
###### Variabili Globali
| **Variabile**     | **Descrizione / Uso in exploit**                                                                                                                                              |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PATH`            | Definisce le directory da cercare per i comandi. Se un binario SUID invoca comandi senza path assoluto, si può abusare del `PATH` per eseguire binari controllati.            |
| `LD_PRELOAD`      | Carica una libreria `.so` personalizzata prima di ogni altra, utile per hijackare funzioni (es: `geteuid()`, `execve()`). **Non funziona con binari SUID su sistemi sicuri.** |
| `LD_LIBRARY_PATH` | Modifica il path da cui vengono caricate le librerie dinamiche. Sfruttabile se il binario carica librerie locali non protette. Ignorato per SUID per sicurezza.               |
| `SHELL`           | Può essere abusata in alcuni script che richiamano `$SHELL`. Se controllabile, si può forzare l’esecuzione di una shell arbitraria.                                           |
| `IFS`             | Internal Field Separator. Modificandola (es. con un carattere nullo o `/`), si possono alterare i comandi interpretati in shell. Usata in exploit di parsing mal gestiti.     |
| `PS1`             | Prompt shell. Usata raramente in exploit, ma può essere sfruttata in contesti interattivi con parsing non sicuro.                                                             |
| `PYTHONPATH`      | Permette di iniettare moduli Python da directory controllate dall’attaccante. Può portare a RCE se usata in ambienti con script Python.                                       |
| `PERL5LIB`        | Equivalente di `PYTHONPATH` per Perl. Può iniettare moduli arbitrari se un binario SUID o uno script Perl ne fa uso.                                                          |
| `TZ`              | Timezone. Alcuni exploit (es. vecchie versioni di `cron` o `sudo`) lo hanno utilizzato per buffer overflow, anche se oggi è meno sfruttabile.                                 |
| `ENV`             | Può puntare a uno script `.sh` eseguito automaticamente da shell `sh` o `bash`. Utilizzata in exploit su shell invocate implicitamente.                                       |
| `BASH_ENV`        | Simile a `ENV`, ma specifica per bash non interattiva. Può essere utile per injectare comandi in ambienti sandbox o wrapper.                                                  |

---
###### NTLM
NTLM (NT LAN Manager) è un protocollo di autenticazione sviluppato da Microsoft per reti Windows. Usa un meccanismo challenge-response basato su hash (NT hash) delle password per verificare l’identità di un utente senza inviare la password in chiaro. È stato in gran parte sostituito da Kerberos in ambienti Active Directory, perché NTLM è più vulnerabile ad attacchi come **pass-the-hash**, **relay** e cracking degli hash.

---
###### LSASS
**LSASS** (Local Security Authority Subsystem Service) è un processo critico di Windows responsabile dell'autenticazione e della gestione della sicurezza locale. È il "guardiano" delle credenziali sul sistema.

---
###### Domain Controller
Un **Domain Controller** è un server all'interno di una rete Windows che gestisce l’autenticazione e l’autorizzazione degli utenti e dei computer. Contiene una copia del database Active Directory e verifica le credenziali durante il login, applica le policy di sicurezza e consente l’accesso alle risorse del dominio. È un componente centrale in qualsiasi infrastruttura AD, e la sua compromissione può significare il controllo totale del dominio.

---
###### Golden Ticket
Un **Golden Ticket** è un _ticket Kerberos falsificato_ che consente a un attaccante di autenticarsi come qualsiasi utente del dominio, inclusi gli amministratori, senza interagire con il Domain Controller (KDC).
Viene generato utilizzando l’hash NTLM dell’account `krbtgt` (l’account interno usato da Kerberos per firmare i TGT) e il SID del dominio.
Permette l’accesso completo a qualsiasi risorsa del dominio che utilizza Kerberos per l'autenticazione.

---
###### Silver Ticket
Un Silver Ticket è un ticket di servizio Kerberos (TGS) falsificato che consente a un attaccante di accedere solo a uno specifico servizio (es. MSSQL, CIFS, HTTP) su una macchina nel dominio.
Viene generato usando l’hash NTLM dell’account di servizio associato a quello SPN (Service Principal Name), insieme al SID del dominio.
Non richiede comunicazione con il Domain Controller → più difficile da rilevare.

-------
###### Service Principal Name
Il Service Principal Name (SPN) è un identificatore univoco usato in Active Directory e Kerberos per mappare un servizio di rete a un account di servizio; permette al Key Distribution Center (KDC) di emettere ticket Kerberos destinati al servizio corretto.