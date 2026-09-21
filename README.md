# Transformit/.github

Archivos por defecto y workflows reutilizables de la organización (ADR de empresa 2026-09-06
"estándares compartidos por plugin y workflows", en `Transformit/hq`).

- `.github/PULL_REQUEST_TEMPLATE.md`: plantilla de PR por defecto para todos los repos.
- `.github/workflows/ci-prototipo.yml`: CI mínimo para repos en fase de prototipo (`workflow_call`;
  ADR de empresa 2026-09-19 "CI mínimo en prototipos"). Siempre título del PR + gitleaks; lint,
  typecheck, pruebas, squawk y migraciones inmutables activos por defecto; Postgres limpio, pgTAP,
  build y E2E se activan por input. Se llama solo en `pull_request`.
- `.github/workflows/ci-next-supabase.yml`: CI completo para productos Next.js + Supabase que ya
  salieron de prototipo (`workflow_call`): knip, depcruise, audit, cobertura, Postgres limpio,
  pgTAP, drift de tipos, build y E2E en cada PR y en cada push a main.
- `.github/workflows/deploy-ssh.yml`: build de imágenes a GHCR, `supabase db push` y despliegue
  por SSH a un VPS (`workflow_call`).
- `.github/workflows/release-please.yml`: changelog y tags semánticos (`workflow_call`).

Uso desde un repo en prototipo (plantilla completa en `Transformit/hq/plantillas/repo/.github/workflows/ci.yml`):

```yaml
# .github/workflows/ci.yml
name: ci
on: { pull_request: {}, workflow_dispatch: {} }
permissions: { contents: read, pull-requests: read }
concurrency: { group: "ci-${{ github.event.pull_request.number || github.ref }}", cancel-in-progress: true }
jobs:
  ci:
    uses: Transformit/.github/.github/workflows/ci-prototipo.yml@main
    with: { gestor: pnpm, postgres-limpio: true }
```

Inputs de `ci-prototipo`: `gestor` (pnpm | yarn | npm | uv | ninguno), `node-version`, `paquetes-apt`,
`lint`/`typecheck`/`pruebas` (activos), `formato`/`build` (apagados), `comandos` (bash del repo, uno
por línea; el camino para Python y monorepos), `ruta-supabase`, `squawk` y `migraciones-inmutables`
(activos, no hacen nada sin `supabase/migrations`), `postgres-limpio` (solo si el PR toca
`supabase/`), `pruebas-db`, `comandos-db`, `e2e`, `titulo-max`, `minutos-calidad` (tope del job
`calidad`, default 10: un proceso colgado —una cola esperando a Redis, un servidor que no cierra—
gasta minutos de Actions hasta ese tope; ponerlo cerca del tiempo normal del repo).

Alertas: cuando `ci` queda en rojo por tope de tiempo, el job final lo anota como error del run
("probablemente superó el tope…"). Para recibirlo por correo o push: GitHub → Settings →
Notifications → Actions → "Send notifications for failed workflows only" (y la app de GitHub
Mobile para push); llega al autor del PR y a quien lanzó el run.

Producto fuera de prototipo:

```yaml
name: ci
on: { pull_request: {}, push: { branches: [main] } }
jobs:
  ci:
    uses: Transformit/.github/.github/workflows/ci-next-supabase.yml@main
    with: { run-e2e: true }
```
