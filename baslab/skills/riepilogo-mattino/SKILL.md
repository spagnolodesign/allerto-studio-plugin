---
name: riepilogo-mattino
description: "This skill should be used when a BasLab studio asks for \"il riepilogo del mattino\", \"il riepilogo di oggi\", \"cosa c'è da fare oggi in studio\", \"mandami il riepilogo\", \"manda il riepilogo all'amministratore\", or when a scheduled Allerto task asks to send the morning summary. It reads the command center's live sources (BasLab and the Allerto switchboard), works out what changed since the previous summary, and emails a short brief to the administrator."
metadata:
  version: "1.0.0"
---

# Riepilogo del mattino (BasLab)

Un'email breve all'amministratore con quello che va fatto oggi e quello che è cambiato da ieri, più il link al centro di comando. È un'email interna allo studio.

Le regole su BasLab e sulla scrittura sono in `${CLAUDE_PLUGIN_ROOT}/shared/regole-baslab.md`. Il modello dell'email è in `references/email.md`.

## Procedura

### 1. Contesto

- Trova il centro di comando: dall'indirizzo nel prompt, oppure con `Artifact` `list` e `type: "Dashboard"` cercando il titolo "Centro di comando". Se non esiste, segui prima la skill `centro-di-comando`.
- Leggi `allerto/config` con `ArtifactData`. Senza `emailAmministratore` non mandare niente: prepara il testo e chiedi a chi mandarlo, oppure, in un'esecuzione programmata, mostralo con SendUserMessage.
- Leggi `allerto/riepilogo`, cioè il riepilogo precedente, se esiste.
- Leggi la collection `datasets` del centro di comando. Usa le **stesse sorgenti** della dashboard, così i numeri dell'email coincidono con la pagina.

### 2. Dati

Esegui con il collegamento Allerto-BasLab le sorgenti seguenti, con lo strumento e gli argomenti scritti in ciascun documento `datasets/<id>`. Non riscrivere le sorgenti: usale come sono.

| Sorgente | Serve per |
|---|---|
| `attivita` | attività aperte: da assegnare, urgenti, con data prevista passata |
| `chiamate` | chiamate non gestite |
| `ticket` | ticket di BasLab |
| `rate_aperte` | residuo delle rate già scadute (totale e per condominio) e residuo in scadenza nei prossimi `soglie.giorniRateInScadenza` giorni |
| `assemblee` | assemblee convocate (seconda convocazione da oggi in poi, senza verbale), in particolare nei prossimi 7 giorni |
| `solleciti` | solleciti generati e non ancora inviati (`dataUltimoInvio` vuota) |
| `polizze` | polizze scadute o in scadenza entro `soglie.giorniPolizzaInScadenza` giorni |

Calcola tu, con Bash, i numeri che la pagina calcola nel browser (scaduto a oggi, in scadenza, convocate), con le stesse regole di `index.html`:
- le date di BasLab sono in UTC: convertile al giorno di Roma prima di confrontarle con oggi;
- scaduto = somma del residuo delle scadenze fino a oggi compreso;
- convocata = `dataSeconda` da oggi in poi e `statoVerbale` falso.

### 3. Cosa è cambiato

Confronta con `allerto/riepilogo`:
- attività nuove, per `id` non presenti ieri;
- chiamate non gestite nuove;
- ticket BasLab nuovi;
- assemblee convocate nuove;
- variazione del residuo scaduto.

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
- Nessuno strumento di posta: manda il testo con SendUserMessage e di' che collegando Gmail o Outlook arriverebbe per email.
- Quando la skill viene lanciata come prova da `configura-studio`, crea sempre una bozza.

### 6. Ricorda

Scrivi `allerto/riepilogo` con `set`, oppure `update` con `if_version`:

```json
{
  "at": "<ora ISO>",
  "residuoScaduto": <numero>,
  "residuoInScadenza": <numero>,
  "attivitaAperte": ["<id>", ...],
  "chiamateNonGestite": ["<id>", ...],
  "ticketBaslab": ["<id>", ...],
  "assembleeConvocate": ["<id>", ...],
  "inviataA": "<email o vuoto>",
  "modo": "inviata | bozza | messaggio"
}
```

## Regole

- Mai più di un riepilogo per giorno per lo stesso indirizzo: se `allerto/riepilogo.at` è di oggi e `modo` è `inviata`, non rimandare. Dillo, a meno che l'utente non lo chieda esplicitamente.
- Se il collegamento Allerto-BasLab non risponde, non mandare un'email vuota. Manda una sola riga, "Il riepilogo di oggi non è disponibile: il collegamento ad Allerto e BasLab non risponde", e fermati.
- Se una sola sorgente fallisce (per esempio un'area di BasLab non abilitata), manda il riepilogo senza quella parte e scrivilo in fondo in una riga.
