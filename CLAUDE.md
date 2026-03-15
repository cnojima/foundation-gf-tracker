# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Foundation GF Tracker is a single-page web application for tracking building upgrades, resource planning, and region progress in the mobile game "Foundation Galactic Frontier". The entire application is contained in a single `index.html` file with embedded CSS and JavaScript.

## Architecture

### Single-File Structure

The application uses a monolithic single-file architecture:
- **HTML Structure** (lines 1016-1119): Defines header, navigation, and four main views (dashboard, planner, resources, regions)
- **CSS Styling** (lines 9-1013): Comprehensive styling with CSS custom properties for theming
- **JavaScript Application** (lines 1121-1857): Data model, state management, and rendering logic

### Data Model

**BUILDINGS Object (lines 1121-1450)**: Defines all building types with structure:
```javascript
{
  name: string,        // Display name
  cat: string,         // Category: 'command', 'movement', 'combat', 'production', 'support'
  icon: string,        // Unicode icon
  color: string,       // Hex color
  desc: string,        // Description
  start: number,       // Starting level (usually 1, Ion Shipyard starts at 0)
  max: number,         // Maximum level (30)
  lvls: {             // Level data
    [level]: [metal, water, coins, time, unlocks, requirements]
    // null values indicate missing data
  }
}
```

**REGIONS Object (lines 1452-1462)**: Game regions with systems and tier levels

**State Object (lines 1468-1473)**:
- `levels`: Current building levels (keyed by building ID)
- `goals`: Target building levels for planning
- `resources`: Current metal/water/coins on hand
- `systems`: Completed region systems (boolean flags)

### State Management

- **Persistence**: LocalStorage with key `fgf_tracker_v2` (line 1467)
- **saveState()** (line 1475): Saves to localStorage
- **loadState()** (line 1478): Loads and initializes missing building levels
- **State mutations** trigger saves and re-renders immediately

### Core Functions

**Building Level Management**:
- `setLevel(key, lvl)` (line 1489): Updates current level, bounded by min/max
- `setGoal(key, lvl)` (line 1498): Sets planning goal for a building
- `clearGoals()` (line 1506): Resets all planning goals

**Resource Calculations**:
- `costBetween(key, from, to)` (lines 1530-1540): Calculates cumulative costs and time between levels, returns `{metal, water, coins, time, hasNull}` where `hasNull` indicates missing data
- `fmt(n)` (line 1516): Formats numbers with K/M/B suffixes
- `fmtTime(s)` (line 1522): Converts seconds to d/h/m/s format

**View Rendering**:
- `renderDashboard()` (line 1543): Building cards grouped by category
- `renderPlanner()` (line 1665): Planning interface with current → goal levels
- `renderSummary()` (line 1699): Total resource/time requirements sidebar
- `renderAffordGrid()` (line 1749): "Can Afford Next Upgrade" list
- `renderRegions()` (line 1766): Region/system checklist
- `openDetail(key)` / `closeDetail()` (lines 1613/1656): Building detail modal

**View Navigation**:
- `showView(id)` (line 1781): Switches between dashboard/planner/resources/regions views

## Development

### Running the Application

No build process required. Open `index.html` directly in a browser:
```bash
open index.html
```

For development with live reload, use any static server:
```bash
python3 -m http.server 8000
# or
npx serve
```

### Testing Changes

1. Make edits to `index.html`
2. Refresh browser (hard refresh: Cmd+Shift+R on macOS)
3. State persists in localStorage between reloads

### Data Entry

Building level data is embedded in the `BUILDINGS.lvls` objects. Format:
```javascript
[metal_cost, water_cost, coins_cost, time_in_seconds, unlock_description, {requirement_building: level}]
```

Use `null` for unknown values. Missing levels (10-17, 25, etc.) have incomplete data.

### Key Implementation Notes

- **Null Handling**: Cost calculations check for `null` values and set `hasNull` flag when encountered
- **Category Filtering**: Buildings grouped by `cat` property for dashboard display
- **Requirements**: Buildings can have upgrade requirements (e.g., Main Cannon 11 requires Guild Liaison 11)
- **Affordability**: Compares current resources against next level costs, highlights affordable upgrades in green
- **Responsive Design**: Media queries at 900px and 500px breakpoints adjust layouts

## GitHub Pages Deployment

The repository is configured to deploy to GitHub Pages. Any changes to `index.html` on the main branch will be automatically published.
