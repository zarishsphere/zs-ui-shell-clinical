# TECH-DESIGN-MVP — `zs-ui-shell-clinical`

> **Document:** Technical Design (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-ui-shell-clinical](https://github.com/zarishsphere/zs-ui-shell-clinical)
> **Layer:** Layer 3 — Frontend MFEs | **Catalog #:** 126
> **Language:** TypeScript 5.8 / React 19 | **License:** Apache 2.0

---

## Technical Summary

**Clinical app shell — composes all clinical MFEs.**

This document defines the **technical architecture, implementation design, complete repository tree, and acceptance criteria** for the MVP of `zs-ui-shell-clinical`.

---

## Module Federation Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'zs_ui_shell_clinical',
      filename: 'remoteEntry.js',
      exposes: {
        './App': './src/App',
      },
      shared: ['react', 'react-dom', 'react-router-dom'],
    })
  ],
  build: { target: 'esnext' }
})
```

## FHIR Data Integration Pattern

```typescript
// hooks/useShell_clinicalData.ts
import { useFHIRQuery } from '@zarishsphere/zs-pkg-ui-fhir-hooks'

export function useShell_clinicalData(tenantId: string) {
  return useFHIRQuery({
    resourceType: 'Patient', // adapt per service
    searchParams: { _tenant: tenantId },
    onError: (err) => console.error('FHIR fetch failed:', err),
  })
}
```

## Offline Strategy (Dexie.js)

```typescript
// Offline-first: read from IndexedDB first, sync to server in background
import { db } from '@zarishsphere/zs-pkg-ui-offline-store'

// On load: hydrate from local DB
const localRecords = await db.table('shell_clinical').toArray()

// Background sync when online
window.addEventListener('online', () => syncToServer(localRecords))
```

## i18n Setup

```typescript
// i18n/en.json
{
  "shell_clinical": {
    "title": "Shell Clinical",
    "loading": "Loading...",
    "error": "Something went wrong",
    "save": "Save",
    "cancel": "Cancel"
  }
}
```

## CI Pipeline

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with: {node-version: '22', cache: 'pnpm'}
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck && pnpm lint && pnpm test && pnpm build
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps chromium
      - run: pnpm exec playwright test
```

## MVP Component Tree

```
zs-ui-shell-clinical/src/components/
│   │   ├── ShellclinicalMain/
│   │   ├── │   │   │   └── ShellclinicalMain.tsx
│   │   ├── │   │   ├── (feature sub-components)
```

---


## Owners & Governance

| Role | GitHub Handle | Responsibility |
|------|--------------|----------------|
| Platform Lead | `@arwa-zarish` | Final approval, RFC votes |
| Technical Lead | `@code-and-brain` | Architecture, Go/TS review |
| DevOps Lead | `@DevOps-Ariful-Islam` | CI/CD, infra, deployment |
| Health Programs | `@BGD-Health-Program` | Clinical content, country programs |

**PR Policy:** All changes via Pull Request. Minimum 1 owner review. CI must pass. No direct commits to `main`.


---

## MVP Acceptance Checklist

- [ ] All MVP files exist in repository with real content (not placeholders)
- [ ] CI pipeline passes on `main` branch
- [ ] No secrets, credentials, or PHI committed
- [ ] README.md reflects current state with setup instructions
- [ ] CODEOWNERS file present
- [ ] All MVP functional requirements verified manually or via automated tests
- [ ] Linked to `CATALOGS.md` and `TODO.md` in `zs-docs-platform`

---

*This document is the authoritative MVP specification for `zs-ui-shell-clinical`.*
*Changes require a Pull Request with at least 1 owner approval.*
