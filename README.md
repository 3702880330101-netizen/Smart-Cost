# SmartCost AI

**Un gemelo economico para experimentar antes de decidir.**

Prototipo de la Feria de Innovacion 2026 para **Ingenieria de Costos**. Responde una pregunta
concreta: *si cambio algo, ¿realmente mejora el resultado, o solo suena a que lo hace?*

En lugar de cambiar el proceso y averiguarlo despues, el prototipo simula el cambio con las
mismas nueve formulas del curso y mide cuanto mejora o empeora la utilidad.

---

## La idea en un minuto

Un estudiante de Ingenieria de Costos tiene un costo y una utilidad. Subir el precio del
material, mejorar un proceso o subir el volumen "suena" a mejora, pero casi nunca lo es en la
misma medida. El riesgo real es mover la variable equivocada y empeorar el resultado sin
notarlo.

SmartCost AI es una copia ejecutable de ese calculo. Se le cambian los datos, se corre, y el
motor dice que paso con numeros.

```
9 variables  ->  9 formulas  ->  4 escenarios  ->  sensibilidad  ->  IA interpreta
                 (Python)      (mismo motor)     (impacto medido)   (no calcula)
```

---

## Que hace y que no hace

**Hace:**

- Calcula costo, margen, punto de equilibrio y utilidad de un escenario de nueve variables.
- Genera cuatro escenarios y compara cuanto cambia la utilidad en cada uno.
- Mede cuanto impacta cada variable y las ordena de mayor a menor.
- Entrega esos numeros a una IA, que los traduce a lenguaje de negocio.
- Compara sus resultados contra una hoja de calculo independiente, con tolerancia de 0.000001.

**No hace, y es importante decirlo:**

- No predice la demanda. El volumen es un dato que ingresa el usuario.
- No modela inventario, fluteo, impuestos, financiamiento ni tiempo del dinero.
- No conoce la capacidad instalada ni los tiempos de entrega.
- No cubre varios productos: es un gemelo de una sola operacion.

---

## Las tres reglas del proyecto

**1. Las nueve formulas son el unico calculo.** Viven programadas en Python, en `colab/motor.py`,
y se pueden leer linea por linea. La pagina web **no** replica ninguna: por eso no puede
discrepar del motor. Hay un chequeo automatico (`tools/verificar_proyecto.py`) que lo verifica
buscando formulas en el codigo del frontend.

**2. La IA interpreta, no calcula.** Recibe un JSON con resultados ya calculados y devuelve
diagnostico, variables criticas, acciones y riesgos. El prompt le dice explicitamente que no
sustituya ninguna formula. Despues, todo monto que menciona se busca en los resultados del
motor; lo que no existe se reporta como dato no verificado.

**3. La hoja de control manda.** Cada prueba compara el motor contra una hoja escrita aparte.
Un modelo que solo se valida a si mismo no esta validado.

---

## Las nueve variables y las nueve formulas

| Clave tecnica | Que es | Unidad |
|---------------|--------|--------|
| `material` | Consumo estandar por unidad | kg / unidad |
| `precio_material` | Costo de compra por unidad de material | Q / unidad |
| `desperdicio` | Consumo adicional | **%** |
| `horas_mod` | Horas de mano de obra directa por unidad | h / unidad |
| `tarifa_mod` | Costo por hora de mano de obra directa | Q / hora |
| `cif_variable` | Costo indirecto de fabricacion variable | Q / unidad |
| `costos_fijos` | Costos fijos del periodo | Q / periodo |
| `volumen` | Unidades a producir y vender | unidades |
| `precio_venta` | Precio de venta por unidad | Q / unidad |

| # | Resultado | Formula |
|---|-----------|---------|
| 1 | Material ajustado | `=material*(1+desperdicio/100)` |
| 2 | Costo MP unitario | `=1*precio_material` |
| 3 | Costo MOD unitario | `=horas_mod*tarifa_mod` |
| 4 | Costo variable unitario | `=2+3+cif_variable` |
| 5 | Costo total | `=4*volumen+costos_fijos` |
| 6 | Costo unitario | `=5/volumen` |
| 7 | Margen contribucion unitario | `=precio_venta-4` |
| 8 | Margen contribucion % | `=7/precio_venta*100` |
| 9 | Margen contribucion total | `=7*volumen` |
| 10 | Punto de equilibrio | `=SI(7<=0;NO CALCULABLE;costos_fijos/7)` |
| 11 | Utilidad | `=7*volumen-costos_fijos` |

Detalle completo, con la version programada al lado de la de hoja de calculo:
[`docs/formulas.md`](docs/formulas.md).

---

## Como se ejecuta

### 1. El motor: Google Colab

1. Abrir `SmartCost_AI.ipynb` en Google Colab.
2. En la Seccion 2, instalar dependencias: `pip install -r requirements.txt`.
3. En la Seccion 3, cargar los modulos. O se pega la URL del repositorio, o se suben los
   ocho `.py` de `colab/` con el selector de archivos de Colab.
4. Ejecutar de arriba abajo, o ir a la Seccion 6 para ver el caso base.

Para que la IA funcione, hay que crear el secreto **`GEMINI_API_KEY`** en el panel de secretos
de Colab, con acceso al notebook. **La clave nunca se escribe en el codigo, ni en la pagina
web.** Sin ella el sistema igual funciona: devuelve una revision generada por el motor y la
marca como tal, sin hacerse pasar por IA.

### 2. La interfaz: GitHub Pages

Publicar el repositorio en GitHub con Pages activado, y abrir la URL que entrega GitHub.

1. En la **Seccion 0** de la pagina, pegar la URL que imprime la Seccion 12 del notebook
   (el tunel de ngrok). Se recuerda en el navegador.
2. Presionar **Calcular escenario**.

La URL del tunel **cambia en cada sesion de Colab**: hay que volver a pegarla. El boton
**Guardar URL** la deja guardada en el navegador, y **Probar conexion** confirma que el motor
responde.

---

## Estructura del proyecto

```
SmartCost-AI/
|-- index.html              Pagina de la interfaz (GitHub Pages)
|-- styles.css              Estilos del tablero
|-- script.js               Logica del frontend. NO calcula nada de costos
|-- SmartCost_AI.ipynb      Notebook de Colab: 13 secciones, motor + API
|-- requirements.txt        Dependencias (Flask, ngrok, cliente de Gemini)
|-- .nojekyll               Necesario para que GitHub Pages sirva la pagina
|
|-- colab/                  MOTOR DE CALCULO. La logica de negocio vive aqui
|   |-- config.py             Las 9 variables, limites, caso base, deltas, sin claves
|   |-- motor.py              Las 9 formulas, validaciones, trazabilidad
|   |-- escenarios.py         Base, nuevo, optimista, adverso; tablas y graficos
|   |-- sensibilidad.py       Impacto por variable y ranking
|   |-- ia_gemini.py          Prompt, payload, auditoria, bitacora, fallback sin IA
|   |-- hoja_control.py       Formulas y valores esperados, written aparte
|   |-- pruebas.py            Las 4 pruebas obligatorias y 6 de control
|   `-- api_server.py         API HTTP: /health /contrato /simulate /pruebas
|
|-- docs/
|   |-- formulas.md           Las 9 formulas y las reglas, con el codigo al lado
|   |-- hoja_control.md       Diseno de la hoja y los 4 casos esperados
|   `-- bitacora_ia.md        Plantilla de registro de uso de IA
|
|-- tests/
|   `-- test_smartcost.py     Suite en pytest (59 pruebas)
|
`-- tools/
    |-- verificar_proyecto.py  Auditoria integral: corre la suite y delega en las dos de abajo
    |-- verificar_frontend.py  ids, balance, CDN de Chart.js y ausencia de formulas en el .js
    `-- build_notebook.py     Verifica la estructura del .ipynb (13 secciones, sin formulas duplicadas)
```

La regla de oro de la estructura: **la logica de negocio esta solo en `colab/`**. El notebook
carga esos modulos en vez de copiarlos, y el frontend no los reimplementa. Asi hay una sola
copia de las formulas en todo el proyecto, y se puede auditar.

---

## La API

El motor se expone por HTTP para que la pagina web lo use.

| Ruta | Metodo | Que hace |
|------|--------|----------|
| `/health` | GET | Estado del motor y si hay clave de IA |
| `/contrato` | GET | Las 9 entradas con sus limites, y los deltas por defecto |
| `/simulate` | POST | Simulacion completa: resultados, escenarios, sensibilidad, graficos, IA |
| `/pruebas` | GET | Corre las 10 pruebas y devuelve el informe |
| `/bitacora` | GET | Bitacora de uso de la IA de la sesion |

Las cinco rutas responden igual con Flask y con el servidor de respaldo de la libreria
estandar. Eso lo cubre `pytest`, porque el prototipo tiene que funcionar en la maquina de
quien lo demuestra, con o sin dependencias instaladas.

Cuerpo minimo de `POST /simulate`:

```json
{
  "material": 0.80, "precio_material": 18.00, "desperdicio": 4,
  "horas_mod": 0.35, "tarifa_mod": 28.00, "cif_variable": 4.50,
  "costos_fijos": 42000, "volumen": 4000, "precio_venta": 58.00
}
```

Opcionales: `escenario_base` (referencia para optimista/adverso), `escenarios`
(`{"optimista": {...}, "adverso": {...}}` con deltas en %), `sensibilidad`
(`{"paso_relativo": 0.10}`), `ia` (`{"activar": true}`), `modo`.

---

## Escenarios

| Escenario | Como se construye |
|-----------|-------------------|
| `base` | Los datos fijados como referencia |
| `nuevo` | Lo que el usuario esta escribiendo ahora |
| `optimista` | La referencia con las mejoras declaradas |
| `adverso` | La referencia con los retrocesos declarados |

Los cuatro salen de **la misma funcion**, `calcular_escenario()`, asi que son comparables por
construccion.

Los porcentajes por defecto se editan desde la pagina web antes de simular, y quedan a la
vista. `desperdicio -50 %` significa que el desperdicio **se reduce a la mitad**, no que se
resta 50 puntos. El motor acota los resultados a los rangos validos y despues declara en la
tabla de cambios cualquier ajuste que haya tenido que hacer.

`material` nunca se mueve por deltas: es un **consumo**, no un precio. Moverlo seria inventar
un dato de produccion.

---

## Sensibilidad

El motor toma cada variable por turno, la sube 10 %, corre las nueve formulas, la baja 10 %,
y repite. El **efecto** es el cambio de utilidad. Al final ordena de mayor a menor, usando el
valor absoluto, porque aqui importa cuanto pesa cada variable y no si conviene subirla o
bajarla; la columna "Favorable" dice cual de las dos.

Ese orden es un numero, no una opinion, y es lo que la IA tiene que respetar.

Caso base:

| # | Variable | Efecto de subir la variable | Cambio | Favorable |
|---|----------|------------------------------|--------|-----------|
| 1 | Precio Venta | Q 23,200.00 | 31.83 % | aumentar |
| 2 | Volumen | Q 11,489.60 | 15.76 % | aumentar |
| 3 | Precio Material | Q -5,990.40 | -8.22 % | reducir |
| 4 | Costos Fijos | Q -4,200.00 | -5.76 % | reducir |
| 5 | Horas MOD | Q -3,920.00 | -5.38 % | reducir |
| 6 | Tarifa MOD | Q -3,920.00 | -5.38 % | reducir |
| 7 | CIF Variable | Q -1,800.00 | -2.47 % | reducir |
| 8 | Desperdicio | Q -230.40 | -0.32 % | reducir |

**El hallazgo de negocio:** el precio de venta domina el resultado con 31.83 %. Antes de
pelearse por los chichos de consumo, hay que entender cuanto espacio de precio existe. Ese
es el tipo de conclusion que el prototipo existe para producir.

---

## Pruebas

Las cuatro obligatorias del documento, mas seis de control:

| # | Prueba | Que protege |
|---|--------|-------------|
| 1 | Escenario base | Coincide con la hoja de control |
| 2 | Precio del material +12 % | Se propaga por toda la cadena |
| 3 | Desperdicio de 4 % a 9 % | Se refleja en el margen |
| 4 | Margen no positivo | El punto de equilibrio no existe y se advierte |
| 5 | Validaciones | Ninguna entrada invalida pasa en silencio |
| 6 | Escenarios del mismo motor | Los cuatro salen de la misma funcion |
| 7 | Sensibilidad ordenada | El orden sale del impacto medido |
| 8 | La IA no calcula | No introduce numeros nuevos |
| 9 | Hoja de control completa | Los cuatro casos, indicador por indicador |
| 10 | Sin IA sigue funcionando | Responde igual sin clave de Gemini |

La PRUEBA 4 es la importante para el criterio: con margen negativo, el motor **no inventa** un
punto de equilibrio. Devuelve `punto_equilibrio: null`, marca `punto_equilibrio_valido: false`
y advierte. Un valor negativo ahi no significaria nada.

### Ejecutar

```bash
# Suite del motor, sin necesidad de Colab
python colab/pruebas.py

# Una sola prueba, por nombre de funcion
python colab/pruebas.py test_margen_no_positivo

# Suite en pytest, incluye el contrato entre la pagina y el motor
pip install pytest
pytest tests -v

# Auditoria: claves de API, nombres, formulas, frontend y notebook
python tools/verificar_proyecto.py

# Solo el frontend, o solo el notebook
python tools/verificar_frontend.py
python tools/build_notebook.py
```

`colab/pruebas.py` devuelve el codigo de salida 0 si todas las pruebas pasan y 1 si alguna
falla, para poder usarlo en integracion continua.

Estado actual:

```
RESULTADO: 10 de 10 pruebas aprobadas.
Comprobaciones totales: 224   Fallidas: 0
59 passed
RESULTADO: TODO CORRECTO
RESULTADO: FRONTEND CORRECTO
RESULTADO: NOTEBOOK CORRECTO
```

---

## Seguridad de la clave

- La clave vive en un **secreto de Colab** o en la variable de entorno `GEMINI_API_KEY`.
- **Nunca** se escribe en `config.py`, en el notebook, ni en la pagina web.
- El navegador **nunca** recibe la clave: solo recibe el texto ya redactado.
- `tools/verificar_proyecto.py` busca claves pegadas en cualquier archivo del repositorio y
  falla si encuentra alguna.

---

## Limitaciones

Cosas que este prototipo **no** resuelve, dichas de frente:

- El analisis es de corto plazo: no modela el valor del dinero en el tiempo.
- La sensibilidad es de **un solo factor**. Cambiar dos variables a la vez puede tener efectos
  que aqui no se ven.
- Los deltas optimista y adverso son **supuestos declarados, no probabilidades**. No hay que
  leerlos como un pronostico.
- La calidad del texto de la IA depende de la version del modelo que este disponible.
- El punto de equilibrio es un modelo de un solo periodo: no representa una curva de demanda
  ni un punto de indiferencia.
- Un solo producto, una sola operacion, sin mezcla.

---

## Reglas para trabajar con IA

- La IA tiene una funcion concreta: interpretar sensibilidad y proponer alternativas medibles.
- Los calculos centrales son reproducibles con formulas, codigo y la hoja de control.
- Todo dato generado por IA se valida antes de utilizarse, y la validacion se muestra.
- No se usan datos personales, confidenciales ni informacion real de ninguna empresa.
- Un resultado generado por IA **no se acepta** si el equipo no puede explicar y validar como
  se obtuvo.

La plantilla para llenar durante la demo: [`docs/bitacora_ia.md`](docs/bitacora_ia.md).
