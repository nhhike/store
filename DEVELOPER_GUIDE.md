# Developer Guide: Dynamic Gallery System

This project features a 100% client-side, dynamic gallery system that fetches media and metadata directly from Google Drive. This guide explains the architecture, file handling logic, and UI components.

---

## 1. Core Architecture
The gallery follows a "Serverless" client-side model using the **Google Drive API v3**. 

- **Entry Point**: `gallery.html`
- **Authentication**: Uses a restricted API Key (`API_KEY`) for read-only access to public folders.
- **Root Context**: Defined by `ROOT_ID`, which points to the main "Gallery" folder on Google Drive.

---

## 2. Dynamic Initialization
1. **`initGallery()`**: Fills the `#gallery-picker` dropdown by listing all subfolders within the `ROOT_ID`. It filters for `application/vnd.google-apps.folder` and `application/vnd.google-apps.shortcut`.
2. **`loadFolder(folderId)`**: Triggered when a collection is selected. It fetches all files within that folder, including images, videos, `.md` files, and `.gpx`/`.kml` tracks.

---

## 3. Tabbed Content Interface
The description area is a tabbed interface that dynamically organizes non-media files.

### Tab Grouping
Files are split into two blocks:
- **Block 1**: Markdown files (`.md`) for textual content.
- **Block 2**: Map files (`.gpx`, `.kml`) for track visualizations.
- A vertical divider separates the two blocks in the header, and a horizontal divider separates the header from the content.

### Sorting & Naming Logic (`prepareTabData`)
Each block follows strict sorting rules:
1. **Numeric Sorting**: The system checks the first two characters of the filename. If they are digits (e.g., `01`, `1`, `10`), the tab is sorted numerically by that value.
2. **Alphabetic Sorting**: Files without numeric prefixes are sorted alphabetically and placed after the numeric group.
3. **Title Cleaning**: Tab labels are "sanitized" by:
   - Stripping the leading digits and any immediately following spaces.
   - Removing the file extension.
   - Example: `02 The Abenaki History.md` → `The Abenaki History`.

### Expanding/Collapsing Logic
- **Markdown Tabs**: Collapsed to ~5 lines (`140px`) by default with a "Read More" toggle. Uses a CSS gradient fade at the bottom.
- **Map Tabs**: Automatically expanded to full height (`450px`) on load/switch.

---

## 4. Mapping System
Tracks are rendered using **Leaflet.js**:
- **GPX**: Rendered via the `Leaflet-GPX` plugin.
- **KML**: Rendered via `Leaflet-Omnivore`.
- **Topographic Layer**: Uses `OpenTopoMap` tiles for hiking-specific detail.
- **Auto-Fit**: Every time a map tab is switched to, `invalidateSize()` and `fitBounds()` are called to ensure the track is perfectly centered and the map fills the container.

---

## 5. Media Grid & Lightbox
### Thumbnail Grid
- Images are fetched using the high-performance thumbnail endpoint: `https://lh3.googleusercontent.com/d/{fileId}=s400`.
- Aspect ratios are locked to `1:1` squares for a clean "App Card" look.

### Lightbox (Media Modal)
- Uses `history.pushState` on open to allow the **mobile back button** (Android/Apple) to close the modal instead of navigating away.
- **Metadata**: Extracts EXIF data (Aperture, Shutter Speed, ISO, Camera Model) from the `imageMediaMetadata` object provided by the Drive API.
- **Navigation**: Supports keyboard arrows, swipe-like button clicks, and "I" key for toggling stats.

---

## 6. UX Components
- **Jump to Images**: A centered button that uses `scroll-margin-top` on the image grid to land perfectly below the fixed navigation bar.
- **Back to Top**: A floating rounded-square button that appears after scrolling 400px.
- **Glassmorphism**: High use of backdrop-filters, semi-transparent backgrounds, and subtle borders to maintain a premium aesthetic.

---

## 7. Developer Notes & Gotchas
- **CORS**: Requests to `alt=media` endpoints require the file to be shared as "Anyone with the link can view". Viewing via `file://` protocol will break fetch requests; always use a local dev server.
- **API Key**: If the gallery lists folders but fails to load images/text, check the API key's "HTTP Referrer" restrictions in the Google Cloud Console.
- **Responsive Handling**: The grid uses `repeat(auto-fill, minmax(140px, 1fr))` to naturally adapt to all screen sizes without media queries.
