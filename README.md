# jrduncans.com

Personal landing page for <https://jrduncans.com>, linking to hosted apps and
profiles elsewhere. It's a single static page served by Cloudflare Workers
static assets; no Worker script runs.

```text
public/index.html   the page (inline CSS, no JavaScript)
public/_headers     security headers applied to every response
wrangler.jsonc      Worker name, assets directory, and custom domain
```

## Develop

```sh
npm install
npm run dev        # http://localhost:8787
```

## Deploy

Pushing to `main` deploys through Workers Builds (Cloudflare's GitHub
integration); no Cloudflare API token is stored in GitHub. `npm run deploy`
deploys from your machine after `npx wrangler login`.

The `www.jrduncans.com` → `jrduncans.com` redirect is a Cloudflare Redirect
Rule on the zone, not part of this repo (see step 5 below).

## One-time cutover from the self-hosted Caddy site

The page used to be served from the Mac mini through Cloudflare Tunnel (the
`self_hosting` repo). To move it:

1. **Connect the repo.** Cloudflare dashboard → Workers & Pages → Create →
   Import a repository → `jrduncans/jrduncans.com`. Keep the project name
   `jrduncans-com` (it must match `name` in `wrangler.jsonc`), no build
   command, and the default deploy command `npx wrangler deploy`.
2. **Check it on workers.dev.** Open
   `https://jrduncans-com.<account-subdomain>.workers.dev` and confirm the
   page renders.
3. **Free the hostnames.** Zero Trust → Networks → Tunnels → the selfhost
   tunnel → Public Hostnames: delete `jrduncans.com` and `www.jrduncans.com`.
   Then in DNS, delete their CNAME records if they're still there. The site is
   down from here until step 4 finishes deploying, usually a minute or two.
4. **Attach the custom domain.** In `wrangler.jsonc`, replace
   `"workers_dev": true` with the phase 2 lines from the comment, then push.
   The deploy creates the DNS record and certificate for `jrduncans.com`.
5. **Redirect www.** DNS → add an `AAAA` record `www` → `100::`, proxied.
   Rules → Redirect Rules → create from the "Redirect from WWW to root"
   template, with status 308 and the query string preserved.
6. **Verify.**

   ```sh
   curl -sI https://jrduncans.com/
   curl -sI https://www.jrduncans.com/some/path?x=1   # 308 to https://jrduncans.com/some/path?x=1
   ```
