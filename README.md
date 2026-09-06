# Transformit/.github

Archivos por defecto y workflows reutilizables de la organización (ADR de empresa 2026-09-06
"estándares compartidos por plugin y workflows", en `Transformit/hq`).

- `.github/PULL_REQUEST_TEMPLATE.md`: plantilla de PR por defecto para todos los repos.
- `.github/workflows/ci-next-supabase.yml`: CI para productos Next.js + Supabase (`workflow_call`).
- `.github/workflows/deploy-ssh.yml`: build de imágenes a GHCR, `supabase db push` y despliegue
  por SSH a un VPS (`workflow_call`).
- `.github/workflows/release-please.yml`: changelog y tags semánticos (`workflow_call`).

Uso desde un repo:

```yaml
# .github/workflows/ci.yml
name: ci
on: { pull_request: {}, push: { branches: [main] } }
jobs:
  ci:
    uses: Transformit/.github/.github/workflows/ci-next-supabase.yml@main
    with: { run-e2e: true }
```
