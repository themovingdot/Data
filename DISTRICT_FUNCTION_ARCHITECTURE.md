# District Function Architecture

Visual guide showing how the dynamic district system works.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR FORM COMPONENT                      │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │  State Management                                   │   │
│  │  • dynamicDistricts: DistrictInfo[]                │   │
│  │  • isLoadingDistricts: boolean                     │   │
│  │  • districtSearch: string                          │   │
│  │  • selectedDistrict: string                        │   │
│  └────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           │ useEffect on mount              │
│                           ▼                                 │
│  ┌────────────────────────────────────────────────────┐   │
│  │  loadDistricts() async function                    │   │
│  │  1. Call getDistrictsFromGeoJSON()                 │   │
│  │  2. Set dynamicDistricts if successful             │   │
│  │  3. Fallback to makkahDistricts if fails           │   │
│  └────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │
                            │ Calls
                            ▼
┌─────────────────────────────────────────────────────────────┐
│            /utils/districtBoundaries.ts                     │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │  getDistrictsFromGeoJSON()                         │   │
│  │                                                     │   │
│  │  1. Call loadDistrictBoundaries()                  │   │
│  │  2. Extract district properties from each feature  │   │
│  │  3. Calculate polygon centroid (lat/lng)           │   │
│  │  4. Return DistrictInfo[]                          │   │
│  └────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           │ Calls                           │
│                           ▼                                 │
│  ┌────────────────────────────────────────────────────┐   │
│  │  loadDistrictBoundaries()                          │   │
│  │                                                     │   │
│  │  1. Fetch from GitHub URL                          │   │
│  │  2. Parse GeoJSON                                  │   │
│  │  3. Transform UTM to WGS84 if needed               │   │
│  │  4. Return DistrictBoundaryCollection              │   │
│  └────────────────────────────────────────────────────┘   │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │ Fetches from
                            ▼
┌─────────────────────────────────────────────────────────────┐
│     GitHub Repository (Raw URL)                             │
│     https://raw.githubusercontent.com/themovingdot/         │
│     Data/main/districtBoundaries.json                       │
│                                                             │
│     GeoJSON FeatureCollection with:                         │
│     • District polygons/multipolygons                       │
│     • Properties: DISTRICT_N, DIST_EN, name_ar             │
│     • Coordinates in UTM or WGS84                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Data Flow Diagram

```
┌──────────┐
│ GeoJSON  │
│   File   │
└─────┬────┘
      │
      │ 1. Fetch
      ▼
┌─────────────────────────┐
│ loadDistrictBoundaries()│
│ • Parse JSON            │
│ • Transform coords      │
└─────────┬───────────────┘
          │
          │ 2. Return boundaries
          ▼
┌─────────────────────────┐
│ getDistrictsFromGeoJSON│
│ • Extract properties   │
│ • Calculate centroids  │
│ • Format output        │
└─────────┬───────────────┘
          │
          │ 3. Return DistrictInfo[]
          ▼
┌─────────────────────────┐
│   Form Component       │
│ • Store in state       │
│ • Render dropdown      │
│ • Enable search        │
└─────────────────────────┘
```

---

## 🔄 Component Lifecycle

```
Component Mount
      │
      ▼
┌──────────────────────┐
│  useEffect runs      │
│  (empty deps [])     │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ setIsLoadingDistricts│
│     (true)           │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Call                 │
│ getDistrictsFromGeo  │
│      JSON()          │
└──────┬───────────────┘
       │
       ├─── Success ───┐
       │               ▼
       │      ┌────────────────────┐
       │      │ setDynamicDistricts│
       │      │ (loaded data)      │
       │      └────────┬───────────┘
       │               │
       └─── Failure ───┤
                       ▼
              ┌────────────────────┐
              │ setDynamicDistricts│
              │ (fallback list)    │
              └────────┬───────────┘
                       │
                       ▼
              ┌────────────────────┐
              │setIsLoadingDistricts│
              │     (false)        │
              └────────┬───────────┘
                       │
                       ▼
              ┌────────────────────┐
              │  Render UI         │
              │  Ready for user    │
              └────────────────────┘
```

---

## 🎨 UI Component Structure

```
┌─────────────────────────────────────────────────────┐
│  District Selector Component                        │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │ Loading State (Conditional)                   │ │
│  │  ⟳ "Loading district list..."                │ │
│  └───────────────────────────────────────────────┘ │
│                      OR                             │
│  ┌───────────────────────────────────────────────┐ │
│  │ Main UI                                       │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │ Label                                   │ │ │
│  │  │ "District *"                            │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │ Search Input                            │ │ │
│  │  │  🔍 [Search district...]           ✕   │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │ Select Dropdown                         │ │ │
│  │  │  ▼ [Select District]                    │ │ │
│  │  │                                         │ │ │
│  │  │  When opened:                           │ │ │
│  │  │  ┌───────────────────────────────────┐ │ │ │
│  │  │  │ 001 - Ajyad                       │ │ │ │
│  │  │  │ 002 - Al Haram and Al Hegla       │ │ │ │
│  │  │  │ 003 - Al Shabikah                 │ │ │ │
│  │  │  │ ...                               │ │ │ │
│  │  │  │ (scrollable, max-height: 300px)   │ │ │ │
│  │  │  └───────────────────────────────────┘ │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │ District Count Info                     │ │ │
│  │  │ "45 of 60 districts"                    │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## 🔍 Search Filter Logic

```
User Types: "ajyad"
      │
      ▼
┌──────────────────────────┐
│ setDistrictSearch        │
│    ("ajyad")             │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Filter dynamicDistricts  │
│                          │
│ For each district:       │
│   - Check ID contains    │
│     "ajyad" ✗            │
│   - Check nameEn         │
│     contains "ajyad" ✓   │
│   - Check nameAr         │
│     contains "ajyad" ✗   │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ filteredDistricts =      │
│ [                        │
│   {                      │
│     id: "001",           │
│     nameEn: "Ajyad",     │
│     nameAr: "أجياد",     │
│     ...                  │
│   }                      │
│ ]                        │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Re-render dropdown with  │
│ only matching results    │
└──────────────────────────┘
```

---

## 🗺️ Map Integration Flow

```
User Selects District
      │
      ▼
┌──────────────────────────┐
│ onValueChange triggered  │
│   value = "001"          │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ setSelectedDistrict      │
│      ("001")             │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Find district in         │
│ dynamicDistricts         │
│                          │
│ district = {             │
│   id: "001",             │
│   lat: 21.4205,          │
│   lng: 39.8262,          │
│   zoom: 15               │
│ }                        │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ setMapZoomLocation       │
│   ({                     │
│     lat: 21.4205,        │
│     lng: 39.8262,        │
│     zoom: 15             │
│   })                     │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Map component receives   │
│ new zoomToLocation prop  │
└──────┬───────────────────┘
       │
       ▼
┌──────────────────────────┐
│ Map animates to          │
│ district location        │
│ and highlights boundary  │
└──────────────────────────┘
```

---

## 📦 File Dependencies

```
Your Form Component
    │
    ├── Imports: getDistrictsFromGeoJSON, DistrictInfo
    │   from: /utils/districtBoundaries.ts
    │
    ├── Imports: makkahDistricts (fallback)
    │   from: /utils/districts.ts
    │
    ├── Imports: UI Components
    │   from: ./ui/select, ./ui/input, ./ui/label
    │
    └── Imports: Icons
        from: lucide-react

/utils/districtBoundaries.ts
    │
    ├── Exports: getDistrictsFromGeoJSON()
    ├── Exports: loadDistrictBoundaries()
    ├── Exports: DistrictInfo interface
    ├── Exports: DistrictBoundary interface
    │
    └── Fetches: GitHub GeoJSON URL

/utils/districts.ts
    │
    ├── Exports: makkahDistricts[] (60 districts)
    └── Exports: externalZones[] (11 zones)
```

---

## 🎯 Key Functions Summary

### 1. `getDistrictsFromGeoJSON()`
**Location:** `/utils/districtBoundaries.ts`  
**Returns:** `Promise<DistrictInfo[]>`  
**Purpose:** Extract district list from GeoJSON boundaries  

**Steps:**
1. Call `loadDistrictBoundaries()`
2. Loop through each feature
3. Extract: `DISTRICT_N`, `DIST_EN`, `name_ar`
4. Calculate centroid from polygon coordinates
5. Return array of districts

### 2. `loadDistrictBoundaries()`
**Location:** `/utils/districtBoundaries.ts`  
**Returns:** `Promise<DistrictBoundaryCollection | null>`  
**Purpose:** Load and parse GeoJSON from GitHub  

**Steps:**
1. Fetch from GitHub URL
2. Parse JSON response
3. Check if coordinate transformation needed (UTM → WGS84)
4. Transform if necessary
5. Return boundary collection

### 3. `calculateCentroid()`
**Location:** `/utils/districtBoundaries.ts`  
**Returns:** `[number, number]` (lat, lng)  
**Purpose:** Calculate center point of polygon  

**Steps:**
1. Take first ring (outer boundary) of polygon
2. Sum all latitude values
3. Sum all longitude values
4. Divide by number of points
5. Return average (centroid)

---

## 💾 Data Structure

### DistrictInfo Interface
```typescript
interface DistrictInfo {
  id: string;        // "001", "002", etc.
  nameEn: string;    // "Ajyad", "Al Haram", etc.
  nameAr: string;    // "أجياد", "الحرم", etc.
  lat: number;       // 21.4205
  lng: number;       // 39.8262
  zoom: number;      // 15
}
```

### GeoJSON Structure (Simplified)
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "DISTRICT_N": "001",
        "DIST_EN": "Ajyad",
        "name_ar": "أجياد",
        "NO": 1
      },
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [39.8200, 21.4220],
            [39.8250, 21.4220],
            [39.8250, 21.4180],
            [39.8200, 21.4180],
            [39.8200, 21.4220]
          ]
        ]
      }
    }
  ]
}
```

---

## 🚀 Quick Copy-Paste Checklist

To add districts to a new form:

- [ ] Import `getDistrictsFromGeoJSON` and `DistrictInfo`
- [ ] Import fallback `makkahDistricts`
- [ ] Add state: `dynamicDistricts`, `isLoadingDistricts`, `districtSearch`
- [ ] Add `useEffect` to load districts on mount
- [ ] Add filter logic for search
- [ ] Render loading state
- [ ] Render search input
- [ ] Render Select dropdown
- [ ] Render district count
- [ ] Add onValueChange handler
- [ ] (Optional) Add map zoom integration

---

That's the complete architecture! Everything you need to transport this function to any form. 🎉
