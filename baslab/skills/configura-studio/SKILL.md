---
name: configura-studio
description: "This skill should be used when a condominium-administration studio that uses BasLab (Arcadia) installs Allerto Studio · BasLab for the first time, or asks to \"configura lo studio\", \"configura Allerto\", \"cambia la firma dello studio\", \"attiva le automazioni di Allerto\". It collects the studio's settings, creates the command center, saves the configuration and schedules the hourly email-draft check.\n"
metadata:
  version: "1.1.0"
---

# Configura lo studio (BasLab)

Prima installazione, o modifica delle impostazioni, di Allerto Studio per uno studio che usa il gestionale BasLab (Arcadia). Alla fine lo studio ha tre cose:
- il centro di comando;
- la configurazione salvata;
- il controllo orario della posta, se lo studio lo vuole.

Il plugin non manda email: niente riepilogo del mattino, solo bozze.

I campi della configurazione e dove vive sono descritti in `${CLAUDE_PLUGIN_ROOT}/shared/configurazione.md`. Leggilo prima di cominciare.

## Procedura

### 1. Collegamenti

- **Allerto e BasLab**: cerca con ToolSearch gli strumenti del collegamento Allerto-BasLab (`elenca_condomini`, `interroga_gestionale`, `ultimeChiamate`, `attivita`) ed esegui `elenca_condomini`. Annota il nome del connettore che li offre: diventa il campo `connettore`. Senza il collegamento non si va avanti: dillo in una frase e fermati.
  - Se trovi lo strumento `sql` su un archivio Danea invece di BasLab, questo studio usa Danea: dillo e proponi il plugin `allerto-studio`.
  - Annota anche quali strumenti di scrittura ci sono (`creaAttivita`, `registra_versamento`): cambiano cosa può fare la skill `apri-ticket`.
- **Posta**: cerca uno strumento per mandare email o creare bozze, per esempio Gmail o Outlook. Usa ToolSearch con "gmail send draft" e poi "outlook mail send".
  - Se non c'è, cerca il connettore con SearchMcpRegistry (`["gmail", "outlook", "email"]`) e proponilo con SuggestConnectors.
  - Se lo studio non vuole collegare la posta, continua: le bozze delle email restano spente.

### 2. Impostazioni

Se esiste già un centro di comando, leggi prima `allerto/config` e proponi i valori attuali.

Per i dati dello studio guarda anche in BasLab: con `descrivi_gestionale` e `interroga_gestionale` leggi le aree `Amministratore` (`Amministratore/dx/list`) e `Azienda` (nome dello studio, indirizzo, telefono, email). Il nome dello studio compare anche nel campo `azienda` delle rate e dei solleciti. Proponi quello che trovi, senza darlo per certo.

Chiedi con AskUserQuestion, al massimo quattro domande per volta, solo ciò che manca:
- nome dello studio e nome dell'amministratore;
- email dell'amministratore (serve alle bozze per riconoscere i messaggi dello studio);
- firma delle email (nome, indirizzo, telefono);
- modalità dei verbali: A solo verbale in BasLab, B solo PDF su Drive, C entrambi;
- bozze delle email: attive o no, fascia oraria (default 8-19, dal lunedì al venerdì). L'avviso arriva sempre come notifica.

Non inventare mai nomi o indirizzi email.

### 3. Centro di comando

Segui la skill `centro-di-comando` (`${CLAUDE_PLUGIN_ROOT}/skills/centro-di-comando/SKILL.md`) per crearlo o aggiornarlo. Ti serve il suo indirizzo per il passo successivo.

### 4. Salva la configurazione

Scrivi `allerto/config` nel database del centro di comando con `ArtifactData`:
- `set` se non esiste;
- `update` con `if_version` se esiste.

Includi tutti i campi raccolti, più `gestionale: "BasLab"`, `connettore`, `centroDiComando`, `versionePlugin` e `aggiornato`.

### 5. Togli il vecchio riepilogo del mattino

Le versioni precedenti del plugin programmavano un'attività "Allerto · riepilogo del mattino". Con `list_triggers` controlla se esiste ancora: se c'è, dillo e proponi di eliminarla. Non eliminarla senza un sì. Se la configurazione contiene ancora il blocco `riepilogo`, toglilo quando la salvi.

### 6. Programma le bozze delle email

Se `bozzeEmail.attivo` è vero e la posta è collegata:
- con `list_triggers` controlla se esiste già "Allerto · bozze email"; se c'è, aggiornala;
- crea l'attività programmata:
  - `name`: "Allerto · bozze email";
  - `cron_expression`: ogni ora nella fascia dello studio, per esempio `CRON_TZ=Europe/Rome 0 8-19 * * 1-5`;
  - `prompt`: "Usa la skill bozze-email del plugin Allerto Studio · BasLab. Centro di comando: <indirizzo>. Leggi la configurazione da lì (collection allerto, documento config), prepara le bozze per le email da rispondere e avvisa l'amministratore se ce ne sono di nuove.";
  - `notifications`: `{push: true}`;
  - `initiation`: `human_request`.
- Riferisci in una frase se le esecuzioni partono da sole o chiedono approvazione.

### 7. Istruzioni del Progetto

Le regole di lavoro arrivano con la skill `regole-studio` e valgono in ogni chat. Se lo studio lavora in un Progetto di claude.ai:
- crea il documento di progetto `claude/regole-archivio-documenti.md` se non esiste (strumento Projects, `project_write`), partendo dalle sezioni della skill `regole-studio`;
- compila `${CLAUDE_PLUGIN_ROOT}/shared/istruzioni-progetto.md` con i dati raccolti e mostralo in un blocco da copiare;
- spiega in una frase che va incollato nelle istruzioni del Progetto: è l'unico passo manuale, perché Claude non può modificare le istruzioni di un Progetto.

Se lo studio non usa un Progetto, salta questo passo.

## Cosa dire alla fine

Una riga per ciascuna cosa:
- il centro di comando: creato o aggiornato;
- la configurazione salvata, con i campi principali;
- il controllo orario delle email, con fascia oraria;
- le istruzioni del Progetto da incollare, se serve.

Poi un solo passo successivo: se `allerto/onboarding` non è `completata`, proponi la lezione guidata della skill `inizia-qui` ("Vuole che le mostri come lavorare con me, un argomento alla volta?"); altrimenti un passo naturale, per esempio "Quando vi fidate, possiamo attivare anche i solleciti ai fornitori".
