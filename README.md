# Browser Extension Background Service Worker as a Router

Make calls to your background service worker file using `slug` as a query parameter, which transforms the request to route accordingly

- used [`plasmo`](https://www.plasmo.com/) to bootstrap project
- used [`itty-router`](https://github.com/kwhitley/itty-router) for routing

1. `pnpm install`
2. `pnpm build`
3. add the extension to Chrome
4. navigate to `<background-sw-path>?slug=%2Fhello` (e.g. `chrome-extension://ihhpgnoehkgbhjlcgpmffcbiajlaokjk/background.5fadff2f.js?slug=%2Fhello`)
5. observe `world` is returned

```ts
import { Router } from "itty-router"

export function createFetchRouter() {
  const router = Router()
  router.get("/hello", () => new Response("world"))
  router.post("/hello", () => new Response("world"))
  router.all("*", () => new Response("Not Found", { status: 404 }))
  return router
}

export async function handleFetch(event: FetchEvent) {
  const router = createFetchRouter()
  const url = new URL(event.request.url)
  const slug = url.searchParams.get("slug")
  // exit early if slug is not present
  if (!slug) return
  // construct a new url passing the slug as the path
  const newURL = new URL(slug, url.origin)
  // construct a new request with the new url
  const newRequest = new Request(newURL, {
    method: event.request.method,
    headers: event.request.headers,
    body: event.request.body
  })
  // call router.handle with the new request
  const response = router.handle(newRequest)
  // wait until promise router.handle resolves
  event.waitUntil(response)
  // return response from itty-router
  event.respondWith(response)
}

self.addEventListener("fetch", handleFetch)
```
