# Logos

Central index of every Piper project logo. Each folder holds a copy of the
approved master artwork. The **master (source of truth)** column points to where
each logo is edited; the **used in** column lists where a copy is served or
embedded, so the index stays accurate as sites and apps change.

Source paths are relative to `~/code`.

| Project | Logo | Master (source of truth) | Used in |
|---|---|---|---|
| Recipes | `logos/recipes/little-simmer-master.png` | `shopping-recipes/brand/recipes/little-simmer-master.png` | Portal + portfolio cards, iOS/macOS app icon |
| JoshuaPiper.dev | `logos/joshuapiper/terminal-pal-master.png` | `shopping-recipes/brand/joshuapiper/terminal-pal-master.png` | Portfolio site, portal card |
| YoMamma | `logos/yomamma/yomamma-master.svg` | `shopping-recipes/brand/yomamma/yomamma-master.svg` | Portal card, YoMamma web app |
| FML — ForMyLife (budget) | `logos/formylife/fml-mark.svg` + `fml-512.png` | `formylife/web/src/assets/fml-mark.svg` | budget.piper.sydney app icon |
| MindCanvas Studio | `logos/mindcanvas/tangent-logo.jpg` | `mindcanvas-studio/src/assets/tangent_logo_paper_pencil_tile.jpg` | mindcanvas.piper.sydney app icon |
| Snapmark (Screeny) | `logos/snapmark/screeny-icon.png` | `snapmark-macos/Resources/ScreenyIcon.png` | macOS app icon + marketing page |
| Piper Group Sydney | `logos/piper-group/piper-group-sydney-logo.svg` | `piper-group-sydney-logo.svg` (draft SVG) | Umbrella brand mark; profile README embeds the emblem crop below |

## Derived icons (profile README)

Small theme-neutral derivatives so the profile README's 28px table icons have
no white glare in dark mode. The masters above stay the source of truth;
regenerate a derivative if its master changes.

| Icon | Derived from | Used in |
|---|---|---|
| `logos/yomamma/yomamma-icon.svg` | `logos/yomamma/yomamma-master.svg` (rounded-tile copy, rx=112) | Profile README app table |
| `logos/mindcanvas/tangent-icon.png` | `logos/mindcanvas/tangent-logo.jpg` (rounded-corner 112px copy) | Profile README app table |
| `logos/recipes/little-simmer-icon.png` | `logos/recipes/little-simmer-master.png` (rounded-corner 112px copy) | Profile README app table |
| `logos/snapmark/snapmark-icon.png` | `logos/snapmark/screeny-icon.png` (rounded-corner 112px copy) | Profile README app table |
| `logos/piper-group/piper-group-mark.svg` | `logos/piper-group/piper-group-sydney-logo.svg` (square emblem crop, viewBox `160 170 540 540`) | Profile README web & self-hosted table |

## Notes

- The Recipes, JoshuaPiper.dev and YoMamma masters are also committed in
  `shopping-recipes/brand/` and checked by `shopping-recipes/scripts/check-web.py`.
  Keep the copies here in sync when a master changes.
- `logos/piper-group/piper-group-sydney-logo.svg` is a hand-authored SVG draft
  (orange theme, Sydney Opera House + bagpiper). No raster master yet.
- Generated-icon artwork is original concept work; no other brand's logo is used.
