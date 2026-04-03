# ai-helpdesk-ticket-triage-n8n
Low-code AI workflow for automatic IT ticket classification using n8n and OpenAI


# AI Help Desk Ticket Triage Workflow (n8n)

Workflow low-code realizzato in **n8n** per analizzare richieste di supporto IT e trasformarle in un output JSON strutturato tramite **OpenAI API**.

L’obiettivo del progetto non è costruire un help desk completo, ma dimostrare in modo semplice e credibile come un workflow low-code possa ricevere un problema tecnico in linguaggio naturale, inviarlo a un modello AI e restituire un risultato già pronto per essere usato in processi di triage, assistenza di primo livello o future integrazioni con sistemi di ticketing.

---

## Obiettivo del progetto

Nella pratica quotidiana, molte richieste di supporto arrivano come testo libero:

- “Non riesco ad accedere alla VPN”
- “Outlook non sincronizza”
- “Il PC è lentissimo e non apre Excel”
- “Ho perso l’accesso a una cartella condivisa”

Questo tipo di input è utile per una persona, ma non è immediatamente utilizzabile da un sistema automatico.  
Un workflow o un’applicazione, per prendere decisioni, ha bisogno di **dati strutturati**.

L’idea dietro questo progetto è quindi molto semplice:

1. ricevere un ticket IT in formato testuale
2. trasformarlo in un prompt chiaro per un modello AI
3. chiedere al modello di restituire un JSON con campi ben definiti
4. convertire la risposta in un output pulito e leggibile
5. restituire il risultato come se fosse una piccola API

Il valore del progetto sta proprio qui: non si limita a inoltrare testo, ma **trasforma un input non strutturato in dati strutturati**, cioè in qualcosa che un sistema può usare davvero.

---

## Perché n8n

La scelta di n8n nasce da una motivazione precisa: realizzare una demo funzionante senza dover costruire un backend completo da zero.

In un progetto tradizionale avremmo dovuto:

- creare un server
- definire endpoint API
- gestire parsing input/output
- integrare la chiamata verso OpenAI
- trattare la risposta
- restituire il JSON finale

Con n8n, invece, tutta questa logica viene orchestrata con nodi visuali.  
Questo non elimina la logica tecnica, ma la rende più veloce da implementare e più facile da mostrare.

La scelta low-code è quindi coerente con l’obiettivo del progetto:

- ridurre il tempo di sviluppo
- mantenere il flusso comprensibile
- concentrarsi sulla logica di integrazione
- dimostrare l’uso pratico di un LLM dentro un workflow tecnico

---

## Architettura del workflow

Il flusso finale è composto da 4 nodi principali:

```text
Webhook → Edit Fields → HTTP Request → Edit Fields

A livello logico, il workflow segue questo percorso:

Ticket IT in linguaggio naturale
→ preparazione del prompt
→ analisi tramite OpenAI
→ parsing della risposta
→ output JSON strutturato
Esempio di input
{
  "user": "Mario Rossi",
  "department": "Amministrazione",
  "issue": "Non riesco ad accedere alla VPN e Outlook non sincronizza le email"
}
Esempio di output
{
  "category": "Connettività e Email",
  "priority": "Alta",
  "possible_cause": "Problema di rete o configurazione VPN errata",
  "first_steps": [
    "Verificare la connessione Internet",
    "Controllare le credenziali VPN",
    "Riavviare il router e il computer",
    "Verificare le impostazioni di Outlook"
  ],
  "escalation": true
}
Logica generale del workflow

Prima di entrare nel dettaglio dei singoli nodi, vale la pena capire la logica complessiva.

Il progetto è stato costruito con una separazione abbastanza chiara tra le diverse responsabilità:

il Webhook riceve i dati
il primo Edit Fields prepara il contenuto da inviare al modello
il nodo HTTP Request delega l’analisi al modello AI
il secondo Edit Fields converte la risposta del modello in un JSON finale pulito

Questa separazione non è casuale.

Abbiamo evitato di concentrare tutto in un unico nodo perché un workflow leggibile è anche un workflow più facile da mantenere.
Quando ogni nodo ha un compito preciso, diventa più semplice:

capire dove avviene un errore
modificare un solo passaggio senza rompere gli altri
spiegare il progetto a un recruiter o a un collega
estendere in futuro il flusso con logiche aggiuntive
Analisi dettagliata dei nodi
1. Webhook
Ruolo del nodo

Il Webhook è il punto di ingresso del workflow.
In pratica è il nodo che permette al flusso di comportarsi come una piccola API.

Quando un client esterno invia una richiesta HTTP POST, n8n riceve il payload JSON e avvia il workflow.

Perché è stato scelto

La scelta del Webhook è naturale perché volevamo simulare uno scenario realistico di integrazione.

Un sistema esterno potrebbe essere:

un form interno aziendale
un portale ticketing
un tool di automazione
Postman, usato solo per test
un futuro backend più strutturato

Usare un Webhook rende il progetto più credibile rispetto a una semplice esecuzione manuale, perché introduce una logica di tipo API, cioè una modalità di utilizzo che è comune nei sistemi reali.

Cosa riceve

Il nodo riceve un JSON simile a questo:

{
  "user": "Mario Rossi",
  "department": "Amministrazione",
  "issue": "Non riesco ad accedere alla VPN e Outlook non sincronizza le email"
}
Cosa fa davvero

Il Webhook non analizza il ticket e non prende decisioni.
Il suo compito è solo:

ricevere il dato
renderlo disponibile ai nodi successivi
attivare il workflow
Scelta logica importante

Abbiamo deciso di non fare elaborazione qui dentro perché il Webhook deve restare il più “pulito” possibile.
Il suo ruolo è di ingresso, non di trasformazione.

2. Edit Fields (Prompt Builder)
Ruolo del nodo

Questo nodo prende i dati in ingresso dal Webhook e costruisce i campi che servono al modello AI.

In particolare, qui vengono preparati due elementi:

system_prompt
prompt
Perché serve davvero

Un modello AI non lavora bene se riceve semplicemente un testo grezzo senza contesto.
Serve una fase intermedia in cui il contenuto viene organizzato e presentato in modo chiaro.

Se avessimo inviato direttamente solo il valore del campo issue, il modello avrebbe avuto meno contesto e meno vincoli.
Invece, costruendo il prompt in questo nodo, abbiamo reso più esplicito:

il ruolo del modello
il formato atteso in output
i campi richiesti
i dati del ticket
system_prompt

Il system_prompt serve a definire il comportamento del modello.
Gli dice, in sostanza:

che è un assistente help desk IT
che deve rispondere solo in JSON
che non deve aggiungere testo superfluo
che l’output deve essere leggibile e strutturato

Questa scelta è importante perché riduce il rischio di risposte “discorsive” o poco controllabili.

prompt

Il prompt vero e proprio contiene il ticket e i dati contestuali.

Qui abbiamo scelto di includere:

utente
reparto
problema

Questo approccio ha una logica precisa: non ci interessa solo il sintomo tecnico, ma anche il contesto.
Un ticket proveniente da un reparto amministrativo, per esempio, può avere un impatto diverso rispetto a un ticket equivalente in un altro contesto.

Perché questo nodo è separato

Abbiamo scelto di costruire il prompt in un nodo dedicato perché così:

la preparazione dell’input AI resta leggibile
il nodo OpenAI riceve già un payload “ordinato”
eventuali modifiche al prompt possono essere fatte senza toccare il resto del workflow

In altre parole, questo nodo è una piccola fase di “traduzione” tra il mondo dell’input utente e il mondo della richiesta al modello.

3. HTTP Request (OpenAI API)
Ruolo del nodo

Questo è il nodo centrale del progetto, cioè quello che fornisce l’intelligenza del sistema.

Qui il workflow invia una richiesta HTTP verso l’API di OpenAI e chiede al modello di analizzare il ticket.

Perché usare OpenAI qui

Fino al nodo precedente il workflow aveva solo organizzato i dati.
La vera analisi semantica del problema avviene qui.

Senza questo passaggio, il sistema potrebbe solo:

inoltrare il testo
applicare regole rigide
usare condizioni statiche

Con OpenAI, invece, il workflow ottiene qualcosa di più flessibile:

classificazione del ticket
stima della priorità
ipotesi sulla causa
primi passi di troubleshooting
valutazione dell’escalation
Perché una chiamata API e non logica statica

Avremmo potuto creare regole tipo:

se trovi “VPN” → categoria rete
se trovi “Outlook” → categoria email
se trovi “non riesco” → priorità media

Ma una logica del genere sarebbe stata fragile e difficile da estendere.
Il vantaggio del modello è che riesce a interpretare il significato complessivo del ticket, non solo singole keyword.

Scelta dell’output strutturato

Nel body della richiesta abbiamo chiesto una risposta in formato JSON strutturato.

Questa scelta è fondamentale.

Se il modello rispondesse in modo libero, per esempio con un testo descrittivo, il risultato sarebbe meno utile per integrazioni successive.
Con un JSON ben definito, invece, l’output è già pronto per essere:

visualizzato in un’interfaccia
inoltrato a un sistema ticketing
salvato in un database
usato in regole successive
Perché questo nodo è il “cervello”

Questo nodo è quello che trasforma davvero il progetto da semplice automazione a integrazione AI.

In termini semplici:

prima di questo nodo abbiamo solo dati grezzi o preparati
dopo questo nodo abbiamo già una classificazione ragionata

Per questo motivo, l’HTTP Request verso OpenAI è il cuore del workflow.

4. Edit Fields finale (Parser e normalizzazione)
Ruolo del nodo

Il secondo nodo Edit Fields serve a prendere la risposta restituita da OpenAI e trasformarla nel formato finale desiderato.

Perché non bastava la risposta dell’API

Anche quando il modello restituisce un contenuto corretto, la risposta HTTP contiene diversi metadati aggiuntivi:

id della response
stato
timestamp
informazioni di billing
struttura interna dell’output

Questi dati sono utili per il provider API, ma non per il client finale del workflow.

Cosa fa quindi questo nodo

Questo nodo:

legge il campo della risposta che contiene il testo JSON generato dal modello
esegue il parsing del contenuto
estrae i singoli campi finali
costruisce il JSON definitivo
Perché estrarre i campi uno per uno

All’inizio la risposta del modello era ancora una stringa JSON.
Se l’avessimo restituita così, avremmo avuto un output meno pulito.

Abbiamo quindi scelto di eseguire il parsing e di esporre direttamente:

category
priority
possible_cause
first_steps
escalation

Questa scelta migliora molto la qualità finale del progetto, perché il risultato non è un testo che “sembra JSON”, ma un vero oggetto JSON.

Motivazione logica

Questo passaggio finale serve a chiudere bene il workflow.

Non volevamo solo dimostrare che il modello sa rispondere.
Volevamo dimostrare che il workflow sa:

ricevere
interpretare
trasformare
restituire un risultato pulito

Ed è proprio questo nodo a rendere il risultato finale più professionale.

Integrazione logica tra i nodi

Una parte importante del progetto non è solo cosa fa ogni nodo singolarmente, ma come i nodi collaborano tra loro.

Dal Webhook al Prompt Builder

Il Webhook riceve il ticket, ma da solo non sa come presentarlo al modello.
Il Prompt Builder prende quel dato e lo riorganizza in una forma utile per l’AI.

Quindi il passaggio logico è:

ricezione del dato → preparazione del contesto
Dal Prompt Builder all’HTTP Request

Il Prompt Builder costruisce un input chiaro, ma non esegue ancora alcuna analisi.
Il nodo HTTP Request prende quel prompt e lo affida al modello.

Il passaggio logico è:

contesto ben costruito → analisi intelligente
Dall’HTTP Request al Parser finale

L’API OpenAI restituisce una risposta valida, ma ancora immersa dentro la struttura della response.
Il nodo finale estrae solo quello che serve.

Il passaggio logico è:

risposta tecnica del provider → output finale utile al client
Visione d’insieme

Se guardiamo il flusso nel suo insieme, ogni nodo prepara il lavoro del successivo:

il Webhook riceve
il Prompt Builder organizza
OpenAI analizza
il Parser pulisce e normalizza

Questa progressione rende il workflow coerente e leggibile.

Scelte progettuali
Perché un progetto low-code

L’obiettivo non era costruire un’app enterprise completa, ma una demo concreta, rapida e presentabile.

Un approccio low-code era quindi la scelta più adatta per:

ridurre la complessità
mostrare un flusso end-to-end
evidenziare la logica di integrazione
mantenere il progetto spiegabile a colpo d’occhio
Perché output JSON

Il JSON è un formato semplice, interoperabile e adatto alle integrazioni.

Restituire un JSON significa che il workflow può essere esteso più facilmente verso:

database
dashboard
sistemi ticketing
altri workflow
chatbot o portali interni
Perché campi come priority ed escalation

Non volevamo una semplice sintesi del problema.
Volevamo un output che fosse già vicino a una logica operativa.

Campi come:

priorità
possibile causa
escalation

rendono il risultato più vicino a un vero processo di triage.

Perché non usare un backend custom

Sarebbe stato possibile usare Python o Node.js, ma avrebbe aumentato il tempo di sviluppo e la quantità di codice da gestire.

Per questo caso d’uso, n8n era sufficiente e coerente con l’obiettivo: costruire un flusso credibile senza introdurre complessità non necessaria.

Sicurezza e gestione credenziali

Il workflow utilizza una chiamata a OpenAI API, quindi richiede una chiave API.

È importante non pubblicare mai una chiave reale nel file workflow.json o nel repository GitHub.
Nel file condiviso pubblicamente va usato un placeholder oppure una configurazione esterna.

Esempio corretto:

Bearer YOUR_OPENAI_API_KEY

oppure una variabile ambiente / credenziale gestita esternamente.

Questo aspetto è importante non solo per sicurezza, ma anche perché fa parte delle buone pratiche di pubblicazione di un progetto tecnico.

Come usare il workflow
1. Importare il file

Importa workflow.json dentro n8n.

2. Configurare la chiave API

Inserisci la tua OpenAI API key nel nodo HTTP Request oppure sostituisci il placeholder con una configurazione sicura.

3. Attivare il workflow

Configura il nodo Webhook in modo che risponda con l’output finale dell’ultimo nodo.

4. Inviare una richiesta POST

Invia un JSON al webhook con i campi:

user
department
issue
5. Ricevere il risultato

Il workflow restituirà un JSON strutturato con la classificazione del ticket.

Possibili estensioni future

Questo progetto è volutamente semplice, ma è stato costruito in modo da poter essere esteso.

Evoluzioni possibili:

integrazione con ServiceNow, Zendesk o Jira
salvataggio automatico in Google Sheets o database
notifiche via email o Slack
categorizzazione più dettagliata
supporto multilingua
confidence score o livello di affidabilità
storico delle richieste analizzate

La struttura a nodi separati facilita proprio questo tipo di evoluzione.

Perché questo progetto è utile nel CV

Questo workflow mostra competenze trasversali che stanno bene in ruoli come:

supporto tecnico evoluto
integrazione AI
automation specialist
pre-sales tecnico
sistemista con approccio moderno ai workflow

In particolare dimostra:

capacità di ragionare per flussi
integrazione di API esterne
uso pratico di un LLM
trasformazione di testo libero in output strutturato
attenzione alla leggibilità del processo
Repository contents
workflow.json → esportazione del workflow n8n
README.md → documentazione del progetto
workflow.png → screenshot del flusso (opzionale)
Conclusione

Questo progetto mostra come, anche con un approccio low-code, sia possibile costruire un’integrazione AI concreta e utile.

Il valore non sta nella complessità tecnica del singolo nodo, ma nella logica complessiva del workflow: partire da un problema scritto in linguaggio naturale e arrivare a una risposta strutturata, coerente e riutilizzabile.

In altre parole, il workflow non si limita a “chiamare un modello”, ma organizza un piccolo processo decisionale che può diventare la base per automazioni più mature in ambito IT support e ticket triage.
