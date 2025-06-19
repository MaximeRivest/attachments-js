# attachments-js

> **Status — WIP (pre‑alpha)**
> Port of the Attachments file‑to‑LLM funnel to **TypeScript / JavaScript**.
> API surface is being defined; nothing executes yet.

---

## Install (coming soon)

```bash
npm add attachments-js
# or
yarn add attachments-js
# or
pnpm add attachments-js
```

---

## One‑liner usage

```ts
import { attachments } from 'attachments-js';

const ctx = await attachments(
  'report.pdf',
  'https://example.com/slides.pptx[3-5]',
);

console.log(String(ctx));   // → model‑ready text
console.log(ctx.images);    // → base64 PNGs
```

`attachments-js` will:

* extract clean text from every supported file format
* convert screenshots and inline images to **base64**
* preserve structure (pages, slides, rows, …)
* stream everything asynchronously

---

## Composable pipeline DSL (paren‑pipe style)

Advanced users can stitch their own flows with a zero‑dependency **paren‑pipe** helper. Import the blocks you need **plus** the toolkit:

```ts
import {
  pipe,      // paren‑pipe core
  _,         // placeholder
  attach,
  load,
  modify,
  present,
  refine,
  adapt,
} from 'attachments-js/pipeline';
```

The canonical order is still:

`attach → load → modify → present → refine → adapt`

### ✨ Tailor‑made document pipeline

```ts
const result = await pipe(
  attach('document.pdf[pages:1-5]')
    )(
  load.pdfToPdfPlumber
    )(
  modify.pages
    )(
  present.markdown
    )(
  present.images
    )(
  refine.addHeaders
    )(
  refine.truncate
    )(
  adapt.claude, 'Analyse this content'
)();          // ← unwrap final value
```

### 🌐 Web‑scraping pipeline

```ts
const title = await pipe(
  attach('https://en.wikipedia.org/wiki/Llama[select:title]')
    )(
  load.urlToDom
    )(
  modify.select
    )(
  present.text
)();
```

### 🔁 Reusable processor

```ts
type CsvAnalyzer = (src: string) => Promise<ReturnType<typeof attach>>;

const csvAnalyzer: CsvAnalyzer = (source) => pipe(
  attach(source)
    )(
  load.csvToPandas
    )(
  modify.limit
    )(
  present.head
    )(
  present.summary
    )(
  present.metadata
    )(
  refine.addHeaders
)();

const analysisCtx = await csvAnalyzer('data.csv[limit:1000]');
const analysis    = await analysisCtx.claude('What patterns do you see?');
```

> **Heads‑up**: function names and import paths may tweak before 1.0—snippets show the intended shape, not the final API contract.

---

## Roadmap

1. **MVP loaders** – PDF · TXT · CSV · Web URL
2. **Type‑safe pipeline helpers** + functional chaining
3. **OpenAI / Claude adapters**
4. **ESM + CJS builds**, declaration bundles, and full tests

👀 **Contributions welcome!** Open an issue with ideas or grab one of the TODOs once the skeleton lands.

---

## License

MIT — same as the original Attachments project.
