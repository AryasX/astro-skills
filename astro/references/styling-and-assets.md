# Styling & Assets

## CSS

### Scoped styles (default)

```astro
<style>
  /* Only affects this component */
  h1 { color: red; }
</style>
```

### Global styles

```astro
<style is:global>
  :root { --primary: blue; }
</style>
```

### Inline styles (not scoped, not bundled)

```astro
<style is:inline>
  /* Passed directly to HTML */
</style>
```

### External CSS

```astro
---
import '../styles/global.css';
---
```

### CSS Modules

```astro
---
import styles from './Component.module.css';
---
<div class={styles.container}>...</div>
```

### Tailwind CSS

```bash
npx astro add tailwind
```

### Sass / SCSS

Write `.scss` files, import normally — Astro handles compilation.

---

## Images

### Import and use

```astro
---
import { Image, getImage } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---
<Image src={heroImage} alt="Hero" width={800} height={600} format="avif" />

<!-- Or get a src string -->
<img src={optimized.src} alt="" />
```

### Responsive images

```astro
<Image
  src={heroImage}
  alt="Hero"
  widths={[400, 800, 1200]}
  sizes="(max-width: 800px) 100vw, 800px"
  format="avif"
/>
```

### Remote images

```astro
---
import { Image } from 'astro:assets';
---
<Image
  src="https://example.com/photo.jpg"
  alt="Remote"
  width={800}
/>
<!-- Must configure allowed remote domains: -->
<!-- image: { domains: ['example.com'] } -->
```

### Image in content (Markdown)

```md
![Alt text](./image.jpg)
```

Images in `src/content/` are optimized automatically via the content layer.

### getImage() for programmatic use

```astro
---
const optimized = await getImage({
  src: heroImage,
  width: 800,
  format: 'webp',
});
---
<img src={optimized.src} alt="" />
```

### Public vs src images

- `src/assets/` — processed and optimized by Astro
- `public/` — copied as-is, no optimization

---

## Fonts API (New in v6)

Configure in astro.config.mjs:

```js
import { fontProviders } from 'astro/config';

export default defineConfig({
  fonts: [
    {
      name: 'Inter',
      cssVariable: '--font-inter',
      provider: fontProviders.fontsource(),
    },
    {
      name: 'Roboto',
      cssVariable: '--font-roboto',
      provider: fontProviders.google(),
    },
  ],
});
```

Use in components:

```astro
---
import { Font } from 'astro:assets';
---
<Font cssVariable="--font-inter" preload />
<style>
  body { font-family: var(--font-inter); }
</style>
```

Low-level access:

```ts
import { fontData, experimental_getFontFileURL } from 'astro:assets';
// fontData contains metadata for programmatic use (e.g., OG image generation)
```

### Font providers

| Provider | Import | Source |
|----------|--------|--------|
| `fontsource()` | `fontProviders.fontsource` | Self-hosted via Fontsource |
| `google()` | `fontProviders.google` | Google Fonts (downloaded & self-hosted) |
| Custom | Build via Font Provider API | Any source |
