# Riepilogo Commesse

Applicazione web a **singolo file HTML** per il monitoraggio delle commesse industriali: quadro sinottico dei ritardi, avanzamento attività da file MS Project, scadenziario, diario e archivio documenti.

Nata da un'esigenza concreta: gestire decine di commesse in produzione senza aprire un foglio Excel per ogni controllo.

## Cosa fa

**Quadro sinottico delle commesse attive**

Una sola tabella con, per ogni commessa: date di apertura e consegna da ordine, attività in ritardo, ritardi su emissione ordini e arrivo materiali, giorni di ritardo sull'inizio fabbricazione, scostamento dell'ultimo WP rispetto alla consegna contrattuale. Le celle critiche si colorano da sole.

**Import da MS Project**

Carichi il file XML esportato da MS Project e l'applicazione:

- ricostruisce l'albero delle attività rispettando i livelli di outline
- calcola i ritardi confrontando la data di fine prevista con oggi, solo sulle attività non chiuse al 100%
- conta separatamente i ritardi dentro i blocchi `PO Issuance` (MR) e i `Material Deliverables` scaduti
- rileva la data di inizio fabbricazione cercando le attività `MANUFACTURING` / `FABRICATION`
- individua la consegna prevista dall'ultimo WP tramite le attività `DELIVERY`

**Diagramma di Gantt**

Generato in SVG dalle stesse attività dell'XML, con intestazione a mesi e settimane. Nessuna libreria esterna.

**Scadenziario manuale**

Attività che non stanno dentro MS Project (solleciti, invii, controlli) con stato, data fine e giorni di ritardo calcolati.

**Diario e problematiche**

Annotazioni e allerte critiche per commessa, con storico consultabile.

**Documenti PDF**

Archivio documenti per commessa, con descrizione, nome file originale e data di caricamento.

**Archivio commesse concluse**

Le commesse chiuse escono dalla home ma restano consultabili e ripristinabili.

**Stampa**

Report stampabile di tutte le commesse o della singola commessa, impaginato in orizzontale su A4.

## Provala subito

Nella cartella [`demo/`](demo/) c'è `database_commesse.json`: un database dimostrativo con **dati completamente inventati** (clienti "Cliente A/B", officine "Officina 1/2").

Contiene due commesse:

- **70777** — in ritardo: 1 MR scaduto, 1 ordine scaduto, 2 materiali scaduti, inizio fabbricazione +11 giorni, ultimo WP a +36 giorni dalla consegna ordine. Due avanzamenti e un'allerta nel diario.
- **70812** — in linea: tutti i contatori a zero.

Più quattro scadenze manuali e tre voci di diario.

Per provarla: apri `Commesse.html`, seleziona `demo/database_commesse.json` quando richiesto, e clicca sul numero blu `70777` nella Home.

**Attenzione:** se salvi mentre è aperto il database demo, sovrascrivi quel file. Tienilo in una cartella separata dai dati reali.

## Come si usa

1. Scarica `Commesse.html`
2. Aprilo con **Microsoft Edge** o **Google Chrome** aggiornato (serve il supporto a `File System Access API`)
3. Al primo avvio ti chiede di selezionare il file `database_commesse.json`
4. Se non ne hai uno, l'applicazione ne crea uno vuoto al primo salvataggio

Non c'è installazione, non c'è server, non c'è account. Il file HTML è l'applicazione.

## Dove stanno i dati

Tutto in locale. Nessun dato lascia il computer.

L'applicazione salva in due posti:

- **`database_commesse.json`** — il database vero e proprio, nella cartella che scegli tu. Puoi metterlo su un disco di rete per condividerlo tra più postazioni.
- **`localStorage` del browser** — una copia di lavoro, usata per riaprire l'applicazione con i dati già caricati.

Il file JSON **non è mai dentro il repository**: è escluso da `.gitignore`. Il codice è pubblico, i dati no.

## Limiti noti

- **Un solo utente alla volta.** Non c'è locking: se due persone scrivono lo stesso JSON contemporaneamente, vince l'ultima che salva. Va bene per uso personale, non per un team.
- **I PDF sono codificati in base64 dentro il JSON.** `localStorage` ha un limite di circa 5 MB per dominio: superato quello, il salvataggio in cache fallisce silenziosamente. Il database su disco non ne risente, ma è il primo punto da rifare.
- **Nessuna autenticazione.** Chi apre il file ha accesso a tutto. È pensato per uso locale.
- **Nessun backup automatico.** Consigliato copiare periodicamente il JSON.

## Requisiti tecnici

Nessuna dipendenza. Zero build step. Solo HTML, CSS e JavaScript vanilla in un unico file.

## Compatibilità

| Browser | Supporto |
|---|---|
| Chrome / Edge (desktop) | Completo |
| Firefox | Parziale — manca `showOpenFilePicker`, il database condiviso non si collega |
| Safari | Parziale |
| Mobile | Non testato |

## Licenza

MIT — vedi [LICENSE](LICENSE).

------------------------------------------------------------------
📈 GANTT TAB (MS Project style)
------------------------------------------------------------------
From the top tab bar choose "📈 Gantt (MS Project style)".

What you can do:
  • Switch order from the dropdown at the top.
  • Time scale: Days / Weeks / Months, with + and − zoom.
  • Red vertical line = TODAY, always visible.
  • Dependencies: arrows linking the finish of one activity to the
    start of the next ("dependencies" field in the database, or
    imported automatically from an MS Project XML file - PredecessorLink).
  • Critical path: activities that would delay the final delivery are
    highlighted in red (CPM algorithm).
  • Milestones: ◆ diamonds for zero-duration activities.
  • Working days: enable the option to exclude Saturday and Sunday
    from duration calculations.
  • Quick filters: All / Critical only / Late / Due soon.
  • Direct editing:
      - drag the bar to move dates
      - drag the bar edges to resize
      - double-click the bar to edit name, dates, % and predecessors
      - "＋ New activity" button to add a phase
  • Export PNG (image to paste into an email or report).
  • Print A4 landscape.

Changes made in the Gantt are saved to the shared database, with the
multi-user lock already in place.
