# Il copione delle lezioni (BasLab)

Per ogni lezione: cosa spiegare, cosa mostrare dal vivo, le frasi da insegnare e l'esercizio. È una traccia, non un testo da leggere: adattala a chi hai davanti e ai dati dello studio.

Le dimostrazioni sono **sempre in sola lettura** e seguono `${CLAUDE_PLUGIN_ROOT}/shared/regole-baslab.md` (esercizio in corso, morosi, date in UTC da portare al fuso di Roma). BasLab si legge in tempo reale: quello che lo studio inserisce oggi si vede subito.

---

## 1. `come-si-lavora` · Come si lavora con Claude nello studio

**Spiega**
- Si lavora dentro il Progetto dello studio: lì Claude conosce le regole, i documenti e il centro di comando.
- Una chat per area: Email, Assemblee, Fornitori e lavori, Contabilità e morosità, Archivio documenti, Scadenze. Quando una chat diventa lunga se ne apre una nuova della stessa area. Claude lo suggerisce da solo, una volta per tema.
- Le regole che valgono sempre: Claude prepara, l'amministratore decide e invia. Niente email inviate, niente scritture in BasLab, niente dati di un condòmino a terzi senza delega, niente numeri inventati. Ogni risposta dice da dove viene il dato.

**Mostra**: nessuna dimostrazione. Elenca quali collegamenti sono attivi e quali mancano, in una riga ciascuno.

**Tocca a te**: "Secondo lei, in quale chat chiederebbe a che punto è la morosità del condominio X?" Commenta la risposta (Contabilità e morosità).

---

## 2. `baslab` · Chiedere a BasLab in parole semplici

**Spiega**: Claude legge il gestionale BasLab dello studio, in sola lettura e in tempo reale, e risponde a domande normali: niente menu, niente report da costruire. Ogni risposta indica la fonte e, per le rate, l'esercizio.

**Mostra** (una sola, la più utile per lo studio):
- quanti condomini ci sono (`elenca_condomini`) e quante assemblee sono convocate da oggi in poi (`Assemblea/dx/list`);
- oppure, per un condominio scelto dall'amministratore, il totale scaduto e il numero di morosi dell'ordinario in corso (`morosi`), senza nomi.

Dopo il risultato, fai notare la riga della fonte e l'esercizio mostrato. Se c'è un ordinario precedente ancora aperto, dillo: i suoi morosi restano lì finché non si chiude il consuntivo.

**Frasi da insegnare**
- "Quanto deve il condòmino Rossi del condominio Ripamonti 128?"
- "Chi abita nell'unità A3 del condominio X, e chi è il proprietario?"
- "Quali esercizi sono aperti nel condominio X?"

**Tocca a te**: "Mi chieda una cosa che oggi cercherebbe in BasLab." Eseguila, rispondi con la fonte e spiega in una riga dove l'hai trovata.

---

## 3. `centralino` · Le chiamate del centralino

**Spiega**: il centralino Allerto risponde alle chiamate e le trascrive. Claude le legge, riconosce il condominio e prepara la risposta o l'attività. Le attività in Allerto si creano solo quando l'amministratore lo chiede.

**Mostra**: `ultimeChiamate` con stato "tutte": quante chiamate negli ultimi giorni, quante non gestite, e il riassunto di una (chi, condominio, cosa chiede). Per il dettaglio usa `dettaglioChiamata`. Se il chiamante non dice il nome, mostra come lo ritrovi in BasLab con `cerca_persone` sulle ultime cifre del numero.

**Frasi da insegnare**
- "Chi ha chiamato oggi e cosa voleva?"
- "Chi è il condòmino che ha chiamato dal 333 123 4567?"
- "Prepara la risposta per la chiamata della signora Bianchi."

**Tocca a te**: "Scelga una chiamata e mi chieda cosa farne." Proponi la risposta; se l'amministratore vuole l'attività, segui la skill `apri-ticket` (controllo dei doppioni compreso).

---

## 4. `email` · Le bozze di risposta

**Spiega**: per le email che chiedono una risposta, Claude scrive una bozza nel thread, con i dati presi da BasLab e firmata come configurato. Le bozze restano in Gmail con l'etichetta "Allerto/Bozza pronta": l'amministratore le rilegge, completa i punti segnati "[DA COMPLETARE]" e preme Invia. Se è attivo, il controllo orario della posta lo fa da solo e manda una notifica.

**Mostra**: cerca in sola lettura le email recenti che chiedono una risposta (come la skill `bozze-email`, passo 2) e mostra quante sono e l'oggetto di due o tre. Non creare bozze.

**Proponi**: "Vuole che prepari la bozza per una di queste, così vede com'è fatta?" Solo con un sì, crea **una** bozza seguendo la skill `bozze-email` e dai il link per aprirla.

**Frasi da insegnare**
- "Controlla la posta."
- "Rispondi all'email del condòmino che chiede il suo estratto conto."
- "Rendi la bozza più breve / più formale."

**Tocca a te**: "Apra la bozza in Gmail e mi dica cosa cambierebbe." Applica la modifica alla bozza (`update_draft`) se lo chiede.

---

## 5. `ticket` · Dalla risposta al ticket

**Spiega**: dopo ogni risposta a una email o a una chiamata, Claude chiede in una riga se aprire un ticket. Prima controlla se ne esiste già uno, sia in BasLab sia tra le attività di Allerto. Apre l'attività in Allerto (se lo strumento c'è) e prepara il testo pronto da inserire come ticket in BasLab: condominio, oggetto, descrizione, priorità, richiedente, recapito.

**Mostra**: i ticket in BasLab (`Ticket/paged-list`) e le attività aperte in Allerto, quanti e uno come esempio. Se sono zero, dillo: è un buon momento per cominciare.

**Frasi da insegnare**
- "C'è già un ticket aperto per l'ascensore del condominio X?"
- "Apri un ticket per questa chiamata."
- "Preparami il ticket da inserire in BasLab."

**Tocca a te**: "Mi descriva un guasto a voce, come farebbe un condòmino." Prepara il testo del ticket con i campi mancanti segnati "da completare".

---

## 6. `centro-di-comando` · La dashboard e la memoria condivisa

**Spiega**
- Il centro di comando è la pagina dello studio: attività, ticket, rate scadute non incassate, solleciti, assemblee, polizze, chiamate, letti da BasLab a ogni apertura.
- Le rate della pagina sono il residuo di ogni emissione già scaduta, su tutte le gestioni: per i morosi per nome si chiede in chat.
- Dentro c'è la memoria condivisa: decisioni, accordi e lavori fatti che tutta la squadra e Claude conoscono. Quello che scrive uno, lo sanno tutti, in ogni chat.
- Si condivide solo con la squadra, con accesso da editor: contiene nomi e importi dei condòmini.

**Mostra**: apri il centro di comando (`Artifact` `open`) e indica il numero più importante di oggi (per esempio il residuo scaduto o le assemblee convocate). Poi leggi le ultime voci della memoria (`memoria`).

**Proponi**: "Vuole provare a scrivere la prima voce nella memoria?" Solo con un sì, registrala con il suo nome come `chi`.

**Frasi da insegnare**
- "Segna che abbiamo deciso di cambiare l'impresa di pulizie al condominio X."
- "Fatto: ho chiamato il fornitore dell'ascensore."
- "Cosa abbiamo deciso sul condominio X nell'ultimo mese?"

**Tocca a te**: "Mi dica una decisione presa questa settimana." Registrala (con il suo sì) e mostrala nella pagina.

---

## 7. `riepilogo` · Il riepilogo del mattino

**Spiega**: ogni mattina feriale, all'orario scelto, Claude manda all'amministratore un'email breve con le cinque cose da fare oggi e cosa è cambiato da ieri, più il link al centro di comando. È un'attività programmata: parte da sola, anche quando nessuno apre Claude.

**Mostra**: leggi l'impostazione in `config.riepilogo` (orario, giorni, invio) e, con `list_triggers`, se l'attività "Allerto · riepilogo del mattino" esiste ed è attiva.

**Proponi**: "Vuole vedere quello di domani in anteprima?" Solo con un sì, esegui la skill `riepilogo-mattino` in modalità bozza.

**Frasi da insegnare**
- "Mandami il riepilogo."
- "Sposta il riepilogo alle 8."
- "Cosa c'è da fare oggi in studio?"

**Tocca a te**: "Le va bene l'orario attuale?" Se vuole cambiarlo, segui la skill `configura-studio` per la sola modifica dell'orario.

---

## 8. `documenti` · L'archivio su Drive e i verbali

**Spiega**
- Ogni condominio ha la sua cartella su Drive, con lo stesso nome del condominio in BasLab, e le sottocartelle standard (01 Assemblee, 02 Bilanci e rendiconti, …, 99 Da smistare).
- I file si chiamano `AAAA-MM-GG Tipo - descrizione.pdf`, con la data del documento. Quello caricato in fretta va in "99 Da smistare": Claude lo rinomina e lo sposta.
- I verbali: dagli appunti o da un vocale Claude prepara il testo, secondo la modalità dello studio (A verbale in BasLab, B PDF su Drive, C entrambi). Sulle delibere fa fede il verbale firmato.

**Mostra**: cerca su Drive la cartella `Condomini` e indica quanti condomini hanno la cartella e se "99 Da smistare" contiene file. Leggi `config.modalitaVerbali`; se manca, chiedila ora.

**Frasi da insegnare**
- "Trova il regolamento del condominio X."
- "Riordina i file da smistare del condominio X."
- "Ecco i miei appunti dell'assemblea: preparami il verbale."

**Tocca a te**: "Mi chieda un documento che le serve spesso." Cercalo e indica cartella e nome del file. Spostamenti e rinomine solo con conferma.

---

## 9. `scadenze` · Agenda, assemblee e scadenze

**Spiega**: Claude incrocia le scadenze di BasLab (rate in arrivo, esercizi in chiusura, assemblee convocate, polizze) con l'agenda dello studio, e le porta nel centro di comando e nel riepilogo. Gli eventi in calendario si creano solo con conferma.

**Mostra**: le scadenze dei prossimi 30 giorni da BasLab (rate da `EmissioneRata/scadenze/dx/list`, assemblee convocate da `Assemblea/dx/list`) e, se Calendar è collegato, gli impegni di oggi. Ricorda di portare le date al fuso di Roma.

**Frasi da insegnare**
- "Quali assemblee ho nei prossimi due mesi?"
- "Quali rate scadono questo mese, e quanto manca da incassare?"
- "Metti in agenda il sopralluogo di giovedì al condominio X." (con conferma)

**Tocca a te**: "Qual è la prossima scadenza che le preoccupa?" Cercala e rispondi con data e fonte.

---

## Il promemoria finale (modello)

| Per… | Scrivi… |
|---|---|
| Chiedere a BasLab | "Quanto deve il condòmino … del condominio …?" |
| Le chiamate | "Chi ha chiamato oggi e cosa voleva?" |
| La posta | "Controlla la posta." |
| I ticket | "Apri un ticket per questo guasto." |
| La memoria | "Segna che abbiamo deciso…" |
| Il riepilogo | "Mandami il riepilogo." |
| I documenti | "Trova il regolamento del condominio …" |
| Le scadenze | "Quali assemblee ho nei prossimi due mesi?" |
| Ripassare | "Riprendiamo la lezione." |

Tieni solo le righe delle lezioni fatte.
