# Flim public site (`website/`)

Decision **011** — static public face of [flim.me](https://flim.me): legal pages, help stubs, OIDC assets, AASA, and OAuth redirect paths.

## Stack

- [Eleventy](https://www.11ty.dev/) → `website/_site`
- Deploy: [statichost.eu](https://www.statichost.eu/) via meta-root [`statichost.yml`](../statichost.yml)

## Local build

```bash
cd website
npm ci
npx @11ty/eleventy
# or: npm run serve
```

Output is `website/_site` (gitignored).

## Human setup (not automated here)

1. Point DNS for `flim.me` (and optional `www`) at statichost.eu.
2. Create the site in the statichost dashboard and link this meta repository.
3. Add a push webhook to `https://builder.statichost.eu/<site-name>` (see their docs).
4. Counsel review of privacy / terms before store submission if required.

`src/_headers` sets `Content-Type: application/json` for
`/.well-known/apple-app-site-association` (required for OIDC associated domains).

Deploy tokens and analytics keys stay out of git.
