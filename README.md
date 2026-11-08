# Octopart for PartCAD

Octopart, published with the other stores as `//pub/svc/commerce/octopart`, in two parts:

* **a store**, `//pub/svc/commerce/octopart:octopart`, that quotes any manufacturer's part,
  by its manufacturer and part number, at the **cheapest authorized offer among every
  distributor Octopart indexes** - DigiKey, Mouser, Farnell, RS, Arrow, TME and the rest -
  in your currency, and says which distributor it is and its number there;
* **a catalogue**, `//pub/svc/commerce/octopart/products`, whose child packages are
  Octopart's categories, each searched with Octopart's search. A part is a manufacturer's
  part, named `<MPN>~<manufacturer>` (`BAV99~nexperia`, since two manufacturers can share a
  part number), says `vendor: <manufacturer>` and `sku: <MPN>` - what every store quotes by -
  and its geometry is the STEP file inside the KiCad library Octopart's CAD partner made of it.

Both are one script, `octopart.py`, asking **Nexar's GraphQL API** (<https://api.nexar.com/graphql>),
which is what the "Octopart API" is now. The queries are validated against Nexar's schema.

Octopart sells nothing itself, and that is what makes it the best catalogue here: its parts
are manufacturers' parts, which this store, DigiKey's, Mouser's and any other quote by
manufacturer and part number.

## Setting it up

Octopart's API is Nexar's, and it answers an app of your own: its part quota is yours.

```shell
pc login //pub/svc/commerce/octopart:octopart_repo
pc login //pub/svc/commerce/octopart:octopart
```

opens <https://portal.nexar.com/apps> in PartCAD's browser. Sign in there (an Octopart or
Altium account will do), create an app with the **Supply** scope if you have none - a scope
cannot be added afterwards - and open its **Authorization** page. The Client Secret is read off
that page; paste the Client ID into PartCAD's banner from the page's "Copy Client ID". Both are
checked with Nexar before they are kept, in `~/.partcad/config.yaml`, by each plugin's name: the
store and the catalogue are two plugins, so log in to both. Where nobody is at a browser,
`pc login --agent ... --set client_id=... --set client_secret=...` takes them, and so do
`NEXAR_CLIENT_ID` and `NEXAR_CLIENT_SECRET`. The token lasts a day and is kept that long.

A new app answers no parts until some are assigned to it under **Manage supply plan**; the
free evaluation app has 100 for its lifetime.

Why not sign in to Octopart itself? Nexar charges a supply query to the app that makes it, so
signing users in through an app of PartCAD's would put every user's searches on one quota, and
that sign-in needs the app's secret, which Nexar's terms forbid publishing. Octopart's own
site, behind a bot check, is not something to script either.

## Using it

```shell
pc list packages //pub/svc/commerce/octopart/products...
pc search parts -P //pub/svc/commerce/octopart/products/discrete-semiconductors/diodes -k bav99
pc info //pub/svc/commerce/octopart/products:BAV99~nexperia
pc supply quote //pub/svc/commerce/octopart/products:BAV99~nexperia#100
```

## How Nexar is asked

| What PartCAD asks | Nexar |
| --- | --- |
| the categories | `supCategories` (not metered), kept a day; their root, *Electronic Parts*, is the catalogue |
| a search | `supSearch(q, filters: {category_id: [...]})`, without sellers: a search lists, it does not price |
| a part, `avail`, `quote` | `supMultiMatch(queries: [{mpn}])`, with every authorized seller's offers, kept an hour |
| its model | `cad.downloadUrlKicad`, where `cad.has3dModel`: the STEP inside the zip |

A quote prices every authorized offer at the quantity actually ordered - its minimum, its
multiple, the price break that reaches - in your currency (Nexar converts; a price already in
it is taken as it is) and takes the cheapest, preferring what is in stock.

## What it cannot do (yet)

* **Nexar meters parts, not requests**, monthly, and its free plan is small (100 or 1,000
  parts, depending on which of Nexar's pages one reads). A search answers as many parts as
  it is asked for: ask for few (`pc search ... --limit`).
* **The model comes inside an ECAD library.** Nexar has no STEP of its own: whether a plan
  includes CAD ("ECAD Modules"), and whether those downloads need an Octopart sign-in, are
  Nexar's to decide. When the download is not a library, the part says so rather than building.
* **No ordering.** Octopart has none; `supOctocart` gives a link to a distributor's cart,
  which a later version could answer as the quote's `cartId`.
* Nexar's terms keep what it answers for a day at most, forbid sharing a key, and ask for a
  link to Octopart where its data is shown.

Run end to end in PartCAD 0.9.22 - a search in a category, a part built from the STEP in its
KiCad library, a quote - against servers standing in for Nexar's (the plugins' `api` and
`identity` parameters point them there). Not yet against Nexar's own, for want of credentials:
the tests answer it in the shapes of Nexar's schema, and the queries are validated against it.

## Tests

```shell
pip install pytest
pip install --no-deps partcad    # for plugin_models, which PartCAD gives every plugin
pytest
```

`distributor.py` is shared, byte for byte, with the other distributor packages (DigiKey,
McMaster-Carr); it is edited in `incubator/_shared/` of the PartCAD repository and copied here,
and `distributor.sha256` is its digest, which the tests hold this copy to.
