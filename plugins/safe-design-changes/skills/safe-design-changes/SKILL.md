---
name: safe-design-changes
description: Use when a product designer or non-engineer asks an AI agent to make a front-end change — copy/text, visual styles, existing components, or new UI — in any project they do not fully understand. Use to change code safely without breaking production: discover and follow the project's own stack, component library, and conventions; never touch config, env vars, secrets, data, API, state, auth, or money/business logic without an engineer. Triggers — "cambiar el texto", "mover/ocultar el botón", "cambiar color/espaciado", "agregar un componente", a designer pairing with an agent on an unfamiliar codebase.
---

# Cambios de diseño seguros

## Overview

Para **product designers (u otra persona no-ingeniera)** que hacen cambios de front-end con un agente de IA en un proyecto que **no conocen a fondo** y que puede estar en **producción** (usuarios reales).

**Principio central:** pensar antes de actuar → descubrir cómo trabaja ESTE proyecto → entender antes de cambiar → tocar solo la capa de UI → imitar el código que ya existe → hacer el cambio más simple posible → nunca tocar zonas peligrosas sin un ingeniero.

Esta skill es **portable**: sirve para cualquier proyecto. No asume librerías ni framework. El agente los descubre cada vez.

## Regla de oro

**Si el cambio no es claramente visual o de texto, PARA y escala a un ingeniero** (ver "Cómo escalar"). No adivines. La duda = parar.

Y: **el código que escribas debe verse como si lo hubiera escrito el equipo del proyecto.** Imita a los archivos vecinos.

## Workflow obligatorio

Para CADA cambio, el agente sigue estos pasos. Sin saltarse ninguno.

### 0. PENSAR (antes de tocar nada)
**No actúes en automático.** Antes de cualquier herramienta o edit, detente y piensa:
- ¿Qué pide REALMENTE la persona? ¿El cambio es visual/texto (zona segura) o toca lógica/datos (zona STOP)?
- ¿Cuál es el camino MÁS simple? ¿Hay una opción más sencilla que la primera que se me ocurrió?
- ¿Qué podría romperse? ¿Qué no sé todavía y debo averiguar antes?

Si no tienes un plan claro de 1-2 frases, **no empieces.** Primero piensa, luego descubre, luego actúa.

### 1. DESCUBRIR el proyecto (antes de todo)
Nunca asumas el stack. Averígualo leyendo el proyecto:
- `package.json` → framework, **librería de UI / design system**, formateadores, scripts de test/lint.
- `CLAUDE.md`, `AGENTS.md`, `README.md`, `.cursor/`, `docs/` → convenciones que el equipo ya escribió. **Síguelas.**
- ¿Cómo se maneja el **texto**? (i18n / diccionarios vs. texto directo) → busca cómo lo hace un componente parecido.
- ¿Cómo se hacen los **estilos**? (tokens del design system, Tailwind, CSS modules, styled…).
- ¿Convenciones de **nombres** y **formato**? (mira 2-3 archivos similares).

Antes de editar, di en una frase: *"Este proyecto usa X (framework), Y (librería de UI), el texto se maneja con Z. Voy a seguir eso."*

### 2. ENTENDER
- Encuentra el archivo exacto y léelo (y 1-2 archivos vecinos similares como referencia).
- Explica a la persona en **lenguaje que un Product Designer entienda sin esfuerzo**: cero jerga técnica; si un término técnico es inevitable, tradúcelo con una analogía o ejemplo. Di **qué hace este archivo** y **qué exactamente va a cambiar**, como si se lo contaras a alguien que no programa.
- Regla: si tu explicación necesita que la persona ya sepa de código para entenderla, **reescríbela más simple.**
- Muestra un **antes / después** claro del fragmento.
- Confirma que el cambio cae en una **Zona segura**. Si toca una **Zona STOP** → para y escala (ver "Cómo escalar").

### 3. CONFIRMAR
- Pide **OK explícito** antes de editar.
- Si la persona no entiende qué hace el código, NO continúes: explícalo hasta que entienda o escala.

### 4. CAMBIAR
- Cambio **mínimo y lo más simple posible**. La solución sencilla casi siempre es la correcta.
- **No sobre-ingenieres (no overengineer):** nada de abstracciones, capas, helpers genéricos, configs ni "preparar para el futuro" que el cambio no necesita HOY. Si dudas entre dos formas, elige la más corta y obvia.
- **Imita el código vecino:** mismos componentes, mismo patrón de texto, mismos tokens/clases, mismo estilo de nombres. Reusa lo que ya existe antes de crear algo nuevo.
- Un cambio a la vez. Sin refactors "de paso".

### 5. VERIFICAR (antes de decir "listo")
Usa los comandos que **ESTE proyecto** define (míralos en `package.json` → `scripts`). Típicamente:
- Lint / formato (ej. `npm run lint`)
- Tipos, si aplica (ej. `tsc --noEmit`)
- **Compilar el proyecto (OBLIGATORIO en cualquier cambio de estilo/CSS):** `npm run build` o `npm run dev`.
- Tests (ej. `npm run test` / unit / e2e)
- Verlo corriendo en el navegador (ej. `npm run dev`)

⚠️ **Lint y tipos NO atrapan errores de CSS/Tailwind/`@apply`.** Esos solo salen al **compilar** (el bundler: Vite/Rsbuild/webpack…). Una clase inexistente (ej. un step de opacidad o un token que no existe) pasa lint y `tsc` sin chistar y revienta en el build o en la pantalla del usuario. **Si tocaste estilos, no digas "listo" hasta que el build compile sin errores.**

**Nunca digas "listo" sin verificar.** Si algo falla, dilo con el error exacto. No silencies errores.

## Zonas SEGURAS (un diseñador puede cambiar)

- **Texto / wording / traducción** — usando el sistema de texto que ya use el proyecto (i18n o el patrón que veas). No hardcodees si el proyecto usa diccionarios.
- **Estilos visuales** — color, espaciado, tamaño, usando los **tokens/clases del design system** existente, no valores sueltos.
- **Componentes existentes** — mostrar/ocultar, reordenar, cambiar props ya soportadas.
- **UI nueva** — usando los **componentes de la librería del proyecto** primero, siguiendo sus convenciones.

## Zonas STOP (requieren ingeniero — NUNCA sin escalar)

PARA si el cambio toca cualquiera de esto (cómo detectarlo en cualquier repo):

- **Configuración y entorno** — `.env`, variables de entorno, archivos de config, build. Cambian a qué servidor/datos apunta la app.
- **Feature flags / toggles** — encienden funcionalidades en producción.
- **Llamadas al backend** — clientes HTTP (axios/fetch), carpetas tipo `api/`, `services/`, endpoints, headers, tokens.
- **Estado y datos** — stores (Pinia/Redux/Vuex/Zustand…), lógica de datos.
- **Lógica de negocio / dinero** — montos, redondeo, precisión decimal, cálculos, precios, impuestos. Un decimal mal = plata mal.
- **Autenticación, seguridad, secretos, routing/permisos.**

Regla simple: **markup y texto = OK. Lógica, datos, config, dinero, conexión, auth = STOP.**

## Cómo escalar (cuando toca una zona STOP)

El agente **no puede contactar a un ingeniero por sí mismo**. Escalar significa: **PARAR y entregarle a la persona un paquete de traspaso listo para reenviar.**

Al parar, el agente:
1. **No aplica el cambio.** No edita el archivo riesgoso.
2. **No investiga ni analiza el código riesgoso** ni propone cómo arreglarlo. Solo identifica que es zona STOP y se detiene.
3. Entrega este traspaso (en este formato):

```
🛑 No puedo hacer este cambio solo — toca una zona que requiere un ingeniero.

Pedido: "<lo que pidió la persona, en una frase>"
Archivo: <ruta:línea>
Zona STOP: <cuál — ej. lógica de dinero / config / backend / estado>
Por qué: <por qué es riesgoso para un no-ingeniero, en una frase>

→ Mensaje listo para reenviar a un ingeniero:
"Hola, necesito ayuda con un cambio en <archivo>. Contexto: <qué quiero lograr>.
Toca <zona STOP>, por eso no lo hago solo. ¿Puedes revisarlo o hacerlo conmigo?"
```

Después de entregar el traspaso, el agente **se detiene** y espera. No insiste, no busca un atajo, no hace "una parte segura" del cambio riesgoso.

## Red flags — PARA

- "Solo cambio esta variable rápido" → ¿es env/config/toggle? STOP.
- "Edito el service para que traiga otro dato" → STOP, es backend.
- "Ajusto este cálculo de monto/redondeo" → STOP, es dinero/lógica.
- "No entiendo bien qué hace pero se ve fácil" → STOP, entiende primero.
- "Invento un componente nuevo" → ¿la librería del proyecto ya tiene uno? Úsalo.
- "Escribo el texto directo aquí" → ¿el proyecto usa i18n/diccionarios? Síguelo.
- "Salto el lint/test, seguro está bien" → No. Verifica siempre.
- "Lint y tipos pasaron, listo" → ¿tocaste estilos? Lint/tipos NO compilan CSS. Corre el build primero.
- "Aprovecho y refactorizo esto otro" → No. Cambio mínimo.

## Errores comunes

| Error | En su lugar |
|---|---|
| Asumir el stack sin mirar | Lee `package.json` + docs del proyecto primero (paso 0). |
| Texto hardcodeado donde el proyecto usa i18n | Usa el sistema de texto del proyecto. |
| Color/tamaño con valor suelto (`#3b82f6`, `13px`) | Usa tokens/clases del design system existente. |
| Crear componente desde cero | Reusa el de la librería del proyecto. |
| Sobre-ingeniería: abstracciones/capas/configs que no se necesitan hoy | Haz lo más simple que resuelva el pedido. |
| Empezar a editar sin un plan claro | Piensa 1-2 frases primero (paso 0). |
| Explicar con jerga que el diseñador no entiende | Reescribe más simple, con analogías. |
| Código que no se parece al del repo | Imita los archivos vecinos. |
| Decir "listo" sin correr lint/tests | Verifica y reporta el resultado real. |
| Confiar solo en lint+tipos tras tocar estilos | Corre el **build** — lint/tipos no compilan CSS/Tailwind. |
| Intentar "una parte" de un cambio en zona STOP | Para completo y entrega el traspaso. |
