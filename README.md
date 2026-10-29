# Convergia · Series de Taylor y Maclaurin

Aplicación web interactiva que aproxima cualquier función f(x) mediante **series de Taylor y Maclaurin** y muestra, término por término, cómo la aproximación converge al valor real.

**Tecnologías:** Python · Flask · SymPy · HTML · CSS · JavaScript (Chart.js y KaTeX por CDN)

---

## Funcionalidades

**Entradas**

| Campo | Descripción |
|---|---|
| `f(x)` | Función arbitraria, ej. `sin(x)`, `x^3 + 2*x`, `exp(-x^2)` |
| `x` | Punto donde se quiere aproximar la función |
| `a` | Punto de desarrollo. Vacío o `0` → **Maclaurin**; distinto de 0 → **Taylor** |
| `N` | Número de términos (1 a 40) |

**Salidas**

- Anillos de resumen: precisión %, términos usados y dígitos exactos.
- Valor real, aproximación final, error absoluto y error relativo (también en %).
- La función y el polinomio resultante en notación matemática.
- **Tabla de iteraciones:** n, f⁽ⁿ⁾(a), término, sumatoria, error absoluto y error relativo (descargable en CSV).
- **Gráficas:** Iteración vs. Términos, Iteración vs. Sumatoria e Iteración vs. Errores (escala logarítmica).

---

## Fundamento matemático

Serie de Taylor de `f` alrededor de `a`:

```
f(x) ≈ Σ [ f⁽ⁿ⁾(a) / n! · (x − a)ⁿ ]      n = 0 … N−1
```

Serie de Maclaurin (caso particular `a = 0`):

```
f(x) ≈ Σ [ f⁽ⁿ⁾(0) / n! · xⁿ ]            n = 0 … N−1
```

Errores en cada iteración (`S_n` = suma acumulada):

```
Error absoluto   = | real − S_n |
Error relativo   = | (real − S_n) / real |
Error relativo % = Error relativo · 100
```

> La serie solo converge dentro de su radio de convergencia. Por ejemplo, `1/(1-x)` alrededor de 0 converge para `|x| < 1`; con `x = 2` la sumatoria diverge.

---

## Instalación y ejecución

**Requisitos:** Python 3.9 o superior, un navegador e internet (Chart.js y KaTeX se cargan desde CDN).

### Windows

1. Instala Python desde [python.org](https://www.python.org/downloads/) marcando **“Add Python to PATH”**.
2. **Forma rápida:** doble clic en `iniciar.bat`.
3. **Forma manual** (CMD dentro de la carpeta del proyecto):

```bat
py -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Las siguientes veces solo hacen falta `venv\Scripts\activate` y `python app.py`.

### Linux / macOS

1. Instala Python y venv (Ubuntu/Debian: `sudo apt install python3 python3-venv python3-pip`).
2. **Forma rápida:**

```bash
chmod +x iniciar.sh
./iniciar.sh
```

3. **Forma manual:**

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

Luego abre **http://127.0.0.1:5000** en el navegador. Para detener el servidor, pulsa `Ctrl + C`.

### Problemas frecuentes

| Problema | Solución |
|---|---|
| `No module named 'sympy'` o `'flask'` | Activa el entorno (`venv\Scripts\activate` / `source venv/bin/activate`) y ejecuta `pip install -r requirements.txt` |
| `'py' no se reconoce` (Windows) | Usa `python` o reinstala Python marcando *Add Python to PATH* |
| `ensurepip is not available` (Linux) | `sudo apt install python3-venv` |
| `Permission denied` con `./iniciar.sh` | `chmod +x iniciar.sh` |
| Puerto 5000 ocupado | Cambia a `app.run(port=5001, debug=True)` en `app.py` |
| El diseño se ve viejo | Recarga con `Ctrl + F5` |
| Sin gráficas ni fórmulas | Revisa la conexión a internet (CDN) |

---

## Ejemplos para probar

| f(x) | x | a | N | Valor real | Qué demuestra |
|---|---|---|---|---|---|
| `exp(x)` | 1 | — | 10 | 2.7182818 | Maclaurin clásica |
| `sin(x)` | `pi/2` | — | 8 | 1 | Términos que alternan de signo |
| `cos(x)` | 1 | — | 6 | 0.5403023 | Solo términos pares |
| `exp(-x^2)` | 0.5 | — | 12 | 0.7788008 | Campana de Gauss |
| `atan(x)` | 0.5 | — | 10 | 0.4636476 | Convergencia más lenta |
| `ln(x)` | 1.5 | 1 | 10 | 0.4054651 | Taylor con `a ≠ 0` |
| `sqrt(x)` | 1.2 | 1 | 8 | 1.0954451 | Taylor alrededor de 1 |
| `x^3 + 2*x` | 2 | — | 4 | 12 | Polinomio: error 0 si N supera el grado |
| `1/(1-x)` | 2 | — | 10 | −1 | **Diverge** (fuera del radio de convergencia) |

Casos que dan error a propósito: `sqrt(x)` o `ln(x)` con `a` vacío (no son derivables en 0).

---

## Estructura del proyecto

```
taylor_app/
├── app.py               # Servidor Flask + cálculo con SymPy (POST /api/calcular)
├── requirements.txt     # Dependencias
├── iniciar.bat          # Arranque con doble clic (Windows)
├── iniciar.sh           # Arranque con un comando (Linux/macOS)
├── templates/
│   └── index.html       # Estructura de la página
└── static/
    ├── css/style.css    # Diseño (tema oscuro, responsive)
    └── js/app.js        # Lógica del navegador, tabla y gráficas
```

---

## Cómo funciona

```
Navegador (HTML/CSS/JS)                  Servidor (Flask + SymPy)
1. El usuario llena el formulario
2. app.js envía JSON a
   POST /api/calcular {f, x, a, n} ───►  3. Valida y convierte los textos a SymPy
                                         4. Calcula el valor real f(x)
                                         5. Para n = 0 … N−1:
                                            · deriva f n veces (sp.diff)
                                            · evalúa f⁽ⁿ⁾(a) y divide entre n!
                                            · término = coef · (x − a)ⁿ
                                            · acumula la suma y calcula errores
7. app.js dibuja anillos, tabla  ◄───    6. Responde un JSON con las filas,
   y gráficas                               el valor real y el LaTeX
```

- **Derivadas simbólicas exactas** con SymPy (no numéricas), por lo que f⁽ⁿ⁾(a) no acumula error de redondeo.
- **Seguridad:** solo se aceptan nombres de una lista blanca (`sin`, `cos`, `exp`, `log`, `sqrt`, `pi`, `e`…) y se rechazan caracteres peligrosos; no se ejecuta código arbitrario.
- **Dígitos exactos:** `−log₁₀(error relativo)`, con tope de 15 (límite de un decimal de 64 bits).
- **Casos especiales:** si el valor real es 0 el error relativo no está definido (se muestra “—”); el polinomio simbólico solo se muestra para N ≤ 12.

---

## Cómo leer los resultados

- **Términos:** cada barra es la contribución de un término; cuando son diminutos, la serie ya convergió (rosa = negativo).
- **Sumatoria:** la curva azul debe acercarse a la línea naranja punteada (valor real).
- **Errores:** una línea que baja rápido indica convergencia rápida; si no baja, `x` está fuera del radio de convergencia o demasiado lejos de `a`.
