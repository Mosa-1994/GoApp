# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Halte is a real-time public transport departure display for De Meern (bus) and Station Vleuten (train) in the Netherlands. It's a static web application with no build process and minimal external dependencies (CDN only).

**Live URL:** https://mosa-1994.github.io/GoApp/

## Development

**No build/test/lint commands** - This is a zero-dependency static HTML application.

To run locally: Open `index.html` directly in a browser or serve with any static file server.

## File Structure

```
├── index.html          # Main HTML structure + all JavaScript (~1060 lines)
├── styles.css          # All CSS styling with CSS custom properties (~540 lines)
├── assets/
│   ├── halte-icon.png  # App icon (PNG)
│   ├── halte-icon.svg  # App icon (SVG)
│   └── goapp-icoon.png # Legacy icon
├── CLAUDE.md           # This file (AI assistant guidance)
├── README.md           # User-facing documentation
└── ROADMAP.md          # Future feature ideas
```

## Architecture

**Single Page Application** using vanilla JavaScript (ES6+):

- Fetches real-time data from drgl.nl (which sources from NDOV/GTFS-RT)
- Three collapsible sections: Bus to Utrecht, Bus to Vleuten, Train from Vleuten
- Each section shows first departure prominently; click to expand for more
- Auto-refreshes every 15 seconds with Visibility API optimization
- Fallback CORS proxy system for reliability (allorigins, corsproxy.io)
- Live map modal showing real-time bus positions with route progress

### External Dependencies (CDN)

- **Leaflet 1.9.4** - Interactive maps for live position tracking
- **Manrope Font** - Google Fonts for typography
- **CartoDB Tiles** - Map tiles (light_all style)

### Data Sources

- Bus: `https://drgl.nl/stop/NL:S:51200123/departurespanel` (HTML)
- Train: `https://drgl.nl/stop/NL:S:vtn/traindeparturespanel` (HTML)
- Live position: `https://drgl.nl/journey/{id}/position` (GeoJSON)
- Route polyline: `https://drgl.nl/journey/{id}/polyline` (JSON)
- Journey stops: `https://drgl.nl/journey/{id}/` (HTML)
- Journey progress: `https://drgl.nl/journey/{id}/progress` (HTML)

### Key Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `BUS_STOP_AREA` | `NL:S:51200123` | De Meern, Veldhuizen bus stop |
| `TRAIN_STOP_AREA` | `NL:S:vtn` | Station Vleuten |
| `REFRESH_INTERVAL` | 15000ms | Main departure refresh interval |
| `POSITION_CACHE_TTL` | 30000ms | Live position cache duration |
| `JOURNEY_CACHE_TTL` | 30000ms | Journey stops/progress cache duration |
| `FETCH_TIMEOUT` | 5000ms | Per-proxy request timeout |
| Max bus departures | 5 per direction | Utrecht and Vleuten separately |
| Max train departures | 6 | All trains from Vleuten |
| Time window | -1 to +120 min | Past 1 min, future 2 hours |

### Data Flow

```
Browser → CORS Proxy → drgl.nl HTML → DOMParser → Extract departures → Render UI
                                                                     ↓
                                                              Live position fetch
                                                                     ↓
                                                              Leaflet map render
```

## Core Features

### HTML Parsing (drgl.nl)

| Selector | Data |
|----------|------|
| `.list-group-item` | Departure rows |
| `.ott-departure-time` | Time (includes delay like "21:10 +11") |
| `.ott-linecode` | Bus line number |
| `.ott-productcategory` | Train type (Sprinter, Intercity) |
| `.ott-destination` | Destination name |
| `.ott-trainplatform` | Platform number |
| `.ott-tripstatus-driving` | Realtime tracking active |
| `.ott-departure-delayed` | Has delay |
| `.ott-departure-cancelled` | Service cancelled |

### Delay Handling

- drgl.nl shows planned time + delay (e.g., "21:10 +11")
- App calculates actual expected time: planned + delay minutes
- Countdown shows minutes until actual arrival
- Status indicators: on-time (green), slight delay ≤5min (yellow), major delay >5min (red)

### Theme System

- Dark mode (default) and light mode
- Toggle via button in status bar
- Persisted in `localStorage` under key `theme`
- Updates `<meta name="theme-color">` for mobile browser chrome
- CSS custom properties in `:root` and `[data-theme="light"]`

### Live Map Feature

- Click any bus departure card with "Live" badge to open map modal
- Shows real-time vehicle position on Leaflet map
- Displays full route polyline (gray) and remaining route to stop (blue)
- Shows "remaining stops" badge when position is between stops
- Auto-refreshes every 10 seconds while modal is open
- Uses Google Polyline Algorithm for decoding route coordinates

### Caching System

Three separate caches to reduce API calls:

1. **positionCache** - Live position data (30s TTL)
2. **journeyStopsCache** - List of stops for a journey (30s TTL)
3. **journeyProgressCache** - Current position text "tussen X en Y" (30s TTL)

### PWA Features

- `apple-mobile-web-app-capable: yes` - Fullscreen on iOS
- `apple-mobile-web-app-status-bar-style: black` - Dark status bar
- `theme-color` meta tag - Browser chrome color
- App icons for home screen installation

## Key JavaScript Functions

### Data Fetching
- `fetchDrglData(url)` - Fetch HTML via CORS proxy (parallel racing)
- `fetchDrglJson(url)` - Fetch JSON via CORS proxy
- `fetchLivePositionStatus(journeyPath)` - Check if live position available
- `fetchJourneyStops(journeyPath)` - Get list of stops for journey
- `fetchJourneyProgress(journeyPath)` - Get current "between X and Y" text

### Parsing
- `parseDrglHtml(html)` - Parse bus departures from HTML
- `parseTrainHtml(html)` - Parse train departures from HTML
- `getDirectionFromDestination(dest)` - Classify as 'utrecht' or 'vleuten'
- `decodePolyline(encoded)` - Decode Google polyline format

### Rendering
- `loadDepartures()` - Main refresh function
- `renderDepartures(utrecht, vleuten, trains)` - Render all sections
- `renderSection(id, title, departures, type)` - Render collapsible section
- `renderDepartureCard(departure)` - Render single bus card
- `renderTrainCard(departure)` - Render single train card

### Live Map
- `openLiveMap(journeyPath, line, dest, isRealtime)` - Open map modal
- `closeMapModal()` - Close and cleanup map
- `refreshLiveMap()` - Update vehicle position
- `loadJourneyRoute()` - Load and display route polyline
- `updateRemainingRoute(lat, lon)` - Highlight remaining route to stop
- `getRemainingStopsCount(journeyPath, targetStopId)` - Calculate stops remaining

### Utilities
- `formatTime(date)` - Format as HH:MM
- `getTimeClass(minutes)` - Return 'now', 'soon', or ''
- `getStatusInfo(departure)` - Get status class and label
- `toggleTheme()` - Switch dark/light mode
- `handleRefreshClick()` - Manual refresh with spinner animation

## CSS Architecture

- CSS custom properties for theming (colors, shadows)
- Mobile-first responsive design (max-width: 420px container)
- Card-based UI with subtle borders and shadows
- Accent colors: `--accent-bus` (pink #fb7185), `--accent-train` (beige #e9b882)
- Status colors: `--success` (green), `--warning` (yellow), `--danger` (red)
- Map canvas filter inverts colors for dark mode

## Conventions

- Dutch language for UI text and comments
- All JavaScript in single `<script>` tag in index.html
- No module system or bundling
- Inline SVG icons (no icon library)
- CSS class naming: BEM-like but simplified
- Error handling: graceful degradation with user-friendly messages
