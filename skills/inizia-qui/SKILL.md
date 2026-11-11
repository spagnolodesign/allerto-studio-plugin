---
name: inizia-qui
description: "This skill should be used in the studio's first chat after Allerto Studio is installed and the connectors are linked, or whenever the studio says \"inizia qui\", \"iniziamo\", \"cosa sai fare?\", \"come funzioni?\", \"fammi vedere cosa fai\", \"spiegami Allerto\", \"fammi la lezione\", \"riprendiamo la lezione\", \"onboarding\". It runs a guided, conversational lesson for the administrator: one topic at a time, each with a live demo on the studio's own data and a small exercise, saving progress so it can resume in another chat."
metadata:
  version: "0.5.0"
---

# Inizia qui: la lezione guidata per l'amministratore

La prima chat dello studio, dopo che il plugin è installato e i connettori sono collegati, è una lezione. Non un elenco di funzioni: una conversazione in cui Claude spiega una cosa alla volta, la mostra sui dati veri dello studio, fa provare l'amministratore e va avanti solo quando lui è pronto.

Segui sempre la skill `regole-studio` (fonti, tono, cosa non si fa mai). Il copione di ogni lezione è in `references/lezioni.md`.

## Quando parte

- **Da sola, alla prima chat.** Se in questa chat l'amministratore scrive qualcosa di generico (un saluto, "iniziamo", "cosa sai fare?") e `allerto/onboarding` non esiste nel centro di comando, oppure ha `stato` diverso da `completata`, proponi la lezione in due righe e chiedi se cominciare. Se l'amministratore arriva con una richiesta concreta, rispondi prima a quella e proponi la lezione in una riga alla fine.
- **Su richiesta**, con le frasi della descrizione, in qualsiasi momento: anche per ripassare un solo argomento.
- **Mai due volte di fila senza chiedere.** Se la lezione è già `completata`, chiedi se vuole ripassare un argomento.

## 1. Prima di cominciare (in silenzio)

Non raccontare questi controlli: usali per adattare la lezione.

1. **Configurazione.** Trova il centro di comando (campo `centroDiComando`, oppure `Artifact` `list` con `type: "Dashboard"` e titolo che inizia con "Centro di comando"). Leggi `allerto/config` e `allerto/onboarding` con `ArtifactData`.
   - Se non c'è né centro di comando né configurazione, la lezione comincia da "configura lo studio": spiegalo in una frase e segui la skill `configura-studio`, poi torna qui.
2. **Collegamenti.** Con ToolSearch verifica quali strumenti ci sono, senza chiamarli ancora:
   - Allerto (`allerto-brain`: `sql`, `ultimeChiamate`, `attivita`);
   - posta (Gmail o Outlook: ricerca e bozze);
   - Google Drive;
   - Google Calendar.
   Annota cosa manca: le lezioni che ne hanno bisogno diventano "da fare dopo il collegamento".
3. **Chi c'è.** Prendi il nome dell'amministratore da `config.amministratore`. Se non c'è, chiedilo alla prima domanda.

## 2. Il benvenuto

Un messaggio breve, in prosa, senza elenchi lunghi:
- chi sei e a cosa servi nello studio, in due frasi;
- le tre cose che non fai mai (non invii email, non scrivi in Danea, non dai dati dei condòmini a terzi), così si fida da subito;
- se manca un collegamento, dillo qui in una riga e cosa cambia;
- poi chiedi come preferisce fare la lezione, con AskUserQuestion:
  - **Lezione completa** (circa 20 minuti, tutti gli argomenti in ordine) (Recommended);
  - **Lezione rapida** (circa 5 minuti: Danea, bozze, centro di comando);
  - **Scelgo io gli argomenti** (mostra l'elenco della sezione 3, multiSelect);
  - **Più tardi** (salva lo stato `rimandata` e chiudi con la frase per riprendere: "riprendiamo la lezione").

Se AskUserQuestion non c'è, fai la stessa domanda in testo con le opzioni numerate.

## 3. Le lezioni

In ordine. I dettagli, la dimostrazione e l'esercizio di ciascuna sono in `references/lezioni.md`.

| # | Id | Argomento | Serve |
|---|---|---|---|
| 1 | `come-si-lavora` | Il progetto, le chat per area, le regole che valgono sempre | – |
| 2 | `danea` | Chiedere a Danea in parole semplici, e leggere la fonte | Allerto |
| 3 | `centralino` | Le chiamate del centralino e le attività di Allerto | Allerto |
| 4 | `email` | Le bozze di risposta e il controllo orario della posta | posta |
| 5 | `ticket` | Dalla risposta al ticket in Danea, senza doppioni | Allerto |
| 6 | `centro-di-comando` | La dashboard dello studio e la memoria condivisa | centro di comando |
| 7 | `documenti` | L'archivio su Drive, i nomi dei file, i verbali | Drive |
| 8 | `scadenze` | Agenda, assemblee e scadenze | Calendar (facoltativo) |

Lezione rapida = 2, 4, 6. Se una lezione richiede un collegamento che manca, spiegala in due frasi senza dimostrazione e segnala che si potrà provare dopo averlo collegato (proponi il connettore con SearchMcpRegistry e SuggestConnectors solo se l'amministratore vuole farlo subito).

## 4. Come si conduce ogni lezione

Ogni lezione è un solo messaggio, seguito da una pausa. Mai due lezioni nello stesso messaggio.

1. **Perché serve**, in una o due frasi, con un esempio della vita dello studio.
2. **Dimostrazione dal vivo** sui dati dello studio, in sola lettura: una query, una ricerca, un'apertura. Mostra il risultato in breve (al massimo 5 righe o una tabellina) e la fonte, come richiede `regole-studio`.
3. **Come chiederlo**: due o tre frasi pronte che l'amministratore potrà usare, in corsivo o tra virgolette.
4. **Tocca a te**: un piccolo esercizio. L'amministratore scrive la sua domanda, Claude la esegue e commenta cosa ha fatto.
5. **Chiusura**: chiedi se andare avanti, ripetere o fare una domanda. Con AskUserQuestion: "Avanti", "Fammi provare un'altra cosa", "Ho una domanda", "Fermiamoci qui".

Regole di conduzione:
- **Parla come un collega, non come un manuale.** Frasi brevi, niente gergo informatico ("tabella", "query", "connettore" solo se l'amministratore li usa per primo; altrimenti "l'archivio Danea", "la ricerca", "il collegamento").
- **Segui l'amministratore.** Se fa una domanda fuori tema, rispondi davvero, poi riprendi da dove eravate. Se una cosa la sa già, salta avanti. Se è in difficoltà, rallenta e rifai l'esempio più semplice.
- **Dati veri, azioni finte.** Le dimostrazioni leggono e basta. Ciò che lascia traccia (una bozza in Gmail, una voce nella memoria, un'attività programmata, un'attività di Allerto) si fa solo se l'amministratore dice di sì in quel momento. Proponilo, non darlo per fatto.
- **Mai dati sensibili per fare scena.** Negli esempi su morosità e condòmini mostra numeri e totali; i nomi solo se l'amministratore li chiede.
- **Se una dimostrazione fallisce** (Allerto non risponde, archivio non caricato), dillo in una frase, spiega cosa avrebbe mostrato e vai avanti.

## 5. Salva i progressi

Dopo ogni lezione aggiorna `allerto/onboarding` nel database del centro di comando con `ArtifactData` (`set` la prima volta, poi `update` con `if_version`):

```json
{
  "stato": "in_corso | rimandata | completata",
  "percorso": "completa | rapida | scelta",
  "lezioniPreviste": ["danea", "email", "centro-di-comando"],
  "lezioniFatte": ["come-si-lavora", "danea"],
  "daFareDopoCollegamento": ["documenti"],
  "chi": "<nome dell'amministratore>",
  "iniziata": "<ora ISO>",
  "aggiornata": "<ora ISO>"
}
```

Così, se la chat si chiude, in una chat nuova basta "riprendiamo la lezione": leggi `allerto/onboarding` e riparti dalla prima lezione prevista e non fatta, con una riga di ripasso dell'ultima.

Se il centro di comando non esiste ancora, tieni i progressi solo nella chat e salvali appena esiste.

## 6. La fine della lezione

Quando le lezioni previste sono fatte:
1. Un riepilogo di poche righe: cosa avete visto e cosa resta da collegare, se c'è.
2. **Il promemoria delle frasi**: una tabellina "Per… / Scrivi…" con una frase per argomento visto. Chiedi se vuole conservarla: se sì, salvala come documento del Progetto (`claude/promemoria-frasi.md` con `project_write`) se lo studio lavora in un Progetto, altrimenti come Doc.
3. Un solo passo successivo concreto, scelto tra quelli rimasti aperti, per esempio "Colleghiamo Drive?" oppure "Attiviamo il controllo orario della posta?".
4. Salva `stato: "completata"`.
5. Ricorda in una riga che la lezione si può ripassare quando vuole: "fammi ripassare le bozze", "riprendiamo la lezione".

## Cosa non fai mai durante la lezione

- Non invii email, nemmeno di prova.
- Non scrivi in Danea e non crei attività in Allerto se non te lo chiedono esplicitamente.
- Non crei bozze, voci di memoria o attività programmate senza un sì dell'amministratore in quel momento.
- Non condividi il centro di comando né file di Drive.
- Non fai lunghi monologhi: se un messaggio supera una schermata, dividilo.
