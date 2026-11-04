# allerto. Studio · BasLab

Il centro di comando dello studio di amministrazione condominiale e le prime automazioni di Allerto, dentro Claude, per gli studi che usano il gestionale **BasLab (Arcadia)**.

È la versione BasLab del plugin `allerto-studio` (che è per Danea Domustudio). Le skill hanno gli stessi nomi e lo stesso comportamento; cambiano le fonti: niente SQL sull'archivio Danea, ma gli strumenti del collegamento Allerto-BasLab, letti in tempo reale. Non installare i due plugin nello stesso studio.

## Cosa fa

| Skill | Cosa fa | Come si chiede |
|---|---|---|
| `inizia-qui` | La prima chat dopo l'installazione: una lezione guidata per l'amministratore, un argomento alla volta, con prove sui dati di BasLab. Salva i progressi e si riprende in un'altra chat | si attiva alla prima chat, oppure "inizia qui", "riprendiamo la lezione" |
| `configura-studio` | Prima installazione: raccoglie i dati dello studio (anche da BasLab), crea il centro di comando, salva la configurazione e programma il riepilogo del mattino | "configura lo studio" |
| `centro-di-comando` | Crea o aggiorna la dashboard live: attività, ticket BasLab, rate scadute non incassate, solleciti, assemblee, polizze, chiamate | "aggiorna il centro di comando" |
| `regole-studio` | Le regole di lavoro dello studio: fonti BasLab, bozze, ticket, memoria condivisa, verbali, forma delle risposte | si attiva da sola |
| `apri-ticket` | Controlla i doppioni in BasLab e in Allerto, apre l'attività in Allerto e prepara il testo del ticket da inserire in BasLab | "apri un ticket", o sì alla domanda dopo una risposta |
| `bozze-email` | Ogni ora cerca le email da rispondere, prepara le bozze nei thread e avvisa l'amministratore | "controlla la posta" (o in automatico ogni ora) |
| `riepilogo-mattino` | Email breve all'amministratore con le cose da fare oggi e cosa è cambiato da ieri | "mandami il riepilogo" (o in automatico ogni mattina) |

## Cosa serve allo studio

1. **Il connettore Allerto collegato a BasLab** (quello che offre `elenca_condomini`, `morosi`, `interroga_gestionale`…). Senza questo il plugin non fa nulla.
2. **La posta** (Gmail o Outlook) collegata a Claude, per bozze e riepilogo. Senza posta il riepilogo arriva come messaggio in Claude.
3. Le **attività programmate** abilitate sull'account, per il riepilogo automatico.

## Installazione in uno studio

1. Installa il plugin:
   - **claude.ai (Team o Enterprise)**: l'Owner aggiunge il repository `spagnolodesign/allerto-studio-plugin` come marketplace in *Impostazioni organizzazione > Plugins & skills* e rende disponibile `allerto-studio-baslab`.
   - **Claude Code**: `/plugin install allerto-studio-baslab --marketplace spagnolodesign/allerto-studio-plugin`.
2. Chiedi a Claude: "configura lo studio".
3. Rispondi alle domande: nome dello studio, email dell'amministratore, firma, orario del riepilogo.
4. Nella prima chat dello studio, Claude propone la lezione guidata (`inizia-qui`).

## Il centro di comando

Le sorgenti sono in `skills/centro-di-comando/assets/datasets.json`. Il campo `connector` vale `{{connettore}}`: la skill lo sostituisce con il nome del connettore dello studio, salvato in configurazione.

| Sezione | Da dove |
|---|---|
| Condomini gestiti | `elenca_condomini` |
| Attività | `attivita` (Allerto) |
| Ticket | `Ticket/paged-list` |
| Rate scadute non incassate, in scadenza a 30 giorni | `EmissioneRata/scadenze/dx/list`, raggruppato per condominio, gestione e scadenza |
| Solleciti | `Sollecito/dx/list` |
| Assemblee | `Assemblea/dx/list` |
| Polizze | `PolizzaAssicurazione/dx/list` |
| Chiamate | `ultimeChiamate` (Allerto) |

Le date relative a oggi le calcola la pagina nel fuso di Roma; le sorgenti non hanno date fisse.

**Le rate della dashboard non sono la morosità per persona.** Sono il residuo lordo di ogni emissione già scaduta, su tutte le gestioni. La morosità per condòmino (strumento `morosi`) richiede una chiamata per condominio e si chiede in chat.

## Aggiornare gli studi

1. Modifica i file del plugin (`datasets.json`, `index.html`).
2. Alza `version` in `.claude-plugin/plugin.json` e `versione` in `datasets.json`.
3. Negli studi basta chiedere "aggiorna il centro di comando". Uno studio che aveva il centro di comando Danea viene portato alle sorgenti BasLab (la skill chiede prima di sovrascrivere una pagina modificata a mano).

## Dove sta la configurazione

Nel database del centro di comando di ciascuno studio (collection `allerto`, documento `config`). Dettagli in `shared/configurazione.md`.

## Convenzioni BasLab

In `shared/regole-baslab.md`: strumenti, esercizi aperti, morosi e scaduto netto, date in UTC, assemblee, solleciti, ticket, uso di `interroga_gestionale`.

## Limiti noti (versione 1.0.0)

- Il collegamento legge i ticket di BasLab ma non li crea: `apri-ticket` apre l'attività in Allerto (se c'è `creaAttivita`) e prepara il testo per BasLab.
- I campi di ticket e polizze sono stati scritti sui nomi più probabili, perché nello studio di prova le liste erano vuote: al primo studio con dati, controllare che le colonne si riempiano.
- L'area `Statistiche` di BasLab non è abilitata per l'utente del collegamento.

## Prossime skill previste

- `solleciti-morosi`: lettere ai condòmini con rate scoperte, partendo dai solleciti di BasLab (sempre in bozza).
- `sollecito-fornitori`: email ai fornitori con ticket in ritardo.
- `smista-email`: legge la posta in arrivo e apre le attività in Allerto senza doppioni.
