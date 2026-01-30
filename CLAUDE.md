# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ConverTool Web is a single-page web application that converts KiCad placement files (CSV/POS) to Charmhigh CHMT-48VB pick-and-place machine DPV format. The entire application is contained in a single `index.html` file with embedded CSS and JavaScript.

## Architecture

### Single File Structure
- **index.html** - Contains everything: HTML structure, CSS styles, and JavaScript (~3500 lines)
- Uses jQuery 3.7.1 (loaded from CDN)
- No build system, bundler, or dependencies to install

### Key Data Model
```javascript
dpvModel = {
  headerLines: [],  // DPV file header metadata
  tables: {
    'EComponent': { header: [], rows: [] },  // Component placements
    'Station': { header: [], rows: [] },      // Material stacks/feeders
    'Panel_Array': { header: [], rows: [] },  // Panel array config
    'Panel_Coord': { header: [], rows: [] }   // Panel coordinates
  }
}
```

### Tab Structure
1. **CSV/POS** - Load/edit KiCad placement data
2. **Components** - DPV EComponent table (placement coordinates)
3. **Material Stacks** - DPV Station table (feeder assignments)
4. **Panel List** - Panel_Array and Panel_Coord tables

### Custom DPVx Format
Extended DPV format with additional `PHead` column in Station table. Standard DPV export strips this column for machine compatibility.

### Browser Storage
- **Master Feeder Library**: `localStorage` key `chmt48vb_feeder_library` - stores feeder configurations by component value (Note)
- **Workflow preference**: `localStorage` key `chmt48vb_hide_workflow` - controls Getting Started dialog

## Deployment

Deploy to remote server:
```bash
sshpass -p 'JLF6ezBhtFEt2@?' scp /home/zditech/CHWebApp/index.html root@srv629042.hstgr.cloud:/root/webserver/sites/default/index.html
```

## Key Functions

- `generateDpvFromCsv()` - Converts CSV data to DPV model
- `parseDpv(text)` / `dpvToText()` - Parse and serialize DPV files
- `validateDpv()` - Validates DPV structure before save
- `autoApplyLibraryToModel()` - Applies Master Feeder Library to stations
- `syncPHeadToComponents(note, phead)` - Syncs Station PHead to matching EComponents
- `sortStationTableById()` - Auto-sorts Material Stacks by ID

## DPV Format Notes

- **Station.ID** maps to **EComponent.STNo.** (feeder slot reference)
- **Station.Note** matches **EComponent.Explain** (component value like "10k")
- IDs must be unique; changing an ID updates all STNo. references
- Standard Charmhigh DPV does not include PHead in Station table
