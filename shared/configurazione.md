# Configurazione dello studio

La configurazione di ogni studio vive nel database del suo centro di comando: collection `allerto`, documento `config`. Così la trovano tutte le skill e le attività programmate, che partono da sessioni nuove, conoscendo solo l'indirizzo del centro di comando.

Si legge e si scrive con lo strumento `ArtifactData` (`get` / `set` / `update` con `if_version`), passando `url` = indirizzo del centro di comando.

## Campi

```json
{
  "studio": "Studio Rossi Amministrazioni",
  "amministratore": "Mario Rossi",
  "emailAmministratore": "mario@studiorossi.it",
  "firma": "Studio Rossi Amministrazioni\nVia Roma 1, Padova · 049 000000",
  "centroDiComando": "https://claude.ai/artifact/…",
  "modalitaVerbali": "C",
  "riepilogo": {
    "attivo": true,
    "orario": "07:45",
    "giorni": "lun-ven",
    "fusoOrario": "Europe/Rome",
    "invio": "automatico"
  },
  "soglie": {
    "giorniRitardoIntervento": 0,
    "giorniPremioInScadenza": 60
  },
  "regoleDanea": {
    "statoChiusa": 3
  },
  "versionePlugin": "0.2.0",
  "aggiornato": "2026-11-02T21:00:00+01:00"
}
```

- `riepilogo.invio`: `automatico` (manda l'email) oppure `bozza` (la lascia in bozza nella posta). Il riepilogo è interno allo studio, quindi `automatico` è il default.
- `modalitaVerbali`: come lo studio archivia i verbali. `A` = solo moduli Danea, `B` = solo PDF su Drive, `C` = entrambi. La usa la skill `regole-studio`.
- `soglie.giorniRitardoIntervento`: da quanti giorni di ritardo un intervento finisce nel riepilogo (0 = appena passa la data prevista).
- `versionePlugin`: la versione del plugin che ha scritto per ultimo il centro di comando. La skill `centro-di-comando` la confronta con la propria per sapere se c'è un aggiornamento.

Se un campo manca, usa il valore dell'esempio, tranne i dati dello studio e le email: quelli non vanno mai inventati e si chiedono all'amministratore.
