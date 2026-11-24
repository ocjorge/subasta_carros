# Subastas de Autos Clásicos · Fase 1: Scraping

Notebook que recupera **250 subastas finalizadas** de [Bring a Trailer](https://bringatrailer.com/auctions/results/) y las guarda en un CSV crudo (`subastas_carros.csv`). Es la primera fase del proyecto de análisis descriptivo de precios y características de autos clásicos (Flet + MongoDB).

## Contenido

| Archivo | Descripción |
|---|---|
| `scraping_subastas_carros.ipynb` | Script de scraping completo, con la explicación de las expresiones regulares |
| `subastas_carros.csv` | Salida del scraping (se genera al ejecutar el notebook) |

## Requisitos

- Python 3.10 o superior (se usa la sintaxis `dict | None`)
- Librerías:

```bash
pip install requests beautifulsoup4 lxml pandas jupyter
```

## Uso

1. Abrir el notebook: `jupyter notebook scraping_subastas_carros.ipynb`
2. Ejecutar las celdas en orden (`Run All`).
3. La celda final de ejecución llama a:

```python
ejecutar_scraping(paginas=8, max_subastas=250, archivo_salida="subastas_carros.csv")
```

Tarda varios minutos porque se pausa de 1.5 a 3 segundos entre peticiones para no saturar el sitio.

## Cómo funciona

1. **`extraer_json_desde_script`**: localiza el JSON embebido en un `<script>` de la página de resultados (`auctionsCompletedInitialData`) y lo convierte a diccionario.
2. **`obtener_urls_subastas`**: toma las URLs de la página 1 desde ese JSON, pide las páginas 2 a 8 a la API de paginación (36 por página) y, si no alcanzó, completa por década desde el HTML. Elimina duplicados conservando el orden.
3. **`extraer_campos`**: descarga cada subasta y extrae los campos con BeautifulSoup y expresiones regulares.
4. **`ejecutar_scraping`**: recorre las URLs (máximo 250) y escribe cada fila con `csv.DictWriter` (`utf-8`, `newline=""`).

## Columnas del CSV

| Columna | Contenido | Ejemplo |
|---|---|---|
| `url` | Enlace de la subasta | `https://bringatrailer.com/listing/...` |
| `titulo` | Título completo (incluye año, marca, modelo y "No Reserve:" si aplica) | `No Reserve: 1987 Porsche 911 Carrera` |
| `resultado_subasta` | Precio y fecha finales | `Sold for USD $52,000 on 5/12/25` |
| `marca` | Viñeta de marca | `Make: Porsche` |
| `modelo` | Viñeta de modelo | `Model: 911` |
| `kilometraje` | Viñeta de distancia | `45k Miles Shown` |
| `transmision` | Viñeta de transmisión | `5-Speed Manual Transaxle` |
| `titulo_legal` | Estatus legal del título | `Clean California Title` |
| `ubicacion` | Viñeta de ubicación | `Location: Austin, Texas` |
| `fotos` | Texto con el número de fotos | `View All 24 Photos` |
| `comentarios` | Texto con el número de comentarios | `42 Comments` |

Los datos se guardan **crudos**, tal como aparecen en la página. El parseo de precios, kilometraje, año, etc. se hace en la Fase 2 (limpieza). El año no tiene columna propia: está dentro de `titulo`.

## Expresiones regulares

Se usan con `re.search`, que devuelve la primera coincidencia en cualquier parte del texto. El detalle de cada una está en la sección 4 del notebook.

| Campo | Patrón | Qué detecta |
|---|---|---|
| Resultado | `(?:Sold for\|Bid to)\s+(?:USD\s+)?\$[\d,]+\s+on\s+\d{1,2}/\d{1,2}/\d{2,4}` | "Sold for" o "Bid to", monto con `$` y fecha mes/día/año |
| Kilometraje | `\b(miles\|kilometers\|km\|tmu)\b` | Viñeta con unidad de distancia |
| Transmisión | `\b(manual\|automatic\|pdk\|dsg\|tiptronic\|cvt\|transmission\|gearbox\|transaxle\|speed)\b` | Viñeta con palabras de transmisión |
| Título legal | `\b(?:Clean\|Salvage\|Rebuilt\|Reconstructed\|Duplicate\|Bonded)\s+(?:[A-Za-z]+\s+){0,2}Title\b\|\bBill of Sale\b` | Estatus + hasta 2 palabras + "Title", o "Bill of Sale" |
| Fotos | `(?:View\s+All\s+)?\d[\d,]*\s+Photos?\b` | "View All 24 Photos", "50 Photos", "view all 3 photos" |
| Comentarios | `\d[\d,]*\s+Comments?\b` | "42 Comments", "150 comments", "0 COMMENTS" |

Todas se aplican con `re.IGNORECASE`.

## Notas

- Si un campo no se encuentra en una página queda vacío en el CSV.
- Las páginas que responden con un código distinto de 200 se omiten y se muestra un aviso, por lo que el CSV puede tener menos de 250 filas.
- El sitio puede cambiar su estructura HTML o bloquear peticiones; si el CSV sale vacío, revisar los selectores y los encabezados `HEADERS`.

## Siguientes fases

2. Limpieza y preprocesamiento (`autos_limpios.csv` + registro de reglas)
3. Carga a MongoDB (`proyecto_autos.subastas`)
4. Gráficas descriptivas y aplicación Flet
