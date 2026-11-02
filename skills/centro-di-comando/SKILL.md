---
name: centro-di-comando
description: "This skill should be used when the studio asks to \"crea il centro di comando\", \"installa il centro di comando\", \"aggiorna il centro di comando\", \"rifai la dashboard dello studio\", \"porta il centro di comando all'ultima versione\", or when another Allerto skill needs the studio's command-center dashboard to exist. It creates or updates the Allerto-branded live dashboard over Danea and the Allerto switchboard from the files shipped with this plugin.\n"
metadata:
  version: "0.1.0"
---

# Centro di comando

Crea o aggiorna il centro di comando dello studio: una dashboard live con il design Allerto, con attività, interventi, segnalazioni, morosità, assemblee, polizze e chiamate.

## Cosa contiene il plugin

Tutto in `${CLAUDE_PLUGIN_ROOT}/skills/centro-di-comando/assets/`:

- `index.html`: la pagina, già con stile Allerto, marcature dei dati e loader.
- `fonts.css`: Hanken Grotesk, Source Serif 4 e JetBrains Mono incorporati, perché la dashboard non carica font esterni.
- `datasets.json`: `{versione, dataset: {<id>: {title, description, source}}}`, cioè una sorgente per sezione. Il campo `source` è già nella forma che vuole la dashboard: connettore `allerto-brain`, strumento e argomenti.

Non modificare questi file per adattarli a uno studio. Ciò che cambia da studio a studio sta nella configurazione (`${CLAUDE_PLUGIN_ROOT}/shared/configurazione.md`).

## Procedura

### 1. Controlla i collegamenti

- Verifica che gli strumenti di `allerto-brain` siano disponibili (`sql`, `ultimeChiamate`, `attivita`). Se mancano, cercali con ToolSearch. Se il connettore non c'è o chiede l'accesso, dillo in una frase: "Serve il collegamento ad Allerto (allerto-brain) con l'archivio Danea dello studio". Poi fermati.
- Esegui una volta `sql` con `SELECT COUNT(*) AS N FROM TCONDOMINI_GENERALE`, per controllare che l'archivio risponda.

### 2. Trova il centro di comando esistente

In ordine:
1. L'indirizzo detto dall'utente, se l'ha dato.
2. Il campo `centroDiComando` della configurazione, se conosci già l'indirizzo.
3. `Artifact` con `action: "list"` e `type: "Dashboard"`: cerca un titolo che inizia con "Centro di comando".

Se non ne trovi, crealo:
1. `Artifact` con `action: "list"` e `scope: "types"`; prendi il `type_url` del tipo **Dashboard**.
2. Pubblica con quel `type_url`, `title: "Centro di comando · allerto."` e `auto_open: "after_first_write"`.

La risposta porta le istruzioni del tipo Dashboard: leggile e rispettale. In particolare valgono le regole su "one writer", sul controllo `check` e sul divieto di verificare con screenshot o browser.

Se il centro di comando esiste, leggi con `ArtifactData`:
- `allerto/config`;
- `files/index.html`;
- tutti i documenti di `datasets`.

Annota le `version` di ciascuno.

### 3. Controlla modifiche fatte a mano

Confronta `files/index.html` dello store con il campo `paginaSha256` di `allerto/config`, calcolando lo sha256 del testo con Bash. Se differiscono, qualcuno ha modificato la pagina dopo l'ultimo aggiornamento. In quel caso:
- dillo all'utente;
- chiedi se sovrascrivere o tenere la sua versione;
- non sovrascrivere senza un sì.

### 4. Prepara i documenti

Usa Bash, con Python e il flag `-I`, per scrivere in una cartella di lavoro:
- `index.json` = `{"text": <contenuto di assets/index.html>}`;
- `fonts.json` = `{"text": <contenuto di assets/fonts.css>}`.

Leggi `assets/datasets.json` per le sorgenti.

### 5. Scrivi tutto in un unico batch

`ArtifactData` con `action: "batch"`:
- `dash/meta`: `{title: "Centro di comando · allerto."}`;
- `datasets/<id>`: un `set` per ogni sorgente di `datasets.json`, con il documento intero (`title`, `description`, `source`);
- `files/fonts.css`: dal `file_path` di `fonts.json`;
- `files/index.html`: dal `file_path` di `index.json`.

Per i documenti che già esistono passa `if_version` con la versione letta. Se il batch fallisce per una versione cambiata, rileggi e riprova una volta.

Non cancellare sorgenti che non sono in `datasets.json`: potrebbero essere aggiunte dello studio.

### 6. Verifica

- Leggi `data/users/me`, documento `check`, come dicono le istruzioni del tipo Dashboard (fino a 5 letture).
- Riesegui una volta ogni sorgente `sql` con i suoi argomenti esatti. Se una query fallisce, segnala quale sezione resterà vuota e perché.

### 7. Aggiorna la configurazione

Aggiorna `allerto/config`:
- `centroDiComando`: l'indirizzo;
- `versionePlugin`: la `versione` di `datasets.json`;
- `paginaSha256`: lo sha256 di `assets/index.html`;
- `aggiornato`: l'ora attuale.

Se `allerto/config` non esiste, crealo con i soli campi noti e suggerisci la skill `configura-studio`.

## Cosa dire alla fine

In due o tre righe:
- se il centro di comando è stato creato o aggiornato, e da quale versione a quale;
- il numero più importante di oggi (per esempio gli interventi in ritardo, presi dalla sorgente `kpi`);
- che ogni lettore vede i dati con il proprio accesso ad Allerto;
- che chi apre la pagina vede anche i nomi dei condòmini morosi, se la condivide.

Non incollare il link: la scheda della dashboard lo mostra già.
