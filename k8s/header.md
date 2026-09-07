An **HTTP header** is like a **sticky note** attached to every request/response between a client (browser/app) and a server.

---

## Simple Analogy

Imagine you send a **letter** to a friend:

| Part | What It Is |
| --- | --- |
| **Envelope address** | The URL (`/auth/login`) |
| **Letter content** | The body (form data, JSON) |
| **Sticky notes on envelope** | **Headers** — extra info about the letter |

---

## Common Headers You Already Use

| Header | What It Says | Example |
| --- | --- | --- |
| `Authorization` | "Here's my ID card" | `Bearer eyJhbG...` |
| `Content-Type` | "I'm sending JSON" | `application/json` |
| `Cookie` | "Remember me? Here's my session" | `sessionid=abc123` |
| `Origin` | "I came from this website" | `https://kapweb.tecorelabs.com` |
| `X-Api-Key` | "Here's my secret key" | `sk-123456789` |

---

## Where You See Headers in Istio

In your **VirtualService CORS policy**, you allowed specific headers:

```yaml
allowHeaders:
  - Authorization      # ← "Let requests with this sticky note pass"
  - X-api-key          # ← "And this one too"
  - Content-Type       # ← "And this one"
```

The browser sends these headers with every request. If a header is **not** in your `allowHeaders` list, the browser **blocks** the request.

---

## Two Types

| Type | Direction | Example |
| --- | --- | --- |
| **Request headers** | Client → Server | `Authorization`, `Cookie`, `X-Api-Key` |
| **Response headers** | Server → Client | `Set-Cookie`, `Cache-Control`, `Access-Control-Allow-Origin` |

---

## In One Line

> **Headers are key-value pairs that carry extra metadata with every HTTP request and response — like sticky notes telling the receiver who you are, what format you're sending, or what you're allowed to do.**