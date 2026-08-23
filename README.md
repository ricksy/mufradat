# Mufradat

The Hugo source for <https://mufradat.com>.

This repository is the site. Clone it, run Hugo, and you have the whole blog
back — no submodules to fetch, no server to log into, nothing to reach for
that is not in here.

## Build it

Hugo **v0.147.1 extended**, the version the published site is built with:

```bash
hugo server            # preview at http://localhost:1313
hugo                   # write the site to public/
```

Pinning the version matters more than it looks: a rebuild on this version is
byte-for-byte identical to what is being served, so any difference in the
output is a real difference and worth reading.

## Write a post

```bash
hugo new content posts/my-new-post.md
```

Then edit it and set `draft: false`. Images go in `static/images/<slug>/` and
are referenced from the post as `/images/<slug>/hero.jpg` — the leading slash
matters, and the path is the *published* one, not the source one.

## Layout

| Path | What it is |
| --- | --- |
| `content/posts/` | The posts |
| `content/about.md`, `content/impressum.md` | The standing pages |
| `static/images/<slug>/` | Post images, one directory per post |
| `themes/PaperMod/` | The theme, vendored — see `themes/PaperMod/VENDORED.md` |
| `hugo.toml` | Site config |
| `public/` | Build output. Ignored: Hugo regenerates it, and committing it made the tree permanently dirty |

## Publishing

Push to `main`. A GitHub Action builds the site and deploys it; there is no
manual build-and-copy step and nothing to remember.

## Backups

`origin` is GitHub. Codeberg is kept as a second remote on a different host,
under a different company, because a backup on one provider is a backup with
one way to fail:

```bash
git push origin main
git push codeberg main
```
