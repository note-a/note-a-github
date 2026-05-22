# Portfolio Content Editing System - Complete Documentation

## Overview

The portfolio content editing system allows you to edit all portfolio content directly from the admin panel without modifying the HTML source code. All changes are saved to the browser's localStorage and automatically applied when portfolio.html loads.

## System Architecture

### Files Involved
1. **archive-admin.html** - Admin panel with portfolio editing interface
2. **portfolio.html** - Main portfolio page that loads saved content
3. **localStorage** - Browser's local storage (key: `adminContents`)

### Data Flow

```
archive-admin.html (Edit) 
    ↓ (SAVE button)
    → localStorage['adminContents']
    ↓ (portfolio.html loads)
    → loadAdminContentsFromStorage()
    → Merge with default contents
    → Display updated content
```

## Features

### 1. Portfolio Editing Interface

Located in the admin panel at `archive-admin.html`, accessible via the **"포트폴리오 편집"** menu item.

#### 7 Editable Sections

| Section | Type | Content | Editable Fields |
|---------|------|---------|-----------------|
| **PROJECT** | Slides | 4 project slides | Title, Category, Author, Year, Content Text |
| **INCLUSION** | Text | Single text block | Full text content |
| **INTRO** | Text | Single text block | Full text content |
| **RULE** | Items List | 5 fundamental rules | Title, Author, Year, Category, Status, Preview |
| **REFERENCE** | Items List | 8 reference items | Title, Author, Year, Category, Status, Preview |
| **VIDEO** | Text | Single text block | Full text content |
| **ARCHIVE** | Photos | 3 archive photos | ID, Title, Filename, Keywords, Description |

### 2. Tabs Interface

Each section can be accessed via tabs at the top of the editing panel:
- PROJECT | INCLUSION | INTRO | RULE | REFERENCE | VIDEO | ARCHIVE

### 3. Save & Reset Functions

- **SAVE Button** (Green) - Saves all changes to localStorage
- **RESET Button** (Red) - Clears all changes and resets to default content

## How to Use

### Editing Portfolio Content

1. **Open Archive Admin Panel**
   - Navigate to `archive-admin.html`
   - Click on **"포트폴리오 편집"** menu item

2. **Select Section**
   - Click on the section tab you want to edit (PROJECT, INCLUSION, etc.)

3. **Edit Content**
   - Fill in the form fields with your new content
   - For slides and items: Edit each slide/item individually
   - For text sections: Edit the full text in the textarea

4. **Save Changes**
   - Click the **SAVE** button at the bottom
   - A confirmation alert will appear
   - Changes are now saved to localStorage

5. **View Changes**
   - Go to `portfolio.html`
   - Refresh the page to load the saved content
   - Your edited content should now appear

### Resetting Changes

- Click the **RESET** button to clear all saved changes
- Confirm in the popup dialog
- All content will revert to defaults
- Refresh `portfolio.html` to see the changes

## Technical Implementation Details

### Archive-Admin.html Functions

#### Form Generation Functions
```javascript
generateProjectSlidesForm()      // Generates 4 project slide forms
generateTextForm(section)        // Generates text editing form
generateItemsListForm(section)   // Generates list item forms
generateArchivePhotosForm()      // Generates archive photo forms
```

#### Save/Load Functions
```javascript
loadPortfolioData()              // Loads saved data from localStorage
loadPortfolioEditForm(subsection) // Loads specific section form
savePortfolioChanges()           // Saves all form data to localStorage
resetPortfolioChanges()          // Clears all saved changes
```

#### Form Selectors
The system uses specific CSS classes to collect form data:
- `.slide-title-${idx}` - Project slide titles
- `.portfolio-text-${section}` - Text content
- `.item-title-${section}-${idx}` - Item titles
- `.photo-title-${idx}` - Archive photo titles

### Portfolio.html Functions

#### Load Function
```javascript
loadAdminContentsFromStorage()   // Called on page load
```

This function:
1. Retrieves `adminContents` from localStorage
2. Merges it with the default `contents` object
3. Applies changes to all 7 sections:
   - project, inclusion, intro, rule, reference, video, archive

#### Initialization
```javascript
document.addEventListener('DOMContentLoaded', function() {
  loadAdminContentsFromStorage();  // Load saved content first
  detectAndSwitchLayout();         // Then setup layout
  // ... rest of initialization
});
```

## localStorage Data Structure

Saved content is stored as JSON under the key `adminContents`:

```json
{
  "project": {
    "slides": [
      {
        "id": "slide_1",
        "title": "Your Title",
        "category": "Category",
        "author": "Author Name",
        "year": "2026",
        "text": "Content..."
      }
    ]
  },
  "inclusion": {
    "text": "Text content..."
  },
  "intro": {
    "text": "Text content..."
  },
  "rule": {
    "items": [
      {
        "id": "RULE_1",
        "title": "Rule Title",
        "author": "Author",
        "year": "2026",
        "category": "Category",
        "status": "STATUS",
        "preview": "Description..."
      }
    ]
  },
  "reference": {
    "items": [
      {
        "id": "REF_1",
        "title": "Reference Title",
        "author": "Author",
        "year": "2026",
        "category": "Category",
        "status": "STATUS",
        "preview": "Description..."
      }
    ]
  },
  "video": {
    "text": "Text content..."
  },
  "archive": {
    "photos": [
      {
        "id": "PHOTO_1",
        "title": "Photo Title",
        "filename": "photo.svg",
        "keywords": ["keyword1", "keyword2"],
        "description": "Description..."
      }
    ]
  }
}
```

## Troubleshooting

### Changes Not Appearing

1. **Verify Save Completed**
   - Check if the alert appeared after clicking SAVE
   - If no alert, check browser console for errors

2. **Check localStorage**
   - Open browser DevTools (F12)
   - Go to Storage → Local Storage
   - Look for `adminContents` key
   - Verify data is there

3. **Refresh Portfolio Page**
   - Changes only apply after portfolio.html is refreshed
   - Try hard refresh (Cmd+Shift+R on Mac, Ctrl+Shift+R on Windows)

4. **Clear Browser Cache**
   - If still not working, clear cache and reload

### Reset Not Working

1. Check if you confirmed the confirmation dialog
2. Try refreshing `portfolio.html` after reset
3. Check that localStorage was actually cleared in DevTools

## Default Content

Each section has default content hardcoded in the form generation functions:

- **PROJECT**: 4 slides about SET concept (Overview, Core Concept, Measurement Paradox, Structure)
- **RULE**: 5 fundamental rules (Observer Is Observed, Incompleteness Is Completeness, Voice Must Merge, Language and Vision, Do Not Disconnect)
- **REFERENCE**: 8 key references (Camera Lucida, On Photography, Discipline and Punish, etc.)
- **ARCHIVE**: 3 photos (CRASH, FLOATING, SHADOW)
- **INCLUSION, INTRO, VIDEO**: Predefined text blocks

You can edit any of these defaults through the admin panel.

## Limitations

- Changes are stored only in browser localStorage (not synchronized across devices)
- Clearing browser data will erase all saved changes
- Maximum content size limited by browser localStorage (typically 5-10MB per domain)

## Future Enhancements

Potential improvements:
1. Cloud backup of content
2. Version history of changes
3. Export/import functionality
4. Multi-user editing with conflict resolution
5. Scheduled publishing

---

**Last Updated**: 2026-05-21
**System Status**: ✓ Complete and Functional
