# Copilot Instructions for my-nx-workspace

## Workspace Architecture

This is an **Nx monorepo** with Angular applications built using **Vite** and **Vitest**, managed by **Nx 21.4.0**. The workspace uses:
- **Angular 20.1** with standalone components (no NgModules)
- **Vite** as the build tool (via `@analogjs/vite-plugin-angular`)
- **Vitest** for unit testing with jsdom environment
- **ESLint** with `@typescript-eslint` for linting
- **TypeScript 5.8** with decorator and helper imports enabled

### Project Structure
- **apps/angular-demo/** - Main Angular application bootstrapped at `src/main.ts`
  - Uses `bootstrapApplication()` with `appConfig` (see [app.config.ts](apps/angular-demo/src/app/app.config.ts))
  - Root component is `App` (see [app.ts](apps/angular-demo/src/app/app.ts))

## Developer Workflows

### Build & Serve
```bash
npx nx serve angular-demo              # Dev server (http://localhost:4200, development config)
npx nx build angular-demo              # Production build (dist/apps/angular-demo)
npx nx serve-static angular-demo       # Static file server (spa: true)
```

### Testing & Linting
```bash
npx nx test angular-demo               # Run Vitest (jsdom, watch mode disabled in config)
npx nx lint angular-demo               # ESLint validation
```

### Code Generation
```bash
npx nx g @nx/angular:component MyComponent --project=angular-demo
npx nx g @nx/angular:service MyService --project=angular-demo
npx nx list @nx/angular                # View available generators
```

### Useful Nx Commands
```bash
npx nx show project angular-demo       # View all targets for a project
npx nx graph                           # Visualize project dependencies
npx nx affected --base=main --head=HEAD # Run affected projects (CI pattern)
```

## Critical Conventions & Patterns

### Standalone Components (Preferred)
Angular components use standalone syntax with `imports`:
```typescript
@Component({
  imports: [RouterModule, CommonModule],
  selector: 'app-root',
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App { }
```
**Why:** No NgModules; cleaner dependency injection; better tree-shaking.

### App Configuration Pattern
Application providers are centralized in `app.config.ts`:
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(appRoutes),
  ],
};
```
**Why:** Single source of truth for dependency injection; easy to add features (HTTP, animations, etc.).

### Routing
Routes are defined in `app.routes.ts` and imported in `app.config.ts`:
```typescript
export const appRoutes: Route[] = [
  // Add routes here
];
```
**Why:** Lazy-loadable, feature-based routing; no forRoot() needed.

## Build & Caching Configuration

### Nx Task Caching
- **Inputs** define what triggers rebuilds (see [nx.json](nx.json))
  - `production` input excludes test files and specs
  - `^production` tracks dependencies' production inputs
- **Outputs** are cached: build results in `dist/apps/angular-demo`, test coverage in `coverage/apps/angular-demo`
- `@angular/build:application` target has `cache: true` and `dependsOn: ["^build"]`

### Vite Configuration
- **Cache dir:** `../../node_modules/.vite/apps/angular-demo`
- **Plugins:** `@analogjs/vite-plugin-angular`, `nxViteTsPaths()`, `nxCopyAssetsPlugin()`
- **Assets:** Public files from `public/` directory
- **Test setup:** `src/test-setup.ts` runs before tests

## TypeScript & Path Aliases

[tsconfig.base.json](tsconfig.base.json) is empty for `paths` (not currently used). If adding path aliases:
```json
"paths": {
  "@app/*": ["apps/angular-demo/src/app/*"],
  "@lib/*": ["libs/*/src/index.ts"]
}
```
Update all `vite.config.mts` files to use `nxViteTsPaths()` plugin to auto-resolve.

## ESLint & Prettier

- **Config:** Root `eslint.config.mjs` + `eslint.config.mjs` in each app
- **Prettier:** `.prettierrc` for formatting; integrated with ESLint
- **Target inputs:** Lint caches based on `*.json`, `eslint.config.mjs`, and source files

## Critical Integration Points

1. **Nx Cloud:** `nxCloudId` in [nx.json](nx.json) for remote caching (CI optimization)
2. **Angular Budgets:** [project.json](apps/angular-demo/project.json#L26) enforces bundle size limits (1mb initial, 8kb component styles)
3. **Zone.js:** Provided by `zone.js` dependency; Angular uses `provideZoneChangeDetection()` with event coalescing
4. **RxJS:** Available for reactive patterns; used by Angular's router and forms

## Gotchas & Best Practices

- **No lazy routes yet:** `appRoutes` is empty; add routes directly, they auto-load due to Vite
- **Test environment:** Vitest uses `jsdom`; no browser APIs by default (use `jsdom` config for DOM testing)
- **Production mode:** Build uses `outputHashing: all`; ensure CDN cache-busting strategies account for this
- **Use Nx commands, not npm scripts:** Workspace has no root npm scripts; use `npx nx` for all tasks (enables caching, parallelization)
