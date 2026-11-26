# pipelines

Workflows reutilizables de PeaceApp. Cada servicio solo declara qué es; la lógica vive aquí.

## Uso

```yaml
name: CI
on:
  push:
    branches: [main, develop]
    tags: ["v*"]
  pull_request:
    branches: [main, develop]
permissions:
  contents: read
  packages: write
  id-token: write
jobs:
  pipeline:
    uses: PeaceApp-UPC-17433/pipelines/.github/workflows/service.yml@v1
    secrets: inherit
```

Servicio Vue: `with: { runtime: node }`.

## Qué hace

| Job | Bloquea |
|---|---|
| verify | pruebas, gate de JaCoCo y quality gate de SonarCloud |
| security | Gitleaks y Trivy (HIGH y CRITICAL con parche disponible) |
| image | Trivy sobre la imagen; publica en GHCR solo en push |
| deploy | solo en tag `vX.Y.Z`: `helm upgrade --atomic` con el chart de `peaceapp-infra` |

## Etiquetas de imagen

| Evento | Etiquetas |
|---|---|
| push a develop | `X.Y.Z-dev.N`, `sha-<commit>` |
| tag `vX.Y.Z` | `X.Y.Z`, `X.Y`, `latest`, `sha-<commit>` |

`X.Y.Z` base sale de `pom.xml` o `package.json`.

## Convenciones

- El nombre del servicio es el nombre del repositorio en minúsculas. Debe existir `deploy/values/<nombre>.yaml` en `peaceapp-infra`.
- Secreto de organización: `SONAR_TOKEN`. Variables de organización: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`.
- Un servicio nuevo = un repositorio con este `ci.yml` + un archivo de valores.

## Dónde ver los resultados

- SonarCloud: tablero con issues, cobertura, duplicación y quality gate por rama y por PR (`sonarcloud.io`, proyecto `<org>_<repo>`).
- Trivy: resumen en la página de la ejecución (Summary) y el reporte completo como artefacto `trivy-fs` y `trivy-image`.
- Pruebas y cobertura: artefacto `reports` (JaCoCo HTML).
