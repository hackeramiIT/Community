
---
---
---

#### INDICE
- [[#Introduzione]]
- [[#Enumerazione]]


---
---
---
### Introduzione
La vulnerabilità **XSS (Cross-Site Scripting)** si verifica quando un’applicazione web incorpora input controllato dall’utente in pagine HTML senza un’appropriata sanitizzazione/escaping, permettendo l’esecuzione di codice lato client (solitamente JavaScript) nel contesto del browser di altri utenti. Le conseguenze possono includere furto di sessioni, account takeover, defacement, pivoting verso risorse interne tramite il browser della vittima e raccolta di credenziali.

> [!INFO]
>  **Dove cercare:** parametri GET/POST, campi form, header (es. `Referer`, `User-Agent`), campi profilo/descrizione pubblica, commenti, campi di ricerca, risultati di ricerca

---
### Enumerazione
#### Javascript
Ispezionando il source code del servizio su [github](https://github.com), spesso troviamo cartelle che non dovrebbero essere messe in produzione e pertanto sono potenzialmente vulnerabili a XSS (eg. `dist`, `public`) in tal caso utilizzare [names.json](https://raw.githubusercontent.com/nice-registry/all-the-package-names/bba7ca95cf29a6ae66a6617006c8707aa2658028/names.json) come wordlist per gobuster.

---

