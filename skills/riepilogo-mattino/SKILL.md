---
name: riepilogo-mattino
description: "This skill should be used when the studio asks for \"il riepilogo del mattino\", \"il riepilogo di oggi\", \"cosa c'è da fare oggi in studio\", \"mandami il riepilogo\", \"manda il riepilogo all'amministratore\", or when a scheduled Allerto task asks to send the morning summary. It reads the command center's live sources (Danea and the Allerto switchboard), works out what changed since the previous summary, and emails a short brief to the administrator.\n"
metadata:
  version: "0.1.0"
---

# Riepilogo del mattino

Un'email breve all'amministratore con quello che va fatto oggi e quello che è cambiato da ieri, più il link al centro di comando. È un'email interna allo studio.

Le regole su Danea e sulla scrittura sono in `${CLAUDE_PLUGIN_ROOT}/shared/regole-danea.md`. Il modello dell'email è in `references/email.md`.

## Procedura

### 1. Contesto

- Trova il centro di comando: dall'indirizzo nel prompt, oppure con `Artifact` `list` e `type: "Dashboard"` cercando il titolo "Centro di comando". Se non esiste, segui prima la skill `centro-di-comando`.
- Leggi `allerto/config` con `ArtifactData`. Senza `emailAmministratore` non mandare niente: prepara il testo e chiedi a chi mandarlo, oppure, in un'esecuzione programmata, mostralo con SendUserMessage.
- Leggi `allerto/riepilogo`, cioè il riepilogo precedente, se esiste.
- Leggi la collection `datasets` del centro di comando. Usa le **stesse sorgenti** della dashboard, così i numeri dell'email coincidono con la pagina.

### 2. Dati

Esegui con `allerto-brain` le sorgenti seguenti, con lo strumento e gli argomenti scritti in ciascun documento `datasets/<id>`:

| Sorgente | Serve per |
|---|---|
| `kpi` | segnalazioni aperte, interventi in ritardo o previsti oggi, premi scaduti o in scadenza, assemblee convocate, più la data del backup Danea |
| `interventi` | elenco degli interventi da seguire (ritardo, fornitore, telefono) |
| `attivita` | attività aperte: da assegnare, urgenti, con data prevista passata |
| `chiamate` | chiamate non gestite |
| `assicurazioni` | premi scaduti o in scadenza |
| `morosita_tot` | totale scoperto e numero di posizioni |

Non riscrivere le query: usale come sono.

### 3. Cosa è cambiato

Confronta con `allerto/riepilogo`:
- attività nuove, per `id` non presenti ieri;
- interventi entrati in ritardo;
- chiamate non gestite nuove;
- variazione dello scoperto totale.

Senza un riepilogo precedente, tratta tutto come nuovo e non scrivere "rispetto a ieri".

### 4. Scrivi l'email

Segui `references/email.md`. In breve:
- oggetto con il numero di cose da fare;
- in cima al massimo 5 cose da fare, in ordine di urgenza;
- poi i numeri e il link.

Usa solo dati letti al passo 2: nessuna stima, nessun numero ricostruito a mente.

### 5. Invia

- Cerca lo strumento di posta con ToolSearch: "gmail send", poi "gmail draft", poi "outlook send mail".
- `riepilogo.invio = "automatico"` e strumento di invio presente: manda a `emailAmministratore`, in testo semplice, con la `firma` in fondo.
- `riepilogo.invio = "bozza"`, oppure nessuno strumento di invio ma uno per le bozze: crea una bozza indirizzata a `emailAmministratore`.
- Nessuno strumento di posta: manda il testo con SendUserMessage e dì che collegando Gmail o Outlook arriverebbe per email.
- Quando la skill viene lanciata come prova da `configura-studio`, crea sempre una bozza.

### 6. Ricorda

Scrivi `allerto/riepilogo` con `set`, oppure `update` con `if_version`:

```json
{
  "at": "<ora ISO>",
  "kpi": { ...la riga di kpi... },
  "scoperto": <SCOPERTO di morosita_tot>,
  "attivitaAperte": ["<id>", ...],
  "interventiInRitardo": ["<ID>", ...],
  "chiamateNonGestite": ["<id>", ...],
  "inviataA": "<email o vuoto>",
  "modo": "inviata | bozza | messaggio"
}
```

## Regole

- Mai più di un riepilogo per giorno per lo stesso indirizzo: se `allerto/riepilogo.at` è di oggi e `modo` è `inviata`, non rimandare. Dillo, a meno che l'utente non lo chieda esplicitamente.
- Se `allerto-brain` non risponde, non mandare un'email vuota. Manda una sola riga, "Il riepilogo di oggi non è disponibile: il collegamento ad Allerto non risponde", e fermati.
- Se il backup Danea ha più di un giorno, scrivilo in cima: i numeri di Danea potrebbero essere vecchi.
