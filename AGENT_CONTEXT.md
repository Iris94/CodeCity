# Project Overview & Architecture: codeCityPortfolio

Ovaj dokument služi kao referenca za sve AI agente i developere koji nastavljaju rad na projektu. Sadrži sažetak trenutne arhitekture, dizajnerskih odluka i kreiranih komponenti.

## Cilj Projekta
Refaktorizacija originalnog Svelte-based "Retrowave" portfolija u visoko-performantnu **Astro + Tailwind CSS** aplikaciju. Glavni fokus je na modularnosti, performansama (nema teških JS biblioteka za animacije tamo gdje nisu potrebne), cross-browser kompatibilnosti i SEO-u. Zadržan je originalni *Outrun / Cyberpunk / Synthwave* vizuelni identitet.

## Tehnološki Stack
- **Framework:** Astro (vanilla komponente)
- **Styling:** Tailwind CSS + Vanilla CSS (za složene layoute, custom scrollbare i 3D efekte)
- **Ikone:** `lucide-astro` (SVG komponente, bez runtime overhead-a)
- **Forme:** Web3Forms (client-side fetch sa bot protection-om)
- **Skripte:** Vanilla JavaScript (koristeći browser API-je poput `IntersectionObserver` i `EventSource`)

## Styling i Tema
Dizajnerski sistem je baziran na custom varijablama definiranim u `src/styles/theme.css`:
- **Boje:** Definisane su specifične cyberpunk boje (npr. `--theme-neon-cyan`, `--theme-neon-red`, `--theme-purple-shuttle`, `--theme-midnight-black`). Tailwind klase često koriste ove varijable (npr. `bg-[var(--theme-midnight-black)]`).
- **Global Styles:** `src/styles/base.css` i `src/styles/global.css` drže bazne stilove, fontove (`Space Grotesk`, monospace) i globalne utility klase (npr. custom scrollbari `::-webkit-scrollbar`).
- **Vizuelni Motivi:** 
  - Bez standardnih `border-radius` zaobljenja na bitnim komponentama (koriste se oštre ivice ili specifični cut-out oblici ako je potrebno).
  - CSS Scanline efekti preko ekrana komponenti.
  - Oštar kontrast i neonski akcenti.

## Kreirane Komponente (`src/components/`)

### Terminal / Hardware Komponente
Sve hardverske komponente primaju `rotate={true}` prop koji aktivira 3D perspective tilt efekat na mouse hover. Sadržaj se u komponente ubacuje preko `<slot />`.

1. **`CompactTerminal.astro` (ex-Datapad)**
   - Vertikalni uređaj (podsjeća na datapad/tablet).
   - Ima gornji hardverski bezel (kamera) i donji (home indikator).
   - Unutrašnji ekran ima scanlines i gornji status bar (`sysId` prop).

2. **`WantedTerminal.astro`**
   - Industrijski, robusni mainframe panel.
   - **Lijeva sekcija (20%):** Tamni metal sa zakovicama (rivets). Sadrži vertikalno ispisana slova "POLICE" (konfigurabilno preko `label` prop-a) sa podnaslovom "wanted list".
   - **Desna sekcija (80%):** Ekran sličan CompactTerminalu sa status barom (`front-end />`).
   - Koristi se za Persona karticu (Dossier sekcija).

3. **`CrtTerminal.astro` (ex-Laptop)**
   - Horizontalni CRT ekran / simulacija laptopa.
   - Sadrži donju bazu sa simuliranim trackpad-om.
   - Podržava `maxWidth` prop za kontrolu širine.

### Funkcionalne & UI Komponente

4. **`Navigation.astro`**
   - Futuristic fixed panel navigacija na vrhu ekrana (`fixed`, `z-50`).
   - Ima ugrađen status bar ("NAV_SYS v2.4.1").
   - **Synth Radio Integracija:** 
     - Sadrži dugme sa `lucide-astro` ikonama (Play/Pause).
     - Pušta muziku sa Nightride.fm API-ja (`https://stream.nightride.fm/nightride.mp3`).
     - Koristi SSE (Server-Sent Events) sa `https://nightride.fm/meta` za dohvatanje i prikaz naziva trenutne numere.
   - Sadrži linkove ka stranicama: `[Cases]` i `[Blog]`.

5. **`GlitchText.astro`**
   - Prima tekst (`text` prop) i ispisuje ga uz "decoder" cyberpunk animaciju (scrambled karakteri koji se pretvaraju u pravi tekst).
   - Animacija se okida tek kada element postane vidljiv na ekranu (koristi `IntersectionObserver`).
   - Prilagodljiv tag (`tag="h1"`, `"h2"`, itd.) i klasa.

6. **`HexRain.astro`**
   - Lagani `<canvas>` overlay koji simulira "Matrix padajuća slova", ali koristi random hex kodove.
   - Boja je `--theme-neon-red`.
   - Element je fiksiran, prekriva cijeli viewport, ima `z-index: 1` i ne blokira klikove (`pointer-events-none`). Uključen je u `Layout.astro` tako da je vidljiv na svim stranicama.

## Struktura Stranice (`src/pages/index.astro`)
Landing stranica je podijeljena u jasne sadržajne blokove, svaki sa `border-t` u neon crvenoj boji i opisnom etiketom iznad:

- **Sekcija 1: Landing Video** - Autoplay pozadinski video sa velikim "front-end umbraco game-dev ai research" naslovima raspoređenim dijagonalno.
- **Sekcija 2: Persona + Dossier** - Koristi `WantedTerminal` na lijevoj strani za prikaz "profila" i `GlitchText` za naslov "Dossier" sa propratnim tekstom na desnoj strani.
- **Sekcija 3: Terminal / Contact** - Koristi `CrtTerminal`. Na scroll aktivira "boot" sekvencu (typing efekat "Establishing connection..."). Nakon kratke pauze prikazuje se Web3Forms forma za slanje maila.

## Napomene za dalji razvoj
- **Web3Forms:** Kontakt forma šalje podatke klijentski. Implementiran je honeypot (`botcheck`) polje. Stanje submit buttona se dinamički mijenja (TRANSMITTING... -> TRANSMISSION SUCCESS).
- **Z-Index strategija:** 
  - `HexRain` je z-index 1. 
  - Content sekcije (videi, terminali) su z-index 2 (i veći za komponente unutar njih).
  - `Navigation` je z-index 50.
- **Optimizacija:** Sve scroll i viewport animacije OBAVEZNO treba da koriste `IntersectionObserver` umjesto konstantnog vezivanja na `window.onscroll`, kako bi se sačuvale performanse browsera.
