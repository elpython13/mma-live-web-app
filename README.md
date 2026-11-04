# mma-live-web-app

Web app for www.mma-live.com: the one-page marketing site for **MMA Live** at <https://mma-live.com/> (`www` redirects to the apex).

Plain static HTML and CSS with no build step. The only JavaScript is a small inline script that switches the store buttons on.

```
index.html          the page (store-link config is at the bottom, in STORE_LINKS)
styles.css          dark theme using the app's palette (mma-live-frontend app/theme/colors.ts)
404.html            not-found page (GitHub Pages serves it automatically)
assets/             app icon (copied from mma-live-frontend store-assets/icons), apple-touch icon, OG image
assets/screens/     Google Play phone screenshots as WebP, 360w and 720w (source: /workspace/mma-design/play-store/phone/listing/01-08, the Play phone set with real fighter photos in 02 and 07)
favicon.ico
robots.txt, sitemap.xml
CNAME               mma-live.com (custom domain for GitHub Pages)
.nojekyll           serve files as-is, skip Jekyll
```

## Preview locally

```sh
python3 -m http.server 8000   # then open http://localhost:8000/
```

## Store buttons (going live)

Neither listing is live yet, so both buttons show as non-clickable **"Coming soon"** badges.
The only place to change is `STORE_LINKS` at the bottom of `index.html`:

```js
var STORE_LINKS = {
  ios:     { url: 'https://apps.apple.com/app/idXXXXXXXXXX', live: false, ... }, // TODO: real App Store ID
  android: { url: 'https://play.google.com/store/apps/details?id=com.mmaliveapp', live: false, ... }
};
```

- **Google Play:** once the listing is public, change `live: false` to `live: true` on the `android` line.
- **App Store:** replace `idXXXXXXXXXX` with the real App Store ID, then set `live: true` on the `ios` line.

The buttons are plain text buttons in the store style. The official Apple and Google badge artwork is left out on purpose because each comes with its own usage guidelines. Swap the badges in at launch if you want them.

## Hosting: GitHub Pages (free, $0/mo)

The repo is **public**, so the free GitHub plan can host it on Pages. Branch-based Pages from `main` / root is used, with no Actions workflow and no build.

### 1. Enable Pages (after this PR is merged into `main`)

1. Go to **Settings → Pages** at <https://github.com/elpython13/mma-live-web-app/settings/pages>.
2. Under **Build and deployment**, set **Source** to *Deploy from a branch*, **Branch** to `main` and the folder to `/ (root)`, then click **Save**.
3. Under **Custom domain**, enter `mma-live.com` and click **Save**. The `CNAME` file already contains this. GitHub runs a DNS check, which fails until step 2 is done, and that's expected.

### 2. DNS records at the registrar

`mma-live.com` uses Namecheap DNS (`dns1/dns2.registrar-servers.com`), so this is done under **Namecheap → Domain List → mma-live.com → Advanced DNS**.
Right now there are no apex or `www` records and no wildcard. `api.mma-live.com` is a CNAME to the AWS load balancer.

Add exactly these records:

| Type  | Host | Value                   |
|-------|------|-------------------------|
| A     | @    | 185.199.108.153         |
| A     | @    | 185.199.109.153         |
| A     | @    | 185.199.110.153         |
| A     | @    | 185.199.111.153         |
| AAAA  | @    | 2606:50c0:8000::153     |
| AAAA  | @    | 2606:50c0:8001::153     |
| AAAA  | @    | 2606:50c0:8002::153     |
| AAAA  | @    | 2606:50c0:8003::153     |
| CNAME | www  | elpython13.github.io.   |

The TTL can stay on Automatic. Remove any Namecheap default "URL Redirect" or parking record on `@` or `www`, because it would conflict.

> **Leave `api.mma-live.com` alone.** It's a separate `CNAME` record for `api` that points to the AWS load balancer for the app's backend. Don't edit or delete it, and **don't add a wildcard (`*`) record**. Only the `@` and `www` records above are new.

GitHub automatically redirects `www.mma-live.com` to `mma-live.com`.

### 3. Turn on HTTPS

DNS can take anywhere from a few minutes to a few hours to propagate. Once the custom-domain check in **Settings → Pages** shows a green tick and the certificate has been issued, tick **Enforce HTTPS**.

### 4. Optional: verify the domain (recommended)

Verifying the domain stops anyone else's GitHub Pages site from claiming `mma-live.com`.

1. Go to **GitHub → your profile Settings → Pages → Add a domain** (<https://github.com/settings/pages>) and enter `mma-live.com`.
2. GitHub gives you a TXT record, with a host like `_github-pages-challenge-elpython13` and a random value. Add it in Namecheap Advanced DNS.
3. Click **Verify**. The TXT record can stay in place.

### Check it worked

```sh
dig +short mma-live.com A          # four 185.199.10x.153 addresses
dig +short www.mma-live.com CNAME  # elpython13.github.io.
dig +short api.mma-live.com        # still the AWS load balancer, unchanged
curl -I https://mma-live.com/      # HTTP/2 200 from GitHub
```

### Alternative if the repo ever goes private: Cloudflare Pages (free)

GitHub Pages on the free plan only works with public repos. If the repo is made private, either make it public again or use Cloudflare Pages, which is free and supports private repos:

1. Sign up at <https://dash.cloudflare.com> and go to **Workers & Pages → Create → Pages → Connect to Git**. Authorise GitHub for `elpython13/mma-live-web-app` only.
2. Set the production branch to `main`, the framework preset to *None*, leave the build command empty and set the output directory to `/`.
3. Under **Custom domains**, add `mma-live.com` and `www.mma-live.com`. Because DNS stays at Namecheap, Cloudflare asks for a `CNAME www → <project>.pages.dev`. Apex domains on Cloudflare Pages need the zone moved onto Cloudflare nameservers. If you do that, re-create the existing `api` CNAME exactly as it is now, with **DNS only** (grey cloud), before switching nameservers.
4. Delete the `CNAME` file, which only GitHub Pages uses. The `404.html` keeps working.
