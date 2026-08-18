# Deficit Tracker

## Cos'è
Web app (PWA) offline-first per tracciare deficit calorico giornaliero: pasti (colazione, spuntino mattina, pranzo, spuntino pomeriggio, cena, varie/extra) con kcal/proteine/carbo/grassi, acqua bevuta, database offline di alimenti comuni, peso + stima automatica massa grassa/magra, mappa prodotti personalizzata. Pensata per iPhone, aggiunta alla home screen.

## Architettura attuale
- **`index.html`** è l'app vera e propria: un unico file HTML che carica React/ReactDOM/Babel da `vendor/` (scaricati localmente, nessuna dipendenza da CDN esterne a runtime) e compila il JSX nel browser. Nessun build step richiesto: si apre/serve così com'è.
- **`vendor/`** — copie locali di React 18, ReactDOM 18 e Babel standalone (per il transform JSX in-browser).
- **`manifest.webmanifest`** + **`icons/`** — per "Aggiungi a Home" su iOS.
- **`sw.js`** — service worker che cachea l'app shell per l'uso offline.
- **Persistenza**: `localStorage` (chiave `tracker-data`), sincrona e locale al dispositivo — niente più `window.storage`.

## Storico: perché non è più un Artifact React
Il vecchio `deficit-tracker.jsx` (ancora presente per riferimento, non più usato) era pensato per girare come Artifact di Claude e usava `window.storage.get/set`, un'API disponibile **solo** dentro il runtime degli Artifact. Il salvataggio falliva sistematicamente ("Storage set failed: Unexpected response type") perché fuori da quel contesto l'API semplicemente non esiste — non era un bug risolvibile con retry. Il file è stato riscritto come `index.html` standalone con `localStorage`, che è sincrono e affidabile.

## Come usarla
Basta aprire `index.html` in un server statico qualsiasi (es. `npx serve .`) o ospitarla (GitHub Pages, Netlify, Cloudflare Pages...). Da Safari su iPhone, aprire l'URL e usare "Condividi → Aggiungi alla schermata Home" per installarla come app standalone con icona.

Nota: aprendo `index.html` direttamente come file locale (`file://`) l'app funziona (localStorage funziona anche su file://), ma il service worker per l'uso offline richiede http/https — per l'uso reale conviene ospitarla.

## Possibili sviluppi futuri
- Esportare/importare i dati (backup manuale, dato che `localStorage` non sincronizza tra dispositivi).
- Editor per rinominare/modificare i prodotti salvati.
