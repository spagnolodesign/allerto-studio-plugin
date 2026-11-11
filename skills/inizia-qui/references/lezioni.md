# Il copione delle lezioni

Per ogni lezione: cosa spiegare, cosa mostrare dal vivo, le frasi da insegnare e l'esercizio. È una traccia, non un testo da leggere: adattala a chi hai davanti e ai dati dello studio.

Le dimostrazioni sono **sempre in sola lettura**. Le query seguono `${CLAUDE_PLUGIN_ROOT}/shared/regole-danea.md` (stato 3 = chiusa, esercizi in corso, scoperto). Danea arriva dal backup notturno: se il campo `backup` della risposta è vecchio di più di un giorno, dillo.

---

## 1. `come-si-lavora` · Come si lavora con Claude nello studio

**Spiega**
- Si lavora dentro il Progetto dello studio: lì Claude conosce le regole, i documenti e il centro di comando.
- Una chat per area: Email, Assemblee, Fornitori e lavori, Contabilità e morosità, Archivio documenti, Scadenze. Quando una chat diventa lunga se ne apre una nuova della stessa area. Claude lo suggerisce da solo, una volta per tema.
- Le regole che valgono sempre: Claude prepara, l'amministratore decide e invia. Niente email inviate, niente scritture in Danea, niente dati di un condòmino a terzi senza delega, niente numeri inventati. Ogni risposta dice da dove viene il dato.

**Mostra**: nessuna dimostrazione. Elenca quali collegamenti sono attivi e quali mancano, in una riga ciascuno.

**Tocca a te**: "Secondo lei, in quale chat chiederebbe a che punto è la morosità del condominio X?" Commenta la risposta (Contabilità e morosità).

---

## 2. `danea` · Chiedere a Danea in parole semplici

**Spiega**: Claude legge l'archivio Danea dello studio, in sola lettura, e risponde a domande normali: niente menu, niente report da costruire. Ogni risposta indica la fonte.

**Mostra** (una sola, la più utile per lo studio):
- quanti condomini sono in archivio e quanti hanno un esercizio ordinario in corso (`TCONDOMINI_GENERALE`, `TCONDOMINI`);
- oppure il totale dello scoperto sulle rate scadute e il numero di posizioni, senza nomi.

Dopo il risultato, fai notare la riga della fonte e la data del backup.

**Frasi da insegnare**
- "Quanto deve il condòmino Rossi del condominio Diamante?"
- "Quali delibere ha preso l'ultima assemblea del condominio X?"
- "Qual è l'IBAN del condominio X?"

**Tocca a te**: "Mi chieda una cosa che oggi cercherebbe in Danea." Eseguila, rispondi con la fonte e spiega in una riga dove l'hai trovata.

---

## 3. `centralino` · Le chiamate del centralino

**Spiega**: il centralino Allerto risponde alle chiamate e le trascrive. Claude le legge, riconosce il condominio e prepara la risposta o l'attività. Le attività in Allerto si creano solo quando l'amministratore lo chiede.

**Mostra**: `ultimeChiamate` con stato "tutte": quante chiamate negli ultimi giorni, quante non gestite, e il riassunto di una (chi, condominio, cosa chiede). Per il dettaglio usa `dettaglioChiamata`.

**Frasi da insegnare**
- "Chi ha chiamato oggi e cosa voleva?"
- "Prepara la risposta per la chiamata della signora Bianchi."
- "Apri un'attività per questa chiamata." (solo questa scrive in Allerto)

**Tocca a te**: "Scelga una chiamata e mi chieda cosa farne." Proponi la risposta; se l'amministratore vuole l'attività, chiedi conferma prima di `creaAttivita` e controlla con `attivita` che non ce ne sia già una.

---

## 4. `email` · Le bozze di risposta

**Spiega**: per le email che chiedono una risposta, Claude scrive una bozza nel thread, con i dati presi da Danea e firmata come configurato. Le bozze restano in Gmail con l'etichetta "Allerto/Bozza pronta": l'amministratore le rilegge, completa i punti segnati "[DA COMPLETARE]" e preme Invia. Se è attivo, il controllo orario della posta lo fa da solo e manda una notifica.

**Mostra**: cerca in sola lettura le email recenti che chiedono una risposta (come la skill `bozze-email`, passo 2) e mostra quante sono e l'oggetto di due o tre. Non creare bozze.

**Proponi**: "Vuole che prepari la bozza per una di queste, così vede com'è fatta?" Solo con un sì, crea **una** bozza seguendo la skill `bozze-email` e dai il link per aprirla.

**Frasi da insegnare**
- "Controlla la posta."
- "Rispondi all'email dell'idraulico sul condominio X."
- "Rendi la bozza più breve / più formale."

**Tocca a te**: "Apra la bozza in Gmail e mi dica cosa cambierebbe." Applica la modifica alla bozza (`update_draft`) se lo chiede.

---

## 5. `ticket` · Dalla risposta al ticket in Danea

**Spiega**: dopo ogni risposta a una email o a una chiamata, Claude chiede in una riga se aprire un ticket (segnalazione) in Danea. Prima controlla se ne esiste già uno aperto sullo stesso tema. Siccome in Danea non scrive, prepara il testo da incollare: titolo, condominio, priorità, descrizione, richiedente, recapito.

**Mostra**: le segnalazioni aperte in Danea (`TSEGNALAZ` con `STATO <> 3`), quante per priorità, e una come esempio.

**Frasi da insegnare**
- "C'è già una segnalazione aperta per l'ascensore del condominio X?"
- "Preparami il ticket da incollare in Danea."

**Tocca a te**: "Mi descriva un guasto a voce, come farebbe un condòmino." Prepara il testo del ticket con i campi mancanti segnati "da completare".

---

## 6. `centro-di-comando` · La dashboard e la memoria condivisa

**Spiega**
- Il centro di comando è la pagina dello studio: attività, interventi, segnalazioni, morosità, assemblee, polizze, chiamate, sempre aggiornati a ogni apertura.
- Dentro c'è la memoria condivisa: decisioni, accordi e lavori fatti che tutta la squadra e Claude conoscono. Quello che scrive uno, lo sanno tutti, in ogni chat.
- Si condivide solo con la squadra, con accesso da editor: contiene nomi e debiti dei condòmini.

**Mostra**: apri il centro di comando (`Artifact` `open`) e indica il numero più importante di oggi (per esempio gli interventi in ritardo). Poi leggi le ultime voci della memoria (`memoria`).

**Proponi**: "Vuole provare a scrivere la prima voce nella memoria?" Solo con un sì, registrala con il suo nome come `chi`.

**Frasi da insegnare**
- "Segna che abbiamo deciso di cambiare l'impresa di pulizie al condominio X."
- "Fatto: ho chiamato il fornitore dell'ascensore."
- "Cosa abbiamo deciso sul condominio X nell'ultimo mese?"

**Tocca a te**: "Mi dica una decisione presa questa settimana." Registrala (con il suo sì) e mostrala nella pagina.

---

## 7. `documenti` · L'archivio su Drive e i verbali

**Spiega**
- Ogni condominio ha la sua cartella su Drive, con lo stesso nome dell'intestazione in Danea, e le sottocartelle standard (01 Assemblee, 02 Bilanci e rendiconti, …, 99 Da smistare).
- I file si chiamano `AAAA-MM-GG Tipo - descrizione.pdf`, con la data del documento. Quello caricato in fretta va in "99 Da smistare": Claude lo rinomina e lo sposta.
- I verbali: dagli appunti o da un vocale Claude prepara il testo, secondo la modalità dello studio (A moduli Danea, B PDF su Drive, C entrambi). Sulle delibere fa fede il verbale firmato.

**Mostra**: cerca su Drive la cartella `Condomini` e indica quanti condomini hanno la cartella e se "99 Da smistare" contiene file. Leggi `config.modalitaVerbali`; se manca, chiedila ora.

**Frasi da insegnare**
- "Trova il regolamento del condominio X."
- "Riordina i file da smistare del condominio X."
- "Ecco i miei appunti dell'assemblea: preparami il verbale."

**Tocca a te**: "Mi chieda un documento che le serve spesso." Cercalo e indica cartella e nome del file. Spostamenti e rinomine solo con conferma.

---

## 8. `scadenze` · Agenda, assemblee e scadenze

**Spiega**: Claude incrocia le scadenze di Danea (rate in arrivo, esercizi in chiusura o scaduti, assemblee convocate, polizze) con l'agenda dello studio, e le porta nel centro di comando. Gli eventi in calendario si creano solo con conferma.

**Mostra**: le scadenze dei prossimi 30 giorni da Danea (rate, chiusure di esercizio, assemblee convocate) e, se Calendar è collegato, gli impegni di oggi.

**Frasi da insegnare**
- "Quali assemblee ho nei prossimi due mesi?"
- "Quali esercizi devo chiudere entro fine anno?"
- "Metti in agenda il sopralluogo di giovedì al condominio X." (con conferma)

**Tocca a te**: "Qual è la prossima scadenza che le preoccupa?" Cercala e rispondi con data e fonte.

---

## Il promemoria finale (modello)

| Per… | Scrivi… |
|---|---|
| Chiedere a Danea | "Quanto deve il condòmino … del condominio …?" |
| Le chiamate | "Chi ha chiamato oggi e cosa voleva?" |
| La posta | "Controlla la posta." |
| I ticket | "Preparami il ticket da incollare in Danea." |
| La memoria | "Segna che abbiamo deciso…" |
| I documenti | "Trova il regolamento del condominio …" |
| Le scadenze | "Quali assemblee ho nei prossimi due mesi?" |
| Ripassare | "Riprendiamo la lezione." |

Tieni solo le righe delle lezioni fatte.
