# TALON EDDYLINE MVP
## Airfield Change Detection System

---

## 📍 What Is TALON EDDYLINE? (In Plain English)

TALON EDDYLINE is an intelligent monitoring system that watches airfields (airports) and detects when things change. Think of it like a security camera for runways, taxiways, and airfield infrastructure—but instead of just recording video, it automatically spots problems and alerts analysts.

### What Problems Does It Solve?

- **Safety Hazards**: Detects obstacles, debris, or damage that could endanger aircraft
- **Maintenance Monitoring**: Identifies pavement cracks, vegetation growth, or structural damage
- **Rapid Alerts**: Instead of waiting for routine inspections, changes are caught immediately
- **Multi-Airport Coverage**: Monitors multiple airports simultaneously and compares detections across locations

### How It Works (Simple Version)

1. **Camera/Satellite images** are collected from airfields
2. **AI models** analyze the images and spot changes (vegetation, cracks, obstructions, etc.)
3. **Analysts review** flagged items with priority ratings
4. **Feedback is logged** so the system learns and improves over time

### Key Features Users Will See

- **Interactive Map**: Click on airports to see detection summaries
- **Hover Panels**: Hover over an airport marker to see a quick breakdown of detection types and priorities
- **Detailed Modals**: Click a detection to see full analysis—images, coordinates, confidence scores, and feedback forms
- **Priority Levels**: P1 (most urgent) through P4 (routine) help analysts prioritize their work
- **Feedback Tracking**: Analysts can confirm valid detections or flag false positives, and their comments are recorded

---

## 🔧 Technical Documentation

### Architecture Overview

TALON EDDYLINE MVP is a single-page application (SPA) built with vanilla HTML, CSS, and JavaScript. It uses Leaflet.js for mapping and integrates data from public airport datasets.

### Technology Stack

- **Frontend Framework**: Vanilla JavaScript (no dependencies except Leaflet)
- **Mapping Library**: Leaflet.js 1.9.4 (with multiple CDN fallbacks for resilience)
- **Data Source**: OurAirports CSV (with graceful fallback to hardcoded coordinates)
- **Styling**: Custom CSS Grid + Flexbox
- **Data Format**: JSON (detection alerts compatible with AMC.Maps ingestion)

### Project Structure

```
index.html               # Single monolithic HTML file containing all logic
├── <style>             # Embedded CSS
│   ├── Header styling
│   ├── Map container
│   ├── Airport panel (hover state)
│   ├── Modal/popup styling
│   └── Responsive media queries
└── <script>            # Embedded JavaScript
    ├── Leaflet loading (multi-CDN fallback)
    ├── Airport data loading
    ├── Map initialization & markers
    ├── Modal management
    └── Feedback submission logic
```

### Key JavaScript Modules

#### 1. **Leaflet Loading** (`loadLeaflet()`)
- Implements resilient multi-CDN fallback strategy
- Tries: unpkg → jsdelivr → cdnjs
- Loading status updates displayed to user
- Throws clear error if all CDNs fail

#### 2. **Airport Data Loading** (`loadAirportsFromOurAirportsOrFallback()`)
- Fetches OurAirports CSV: `https://raw.githubusercontent.com/davidmegginson/ourairports-data/main/airports.csv`
- Parses CSV with quoted-field support via `parseCsvLine()`
- Filters for DEMO_AIRPORT_IDENTS: ["KCOS", "KAFF", "77CO", "68CO", "CO24", "19CO"]
- Falls back to hardcoded coordinates if network/CORS fails
- Returns stable sorted array matching demo intent

#### 3. **Map Initialization** (`initializeMap()`)
- Sets Esri World Imagery basemap
- Loads airports and creates Leaflet markers with custom `.airport-pill` icons
- Binds hover handlers → `showAirportPanel()`
- Binds click handlers → popups showing detection summaries
- Auto-fits map bounds to all demo airports

#### 4. **Airport Panel Hover** (`showAirportPanel()`, `hideAirportPanelSoon()`)
- Shows detection priority counts grouped by P1–P4
- Hides zero counts (requirement #2)
- Auto-hides on mouseout with 200ms debounce
- Panel "sticks" if user hovers over it
- Displays airport name, ident, lat/lon, and last update date

#### 5. **Modal & Feedback** (`openModal()`, `submitFeedback()`, `submitComment()`)
- Opens detailed detection view with:
  - Detection type, timestamp, coordinates (WGS84)
  - Confidence bar (0.0–1.0 scale)
  - Side-by-side reference vs. detection images
  - Alert JSON preview (what AMC.Maps ingests)
- Analyst identifier required for feedback logging
- Actions: "confirm_valid" or "reject_false_positive"
- Free-text comments optional
- JSON auto-updates after feedback submission

### Data Model

#### Detection Object
```javascript
{
  id: number,
  airport_ident: string,        // e.g., "KCOS"
  type: string,                 // Full category label
  typeShort: string,            // Abbreviated display name
  icao: string,
  timestamp: ISO8601,
  coordinates: { lat, lon },
  confidence: 0.0-1.0,
  priority: 1-4,
  description: string,
  sourceImageTimestamp: ISO8601,
  estimatedHeight?: string      // Optional objective measurement
}
```

#### Alert JSON (Req 2 Output)
```javascript
{
  icao: string,
  detection_id: number,
  detection_type: string,
  detection_timestamp_iso8601: ISO8601,
  coordinates_wgs84: { lat, lon },
  source_image_timestamp_iso8601: ISO8601,
  model_confidence_0_to_1: number,
  link_to_alert_imagery: string,
  optional_estimated_height: string | null,
  analyst_feedback?: {
    analyst_identifier: string,
    action_taken: string,       // "confirm_valid" | "reject_false_positive" | "comment_added"
    free_text_comment: string | null,
    feedback_timestamp_iso8601: ISO8601
  } | null
}
```

### Requirements Mapping

| Requirement | Implementation |
|---|---|
| **Req 2** (Alert JSON) | Embedded in modal, auto-updates on feedback submission |
| **Req 2 Objective** (Analyst logging) | Analyst ID + timestamp + action captured and stored in JSON |
| **Req 3** (Reference + Detection imagery) | Side-by-side image containers with simulated runway/detection box |
| **Req 3 Objective** (Visual cues) | Red bounding box + label on detection image; baseline imagery on left |
| **Req 7** (Retention policy) | Placeholder badge in modal; production would filter detections server-side |

### Browser Compatibility

- **Modern browsers** (Chrome, Firefox, Safari, Edge)
- **Leaflet.js** requires ES6 support
- **Graceful degradation**: If CDN load fails, user sees error modal with instructions

### Performance Notes

- **Single HTML file**: ~40KB uncompressed (no build step required)
- **Lazy loading**: Airport data fetched on page load via async/await
- **Event delegation**: Hover/click handlers attached at marker level (scales to ~100s of markers)
- **CSS animations**: Hardware-accelerated (transform, opacity)

### Security Considerations

- **No backend**: All logic client-side (safe for air-gapped networks)
- **Data escaping**: `escapeHtml()` prevents XSS in detection names/comments
- **CORS handling**: Multi-CDN fallback tolerates network restrictions
- **No authentication**: MVP assumes trusted analyst environment

### Extending TALON EDDYLINE

#### Adding New Detection Types
Update the `detections` array in `<script>`:
```javascript
detections.push({
  id: 50,
  airport_ident: "KNEW",
  type: "Your Detection Type",
  typeShort: "Abbreviated",
  icao: "KNEW",
  timestamp: "2025-01-30T12:00:00Z",
  coordinates: { lat: 38.5, lon: -104.5 },
  confidence: 0.87,
  priority: 2,
  description: "Description of what was detected",
  sourceImageTimestamp: "2025-01-30T10:00:00Z"
});
```

#### Connecting to a Backend
Replace OurAirports fetch in `loadAirportsFromOurAirportsOrFallback()`:
```javascript
const resp = await fetch("/api/airports", { method: "GET" });
const airports = await resp.json();
```

Replace feedback submission in `submitFeedback()`:
```javascript
await fetch("/api/feedback", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(updated)
});
```

---

## 🚀 Getting Started

### Run Locally
1. Clone the repository
2. Open `index.html` in a web browser
3. The map loads automatically; hover/click airports to explore detections

### Troubleshooting

**Map doesn't load?**
- Check browser console for CDN errors
- Try a different network (some restrict unpkg/jsdelivr/cdnjs)
- See "Vendor Option" in error modal for local hosting

**Airports not showing?**
- Network request to OurAirports may be blocked
- App falls back to hardcoded coordinates (still functional)
- Check browser Network tab for CORS errors

---

## 📝 Notes for Analysts

- **Priority 1 (P1)**: Immediate safety hazards (vegetation blocking runway, debris, critical damage)
- **Priority 2 (P2)**: Structural concerns (foundation damage, major cracks)
- **Priority 3 (P3)**: Maintenance items (minor pavement damage, drainage issues)
- **Priority 4 (P4)**: Routine observations (grass growth, surface wear)

Always include your analyst identifier and any relevant comments when providing feedback—this helps the system improve!

---

## 📄 License & Attributions

- **Leaflet.js**: Open-source map library (BSD 2-Clause)
- **OurAirports Data**: Public dataset under CC-BY-SA 4.0
- **Esri Tiles**: Used with attribution per terms of service