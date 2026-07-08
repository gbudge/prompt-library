You are a Principal Frontend Platform Architect. You will generate a complete, production-grade project — every file, fully implemented, no placeholders — that satisfies the following specification.

---

# Svelte + Vite + TypeScript + Tailwind Extension-First Canvas Framework

## Core Concept
The app is a **blank canvas** that becomes a composed application entirely through **extensions**.  
It operates in three distinct modes, each for a different user role:

- **Build mode** (Developer): Authoring and compiling extensions into pre-bundled artifacts. This mode is external to the host app — it is a separate Vite build pipeline.
- **Design mode** (App Composer): Drag, drop, configure, and arrange pre-compiled extensions on the canvas. This is the editing environment within the host app, toggled by a floating button.
- **Run mode** (End User): The composed application runs without any editing UI, respecting all configurations, visibility, and communication channels.

Everything user-facing is an extension. The core host provides only a minimal kernel for extension lifecycle, placement, isolation, and communication.

---

## Mandatory Stack
- **Svelte** (not SvelteKit)
- **Vite** (multi‑build: one config for the host, separate configs for extensions)
- **TypeScript** (strict mode, avoid `any`)
- **Tailwind CSS** (with JIT, PostCSS)
- **Browser-native APIs** where practical
- **No backend** except optional local persistence or extension registry serving (static JSON)

---

## Project Structure (all files required)

package.json
vite.config.ts               # host build config
tsconfig.json
tailwind.config.js
postcss.config.js
index.html
src/
  main.ts
  App.svelte
  app.css
  lib/
    platform/
      context.ts             # ExtensionPlatformContext implementation (direct & sandboxed proxy)
      contract.ts            # All shared types & interfaces
      validation.ts          # config schema validation (lightweight)
    extension/
      registry.ts            # runtime registry loader (fetches registry.json, validates manifests)
      loader.ts              # dynamic import of extension bundles + sandboxed iframe support
      instance.ts            # ExtensionInstance creation, lifecycle, proxy
    canvas/
      Canvas.svelte          # renders zones & extension components (direct or iframe)
      zone.ts                # zone layout system
    design-mode/
      DesignToggle.svelte    # floating button to enter/exit Design mode
      Palette.svelte         # extension palette with search/drag
      InstancePanel.svelte   # config/edit panel
      DesignMode.svelte      # wrapper for all design UI (toggle, palette, panel)
    broadcast/
      bus.ts                 # typed broadcast channel + command bus
    diagnostics/
      store.ts               # diagnostics store
      panel.svelte           # diagnostics overlay (shown in Design mode)
    persistence/
      store.ts               # local storage persistence with versioned schema
    theme/
      tokens.ts              # design token definition & context
    i18n/
      service.ts             # internationalization service
    storage/
      service.ts             # scoped extension storage (IndexedDB)
    modal/
      service.ts             # modal orchestration
    toast/
      service.ts             # toast notification service
    sandbox/
      iframe-bridge.ts       # postMessage bridge for sandboxed extensions
      sandbox-bootstrap.html # minimal HTML for sandboxed iframe
extensions/
  navigation-bar/
    extension.vite.config.ts
    manifest.ts
    Extension.svelte
    config.schema.ts
    index.ts
  login/
    extension.vite.config.ts
    manifest.ts
    Extension.svelte
    config.schema.ts
    index.ts
  notification-center/
    extension.vite.config.ts
    manifest.ts
    Extension.svelte
    config.schema.ts
    index.ts
README.md
registry.json               # example/auto-generated registry file
scripts/
  build-extensions.ts        # builds all extensions using Vite programmatic API
  generate-registry.ts       # generates registry.json from extensions folders

---

## Mode Details

### 1. Build Mode (Developer Tooling)
- A separate Vite library build for each extension under `extensions/<id>/`.
- Each extension folder contains its own `extension.vite.config.ts` that outputs a single ES module bundle (e.g., `dist/extension.js`) and an optional CSS file.
- Build mode is **external** to the host; the host never builds extensions directly.
- For this project, provide the config files for the three example extensions and a script (`scripts/build-extensions.ts`) that builds all extensions in bulk.
- The extension bundles are placed into `extensions/<id>/dist/` (or a public directory served by the host). For simplicity, the host can serve them from `/extensions/<id>/dist/`.

### 2. Design Mode (Composer UI)
- Floating button (always visible) toggles Design mode.
- When active:
  - Shows an extension palette loading all registered extensions from the runtime registry (e.g., `/registry.json`).
  - Palette supports search/filter by name/category, drag‑and‑drop onto the canvas.
  - Canvas becomes editable: instances can be moved, resized (if allowed), locked, deleted.
  - Clicking an instance opens an **InstancePanel** with:
    - Enable/disable toggle
    - Visible/hidden toggle
    - Lock/unlock
    - Configuration form (generated from the extension's config schema)
    - Remove button
  - Diagnostics panel is visible (shows manifest validation errors, load failures, broadcast violations, etc.).
- All state changes are persisted immediately (to `localStorage` via the persistence store).
- Visual distinction: Design mode applies a subtle overlay or border to editable zones.

### 3. Run Mode (End User)
- Default mode. No design UI visible.
- Only enabled + visible extension instances are rendered.
- Extensions communicate freely via broadcast channels and command bus.
- Errors in extensions are isolated with error boundaries; the canvas continues to function.

---

## Extension Architecture (True Runtime Plugins)

### Extension Manifest & Contracts
Every extension must export:
```ts
export interface ExtensionManifest {
  id: string;
  name: string;
  version: string;
  contractVersion: string;         // "1.0.0"
  description?: string;
  category?: string;
  icon?: string;                   // emoji or icon URL
  capabilities: ExtensionCapability[];
  placement: {
    allowedZones: CanvasZone[];
    defaultWidth?: number;
    defaultHeight?: number;
    resizable?: boolean;
    draggable?: boolean;
  };
  config: {
    schemaVersion: string;
    defaults: Record<string, unknown>;
    schema: ExtensionConfigSchema; // JSON Schema subset
  };
  runtime: {
    component: ExtensionComponentRef; // reference to the Svelte component (imported at compile time)
  };
  channels?: {
    publishes?: BroadcastChannelDefinition[];
    subscribes?: BroadcastChannelDefinition[];
  };
  commands?: {
    publishes?: CommandDefinition[];   // request/response commands
    handles?: CommandDefinition[];
  };
  permissions?: ExtensionPermission[];
  lifecycle?: {
    supportsMount?: boolean;
    supportsUnmount?: boolean;
    supportsSuspend?: boolean;
    supportsResume?: boolean;
  };
  sandbox?: {
    mode?: "direct" | "iframe";       // default "direct"
    allowSameOrigin?: boolean;        // only used in iframe mode, default false
  };
}
```

Invalid manifests are rejected gracefully and logged to diagnostics.

### Extension Discovery and Loading (Runtime)
- A runtime registry file (registry.json or a small endpoint) lists all available extensions with their metadata and paths to the compiled bundles.
- The host fetches this registry at startup (or on entering Design mode).
- Extensions are loaded dynamically:
  ```ts
  const extModule = await import(`/extensions/${id}/dist/extension.js`);
  ```
- The module must export: manifest, Component, schema.
- The host then validates the manifest and registers the extension.
- No import.meta.glob is used for production discovery. (A dev‑only convenience may use it to auto‑generate the registry, but the core architecture is runtime‑based.)

For this project:
- Provide a script (scripts/generate-registry.ts) that scans extensions/*/manifest.ts and outputs a registry.json file containing an array of extension metadata and bundle paths.
- The host's registry.ts loads this JSON at runtime.

---

## Platform Context (Exposed to Extensions)

The full context given to each extension instance (via a prop or a reactive store) includes:

```ts
export interface ExtensionPlatformContext {
  extensionId: string;
  instanceId: string;
  mode: "run" | "design";
  config: {
    get<T = unknown>(key: string): T | undefined;
    set<T = unknown>(key: string, value: T): void;
    getAll(): Record<string, unknown>;
  };
  state: {
    get<T = unknown>(key: string): T | undefined;
    set<T = unknown>(key: string, value: T): void;
    clear(key?: string): void;
  };
  broadcast: {
    publish<T = unknown>(channel: string, event: string, payload: T): void;
    subscribe<T = unknown>(channel: string, event: string, handler: BroadcastHandler<T>): SubscriptionHandle;
    unsubscribe(subscriptionId: string): void;
  };
  command: {
    request<T = unknown>(command: string, payload?: unknown): Promise<T>;
    respond<T = unknown>(command: string, handler: CommandHandler<T>): void;
  };
  lifecycle: {
    requestDisable(): void;
    requestHide(): void;
    requestRemove(): void;
  };
  diagnostics: {
    log(level: "debug" | "info" | "warn" | "error", message: string, data?: unknown): void;
  };
  theme: {
    getToken(name: string): string;
    subscribe(callback: (tokens: Record<string, string>) => void): () => void;
  };
  i18n: {
    t(key: string, params?: Record<string, unknown>): string;
    locale: string;
  };
  storage: {
    get<T = unknown>(key: string): Promise<T | undefined>;
    set<T = unknown>(key: string, value: T): Promise<void>;
    delete(key: string): Promise<void>;
  };
  modal: {
    open(options: { component: any; props?: any; onClose?: (result: any) => void }): void;
    close(result?: any): void;
  };
  toast: {
    show(message: string, type?: "info" | "success" | "warning" | "error", duration?: number): void;
  };
  capabilities: {
    has(extensionId: string): boolean;
    hasChannel(channel: string): boolean;
  };
  canvas: {
    width: Writable<number>;
    height: Writable<number>;
    on(event: "resize" | "zoom", handler: (data: any) => void): void;
  };
}
```

In sandboxed iframe mode, this entire context is serialized over postMessage with a proxy that mimics the same interface. The host provides both a direct implementation and a proxy implementation; the extension's code remains unchanged.

---

## Primitives Implementation Requirements

- **Theme Tokens:** Host defines a set of CSS custom properties. platform.theme.getToken() returns their computed value. A theme.subscribe() allows extensions to react to dark/light mode changes broadcast on a reserved channel platform.theme.
- **i18n:** Extensions can register their own message catalogs via the manifest (optional). The host uses a central i18n service; platform.i18n.t() looks up the key in the extension's own namespace or falls back to the host’s default locale.
- **Scoped Storage:** Uses IndexedDB (or a simple in‑memory store with IndexedDB backup). Each extension has its own object store keyed by extensionId.
- **Modal Orchestrator:** Manages a global modal layer. Only one modal is shown at a time; new modals are queued. The host renders a <ModalHost/> in the overlay zone.
- **Command Bus (Request/Response):** Built on top of broadcast channels. A command handler declared in the manifest can respond to a request. The platform.command.request() returns a promise that resolves with the response. Correlation IDs are handled automatically.
- **Capability Query:** capabilities.has() checks the registry; hasChannel() checks if any registered extension publishes/subscribes to that channel.
- **Toast Service:** Renders transient notifications in a fixed position; does not require the extension to be visible.
- **Canvas Events:** The host updates canvas.width and canvas.height reactively and fires events on resize/zoom.

---

## Canvas System

- The canvas is a CSS grid with named areas for the zones: root, top, bottom, left, right, center, overlay, modal, sidebar, navbar, footer.
- Extensions are placed inside these zones based on their layout.zone. The layout can be absolute‑positioned within zones to support free‑form arrangement in Design mode.
- All zone rendering is generic; no special‑casing for navigation bars or dashboards.

---

## Broadcast & Command Channels

- Extensions declare channels.publishes and channels.subscribes with version strings.
- The bus enforces these declarations: publish/subscribe without permission are blocked and logged.
- Disabled extensions must not publish or receive events.
- platform.command is an abstraction over channels: commands are published on _command.<name> channels, and the host routes responses using correlation IDs.

---

## Persistence

- src/lib/persistence/store.ts loads/saves AppCompositionState from localStorage.
- State includes: list of instances (each with id, extensionId, layout, config, enabled, visible, locked) plus optional metadata.
- On load, the host validates stored instances against the current registry; missing extensions are logged and skipped.
- State is versioned (schemaVersion) and includes a migration mechanism.

---

## Sandboxing Implementation

- For sandbox.mode: "iframe", the host creates an iframe with sandbox="allow-scripts" (no allow-same-origin by default).
- The iframe loads src/lib/sandbox/sandbox-bootstrap.html, which imports the extension’s compiled bundle and mounts the Svelte component.
- A MessageChannel bridge passes all platform context calls between host and iframe. The host creates a Proxy that matches ExtensionPlatformContext and forwards calls.
- The iframe receives a similar proxy that forwards to the host.
- All extension DOM is contained within the iframe; the iframe itself is absolutely positioned inside the appropriate canvas zone.
- Resize and drag events in Design mode must be translated into postMessage events to update layout.

### Sandbox Bridge Contract (Strict Adherence Required)
For iframe-bridge.ts, you MUST implement a correlation-based async proxy pattern.
- Host-to-Iframe calls: `{ type: 'platform-request', id: string, method: string, args: unknown[] }`
- Iframe-to-Host responses: `{ type: 'platform-response', id: string, result?: unknown, error?: string }`
- Iframe-to-Host calls (e.g., lifecycle): `{ type: 'extension-request', id: string, method: string, args: unknown[] }`
- Use MessageChannel (not just window.postMessage) for dedicated host-instance communication to avoid global namespace pollution.
- The Proxy handler MUST return Promises for all async context methods and resolve/reject based on matching correlation IDs.

---

## Config Schema & Validation

- ExtensionConfigSchema is a strict TypeScript discriminated union supporting at least: string, number, boolean, select, color.
- Implement a lightweight validateConfig(schema, data) function in src/lib/platform/validation.ts.
- The InstancePanel MUST use this validator before calling context.config.set(). Invalid states must never be persisted.
- The configuration form in Design mode is generated from the schema fields.

---

## Extension Build Script Specifics

In scripts/build-extensions.ts:
- Use `import { build } from 'vite'` (programmatic API).
- Iterate over extensions/*/extension.vite.config.ts.
- Each extension config MUST define build.lib.entry, build.lib.formats: ['es'], and build.outDir: 'dist'.
- Handle errors per-extension without failing the entire batch. Log successes/failures to console.

---

## Drag-and-Drop Payload

When dragging an extension from the Palette onto the Canvas, the data transfer MUST be a JSON string:
```ts
interface DragPayload {
  extensionId: string;
}
```
The Canvas drop handler reads this, creates a new instance, and places it at the drop coordinates.

---

## Example Extensions (Fully Functional)

1. **navigation-bar:** Category "navigation", allowed zones navbar/top. Configurable title and menu items. Subscribes to auth.session; hides authenticated items on timeout/logout. Sandbox: direct.
2. **login:** Category "auth", allowed zones center/modal/sidebar. Simulates login/logout, publishes auth.session events. Configurable timeout. Sandbox: direct.
3. **notification-center:** Category "utility", allowed zones overlay/top/right. Subscribes to auth.session; shows toasts on timeout/logout. Sandbox: direct.

All three must work end‑to‑end using broadcast channels only, with no direct imports of each other.

---

## Execution & Continuation Protocol

Due to the size of this codebase, you MUST follow these rules:

1. Generate files in this exact order:
   - src/lib/platform/contract.ts
   - src/lib/platform/validation.ts
   - src/lib/platform/context.ts
   - src/lib/extension/loader.ts
   - src/lib/extension/registry.ts
   - src/lib/extension/instance.ts
   - src/lib/broadcast/bus.ts
   - src/lib/persistence/store.ts
   - src/lib/theme/tokens.ts
   - src/lib/i18n/service.ts
   - src/lib/storage/service.ts
   - src/lib/modal/service.ts
   - src/lib/toast/service.ts
   - src/lib/diagnostics/store.ts
   - src/lib/canvas/zone.ts
   - src/lib/sandbox/sandbox-bootstrap.html
   - src/lib/sandbox/iframe-bridge.ts
   - src/lib/canvas/Canvas.svelte
   - src/lib/design-mode/DesignToggle.svelte
   - src/lib/design-mode/Palette.svelte
   - src/lib/design-mode/InstancePanel.svelte
   - src/lib/design-mode/DesignMode.svelte
   - src/lib/diagnostics/panel.svelte
   - src/App.svelte
   - src/main.ts
   - src/app.css
   - index.html
   - package.json
   - vite.config.ts
   - tsconfig.json
   - tailwind.config.js
   - postcss.config.js
   - extensions/navigation-bar/manifest.ts
   - extensions/navigation-bar/config.schema.ts
   - extensions/navigation-bar/Extension.svelte
   - extensions/navigation-bar/index.ts
   - extensions/navigation-bar/extension.vite.config.ts
   - extensions/login/manifest.ts
   - extensions/login/config.schema.ts
   - extensions/login/Extension.svelte
   - extensions/login/index.ts
   - extensions/login/extension.vite.config.ts
   - extensions/notification-center/manifest.ts
   - extensions/notification-center/config.schema.ts
   - extensions/notification-center/Extension.svelte
   - extensions/notification-center/index.ts
   - extensions/notification-center/extension.vite.config.ts
   - scripts/generate-registry.ts
   - scripts/build-extensions.ts
   - README.md
   - registry.json
2. Each file must be complete and fully functional before moving to the next.
3. If you approach your output token limit, STOP cleanly at the end of a complete file.
4. End your response with exactly: `<!-- CONTINUE: Next file is [path/to/next/file] -->`
5. Do NOT summarize, truncate, or use comments like "// ... rest of code" or "// similar to above". Every line must be production-ready.
6. When I reply with "continue", resume exactly from the announced file with no preamble.

---

## Acceptance Criteria

1. App starts in Run mode with a blank canvas.
2. A floating Design button toggles Design mode.
3. Design mode displays discovered extensions from the runtime registry.
4. User can drag an extension from the palette onto the canvas.
5. User can configure an extension instance (form from schema).
6. User can enable/disable, show/hide, lock/unlock, and remove an instance.
7. Layout and config persist after reload.
8. Run mode hides all design UI.
9. Extensions never directly import each other.
10. Extensions communicate through broadcast channels (and command bus if applicable).
11. Login extension broadcasts auth.session timeout; navigation and notification react accordingly.
12. Invalid manifests are rejected gracefully and logged to diagnostics.
13. Extension render errors are isolated with error boundaries.
14. All new primitives (theme, i18n, storage, modal, toast, command, canvas, capabilities) are implemented and functional.
15. Sandboxed iframe mode works for at least one extension (you can test by converting one of the examples).
16. Project builds and runs with npm run dev and npm run build (including extension builds via the provided script).
17. README explains the three modes, how to add a new extension, build it, and load it into the host.

---

## Output Format

Return every file with its exact path using the format:

```
// path: relative/path/to/file
<file contents>
```

Do not omit any file. Do not use placeholders. Produce a complete, working codebase that satisfies all requirements.
