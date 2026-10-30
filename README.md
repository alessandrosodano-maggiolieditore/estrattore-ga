# Estrattore Pronunce Giustizia Amministrativa

Applicazione web (Node/Express + React) che scarica in blocco i provvedimenti pubblicati su
[giustizia-amministrativa.it — Decisioni e Pareri](https://www.giustizia-amministrativa.it/web/guest/dcsnprr):
ricerca avanzata, scorrimento di tutte le pagine dei risultati, download dei documenti originali
(PDF o HTML, come forniti dal portale), estrazione del testo integrale, indice CSV, report e archivio ZIP.

Ricerca predefinita: parola chiave **"gara"** + **data odierna** (GG/MM/AAAA, Europe/Rome) come frase esatta.

## Avvio

Requisiti: Node.js 22+.

```bash
npm install
npm run dev
```

L'app è su http://localhost:3000.

Se la macchina esce su internet tramite proxy, avviare con:

```bash
NODE_USE_ENV_PROXY=1 npm run dev
```

## Struttura dell'archivio ZIP

```
Sentenze_GA_AAAA-MM-GG/
  documenti_originali/   PDF o HTML così come scaricati dal portale
  documenti_pdf/         copia del PDF originale, oppure PDF generato dal testo per gli originali HTML
  testi_integrali/       testo UTF-8 con intestazione dei metadati
  indice_pronunce.csv    una riga per provvedimento (separatore ;)
  report_estrazione.json / .md
```

## Note

- La data viene cercata come "frase esatta" nel testo: il portale restituisce i provvedimenti che la
  *contengono*. L'estrattore legge la data di pubblicazione reale da ogni documento e segnala nel log
  e nel CSV (colonna `DataConformeRicerca`) quelli pubblicati in un giorno diverso.
- Risultati e documenti sono tenuti in memoria: un riavvio del server li cancella. Scaricare lo ZIP
  prima di riavviare. Per lo stesso motivo le ricerche oltre 5.000 risultati vengono rifiutate.
- Il pianificatore giornaliero funziona solo finché il server resta acceso.
- Lo scraping si basa sull'HTML del portale: se il portale cambia struttura, la sezione
  "Diagnostica Portale" aiuta a capire cosa non va.

## API principali

| Metodo | Percorso | Descrizione |
| :-- | :-- | :-- |
| POST | `/api/search/start` | avvia l'estrazione (`paroleChiave`, `fraseEsatta`, `sede`, `tipoProvvedimento`) |
| POST | `/api/search/pause` · `/resume` · `/stop` | controllo del lavoro in corso |
| GET | `/api/search/status` · `/api/search/events` (SSE) | stato (senza testi integrali) |
| GET | `/api/documents/:id` | singola pronuncia con testo integrale |
| GET | `/api/documents/:id/original` | documento originale (PDF/HTML) |
| GET | `/api/documents/search?q=` | ricerca nel testo delle pronunce acquisite |
| GET | `/api/export/zip` · `/api/export/csv` · `/api/export/report?format=json\|md` | esportazioni |
