# Visual Effects — Seventh Sky Store

#visual #frontend #webgl

---

## Overview

Immersive visual experience using WebGL (OGL), Framer Motion, Lenis smooth scroll, and custom cursor system.

---

## WebGL Galaxy Effect

### Component: `Galaxy.tsx`

```typescript
// src/components/collections/Galaxy.tsx
// OGL + custom GLSL fragment shader
```

### Shader Features

| Feature | Uniform | Description |
|---------|---------|-------------|
| Star layers | `uDensity`, `NUM_LAYER` | 4 layers of stars |
| Twinkling | `uTwinkleIntensity` | Per-star brightness oscillation |
| Color shift | `uHueShift`, `uSaturation` | HSV color manipulation |
| Mouse repulsion | `uMouseRepulsion`, `uRepulsionStrength` | Stars push away from cursor |
| Auto rotation | `uRotationSpeed` | Continuous slow rotation |
| Glow rays | `uGlowIntensity` | Cross-shaped star rays |
| Transparency | `uTransparent` | Alpha blending for overlay |

### GLSL Highlights

```glsl
// Star generation with hash-based positioning
float Hash21(vec2 p) { ... }

// HSV color space for star colors
vec3 hsv2rgb(vec3 c) { ... }

// Star shape with rays
float Star(vec2 uv, float flare) { ... }

// Multi-layer parallax depth
for (float i = 0.0; i < 1.0; i += 1.0 / NUM_LAYER) {
    float depth = fract(i + uStarSpeed * uSpeed);
    float scale = mix(20.0 * uDensity, 0.5 * uDensity, depth);
    col += StarLayer(uv * scale + i * 453.32) * fade;
}
```

### Props

```typescript
interface GalaxyProps {
  focal?: [number, number];        // [0.5, 0.5] center
  rotation?: [number, number];     // [1.0, 0.0] no skew
  starSpeed?: number;              // 0.5
  density?: number;                // 1
  hueShift?: number;               // 140 (blue-green)
  disableAnimation?: boolean;      // false
  speed?: number;                  // 1.0
  mouseInteraction?: boolean;      // true
  glowIntensity?: number;          // 0.3
  saturation?: number;             // 0.0
  mouseRepulsion?: boolean;        // true
  twinkleIntensity?: number;       // 0.3
  rotationSpeed?: number;          // 0.1
  repulsionStrength?: number;      // 2
  autoCenterRepulsion?: number;    // 0
  transparent?: boolean;           // true
}
```

### Usage

```tsx
// app/(site)/collections/page.tsx
<Galaxy
  transparent={true}
  mouseRepulsion={true}
  density={1.2}
  hueShift={140}
  className="fixed inset-0 -z-10"
/>
```

---

## Flying Posters

### Component: `FlyingPosters.tsx`

```typescript
// src/components/collections/FlyingPosters.tsx
// Framer Motion 3D transforms
```

### Features
- 3D rotating posters in perspective space
- Staggered animation delays
- Hover pause/resume
- Responsive sizing

---

## Custom Cursor System

### Component: `CustomCursor.tsx`

```typescript
// src/components/layout/CustomCursor.tsx
// Two modes: 'explore' (circle) and 'pointer' (dot)
```

### Modes

| Mode | Trigger | Visual |
|------|---------|--------|
| `explore` | Default | Large circle with trail |
| `pointer` | Hover links/buttons | Small dot |

### Implementation

```typescript
const [cursorMode, setCursorMode] = useState<'explore' | 'pointer'>('explore');

useEffect(() => {
  const handleMouseMove = (e: MouseEvent) => {
    setCursorPos({ x: e.clientX, y: e.clientY });
  };
  
  const handleMouseOver = (e: MouseEvent) => {
    if (e.target.matches('a, button, [role="button"]')) {
      setCursorMode('pointer');
    }
  };
  
  const handleMouseOut = () => setCursorMode('explore');
  
  window.addEventListener('mousemove', handleMouseMove);
  document.addEventListener('mouseover', handleMouseOver);
  document.addEventListener('mouseout', handleMouseOut);
  
  return () => { ... };
}, []);
```

---

## Smooth Scrolling (Lenis)

### Setup

```typescript
// src/components/ui/SmoothScroll.tsx
import { Lenis } from 'lenis';

useEffect(() => {
  const lenis = new Lenis({
    duration: 1.2,
    easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
    smoothWheel: true,
    wheelMultiplier: 1,
  });
  
  function raf(time: number) {
    lenis.raf(time);
    requestAnimationFrame(raf);
  }
  requestAnimationFrame(raf);
  
  return () => lenis.destroy();
}, []);
```

---

## Framer Motion Animations

### Page Transitions

```typescript
// Layout components
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.3 }}
>
  {children}
</motion.div>
```

### Hover Effects

```tsx
// ProductCard.tsx
<motion.div
  whileHover={{ y: -8, scale: 1.02 }}
  transition={{ duration: 0.2 }}
>
  {children}
</motion.div>
```

### Staggered Lists

```tsx
// ProductsView.tsx
<motion.div
  initial="hidden"
  animate="show"
  variants={{
    hidden: { opacity: 0 },
    show: { transition: { staggerChildren: 0.1 } }
  }}
>
  {products.map(p => (
    <motion.div key={p.id} variants={{ hidden: { opacity: 0, y: 20 }, show: { opacity: 1, y: 0 } }}>
      <ProductCard product={p} />
    </motion.div>
  ))}
</motion.div>
```

---

## Hero Carousel

### Component: `HeroCarousel.tsx`

```typescript
// src/components/home/HeroCarousel.tsx
// embla-carousel-react
```

### Features
- Auto-play (5s interval)
- Touch/swipe support
- Parallax background
- Keyboard navigation
- Dot indicators

---

## Parallax Hero

### Component: `HeroSection.tsx`

```tsx
// Parallax effect on scroll
<motion.div
  style={{ transform: `translateY(${scrollY * 0.3}px)` }}
>
  <HeroCarousel />
</motion.div>
```

---

## Click Ripple Effect

### Component: `ClickRipple.tsx`

```typescript
// src/components/ui/ClickRipple.tsx
// CSS-based ripple on click
```

### Usage

```tsx
<ClickRipple>
  <button>Click me</button>
</ClickRipple>
```

---

## Performance Considerations

| Effect | Optimization |
|--------|--------------|
| WebGL Galaxy | `requestAnimationFrame`, cleanup on unmount, `WEBGL_lose_context` |
| Framer Motion | `will-change`, `transform` only, reduce layout thrashing |
| Lenis | Passive event listeners, RAF loop |
| Custom Cursor | CSS `transform`, `pointer-events: none` |

---

## Accessibility

| Feature | Implementation |
|---------|----------------|
| Reduced motion | `prefers-reduced-motion` media query disables animations |
| WebGL fallback | Static background image if WebGL unavailable |
| Focus visible | Custom cursor hides on keyboard navigation |
| ARIA labels | All interactive elements labeled |

---

## Related Notes

- [[Frontend Structure#Collections Components]] — Galaxy, FlyingPosters
- [[Frontend Structure#Layout Components]] — CustomCursor, Navbar
- [[Architecture#Frontend Architecture]] — Component organization
- [[Product Catalog]] — ProductCard hover effects