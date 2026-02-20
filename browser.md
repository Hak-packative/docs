# How Browsers Work — Diagram

> Source: [Populating the page: how browsers work — MDN](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)

---

## Key MDN Quotes

> "Two major issues in web performance are issues having to do with latency and issues having to do with the fact that for the most part, browsers are single-threaded."

> "After the eight round trips to the server, the browser is finally able to make the request." *(DNS + TCP 3-way handshake + TLS 5 round trips)*

> "The first chunk of content is usually 14KB of data." *(TCP slow start)*

> "While the browser builds the DOM tree, this process occupies the main thread. As this happens, the preload scanner will parse through the content available and request high-priority resources like CSS, JavaScript, and web fonts."

> "Building the CSSOM is very, very fast... the total time to create the CSSOM is generally less than the time it takes for one DNS lookup."

> "JavaScript is parsed, compiled, and interpreted. The scripts are parsed into abstract syntax trees. Some browser engines take the abstract syntax trees and pass them into a compiler, outputting bytecode."

> "The browser also builds an accessibility tree that assistive devices use to parse and interpret content... Until the AOM is built, the content is not accessible to screen readers."

> "The first time the size and position of each node is determined is called layout. Subsequent recalculations of layout are called reflows."

> "To ensure smooth scrolling and animation, everything occupying the main thread, including calculating styles, along with reflow and paint, must take the browser less than 16.67ms to accomplish."

> "Promoting content into layers on the GPU (instead of the main thread on the CPU) improves paint and repaint performance. There are specific properties and elements that instantiate a layer, including `<video>` and `<canvas>`, and any element which has the CSS properties of `opacity`, a 3D `transform`, `will-change`."

> "Time to Interactive (TTI) is the measurement of how long it took from that first request which led to the DNS lookup and TCP connection to when the page is interactive — interactive being the point in time after the First Contentful Paint when the page responds to user interactions within 50ms."

---

## C1 — Overview

```mermaid
flowchart TD
  NAV["Navigation"]
  RESP["Response"]
  PARSE["Parsing"]
  RENDER["Render"]
  INTERACT["Interactivity"]

  NAV --> RESP --> PARSE --> RENDER --> INTERACT
```

---

## C2 — Detailed

```mermaid
flowchart TD
  subgraph NAV["Navigation"]
    A1["DNS Lookup"] --> A2["TCP Handshake"] --> A3["TLS Negotiation"]
  end

  subgraph RESP["Response"]
    B1["HTTP GET Request"] --> B2["TTFB"] --> B3["TCP Slow Start"]
  end

  subgraph PARSE["Parsing"]
    C1["Building the DOM Tree"] --> C3["Building the CSSOM Tree"]
    C3 --> C4["JavaScript Compilation"]
    C4 --> C5["Building the Accessibility Tree"]
    C2["Preload Scanner"] -.->|"parallel fetches"| C3
    C2 -.->|"parallel fetches"| C4
  end

  subgraph RENDER["Render"]
    D1["Style"] --> D2["Layout"] --> D3["Paint"] --> D4["Compositing"]
  end

  subgraph INTERACT["Interactivity"]
    E1["TTI — Time to Interactive"]
  end

  NAV --> RESP --> PARSE --> RENDER --> INTERACT
```

---

## C3 — Pitfalls

```mermaid
flowchart LR
  subgraph NAV["Navigation"]
    direction TB
    A1["DNS Lookup"] --> A2["TCP Handshake"] --> A3["TLS Negotiation"]
  end

  subgraph RESP["Response"]
    direction TB
    B1["HTTP GET"] --> B2["TTFB"] --> B3["TCP Slow Start"]
  end

  subgraph PARSE["Parsing"]
    direction TB
    C1["Building the DOM Tree"] --> C3["Building the CSSOM Tree"] --> C4["JavaScript Compilation"] --> C5["Building the Accessibility Tree"]
    C2["Preload Scanner"] -.->|"parallel"| C3
    C2 -.->|"parallel"| C4
  end

  subgraph RENDER["Render"]
    direction TB
    D1["Style"] --> D2["Layout"] --> D3["Paint"] --> D4["Compositing"]
  end

  subgraph INTERACT["Interactivity"]
    direction TB
    E1["TTI"]
  end

  NAV --> RESP --> PARSE --> RENDER --> INTERACT

  A1 -.-> PA1
  subgraph PA1[" "]

    A1a["too many unique origins"] --- A1b["fonts, analytics, CDN on separate hosts"]
  end

  A2 -.-> PA2
  subgraph PA2[" "]

    A2a["not using HTTP/2"]
  end

  A3 -.-> PA3
  subgraph PA3[" "]

    A3a["TLS 1.2 needs more round trips"]
  end

  B2 -.-> PB2
  subgraph PB2[" "]

    B2a["SSR blocking on slow API calls"] --- B2b["no response caching"]
  end

  B3 -.-> PB3
  subgraph PB3[" "]

    B3a["initial HTML over 14KB"] --- B3b["critical CSS not inlined"]
  end

  C1 -.-> PC1
  subgraph PC1[" "]

    C1a["sync script blocks parser"] --- C1b["deeply nested DOM"]
  end

  C2 -.-> PC2
  subgraph PC2[" "]

    C2a["LCP image not preloaded"] --- C2b["fonts without preload"]
  end

  C3 -.-> PC3
  subgraph PC3[" "]

    C3a["large or unused CSS"] --- C3b["render-blocking stylesheets"]
  end

  C4 -.-> PC4
  subgraph PC4[" "]

    C4a["large JS bundles"] --- C4b["no code splitting"]
  end

  C5 -.-> PC5
  subgraph PC5[" "]

    C5a["missing ARIA roles"] --- C5b["dynamic DOM updates"]
  end

  D1 -.-> PD1
  subgraph PD1[" "]

    D1a["overly specific CSS selectors"] --- D1b["frequent JS style mutations"]
  end

  D2 -.-> PD2
  subgraph PD2[" "]

    D2a["img without width and height"] --- D2b["layout thrashing"]
  end

  D3 -.-> PD3
  subgraph PD3[" "]

    D3a["box-shadow and filter on many elements"] --- D3b["repaint area too large"]
  end

  D4 -.-> PD4
  subgraph PD4[" "]

    D4a["animating width or height"] --- D4b["overusing will-change"]
  end

  E1 -.-> PE1
  subgraph PE1[" "]

    E1a["heavy JS after load"] --- E1b["long tasks block response"]
  end

  style PA1 fill:none,stroke:none
  style PA2 fill:none,stroke:none
  style PA3 fill:none,stroke:none
  style PB2 fill:none,stroke:none
  style PB3 fill:none,stroke:none
  style PC1 fill:none,stroke:none
  style PC2 fill:none,stroke:none
  style PC3 fill:none,stroke:none
  style PC4 fill:none,stroke:none
  style PC5 fill:none,stroke:none
  style PD1 fill:none,stroke:none
  style PD2 fill:none,stroke:none
  style PD3 fill:none,stroke:none
  style PD4 fill:none,stroke:none
  style PE1 fill:none,stroke:none

  %% Critical
  style B2a fill:#ef4444,color:#fff,stroke:none
  style B3a fill:#ef4444,color:#fff,stroke:none
  style C2a fill:#ef4444,color:#fff,stroke:none
  style C4a fill:#ef4444,color:#fff,stroke:none
  style D2a fill:#ef4444,color:#fff,stroke:none

  %% High
  style B2b fill:#f97316,color:#fff,stroke:none
  style B3b fill:#f97316,color:#fff,stroke:none
  style C1a fill:#f97316,color:#fff,stroke:none
  style C2b fill:#f97316,color:#fff,stroke:none
  style C3a fill:#f97316,color:#fff,stroke:none
  style C3b fill:#f97316,color:#fff,stroke:none
  style E1b fill:#f97316,color:#fff,stroke:none

  %% Medium
  style A1a fill:#eab308,color:#000,stroke:none
  style A1b fill:#eab308,color:#000,stroke:none
  style A2a fill:#eab308,color:#000,stroke:none
  style C1b fill:#eab308,color:#000,stroke:none
  style C4b fill:#eab308,color:#000,stroke:none
  style D1b fill:#eab308,color:#000,stroke:none
  style D2b fill:#eab308,color:#000,stroke:none
  style D4a fill:#eab308,color:#000,stroke:none
  style E1a fill:#eab308,color:#000,stroke:none

  %% Low
  style A3a fill:#94a3b8,color:#fff,stroke:none
  style C5a fill:#94a3b8,color:#fff,stroke:none
  style C5b fill:#94a3b8,color:#fff,stroke:none
  style D1a fill:#94a3b8,color:#fff,stroke:none
  style D3a fill:#94a3b8,color:#fff,stroke:none
  style D3b fill:#94a3b8,color:#fff,stroke:none
  style D4b fill:#94a3b8,color:#fff,stroke:none
```

**Legend**

🔴 Critical — direct, measurable hit on Core Web Vitals
🟠 High — significant impact, fix after critical
🟡 Medium — worth addressing once high items are resolved
⚫ Low — MDN explicitly flags these as not worth chasing first

---

## The Two Root Problems (per MDN)

| Problem | What it means | What it affects |
|---------|--------------|-----------------|
| **Latency** | Time to transmit bytes over the network. DNS + TCP + TLS = 8 round trips before a single byte of content is received. | TTFB, FCP, LCP |
| **Single-threaded main thread** | Style, Layout, Paint, and JS all run on one thread. If JS is executing, the browser cannot respond to interactions. Budget is 16.67ms per frame. | INP, CLS, jank |

## Key Optimization Levers per Phase

| Phase | Bottleneck | Lever |
|-------|-----------|-------|
| **Navigation** | DNS per unique hostname (fonts, CDN, analytics each cost 1 lookup) | `dns-prefetch`, consolidate origins |
| **Response** | 14KB TCP slow start limit — content beyond 14KB needs extra round trips | Inline critical CSS, keep SSR HTML under 14KB for first paint |
| **Parsing** | Sync `<script>` blocks HTML parser; CSSOM blocks render tree | `async` / `defer` on scripts, `<link rel="preload">` for LCP image |
| **Render — Layout** | `<img>` / `<video>` without `width` + `height` cause reflow → CLS | Always set dimensions on media elements |
| **Render — Paint** | Layers are expensive on memory; too many hurt, too few hurt scroll | Promote only scroll/animation elements via `will-change: transform` |
| **Interactivity** | JS running after load blocks TTI; heavy handlers block INP | Code-split, defer non-critical JS, keep event handlers under 50ms |

---

*Source: [MDN — Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)*
