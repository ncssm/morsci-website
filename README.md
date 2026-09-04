# The Morganton Scientific — website

The content for <https://morgantonscientific.ncssm.edu>: the landing page and the pages that list
published articles.

**This repository does not contain the articles.** Each volume lives in its own repository —
[`ncssm/morgantonscientific2026`](https://github.com/ncssm/morgantonscientific2026) and so on — which
submits to Curvenote. This repository only *lists* what Curvenote has published.

Publishing a volume therefore does not put it on the website. They are two separate steps.

---

## ⚠️ There is no preview

Pushing to `main` deploys immediately. There is no draft mode, no preview build, and no checks
gating a merge — nothing here resembles the `draft` label and per-article checks in the volume
repositories.

Work on a branch, open a pull request so someone can read the diff, and **check the live site once
the action finishes** rather than before.

The blast radius is small — the content is a handful of short files, and a mistake is fixed by
another merge — but the safety net you are used to in the volume repo does not exist here.

---

## How deployment works

`.github/workflows/deploy.yml` fires on push to `main`:

```yaml
jobs:
  build-and-deploy:
    uses: curvenote/actions/.github/workflows/push.yml@v1
    with:
      landing-content: ncssm-mor
    secrets:
      CURVENOTE: ${{ secrets.CURVENOTE_TOKEN }}
```

which runs two commands:

```bash
curvenote work push --public -y                # upload this repository's content
curvenote site init ncssm-mor --set-content    # make it the venue's landing content
```

`--set-content` requires **admin permission** on the Curvenote token. The repository secret has it;
a personal token may not.

---

## The pages

| File | URL | Purpose |
|---|---|---|
| `index.md` | `/` | Homepage — hero banner, then the 3 most recent articles |
| `articles.md` | `/articles` | Every volume, newest first, separated by `---` |
| `<year>.md` | `/2026` | A single volume on its own page |
| `volumes.md` | `/volumes` | The `volumes` collection |
| `myst.yml` | — | Table of contents |

Every listing is the same directive, which queries Curvenote live:

````markdown
```{cn:articles}
:venue: ncssm-mor
:collection: 2026
:show-thumbnails:
:show-date:
:show-authors:
:layout: cards
```
````

`:limit: 3` restricts the count — used on the homepage, which also omits `:layout: cards`.

**The table of contents is hidden from the site.** `/2026` and `/volumes` exist and work, but nothing
links to them; they are reachable only by typing the URL. Adding a nav is possible if that is ever
wanted.

---

## Adding a new volume

**First, publish the volume in Curvenote.** `cn:articles` lists *published* articles, so a volume that
has only been submitted renders an empty list — with no error on either side. In the volume
repository that means running `publish_dispatch.yml`.

Then here, on a branch — four files, two of them one-line changes:

**1. Create `<year>.md`.** Copy last year's and change the title and collection:

````markdown
---
title: '2027'
description: All student articles
---

```{cn:articles}
:venue: ncssm-mor
:collection: 2027
:show-thumbnails:
:show-date:
:show-authors:
:layout: cards
```
````

**2. Add a section at the top of `articles.md`**, above the previous year, with a `---` rule after it:

```markdown
## 2027

```{cn:articles}
:venue: ncssm-mor
:collection: 2027
...
```

---

## 2026
```

**3. Point the homepage at the new volume** — one line in `index.md`:

```diff
-:collection: 2026
+:collection: 2027
```

**4. Add the page to the table of contents** in `myst.yml`, first among the children:

```yaml
    - file: articles.md
      children:
        - file: 2027.md
        - file: 2026.md
        - file: 2025.md
```

**5. Merge, then check the site.** The homepage should show three articles from the new volume, and
`/articles` should list the new section first.

---

## If a listing comes up empty

Two causes, neither of which reports an error:

- **The volume is not published.** Submitted is not enough.
- **The collection name does not match.** `:collection: 2026` here must equal the `collection` the
  volume repository submits under, set in all four of its workflows:

  ```yaml
        venue: ncssm-mor
        collection: '2026'
  ```

---

## History

The site was rebuilt in September 2026 across two pull requests: [#5](../../pull/5) introduced the
hero banner and explicit per-volume pages in place of an autopopulated list, and [#6](../../pull/6)
added the 2026 listings and replaced the old `curvenote/action-myst-publish@v1` deploy step with the
reusable `push.yml` workflow above.
