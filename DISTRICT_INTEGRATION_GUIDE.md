# Dynamic District Integration Guide

Complete guide to integrate dynamic Makkah districts loading from GeoJSON in any form.

---

## 📋 Overview

This guide shows how to:
1. Load districts dynamically from GeoJSON map boundaries
2. Display districts in a searchable dropdown
3. Integrate with map for zoom and highlighting
4. Implement bilingual support (English/Arabic)

---

## 🗂️ Step 1: Import Required Functions and Types

```tsx
import { getDistrictsFromGeoJSON, type DistrictInfo } from '../utils/districtBoundaries';
import { makkahDistricts, externalZones } from '../utils/districts'; // Fallback
```

### What You Get:
- **`DistrictInfo`** interface:
  ```typescript
  interface DistrictInfo {
    id: string;        // e.g., "001", "002"
    nameEn: string;    // e.g., "Ajyad"
    nameAr: string;    // e.g., "أجياد"
    lat: number;       // Centroid latitude
    lng: number;       // Centroid longitude
    zoom: number;      // Recommended zoom level (usually 15)
  }
  ```

- **`getDistrictsFromGeoJSON()`** function:
  - Returns: `Promise<DistrictInfo[]>`
  - Loads from: `https://raw.githubusercontent.com/themovingdot/Data/main/districtBoundaries.json`
  - Auto-calculates centroids from polygon boundaries
  - Handles UTM to WGS84 coordinate transformation if needed

---

## 🎯 Step 2: Add State Variables in Your Component

```tsx
export default function YourFormComponent() {
  // State for dynamic districts loaded from GeoJSON
  const [dynamicDistricts, setDynamicDistricts] = useState<DistrictInfo[]>([]);
  const [isLoadingDistricts, setIsLoadingDistricts] = useState(true);
  
  // State for search functionality (optional but recommended)
  const [districtSearch, setDistrictSearch] = useState('');
  
  // State for selected district
  const [selectedDistrict, setSelectedDistrict] = useState('');
  
  // State for map zoom (optional - if you have a map)
  const [mapZoomLocation, setMapZoomLocation] = useState<{ 
    lat: number; 
    lng: number; 
    zoom: number 
  } | null>(null);

  // ... rest of your component
}
```

---

## 🔄 Step 3: Load Districts on Component Mount

```tsx
useEffect(() => {
  const loadDistricts = async () => {
    setIsLoadingDistricts(true);
    console.log('📋 Loading districts from GeoJSON...');
    
    try {
      const districts = await getDistrictsFromGeoJSON();
      
      if (districts.length > 0) {
        setDynamicDistricts(districts);
        console.log(`✅ Loaded ${districts.length} districts from GeoJSON`);
      } else {
        // Fallback to hardcoded list if GeoJSON returns empty
        console.log('⚠️ GeoJSON returned no districts, using hardcoded fallback');
        setDynamicDistricts(makkahDistricts);
      }
    } catch (error) {
      console.error('❌ Failed to load districts from GeoJSON, using fallback:', error);
      setDynamicDistricts(makkahDistricts); // Fallback to hardcoded list
    } finally {
      setIsLoadingDistricts(false);
    }
  };
  
  loadDistricts();
}, []); // Empty dependency array = run once on mount
```

---

## 🔍 Step 4: Implement Search Filter Logic

```tsx
// Filter districts based on search input
const filteredDistricts = dynamicDistricts.filter(district => {
  if (!districtSearch) return true; // Show all if no search
  
  const searchLower = districtSearch.toLowerCase();
  return (
    district.id.toLowerCase().includes(searchLower) ||
    district.nameEn.toLowerCase().includes(searchLower) ||
    district.nameAr.includes(districtSearch) // Arabic doesn't need toLowerCase
  );
});
```

---

## 🎨 Step 5: Render the District Selection UI

### 5A. Loading State

```tsx
{isLoadingDistricts && (
  <div className="p-4 bg-blue-50 rounded-lg border border-blue-200">
    <p className="text-blue-700 flex items-center gap-2">
      <RefreshCw className="w-4 h-4 animate-spin" />
      {language === 'ar' ? 'جاري تحميل قائمة الأحياء...' : 'Loading district list...'}
    </p>
  </div>
)}
```

### 5B. Search Bar (Optional but Recommended)

```tsx
{!isLoadingDistricts && (
  <div>
    <Label className="text-base mb-2 block">
      {language === 'ar' ? 'الحي' : 'District'}
      <span className="text-red-500">*</span>
    </Label>
    
    {/* Search Input */}
    <div className="relative mb-2">
      <Search className={`absolute ${language === 'ar' ? 'right-3' : 'left-3'} top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400`} />
      <Input
        type="text"
        placeholder={language === 'ar' ? 'ابحث عن الحي...' : 'Search district...'}
        value={districtSearch}
        onChange={(e) => setDistrictSearch(e.target.value)}
        className={`h-10 text-base ${language === 'ar' ? 'pr-10 pl-8' : 'pl-10 pr-8'}`}
      />
      {districtSearch && (
        <button
          onClick={() => setDistrictSearch('')}
          className={`absolute ${language === 'ar' ? 'left-3' : 'right-3'} top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600`}
        >
          ✕
        </button>
      )}
    </div>
    
    {/* Dropdown continues below... */}
  </div>
)}
```

### 5C. District Dropdown

```tsx
<Select 
  value={selectedDistrict} 
  onValueChange={(value) => {
    setSelectedDistrict(value);
    
    // Optional: Zoom map to selected district
    const district = dynamicDistricts.find(d => d.id === value);
    if (district) {
      setMapZoomLocation({ 
        lat: district.lat, 
        lng: district.lng, 
        zoom: district.zoom 
      });
    }
  }}
>
  <SelectTrigger className="h-12 text-base">
    <SelectValue placeholder={language === 'ar' ? 'اختر الحي' : 'Select District'} />
  </SelectTrigger>
  
  <SelectContent className="max-h-[300px]">
    {filteredDistricts.length > 0 ? (
      filteredDistricts.map((district) => (
        <SelectItem 
          key={district.id} 
          value={district.id} 
          className="text-base py-3"
        >
          {district.id} - {language === 'ar' ? district.nameAr : district.nameEn}
        </SelectItem>
      ))
    ) : (
      <div className="text-center py-4 text-gray-500">
        {language === 'ar' ? 'لا توجد نتائج' : 'No results found'}
      </div>
    )}
  </SelectContent>
</Select>

{/* District count info */}
<p className="text-xs text-gray-500 mt-1">
  {language === 'ar' 
    ? `${filteredDistricts.length} من ${dynamicDistricts.length} حي` 
    : `${filteredDistricts.length} of ${dynamicDistricts.length} districts`}
</p>
```

---

## 🗺️ Step 6: Integrate with Map (Optional)

### 6A. Display Selected District on Map

If you're using a map component like DualLocationPicker:

```tsx
<DualLocationPicker
  // ... other props
  originDistrictId={selectedDistrict}
  zoomToLocation={mapZoomLocation}
  onOriginSelect={(lat, lng, address, district, neighborhood) => {
    console.log('Location selected:', lat, lng, address, district, neighborhood);
    // Update your form state here
  }}
/>
```

### 6B. Get District Name for Display

```tsx
// Helper function to get district display name
const getDistrictDisplayName = (districtId: string, language: 'en' | 'ar'): string => {
  const district = dynamicDistricts.find(d => d.id === districtId);
  const zone = externalZones.find(z => z.id === districtId);
  
  if (district) {
    return `${district.id} - ${language === 'ar' ? district.nameAr : district.nameEn}`;
  } else if (zone) {
    return `${zone.id}. ${language === 'ar' ? zone.nameAr : zone.nameEn}`;
  }
  
  return districtId;
};

// Usage in map component
const districtName = getDistrictDisplayName(selectedDistrict, language);
```

---

## 📦 Complete Example - Minimal Implementation

Here's a minimal working example you can copy:

```tsx
import { useState, useEffect } from 'react';
import { getDistrictsFromGeoJSON, type DistrictInfo } from '../utils/districtBoundaries';
import { makkahDistricts } from '../utils/districts';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from './ui/select';
import { Input } from './ui/input';
import { Label } from './ui/label';
import { Search, RefreshCw } from 'lucide-react';

export default function DistrictSelector({ 
  language = 'en',
  onDistrictSelect 
}: { 
  language?: 'en' | 'ar';
  onDistrictSelect: (districtId: string) => void;
}) {
  const [dynamicDistricts, setDynamicDistricts] = useState<DistrictInfo[]>([]);
  const [isLoadingDistricts, setIsLoadingDistricts] = useState(true);
  const [districtSearch, setDistrictSearch] = useState('');
  const [selectedDistrict, setSelectedDistrict] = useState('');

  // Load districts on mount
  useEffect(() => {
    const loadDistricts = async () => {
      setIsLoadingDistricts(true);
      try {
        const districts = await getDistrictsFromGeoJSON();
        setDynamicDistricts(districts.length > 0 ? districts : makkahDistricts);
      } catch (error) {
        console.error('Failed to load districts:', error);
        setDynamicDistricts(makkahDistricts);
      } finally {
        setIsLoadingDistricts(false);
      }
    };
    loadDistricts();
  }, []);

  // Filter districts
  const filteredDistricts = dynamicDistricts.filter(district => {
    if (!districtSearch) return true;
    const searchLower = districtSearch.toLowerCase();
    return (
      district.id.toLowerCase().includes(searchLower) ||
      district.nameEn.toLowerCase().includes(searchLower) ||
      district.nameAr.includes(districtSearch)
    );
  });

  // Loading state
  if (isLoadingDistricts) {
    return (
      <div className="p-4 bg-blue-50 rounded-lg border border-blue-200">
        <p className="text-blue-700 flex items-center gap-2">
          <RefreshCw className="w-4 h-4 animate-spin" />
          {language === 'ar' ? 'جاري تحميل قائمة الأحياء...' : 'Loading district list...'}
        </p>
      </div>
    );
  }

  // Main UI
  return (
    <div>
      <Label className="text-base mb-2 block">
        {language === 'ar' ? 'الحي' : 'District'}
        <span className="text-red-500">*</span>
      </Label>
      
      {/* Search Bar */}
      <div className="relative mb-2">
        <Search className={`absolute ${language === 'ar' ? 'right-3' : 'left-3'} top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400`} />
        <Input
          type="text"
          placeholder={language === 'ar' ? 'ابحث عن الحي...' : 'Search district...'}
          value={districtSearch}
          onChange={(e) => setDistrictSearch(e.target.value)}
          className={`h-10 text-base ${language === 'ar' ? 'pr-10 pl-8' : 'pl-10 pr-8'}`}
        />
        {districtSearch && (
          <button
            onClick={() => setDistrictSearch('')}
            className={`absolute ${language === 'ar' ? 'left-3' : 'right-3'} top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600`}
          >
            ✕
          </button>
        )}
      </div>
      
      {/* District Dropdown */}
      <Select 
        value={selectedDistrict} 
        onValueChange={(value) => {
          setSelectedDistrict(value);
          onDistrictSelect(value);
        }}
      >
        <SelectTrigger className="h-12 text-base">
          <SelectValue placeholder={language === 'ar' ? 'اختر الحي' : 'Select District'} />
        </SelectTrigger>
        <SelectContent className="max-h-[300px]">
          {filteredDistricts.length > 0 ? (
            filteredDistricts.map((district) => (
              <SelectItem 
                key={district.id} 
                value={district.id} 
                className="text-base py-3"
              >
                {district.id} - {language === 'ar' ? district.nameAr : district.nameEn}
              </SelectItem>
            ))
          ) : (
            <div className="text-center py-4 text-gray-500">
              {language === 'ar' ? 'لا توجد نتائج' : 'No results found'}
            </div>
          )}
        </SelectContent>
      </Select>
      
      {/* District count */}
      <p className="text-xs text-gray-500 mt-1">
        {language === 'ar' 
          ? `${filteredDistricts.length} من ${dynamicDistricts.length} حي` 
          : `${filteredDistricts.length} of ${dynamicDistricts.length} districts`}
      </p>
    </div>
  );
}
```

---

## 🎯 Usage in Any Form

```tsx
<DistrictSelector 
  language={language}
  onDistrictSelect={(districtId) => {
    // Save to your form state
    setFormData({ ...formData, district: districtId });
    
    // Optional: Zoom map
    const district = dynamicDistricts.find(d => d.id === districtId);
    if (district) {
      setMapZoomLocation({ lat: district.lat, lng: district.lng, zoom: 15 });
    }
  }}
/>
```

---

## 📝 Key Features Implemented

✅ **Dynamic Loading** - Loads districts from live GeoJSON file  
✅ **Automatic Fallback** - Uses hardcoded list if GeoJSON fails  
✅ **Search Functionality** - Filter by ID, English name, or Arabic name  
✅ **Bilingual Support** - Full English and Arabic support  
✅ **Loading States** - Shows spinner while loading  
✅ **District Count** - Shows filtered/total count  
✅ **Map Integration** - Auto-zoom to selected district  
✅ **Clear Button** - Clear search with ✕ button  

---

## 🔧 Troubleshooting

### Districts not loading?
- Check console for errors
- Verify GeoJSON URL is accessible: `https://raw.githubusercontent.com/themovingdot/Data/main/districtBoundaries.json`
- System will automatically fallback to hardcoded list

### Search not working?
- Make sure `districtSearch` state is properly connected to Input component
- Verify filter logic includes both English and Arabic

### Map not zooming?
- Ensure `mapZoomLocation` state is passed to map component
- Check that map component supports `zoomToLocation` prop

---

## 📚 Related Files

- **`/utils/districtBoundaries.ts`** - GeoJSON loading and parsing
- **`/utils/districts.ts`** - Hardcoded fallback district list
- **`/components/RoadsideInterviewSurvey.tsx`** - Full implementation example
- **`/components/DualLocationPicker.tsx`** - Map component with district display

---

## 🚀 Next Steps

1. Copy the minimal example above
2. Adjust styling to match your form
3. Connect to your form's state management
4. Test with and without internet to verify fallback works
5. Add map integration if needed

---

## 💡 Tips

- The GeoJSON loads **once per component mount** - very efficient
- Search is **real-time** and supports both languages
- **Fallback is automatic** - no user intervention needed
- Districts are **sorted by ID** for consistent ordering
- **Centroid calculation** is automatic from polygon boundaries

---

**That's it!** You now have a complete, reusable district selector that can be transported to any form. 🎉
