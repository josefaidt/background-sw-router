# Browser Extension Background Service Worker as a Router

- used [`plasmo`](https://www.plasmo.com/) to bootstrap project
- used [`itty-router`](https://github.com/kwhitley/itty-router) for routing

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
