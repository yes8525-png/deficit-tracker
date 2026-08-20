# Deficit Tracker

## Cos'è
Web app (PWA) offline-first per tracciare deficit calorico giornaliero: pasti (colazione, spuntino mattina, pranzo, spuntino pomeriggio, cena, varie/extra) con kcal/proteine/carbo/grassi, acqua bevuta, database offline di alimenti comuni, peso + stima automatica massa grassa/magra, mappa prodotti personalizzata. Pensata per iPhone, aggiunta alla home screen.

## Architettura attuale
- **`index.html`** è l'app vera e propria: un unico file HTML che carica React/ReactDOM/Babel da `vendor/` (scaricati localmente, nessuna dipendenza da CDN esterne a runtime) e compila il JSX nel browser. Nessun build step richiesto: si apre/serve così com'è.
- **`vendor/`** — copie locali di React 18, ReactDOM 18 e Babel standalone (per il transform JSX in-browser).
- **`manifest.webmanifest`** + **`icons/`** — per "Aggiungi a Home" su iOS.
- **`sw.js`** — service worker che cachea l'app shell per l'uso offline.
- **Persistenza**: doppia.
  - `localStorage` (chiave `tracker-data`) — cache locale sincrona, usata per il primo render e come fallback offline.
  - **Supabase** (tabella `tracker_state`, riga fissa con id `28434d49-5166-4e1d-a293-b9ac185df4d6`, progetto `odnadjqepoeqnyldduod`) — fonte di verità. Ad ogni avvio l'app scarica questa riga e la usa al posto del locale; ad ogni salvataggio scrive sia in locale che su Supabase. Introdotta perché su iOS il solo `localStorage` (specie per PWA da schermata Home) può venire svuotato da Safari senza preavviso — con Supabase i dati sopravvivono a quello, e sono anche identici su qualunque dispositivo/browser si apra il link.
  - Il piccolo indicatore in alto a destra (● salvato online / salvataggio… / offline · solo locale) mostra lo stato della sincronizzazione col server.
  - Sicurezza: nessun vero login (app mono-utente). La client key di Supabase è pubblica per design (embeddata nel JS, repo pubblico); le RLS policy limitano le operazioni alla singola riga con quell'id fisso, ma chiunque trovasse quell'id nel sorgente potrebbe leggerlo/sovrascriverlo. Accettabile per dati personali di fitness non critici; se in futuro serve isolamento vero, va aggiunta autenticazione.

## Storico: perché non è più un Artifact React
Il vecchio `deficit-tracker.jsx` (ancora presente per riferimento, non più usato) era pensato per girare come Artifact di Claude e usava `window.storage.get/set`, un'API disponibile **solo** dentro il runtime degli Artifact. Il salvataggio falliva sistematicamente ("Storage set failed: Unexpected response type") perché fuori da quel contesto l'API semplicemente non esiste — non era un bug risolvibile con retry. Il file è stato riscritto come `index.html` standalone con `localStorage`, che è sincrono e affidabile.

## Come usarla
**È già online**: https://yes8525-png.github.io/deficit-tracker/ — pubblicata automaticamente da GitHub Actions (`.github/workflows/deploy-pages.yml`) ad ogni push su `master`. Da Safari su iPhone, aprire l'URL e usare "Condividi → Aggiungi alla schermata Home" per installarla come app standalone con icona.

In alternativa, per sviluppo locale: aprire `index.html` in un server statico qualsiasi (es. `npx serve .`).

Nota: aprendo `index.html` direttamente come file locale (`file://`) l'app funziona (localStorage funziona anche su file://), ma il service worker per l'uso offline richiede http/https — per questo conviene usare il link GitHub Pages.

Importante: i dati (`localStorage`) sono legati al dominio/origine visitata — un URL diverso (es. un'anteprima di test) parte sempre con dati vuoti e non condivide nulla con https://yes8525-png.github.io/deficit-tracker/. Usare la funzione di backup (tab Prodotti → "Esporta/Importa backup") per spostare i dati tra istanze diverse.

## Possibili sviluppi futuri
- ~~Esportare/importare i dati (backup manuale, dato che `localStorage` non sincronizza tra dispositivi).~~ Fatto: tab Prodotti → card "Backup dati" (esporta JSON, importa con conferma e validazione formato).
- ~~Editor per rinominare/modificare i prodotti salvati.~~ Fatto: icona matita su ogni prodotto della mappa, apre un form di modifica inline.
- ~~I dati sparivano su iOS (Safari/PWA svuotava `localStorage`).~~ Fatto: i dati ora vivono su Supabase, `localStorage` resta solo come cache/fallback offline.
- Se un giorno servisse davvero multi-utente o maggiore sicurezza: aggiungere autenticazione Supabase (es. magic link via email) invece della riga fissa condivisa dalla client key pubblica.
