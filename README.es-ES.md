# Cryptex

Cryptex es un kit de herramientas de criptoanálisis clásico para flujos de trabajo educativos y de investigación.

Incluye:
- Motores de cifrado para sustitución, Vigenere, transposición columnar, Playfair, afino y rail fence.
- Solucionadores de cracking para todas las principales familias de cifrados compatibles, incluyendo el modo de texto plano conocido restringido.
- Puntuación adaptativa de n-gramas con modelos de lenguaje Kneser-Ney.
- Autodetección y orquestación de solucionadores competitivos.
- Utilidades de evaluación para convergencia, benchmarking, comportamiento de transición de fase, pruebas de estrés y paquetes de desafíos históricos.

## Requisitos

- Python 3.14+
- Administrador de paquetes uv: https://docs.astral.sh/uv/getting-started/installation/

## Instalación

```bash
uv sync --dev
```

## Inicio Rápido

```bash
# Entrenar modelos de lenguaje (la primera ejecución descarga el corpus)
uv run cryptex train

# Demo de sustitución en vivo
uv run cryptex demo --cipher substitution --method mcmc

# Crackear texto desconocido con enrutamiento automático
uv run cryptex crack --auto --text "xqzv 9988 @@ lxfopv ef rnhr -- ???"

# Ejecutar suite de pruebas
uv run pytest -q
```

## Qué está implementado

### Motores de cifrado

- `substitution`: sustitución monoalfabética.
- `vigenere`: cifrado polialfabético de Vigenere.
- `transposition`: cifrado de transposición columnar.
- `playfair`: cifrado de dígrafos Playfair.
- `affine`: transformación monoalfabética afina.
- `railfence`: transposición de rail fence.
- `noisy-substitution`: sustitución envuelta con ruido de corrupción/eliminación.

### Solucionadores y analizadores

- `MCMC`: recocido simulado adaptativo con templado opcional y propuestas de cruce.
- `HMM/EM`: decodificación probabilística tolerante a `noisy-substitution`.
- `Genetic`: búsqueda de sustitución basada en población para comparación.
- `Vigenere cracker`: IoC + Kasiski + subproblemas de César puntuados por n-gramas.
- `Transposition cracker`: búsqueda MCMC sobre permutaciones de columnas.
- `Playfair cracker`: ascenso de colina basado en población con operadores de movimiento.
- `Affine cracker`: búsqueda exhaustiva sobre claves `(a, b)` válidas.
- `Rail fence cracker`: búsqueda de recuento de rieles con puntuación de n-gramas.
- `KPA`: extracción de pistas de texto plano conocido y cracking de sustitución restringido.
- `Detector`: predicción de familia de cifrado basada en características con confianza y razonamiento.

### Núcleo de lenguaje y puntuación

- Modelo de lenguaje de n-gramas Kneser-Ney (variantes con y sin espacio).
- Ganchos de soporte multi-idioma y detección de idioma basada en frecuencia (`english`, `french`, `german`, `spanish`).
- Pipeline de E/S de mapeo fantasma robusto para preservar la puntuación/mayúsculas alrededor del cracking de sustitución.

## Descripción General de Comandos

| Comando | Propósito |
| --- | --- |
| `uv run cryptex train [--force]` | Descargar corpus y entrenar/cargar modelos de lenguaje |
| `uv run cryptex demo ...` | Cifrar texto de muestra y crackear en vivo |
| `uv run cryptex crack ...` | Crackear texto cifrado personalizado desde texto/archivo/stdin |
| `uv run cryptex detect ...` | Predecir la familia del cifrado e imprimir diagnósticos de características |
| `uv run cryptex analyse ...` | Diagnósticos de convergencia MCMC + reportes JSON |
| `uv run cryptex benchmark ...` | Comparar MCMC, GA, línea base de frecuencia, línea base de ascenso de colina |
| `uv run cryptex phase-transition ...` | Tasa de éxito vs longitud del texto cifrado |
| `uv run cryptex stress-test` | Ejecutar suite de robustez adversaria |
| `uv run cryptex historical` | Ejecutar desafíos históricos seleccionados |
| `uv run cryptex language ...` | Entrenar/listar/detectar modelos de lenguaje |
| `uv run cryptex kpa ...` | Extracción de pistas de texto plano conocido |

Ejecuta la ayuda en cualquier momento:

```bash
uv run cryptex --help
uv run cryptex crack --help
```

## Manual de Ejecución Completo

### 1) Entrenar modelos

```bash
uv run cryptex train
```

### 2) Ejecutar cada ruta de demo de cifrado/método

```bash
uv run cryptex demo --cipher substitution --method mcmc
uv run cryptex demo --cipher substitution --method hmm
uv run cryptex demo --cipher substitution --method genetic

uv run cryptex demo --cipher vigenere
uv run cryptex demo --cipher transposition
uv run cryptex demo --cipher playfair
uv run cryptex demo --cipher affine
uv run cryptex demo --cipher railfence
uv run cryptex demo --cipher noisy-substitution --method mcmc
```

### 3) Estilos de entrada del comando crack

Texto directo:

```bash
uv run cryptex crack --cipher substitution --method mcmc --text "wkh txlfn eurzq ira mxpsv ryhu wkh odcb grj"
```

Archivo:

```bash
uv run cryptex crack --cipher vigenere --file .\samples\vig.txt
```

stdin:

```bash
Get-Content .\samples\mystery.txt | uv run cryptex crack --cipher substitution
```

Autodetección + cracking competitivo:

```bash
uv run cryptex crack --auto --text "xqzv 9988 @@ lxfopv ef rnhr -- ???"
```

Salidas JSON/plain, tiempo de espera (timeout), top-k candidatos:

```bash
uv run cryptex crack --cipher substitution --text "..." --output-format json --top-k 3 --timeout 30
uv run cryptex crack --cipher substitution --text "..." --output-format plain --top-k 5
```

Cracking restringido por texto plano conocido:

```bash
uv run cryptex crack --cipher substitution --text "..." --known-pair "truth:abcde" --known-pair "single:qwert"
```

Ayudante de extracción de pistas KPA:

```bash
uv run cryptex kpa --ciphertext "..." --known "knownword"
```

Para las rutas de `substitution`/`noisy-substitution`, el mapeo fantasma preserva la puntuación/diseño y restaura las mayúsculas en la salida final.

### 4) Detección

```bash
uv run cryptex detect --text "lxfopvefrnhr"
uv run cryptex detect --file .\unknown.txt
```

### 5) Reportes de análisis y benchmark

```bash
uv run cryptex analyse --output .\plots
uv run cryptex benchmark --length 300 --trials 3 --success-threshold 0.20 --output .\plots
uv run cryptex phase-transition --trials 3 --output .\plots
```

Artefactos JSON generados:
- `plots/convergence_plot.json`
- `plots/key_heatmap.json`
- `plots/frequency_comparison.json`
- `plots/benchmark.json`
- `plots/phase_transition.json`

### 6) Flujos de trabajo de estrés, históricos y de lenguaje

```bash
uv run cryptex stress-test
uv run cryptex historical

uv run cryptex language list
uv run cryptex language train --lang french
uv run cryptex language train --lang german
uv run cryptex language train --lang spanish
uv run cryptex language detect --text "bonjour le monde"
```

Etiquetas de idioma compatibles: `english`, `french`, `german`, `spanish`.

## Artefactos de Benchmark y Validación

Utiliza el arnés de benchmark del proyecto para ejecutar un barrido personalizado de 10 textos a través de los algoritmos implementados:

```bash
uv run python .\tools\run_custom_benchmarks.py
```

Salida principal:
- `plots/custom_benchmark_suite.json`

La discusión detallada del benchmark y los resultados medidos se mantienen en `BENCHMARKS.md`.

### Instantánea de Mediciones

De `plots/benchmark.json` (benchmark de sustitución integrado):

| Método | SER Promedio | Tasa de Éxito | Tiempo Promedio (s) |
| --- | ---: | ---: | ---: |
| MCMC | 0.0000 | 1.0000 | 10.4524 |
| Genetic Algorithm | 0.0000 | 1.0000 | 8.5106 |
| Frequency Analysis | 0.6231 | 0.0000 | 0.0002 |
| Hill Climb + Freq Init | 0.2828 | 0.6667 | 0.1514 |

De `plots/custom_benchmark_suite.json` (10 textos personalizados, todos los algoritmos implementados):
- Ejecuciones totales del solucionador: 130
- Pruebas de codificación/decodificación roundtrip: 10/10 aprobadas para substitution, vigenere, transposition, playfair, affine, rail fence y noisy-substitution (ruta de decodificación best-effort).
- Precisión del detector de cifrado: 39/60 (65.0%)
- Precisión de detección de idioma: 9/10 (90.0%)

Consulta `BENCHMARKS.md` para ver la clasificación completa por algoritmo y su interpretación.

## Puertas de Calidad de Desarrollo

```bash
uvx ruff check --fix
uvx ruff format
uvx ty check
uv run pytest -q
```

## Limitaciones Conocidas

- Los textos cifrados muy cortos pueden seguir siendo ambiguos.
- Playfair es el modo más difícil y puede requerir textos más largos y presupuestos de iteración más altos.
- Los payloads que no sean exclusivamente alfabéticos fallan limpiamente con `Error: no decipherable alphabetic content found.`.
- Los archivos faltantes fallan limpiamente con `Error reading file '<path>': ...`.

## Documentación Adicional

- `BENCHMARKS.md` para comandos de benchmark reproducibles y resultados medidos del proyecto.
