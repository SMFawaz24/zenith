# ZENITH — The Celestial Eye 🔭

A real-time cosmic radar that shows you what's in the sky above **any location on Earth** — the ISS, planets, bright stars, the Moon, and the Sun — computed with astronomical precision and rendered through three immersive visualizations.

> **[▶ Live Demo](https://zenith-smfawaz24.vercel.app)** · **[GitHub](https://github.com/SMFawaz24/zenith)**  · Built for Astral Web Innovate Hackathon

![ZENITH Preview](https://img.shields.io/badge/ZENITH-Live-blue?style=for-the-badge)

---

## What It Does

Pick any location on Earth → instantly see what's in the sky above it right now.

Type a city name, click on the globe, or share your GPS — ZENITH computes the real-time position of **30+ celestial objects** using astronomical algorithms and displays them across three interactive views: a 3D rotating globe, a polar sky map, and an animated radar sweep.

---

## Features

### Core
- 🌍 **Interactive 3D Globe** — Canvas 2D Earth with detailed continent coastlines, atmospheric glow, grid lines, and specular lighting
- 🗺️ **Sky Map** — Stereographic polar projection showing the dome of the sky above you (zenith at center, horizon at edge)
- 📡 **Radar View** — Rotating sweep with phosphor-green trail, showing objects as radar blips
- 🔄 **Smooth view switching** — Crossfade transitions between all three modes

### Real-Time Tracking
- 🛰️ **ISS Live Tracking** — Position fetched every 5 seconds from NASA's API, with orbit trail drawn on the globe
- ⚡ **ISS Telemetry** — Live speed (km/s), orbital altitude, lat/lon coordinates
- 📅 **ISS Pass Predictions** — Next 5 upcoming passes with time, duration, and max elevation for your location
- 🗺️ **ISS Ground Track** — Mini SVG world map showing the orbital path and current position

### Celestial Objects (30+)
| Category | Objects | Method |
|---|---|---|
| Star | Sun | Astronomical Almanac low-precision solar position |
| Satellite | Moon (with phase) | ELP2000 truncated lunar theory |
| Planets | Mercury, Venus, Mars, Jupiter, Saturn | Mean orbital elements from J2000 epoch |
| Stars | Sirius, Canopus, Arcturus, Vega, Capella, Rigel, Procyon, Betelgeuse, Altair, Aldebaran, Antares, Spica, Pollux, Fomalhaut, Deneb, Regulus | FK5 J2000 catalog |
| Satellites | ISS, Hubble, NOAA-19, Sentinel-2A, Landsat-9, GOES-18, Starlink, GPS-IIF, Terra | Live API + orbital simulation |

### Location Input
- 🔍 **City search** — Type any city name (powered by OpenStreetMap Nominatim)
- 📍 **GPS** — One-click browser geolocation
- 🖱️ **Globe click** — Click anywhere on the 3D globe to set observer position
- ⌨️ **Manual coordinates** — Enter `lat, lon` directly (e.g., `40.7, -74.0`)

### UI/UX
- 🌙 **Moon phase visualization** — Phase name, illumination %, and rendered phase graphic
- 📊 **Object detail cards** — Click any object for altitude, azimuth, distance, magnitude
- 🔘 **Filter toggles** — Show/hide ISS, Satellites, Planets, Stars, Moon, Sun, Constellations, Orbits
- ✨ **Constellation lines** — Summer Triangle and Orion Belt with animated stroke drawing
- 📱 **Responsive design** — Desktop 3-column, tablet 2-column, mobile full-screen with bottom sheet
- 👆 **Touch support** — Full touch interaction for globe rotation on mobile
- 🎬 **Boot animation** — Clean fade-in sequence with staggered UI reveal

---

## How to Run Locally

**Zero setup required.** This is a single HTML file with no dependencies.

```bash
# Option 1: Just open it
# Double-click index.html in your file explorer, or:
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux

# Option 2: Local server (needed for GPS to work)
npx serve .
# Then open http://localhost:3000
```

> **Note:** GPS (navigator.geolocation) requires HTTPS or localhost. All other features work by opening the file directly.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 |
| Styling | Vanilla CSS (no Tailwind, no frameworks) |
| Logic | Vanilla JavaScript (no React, no libraries) |
| Rendering | HTML5 Canvas 2D API |
| Fonts | Google Fonts (Cormorant Garamond, Inter, DM Mono) |
| Hosting | Vercel (static deployment) |

**Total dependencies: 0** · **Total files: 1** · **Build step: none**

---

## APIs Used

### ISS Position — `api.wheretheiss.at`
```
GET https://api.wheretheiss.at/v1/satellites/25544
```
Returns real-time ISS latitude, longitude, altitude (km), and velocity (km/h).  
Polled every **5 seconds**. On failure, falls back to orbital simulation using:
```
lat = 51.6° × sin(t / 5580s × 2π)    // 51.6° orbital inclination
lon = (t × 360° / 5580s) mod 360 - 180  // ~93 min orbital period
```

**Why this API:** Free, no API key required, returns clean JSON, reliable uptime.

### Geocoding — OpenStreetMap Nominatim
```
GET https://nominatim.openstreetmap.org/search?q={city}&format=json&limit=1
```
Converts city names to lat/lon coordinates. Requires a `User-Agent` header.

**Why this API:** Free, no API key, global coverage, respects privacy.

---

## Astronomical Algorithms

All celestial positions are computed **client-side** using formulas from Jean Meeus' *Astronomical Algorithms* (the standard reference for positional astronomy). No external APIs needed.

### Time System
```
Julian Day:  JD = Date.now() / 86400000 + 2440587.5
Century:     T  = (JD - 2451545.0) / 36525.0
```

### Sidereal Time
```
GMST = 280.46061837 + 360.98564736629 × (JD - 2451545) + 0.000387933 × T²
LST  = (GMST + observer_longitude) mod 360
```
Greenwich Mean Sidereal Time tells us which stars are overhead at Greenwich; adding the observer's longitude gives the Local Sidereal Time.

### Coordinate Conversion (Equatorial → Horizontal)
Converts celestial coordinates (RA/Dec) to what you actually see (altitude above horizon, compass direction):
```
Hour Angle:  H = LST - RA
Altitude:    sin(alt) = sin(dec)·sin(lat) + cos(dec)·cos(lat)·cos(H)
Azimuth:     from atan2, corrected for quadrant
```

### Solar Position (Astronomical Almanac)
```
n = JD - 2451545.0                           // days from J2000
L = (280.460 + 0.9856474·n) mod 360          // mean longitude
g = (357.528 + 0.9856003·n)°                 // mean anomaly
λ = L + 1.915·sin(g) + 0.020·sin(2g)        // ecliptic longitude
ε = 23.439 - 0.0000004·n                     // obliquity
RA  = atan2(cos(ε)·sin(λ), cos(λ))
Dec = asin(sin(ε)·sin(λ))
```

### Lunar Position (ELP2000 Truncated)
```
L' = 218.3164 + 481267.8812·T               // mean longitude
M  = 134.9634 + 477198.8676·T               // mean anomaly
D  = 297.8502 + 445267.1115·T               // mean elongation
F  = 93.2721 + 483202.0175·T                // argument of latitude
λ  = L' + 6.289·sin(M) + 1.274·sin(2D-M) + 0.658·sin(2D)
β  = -5.128·sin(F)
```
Phase is computed from the elongation angle between Moon and Sun.

### Planet Positions
Simplified mean orbital elements propagated from J2000 epoch:
```
Mean anomaly:  M = M₀ + Md × (JD - 2451545)
RA ≈ M mod 360
Dec ≈ inclination × sin(RA)
```

### Star Catalog
16 brightest stars from the FK5 J2000 catalog with fixed RA/Dec coordinates (stellar proper motion is negligible over human timescales).

---

## Design Philosophy

**Direction: Obsidian & Starlight** — inspired by Brian Eno's Cosmos app, Apple Vision Pro spatial UI, and Monocle magazine.

| Principle | Implementation |
|---|---|
| One strong visual idea | The globe/sky IS the interface, not a background behind panels |
| Restraint | Pure black `#000000`, white text, single desaturated accent `#a8c4d4` |
| Negative space | No sidebars permanently visible; elements float in contextually |
| Typography hierarchy | Display (Cormorant Garamond 300), UI (Inter 300/500), Data (DM Mono 300) |
| Glass morphism | `backdrop-filter: blur(24px)` on all floating elements |
| Micro-interactions | 400ms fade-ins, hover opacity shifts, constellation line drawing, parallax star field |
| 60fps | All animations use `requestAnimationFrame`, no DOM thrashing |

---

## Architecture

```
index.html (single file, ~2200 lines)
│
├── <style>           — Design system (CSS custom properties, glass effects, responsive)
├── <body>            — Semantic HTML (canvas + floating UI overlays)
└── <script>          — Application logic
    │
    ├── Constants     — Star catalog, planet elements, satellite data
    ├── Astronomy     — Jean Meeus formulas (JD, GMST, LST, eq→hz, Sun, Moon, planets)
    ├── ISS Tracker   — API fetch with fallback simulation, pass predictions
    ├── Globe         — Canvas 2D Earth rendering (continents, atmosphere, grid, objects)
    ├── Sky Map       — Polar stereographic projection with constellation lines
    ├── Radar         — Animated sweep with phosphor trail
    ├── UI            — Object list, detail cards, filters, solar status, telemetry
    ├── Location      — Nominatim search, GPS, globe click, manual input
    ├── Events        — Mouse/touch handlers, view switching, drag rotation
    ├── Render Loop   — 60fps requestAnimationFrame with throttled UI updates
    └── Boot          — Staggered reveal animation sequence
```

---

## Project Structure

```
zenith/
├── index.html        # The entire application (single file, zero dependencies)
├── README.md         # This file
└── .gitignore        # Git ignore rules
```

---

## Deployment

### Vercel (recommended)
```bash
# 1. Push to GitHub
git init
git add .
git commit -m "ZENITH — The Celestial Eye"
git remote add origin https://github.com/YOUR_USERNAME/zenith.git
git push -u origin main

# 2. Go to vercel.com → Import Git Repository → Select your repo
# 3. Deploy (no configuration needed — Vercel auto-detects static sites)
# 4. Your URL: https://zenith-yourname.vercel.app
```

### Netlify (alternative)
```bash
# Drag-and-drop the folder at app.netlify.com/drop
# Or connect your GitHub repo at netlify.com
```

---

## Browser Support

| Browser | Status |
|---|---|
| Chrome 90+ | ✅ Full support |
| Firefox 90+ | ✅ Full support |
| Safari 15+ | ✅ Full support |
| Edge 90+ | ✅ Full support |
| Mobile Chrome/Safari | ✅ Touch support included |

---

## What I'd Build Next

Given more time, these would be the next additions:

- **WebGL globe** — Replace Canvas 2D with Three.js for a photorealistic Earth with cloud layers and day/night terminator
- **TLE parsing** — Use real Two-Line Element sets for accurate satellite tracking instead of simulated orbits
- **Notification system** — Push alerts when the ISS is about to pass over your location
- **Augmented reality** — Use device orientation API to overlay the sky map on your phone's camera
- **Deep sky objects** — Messier catalog (galaxies, nebulae, star clusters)
- **Historical mode** — Scrub through time to see the sky at any date/time
- **Multi-language** — i18n support for the interface
- **PWA** — Offline support with service worker for cached star/planet data

---

## License

MIT — Built for the Astral Web Innovate hackathon.

---

<p align="center">
  <strong>ZENITH</strong> · The Celestial Eye<br>
  <em>See what's above you.</em>
</p>
