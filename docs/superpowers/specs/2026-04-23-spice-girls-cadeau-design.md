# Spice Girls Cadeau — Design

## Context

Bruidspaar (anti-Spice Girls) krijgt op hun bruiloft een QR-code op een cadeaukaart. Scannen brengt ze naar een pagina die ze dwingt "Stop Right Now" van de Spice Girls in zijn geheel uit te zitten voordat de echte cadeauboodschap (geld, af te wikkelen via Eefje) verschijnt.

## Scope

Eén statische webpagina, gehost op GitHub Pages onder `daanvanboerdonk/kadobramsofie`. Eindresultaat-URL: `https://daanvanboerdonk.github.io/kadobramsofie/`.

## Gebruikersflow

1. Gast scant QR → landt op de pagina
2. **Startscherm:** knop "🎁 Klik hier voor jullie cadeau" op bruiloft-achtige achtergrond
3. **Tap op de knop** → startscherm verdwijnt, video begint (met geluid, fullscreen binnen viewport)
4. **Videoscherm:** YouTube iframe met "Stop Right Now", geen controls, niet te pauzeren of skippen
5. **Video eindigt** → cadeau-scherm verschijnt met confetti-animatie

## Architectuur

Eén `index.html` met inline CSS en JS. Geen build-stap, geen framework, geen npm. Drie states worden door JavaScript gewisseld door `hidden`-classes te togglen op drie `<section>`-elementen:

- `#start` — startscherm met knop
- `#video` — YouTube iframe + klik-blokkerende overlay
- `#gift` — cadeautekst + confetti

## Technische details

### YouTube integratie
- YouTube IFrame Player API geladen via `<script src="https://www.youtube.com/iframe_api">`
- Video-ID: `2ZF2alwwSEU` (officiële "Stop Right Now" clip). Bij check: als deze niet embeddable is, wijken we uit naar een andere officiële upload.
- Iframe-parameters: `controls=0`, `disablekb=1`, `modestbranding=1`, `rel=0`, `playsinline=1`, `fs=0`, `iv_load_policy=3`
- `onReady`: player klaar, maar pas `playVideo()` aanroepen op basis van user-tap (startknop-handler)
- `onStateChange`: als state `PAUSED` (2) → direct `playVideo()`. Als state `ENDED` (0) → wissel naar `#gift`.

### Anti-skip / anti-interactie
- Transparante `<div>` met `position: absolute; inset: 0; z-index: 10;` over de iframe — vangt alle klikken en touches af
- `disablekb=1` blokkeert YouTube's eigen keyboard shortcuts
- Paused → playing auto-resume via `onStateChange`
- **Expliciet niet afgedekt:** fysieke volumeknoppen op het apparaat. Accepteerbaar: ze kunnen muten maar moeten alsnog 3:23 wachten.

### Confetti
- Library `canvas-confetti` via CDN (één `<script>`-tag, ~5KB gzipped, geen dependencies)
- Afgevuurd zodra `#gift` zichtbaar wordt; één burst bij reveal, daarna een zachte herhaling elke paar seconden voor 10–15 seconden

### Stijl
- Bruiloft-achtige, onschuldige look op het startscherm zodat de prank niet weggeeft: pastel achtergrond, serif-koptekst, hart- of bloem-emoji
- Mobile-first: startknop groot en centraal, video vult de viewport, cadeautekst leesbaar op klein scherm
- Nederlandse tekst overal

### Cadeautekst
> **Gefeliciteerd! 🎉**
>
> Jullie hebben je doel bereikt en jullie hebben je cadeau verdiend.
>
> Stuur een foto van deze pagina naar Eefje, en zij gaat het regelen.

## Deploy

1. `git init` in `/Users/daan/VIBECODE/kadobramsofie` (al gedaan)
2. `index.html` committen
3. `gh repo create daanvanboerdonk/kadobramsofie --public --source=. --push`
4. GitHub Pages aanzetten op `main` branch, root directory, via `gh api` of de repo settings
5. Wachten op eerste Pages-build (~1 min), URL teruggeven

## Niet in scope

- Analytics / tracking
- Fallback als YouTube de video blokkeert (bij embeddability-probleem wisselen we handmatig naar andere video-ID)
- Meerdere talen
- Authenticatie / beveiliging (iedereen met de URL kan de pagina zien; dat is prima voor dit cadeau)
