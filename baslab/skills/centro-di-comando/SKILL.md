---
name: centro-di-comando
description: "This skill should be used when a BasLab studio asks to \"crea il centro di comando\", \"installa il centro di comando\", \"aggiorna il centro di comando\", \"rifai la dashboard dello studio\", \"porta il centro di comando all'ultima versione\", or when another Allerto skill needs the studio's command-center dashboard to exist. It creates or updates the Allerto-branded live dashboard over BasLab (Arcadia) and the Allerto switchboard from the files shipped with this plugin."
metadata:
  version: "1.0.0"
---

# Centro di comando (BasLab)

Crea o aggiorna il centro di comando dello studio: una dashboard live con il design Allerto, con attività, ticket, rate scadute, solleciti, assemblee, polizze e chiamate. I dati di BasLab sono letti in tempo reale a ogni apertura.

## Cosa contiene il plugin

Tutto in `${CLAUDE_PLUGIN_ROOT}/skills/centro-di-comando/assets/`:

- `index.html`: la pagina, già con stile Allerto, marcature dei dati, loader e calcoli.
- `fonts.css`: Hanken Grotesk, Source Serif 4 e JetBrains Mono incorporati, perché la dashboard non carica font esterni.
- `datasets.json`: `{versione, gestionale, dataset: {<id>: {title, description, source}}}`, una sorgente per sezione. Ogni `source` chiama uno strumento del collegamento Allerto-BasLab; il campo `connector` vale `{{connettore}}` e va sostituito con il nome del connettore dello studio.

| Sorgente | Strumento | Sezione |
|---|---|---|
| `condomini` | `elenca_condomini` | condomini gestiti |
| `attivita` | `attivita` | attività di Allerto |
| `ticket` | `interroga_gestionale` → `Ticket/paged-list` | ticket di BasLab |
| `rate_aperte` | `interroga_gestionale` → `EmissioneRata/scadenze/dx/list`, raggruppato per condominio, gestione, scadenza | rate scadute non incassate, in scadenza a 30 giorni |
| `solleciti` | `interroga_gestionale` → `Sollecito/dx/list` | ultimi solleciti |
| `assemblee` | `interroga_gestionale` → `Assemblea/dx/list` | assemblee e convocate |
| `polizze` | `interroga_gestionale` → `PolizzaAssicurazione/dx/list` | polizze |
| `chiamate` | `ultimeChiamate` | centralino |

Le date ("scaduto a oggi", "entro 30 giorni") le calcola la pagina nel fuso di Roma: le sorgenti non contengono date fisse, così restano giuste ogni giorno.

Non modificare questi file per adattarli a uno studio. Ciò che cambia da studio a studio sta nella configurazione (`${CLAUDE_PLUGIN_ROOT}/shared/configurazione.md`).

## Procedura

### 1. Controlla i collegamenti

- Verifica che ci siano gli strumenti del collegamento Allerto-BasLab (`elenca_condomini`, `interroga_gestionale`, `ultimeChiamate`, `attivita`). Se mancano, cercali con ToolSearch. Se il connettore non c'è o chiede l'accesso, dillo in una frase: "Serve il collegamento ad Allerto con il gestionale BasLab dello studio". Poi fermati.
- Se trovi invece lo strumento `sql` su un archivio Danea, questo studio usa Danea: dillo e fermati, serve l'altro plugin.
- Esegui una volta `elenca_condomini` per controllare che BasLab risponda.
- Annota il **nome del connettore** che offre questi strumenti: è il valore di `connettore` (se `allerto/config` lo ha già, usa quello).

### 2. Trova il centro di comando esistente

In ordine:
1. L'indirizzo detto dall'utente, se l'ha dato.
2. Il campo `centroDiComando` della configurazione, se conosci già l'indirizzo.
3. `Artifact` con `action: "list"` e `type: "Dashboard"`: cerca un titolo che inizia con "Centro di comando".

Se non ne trovi, crealo:
1. `Artifact` con `action: "list"` e `scope: "types"`; prendi il `type_url` del tipo **Dashboard**.
2. Pubblica con quel `type_url`, `title: "Centro di comando · allerto."` e `auto_open: "after_first_write"`.

La risposta porta le istruzioni del tipo Dashboard: leggile e rispettale. In particolare valgono le regole su "one writer", sul controllo `check`, sul formato del campo `connector` delle sorgenti e sul divieto di verificare con screenshot o browser. Se le istruzioni vogliono il connettore scritto in un altro modo, usa quel modo per sostituire `{{connettore}}`.

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

Se il centro di comando esistente era stato creato con il plugin Danea (sorgenti con `tool: "sql"`), dillo: l'aggiornamento sostituisce la pagina e le sorgenti con quelle BasLab.

### 4. Prepara i documenti

Usa Bash, con Python e il flag `-I`, per scrivere in una cartella di lavoro:
- `index.json` = `{"text": <contenuto di assets/index.html>}`;
- `fonts.json` = `{"text": <contenuto di assets/fonts.css>}`;
- `datasets.json` con `{{connettore}}` sostituito dal nome del connettore in ogni `source.connector`.

### 5. Scrivi tutto in un unico batch

`ArtifactData` con `action: "batch"`:
- `dash/meta`: `{title: "Centro di comando · allerto."}`;
- `datasets/<id>`: un `set` per ogni sorgente, con il documento intero (`title`, `description`, `source`);
- `files/fonts.css`: dal `file_path` di `fonts.json`;
- `files/index.html`: dal `file_path` di `index.json`.

Per i documenti che già esistono passa `if_version` con la versione letta. Se il batch fallisce per una versione cambiata, rileggi e riprova una volta.

Le sorgenti Danea vecchie (`kpi`, `interventi`, `segnalazioni`, `morosita_tot`, `morosita_condominio`, `morosi`, `assicurazioni`, `per_condominio`) non servono più alla pagina BasLab: proponi di cancellarle, non cancellarle da solo. Non toccare sorgenti aggiunte dallo studio.

### 6. Verifica

- Leggi `data/users/me`, documento `check`, come dicono le istruzioni del tipo Dashboard (fino a 5 letture).
- Riesegui una volta ogni sorgente `interroga_gestionale` con i suoi argomenti esatti. Se una risponde con un errore (per esempio "BasLab non accetta l'accesso"), segnala quale sezione resterà vuota e perché. Una sezione con zero righe (nessun ticket, nessuna polizza) non è un errore: la pagina lo scrive.

### 7. Aggiorna la configurazione

Aggiorna `allerto/config`:
- `centroDiComando`: l'indirizzo;
- `gestionale`: `BasLab`;
- `connettore`: il nome usato;
- `versionePlugin`: la `versione` di `datasets.json`;
- `paginaSha256`: lo sha256 di `assets/index.html`;
- `aggiornato`: l'ora attuale.

Se `allerto/config` non esiste, crealo con i soli campi noti e suggerisci la skill `configura-studio`.

## Cosa dire alla fine

In due o tre righe:
- se il centro di comando è stato creato o aggiornato, e da quale versione a quale;
- il numero più importante di oggi (per esempio il residuo delle rate scadute o le assemblee convocate);
- che ogni lettore vede i dati con il proprio accesso ad Allerto;
- che chi apre la pagina vede anche i nomi dei condòmini sollecitati, se la condivide.

Non incollare il link: la scheda della dashboard lo mostra già.
