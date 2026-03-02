name: flex-analyzer
version: 2.1.0
description: Motor de análisis de marketing y optimización de campañas impulsado por IA
author: FlickClaw Engineering
tags: [marketing, ai, automation, analytics]
dependencies:
  - python>=3.9
  - openai>=4.0
  - pandas>=2.0
  - numpy>=1.24
  - scikit-learn>=1.2
  - matplotlib>=3.7
  - seaborn>=0.12
  - plotly>=5.14
  - jupyter>=1.0
  - google-analytics-data>=0.19
  - facebook-business>=22.0
  - tweepy>=4.14
  - requests>=2.31
required_env_vars:
  - OPENAI_API_KEY
  - GA4_CREDENTIALS_PATH (optional)
  - FB_ACCESS_TOKEN (optional)
  - TWITTER_BEARER_TOKEN (optional)
system_requirements:
  memory_min_gb: 4
  disk_min_gb: 10
  network: outbound HTTPS required
platforms: [linux, macos]
---

# Flex Analyzer

Motor de análisis de marketing y optimización de campañas impulsado por IA para la toma de decisiones basada en datos.

## Propósito

Flex Analyzer proporciona inteligencia de marketing profunda analizando el rendimiento de campañas, patrones de comportamiento del cliente, efectividad del contenido y posicionamiento competitivo. Se utiliza para:
- Predecir el ROI de la campaña antes del lanzamiento
- Identificar vacíos de contenido y oportunidades
- Automatizar el análisis de pruebas A/B
- Generar informes semanales de rendimiento de marketing
- Detectar anomalías en la segmentación de audiencia
- Optimizar la asignación de presupuesto publicitario
- Analizar el sentimiento en canales sociales
- Crear briefs de contenido respaldados por datos

## Alcance

### Comandos Principales

- `flex-analyze campaign --source <csv/json/ga4/fb> --metrics <list> --period <dates>`
- `flex-analyze content --type <blog/social/email> --performance <csv> --generate-brief`
- `flex-analyze audience --segments <file> --behavior <events> --predict <metric>`
- `flex-analyze competitor --domains <list> --metrics <market-share/sentiment/backlinks>`
- `flex-analyze roi --campaigns <ids> --attribution <model> --budget <amount>`
- `flex-analyze sentiment --source <twitter/fb/comments> --query <string> --timeline <days>`
- `flex-analyze ab-test --results <csv> --confidence <0.95> --recommend`
- `flex-analyze forecast --historical <csv> --model <prophet/arima> --horizon <days>`
- `flex-analyze dashboard --export <html/pdf> --email <address> --schedule <cron>`

### Comandos de Configuración

- `flex-config init --api-key <key> --brand <name> --industry <sector>`
- `flex-config connectors --add-ga4 <credential-path> --add-fb <token> --add-twitter <token>`
- `flex-config models --select <gpt-4/claude-3/gpt-3.5-turbo> --temperature <0.0-1.0>`
- `flex-config alerts --thresholds <yaml> --notify <email/slack/webhook>`

## Proceso de Trabajo

### 1. Inicialización y Validación
```bash
flex-config init --api-key sk-... --brand "Acme Corp" --industry "ecommerce"
```
Valida claves API, prueba conexiones, crea `~/.flex-analyzer/config.yaml` con:
- Parámetros de voz de marca
- Benchmarks de industria
- Definiciones de KPI
- Políticas de retención de datos

### 2. Ingesta de Datos
- Para CSV/JSON: validar esquema, verificar nulos, normalizar fechas
- Para GA4: autenticar con cuenta de servicio, consultar con `run_report()` usando métricas definidas
- Para Facebook: usar Marketing API para obtener insights de campaña con desgloses
- Para Twitter: obtener tweets con parámetros de consulta, manejo de límites de tasa (450 req/15min)

### 3. Pipeline de Análisis con IA
1. **Preprocesamiento**: Eliminación de valores atípicos con IQR (1.5x), imputación de valores faltantes (mediana), transformación log de métricas sesgadas
2. **Ingeniería de Características**: Crear métricas derivadas (CTR, CPC, ROAS, tasa de engagement), calcular promedios móviles (7/30d)
3. **Segmentación**: Clustering K-means (método del codo para k) en vectores de comportamiento del cliente
4. **Generación de Ideas**: Ingeniería de prompts con contexto:
   ```
   System: You are a senior marketing analyst with 10 years experience.
   Context: Brand {brand} in {industry}, analyzing {period} performance.
   Data: {summary_stats, top_performers, trends}
   Task: Provide 5 actionable insights with confidence scores (0-1).
   Format: JSON with fields: insight, confidence, supporting_data, recommended_action
   ```
5. **Visualización**: Generar gráficos Plotly (HTML interactivo) o PNGs estáticos con colores de marca

### 4. Generación de Resultados
- **Informe JSON**: `analysis_<timestamp>.json` con todas las métricas, ideas, predicciones
- **Dashboard HTML**: Gráficos incrustados, resumen ejecutivo, tablas exportables
- **Brief por Email**: Formato HTML con botones CTA para acciones recomendadas
- **Mensaje Slack**: Bloques preformateados con métricas clave y enlace al informe completo

### 5. Automatización (Opcional)
- Sintaxis de cron: `0 8 * * 1 flex-analyze campaign --weekly --email team@company.com`
- Integración webhook: POST resultados a Notion/Asana/Trello
- Bot Slack: `/flex-analyze last-week` genera informe instantáneo

## Reglas de Oro

1. **Privacidad de Datos**: Nunca registrar PII (email, teléfono, dirección). Hash de IDs de usuario con SHA-256 antes del almacenamiento. Eliminar datos crudos después de 30 días (configurable).
2. **Cuotas de API**: Respetar límites de tasa—implementar retroceso exponencial (1s, 2s, 4s, 8s máx). Alertar cuando se consume 80% de la cuota.
3. **Consistencia del Modelo**: Usar misma versión de modelo para análisis comparables. Rastrear ID de modelo en metadatos del informe.
4. **Significancia Estadística**: Para pruebas A/B, requerir p-valor < 0.05 Y tamaño mínimo de muestra 1000 por variante. Marcar de otro modo.
5. **Mitigación de Sesgo**: Verificar paradoja de Simpson en informes segmentados. Incluir métricas de diversidad (distribución de género/edad de audiencia).
6. **Precisión de Atribución**: Por defecto usar atribución basada en datos (cadena de Markov). Usar último clic solo si se solicita explícitamente.
7. **Seguridad de Exportación**: Cifrar todas las exportaciones con `openssl enc -aes-256-cbc -salt -in file -out file.enc`. Almacenar claves de descifrado separadas.
8. **Auditar Rastro**: Registrar cada comando con ID de usuario, timestamp, hash de entrada y checksum de salida en `~/.flex-analyzer/audit.log`.

## Ejemplos

### Ejemplo 1: Análisis de Campaña (Últimos 30 Días)
```bash
$ flex-analyze campaign \
  --source ga4 \
  --metrics "sessions,conversions,revenue,cost" \
  --period "2024-01-01,2024-01-31" \
  --breakdown "campaign,source_medium" \
  --confidence 0.95 \
  --output json
```
Salida: `campaign_analysis_20240131_143022.json` que contiene:
```json
{
  "period": "2024-01-01 to 2024-01-31",
  "total_spend": 45678.90,
  "total_revenue": 234567.12,
  "roas": 5.14,
  "top_campaigns": [
    {"name": "Summer Sale FB", "roas": 8.42, "spend": 12345.00}
  ],
  "insights": [
    {
      "confidence": 0.94,
      "text": "Video ads CTR 47% higher than static images. Recommend shifting 30% budget to video.",
      "supporting_data": {"video_ctr": "2.3%", "image_ctr": "1.6%"}
    }
  ]
}
```

### Ejemplo 2: Generación de Brief de Contenido
```bash
$ flex-analyze content \
  --type blog \
  --performance top100_posts.csv \
  --generate-brief \
  --target-audience "small business owners" \
  --keywords ["email marketing", "automation", "ROI"]
```
Salida: `content_brief_20240131.md`:
```markdown
# Content Brief: Email Marketing for SMBs

## Top Performing Patterns (based on 100 posts)
- Avg word count: 1,247 (optimal range: 1,200-1,400)
- Header structure: 3-5 H2s, 2 internal links
-CTA placement: 2nd paragraph + conclusion

## Recommended Topic Clusters
1. "Email Automation ROI Calculator" (search volume: 2,400/mo, difficulty: 45)
2. "Small Business Email Templates" (volume: 1,800/mo, difficulty: 38)

## Title Options (CTR predicted > 3.2%)
- "Email Marketing ROI: How We Generated $45K in 30 Days"
- "7 Email Automation Workflows for Busy Entrepreneurs"

## Content Outline
1. Problem statement (pain points)
2. Framework introduction
3. Step-by-step implementation
4. Template gallery
5. Expected outcomes (with calculator)

## SEO Recommendations
- Target keyword in first 100 words
- Include 3 FAQ schema questions
- Internal link to: /tools/email-calculator
```

### Ejemplo 3: Dashboard de Análisis de Sentimiento
```bash
$ flex-analyze sentiment \
  --source twitter \
  --query "#FlexAnalyzer OR from:flex_claw" \
  --timeline 7 \
  --generate-dashboard \
  --api-key $TWITTER_BEARER
```
Crea `sentiment_dashboard_20240131.html` con:
- Gráfico de polaridad de sentimiento en tiempo real (positivo/neutral/negativo)
- Temas más mencionados (nube de palabras)
- Métricas de engagement de influencers
- Mapa de calor de horas pico de actividad
- Plantillas de respuesta sugeridas para menciones negativas

### Ejemplo 4: Recomendación de Prueba A/B
```bash
$ flex-analyze ab-test \
  --results ab_test_landing_page.csv \
  --confidence 0.99 \
  --recommend \
  --min-effect-size 0.05
```
Formato CSV:
```
variant,visitors,conversions
control,10470,843
treatment_a,10521,912
treatment_b,10512,876
```

Salida:
```
✅ Statistically significant results detected (p = 0.023)

RECOMMENDATION: Treatment A
- Conversion rate: 8.66% (vs Control: 8.05%)
- Relative improvement: +7.6%
- Confidence interval: [95% CI 1.8% to 13.2%]
- Sessions needed for 95% power: 12,341 (current: 10,521)
- Expected monthly revenue uplift: $12,450

ROLLBACK TRIGGER: If conversion rate after 7 days < 7.8%, revert to Control.
```

## Dependencias y Requisitos

**Paquetes de Python** (instalados via `pip install flex-analyzer`):
```
openai>=4.0  # GPT-4 for insights
pandas>=2.0   # Data manipulation
scikit-learn>=1.2  # Clustering, anomaly detection
plotly>=5.14  # Interactive charts
google-analytics-data>=0.19  # GA4 API
facebook-business>=22.0  # Meta Ads API
```

**APIs Externas** (requieren credenciales):
- OpenAI API key (gpt-4-turbo-preview recomendado, 4K contexto)
- Google Analytics 4 (JSON de cuenta de servicio con Analytics Readonly)
- Facebook Marketing API (token de usuario del sistema con ads_read)
- Twitter API v2 (Bearer token con tweet.read, users.read)

**Sistema**:
- Python 3.9+ con soporte venv
- 4GB RAM mínimo (8GB recomendado para >100K filas)
- 10GB disco para caché e informes
- HTTPS saliente (puerto 443) a todos los endpoints de API

**Archivos de Configuración**:
- `~/.flex-analyzer/config.yaml` (auto-generado)
- `~/.flex-analyzer/credentials/` (tokens cifrados)
- `~/.flex-analyzer/cache/` (caché de respuestas API, TTL 1h)

## Verificación

### Comprobación Post-Instalación
```bash
flex-analyzer --version
# Esperado: Flex Analyzer 2.1.0

flex-config validate --all
# Comprobaciones esperadas:
# ✓ OpenAI API key: valid
# ✓ GA4 credentials: accessible
# ✓ Facebook token: permissions OK
# ✓ Disk space: 12.4GB free (min 10GB)
# ✓ Memory: 8.2GB available (min 4GB)
```

### Conjunto de Datos de Prueba
```bash
flex-analyzer test-run --sample marketing_sample_2024.csv
# Debería producir:
# - analysis_test_<timestamp>.json
# - dashboard_test_<timestamp>.html
# - Log: Todas las ideas confianza > 0.75
# - Sin advertencias sobre datos faltantes
```

### Monitor de Salud
```bash
flex-analyzer health
# Salida:
# {
#   "status": "healthy",
#   "last_successful_run": "2024-01-31T14:30:22Z",
#   "api_quotas": {
#     "openai": "12% used",
#     "ga4": "3% used",
#     "facebook": "18% used"
#   },
#   "cache_size": "2.3GB",
#   "disk_available": "12.1GB"
# }
```

## Solución de Problemas

### Error: `OpenAI API quota exceeded`
**Síntoma**: `openai.RateLimitError: You exceeded your current quota`
**Solución**:
1. Verificar uso: https://platform.openai.com/usage
2. Reducir tamaño de lote: añadir `--batch-size 5` (por defecto 10)
3. Cambiar a modelo más barato: `flex-config models --select gpt-3.5-turbo`
4. Si es crítico, usar flag `--no-ai` para análisis solo estadístico

### Error: `GA4 property not found`
**Síntoma**: `google.api_core.exceptions.NotFound: Not found: Property`
**Solución**:
1. Verificar que cuenta de servicio tiene rol `Analytics Readonly` en GA4 Admin
2. Comprobar que `GA4_CREDENTIALS_PATH` apunta a JSON válido (no expirado)
3. Probar conexión: `flex-config connectors --test-ga4`
4. Asegurar que ID de propiedad coincide: `properties/123456789` (no solo `123456789`)

### Error: `Facebook Marketing API permission denied`
**Síntoma**: `(#200) The user hasn't authorized the application...`
**Solución**:
1. Regenerar token de usuario del sistema con permiso `ads_read`
2. Confirmar que token no ha expirado (tokens de 90 días)
3. Asignar token a cuenta publicitaria en Business Settings
4. Probar: `flex-config connectors --test-fb`

### Error: `MemoryError during large dataset processing`
**Síntoma**: Proceso terminado con código de salida 137
**Solución**:
1. Habilitar procesamiento por fragmentos: `--chunk-size 10000`
2. Aumentar swap: `sudo fallocate -l 8G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile`
3. Filtrar datos: `--filters "date >= '2024-01-01' and spend > 0"`
4. Muestrear: `--sample 0.1` (10% de los datos)

### Error: `Invalid confidence threshold`
**Síntoma**: `ValueError: Confidence must be between 0.8 and 0.999`
**Solución**: Usar rango válido: `--confidence 0.95` (por defecto) o `--confidence 0.99` para pruebas estrictas. No usar 0.99 para muestras pequeñas (<10K).

### Error: `Certificate verify failed`
**Síntoma**: `ssl.SSLCertVerificationError` durante llamadas API
**Solución**:
```bash
# Actualizar bundle de CA
sudo apt-get install ca-certificates  # Ubuntu/Debian
sudo update-ca-certificates

# O eludir (no recomendado): export SSL_CERT_FILE=/path/to/cacert.pem
```

### Advertencia: `High missing data in column X (45% null)`
**Implicación**: Los resultados del análisis pueden no ser fiables.
**Acción**:
1. Inspeccionar fuente de datos: `flex-analyze inspect --source raw_data.csv`
2. Imputar nulos: `--impute median` o `--impute forward-fill`
3. Eliminar columna: `--exclude-columns X,Y,Z`
4. Contactar proveedor de datos si es inesperado

## Comandos de Rollback

### Rollback de Análisis Reciente
```bash
# Listar últimas 10 análisis
flex-analyzer history --limit 10

# Restaurar versión anterior (por timestamp)
flex-analyzer rollback --timestamp 20240130_093022 \
  --restore-config \
  --notify-team
```
Rollback incluye:
- Archivo de configuración (`~/.flex-analyzer/config.yaml`)
- Selección de modelo
- Referencias de credenciales (no secretos)
- Umbrales de alerta

### Restaurar desde Backup
```bash
# Restauración completa del sistema (config + caché)
flex-analyzer restore --backup ~/.flex-analyzer/backups/20240128_120000.tar.gz \
  --verify-checksum \
  --dry-run  # Quitar para ejecutar
```

### Deshacer Último Cambio de Configuración
```bash
flex-config undo --steps 1  # Revertir último cambio
flex-config undo --steps 3  # Revertir últimos 3 cambios
```
Usa historial tipo git almacenado en `~/.flex-analyzer/.config_history/`.

### Parada de Emergencia (Matar Trabajos en Ejecución)
```bash
flex-analyzer stop --all --force
# Envía SIGINT, espera 5s, luego SIGKILL
# Borra archivos temporales en /tmp/flex-analyzer-*
# Libera bloqueos de API
```

### Revertir Versión de Modelo de IA
```bash
flex-config models --list
# Muestra: gpt-4-turbo (actual), gpt-3.5-turbo, claude-3-opus

flex-config models --select gpt-3.5-turbo-1106  # Degradar
# Re-ejecuta último análisis con modelo antiguo para comparación
```

### Revertir a Estilo de Idea Previo
```bash
flex-config insights --restore-defaults \
  --format markdown \
  --confidence-threshold 0.8
# Sobrescribe cualquier plantilla de prompt personalizada
```

### Limpiar Caché y Re-fetch
```bash
flex-analyzer cache --clear --older-than 7d  # Borrar +7 días
flex-analyzer cache --clear-all  # Opción nuclear
flex-analyzer refresh --force --sources ga4,facebook  # Re-descargar
```

### Rollback de Migración de Base de Datos (si se usa PostgreSQL)
```bash
# Flex Analyzer puede usar SQLite local (por defecto) o PostgreSQL
flex-db downgrade --target v1.2.0  # Última versión estable
flex-db verify --integrity  # Verificar conteos de filas y checksums
```

## Problemas Comunes y Soluciones Rápidas

| Síntoma | Causa Probable | Comando para Solucionar |
|---------|----------------|-------------------------|
| `401 Unauthorized` | Token API expirado | `flex-config connectors --refresh-all` |
| `Slow GA4 queries` | Rango de fechas demasiado grande | Añadir `--max-rows 100000` o reducir periodo |
| `No social data` | Twitter API v2 no habilitada | Habilitar en portal dev, recrear Bearer token |
| `Empty report` | Todos los datos filtrados | Quitar `--filters`, verificar formato de fecha (YYYY-MM-DD) |
| `plt.show() blocked` | Backend no interactivo | Establecer `MPLBACKEND=Agg` en env o usar `--no-display` |
| `JSON decode error` | Cuota API excedida en medio de ejecución | Reintentar con `--retry 3 --backoff exponential` |
| `File not found` | Ruta de credenciales GA4 incorrecta | `export GA4_CREDENTIALS_PATH=~/creds/ga4.json` |
| `Slack webhook failed` | Formato URL inválido | Verificar que webhook termina en `/services/...` |

## Uso Avanzado

### Modelo de Atribución Multi-fuente
```bash
flex-analyze roi \
  --campaigns "fb_summer,tw_holiday,google_pmax" \
  --attribution MarkovChain \
  --lookback-window 90d \
  --custom-weights "direct:0.4,organic:0.3,social:0.2,email:0.1"
```

### Informe Semanal Automatizado con Slack
```bash
# Configurar en crontab:
0 8 * * 1 flex-analyze campaign \
  --weekly \
  --generate-dashboard \
  --slack-webhook https://hooks.slack.com/services/... \
  --email marketing@company.com
```

### Inteligencia Competitiva (Lote)
```bash
flex-analyze competitor \
  --domains "competitor1.com,competitor2.com,competitor3.com" \
  --metrics "backlinks,domain-authority,top-keywords,ad-spend-estimate" \
  --output competitors_analysis_$(date +%Y%m).xlsx
```

### Inyección de Prompt Personalizado
```bash
flex-analyze content \
  --performance posts.csv \
  --custom-prompt "
    Analyze these posts and identify gaps in keyword coverage.
    Focus on long-tail opportunities with KD < 30.
    Return JSON: {gaps: [{keyword, volume, kd, current_rank}]}
  " \
  --output json
```
```