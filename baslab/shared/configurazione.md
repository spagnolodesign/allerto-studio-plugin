# Configurazione dello studio

La configurazione di ogni studio vive nel database del suo centro di comando: collection `allerto`, documento `config`. Così la trovano tutte le skill e le attività programmate, che partono da sessioni nuove, conoscendo solo l'indirizzo del centro di comando.

Si legge e si scrive con lo strumento `ArtifactData` (`get` / `set` / `update` con `if_version`), passando `url` = indirizzo del centro di comando.

## Campi

```json
{
  "studio": "Studio Rossi Amministrazioni",
  "amministratore": "Mario Rossi",
  "emailAmministratore": "mario@studiorossi.it",
  "firma": "Studio Rossi Amministrazioni\nVia Roma 1, Milano · 02 000000",
  "gestionale": "BasLab",
  "connettore": "Allerto Brain BASLAB",
  "centroDiComando": "https://claude.ai/artifact/…",
  "modalitaVerbali": "C",
  "riepilogo": {
    "attivo": true,
    "orario": "07:45",
    "giorni": "lun-ven",
    "fusoOrario": "Europe/Rome",
    "invio": "automatico"
  },
  "bozzeEmail": {
    "attivo": true,
    "orario": "08-19",
    "giorni": "lun-ven",
    "massimoPerGiro": 15
  },
  "soglie": {
    "giorniRateInScadenza": 30,
    "giorniPolizzaInScadenza": 60
  },
  "regoleBaslab": {
    "assembleaTipoStraordinaria": 1
  },
  "versionePlugin": "1.0.0",
  "paginaSha256": "…",
  "aggiornato": "2026-11-04T17:00:00+01:00"
}
```

- `gestionale`: sempre `BasLab` per questo plugin. Se uno studio usa Danea, serve l'altro plugin (`allerto-studio`).
- `connettore`: il nome del connettore Allerto collegato a BasLab, come lo vuole il tipo Dashboard nelle sorgenti. Lo trova `configura-studio` (è il connettore che offre `elenca_condomini` e `interroga_gestionale`). La skill `centro-di-comando` lo mette al posto di `{{connettore}}` in `datasets.json`.
- `riepilogo.invio`: `automatico` (manda l'email) oppure `bozza` (la lascia in bozza nella posta). Il riepilogo è interno allo studio, quindi `automatico` è il default.
- `modalitaVerbali`: come lo studio archivia i verbali. `A` = solo verbale in BasLab (assemblea → verbale), `B` = solo PDF su Drive, `C` = entrambi. La usa la skill `regole-studio`.
- `bozzeEmail`: il controllo orario della posta della skill `bozze-email`. L'avviso arriva sempre come notifica dell'attività programmata, mai per email.
- `soglie.giorniRateInScadenza`: finestra delle rate "in scadenza" nel riepilogo (la dashboard usa 30 giorni).
- `regoleBaslab`: convenzioni dello studio diverse da quelle di `shared/regole-baslab.md`.
- `versionePlugin` e `paginaSha256`: li scrive la skill `centro-di-comando` per sapere se c'è un aggiornamento e se la pagina è stata modificata a mano.

Se un campo manca, usa il valore dell'esempio, tranne i dati dello studio, le email e il connettore: quelli non vanno mai inventati.
