# Fork maintenance notes

Recorded maintenance baseline: 2026-10-24. This is a snapshot, not a promise that upstream or advisory data is still current.

- Upstream: [https://github.com/DayuanJiang/next-ai-draw-io](https://github.com/DayuanJiang/next-ai-draw-io)
- Integrated upstream commit: [`2788253c65a2df0796cdb6954785b96c12b2fa00`](https://github.com/DayuanJiang/next-ai-draw-io/commit/2788253c65a2df0796cdb6954785b96c12b2fa00)
- Verified maintenance code commit: `aff8eeb5f9abd71a50d9432a19fb61031561f7f6`
- Deployment branch: `main`

This fork retains deployment-specific privacy/indexing settings and function limits while incorporating upstream UI, editor, MCP, and desktop work.

## Preserve these fork differences

| Paths | Intent |
| --- | --- |
| `app/[lang]/layout.tsx, app/robots.ts, app/sitemap.ts, lib/seo.ts` | Keep the configured deployment canonical host, noindex metadata, crawler-readable policy, and empty sitemap together. Robots rules are not access control. |
| `vercel.json` | Keep noindex headers and the existing per-route memory/duration settings. |
| `.github/workflows/docker-build.yml` | Keep the fork-specific image publication behavior; do not accidentally publish using upstream ownership/credentials. |
| `package-lock.json; packages/mcp-server/package-lock.json` | Treat the web/desktop/tooling root and MCP server as two independent npm lock scopes. Electron is in the root lock; there is no separate desktop lock. |

See the upstream [offline deployment guide](docs/en/offline-deployment.md) for general options. This fork document takes precedence for its preserved deployment delta.

## Dependency and runtime baseline

Use npm and the committed lockfiles. The recorded application uses Next 16.4.2, React 19.3.1 and Electron 42.11.18; root and MCP installs must be checked separately. The web CI uses Node 24; the upstream MCP CI job has its own Node setting and is not evidence that every job pins the same runtime.

The root locks patched Undici 6.29.1 and 7.30.1, Zod 4.6.7 and fast-uri 3.1.9. The MCP lock also refreshes body-parser, ip-address, Zod, fast-uri, Vitest and Vite within compatible declarations. This includes a same-major Vite minor update, not only patch updates. The existing OpenRouter AI SDK override is a compatibility policy, not a claim that it was introduced for the Undici advisory.

The manifest and committed lockfile are authoritative for exact dependency resolution. Review both direct and transitive changes; avoid mixing package managers.

## Resolution policy and removal conditions

| Scoped selection | Current declaration | Reason |
| --- | --- | --- |
| `@openrouter/ai-sdk-provider > ai` | `^6.0.1` | Retain the existing AI SDK compatibility selection. |
| `@node-minify/core > glob` | `^10.5.0` | Patch the minifier file-matching dependency in the compatible major. |
| `@tailwindcss/typography > postcss-selector-parser` | `^7.1.6` | Patch the CSS selector parser dependency. |
| `miniflare@5.20261006.1-alpha > undici` | `7.30.1` | Replace an exact vulnerable development-tool dependency with Undici 7.30.1. |

For every entry above, remove or narrow it only after the parent dependency naturally resolves a patched compatible version, a fresh lockfile proves the vulnerable path is absent, the affected tests pass, and the relevant application/docs/MCP build passes. Change one dependency family at a time. Preserve parent-specific major-version constraints. Do not use audit suppression or a fabricated package version as a substitute for a fix.

## Known limitations at this baseline

Both npm audit scopes returned zero findings at this baseline. Web CI, MCP packed-package browser tests, and Vercel builds were verified separately. Electron GUI, installers and signing were not exercised; a web build does not validate those desktop artifacts.

## Future upstream sync and release

1. Compare upstream changes against the preserved paths above and keep unrelated local work intact.
2. Use the repository package manager and lockfile; review scoped overrides and affected regression tests.
3. Keep source checks, CI, provider build readiness, live-page checks, and external-service behavior as separate evidence.
4. Inspect added files and diffs for credentials before publishing. Store real credentials, deployment addresses for private services, user data and environment values outside Git.
5. After the reviewed commit is deployed, verify the intended domain and indexing policy. Remove only task-created branches that are already merged; preserve unrelated and unmerged branches.
