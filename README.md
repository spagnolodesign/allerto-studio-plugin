# allerto. Studio

Il centro di comando dello studio di amministrazione condominiale e le prime automazioni di Allerto, dentro Claude.

## Cosa fa

| Skill | Cosa fa | Come si chiede |
|---|---|---|
| `inizia-qui` | La prima chat dopo l'installazione: una lezione guidata per l'amministratore, un argomento alla volta, con prove sui dati dello studio. Salva i progressi e si riprende in un'altra chat | si attiva alla prima chat, oppure "inizia qui", "riprendiamo la lezione" |
| `configura-studio` | Prima installazione: raccoglie i dati dello studio, crea il centro di comando, salva la configurazione e programma il riepilogo del mattino | "configura lo studio" |
| `centro-di-comando` | Crea o aggiorna la dashboard live con il design Allerto: attività, interventi, segnalazioni, morosità, assemblee, polizze, chiamate | "aggiorna il centro di comando" |
| `regole-studio` | Le regole di lavoro dello studio: fonti, bozze, ticket, memoria condivisa, verbali, forma delle risposte | si attiva da sola |
| `bozze-email` | Ogni ora cerca le email da rispondere, prepara le bozze nei thread e avvisa l'amministratore | "controlla la posta" (o in automatico ogni ora) |
| `riepilogo-mattino` | Email breve all'amministratore con le cose da fare oggi e cosa è cambiato da ieri | "mandami il riepilogo" (o in automatico ogni mattina) |

## Cosa serve allo studio

1. **Il connettore Allerto (`allerto-brain`)**, collegato con l'accesso all'archivio Danea dello studio. Senza questo il plugin non fa nulla.
2. **La posta** (Gmail o Outlook) collegata a Claude, per mandare il riepilogo. Senza posta il riepilogo arriva come messaggio in Claude.
3. Le **attività programmate** abilitate sull'account, per il riepilogo automatico.

## Installazione in uno studio

1. Installa il plugin:
   - **claude.ai / Cowork (Team o Enterprise)**: l'Owner aggiunge questo repository come marketplace in *Impostazioni organizzazione > Plugins & skills* e rende disponibile `allerto-studio`.
   - **Claude Code**: `/plugin install allerto-studio --marketplace spagnolodesign/allerto-studio-plugin`.
2. Chiedi a Claude: "configura lo studio".
3. Rispondi alle domande: nome dello studio, email dell'amministratore, firma, orario del riepilogo.
4. Nella prima chat dello studio, Claude propone la lezione guidata (`inizia-qui`). Si può chiedere in ogni momento con "inizia qui".

## Aggiornare gli studi

1. Modifica i file del plugin:
   - le query e le sezioni sono in `skills/centro-di-comando/assets/datasets.json`;
   - la pagina è in `skills/centro-di-comando/assets/index.html`.
2. Alza `version` in `.claude-plugin/plugin.json` e `versione` in `datasets.json`.
3. Quando gli studi ricevono la nuova versione, basta chiedere "aggiorna il centro di comando".

La skill non sovrascrive mai una pagina modificata a mano senza chiedere. Non cancella sorgenti aggiunte dallo studio.

## Dove sta la configurazione

Nel database del centro di comando di ciascuno studio (collection `allerto`, documento `config`). Così le attività programmate, che partono da sessioni nuove, la ritrovano conoscendo solo l'indirizzo del centro di comando. Dettagli in `shared/configurazione.md`.

## Convenzioni Danea

In `shared/regole-danea.md`: stato 3 = chiusa, scala delle priorità, esercizi in corso, calcolo dello scoperto. Se uno studio usa Danea in modo diverso, va indicato nella sua configurazione.

## Prossime skill previste

- `sollecito-fornitori`: email ai fornitori con interventi in ritardo (in bozza finché lo studio non si fida).
- `solleciti-morosi`: lettere ai condòmini con rate scoperte (sempre in bozza).
- `smista-email`: legge la posta in arrivo e apre le attività in Allerto senza doppioni.
