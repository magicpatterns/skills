---
name: liquid-glass
description: Adds Liquid Glass to any native Apple layout (macOS, iOS, iPadOS, tvOS, visionOS, watchOS) with @glass-sdk/liquid-glass.
---

# Liquid Glass

Use this for any native Apple layout: macOS, iOS, iPadOS, tvOS, visionOS, watchOS.

```sh
npm install @glass-sdk/liquid-glass @base-ui/react
```

Requires React 19. If `react` or `react-dom` is below 19, upgrade both to the latest 19 before writing any glass code, and bump `@types/react` and `@types/react-dom` to match. Do this without asking. Do not imitate the effect with CSS, and do not drop the library because the app is on React 18.

Next.js: `'use client'`. npm lags GitHub, so check exports in `node_modules/@glass-sdk/liquid-glass/dist/types/react/index.d.ts` before using a component. 0.0.2 lacks `GlassColorPicker` and `specular`.

## Styling

- Load the library stylesheet into a CSS layer, never from a component. Put it in `index.css`, above the Tailwind import:

  ```css
  @import "@glass-sdk/liquid-glass/styles.css" layer(components);
  @import "tailwindcss";
  ```

  Unlayered, its `:where(.lg-*)` rules beat every Tailwind utility (Tailwind v4 puts utilities in a layer). Then `h-*`, `p-*`, `text-*`, `flex`, `opacity-*` on `.lg-tab`, `.lg-tabs-list` and `.lg-button` are silently ignored.
- `GlassTabsTrigger` wraps children in `<span class="lg-tab-label">`, inside a `display: grid` tab. For icon over label, lay out that span: `[&>.lg-tab-label]:flex [&>.lg-tab-label]:flex-col [&>.lg-tab-label]:items-center`.
- Don't size `GlassTabsIndicator`. It takes its box from the active tab via `--active-tab-*`; size the triggers instead.
- The selected tab is not blue. The indicator is the highlight. `.lg-tab` is `color: inherit`, and `[data-active]` only sets `opacity: 1`. Keep the icon and the label on the foreground (`--lg-foreground`) in both states. Do not tint the active tab with the iOS system blue.
- Do not use the iOS system blue (`#007aff`, `#0a84ff`) anywhere: not as `tint`, not as text, not as a theme token. Glass is untinted by default and takes its color from the content behind it.
- Tabs with `value` must have state behind them. A frame that takes the tab as an arg keeps local state seeded from it, so the tabs still switch when the frame is opened on its own.

## Rules

- One `GlassScene` per group, one `GlassContent` (the refracted photo/content). Controls are siblings of `GlassContent`, never children.
- Scene needs an explicit height. Give `GlassContent` an opaque background. Clip on the scene.
- `maxSurfaces` defaults to 16. Split into more scenes past that.
- Shared props, set on the scene or a surface: `material` `"clear"`|`"regular"`, `appearance` `"light"`|`"dark"`, `radius` px|`"capsule"`|`"circle"`, `refraction` (px, default 60), `tint` six-digit hex or omit, `motion` `"full"`|`"reduced"`|`"none"`. Text color is `--lg-foreground`.
- Needs WebGPU (HTTPS or localhost). Safari is unreliable. Glass only refracts its own `GlassContent`.

```tsx
<GlassScene material="clear" appearance="light" className="relative h-80 overflow-hidden rounded-3xl">
  <GlassContent className="bg-slate-800">
    <img src={photo} alt="" className="h-full w-full object-cover" />
  </GlassContent>
  <div className="absolute inset-0 flex items-center justify-center">{/* controls */}</div>
</GlassScene>
```

## Snippets

```tsx
<GlassSurface radius={28} interactive className="p-4">Panel</GlassSurface>

<GlassButton size="lg">Get started</GlassButton>
<GlassButton size="icon" aria-label="Like"><Heart size={18} /></GlassButton>
// size: default | xs | sm | lg | icon | icon-xs | icon-sm | icon-lg

<GlassToolbar aria-label="Playback">
  <GlassToolbarButton aria-label="Play"><Play size={18} /></GlassToolbarButton>
  <GlassToolbarSpacer sizing="fixed" /> {/* or "flexible" with a toolbar width */}
  <GlassMenu>
    <GlassToolbarButton aria-label="More" render={<GlassMenuTrigger />}><MoreHorizontal size={18} /></GlassToolbarButton>
    <GlassMenuContent>
      <GlassMenuItem onClick={save}>Save</GlassMenuItem>
      <GlassMenuSeparator />
      <GlassMenuSubmenu>
        <GlassMenuSubmenuTrigger>Share</GlassMenuSubmenuTrigger>
        <GlassMenuSubmenuContent><GlassMenuItem onClick={copyLink}>Copy link</GlassMenuItem></GlassMenuSubmenuContent>
      </GlassMenuSubmenu>
    </GlassMenuContent>
  </GlassMenu>
</GlassToolbar>
// Menus stay inside their scene. GlassMenuContent: side, align, sideOffset.

<GlassTabs aria-label="Music" defaultValue="listen" items={[{ value: 'listen', label: 'Listen' }, { value: 'library', label: 'Library' }]} />

<GlassTabsRoot defaultValue="preview">
  <GlassTabsList aria-label="View"><GlassTabsIndicator /><GlassTabsTrigger value="preview">Preview</GlassTabsTrigger></GlassTabsList>
  <GlassTabsContent value="preview">…</GlassTabsContent>
</GlassTabsRoot>

<GlassSlider aria-label="Volume" value={volume} onValueChange={(v) => setVolume(v as number)} />
<GlassSlider ticks min={0} max={100} step={25} defaultValue={50} aria-label="Intensity" /> {/* step must divide the range */}
<GlassSwitch checked={on} onCheckedChange={setOn} />

<GlassProgressiveBlur edge="bottom" size={140} blur={24} /> {/* sibling of GlassContent; no WebGPU */}

const viewport = useRef<HTMLDivElement>(null)
<GlassContent layout="scroll" ref={viewport}>{items}</GlassContent>
<GlassScrollEdges target={viewport} size={80} blur={20} />

<GlassShape radius={14} render={<img src={art} alt="" />} />
<GlassColorPicker value={color} onValueChange={setColor} /> {/* own scene; newer releases only */}
```

```ts
import { createGlassScene } from '@glass-sdk/liquid-glass/dom'
const scene = createGlassScene(root)
scene.setContent(content)
const remove = scene.addSurface(el, { material: 'regular', radius: 28 })
// remove(); scene.dispose()
```
