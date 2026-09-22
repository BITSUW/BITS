# BITS at UW — website

Static site for the Business Information Technology Society at the University of Washington.
No build step, no dependencies. Plain HTML, CSS, and about 20 lines of JavaScript.

## Structure

```
index.html          all page content
css/styles.css      all styling; design tokens live at the top in :root
js/main.js          mobile menu toggle + footer year
assets/             logos
```

## Run it locally

Open `index.html` in a browser. That's it. If you'd rather use a local server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Publish on GitHub Pages

1. Push this folder to a repo (for example `bits-uw/website`).
2. Settings → Pages → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`.
3. The site goes live at `https://<org>.github.io/<repo>/` in a minute or two.

For a custom domain, add a file named `CNAME` at the root containing the domain, then point
the DNS at GitHub.

## Editing content

Everything is in `index.html`, marked with comment banners for each section.

**Next event** — there's no hardcoded date/time/room card anymore. The Events section opens
straight into the Instagram feed, with a line of text explaining that's where meeting details
get posted. To change that wording, edit the `<p class="event-note">` inside
`<div class="event-instagram">`.

**Guest speaker interest form** — the button in "Get in touch" (`#contact`) links to the same
Google Form used for guest speaker / partnership inquiries:
`https://forms.gle/AE6JBFkrTckZeVzV9`. If that form ever changes, update the `href` on that
button.

**Executives** — each person is one `<li class="member">`. Copy an existing block to add
someone. The `member-mark` div holds their initials; if you'd rather use photos, replace
that div with:

```html
<img class="member-photo" src="assets/first-last.jpg" alt="">
```

and add a matching `.member-photo` rule in the stylesheet (same 3rem box, or larger).

**Links** — the Discord, Instagram, and LinkedIn URLs appear in the hero, the join band, and
the footer. Search for `discord.gg` to find them all.

## Colors and type

The palette is the logo's black and white plus the sun-to-sky gradient the club supplied.
Change the values in `:root` at the top of `css/styles.css` and the whole site follows.

| Token | Value | Used for |
| --- | --- | --- |
| `--ink` | `#000000` | nav, events band, footer, text on the gradient |
| `--paper` | `#FFFFFF` | page background |
| `--sun` | `#F9C925` | gradient start (bottom-left); accents on black |
| `--sage` | `#B2C4B5` | gradient midpoint |
| `--sky` | `#43B2E8` | gradient end (top-right); section markers, link underlines |
| `--deep` | `#0B5A82` | darker blue for link hovers and focus rings on light backgrounds |
| `--wash` | `#EEF3F0` | pale sage tint behind the mission statement |

The gradient itself is one variable, `--spectrum`, running at 45° from sun through sage to
sky. It fills the hero and the closing "join" band, and each executive's initials tile
shows a different slice of it. To retire it from a section, swap `var(--spectrum)` for a
flat color.

`--sky` is too light to use as text on white (about 2.2:1 contrast), so text and focus
rings on light backgrounds use `--deep` instead.

Type is Space Grotesk for headings and IBM Plex Sans for body text, both loaded from Google
Fonts in the `<head>`.

## Instagram feed

The Instagram feed lives inside the Events section now (`#instagram` in `index.html` is a
subsection of `#events`, not its own page section) — it's the first thing in Events, since
Instagram is the source of truth for meeting details. It uses an
[Elfsight](https://elfsight.com) widget to auto-pull recent posts from `@bitsatuw`. It's two
lines dropped straight in:

```html
<script src="https://elfsightcdn.com/platform.js" async></script>
<div class="elfsight-app-6d069763-a333-477a-80e8-5a832ea95518" data-elfsight-app-lazy></div>
```

**Is the app ID in that snippet safe to commit publicly?** Yes. It's a public widget
identifier, not a credential — comparable to a YouTube video ID or a Google Analytics tag.
It only tells Elfsight's CDN which display config to render; it can't be used to access your
Elfsight account, see private data, or change the widget's settings. The only thing someone
could do by copying it is show the same public feed on another page.

**To change how many posts show, the layout, or which account it pulls from:** log into
whoever set up the Elfsight widget's dashboard at elfsight.com and edit the app there — the
site doesn't need to change. To swap in a different widget entirely (a different provider, or
a from-scratch build against the official Instagram Graph API), replace the two lines above
inside `.insta-embed` in `index.html`.

**Free-tier limits:** Elfsight's free plan caps impressions per month and adds a small
"Powered by Elfsight" badge to the widget. If that's a problem, either upgrade the plan or
swap to another provider (SnapWidget, Behold.so) using the same drop-in pattern.

## Email list signup

There's a "Join our email list" section (`#email-list` in `index.html`) with a name + email
form, but **it isn't wired to anything yet** — right now submitting it does nothing. The
`<form>` tag has a `TODO` comment right above it in the HTML with the same instructions
below. Pick whichever of these fits how BITS wants to manage the list:

**Option A — Google Form (free, easiest if you already use Google Forms):**
1. Create a Google Form with a Name and an Email question.
2. Open the live form, right-click → "Inspect" (or use the form's prefilled-link feature) to
   find each field's `entry.XXXXXXXXX` ID, and the form's `formResponse` submit URL — it looks
   like `https://docs.google.com/forms/d/e/FORM_ID/formResponse`.
3. In `index.html`, set the `<form>` tag's `action` to that URL, and change `name="name"` /
   `name="email"` on the two inputs to the matching `entry.XXXXXXXXX` values.
4. Google Forms will reject a normal redirect back to your page, so also add
   `target="hidden_iframe"` to the `<form>` tag and follow Google's standard
   ["submit a form without leaving the page"](https://www.google.com/search?q=google+form+submit+hidden+iframe)
   pattern if you want visitors to stay on your site after signing up.

**Option B — An email service (Mailchimp, ConvertKit, etc.):**
Most of these give you a ready-made embed snippet (its own `<form>` with its own field names
and action URL) when you create a signup form in their dashboard. Delete the placeholder
`<form>...</form>` block entirely and paste theirs in its place — the surrounding
`.signup-form`-styled look will be theirs to restyle, not this site's CSS.

**Option C — Formspree (no backend, minimal setup):**
1. Sign up free at [formspree.io](https://formspree.io) and create a form to get a URL like
   `https://formspree.io/f/yourFormID`.
2. Set the `<form>` tag's `action` to that URL and leave `method="POST"` as-is. The existing
   `name="name"` / `name="email"` fields work with no other changes.

Whichever option, submissions should land in a place someone actually checks — a spreadsheet,
an inbox, or your ESP's dashboard — so confirm that before pointing people at the live page.

## Notes

- The two LinkedIn URLs on the old site disagreed (`/in/uwbits` vs `/company/uwbits`) and so
  did the two Discord invites. This build uses `/company/uwbits` and `discord.gg/CEKbdgX9`.
  Confirm both and fix if needed.
- The old "Our Instagram" button actually pointed at a Google Form. Here it's an RSVP button
  on the event, and Instagram links to Instagram.
- Executive headshots aren't included. Initials stand in until you drop photos in `assets/`.
