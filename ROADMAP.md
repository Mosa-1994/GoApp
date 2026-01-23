# Roadmap OV Tijden

Doel: van single-page vertrektijden naar een robuuste, configureerbare en offline-vriendelijke PWA.

## Fase 1: Betrouwbaarheid en basisconfig (1-2 dagen)
- Voeg gebruikersinstellingen toe voor haltes/stations (bus + trein), met defaults als fallback.
- Sla voorkeuren en thema op in localStorage (haltes, filters, open/gesloten secties).
- Splits statusmeldingen per bron (bus/trein) met duidelijke foutstates.
- Introduceer eenvoudige retry/backoff bij fetch-fouten en toon laatste succesvolle update per sectie.
- Maak onderscheid tussen "geen ritten" en "geen data" in de UI.

## Fase 2: PWA-afwerking en offline (1-2 dagen)
- Voeg `manifest.webmanifest` toe met iconen, naam, theme-color en start_url.
- Bouw een service worker met cache-first voor assets en stale-while-revalidate voor data.
- Toon update-notificatie wanneer nieuwe versie van de app beschikbaar is.
- Voeg duidelijke offline-indicator toe en toon laatst bekende data als fallback.

## Fase 3: Reisinformatie en filters (2-3 dagen)
- Filters op lijn, richting en type (bus/trein) met snelle toggles.
- Toegang tot verstoringen/rituitval (als data beschikbaar is) met prominente banner.
- UI voor sortering (vertrek, vertraging, bestemming) en “toon meer” per sectie.
- Kleine “trend” indicator bij vertraging (neemt toe/af) indien informatie aanwezig.

## Fase 4: Notificaties en automatisering (2-4 dagen)
- Lokale meldingen voor gekozen lijnen of bestemmingen ("vertrekt over 5 min").
- Tijdschema voor stilte-uren (geen meldingen s avonds/nachts).
- Optionele achtergrond-refresh (wanneer toegestaan) om meldingen actueel te houden.

## Fase 5: Kwaliteit en toegankelijkheid (1-2 dagen)
- ARIA-labels, focus states, toetsenbordnavigatie en live-region voor updates.
- Verbeterde testbaarheid: parser unit tests met vaste HTML fixtures.
- Telemetrie (optioneel) met foutlogging, zonder tracking van gebruikersdata.

## Uit scope (voor nu)
- Account login of synchronisatie tussen apparaten.
- Routeplanning of volledige reisadviezen.
- Volledige multi-stad ondersteuning.

## Acceptatiecriteria (MVP+)
- App werkt offline met laatste bekende data.
- Haltes/stations zijn instelbaar en persistente voorkeuren werken.
- Foutstates zijn per bron zichtbaar met duidelijke labels.
- PWA kan op iOS/Android toegevoegd worden met juiste iconen en splash.
