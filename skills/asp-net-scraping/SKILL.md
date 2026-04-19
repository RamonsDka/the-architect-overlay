---
name: asp-net-scraping
description: >
  Técnicas y mejores prácticas para hacer scraping en portales ASP.NET WebForms.
  Trigger: Cuando se trabaja con scraping en portales gubernamentales o empresariales ASP.NET.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Apply

- Scraping de portales ASP.NET WebForms (como portales gubernamentales)
- Portales que usan ViewState, EventValidation, postbacks
- Sitios con anti-bot básico a medio
- Páginas con paginación ASP.NET

## Componentes Críticos de ASP.NET

| Componente | Qué es | Por qué rompe el scraping |
|------------|--------|---------------------------|
| `__VIEWSTATE` | Estado serializado de la página | Cambia en CADA postback |
| `__EVENTVALIDATION` | Token anti-forgery | Valida que los valores POST fueron generados por el servidor |
| `__EVENTTARGET` | Control que disparó el evento | Indica qué dropdown/botón provocó el postback |
| `__EVENTARGUMENT` | Argumentos del evento | Paginación: `Page$2`, `Page$3`, etc. |

## Herramientas Recomendadas

### Playwright + playwright-stealth (RECOMENDADO)

```python
from playwright.sync_api import sync_playwright
from playwright_stealth import stealth_sync

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(
        user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
        viewport={"width": 1920, "height": 1080}
    )
    page = context.new_page()
    stealth_sync(page)
    
    # Bloquear recursos innecesarios para velocidad
    page.route("**/*.{png,jpg,jpeg,gif,css,woff,woff2}", lambda route: route.abort())
    
    page.goto("https://portal.gov.co/...")
```

### Configuración de Headers Efectiva

```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/132.0.0.0 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
    'Accept-Language': 'es-CO,es;q=0.9,en;q=0.8',
    'Referer': url,
    'Origin': 'https://portal.gov.co',
}
```

## Técnicas de Stealth

| Técnica | Efectividad |
|---------|-------------|
| playwright-stealth | Media |
| Rotación de User-Agent | Alta |
| Simulación de comportamiento humano | Alta |
| Headed mode (headless=False) | Media |
| Proxies residenciales | Muy Alta |

## Manejo de ViewState - Estrategia

```
FLUJO CORRECTO para paginación ASP.NET:

1. GET → Extraer __VIEWSTATE, __EVENTVALIDATION
2. POST con __EVENTTARGET="ctl00$Content$Grid", __EVENTARGUMENT="Page$2"
3. Del response → Extraer NUEVOS __VIEWSTATE, __EVENTVALIDATION
4. POST con los nuevos valores y __EVENTARGUMENT="Page$3"
5. Repetir...
```

```python
def get_hidden_fields(soup):
    """Extrae TODOS los hidden inputs de ASP.NET"""
    fields = {}
    for tag in soup.find_all('input', type='hidden'):
        if tag.get('name') and tag.get('value'):
            fields[tag['name']] = tag.get('value', '')
    return fields
```

## Retry y Timeouts

```python
import tenacity
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=2, min=4, max=60),
    retry=retry_if_exception_type((TimeoutError, ConnectionError))
)
def scrape_with_retry(page, url):
    page.goto(url, timeout=60000, wait_until="domcontentloaded")
    return page.content()
```

**Timeouts recomendados para ASP.NET**:
- 60s para páginas pesadas (default 30s no alcanza)
- Usar `domcontentloaded` en vez de `networkidle` (ASP.NET tiene polling activo)

## Problemas Comunes y Soluciones

| Problema | Solución |
|----------|----------|
| "Invalid ViewState" | Extraer __VIEWSTATE del response ANTES del siguiente POST |
| Timeout en paginación | Usar `domcontentloaded` en vez de `networkidle` |
| 403 Forbidden | Agregar stealth, cambiar user-agent, revisar headers |
| Portal siempre offline | Verificar cwd en subprocess, variables de entorno |
| "Connection closed" | Agregar `--disable-blink-features=AutomationControlled` |

## Configuración Stealth en Playwright

```python
page.add_init_script("""
    Object.defineProperty(navigator, 'webdriver', { get: () => undefined });
    Object.defineProperty(navigator, 'plugins', { get: () => [1, 2, 3, 4, 5] });
    Object.defineProperty(navigator, 'languages', { get: () => ['es-DO', 'es', 'en-US', 'en'] });
    window.chrome = { runtime: {} };
""")

# Args adicionales
args = [
    '--disable-blink-features=AutomationControlled',
    '--disable-dev-shm-usage',
    '--no-sandbox',
]
```

## Commands Útiles

```bash
# Instalar playwright-stealth
pip install playwright-stealth

# Instalar chromium
playwright install chromium

# Verificar instalación
python -c "from playwright.sync_api import sync_playwright; print('OK')"
```
