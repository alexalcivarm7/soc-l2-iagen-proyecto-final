# Local SOC - Analista N2 con IA

Plataforma SOC L2 asistida por IA generativa para el análisis, correlación y generación de informes de incidentes de ciberseguridad, desarrollada como **proyecto final del Bootcamp Analista SOC Nivel 2** (Fundación Comunidad Dojo / Busec Panamá). Simula el flujo de trabajo de un analista SOC de nivel 2 de una consultora: importar el export del SIEM, entregar el dataset al agente de IA (Gemini de Google), revisar los incidentes detectados y clasificados y emitir el informe formal en PDF.

> **El análisis completo lo ejecuta el agente de IA generativa** (Gemini de Google): detección de incidentes, agrupación, clasificación (crítico / alto / medio / bajo), mapeo MITRE ATT&CK, evidencia y redacción de todos los textos del informe. La aplicación normaliza los datos, valida la respuesta estructurada del agente y compone el PDF — no aplica reglas de detección locales ni plantillas.

---

## Estructura del repositorio

soc-l2-iagen-proyecto-final/

│

├── [README.md](./readme.md) → Índice principal (Este doc)

├── [index.html](./index.html) → Aplicación web 

├── [Arquitectura_3_capas_IA.png](./Arquitectura_3_capas_IA.png) → Arquitectura de app web

├── [SOC_L2_Logs_Ampliados_Multiples_Incidentes.xlsx](./SOC_L2_Logs_Ampliados_Multiples_Incidentes.xlsx) → Dataset del cliente

└── Informe_Incidente_SOC2_AAAAMMDD.pdf               → Informe generado

---

## Cómo ejecutarla

1. Descarga `index.html` (todo el proyecto es un único archivo).
2. Ábrelo con doble clic en tu navegador (Chrome, Edge o Firefox). Necesita internet: las librerías cargan por CDN (SheetJS, Chart.js, jsPDF, autotable) y el análisis se ejecuta en la API de Gemini.
3. Crea una clave gratuita en [aistudio.google.com/apikey](https://aistudio.google.com/apikey), pégala en la pestaña **2 · Agente de IA** y pulsa **Guardar clave** (modelo por defecto `gemini-3.8-flash`; el botón **«🔍 Detectar»** lista los modelos activos de tu cuenta).
4. Carga el Excel del incidente (o el dataset de demostración incluido) en la pestaña 1.
5. Pulsa **«✨ Analizar dataset con IA»** en la pestaña 2 y revisa dashboard, incidentes y genera el informe PDF.

---

## Las 5 pestañas

| # | Pestaña | Qué hace |
|---|---|---|
| 1 | **Cargar datos** | Lee y normaliza el Excel del SIEM en el navegador (mapeo automático de columnas Log_ID, Timestamp, Hostname, Event_ID, User, Source_IP...), resumen del dataset y datos de la portada del informe (analista, consultora, cliente, clasificación TLP). |
| 2 | **Agente de IA** | Conexión con Gemini (clave + modelo) y botón «Analizar dataset con IA»: detección, clasificación, evidencia y redacción de resumen ejecutivo, resumen técnico y recomendaciones. Incluye chat con el agente sobre su propio análisis. |
| 3 | **Dashboard** | Lectura del agente, KPIs, tarjetas por severidad (crítico/alto/medio/bajo) y 4 gráficas: severidad asignada por la IA, actividad en el tiempo, categorías de ataque e IPs implicadas. |
| 4 | **Incidentes** | Una tarjeta por incidente detectado por la IA: severidad, categoría, MITRE ATT&CK, descripción, evidencia citada y tabla con las **filas reales del Excel** que la sustentan, más recomendaciones. |
| 5 | **Informe PDF** | Vista previa del contenido (100 % IA) y generación del informe formal con jsPDF: portada TLP, resumen ejecutivo, indicadores clave, clasificación de riesgos con gráficas, detalle de incidentes con evidencia, resumen técnico, recomendaciones priorizadas y anexo metodológico. |

---

## Arquitectura (100 % IA generativa)

```
 Excel del SIEM (.xlsx)
 │ (SheetJS, en el navegador — normalización y payload TSV)
 │ 
 ├── [Capa 1] Carga y normalización de datos
 │      ├── Mapeo automático de columnas del export del SIEM
 │      ├── Resumen del dataset (eventos, columnas, equipos, cuentas, ventana)
 │      └── Payload compacto (TSV) listo para el agente
 │
 ├── [Capa 2] Agente de IA generativa — Gemini de Google
 │      ├── Recibe el dataset completo y hace TODO el análisis
 │      ├── Devuelve un JSON ESTRUCTURADO (responseSchema):
 │      │   incidentes (severidad, categoría, MITRE, momento, descripción,
 │      │   evidencia, Log_IDs, IPs, cuentas, equipos, recomendaciones)
 │      │   + resumen ejecutivo + resumen técnico + recomendaciones
 │      └── La app valida y normaliza el JSON (reintentos y fallbacks incluidos)
 │
 └── [Capa 3] Generador del informe PDF (jsPDF + autotable + Chart.js)
        ├── Portada con clasificación TLP, resumen ejecutivo, indicadores clave
        ├── Clasificación de riesgos con gráficas, detalle de incidentes con
        │   Evidencia (filas reales del Excel), resumen técnico.
        └── Recomendaciones priorizadas, anexo metodológico, paginación dinámica
```

---

## Cómo funciona el agente

- La pestaña 1 convierte el Excel en una tabla TSV compacta (máx. 1500 filas con muestreo uniforme) y la pestaña 2 la envía a `generativelanguage.googleapis.com` con un **schema JSON estricto**: el modelo no puede responder "texto libre", debe devolver la estructura completa.
- La app **valida y normaliza** la respuesta: severidades canónicas (`critico/alto/medio/bajo`), IDs `INC-01...` reordenados por severidad, listas coerced y estadísticas derivadas (conteos por severidad, categorías, top IPs).
- La **evidencia es verificable**: el agente cita `Log_ID` de filas reales del Excel; la pestaña 4 y el PDF muestran esas filas tal cual están en el archivo.
- El **chat** de la pestaña 2 responde únicamente con base en el JSON del análisis (cita INC-XX y cifras exactas).
- Reintentos automáticos ante 429/5xx, fallback si el modelo no soporta `responseSchema` (extrae el JSON del texto), y mensajes claros para clave inválida, cuota, modelo inexistente o bloqueo de ubicación.

> Nota: al ser un análisis de IA generativa, la redacción puede variar ligeramente entre ejecuciones; la evidencia siempre cita Log_IDs del Excel, que puedes verificar uno a uno.

---

## Privacidad

- La **clave API se guarda solo en `localStorage`** de tu navegador: nunca va dentro de los archivos ni del repositorio.
- Para analizar, el contenido del dataset **viaja a la API de Gemini de Google** — así funciona una app integrada con IA generativa. El dataset incluido es sintético de práctica.
- La app no envía nada a ningún otro servicio; no hay telemetría ni backend.

---
## Stack

<div align="center">
 
[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)](https://developer.mozilla.org/es/docs/Web/HTML) [![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)](https://developer.mozilla.org/es/docs/Web/CSS) [![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/es/docs/Web/JavaScript) [![SheetJS](https://img.shields.io/badge/SheetJS-01723a?style=for-the-badge)](https://sheetjs.com/) [![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)](https://www.chartjs.org/) [![jsPDF](https://img.shields.io/badge/jsPDF-D82928?style=for-the-badge)](https://parall.ax/products/jspdf) [![jsPDF-AutoTable](https://img.shields.io/badge/jsPDF_AutoTable-B30B00?style=for-the-badge)](https://github.com/simonbengtsson/jsPDF-AutoTable) [![Google Gemini API](https://img.shields.io/badge/Gemini_API-8E75B2?style=for-the-badge&logo=googlegemini&logoColor=white)](https://ai.google.dev/)
</div>

---

## Autor

Alex Vicente Alcivar Moya - IT Engineer 

---

## Redes Sociales

[![LinkedIn](https://img.shields.io/badge/Linkedin-000000?style=for-the-badge&logo=linkedin&logoColor=00ff41)](https://www.linkedin.com/in/alexalcivarm7) [![GitHub](https://img.shields.io/badge/GitHub-000000?style=for-the-badge&logo=github&logoColor=00ff41)](https://github.com/alexalcivarm7) [![TryHackMe](https://img.shields.io/badge/Tryhackme-000000?style=for-the-badge&logo=tryhackme&logoColor=00ff41)](https://tryhackme.com/p/alexalcivarm7)

---

>  *Escenario sintético de práctica; los datos del dataset son ficticios (IPs de documentación RFC 5737 / RFC 2606). Material educativo.*
>
<div align="center">
<a href="https://github.com/alexalcivarm7/soc-l2-iagen-proyecto-final">
  <img src="https://komarev.com/ghpvc/?username=alexalcivarm7-soc-l2&label=REPO+VIEWS&color=1F6FEB&style=for-the-badge" alt="Repo Views"/>
</a>
</div>

---


