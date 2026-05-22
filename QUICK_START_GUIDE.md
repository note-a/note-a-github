# Portfolio Editing System - Quick Start Guide

## 🚀 Quick Start (5 minutes)

### Step 1: Open Admin Panel
- Open your browser and go to: `archive-admin.html`
- Click on **"포트폴리오 편집"** in the left menu

### Step 2: Edit Your Content
Choose what you want to edit by clicking on a tab:

| Tab | What You Can Edit |
|-----|------------------|
| **PROJECT** | 4 project slides (title, category, author, year, content) |
| **INCLUSION** | Full text content for the Inclusion section |
| **INTRO** | Full text content for the Intro section |
| **RULE** | 5 fundamental rules (each with title, author, year, category, status, description) |
| **REFERENCE** | 8 reference items (books/papers with full details) |
| **VIDEO** | Full text content for the Video section |
| **ARCHIVE** | 3 archive photos (title, filename, keywords, description) |

### Step 3: Make Changes
- Edit the form fields directly
- For text sections: Type in the textarea
- For slides/items: Edit each field individually
- Keywords in archive photos: Separate with commas

### Step 4: Save
- Click the **SAVE** button (green button at bottom)
- You'll see a confirmation message
- Your changes are now saved!

### Step 5: View Changes
- Go to `portfolio.html`
- Refresh the page (Cmd+R on Mac, Ctrl+R on Windows)
- Your new content will appear!

---

## 📝 Example: Editing a Project Slide

1. Open admin panel → Click "포트폴리오 편집"
2. Click **PROJECT** tab
3. You see: **SLIDE 1 / 4**
4. Edit the fields:
   - **TITLE**: Change "Project SET Overview" to your new title
   - **CATEGORY**: Change "Concept" to your category
   - **AUTHOR**: Update author name
   - **YEAR**: Update year
   - **CONTENT TEXT**: Write or paste your new content
5. Click **SAVE**
6. Go to `portfolio.html` and refresh
7. Your new project slide content appears!

---

## 📚 Example: Editing References

1. Open admin panel → Click "포트폴리오 편집"
2. Click **REFERENCE** tab
3. You see: **ITEM 1** (Camera Lucida, Roland Barthes)
4. Edit the fields for this item or scroll to edit others
5. Fields available:
   - **TITLE**: Book/paper title
   - **AUTHOR**: Author name
   - **YEAR**: Publication year
   - **CATEGORY**: Category type
   - **STATUS**: Status (FOUNDATIONAL, CONTEMPORARY, etc.)
   - **PREVIEW / DESCRIPTION**: Write a description
6. Click **SAVE**
7. Your updated references appear in portfolio.html!

---

## 🔄 Resetting to Defaults

If you want to go back to the original content:

1. Click the **RESET** button (red button at bottom)
2. Confirm in the popup: "모든 변경사항을 초기화하시겠습니까?"
3. All your edits are cleared
4. Refresh `portfolio.html` to see original content

**Warning**: This cannot be undone! Make sure you want to reset before clicking.

---

## ❓ Troubleshooting

### "My changes don't appear in portfolio.html"
- ✓ Did you click SAVE in the admin panel?
- ✓ Did you see the confirmation message?
- ✓ Did you refresh portfolio.html? (Cmd+Shift+R for hard refresh)

### "I can't find the portfolio edit section"
- ✓ Make sure you're on `archive-admin.html`, not `portfolio.html`
- ✓ Look for **"포트폴리오 편집"** in the left menu
- ✓ Click it if it's not already highlighted

### "I accidentally reset everything!"
- Unfortunately, there's no undo after reset
- But you can re-enter the content manually through the forms
- The default content is preserved in the code - you can also check IMPLEMENTATION_SUMMARY.md for default values

### "Changes are gone after I closed the browser"
- This is normal - changes are stored locally
- If you cleared browser data/history, saved changes are deleted
- Just re-edit the content and save again

---

## 📖 Content Reference

Here's what gets edited in each section:

### PROJECT Section
- **4 Slides** about the SET project
- Each has: Title, Category, Author, Year, Long text content
- Edit to describe your project's philosophy and structure

### INCLUSION Section
- **Single text block** about SET inclusion
- Edit the entire text to explain inclusion concepts

### INTRO Section
- **Single text block** introduction to SET
- Edit to provide navigation instructions and key concepts

### RULE Section
- **5 Fundamental Rules**
- Each rule has: Title, Author, Year, Category, Status, Description
- Edit rules and descriptions to match your framework

### REFERENCE Section
- **8 Reference Items** (books, papers, theoretical works)
- Each has: Title, Author, Year, Category, Status, Description
- Add or modify references that influenced your work

### VIDEO Section
- **Single text block** about video archive
- Edit to describe video documentation system and access

### ARCHIVE Section
- **3 Archive Photos**: CRASH, FLOATING, SHADOW
- Each has: ID, Title, Filename, Keywords, Description
- Edit titles, keywords, and descriptions for photo search

---

## 💡 Tips

1. **Use meaningful keywords** in archive photos so they're searchable
2. **Keep formatting simple** - the system handles basic text
3. **Back up important content** - copy your text to a document before major edits
4. **Test changes** - after saving, always check portfolio.html to verify
5. **Use consistent formatting** - keep similar items in consistent style

---

## 🔗 Links

- **Admin Panel**: `archive-admin.html`
- **Portfolio View**: `portfolio.html`
- **Full Documentation**: `PORTFOLIO_EDITING_SYSTEM.md`
- **Implementation Details**: `IMPLEMENTATION_SUMMARY.md`

---

**Ready to edit your portfolio! 🎉**
