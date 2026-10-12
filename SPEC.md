# Aperture: theme spec

A large, image-first Masthead theme for photographers. **One post = one
photograph.** Each photo lives in a post option. The post body is the story
behind the shot. Every photo appears in one big, dynamic gallery, and you
filter it by tag.

Status: **built (v1.0.0).** All open questions are resolved; see Decisions at
the bottom. Changes made during the build are noted inline.

## Goals

- **One post = one photo.** The photo, its alt text and its preferred tile size
  are post options. The body is optional text that sits under the enlarged
  photo.
- **One big gallery of everything.** All posts show as a dense mosaic with
  tiles in different sizes. No hand-curated lists per page. **Tags** do the
  filtering.
- **A snazzy homepage** with 2 variants, chosen by a dropdown on one `Home`
  theme page.
- **Nav position is a setting:** top bar or sidebar.

## Non-goals (v1)

- Hand-picked photo lists on pages (Halide's `Gallery` page model). Tags
  replace them.
- EXIF read from the file. Masthead passes only a URL, so it can't. The
  `camera` and `location` post options are typed in by hand.
- Image resizing and `srcset`. The platform serves the original upload, so
  photographers should upload web-sized JPEGs (about 2400px on the long edge).
- A JS lightbox and slideshow. The enlarged view is the post page.
- Pagination, search UI and comments.

## Platform facts this relies on

Verified against `masthead` CLI 0.4.1 and the `documentation/` and `broadsheet/`
themes.

- `render_version: "v1"` gives `post.post_options.<key>` and
  `page.page_options.<key>`, and post options are present on **every post in
  `posts`**. That is what lets the gallery read each photo and size.
- **`posts` is empty in `post.liquid`** in CLI 0.4.1, which claims to copy the
  production route table. `posts_by_tag["slug"]` *is* available there. So
  prev/next pages through the post's **first tag** (build change).
- `tags` (`name`, `slug`, `active`) reaches the index and theme pages.
- **Server-side `?tag=` filtering only happens on `index`.** On a custom theme
  page the CLI passes `tags` but ignores `?tag=` (`current_tag` stays empty and
  `posts` is unfiltered). Tested in a throwaway theme.
  → The gallery lives on a theme page, so it filters on the client (see
  Filtering).
- No image dimensions reach the template, so tile shapes are **cropped**
  (`object-fit: cover`). The full, uncropped photo is shown on the post page.
- List tokens are allowed (Broadsheet ships a `nav` list token). Not needed
  here.
- No partials, so the mosaic markup is duplicated in `pages/gallery.liquid`
  and the Full-frame home strip.

## Information architecture

| Route | Template | What it is |
|---|---|---|
| `/` (default) | `index.liquid` | **Setup guide.** No posts are rendered. It explains how the theme works and how to replace this page (below). |
| `/` (when Home is set as homepage) | `pages/home.liquid` | Snazzy homepage, variant A or B |
| `/gallery` (any slug) | `pages/gallery.liquid` | **The** gallery of all photos. It is the only place the full mosaic lives. |
| `/<post>` | `post.liquid` | Enlarged photo, then title and meta, then body, then prev/next |
| `/<page>` | `page.liquid` | Prose page (About, Contact) |

### Default index: setup guide

`index.liquid` is what a fresh site shows before the owner sets a homepage. It
renders inside the normal layout (nav included), so the owner sees the theme's
look straight away. It shows no posts, only a short guide:

1. **Add photos.** Make one post per photograph. Set **Photo**, **Alt text**
   and optionally **Tile size**, **Crop focus**, **Camera** and **Location**
   under Post options. Tag each post (`Landscape`, `Street`…), because tags
   drive the gallery filter.
2. **Create the gallery.** Go to New page → Theme page → **Gallery** and give
   it a slug such as `gallery`. If the slug isn't `gallery`, set the **Gallery
   URL** setting to match.
3. **Create the homepage.** Go to New page → Theme page → **Home** and pick
   *Full frame* or *Contact sheet*. Tag photos `Featured` to put them on the
   home page.
4. **Replace this page.** There are two ways, and either works. You can open
   the Home page (or the Gallery page) in the page editor and switch on its
   homepage option. Or you can choose that page under Settings → Homepage.
   This guide then disappears.
5. **Tune the look.** In Settings → Theme, set the nav position (top bar or
   sidebar), the colour scheme (system, light or dark), the accent, and the
   gallery gap and columns.

The guide is static copy in the template, with no tokens. `/search` also
renders `index`, but Aperture has no search UI, so the route is left as is.

## Gallery

- A CSS grid with `grid-auto-flow: dense` and square-ish base cells. Columns
  drop from 4 → 3 → 2 by width (1 column under 520px, where every tile is
  full width).
- **Tile size** comes from the post option `tile_size` (named so it can't
  clash with Liquid's `size`):

  | `tile_size` | Span (cols × rows) |
  |---|---|
  | `auto` (default) | picked from a fixed rhythm by position, e.g. L, S, S, W, S, T, S, S, … |
  | `small` | 1 × 1 |
  | `wide` | 2 × 1 |
  | `tall` | 1 × 2 |
  | `large` | 2 × 2 |
  | `panorama` | full row × 1 |

  With `auto`, the gallery looks varied with zero effort. Setting a size
  overrides it for that one photo. Dense packing fills the holes.
- **Crop position** comes from the post option `focus` (center, top, bottom,
  left, right), which maps to `object-position`.
- **Tile hover/focus:** the photo scales slightly and the title fades in at the
  bottom-left. On touch screens the title isn't shown and the tap goes straight
  to the post.
- Posts **without a photo** are skipped in the gallery. They still exist as
  posts.
- `loading="lazy"` on every tile except the first 4.
- A setting for the gap between tiles (`gap` token).

### Filtering

- A row of chips above the gallery: `All · Landscape · Street · Portrait …`
  from the site's `tags`. The Gallery page's `show_filter` option hides the
  row.
- Chips are links to `?tag=<slug>`. A small inline script (about 15 lines)
  reads `?tag=`, marks the active chip and hides the tiles without that tag.
  Each tile carries `data-tags="|slug|slug|"`. Without JS, every photo shows.
  The URL is shareable.
- Tags on the post page link to `<gallery_url>?tag=<slug>`.

## Post page (the enlarged photo)

1. **The photo**, uncropped, as large as the viewport allows: `max-height:
   90vh`, centred, on the page background. Clicking it opens the original file
   in a new tab.
2. **Title**, date and tags (tags link to the filtered gallery), plus the
   optional camera and location line.
3. **`body_html`**, in a readable column under the photo.
4. **← Previous / Next →** thumbnails within the post's first tag ("Next in
   Portrait →"), from `posts_by_tag`. A post with no tags has no pager.
   ← back to the gallery.

`og:image` is set from the photo.

## Homepage (`pages/home.liquid`, `variant` dropdown)

Both variants pull their photos from tags, the same way Broadsheet's lead
does, so promoting a photo means tagging it. The hero photo falls back from
the `hero_image` upload, to the newest post tagged `hero_tag` (default
`Featured`), to the newest post.

### Variant A: "Full frame"

- A hero photo bleeding the full viewport (100svh) with a soft gradient, the
  site name and tagline set large at the bottom-left, and a ↓ scroll cue.
- Under it, a "Selected work" strip: the latest N posts tagged `hero_tag` in
  the gallery mosaic, then a "View all work →" button to `gallery_url`.

### Variant B: "Contact sheet"

- A big typographic intro: the site name in a large display size, then the
  tagline and a short intro paragraph. Next to it on desktop, a slowly drifting
  **film strip** of featured photos, pure CSS (`@keyframes` translate, paused
  on hover, static under `prefers-reduced-motion`).
- Under it, frame-numbered thumbnails (`01`, `02` …) in an even grid, like a
  contact sheet. Each links to its post, and the section ends with "View all
  work →".

### Home page options

| key | type | default | notes |
|---|---|---|---|
| `variant` | select `full-frame` / `contact-sheet` | `full-frame` | |
| `tagline` | string | `""` | falls back to `site.description` |
| `intro` | text | `""` | B only |
| `hero_image` | file | `""` | overrides the tag pick (A) |
| `hero_tag` | string | `Featured` | slugged before matching, same as Broadsheet |
| `count` | number | `8` | photos in the strip or sheet |
| `cta_label` | string | `View all work` | |

## Navigation

The `nav_position` token chooses the layout.

- **`top`:** a slim bar with the site name on the left and `pages` on the
  right. It's transparent over the Full-frame hero and solid everywhere else.
- **`sidebar`:** a fixed left column (about 240px) with the site name, `pages`
  stacked, and an optional `sidebar_text` at the bottom. The gallery fills the
  rest.
- **Mobile (< 900px):** both become a top bar. The pages sit behind a
  CSS-only menu toggle (`<details>`).
- A `show_nav` page option per page, so a page can hide the chrome.

## Manifest

`name: "Aperture"`, `slug: "aperture"`, `version: "1.0.0"`,
`render_version: "v1"`.

### Post options

| key | type | default | notes |
|---|---|---|---|
| `photo` | file | `""` | The photograph. Empty means the post isn't in the gallery. |
| `alt` | string | `""` | Alt text. Falls back to the title. |
| `tile_size` | select `auto/small/wide/tall/large/panorama` | `auto` | Preferred tile size |
| `focus` | select `center/top/bottom/left/right` | `center` | Crop anchor in tiles |
| `camera` | string | `""` | e.g. `Leica M6 · 35mm · f/8 · 1/250 · Portra 400` |
| `location` | string | `""` | |

### Page options (ordinary pages)

| key | type | default |
|---|---|---|
| `show_nav` | boolean | `true` |

### Theme pages

- `home`: the options above.
- `gallery`: `intro` (text), `show_filter` (boolean).

### Tokens

| key | type | default | category |
|---|---|---|---|
| `nav_position` | select `top/sidebar` | `top` | Layout |
| `sidebar_text` | text | `""` | Layout |
| `gallery_url` | string | `/gallery` | Layout: where "View all", tag links and the post's "back to gallery" go |
| `gap` | length | `6px` | Gallery |
| `columns` | number | `4` | Gallery: desktop maximum |
| `show_titles` | boolean | `true` | Gallery: hover titles |
| `color_scheme` | select `system/light/dark` | `system` | Appearance |
| `accent` | color | `#e8b04a` | Appearance |
| `favicon` | file | `""` | Appearance |

The theme ships a light palette and a dark palette in `theme.css`. `muted` and
`rule` tones are derived from each with `color-mix`.

## Typography and colour

- With `color_scheme: system`, the palette follows `prefers-color-scheme`
  (pure CSS). With `light` or `dark`, the theme is locked to that palette
  through a `data-scheme` attribute on `<html>`. There is no visitor toggle.
- Light is off-white `#f6f5f2` / ink `#141414`. Dark is `#0b0b0c` / `#f2f0ec`.
  Exact tones get tuned during the build.
- Display face: a system grotesk stack set large and tight. Body: the system
  sans stack. No web fonts and no third-party requests.
- Visible focus rings, AA contrast, `prefers-reduced-motion` respected, and a
  skip link.

## Files

```
aperture/
  SPEC.md
  manifest.json
  theme.css
  templates/
    layout.liquid        head/og, nav (top|sidebar), shell
    index.liquid         setup guide (static, no posts)
    post.liquid          enlarged photo, meta, body, prev/next
    page.liquid          prose page
    not_found.liquid
    pages/home.liquid    + home.json    (2 variants)
    pages/gallery.liquid + gallery.json (client-filtered gallery)
  preview.json           site + tokens
  preview/posts/*.md     ~20 photo posts (picsum, mixed sizes, 5 tags, 1 without photo)
  preview/pages/         home.md, gallery.md, about.md (preview.json leaves the homepage unset, so the guide shows)
  README.md
```

## Build steps

1. `masthead new Aperture` → move into `aperture/`, then strip the scaffold
   blog page and example tokens.
2. Manifest, templates, CSS and preview content.
3. `masthead validate`.
4. `masthead preview`, checked in a browser:
   - both nav positions at desktop and mobile, and all three `color_scheme`
     values
   - both home variants
   - gallery `auto` rhythm and every `tile_size` value
   - default index shows the guide with both nav positions
   - filter chips on `/gallery?tag=`, plus a tag link from a post
   - post page with a tall, a wide and a no-body photo
   - prev/next at both ends of a tag
   - reduced motion

## Decisions (were open questions)

1. Home variants: **Full frame + Contact sheet**.
2. Where "all photos" lives: **only the `Gallery` theme page**. The default
   index is a static setup guide that explains the theme and how to replace
   it with Home or Gallery as the homepage.
3. Mosaic: tiles **crop** to their size, and a **`focus`** post option sets
   the crop anchor.
4. Photo metadata: **`camera` and `location`**, both free text.
5. Enlarged view: **the post page** (photo, then title and meta, then body,
   then prev/next). No lightbox.
6. Palette: **follows the system** by default. The `color_scheme` token can
   lock it to light or dark.
