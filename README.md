# Open Laser Target Library

[Browse the target library](https://makido.github.io/open-laser-targets/)

Fifteen Swiss and NRA laser-practice target editions with OLTS 0.5 definitions.
Each individual PDF contains a target and its scoring companion. The collection
PDF contains the fifteen target pages only.

The catalogue works directly from disk and on GitHub Pages. It offers English,
German or system language, plus bright, dark or system appearance. Explicit
preferences are saved locally in the browser when storage is available.

## Printing

Download a PDF rather than printing the web page. Use actual size (100%), centre
it on A4 or US Letter, and measure both calibration strips. Swiss editions use
50 mm strips; NRA editions use 2-inch strips. Use the practice distance printed
on the selected target. Readers must support OLTS 0.5.

## Sources and verification

See [dimensions, geometry notes and source references](reference.html).
Traced silhouettes are identified in the catalogue; NRA sheets are practice
reproductions. Physical printing and camera/app testing remain necessary.

## Hosting

This repository contains the generated static website. GitHub Pages serves the
root of the main branch. `.nojekyll` keeps the files unchanged. There are no
server-side dependencies, installation steps, external fonts or analytics.

To view offline, download the repository and open `index.html` in a browser.
The target QR landing service on doxyo.com is separate from this catalogue.
