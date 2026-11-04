---
name: configura-studio
description: "This skill should be used when a condominium-administration studio installs Allerto Studio for the first time, or asks to \"configura lo studio\", \"configura Allerto\", \"imposta il riepilogo del mattino\", \"cambia l'orario del riepilogo\", \"cambia la firma dello studio\", \"attiva le automazioni di Allerto\". It collects the studio's settings, creates the command center, saves the configuration and schedules the morning summary.\n"
metadata:
  version: "0.3.0"
---

# Configura lo studio

Prima installazione, o modifica delle impostazioni, di Allerto Studio per uno studio. Alla fine lo studio ha tre cose:
- il centro di comando;
- la configurazione salvata;
- il riepilogo del mattino programmato.

I campi della configurazione e dove vive sono descritti in `${CLAUDE_PLUGIN_ROOT}/shared/configurazione.md`. Leggilo prima di cominciare.

## Procedura

### 1. Collegamenti

- **Allerto**: verifica gli strumenti di `allerto-brain` (cercali con ToolSearch se differiti) ed esegui `sql` con `SELECT COUNT(*) AS N FROM TCONDOMINI_GENERALE`. Senza Allerto non si va avanti: dillo in una frase e fermati.
- **Posta**: cerca uno strumento per mandare email o creare bozze, per esempio Gmail o Outlook. Usa ToolSearch con "gmail send draft" e poi "outlook mail send".
  - Se non c'è, cerca il connettore con SearchMcpRegistry (`["gmail", "outlook", "email"]`) e proponilo con SuggestConnectors.
  - Se lo studio non vuole collegare la posta, continua: il riepilogo arriverà come messaggio in Claude invece che per email.

### 2. Impostazioni

Se esiste già un centro di comando, leggi prima `allerto/config` e proponi i valori attuali.

Per i dati dello studio guarda anche in Danea: nome dello studio, amministratore, email. Le tabelle di configurazione dello studio si trovano esplorando lo schema (`RDB$RELATIONS`). Proponi quello che trovi, senza darlo per certo.

Chiedi con AskUserQuestion, al massimo quattro domande per volta, solo ciò che manca:
- nome dello studio e nome dell'amministratore;
- email a cui mandare il riepilogo;
- firma delle email (nome, indirizzo, telefono);
- orario e giorni del riepilogo (default: 7:45, dal lunedì al venerdì);
- invio del riepilogo automatico o in bozza (default: automatico, perché è un'email interna);
- modalità dei verbali: A solo moduli Danea, B solo PDF su Drive, C entrambi;
- bozze delle email: attive o no, fascia oraria (default 8-19, dal lunedì al venerdì). L'avviso arriva sempre come notifica.

Non inventare mai nomi o indirizzi email.

### 3. Centro di comando

Segui la skill `centro-di-comando` (`${CLAUDE_PLUGIN_ROOT}/skills/centro-di-comando/SKILL.md`) per crearlo o aggiornarlo. Ti serve il suo indirizzo per il passo successivo.

### 4. Salva la configurazione

Scrivi `allerto/config` nel database del centro di comando con `ArtifactData`:
- `set` se non esiste;
- `update` con `if_version` se esiste.

Includi tutti i campi raccolti, più `centroDiComando`, `versionePlugin` e `aggiornato`.

### 5. Programma il riepilogo del mattino

- Cerca gli strumenti delle attività programmate con ToolSearch (`create_trigger`, `list_triggers`, `update_trigger`).
- Con `list_triggers` controlla se esiste già un'attività "Allerto · riepilogo del mattino". Se c'è, aggiornala invece di crearne un'altra.
- Crea l'attività programmata:
  - `name`: "Allerto · riepilogo del mattino";
  - `cron_expression`: nel fuso dello studio, per esempio `CRON_TZ=Europe/Rome 44 7 * * 1-5` per "7:45 dal lunedì al venerdì". Rispetta le indicazioni dello strumento sui minuti da evitare.
  - `prompt`: un'istruzione completa e autonoma, perché ogni esecuzione parte da zero:
    > Usa la skill riepilogo-mattino del plugin Allerto Studio. Centro di comando: <indirizzo>. Leggi la configurazione da lì (collection allerto, documento config) e manda il riepilogo di oggi all'amministratore come indicato.
  - `initiation`: `human_request`;
  - `requires_local_device`: non impostarlo, perché gira tutto nel cloud.
- Riferisci in una frase se le esecuzioni partono da sole o chiedono approvazione, come indicato nella risposta dello strumento.
- Se non esiste uno strumento per le attività programmate, dillo: il riepilogo si potrà chiedere a mano con "riepilogo del mattino".

### 5bis. Programma le bozze delle email

Se `bozzeEmail.attivo` è vero e la posta è collegata:
- con `list_triggers` controlla se esiste già "Allerto · bozze email"; se c'è, aggiornala;
- crea l'attività programmata:
  - `name`: "Allerto · bozze email";
  - `cron_expression`: ogni ora nella fascia dello studio, per esempio `CRON_TZ=Europe/Rome 0 8-19 * * 1-5`;
  - `prompt`: "Usa la skill bozze-email del plugin Allerto Studio. Centro di comando: <indirizzo>. Leggi la configurazione da lì (collection allerto, documento config), prepara le bozze per le email da rispondere e avvisa l'amministratore se ce ne sono di nuove.";
  - `notifications`: `{push: true}`;
  - `initiation`: `human_request`.
- Riferisci in una frase se le esecuzioni partono da sole o chiedono approvazione.

### 6. Istruzioni del Progetto

Le regole di lavoro arrivano con la skill `regole-studio` e valgono in ogni chat. Se lo studio lavora in un Progetto di claude.ai:
- crea il documento di progetto `claude/regole-archivio-documenti.md` se non esiste (strumento Projects, `project_write`), partendo dalle sezioni della skill `regole-studio`;
- compila `${CLAUDE_PLUGIN_ROOT}/shared/istruzioni-progetto.md` con i dati raccolti e mostralo in un blocco da copiare;
- spiega in una frase che va incollato nelle istruzioni del Progetto: è l'unico passo manuale, perché Claude non può modificare le istruzioni di un Progetto.

Se lo studio non usa un Progetto, salta questo passo.

### 7. Prova

Esegui subito la skill `riepilogo-mattino` una volta, in modalità bozza, così l'amministratore vede com'è fatta l'email. Riporta che la bozza è pronta e dove trovarla.

## Cosa dire alla fine

Una riga per ciascuna cosa:
- il centro di comando: creato o aggiornato;
- la configurazione salvata, con i campi principali;
- il riepilogo programmato, con orario, giorni e se parte da solo;
- il controllo orario delle email, con fascia oraria;
- le istruzioni del Progetto da incollare, se serve;
- la bozza di prova.

Poi un solo passo successivo: se `allerto/onboarding` non è `completata`, proponi la lezione guidata della skill `inizia-qui` ("Vuole che le mostri come lavorare con me, un argomento alla volta?"); altrimenti un passo naturale, per esempio "Quando vi fidate, possiamo attivare anche i solleciti ai fornitori".
