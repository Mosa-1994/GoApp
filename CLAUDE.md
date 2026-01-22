# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OV Tijden is a real-time public transport departure display for De Meern (bus) and Station Vleuten (train) in the Netherlands. It's a single-file static web application with no build process or dependencies.

## Development

**No build/test/lint commands** - This is a zero-dependency static HTML application.

To run locally: Open `GoApp/index.html` directly in a browser or serve with any static file server.

All modifications are made directly to `GoApp/index.html` which contains HTML, CSS, and JavaScript in a single file.

## Architecture

**Single Page Application** using vanilla JavaScript (ES6+):
- Fetches real-time data from drgl.nl (which sources from NDOV/GTFS-RT)
- Three collapsible sections: Bus to Utrecht, Bus to Vleuten, Train from Vleuten
- Each section shows first departure prominently; click to expand for more
- Auto-refreshes every 15 seconds with Visibility API optimization
- Fallback CORS proxy system for reliability (allorigins, corsproxy.io)

**Data Sources:**
- Bus: `https://drgl.nl/stop/NL:S:51200123/departurespanel` (HTML)
- Train: `https://drgl.nl/stop/NL:S:vtn/traindeparturespanel` (HTML)

**Key Constants:**
- Bus stop area: `NL:S:51200123` (De Meern, Veldhuizen)
- Train station: `NL:S:vtn` (Station Vleuten)
- Refresh interval: 15 seconds
- Time window: 120 minutes ahead, 1 minute behind
- Max departures: 5 per bus direction, 6 trains

**Data Flow:** Browser → CORS Proxy → drgl.nl HTML → DOMParser → Extract departures → Render UI

**HTML Parsing (drgl.nl):**
- Rows: `.list-group-item`
- Time: `.ott-departure-time` (includes delay like "21:10 +11")
- Bus line: `.ott-linecode`
- Train type: `.ott-productcategory`
- Destination: `.ott-destination`
- Platform: `.ott-trainplatform`
- Status classes: `.ott-tripstatus-driving` (realtime), `.ott-departure-delayed`

**Delay Handling:**
- drgl.nl shows planned time + delay (e.g., "21:10 +11")
- App calculates actual expected time: planned + delay minutes
- Countdown shows minutes until actual arrival
