# Minutas — MyRacing

Registro de reuniones y avances del equipo para la gestión del proyecto
(requisito `Enunciado.md` §4 — metodologías ágiles y evidencia de seguimiento).

## Metodología

**Scrum ligero**: sprint semanal, minuta por reunión, trackeo de tareas en los
Projects de GitHub por sub-repo, feature branches por tarea.

---

## 2026-09-08 — Reordenamiento de repos + pipelines CI/CD

**Asistentes:** santi (sesión de puesta al día del proyecto)

**Objetivo:** acomodar la arquitectura de repositorios antes de tocar código/tests.

### Decisiones
1. **Topología:** se mantienen 3 repos separados (Backend, Frontend, DevOps)
   con Gitflow simplificado (`main`, `develop`, `feature/*`).
2. **Docker:** pasa a ser **opcional** y externo al flujo principal. Se eliminan
   los Dockerfiles de app y los compose de producción. Deploy por **buildpack**
   (Railway/Render). Solo queda `compose.dev.yaml` (MySQL opcional para dev).
3. **CI/CD:** se crea el esqueleto de pipelines en Backend y Frontend. El deploy
   está **bloqueado si fallan los tests** (`workflow_run` sobre CI exitoso en `main`).
   El E2E full-stack corre en el pipeline del **frontend**.
4. **Ramas feature obsoletas:** eliminadas de Backend. Antes de borrar se crearon
   tags de backup por si hay trabajo no mergeado:
   - `backup/MRB-validation-deadline` (validación de deadline — NO está en develop)
   - `backup/PR2-logica-revocacion` (revocación de tokens — NO está en develop)
   - `backup/improvements` (JWT secrets sin fallback en prod — parcial)
5. **Env expuesto:** el `.env` del frontend (con `VITE_MP_PUBLIC_KEY`) estaba
   commiteado; se sacó del tracking y se refinó `.gitignore`.

### Hallazgos de estado (para la fase de tareas)
- Build backend: **OK** (`tsc --noEmit` exit 0). El `dist/` local quedó con
  dueño root (artefacto local, no afecta CI).
- Build frontend: **ROTO** — 6 errores TS: `useToast.ts` (módulo `./Toast`,
  tipo `open`), `ResetPasswordPage.tsx` (import sin usar), `usePaymentStatus.ts`
  (tipo `{}` no asignable a `User`), `apiClient.ts` (`string | null`), `raceUserService.ts`
  (param sin usar). **Primera tarea de la fase de features.**

### Pendientes
- Portar a `develop` el contenido de los tags `backup/*` que no está vigente
  (deadline de inscripción, revocación de tokens, secrets sin fallback).
- Configurar deploy manual inicial en la plataforma elegida (Railway/Render).
- Definir secrets del repo (GitHub Actions): `DB_*`, `JWT_*`, `MERCADOPAGO_*`, `URL_*`, `VITE_*`.