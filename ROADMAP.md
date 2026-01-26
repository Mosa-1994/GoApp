# Roadmap: Toekomstige Functionaliteiten

Dit document beschrijft uitbreidingen voor de Halte-app. Alle functionaliteiten zijn **gratis te implementeren** met open APIs, browser APIs of lokale opslag.

---

## 1. Offline Modus (Service Worker)

**Prioriteit:** Hoog
**Complexiteit:** Gemiddeld
**Technologie:** Service Worker API, Cache API

### Beschrijving
Maak de app bruikbaar zonder internetverbinding door recent opgehaalde data te cachen.

### Implementatie
1. Registreer een Service Worker (`sw.js`)
2. Cache statische bestanden (HTML, CSS, iconen) bij installatie
3. Implementeer "stale-while-revalidate" strategie voor API responses
4. Toon laatst bekende vertrektijden met timestamp wanneer offline
5. Voeg offline indicator toe aan status bar

### Technische Details
```javascript
// sw.js registratie in index.html
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}

// Cache strategie
const CACHE_NAME = 'halte-v1';
const OFFLINE_DATA_KEY = 'halte-offline-departures';
```

### Gratis Resources
- Browser native API (geen externe service nodig)
- Workbox library (optioneel, open source)

---

## 2. Favoriete Haltes

**Prioriteit:** Hoog
**Complexiteit:** Gemiddeld
**Technologie:** localStorage, drgl.nl API

### Beschrijving
Laat gebruikers meerdere haltes toevoegen en snel wisselen tussen locaties.

### Implementatie
1. Zoekfunctie voor haltes via drgl.nl (`/stops/search?q=...`)
2. Opslaan van favorieten in localStorage
3. Swipe of tab-navigatie tussen haltes
4. Per halte onthouden welke richtingen relevant zijn
5. "Huidige locatie" optie met Geolocation API

### Data Model
```javascript
// localStorage: 'halte-favorites'
[
  {
    id: 'NL:S:51200123',
    name: 'De Meern, Veldhuizen',
    type: 'bus',
    directions: ['utrecht', 'vleuten']
  },
  {
    id: 'NL:S:vtn',
    name: 'Station Vleuten',
    type: 'train',
    directions: ['all']
  }
]
```

### UI Aanpassingen
- Hamburger menu of bottom sheet voor halte-selectie
- "+" knop om nieuwe halte toe te voegen
- Swipe gestures voor navigatie
- Lange druk om halte te verwijderen

### Gratis Resources
- localStorage (browser native)
- Geolocation API (browser native)
- drgl.nl zoek-endpoint (gratis)

---

## 3. Push Notificaties

**Prioriteit:** Gemiddeld
**Complexiteit:** Hoog
**Technologie:** Web Push API, Notification API

### Beschrijving
Stuur meldingen bij vertraging, uitval of wanneer de bus/trein bijna vertrekt.

### Implementatie
1. Vraag notificatie-permissie aan gebruiker
2. Registreer Push subscription via Service Worker
3. Lokale notificaties (geen server nodig voor basis):
   - Timer-based: "Bus 28 vertrekt over 5 minuten"
   - Bij detectie van nieuwe vertraging
4. Configureerbare triggers per lijn/richting

### Notificatie Types
| Type | Trigger | Voorbeeld |
|------|---------|-----------|
| Vertrek reminder | X minuten voor vertrek | "Bus 28 naar Utrecht vertrekt over 5 min" |
| Vertraging alert | Nieuwe vertraging gedetecteerd | "Bus 28 heeft +8 min vertraging" |
| Uitval melding | Rit vervalt | "Sprinter naar Utrecht Centraal rijdt niet" |

### Technische Details
```javascript
// Permissie aanvragen
Notification.requestPermission().then(permission => {
  if (permission === 'granted') {
    // Notificaties inschakelen
  }
});

// Lokale notificatie
new Notification('Halte', {
  body: 'Bus 28 naar Utrecht vertrekt over 5 minuten',
  icon: '/assets/halte-icon.png',
  tag: 'departure-reminder'
});
```

### Gratis Resources
- Notification API (browser native, geen server nodig voor lokale notificaties)
- Service Worker voor achtergrond-checking

---

## 4. Verstoringen & Werkzaamheden

**Prioriteit:** Hoog
**Complexiteit:** Gemiddeld
**Technologie:** NDOV/OVapi, NS API (gratis tier)

### Beschrijving
Toon actuele verstoringen, omleidingen en geplande werkzaamheden.

### Data Bronnen (Gratis)
1. **drgl.nl alerts** - Check of er alerts zijn in de HTML response
2. **OVapi.nl** - Open data, gratis te gebruiken
3. **NS Reisinformatie API** - Gratis voor niet-commercieel gebruik (50 req/min)

### Implementatie
1. Haal verstoringsdata op bij elke refresh
2. Toon banner bovenaan bij actieve verstoring
3. Filter op relevante lijnen/trajecten
4. Icoon bij getroffen vertrektijden
5. Modal met details bij klik op banner

### UI Design
```
┌─────────────────────────────────────┐
│ ⚠️ Verstoring bus 28: omleiding     │
│    via Meernbrug                    │
└─────────────────────────────────────┘
```

### API Endpoints
```
# OVapi (gratis, geen API key)
https://v0.ovapi.nl/line/

# NS API (gratis registratie vereist)
https://gateway.apiportal.ns.nl/reisinformatie-api/api/v3/disruptions
```

### Gratis Resources
- OVapi.nl: volledig open, geen registratie
- NS API: gratis tier met 50 requests/minuut

---

## 5. Looptijd naar Halte

**Prioriteit:** Gemiddeld
**Complexiteit:** Laag
**Technologie:** Geolocation API, Haversine formule

### Beschrijving
Bereken hoeveel minuten lopen het is naar de halte en markeer haalbare vertrektijden.

### Implementatie
1. Vraag locatie-permissie (eenmalig of continu)
2. Bereken afstand met Haversine formule (geen externe API nodig)
3. Schat looptijd: ~80 meter/minuut (5 km/u)
4. Markeer vertrektijden als "haalbaar" of "niet haalbaar"
5. Optioneel: integreer met route-API voor nauwkeuriger schatting

### Technische Details
```javascript
// Haversine formule voor afstand
function getDistanceKm(lat1, lon1, lat2, lon2) {
  const R = 6371; // Radius aarde in km
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLon = (lon2 - lon1) * Math.PI / 180;
  const a = Math.sin(dLat/2) ** 2 +
            Math.cos(lat1 * Math.PI / 180) *
            Math.cos(lat2 * Math.PI / 180) *
            Math.sin(dLon/2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}

// Looptijd in minuten
const walkingMinutes = Math.ceil(distanceKm / 0.08); // 80m/min
```

### UI Aanpassingen
- Badge "🚶 5 min" naast haltesnaam
- Groene/rode indicator bij vertrektijden
- Instellingen voor loopsnelheid

### Gratis Resources
- Geolocation API (browser native)
- Wiskunde (Haversine) - geen API nodig
- Optioneel: OSRM (open source routing, self-hosted of gratis instances)

---

## 6. Weer Integratie

**Prioriteit:** Laag
**Complexiteit:** Laag
**Technologie:** Open-Meteo API (volledig gratis)

### Beschrijving
Toon actueel weer en korte voorspelling voor het plannen van de reis.

### Implementatie
1. Haal weerdata op van Open-Meteo (geen API key nodig!)
2. Toon in header: temperatuur + icoon
3. Waarschuwing bij extreem weer (storm, ijzel)
4. Cache weerdata voor 30 minuten

### API Endpoint
```
https://api.open-meteo.com/v1/forecast
  ?latitude=52.08
  &longitude=5.01
  &current_weather=true
  &hourly=precipitation_probability
```

### UI Design
```
┌──────────────────────────────┐
│  HALTE            ☀️ 18°    │
└──────────────────────────────┘
```

### Gratis Resources
- **Open-Meteo**: 100% gratis, geen API key, geen limiet voor persoonlijk gebruik
- Alternatief: Buienradar (NL-specifiek, gratis API beschikbaar)

---

## 7. Widget / Snelle Toegang

**Prioriteit:** Gemiddeld
**Complexiteit:** Gemiddeld
**Technologie:** Web App Manifest, Shortcuts API

### Beschrijving
Voeg snelkoppelingen toe voor directe toegang tot specifieke haltes/richtingen.

### Implementatie
1. Maak `manifest.webmanifest` met shortcuts
2. Dynamische shortcuts per favoriete halte
3. URL parameters voor directe navigatie (`?stop=NL:S:vtn`)
4. iOS: Siri Shortcuts integratie via Web Clips

### Manifest Voorbeeld
```json
{
  "name": "Halte",
  "short_name": "Halte",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#0a0a0b",
  "background_color": "#0a0a0b",
  "icons": [
    { "src": "/assets/halte-icon.png", "sizes": "192x192", "type": "image/png" }
  ],
  "shortcuts": [
    {
      "name": "Bus naar Utrecht",
      "short_name": "Utrecht",
      "url": "/?stop=NL:S:51200123&direction=utrecht",
      "icons": [{ "src": "/assets/bus-icon.png", "sizes": "96x96" }]
    },
    {
      "name": "Trein Vleuten",
      "short_name": "Trein",
      "url": "/?stop=NL:S:vtn",
      "icons": [{ "src": "/assets/train-icon.png", "sizes": "96x96" }]
    }
  ]
}
```

### Gratis Resources
- Web App Manifest (browser native)
- PWA Shortcuts API (Chrome, Edge)

---

## 8. Reis Plannen Link

**Prioriteit:** Laag
**Complexiteit:** Zeer Laag
**Technologie:** Deep links

### Beschrijving
Voeg knop toe om direct door te linken naar 9292.nl of NS app voor complete reisplanning.

### Implementatie
1. "Plan reis" knop bij elke sectie
2. Genereer deep link met vooringevulde vertrekhalte
3. Detecteer geïnstalleerde apps (NS, 9292)

### Deep Link Formaten
```javascript
// 9292.nl
`https://9292.nl/reisadvies/station-vleuten/amsterdam-centraal`

// NS (universal link)
`https://www.ns.nl/reisplanner#/?vertrek=Vleuten&aankomst=Utrecht%20Centraal`

// Google Maps
`https://www.google.com/maps/dir/?api=1&origin=Station+Vleuten&destination=Utrecht+Centraal&travelmode=transit`
```

### Gratis Resources
- Alle deep links zijn gratis te gebruiken

---

## 9. Toegankelijkheid Verbeteringen

**Prioriteit:** Hoog
**Complexiteit:** Gemiddeld
**Technologie:** ARIA, Semantic HTML

### Beschrijving
Verbeter toegankelijkheid voor screenreaders en gebruikers met beperkingen.

### Implementatie
1. **ARIA labels** voor alle interactieve elementen
2. **Focus management** bij modal openen/sluiten
3. **Reduced motion** media query respecteren
4. **High contrast** modus ondersteuning
5. **Keyboard navigatie** volledig werkend
6. **Screen reader announcements** bij data refresh

### Technische Details
```css
/* Respecteer reduced motion voorkeur */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* High contrast modus */
@media (prefers-contrast: high) {
  :root {
    --border: #ffffff;
    --text-secondary: #ffffff;
  }
}
```

```html
<!-- Live region voor updates -->
<div aria-live="polite" aria-atomic="true" class="sr-only" id="announcer">
  <!-- "Vertrektijden bijgewerkt om 14:32" -->
</div>
```

### Gratis Resources
- Alle browser APIs (native)
- axe DevTools (gratis Chrome extensie voor testen)

---

## 10. Statistieken & Geschiedenis

**Prioriteit:** Laag
**Complexiteit:** Gemiddeld
**Technologie:** IndexedDB, localStorage

### Beschrijving
Houd lokaal bij hoe betrouwbaar lijnen zijn en toon persoonlijke statistieken.

### Implementatie
1. Log elke observed vertraging in IndexedDB
2. Bereken gemiddelde vertraging per lijn/tijdstip
3. Toon "betrouwbaarheids-score" bij lijnen
4. Weekoverzicht van reispatroon
5. Data export functie (JSON/CSV)

### Data Model
```javascript
// IndexedDB schema
{
  store: 'observations',
  keyPath: 'id', // auto-increment
  indexes: ['line', 'timestamp', 'delay']
}

// Observation record
{
  id: 1,
  line: '28',
  destination: 'Utrecht CS',
  scheduledTime: '2024-01-15T08:15:00',
  actualDelay: 3, // minuten
  timestamp: 1705305300000
}
```

### UI Design
```
┌─────────────────────────────────────┐
│ Bus 28 naar Utrecht                 │
│ ████████████░░░░ 78% op tijd       │
│ Gem. vertraging: +2 min             │
└─────────────────────────────────────┘
```

### Gratis Resources
- IndexedDB (browser native)
- Geen server nodig - alle data lokaal

---

## 11. Geluidsmeldingen

**Prioriteit:** Laag
**Complexiteit:** Laag
**Technologie:** Web Audio API

### Beschrijving
Optionele geluidssignalen bij belangrijke events.

### Implementatie
1. Subtiel geluid bij "Nu" status
2. Waarschuwingsgeluid bij vertraging >5 min
3. Instelbaar volume en aan/uit per type
4. Respecteer "Do Not Disturb" modus

### Technische Details
```javascript
// Genereer toon met Web Audio API (geen audio files nodig)
function playTone(frequency, duration) {
  const ctx = new AudioContext();
  const oscillator = ctx.createOscillator();
  const gain = ctx.createGain();

  oscillator.connect(gain);
  gain.connect(ctx.destination);

  oscillator.frequency.value = frequency;
  gain.gain.value = 0.1;

  oscillator.start();
  oscillator.stop(ctx.currentTime + duration);
}

// Zachte "ping" voor vertrek
playTone(880, 0.1); // A5, 100ms
```

### Gratis Resources
- Web Audio API (browser native)
- Geen audio bestanden nodig

---

## 12. Auto-thema op Basis van Tijd

**Prioriteit:** Laag
**Complexiteit:** Zeer Laag
**Technologie:** JavaScript Date, CSS

### Beschrijving
Automatisch wisselen tussen light/dark mode op basis van tijd of systeem.

### Implementatie
1. Optie: "Systeem" (volg OS voorkeur)
2. Optie: "Automatisch" (light 7:00-19:00)
3. Optie: "Altijd donker" / "Altijd licht"

### Technische Details
```javascript
// Volg systeem voorkeur
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');
prefersDark.addEventListener('change', e => {
  setTheme(e.matches ? 'dark' : 'light');
});

// Tijd-gebaseerd
function getAutoTheme() {
  const hour = new Date().getHours();
  return (hour >= 7 && hour < 19) ? 'light' : 'dark';
}
```

### Gratis Resources
- `prefers-color-scheme` media query (browser native)

---

## 13. Deel Functie

**Prioriteit:** Laag
**Complexiteit:** Zeer Laag
**Technologie:** Web Share API

### Beschrijving
Deel vertrektijden of app-link via native share sheet.

### Implementatie
```javascript
async function shareDeparture(departure) {
  const text = `Bus ${departure.line} naar ${departure.destination} vertrekt om ${departure.time}`;

  if (navigator.share) {
    await navigator.share({
      title: 'Halte - Vertrektijd',
      text: text,
      url: window.location.href
    });
  } else {
    // Fallback: copy to clipboard
    await navigator.clipboard.writeText(text);
  }
}
```

### Gratis Resources
- Web Share API (browser native)
- Clipboard API als fallback

---

## 14. Dichtsbijzijnde Halte Zoeken

**Prioriteit:** Gemiddeld
**Complexiteit:** Gemiddeld
**Technologie:** Geolocation API, drgl.nl

### Beschrijving
Vind automatisch de dichtstbijzijnde halte op basis van huidige locatie.

### Implementatie
1. Vraag locatie-permissie
2. Query drgl.nl voor haltes in de buurt
3. Sorteer op afstand
4. Toon top 5 dichtstbijzijnde haltes
5. Optie om halte toe te voegen aan favorieten

### API Endpoint
```
https://drgl.nl/stops/nearby?lat=52.08&lon=5.01&radius=1000
```

### Gratis Resources
- Geolocation API (browser native)
- drgl.nl nearby endpoint (gratis)

---

## Implementatie Volgorde (Aanbevolen)

### Fase 1: Basis Verbeteringen (1-2 weken)
| # | Feature | Inspanning |
|---|---------|------------|
| 1 | Offline Modus | Gemiddeld |
| 2 | Toegankelijkheid | Gemiddeld |
| 3 | Auto-thema | Klein |
| 4 | Manifest/PWA | Klein |

### Fase 2: Uitgebreide Functionaliteit (2-3 weken)
| # | Feature | Inspanning |
|---|---------|------------|
| 5 | Favoriete Haltes | Gemiddeld |
| 6 | Looptijd naar Halte | Klein |
| 7 | Verstoringen | Gemiddeld |
| 8 | Dichtsbijzijnde Halte | Gemiddeld |

### Fase 3: Extra Features (2-3 weken)
| # | Feature | Inspanning |
|---|---------|------------|
| 9 | Push Notificaties | Groot |
| 10 | Weer Integratie | Klein |
| 11 | Widget / Shortcuts | Gemiddeld |

### Fase 4: Nice-to-Have (1-2 weken)
| # | Feature | Inspanning |
|---|---------|------------|
| 12 | Statistieken | Gemiddeld |
| 13 | Geluidsmeldingen | Klein |
| 14 | Reis Plannen Link | Klein |
| 15 | Deel Functie | Klein |

---

## Technische Vereisten Samenvatting

| Feature | API Key Nodig? | Server Nodig? | Browser Support |
|---------|----------------|---------------|-----------------|
| Offline Modus | Nee | Nee | 97%+ |
| Favoriete Haltes | Nee | Nee | 99%+ |
| Push Notificaties | Nee* | Nee* | 95%+ |
| Verstoringen | Optioneel (NS) | Nee | 99%+ |
| Looptijd | Nee | Nee | 95%+ |
| Weer | Nee | Nee | 99%+ |
| Widgets | Nee | Nee | 85%+ |
| Toegankelijkheid | Nee | Nee | 99%+ |
| Statistieken | Nee | Nee | 97%+ |
| Geluid | Nee | Nee | 97%+ |
| Auto-thema | Nee | Nee | 95%+ |
| Delen | Nee | Nee | 90%+ |

*Lokale notificaties vereisen geen server. Voor push op achtergrond is een push server nodig (gratis via Firebase Cloud Messaging).

---

## Gratis API Services

| Service | Gebruik | Limiet | API Key |
|---------|---------|--------|---------|
| drgl.nl | OV data | Onbeperkt | Nee |
| OVapi.nl | Verstoringen, lijnen | Onbeperkt | Nee |
| Open-Meteo | Weer | Onbeperkt | Nee |
| NS API | Trein verstoringen | 50 req/min | Ja (gratis) |
| OpenStreetMap/Leaflet | Kaarten | Fair use | Nee |
| CartoDB Tiles | Kaarttegels | Fair use | Nee |
| OSRM | Routing | Self-host of demo | Nee |

---

## Acceptatiecriteria (MVP+)

- [ ] App werkt offline met laatst bekende data
- [ ] Haltes/stations zijn instelbaar via favorieten
- [ ] Foutstates zijn per bron zichtbaar met duidelijke labels
- [ ] PWA kan op iOS/Android toegevoegd worden met juiste iconen
- [ ] Verstoringen worden getoond indien beschikbaar
- [ ] Toegankelijkheid voldoet aan WCAG 2.1 AA
- [ ] Looptijd naar halte wordt berekend en getoond
