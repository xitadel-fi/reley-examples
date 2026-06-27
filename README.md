# reley-examples

Catalog + downloadable `.reley.zip` snapshots for the Reley Community page
(https://reley.xyz/community).

## Layout

```
.
├── packages.json   # catalog index served to the landing page
├── packages/       # *.reley.zip files referenced by packages.json
├── projects/       # unzipped Reley project source (one folder per package)
└── thumbs/         # PNG thumbnails referenced by packages.json
```

`projects/` mirrors the contents of each zip — the same `<slug>/.reley.json`
+ `<slug>/.reley/` tree the Reley app reads. Diff-friendly: changes to a
program's IDL, an account blob, or a manifest field show up in git history
without having to crack open the archive. The zip is just `cd projects && zip -rq ../packages/<slug>.reley.zip <slug>`.

## Hosting

Two options:

1. **GitHub raw (default)** — every `zipUrl` / `thumbnail` in `packages.json`
   points at `raw.githubusercontent.com/hoangtuanictvn/reley-examples/main/…`.
   No extra infrastructure. Set
   `NEXT_PUBLIC_COMMUNITY_INDEX_URL=https://raw.githubusercontent.com/hoangtuanictvn/reley-examples/main/packages.json`
   in the landing `.env`.

2. **R2 / S3** — upload `packages/`, `thumbs/`, and `packages.json` to your
   bucket. Rewrite the URLs in `packages.json` to your CDN, then set
   `NEXT_PUBLIC_COMMUNITY_INDEX_URL` to the bucket URL.

## Add a package

1. In the Reley desktop app, build a project: clone the program(s), patch any
   PDA state, snapshot a working sandbox.
2. **File → Export Project as .zip…** → save into `packages/`.
3. Drop a 16:9 PNG thumbnail into `thumbs/`.
4. Append an entry to `packages.json`:

   ```json
   {
     "id": "<slug>",
     "categoryId": "lending | dex | perps | stablecoin | nft | infra",
     "title": "...",
     "description": "...",
     "thumbnail": "https://raw.githubusercontent.com/hoangtuanictvn/reley-examples/main/thumbs/<file>.png",
     "zipUrl":    "https://raw.githubusercontent.com/hoangtuanictvn/reley-examples/main/packages/<slug>.reley.zip",
     "author": "nhannt",
     "size": <bytes>,
     "sha256": "<hex>",
     "programIds": ["..."],
     "network": "mainnet-beta",
     "tags": ["..."],
     "createdAt": "YYYY-MM-DD"
   }
   ```

5. Compute size + sha256:

   ```bash
   wc -c <  packages/<slug>.reley.zip          # size
   shasum -a 256 packages/<slug>.reley.zip     # sha256
   ```

6. Bump `updatedAt` at the top of `packages.json`. Commit + push.

The landing page revalidates the catalog every 60 s.

## Schema reference

- `categories[].id` — referenced by `packages[].categoryId`.
- `categories[].order` — left-sidebar sort key (lower = top).
- `packages[].network` — `mainnet-beta | devnet | testnet | custom`.
- `packages[].programIds[]` — surfaced in the card as a quick-scan badge.

## Current entries

| id                          | category   | program                                                                  | zip size |
| --------------------------- | ---------- | ------------------------------------------------------------------------ | -------- |
| kamino-lend-main            | lending    | `KLend2g3cP87fffoy8q1mQqGKjrxjC8boSyAYavgmjD`                            | 558 KB   |
| meteora-damm-v2             | dex        | `cpamdpZCGKUy5JxQXB4dcpGPiikHawvSWAd6mEn1sGG`                            | 351 KB   |
| circle-usdc-mint            | stablecoin | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` + USDC mint account        | 5 KB     |
| metaplex-token-metadata     | nft        | `metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s`                            | 178 KB   |

Each zip is a real cloned snapshot — ELF + Anchor IDL (when published
on-chain) + at least one key account where applicable. Open in Reley and
the program is immediately runnable in the LiteSVM sandbox.
