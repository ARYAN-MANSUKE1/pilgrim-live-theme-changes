# Pilgrim Data — live theme changes

The theme files changed on the live site **pilgrimdata.in** during the AI Feature
Development engagement (T-Rex Agency, September–October 2026), with what was
changed, why, and how to carry the changes over if the theme is ever replaced.

This is **not the complete theme**. It holds only the files that were changed. Every
other file of the active theme `pilgrim-data-services` is as it was. The translation
pipeline and admin dashboard are in a separate repository:
`github.com/ARYAN-MANSUKE1/Pilgrim-pipeline`.

## Contents

```
theme/                              exact copies of the changed files, at their path inside the theme
  functions.php                     language audio + popup, ad system
  single-temple.php                 ad slots, related temples in 10 languages
  header.php                        language dropdown with 10 languages
  footer.php                        footer text in 10 languages
  inc/acf.php                       fields for the 7 extra languages
  inc/template-tags.php             10-language map with the real field names
  inc/shortcodes.php                10-language label dictionary + helpers
  template-parts/home-content.php   homepage text in 10 languages
  additional-css.css                Customize -> Additional CSS (not a theme file)
settings/
  scf-advertisements-export.json    Advertisements post type + "Advertisement Details" fields
diffs/                              before -> after diff of every changed file
tests/test_ads.php                  checks of the ad matching (php tests/test_ads.php)
```

Theme path on the server: `wp-content/themes/pilgrim-data-services/`.

## How the changes were made

- **Theme files** were edited in WP Admin → Appearance → **Theme File Editor**. That
  editor cannot create new files, which is why the ad system and the language
  audio / popup code are inside `functions.php` rather than in their own `inc/` files.
- **CSS fixes** were added in Appearance → Customize → **Additional CSS**.
- **Ad fields** were imported in SCF → Tools → Import (`settings/`).
- **No plugins** were installed or updated.
- The site runs the **10Web** optimizer and page cache. It delays inline scripts until
  the visitor's first scroll or tap, and serves cached pages. Several changes below
  exist because of that. After any change, clear the 10Web cache.

## Why most of the changes were needed

The site publishes 10 languages: Marathi, English, Hindi, Gujarati, Tamil, Telugu,
Malayalam, Kannada, Bengali and Punjabi. The theme was written for three. Articles in
the other 7 were stored in the database, but the theme either read the wrong field
names or had no text for them. On those 7 languages, the homepage, footer, related
temples and parts of the temple page stayed in Marathi or went blank.

Most edits replace three hard-coded strings with a lookup that covers all 10. The
language-switching mechanism itself was **not** changed. Every translatable piece is
printed once per language as `<div class="{code}_lang dis_lang">`, and the theme's
`assets/js/theme.js` shows the selected one and stores it in the `language` cookie.

## What changed, file by file

### functions.php
| Code | What it does |
| --- | --- |
| `pds_audio_url()`, `pds_language_audio_and_nav_fit()` (wp_footer, 20) | Audio button next to the language dropdown that plays the temple's recording in the selected language; shrinks the header menu so long labels (e.g. Malayalam) stay on one line |
| `pds_language_selection_popup()` (wp_footer, 30) | First-visit "Choose your language" popup. Uses the same `language` cookie as the dropdown, plus `pilgrim_lang_seen` if closed without choosing. The script has `data-two-no-delay` so 10Web runs it immediately, and `#pilgrim-lang-popup-overlay[hidden]{display:none}` makes closing actually hide it |
| `pds_get_matching_ad()`, `pds_display_ad( $slot )` and helpers | Ad system: picks a running ad for the temple's city and the slot (status *active*, date range, target cities; a state includes its districts). Result cached 20 minutes per temple and slot |
| `pds_ajax_track_click()` + footer script `pdsTrackClick()` | Click counting (`_ad_clicks`). The script is excluded from the 10Web delay; only real Advertisements posts are counted |
| impression counter in `pds_display_ad()` | `_ad_impressions`. Counts page renders, so cached page views are not counted |
| `pds_flush_ad_cache()` | Clears the ad cache on every ad save, or at `/?pds_flush_ads=1` as an admin |
| `register_post_meta(...)` for the ad fields | Lets logged-in editors read and write the ad fields over the REST API (the dashboard's Ads tab). SCF's "Show in REST API" was not used because `/wp/v2/advertisements` is public and would expose campaign prices |
| `pds-ad-styles` (wp_head) | Ad label and image styles; image height capped at 250px for banners and 600px for side columns, so a wrongly shaped image cannot push the article off screen |

The mobile/desktop ad image is chosen by the browser (`<picture>`), not by the
server, so the page cache can't serve one device's image to everyone.

Ad slots: `temple_top` (banner above the article), `temple_left` / `temple_right`
(160px sticky columns, hidden below 769px), `temple_bottom` (banner below the
article). Recommended images: 728 × 90 banners, 160 × 600 sides.

Ad field names: new names (`ad_status`, `ad_placement`, `ad_target_regions`,
`ad_creative_desktop`, …) with the old SCF names (`campaign_status`, `placement`,
`target_cities`, `ad_creative`, …) as fallback. Dates `Y-m-d`, `d/m/Y` or `Ymd`.

### single-temple.php
- Renders each ad slot first and only places it on the page if an ad came out. With
  no running ad, the page is the plain full-width layout; with ads it becomes a 2- or
  3-column grid (`.pds-temple-cols.has-left/has-right`).
- The related-temples heading and cards loop over all 10 languages. They were
  Marathi / English / Hindi only, and the Gujarati lookup used the misspelt field
  `gujrati_title`.

### inc/template-tags.php
- `pds_get_temple_languages()` now lists all 10 languages with the field names the
  content is really stored under (see the table below). Gujarati previously pointed
  at `gj_translation` / `gujrati_title`, which never held data. English gained
  `english_title`, and every language gained an audio field.
- `pds_get_temple_unavailable_message()` falls back to the dictionary's translated
  message for languages that had none.
- Unchanged: `pds_image_url()` and `pds_skip_lazy_attr()` were already in the live file.

### inc/shortcodes.php
- `pds_i18n_strings()`: one dictionary of the site's fixed labels, 31 keys × 10
  languages (state and city names, section headings, "View details", social and
  legal links, About Us, "content not available").
- `pds_t( $key, $code )` returns one label, falling back to English.
  `pds_lang_blocks( $key, $tag, $class, $display, $inner )` prints the full set of
  `.{code}_lang .dis_lang` blocks for a key.
- `[temple_grid]` / `[temple_slider]` show titles and "View details" in 10 languages.
- `pds_temple_popup_i18n()` adds the "not available in this language" messages for
  the 6 languages missing from `pdsTemplePopup.i18n`.

### template-parts/home-content.php, footer.php, header.php
- Homepage: state, city and gallery labels are dictionary keys (`st_*`, `ct_*`,
  `g_*`); headings, buttons and About Us loop over 10 languages.
- Footer: link labels are dictionary keys; copyright in 10 languages.
- Header: the language dropdown is generated from `pds_get_temple_languages()`.

### inc/acf.php
- `pds_register_missing_language_fields()` registers title / content / audio fields
  for the 7 extra languages, so they appear in the temple editor and the REST `acf`
  object. The data already existed under these keys.

### additional-css.css (Customize → Additional CSS)
Article photos float at 50% width, and the white article card had no clearfix.
Where the text beside the last photo was shorter than the photo (it depends on the
temple and the language), the photo hung out of the card and squeezed the gallery to
half width. Rules 1–2 contain the floats and keep "Useful Information" below the last
photo. Rule 3 lets two photos sit side by side on desktop.

### Database / settings
- SCF: Advertisements post type + Advertisement Details field group
  (`settings/scf-advertisements-export.json`). The Placement field has exactly
  `temple_top`, `temple_left`, `temple_right`, `temple_bottom`. "Target Categories" is
  not used by the code (ads are targeted by city).
- Menu item 1981: Kannada "Home" label, Telugu vowel sign U+0C41 → Kannada U+0CC1.

### Language fields
| Code | Language | Content field | Title field | Audio field |
| --- | --- | --- | --- | --- |
| mr | Marathi | `mr_translation` | `marathi_title` | `marathi_audio` |
| en | English | `en_translation` | `english_title` (else post title) | `english_audio` |
| hi | Hindi | `hi_translation` | `hindi_title` | `hindi_audio` |
| gj | Gujarati | `gujarati_content` | `gujarati_title` | `gujarati_audio` |
| ta | Tamil | `tamil_content` | `tamil_title` | `tamil_audio` |
| te | Telugu | `telugu_content` | `telugu_title` | `telugu_audio` |
| ml | Malayalam | `malayalam_content` | `malayalam_title` | `malayalam_audio` |
| kn | Kannada | `kannada_content` | `kannada_title` | `kannada_audio` |
| bn | Bengali | `bengali_content` | `bengali_title` | `bengali_audio` |
| pa | Punjabi | `punjabi_content` | `punjabi_title` | `punjabi_audio` |

## Status on live (checked 6 October 2026)

Live: language popup, audio button, ad system (top / left / right) with click
tracking, all 10-language changes, CSS rules 1–2, the SCF import and the menu fix.

Not yet pasted:
- `functions.php`: ad image height cap, REST ad fields.
- `single-temple.php`: bottom ad slot.
- CSS: rule 3.
- SCF: the four Placement choices.

## Deploying an update

1. Theme File Editor (or FTP): replace the file with the copy from `theme/`, at the
   same path. Paste whole files. Never mix them with older copies of the theme, which
   would undo the 10-language work.
2. Additional CSS: paste `theme/additional-css.css`.
3. Clear the 10Web cache. For ads, also visit `/?pds_flush_ads=1` as an admin.
4. Check a temple page in a few languages, on desktop and phone.

## If the theme is replaced

Nothing below lives in the database, so a new theme starts without it. All content
(translations, titles, audio files, ads, cities) **is** in the database and survives.

1. **Language fields.** Keep the field names in the table above; the pipeline writes
   to exactly these keys. Port `pds_get_temple_languages()` (the single list every
   template loops over) and `pds_register_missing_language_fields()`.
2. **Language switching.** Either keep the `.{code}_lang .dis_lang` blocks plus
   `#language-select` and the `language` cookie (handled by the old `theme.js`), or
   rebuild it. The popup and audio button call `window.pdsLanguage.set()` when
   `theme.js` is present and fall back to toggling `.dis_lang` themselves.
3. **Fixed site text.** Port `pds_i18n_strings()`, `pds_t()` and `pds_lang_blocks()`
   from `inc/shortcodes.php`; the 31 keys hold every label in 10 languages.
4. **Popup and audio button.** Copy `pds_language_selection_popup()` and
   `pds_language_audio_and_nav_fit()`. The audio button looks for `#language-select`;
   the menu fit looks for `#menu-primary-menu` and `.site-branding`. Keep
   `data-two-no-delay` on the popup script while 10Web is in use.
5. **Ads.** Copy the ad block from `functions.php` (from `define( 'PDS_AD_POST_TYPE'`
   to the ad styles) and call `pds_display_ad( 'temple_top' | 'temple_left' |
   'temple_right' | 'temple_bottom' )` where the slots should appear. It prints
   nothing when no ad runs, so wrap it the way `single-temple.php` does to avoid empty
   space. The Advertisements post type and fields come from SCF, not the theme.
6. **CSS.** `additional-css.css` targets `.temple-details`, `.content-section`,
   `.dis_lang` and `.hero`. Adapt the selectors to the new theme's markup, or drop
   them if it already contains floated images.
7. **Dashboard's Ads tab** needs the `register_post_meta` block from `functions.php`.

## Checks

```
php tests/test_ads.php        # ad matching: dates, status, cities, legacy fields (14 checks)
php -l theme/functions.php    # syntax, likewise for each file
```
