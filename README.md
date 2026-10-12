# Aperture

A large, image-first [Masthead](https://masthead.site) theme for photographers.
**One post is one photograph.** Every photo lands in one big mosaic, and tags
filter it.

## Setup

Until you choose a homepage, the site shows a short setup guide (that's
`index.liquid`).

1. **Add photos.** Make one post per photograph and fill in its post options
   (see the table below). Tag each post, because tags drive the gallery filter.
2. **Gallery.** Go to New page → Theme page → **Gallery**, with the slug
   `gallery`. With any other slug, set the **Gallery URL** setting to match.
3. **Home.** Go to New page → Theme page → **Home** and pick *Full frame* or
   *Contact sheet*.
4. **Make it the homepage.** Either switch on the page's own homepage option,
   or pick it under Settings → Homepage.

## Post options

| Option | What it does |
|---|---|
| Photo | The photograph. Posts without one stay out of the gallery. |
| Alt text | For screen readers. Falls back to the title. |
| Tile size | `auto` (picked from a rhythm), `small`, `wide`, `tall`, `large`, `panorama` |
| Crop focus | Which part survives the tile crop: center, top, bottom, left, right |
| Camera, Location | Free text, shown under the enlarged photo |

The post page shows the photo uncropped (up to 90% of the viewport height),
then the title, meta and tags, then the post body. It ends with
previous/next links within the post's **first tag**.

## Gallery

A dense CSS grid. Each row is ¾ of a column wide, so the tile shapes come out
as `small` 4:3, `tall` 2:3, `large` 4:3, `wide` 8:3 and `panorama` the full
row. It drops to 3, 2 and 1 columns as the screen narrows. The tag chips link
to `?tag=<slug>`. A ~10-line inline script hides the non-matching tiles,
because Masthead only filters `?tag=` on the server for the index. Without JS,
every photo shows.

## Home variants

Both pull photos from posts tagged with **Featured tag** (default `Featured`).
If nothing is tagged, they use the latest photos.

- **Full frame**: a full-viewport hero showing the *Hero image* upload, or the
  newest featured photo. Your name and tagline sit over it, and a "Selected
  work" mosaic follows.
- **Contact sheet**: big type, an intro and a slowly drifting CSS film strip,
  followed by frame-numbered thumbnails (`01`, `02`…).

## Settings

- **Navigation**: `top` bar or `sidebar`. Both become a top bar with a menu on
  phones.
- **Sidebar text**: a short note at the bottom of the sidebar.
- **Gallery URL**: where "View all", tag links and "Back to the gallery" point.
  It must start with `/`.
- **Gallery**: gap, desktop columns (2–6) and hover titles.
- **Colour scheme**: `system` follows the visitor; `light` or `dark` locks it.
- **Appearance**: accent colour and favicon.

## Maintainer notes

- `pages/gallery.liquid` and the Full-frame home both render mosaic tiles.
  There are no partials, so the markup is duplicated; keep the two in sync.
- The option is called `tile_size` and the loop variable `shape`. A Liquid
  variable named `size` rendered as `2` in the preview.
- `posts` is empty on post renders, but `posts_by_tag` is not. That's why the
  pager is per-tag.
- Preview: `masthead preview`. The preview content lives in `preview/`, and the
  homepage is left unset so the guide shows. The home variants are at `/home`
  and `/home-sheet`.
