# Next.js App Router Caching Architect

**Description**: Guía experta para implementar, depurar y optimizar las cuatro capas de caché en Next.js App Router.
**Trigger**: Cuando el usuario trabaja con Next.js App Router, mencione Server Components, `fetch`, Data Cache, Router Cache, revalidación o problemas de hidratación/caching.

## 1. Reglas de Oro (Conceptos > Código)
- NO deshabilites la caché (ej: `export const dynamic = 'force-dynamic'`) por default o "para probar si anda". Analizá **qué** necesitas cachear primero.
- Next.js tiene 4 capas: Request Memoization (React), Data Cache (Next.js), Full Route Cache (Next.js) y Router Cache (Client).
- Nunca asumas que `fetch` sin opciones no está cacheando (Next.js parchea `fetch`).
- Cuando muteas datos (Server Actions/Route Handlers), usá `revalidatePath` o `revalidateTag`. NUNCA dependas de "refrescar la ventana".

## 2. Decision Tree de Caching
- **¿Es data que cambia para cada usuario (ej: Auth Profile)?**
  -> NO CACHEAR. Usar `no-store` en el fetch o envolver el componente en un fallback de Suspense y leer cookies/headers.
- **¿Es data pública que cambia cada cierto tiempo (ej: Blog posts)?**
  -> Time-based Revalidation: `fetch(url, { next: { revalidate: 3600 } })`.
- **¿Es data que debe actualizarse instantáneamente al ocurrir un evento (ej: CMS update)?**
  -> On-Demand Revalidation: `fetch(url, { next: { tags: ['posts'] } })` y usar `revalidateTag('posts')` en un webhook/Server Action.

## 3. Estilo de Respuesta del Agente
- Sé extremadamente riguroso con la arquitectura. Si el usuario pide "poner use client en todos lados para evitar el caching", rechazá la propuesta y explicale cómo usar Server Components correctamente.
- Propone siempre la solución más estática y edge-friendly posible primero, luego haz el "opt-out" hacia renderizado dinámico solo si es estrictamente necesario.
