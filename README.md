# annalchemy.org

The AnnAlchemy™ site: a set of hand-written static HTML pages, one file per page,
no build step and no framework. Every page is self-contained — its CSS lives in a
`<style>` block in its own `<head>`, its JS in a `<script>` at the end of its own
`<body>`. There is nothing to install and nothing to compile. Open a file, edit it,
push it.

Two brands live here. **AnnAlchemy™** is the parent, and its pages are cream on
espresso. **Self-Loyalty Lab™** is a distinct second brand under it, plum on warm
white. The Relapse Control Guide™ sits under AnnAlchemy; the Boyfriend Boundary
Guide, the Self-Return family and the System sit under Self-Loyalty Lab. Where
both appear together — the links page — AnnAlchemy is the mother mark.

## Deployment

Cloudflare Workers static assets. `wrangler.jsonc` serves the repository root as
the asset directory, so a file's path in the repo is its path on the site:
`self-loyalty-lab/index.html` is served at `/self-loyalty-lab/`.

There is no server, so there are no server-side redirects. Vanity URLs are
meta-refresh stubs instead — see `sll/` and `selfloyaltylab/`, each a few hundred
bytes that bounce to the real page.

## Pages

| Path | What it is |
|---|---|
| `index.html` | Main AnnAlchemy sales page — Relapse Control Guide™ |
| `start.html` | Alternate entry to the same offer, aligned to `index.html` |
| `when-it-comes-back.html` | *When It Comes Back: The First 48 Hours™* — the downsell |
| `links/` | Link-in-bio page, both brands' products |
| `links2/` | Earlier, lighter links page |
| `facebook/` | Facebook profile chooser |
| `terms.html`, `privacy-policy.html`, `refund-policy/` | Policy pages, cream background |
| `self-loyalty-lab/` | Self-Loyalty Lab sales page |
| `self-loyalty-lab/start/` | The Starting Point™ Assessment — 15 questions, scored in the browser |
| `check/` | The Over-Giving Check™ — Tally form wrapper |
| `assistant/`, `workbook/`, `sll/`, `selfloyaltylab/` | Vanity redirect stubs |
| `self-loyalty-lab/source/` | Design sources, not served content — see below |
| `img/` | Portraits and guide preview covers |

## Product links

Every buy button on the site points at Payhip. Change one and change it
everywhere it appears.

| Product | Link |
|---|---|
| Relapse Control Guide™ | `payhip.com/b/ZzmSJ` |
| When It Comes Back: The First 48 Hours™ | `payhip.com/b/MWcLe` |
| Boyfriend Boundary Guide, English | `payhip.com/b/c5ZAz` |
| Boyfriend Boundary Guide, Taglish | `payhip.com/b/uCRbB` |
| Self-Return Guide, English | `payhip.com/b/LTXpY` |
| Self-Return Guide, Taglish | `payhip.com/b/Skgca` |
| Self-Return Workbook, English | `payhip.com/b/CBtId` |
| Self-Return Assistant, 3-month pass | `payhip.com/b/XiBCj` |
| The Self-Loyalty Lab System | `payhip.com/b/g9PLB` |

## The AnnAlchemy design system

Tokens are defined once in `:root` at the top of each page's stylesheet, and
everything downstream reads them. To restyle a page, change the tokens — not the
rules that consume them.

**Palette.** The identity sheet's reversed colourway: cream on espresso. Surfaces
step along an espresso → black axis (`--bg:#15110D`, `--paper:#2A1F18`), text is
cream (`--ink:#F4EDE4`), and clay (`#C4A88C`) is the only accent hue. Do not
introduce a new colour; take a tint of clay instead.

**Type.** Georgia for large display headings, Inter for body, and Cormorant
Garamond reserved for the ANNALCHEMY™ wordmark alone. Cormorant is a wordmark
face here, not a heading face.

**The mark.** A triangle with a horizontal crossbar — the alchemical symbol for
earth, doubling as a capital A — drawn as inline SVG, never filled:
`M50 20 L76 74 H24 Z` with the crossbar at `M35.5 56 H64.5`, on a 100×100 viewBox.
Stroke weight steps with size per the identity sheet; 5.5 at 44px, 7 at small
sizes. Espresso stroke on light, cream stroke on dark.

**No hairlines.** Cards and buttons carry no visible border. `--line` and
`--line-strong` are set to `transparent` rather than removed, which keeps every
box's geometry intact so nothing shifts. Separation comes from depth instead: the
`--shadow-*` tokens each carry a lit top edge (`inset 0 1px 0` cream at low alpha),
a dark base (`inset 0 -1px 0`) and an outer drop shadow. Every consumer of a shadow
token inherits the emboss for free. Where a real rule between content is wanted,
use `--divider`.

**Buttons** are struck metal: a multi-stop vertical gradient with a bright crown, a
wide dark waist behind the label, and a bounce-lit base. Two tiers, deliberately
distinguishable — outbound links (`a[target="_blank"].btn-primary`) run a brighter
crown and a white label, and in-page scroll-to buttons run the same waist with a
subdued crown.

Self-Loyalty Lab runs its own smaller system: plum `#5E2A53` on warm white
`#FBF7F0`, near-black `#171417` for the dark bands, Playfair Display for display
type, EB Garamond for body, Archivo for UI.

## Scroll reveal

Every real page carries the same reveal: text, cards, sections and paddings blur,
fade and rise as they enter the viewport. The implementation is identical across
pages and worth understanding before you touch it.

- **Two depths.** A container travels 28px; text inside it travels a shorter 12px
  just after. A card should arrive as one object whose contents settle into it, not
  as a frame that fills in piece by piece. Nesting deeper than two compounds badly,
  so `MAX_DEPTH` caps it.
- **JS applies the classes.** Nothing is hidden in the HTML, so the page renders in
  full with JS off or broken. This is not optional — content must never be lost to
  the animation.
- **Classes are stripped once the transition ends**, so no stray transform survives
  to break `position:sticky` further down the page.
- **An IntersectionObserver is not enough on its own.** A fast flick can carry a
  block through the viewport between two observer deliveries and strand it invisible
  for good, so a rAF-throttled scroll sweep backs it up. The sweep also has to
  special-case maximum scroll: anything sitting in the bottom 6% dead zone could
  never otherwise reveal, so at the end everything remaining is taken.
- **Anchor jumps** pre-paint their destination, so `#system` never lands you
  mid-animation.
- **`prefers-reduced-motion: reduce` exempts the page entirely**, including when the
  setting is changed mid-session.

The Self-Loyalty Lab pages add one thing: the assessment builds its questions and
its whole result screen at runtime, so the collector reruns on mutation. Without
that the result would arrive faded out and never come back.

## Self-Loyalty Lab: source → page

`self-loyalty-lab/source/SelfLoyalty_Lab_Sales_Page.dc.html` is the Design Canvas
source for the sales page. It is kept verbatim as the design of record and is
**not** served.

It is not a web page, and pasting it in as one will not work. It uses `<x-dc>`,
`<helmet>`, `sc-if`, `style-hover` and `image-slot`, none of which mean anything to
a browser, it loads `support.js` and `image-slot.js` from a path that does not
exist here, and every buy button in it is `href="#"`. Rendered as-is it looks
roughly right and sells nothing.

`self-loyalty-lab/index.html` is that source rendered for the browser, preserving
the layout and styling exactly while resolving each construct: the wrappers are
unwrapped, `style-hover` becomes real `:hover` / `:focus-visible` CSS, the portrait
slot becomes `img/anna-cadano-founder-portrait.jpg`, both `sc-if` conditionals ship
shown, and the seven buy buttons take their Payhip links.

If the design changes, update the source and re-render it. Editing only the source
changes nothing that anyone can see.

## Conventions and traps

A few things in here have bitten before:

- **`background:` shorthand resets `background-color`.** If a page needs both, the
  colour declaration must come *after* the shorthand, not before. Removing what
  looks like a redundant duplicate is how the main page lost its background once.
- **Never remove a border to hide it** — set it transparent. Removing it changes the
  box's size and shifts everything inside.
- **Inline `style="color:…"` beats any stylesheet hover rule.** Move the colour to a
  class rather than escalating to `!important`.
- **Check contrast against rendered pixels, not against intent.** A gradient face
  with a light label is easy to get wrong, and the label usually sits on the waist
  rather than the crown, which changes the answer.
- **Test at 390px as well as desktop.** Mobile gutters have been zeroed by a
  `@media (max-width:600px)` rule before now.

## Open items

- **The crisis hotline numbers on the Self-Loyalty Lab page are unverified.** 911,
  the NCMH Crisis Hotline on 1553, the PNP Women and Children Protection Center and
  the DSWD number all need checking against current sources. This is the one thing
  on the site where being wrong hurts someone.
- **Price mismatch.** The Self-Return Guide Taglish edition reads $29 on the
  Self-Loyalty Lab sales page and $25 on the links page. One is wrong.
- The borderless emboss and the struck-metal buttons are applied to `index.html`
  only. `start.html`, the links pages and the policy pages still carry visible
  hairlines and flat buttons, so the two entry pages to the same offer no longer
  match each other.

---

Anna Cadano, RN · hello@annalchemy.org
