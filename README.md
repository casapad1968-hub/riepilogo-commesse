# Riepilogo Commesse

Applicazione web a **singolo file HTML** per il monitoraggio delle commesse industriali: dashboard esecutiva, quadro sinottico dei ritardi, Gantt stile MS Project, scadenziario, diario e archivio documenti.

Nata da un'esigenza concreta: gestire decine di commesse in produzione senza aprire un foglio Excel per ogni controllo.

- **Italiano:** `Commesse.html`
- **English:** `Orders.html`

---

## Cosa fa

### Dashboard operativa

Home su tre colonne:

- **Sinistra** — quattro indicatori sintetici: commesse attive, in linea, in ritardo, in attenzione.
- **Centro** — avvisi sulle consegne entro 15 giorni, filtri rapidi (Tutte / In ritardo / Attenzione / In linea) e tabella di riepilogo con semaforo, barra di avanzamento e note critiche. Clic sulla riga per aprire la commessa.
- **Destra** — **albero delle commesse**: ricerca, commesse espandibili con contatore fasi, fasi annidate con percentuale. Clic su un nodo per aprire il dettaglio.

### Quadro sinottico completo

Una sola tabella con, per ogni commessa: date di apertura e consegna da ordine, attività in ritardo, ritardi su emissione ordini e arrivo materiali, giorni di ritardo sull'inizio fabbricazione, scostamento dell'ultimo WP rispetto alla consegna contrattuale. Le celle critiche si colorano da sole.

### Import da MS Project

Carichi il file XML esportato da MS Project e l'applicazione:

- ricostruisce l'albero delle attività rispettando i livelli di outline
- calcola i ritardi confrontando la data di fine prevista con oggi, solo sulle attività non chiuse al 100%
- conta separatamente i ritardi dentro i blocchi `PO Issuance` (MR) e i `Material Deliverables` scaduti
- rileva la data di inizio fabbricazione cercando le attività `MANUFACTURING` / `FABRICATION`
- individua la consegna prevista dall'ultimo WP tramite le attività `DELIVERY`
- importa le dipendenze tra fasi leggendo `PredecessorLink` e traducendole in nomi attività

### Gantt stile MS Project

Due viste:

**Riepilogo progetti** (scheda dedicata) — tutte le commesse attive su un'unica scala temporale, una riga riassuntiva per commessa, espandibile sulle singole fasi. Filtri per criticità, ritardo e scadenza. Sola lettura.

**Dettaglio commessa** — Gantt completo ed editabile:

- scala Giorni / Settimane / Mesi con zoom, linea rossa OGGI, weekend ombreggiati
- dipendenze con frecce (anche in rosso se entrambe le fasi sono critiche)
- **percorso critico** calcolato con CPM (forward/backward pass)
- milestone a rombo, righe di sintesi con barra MS Project
- opzione **solo giorni lavorativi** per il calcolo delle durate
- modifica diretta: trascinamento barre, ridimensionamento bordi, doppio clic per la finestra completa
- campo percentuale modificabile direttamente in tabella
- creazione ed eliminazione fasi
- export PNG e stampa A4 orizzontale

Per una commessa **senza file Project**, il Gantt offre due pulsanti: `＋ Nuova fase` per aggiungere le fasi una a una, oppure `🗂️ Crea struttura base` che genera 14 fasi già pronte (PO ISSUANCE, MATERIAL DELIVERABLES, MANUFACTURING, DELIVERY) con date calcolate dalla data apertura velina e dalla consegna ordine, e dipendenze già collegate.

### Scadenziario manuale

Attività che non stanno dentro MS Project (solleciti, invii, controlli) con stato, data fine e giorni di ritardo calcolati.

### Diario e problematiche

Annotazioni e allerte critiche per commessa, con storico consultabile.

### Documenti PDF

Archivio documenti per commessa, con descrizione, nome file originale e data di caricamento. I PDF **non** finiscono nel JSON: vengono salvati in `archivio_pdf/<numero_commessa>/` tramite File System Access API, con fallback su IndexedDB. Il database resta leggero.

### Archivio commesse concluse

Le commesse chiuse escono dalla home ma restano consultabili e ripristinabili.

### Densità tabelle

Pulsante nell'header che alterna **Spaziosa** (default: righe alte, descrizione visibile, note per esteso) e **Compatta** (righe a metà altezza, descrizione nascosta, note troncate, barre sottili). La preferenza è salvata in `localStorage`.

### Stampa

Report stampabile di tutte le commesse o della singola commessa, impaginato in orizzontale su A4.

---

## Provala subito

Nella cartella [`demo/`](demo/) c'è `database_commesse.json`: un database dimostrativo con **dati completamente inventati** (clienti "Cliente A/B", officine "Officina 1/2").

Contiene due commesse:

- **70777** — in ritardo: 1 MR scaduto, 1 ordine scaduto, 2 materiali scaduti, inizio fabbricazione +11 giorni, ultimo WP a +36 giorni dalla consegna ordine. Due avanzamenti e un'allerta nel diario.
- **70812** — in linea: tutti i contatori a zero.

Più quattro scadenze manuali e tre voci di diario.

**Attenzione:** se salvi mentre è aperto il database demo, sovrascrivi quel file. Tienilo in una cartella separata dai dati reali.

---

## Come si usa

1. Scarica `Commesse.html` (o `Orders.html` per la versione inglese)
2. Aprilo con **Microsoft Edge** o **Google Chrome** aggiornato (serve il supporto a `File System Access API`)
3. Al primo avvio puoi lavorare sui dati dimostrativi incorporati, oppure collegare il tuo database con **Collega Archivio Condiviso**
4. Se non ne hai uno, l'applicazione ne crea uno vuoto al primo salvataggio

Non c'è installazione, non c'è server, non c'è account. Il file HTML è l'applicazione.

---

## Dove stanno i dati

Tutto in locale. Nessun dato lascia il computer.

L'applicazione scrive in tre posti:

- **`database_commesse.json`** — il database vero e proprio, nella cartella che scegli tu. Puoi metterlo su un disco di rete per condividerlo tra più postazioni.
- **`archivio_pdf/`** — i PDF allegati, suddivisi per numero di commessa.
- **`localStorage` / IndexedDB del browser** — copia di lavoro e fallback per i PDF.

Il file JSON **non è mai dentro il repository**: è escluso da `.gitignore`. Il codice è pubblico, i dati no.

---

## Multi-utente

Basato su **file di lock** in stile Excel, con controllo dei timestamp:

- all'apertura, se un altro operatore ha il database in modifica, il secondo utente riceve una schermata bloccante con nome operatore e ora di presa del lock
- l'accesso passa automaticamente in **sola visualizzazione**: pulsanti di scrittura disattivati, banner rosso con pulsante di ricarica
- il lock viene rinnovato periodicamente e rilasciato su `pagehide` / `beforeunload`
- prima di ogni salvataggio viene confrontato il `lastModified` del file: se è cambiato, l'app avvisa del conflitto e propone di ricaricare

Un solo utente alla volta in scrittura. Più lettori contemporanei sono possibili.

---

## Requisiti tecnici

Nessuna dipendenza. Zero build step. Solo HTML, CSS e JavaScript vanilla in un unico file.

Nessuna chiamata di rete in uso locale: font di sistema, icone Unicode, grafici generati in DOM/SVG/Canvas.

---

## Compatibilità

| Browser | Supporto |
|---|---|
| Chrome / Edge (desktop) | Completo |
| Firefox | Parziale — manca `showOpenFilePicker`, il database condiviso non si collega |
| Safari | Parziale |
| Mobile | Non testato |

---

## Limiti noti

- **Nessuna autenticazione.** Chi apre il file ha accesso a tutto. È pensato per uso locale o aziendale protetto.
- **Nessun backup automatico.** Consigliato copiare periodicamente il JSON.
- **Un utente alla volta in scrittura.** Il lock impedisce la scrittura concorrente, non la lettura.

---

## Licenza

MIT — vedi [LICENSE](LICENSE).
