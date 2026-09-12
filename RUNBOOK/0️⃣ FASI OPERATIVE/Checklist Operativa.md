---

---

#Centroid

---
---
---
#### INDICE
- [[#Introduzione]]
- [ ] [[#Reconnaisance passivo]]
- [ ] [[#Reconnaisance attivo]]
- [ ] [[#Exploitation]]
- [ ] [[#Post Exploitation]]

---
---
---
### Introduzione
Il presente[[🔡 Glossario#Runbook| runbook°]] è progettato per guidare il penetration tester attraverso un ciclo completo di test. Le fasi principali durante un Penetration Test che ogni attaccante dovrebbe seguire sono le seguenti e nel seguente ordine:

- [ ] [[#Reconnaisance passivo]]
- [ ] [[#Reconnaisance attivo]]
- [ ] [[#Exploitation | Exploitation]]
- [ ] [[#Post Exploitation | Post Exploitation]]

Il book è pensato per essere *educativo* ed aiutare il penetration tester durante ogni fase, anche gli attaccanti alle prime armi. In ogni sezione sono presente informazioni dettagliate sulla metodologia da utilizzare per testare il target. In aggiunta sono presenti writeups utili per [[Linux]], [[Windows]] e[[📚 WRITEUPS/🌍 Ambienti/Active Directory | Active Directory]] consultabili in qualsiasi momento.

---
### Reconnaisance passivo
Il *Reconnaisance Passivo* è la prima fase di un attacco per un penetration tester e consiste nella raccolta informazioni sul target senza interagire direttamente con esso. 

L'obiettivo di questa fase è quello di raccogliere quante più informazioni possibili sul target utilizzando fonti pubbliche:

- [ ] [[1. Sottodomini| Sottodomini]]
- [ ] [[2. Data Leaks| Data Leaks]]
- [ ] [[3. Google Dorks| Google Dorks]]

---
### Reconnaisance attivo
La *ricognizione attiva* è una fase del penetration test in cui l’attaccante interagisce direttamente con il sistema target per raccogliere informazioni. A differenza della[[00 - Reconnaisance Passivo| ricognizione passiva]], che si limita a osservare fonti pubbliche senza farsi notare, quella attiva comporta l'invio di richieste al target, rendendo più probabile il rilevamento da parte di strumenti di logging e sistemi di difesa. 

La fase si suddivide nei seguenti steps:

- [ ] [[1. Scansione Porte| Scansionare le porte]]
- [ ] [[2. Servizi Online| Identificare servizi esposti online]]
- [ ] [[3. Sottodomini| Identificare sottodomini]]
- [ ] [[4. Fuzzing Directories| Fuzzing directories]]

---
### Exploitation
La fase di **exploitation** rappresenta il momento cruciale di un'attività di penetration testing: è qui che le vulnerabilità identificate vengono **attivamente sfruttate** per ottenere un accesso non autorizzato, elevare i privilegi o compromettere risorse critiche del sistema target.

L’obiettivo non è solo dimostrare che un difetto di sicurezza esiste, ma soprattutto **valutarne l’impatto reale**. A seconda del contesto, l’exploitation può consistere:

- [[21 - FTP| Exploit porte]]
- [[API | Web Exploitation]]
- [[Drupal | CMS]]

La fase di exploitation segna il passaggio dalla teoria alla pratica: dimostra se una vulnerabilità può trasformarsi in una **minaccia concreta**.

---
### Post Exploitation
Una volta ottenuto l’accesso iniziale al sistema target, entra in gioco la fase di **Post Exploitation**, in cui l’obiettivo non è più “entrare”, ma **capire, consolidare e sfruttare** l’accesso ottenuto.

Questa fase include attività strategiche mirate a effettuare privilege escalation su ambienti:

- [ ] [[Code Injection| Linux]]
- [ ] [[Abusing Token Privileges| Windows]]
- [ ] [[AS-REP Roasting| Active Directory]]

Durante questa fase, il focus si sposta dall’attacco all’**analisi dell’ambiente** compromesso. Ogni azione è guidata da un approccio tattico che valuta rischi, privilegi ottenuti, e possibili escalation.

La post exploitation è ciò che distingue un penetration test tecnico da una vera **simulazione di attacco** a fini di intelligence.
