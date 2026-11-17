# Crop Disease Identifier

Take a photo of a plant leaf, identify the disease and get treatment suggestions.

**Status:** Home page + Check Disease page are done. Result and About/Help are
stubbed links so they can be added later without touching the existing code.

---

## 1. Folder structure

```
crop-disease-identifier/
├── index.html              Home page
├── check.html              Check Disease page
├── README.md
├── css/
│   └── style.css           one stylesheet for every page
├── js/
│   ├── main.js             shared: navbar, offline banner, image fallbacks
│   ├── config.js           every tweakable value (URL, sizes, crop list, mock data)
│   ├── api.js              the only file that talks to the backend
│   └── upload.js           Check Disease page logic
└── assets/
    ├── icons/favicon.svg
    └── images/hero-fallback.svg
```

---
## 2. Run it locally

The pages use **ES modules**, and browsers block modules opened straight from a
file (`file://`). Start any tiny static server from inside the project folder:

```bash
# Python (already installed on most systems)
python -m http.server 5500

# or Node
npx serve -l 5500
```

Then open https://crop-disease-identifier.netlify.app .

`npx serve` is the easiest option if you have Node installed.

---

## 3. Turn off mock mode

Open `js/config.js` and change one line:

```js
export const MOCK_MODE = true;   //  ->  false
```

| `MOCK_MODE` | What happens |
| ----------- | ------------ |
| `true`      | No server needed. Realistic fake results are returned after a 1.5 s delay so you can build the UI. |
| `false`     | The page really uploads the photo to your backend. |

---

## 4. Connect the real backend

1. Point `API_BASE_URL` at your server in `js/config.js`:

   ```js
   export const API_BASE_URL = 'https://your-domain.com';
   export const PREDICT_PATH = '/predict';   // final URL: https://your-domain.com/predict
   ```

2. Set `MOCK_MODE = false`.

3. The app sends `multipart/form-data` with two fields:

   | Field    | Type   | Notes                              |
   | -------- | ------ | ---------------------------------- |
   | `image`  | File   | JPEG, already compressed client-side |
   | `crop`   | string | e.g. `Tomato`                      |

4. Expected JSON reply:

   ```json
   {
     "crop": "Tomato",
     "disease": "Early Blight",
     "scientific_name": "Alternaria solani",
     "confidence": 0.92,
     "symptoms": ["Dark brown spots with concentric rings"],
     "natural_treatment": ["Remove and destroy infected leaves"],
     "chemical_treatment": ["Mancozeb 75% WP @ 2.5 g per litre"]
   }
   ```

5. **CORS:** if the frontend and the API are on different origins, allow the
   page's origin on the server, otherwise `fetch()` fails and the UI shows
   "Cannot reach the server".

Request timeout is `REQUEST_TIMEOUT_MS` (20 s) in `config.js`.

---

## 5. How the flow works

1. `check.html` → user picks a file (button, camera, or drag & drop).
2. `upload.js` validates it: **JPG/PNG only, max 5 MB**. Errors appear inline
   under the drop zone, never in an `alert()`.
3. The image is compressed in the browser (canvas, max 800 px, JPEG 0.7) —
   a 4 MB phone photo becomes roughly 80 KB, which matters on slow networks.
4. "Check Disease" stays disabled until **both** an image and a crop are chosen.
5. On submit, everything is disabled and a spinner shows *Checking your leaf...*.
6. `api.js` returns the result, `upload.js` stores it in `sessionStorage`
   (`cdi.result`, `cdi.preview`, `cdi.crop`) and redirects to `result.html`.

### Building `result.html` later

`main.js` has a page registry — add one line:

```js
const PAGES = {
  check:  () => import('./upload.js').then((m) => m.initCheckPage),
  result: () => import('./result.js').then((m) => m.initResultPage),  // add this
};
```
Then in `result.js`:

```js
import { STORAGE } from './config.js';

export function initResultPage() {
  const raw = sessionStorage.getItem(STORAGE.RESULT);
  if (!raw) { window.location.replace('check.html'); return; }  // nothing to show

  const result = JSON.parse(raw);
  const preview = sessionStorage.getItem(STORAGE.PREVIEW);
  // ... render result.disease, result.confidence, result.symptoms, etc.
}
```

## 6. Notes

- **Mobile first.** Tested layout range: 360 px → desktop. Single stylesheet,
  no CSS framework, one web font (with a system-font fallback if Google Fonts is
  blocked).
- **Accessibility:** semantic landmarks, `alt` text, visible focus rings, all
  buttons at least 44 px tall, `aria-live` regions for loading and error text.
- **Offline:** a banner appears automatically whenever `navigator.onLine` is
  false.
- **Images:** the hero photo is lazy-loaded and has a green-gradient
  background plus a local SVG fallback, so the page still looks right with no
  internet.


## 7. Safety

The treatment lists are guidance, not a prescription. Always follow the
product label, wear protective gear, and respect pre-harvest intervals.
