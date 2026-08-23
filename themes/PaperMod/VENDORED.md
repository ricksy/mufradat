# PaperMod, vendored

This theme was a git submodule pinned at
`3bb0ca281fd17eff8e3489011a444f326d7c4c72`
(<https://github.com/adityatelange/hugo-PaperMod>). It is now committed
directly into this repository.

**Why.** This repository exists so the blog survives losing the machine that
serves it. A submodule is a promise that a second repository, on a host
nobody here controls, will still be there and will still have that commit.
`git clone && hugo` should need nothing else and reach nowhere else.

The cost is that theme updates are manual, which for a theme changed roughly
never is the cheaper side of the trade.

## Updating it

```bash
git clone --depth 1 https://github.com/adityatelange/hugo-PaperMod /tmp/papermod
rm -rf themes/PaperMod && mv /tmp/papermod themes/PaperMod
rm -rf themes/PaperMod/.git
# then rebuild, compare the output, and commit -- noting the new commit here
```
