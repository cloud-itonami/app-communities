# Operator quickstart — app-communities

**This repository contains no source code.** 30 of its 33 tracked files are hashed
SvelteKit build output; the other three are `README.edn`, `migration.edn` and an empty
deploy-state file. Nothing here can be built, tested, or run, and nothing in the
workspace says it is deployed anywhere.

That is the whole finding, and the rest of this document is how each half of it was
measured, because "there is no source" is exactly the kind of claim that deserves
commands rather than assertion.

Steps marked ✅ were run on 2026-08-16.

---

## 1. ✅ No source, and the artifacts are five months older than the repository

```bash
git ls-files | wc -l                                  # 33
git ls-files | grep -cE '\.(ts|svelte|cljc?|clj|kotoba|py|rs)$'   # 0
git ls-files | grep -v 'static/_app'
#   README.edn
#   migration.edn
#   wasm/.deploy-state.json
```

Everything else lives under
`wasm/etzhayyim-performer-sys-etzhayyim-actors-pba7d22f-org-community-<name>-<nanoid>/static/_app/`
for two community surfaces — `fujian-nanyin-music` and `scotland-highland-games` —
in the content-hashed layout SvelteKit's adapter emits.

The build stamps date them:

```bash
cat wasm/*/static/_app/version.json
#   {"version":"1773795649553"}   fujian-nanyin-music      2026-03-18T01:00:49Z
#   {"version":"1773795303088"}   scotland-highland-games  2026-03-18T00:55:03Z
git log --format='%ci %s' | tail -1
#   2026-07-20 02:12:34 +0900 chore: extract communities app from root
```

The two were built 346 seconds apart in one session on **18 March**, and this
repository's own first commit is from **20 July** — four months later. The artifacts
were not produced here and were already old when they arrived.

## 2. ✅ The provenance chain records artifacts all the way back

`migration.edn` names where the tree came from, so the claim is checkable rather than
inferred:

```bash
cat migration.edn
#   :source {:repository "etzhayyim/root"
#            :revision "7a08afb44a426568ceb8137f8b15881d8a2904e2"
#            :path "60-apps/etzhayyim-project-communities"
#            :tracked-files 31 :bytes 412830}

git -C <root>/orgs/etzhayyim/root ls-tree -r --name-only \
    7a08afb44a426568ceb8137f8b15881d8a2904e2 -- 60-apps/etzhayyim-project-communities
```

The recorded source path, at the recorded revision, holds **the same `static/_app`
artifacts and nothing else**. So this is not a case of an extraction leaving source
behind: there was no source at the place the extraction copied from either. The
`tracked-files 31` in the record plus the two files `:identity/:allowed-additions`
permits (`README.edn`, `migration.edn`) accounts for all 33 files here.

## 3. ✅ Nothing in the workspace can rebuild them

The directory name embeds the generator and its nanoid, so its siblings can be
counted:

```bash
python3 - <<'EOF'
import os,glob,collections
dirs=[d for d in glob.glob('orgs/*/*/**/etzhayyim-performer-sys-*pba7d22f*',recursive=True) if os.path.isdir(d)]
c=collections.Counter()
for d in dirs:
    c[('pkg' if os.path.exists(os.path.join(d,'package.json')) else '-')
      + '/' + ('src' if glob.glob(os.path.join(d,'src','**','*'),recursive=True) else '-')
      + '/' + ('static' if os.path.isdir(os.path.join(d,'static')) else '-')] += 1
print(len(dirs), dict(c))
EOF
#   208 {'pkg/-/static': 202, '-/-/static': 6}
```

**208 directories carry this generator's nanoid. 202 have a `package.json`; the six
that do not are these two surfaces, in three copies** — this repository,
`etzhayyim/com-etzhayyim-app-communities` (the pre-rename checkout, which is not in
`manifest/west.yml`), and the `etzhayyim/root` path above.

Two things follow, and the second matters more:

- this repository is missing what 202 of its 208 siblings have.
- **not one of the 208 has a `src/`**, and a sibling's manifest is
  `{"scripts": {"test": "echo \"no tests\""}, "dependencies": {"@etzhayyim/kotodama-host-sdk": …}}`
  — no build script. So even the 202 cannot regenerate their own `static/_app`. The
  generator's output is committed; the generator's input is not in this workspace.

Anyone asked to change what these two pages say has nothing to change. The correct
first step is to find the generator, not to hand-edit hashed chunks — editing
`nodes/2.C5554d_r.js` would break the content hash its own filename asserts.

## 4. ✅ Nothing says it is deployed

Three independent places that would record a deployment, all empty:

```bash
cat wasm/.deploy-state.json
#   {"version": 1, "apps": {}}
git ls-files | grep -c wrangler          # 0
```

and the workspace's own surface index has no row for it:

```bash
nbb --classpath ".:scripts/nbb_compat" -e '…'   # 0 rows mentioning "communities"
#   in 90-docs/surface/surface.datoms.edn
```

So the file whose purpose is to record what is deployed records nothing, there is no
`wrangler.jsonc` to declare routes or a worker name, and the index that answers "which
host serves which path" does not mention this app. That is three absences agreeing —
not proof it was never deployed, but nothing here claims it is.

## 5. ✅ What the two surfaces share

```bash
A=wasm/*fujian-nanyin-music*; B=wasm/*scotland-highland-games*
for f in $(cd $A && find . -type f | sort); do
  [ -f "$B/$f" ] && cmp -s "$A/$f" "$B/$f" && echo "same $f" || echo "only-in-A $f"
done
#   9 files identical in both — the framework chunks and the stylesheet
#   5 present only in A: one chunk, entry/app, entry/start, nodes/1, nodes/2
#   version.json differs
```

The nine shared files are byte-identical, so the two surfaces are the same framework
build with different page code. **The five differing files are named by content hash,
so a different name already means different content** — comparing their bytes adds
nothing, and a first attempt at this comparison reported them as "differs" when
`cmp` had simply failed on a path absent in B. The real statement is 9 shared, 5
app-specific, 1 stamp.

## 6. What the maturity instrument sees here ✅

```
· orgs/cloud-itonami/app-communities  own=0.049  axis-docs=0bp → +2500bp
    ⚠ README が .md ではないので docs の README 成分は 0（README.edn 等が 1 件）
    ⚠ taxonomy に :repo/kind の行が無い → :default の重みで採点されている
```

Both warnings are about the instrument, not this repository: `README.edn` declares
`:canonical-metadata :edn`, so EDN is deliberately canonical here while the score reads
`README.md`, and there is no row in `manifest/repo-taxonomy.edn`, so the score is
computed against a guessed weight profile and this `own` is not comparable to a
repository whose kind is known (ADR-2608052000).

Worth adding, since a score can only reward what it can count: `axis-substrate` reads
0 because there is no `src/`, and that 0 is **correct here** — the two nested `static/`
trees are output, not substrate. This is one repository where a low score is the honest
one.
