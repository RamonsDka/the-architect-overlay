# Awesome Fullstack LLM Architect

**Description**: Mejores prácticas para el desarrollo full-stack de aplicaciones integradas con Large Language Models (LLMs), RAG (Retrieval-Augmented Generation) y AI Agents.
**Trigger**: Cuando el usuario solicite crear una app de IA, integrar un modelo LLM, implementar RAG, crear agentes autónomos o trabajar con Vercel AI SDK/LangChain.

## 1. Reglas de Oro
- **Seguridad primero**: JAMÁS pongas una API key de un LLM (OpenAI, Anthropic, Gemini, etc.) en el frontend. Toda la comunicación con el modelo debe pasar por un backend seguro.
- **Streaming es innegociable**: Las respuestas de LLM toman tiempo. Implementar respuestas en streaming (Vercel AI SDK, Server-Sent Events, WebSockets) para mejor UX, en lugar de esperar 30 segundos una respuesta en bloque.
- **Control de Contexto (Context Window)**: Limitar y resumir activamente la cantidad de texto que el usuario o el sistema envía. Un contexto limpio equivale a menos "alucinaciones" (hallucinations) y menor costo.
- **Métricas y Tracing**: La IA es probabilística. Exige implementar logs para los inputs de usuario y los outputs del LLM (ej: Langfuse, Helicone) para monitorear calidad y costos.

## 2. Decision Tree de Arquitectura AI
- **¿Es una simple llamada API?** -> Next.js Route Handlers / Express.
- **¿Se requiere contexto de documentos privados?** -> RAG (Embeddings + Vector DB como Pinecone, PgVector, Weaviate).
- **¿El modelo necesita usar herramientas externas (ej: consultar una base de datos o buscar en la web)?** -> Agents / Function Calling.

## 3. Estilo de Respuesta
- Cuestiona la necesidad de usar IA. Si el problema se puede resolver con un simple `if/else` o un filtro, no gastes tokens al pedo. No todo es un "LLM problem".
- Separa claramente el código del UI, el backend API y la lógica de prompt engineering.
- Ofrece siempre un plan de fallback (¿Qué pasa si el LLM falla o responde algo tóxico?).
