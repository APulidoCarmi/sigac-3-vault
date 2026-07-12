# Manifiesto — SP-18: Portal de Documentos del Cliente

Implementado 2026-07-12. Este es el **último sub-plan** del paraguas
[[2026-07-10-refactor-flujo-ejecutivo]] (Fase 4 cierra la lista completa de sub-planes).

## Ramas
- `carmi-odin-api-v2`: `refactor/customs-operation-sp18`, encadenada desde
  `refactor/customs-operation-sp17` (que a su vez trae el diff acumulado de SP-04/05
  y demás fases previas). Nada comiteado.
- `carmi-digital`: `refactor/customs-operation-sp18`, misma cadena.

## Decisión de diseño: alcance del token por compañía, no por Reference

El D1 del sub-plan solo decía "reusa el patrón de public-token de OSD" (que ata el
token a **un** reporte). Al verificar el código real (`osd-reports.controller.ts`) y el
modelo `Reference` (`purchaseOrder` en `clientCompanyId`), encontré que "búsqueda por
orden de compra" (M9) no encaja con un token 1:1 a una sola Reference — el cliente
necesita **un solo link de portal** y buscar, dentro de él, cuál de sus expedientes
corresponde a una OC dada. Decisión (no bloqueante, documentada aquí en vez de
preguntar porque el criterio se deriva directo de los propios requisitos M8/M9 del
sub-plan): el token público (`ReferencePublicToken`) se ata a **`clientCompanyId`**, no
a una Reference — más cercano al precedente `CompanyOnboardingToken` (también por
compañía) que al de OSD. Cada acción sobre una Reference puntual valida
`reference.clientCompanyId === token.clientCompanyId` y devuelve **404** (no 403) si no
coincide, para no confirmarle a un tenedor de token la existencia de expedientes de
otra compañía (mitigación IDOR/enumeración explícitamente pedida por el orquestador).

## Signed URLs: ~10 min para preview, no el `publicUrl` de la carga

El D1 menciona "Signed URL de GCP (preview/descarga, expiran ~10 min)". El upload en sí
se hace con `StorageService.uploadFile(..., isPublic: false)` (no genera un signed URL
de larga vida en ese momento). El preview/descarga individual de cada documento en
`getExpediente()` genera un signed URL fresco de 600s vía
`StorageService.generateSignedUrl()` en cada request — no se persiste, no vive más que
la respuesta HTTP. La descarga del ZIP completo no usa signed URLs: el backend
descarga los buffers vía `StorageService.downloadFile(fileId)` y arma el ZIP en
streaming con `archiver`.

## Dependencia nueva: `archiver`

No existía precedente de generación de ZIP en el repo. Se agregó `archiver@7.0.1` +
`@types/archiver@6.0.4` (no la v8 más reciente: `archiver@8`/`@types/archiver@8` son
**ESM-only** — `"type": "module"` sin `main` CJS — incompatible con este proyecto
Nest/CommonJS; rompía tanto en Jest como en el build. Se fijó a la última mayor con
soporte CJS y con la API de función-factory (`archiver('zip', opts)`) que documentan
los tipos v6/v7, en vez de la API de clases (`new ZipArchive()`) que exige v8.

## Migración Prisma (CLI, según regla dura)

`npx prisma migrate dev --name add-reference-public-token-and-client-upload-flag`
(verifiqué `migrate status` antes — DB en sync). Migración generada:
`20260712072257_add_reference_public_token_and_client_upload_flag`:
- `reference_documents.uploadedByClient BOOLEAN NOT NULL DEFAULT false`.
- Tabla nueva `reference_public_tokens` (`id`, `token` único, `clientCompanyId` FK a
  `companies`, `createdAt`, `expiresAt`, `revokedAt`, `createdBy`), índices en `token` y
  `clientCompanyId`.

No se tocó SQL a mano; el archivo de migración es 100% generado por la CLI.

## Archivos tocados/creados (`carmi-odin-api-v2`)

Nuevos:
- `prisma/migrations/20260712072257_add_reference_public_token_and_client_upload_flag/migration.sql`
- `src/reference-portal/reference-portal.module.ts`
- `src/reference-portal/services/reference-portal-token.service.ts` (+ `.spec.ts`)
- `src/reference-portal/services/reference-portal.service.ts` (+ `.spec.ts`)
- `src/reference-portal/controllers/reference-portal.controller.ts` (admin, JWT) (+ `.spec.ts`)
- `src/reference-portal/controllers/reference-portal-public.controller.ts` (público, sin guard) (+ `.spec.ts`)
- `src/reference-portal/dtos/upload-client-document.dto.ts`

Modificados:
- `prisma/schema.prisma`: modelo `ReferencePublicToken`; `ReferenceDocument.uploadedByClient`;
  relación inversa en `Company`.
- `src/app.module.ts`: registra `ReferencePortalModule`.
- `src/reference-documents/dtos/create-reference-document.dto.ts`: campo opcional
  `uploadedByClient`.
- `src/reference-documents/services/reference-documents.service.ts`: persiste
  `uploadedByClient` y anota `source: 'CLIENT_PORTAL'` en el historial cuando aplica.
- `package.json`/`pnpm-lock.yaml`: + `archiver@7.0.1`, `@types/archiver@6.0.4` (dev).

## Archivos tocados/creados (`carmi-digital`)

Nuevos:
- `app/(public)/document-portal/[token]/page.tsx` — pantalla del portal: búsqueda por
  OC, vista de expediente (documentos + badges de glosado + totales), subida
  (`FileDropzone`), descarga de ZIP.
- `app/api/public/document-portal/[token]/search/route.ts`
- `app/api/public/document-portal/[token]/references/[referenceId]/route.ts`
- `app/api/public/document-portal/[token]/references/[referenceId]/documents/route.ts`
- `app/api/public/document-portal/[token]/references/[referenceId]/expediente-zip/route.ts`
- `features/document-portal/api/documentPortalService.ts`

Todos siguen el patrón ya establecido por OSD (`app/api/public/osd/[token]/*`): proxy
Next.js API route → backend, sin pasar por `apiOdinClient` (que asume sesión
autenticada), reshape de `{success, data}` a la forma que consume la página. No se tocó
`components/operations`, `customerPortal/operations` ni `actions/operations` (fuera de
alcance) — el portal es una superficie nueva e independiente.

## Gates

- Backend: `npx tsc --noEmit` limpio en archivos nuevos/tocados; `npx eslint
  src/reference-portal` limpio; `npx nest build` sin errores; `npx jest` completo:
  **527 suites / 3472 tests, todos verdes** (sin regresiones).
- Frontend: `npx tsc --noEmit` limpio (0 errores en todo el repo); `npx eslint` en los
  archivos nuevos: 0 errores (3 warnings `no-explicit-any` en bloques `catch`,
  consistente con el patrón ya usado en `patent-onboarding/[token]/page.tsx`); `npx next
  build` exitoso (la ruta `/document-portal/[token]` compila); `npx jest`: mismos 2
  fallos preexistentes por dependencia faltante `react-i18next` en
  `documentsDashboard` (no relacionados, no tocados por mí).

## Pendiente / no ejecutado

- **Playwright E2E**: no se ejecutó el flujo real (subir documento → verlo en
  Expediente → descargar ZIP) porque requiere datos sembrados (compañía cliente +
  Reference con OC de prueba + token público activo) que no existían en este entorno.
  Validado por: build, typecheck, lint, tests unitarios de los guards de seguridad
  (IDOR, expiración/revocación de token) y del flujo de negocio (upload marca
  `uploadedByClient`, dispara reglosado; búsqueda por OC scoped a la compañía). Queda
  como pendiente explícito para una sesión de verificación manual/QA con datos reales.
- No se generó un endpoint interno en el front para que el ejecutivo vea/copie el link
  del portal (el sub-plan no lo pide explícitamente y el front de `customerPortal` está
  fuera de alcance); el backend ya expone `POST/GET/DELETE
  reference-portal/companies/:companyId/token` para que cualquier UI futura lo consuma.

## Estado del paraguas

Con SP-18 completo, **todos los sub-planes de la Fase 4 (y del paraguas completo)
están cerrados**, salvo:
- **SP-16 y SP-17**: siguen **🚧 Bloqueados** en fase de diseño (D1 con brechas de
  dominio no resueltas) — pendientes de decisión humana. No se tocaron en esta sesión.
- El resto de fases (1–3) y sub-planes previamente completados (SP-01 a SP-15) se dan
  por buenos según el estado ya registrado en el índice del paraguas.

Ver actualización del índice en [[2026-07-10-refactor-flujo-ejecutivo]] (sección
"Estado del paraguas").
