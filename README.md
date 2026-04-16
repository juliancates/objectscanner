# Object Scanner — Perspective Correction → SVG Trace

A browser-based tool that photographs an object placed on a printed reference mat, automatically corrects the perspective from any shooting angle, and produces a clean scalable vector graphic (SVG) of the object's outline at exact physical dimensions.

No installation, no server, no account. Open `scanner.html` in any modern browser and it works entirely offline after the first load.

---

## How it works

### 1. Print the mat

Download [`scan-mat-v2-usletter.pdf`](scan-mat-v2-usletter.pdf) and print it at **exactly 100% scale** — no "fit to page", no scaling. The PDF is authored at precise physical dimensions so the geometry is exact.

The mat is US Letter (8.5 × 11 in) and contains:

- **70 fiducial dots** — filled black circles at known physical positions (0.42 in inset, 0.5 in spacing, 0.18 in radius) forming a border around the scanning area
- **¼ in grid** with 1 in major lines and inch labels for reference
- A dashed inner border marking the usable scanning area

### 2. Photograph

Place your object anywhere inside the dot border. Photograph it from any angle — the dots provide enough reference geometry to correct steep perspective. Ensure:

- All four sides of the dot border are visible in the frame
- Good even lighting (shadows reduce detection accuracy)
- The paper is brighter than the background

### 3. Detect & trace

Upload the photo. The app automatically:

1. **Detects the fiducial dots** — finds dark circular blobs on the paper using three thresholding strategies, filters by size consistency, and matches candidates to the known dot grid
2. **Computes a homography** — a pure-JS RANSAC implementation fits a 3×3 perspective transform from matched dot positions to their known physical coordinates, automatically rejecting outliers
3. **Warps the image** — reprojects to a flat 4× resolution view (3264 × 4224 px, 384 dpi) using Lanczos4 interpolation
4. **Erases the dot border** — all 70 dot positions are painted white using their exact physical coordinates before tracing
5. **Traces to SVG** — CLAHE contrast normalisation, Otsu thresholding, morphological cleanup, contour finding with hierarchy (for holes), and Catmull-Rom bezier smoothing produce a solid-fill SVG

The output SVG has exact physical dimensions derived from the homography.

---

## Files

| File | Description |
|------|-------------|
| `scanner.html` | The complete application — single self-contained HTML file |
| `scan-mat-v2-usletter.pdf` | Printable reference mat — US Letter, print at 100% |

---

## Usage

```
1. Print scan-mat-v2-usletter.pdf at 100% scale (no fit-to-page)
2. Place object on mat inside the dot border
3. Open scanner.html in a browser
4. Upload a photo of the object on the mat
5. Wait for automatic dot detection (~2s after OpenCV loads)
6. Click "Run / Re-trace"
7. Download SVG
```

OpenCV.js (~8 MB) is loaded from a CDN on first use and cached by the browser. Subsequent uses work offline.

---

## Trace settings

| Setting | Description |
|---------|-------------|
| **Threshold** | Otsu auto (recommended for most objects), Adaptive (better for uneven lighting), Fixed (manual control) |
| **Smooth** | Catmull-Rom bezier tension — 0 = straight lines, 1 = maximum smoothing |
| **Simplify** | Ramer–Douglas–Peucker epsilon — reduces path complexity |
| **Min area** | Minimum contour area in px² — filters out noise fragments |

---

## Technical notes

**Homography** — `cv.findHomography` is not available in the opencv.js 1.2.1 build used here. The perspective transform is computed with a pure-JavaScript normalised DLT algorithm with RANSAC, implemented from scratch.

**Physical accuracy** — The mat PDF is authored with ReportLab in points (1/72 in), not pixels, so dot positions are not subject to screen DPI rounding. The SVG output uses `width` and `height` in inches derived from the homography, giving sub-millimetre dimensional accuracy when printed.

**Dot redundancy** — With 70 dots, the system tolerates losing ~25 dots to hand occlusion, shadows, or frame edges and still produces a robust homography. The minimum for a valid solve is 6 matched dots.

---

## Browser requirements

Any modern browser with WebAssembly support (Chrome, Firefox, Safari, Edge — all 2019+). No extensions or special permissions required.

---

## License

MIT
