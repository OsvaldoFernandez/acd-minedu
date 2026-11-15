# Reflexión · Clase 6 y el desafío del millón

Notas de lo que aprendí recorriendo la clase 6 (abandono escolar en Chile, datos MINEDUC), con los números de mi propia corrida del pipeline (`abandono_chile`, Python, 2026-11-13).

## 1. El problema y los datos

- **La unidad es alumno × año ("la foto").** El recorte es el 10 % de los *alumnos*, no de las filas, para poder reconstruir la historia de cada uno.
- **La clase la construimos nosotros:** `abandono_t1 = 1` si el alumno no aparece en *ninguna* escuela en t+1 y no egresó. Viene precalculada desde los datos completos del ministerio.
  - Letra chica: hay ~22.500 filas que no aparecen en t+1 en el recorte y aun así tienen `abandono_t1 = 0`. Mi hipótesis es que pasaron a educación de adultos, que el recorte no incluye.
- **La etiqueta llega un año tarde.** La foto t se conoce al cerrar t, pero su respuesta recién en t+1.
- **`sit_fin`** (P/R/Y) describe el año t, no el abandono. Retirado ≠ abandono: de los retirados, el 41 % abandona, el 27 % aparece en otra escuela, el 9 % vuelve a la misma y el 23 % no está en el recorte sin ser abandono.
- **Los retirados tienen promedio y asistencia en 0, y es "sin dato", no un cero real.**
- **Con 1,5 % de positivos, la exactitud no sirve.** El AUC mide **orden**, no **probabilidades**. Ordenar bien y estar calibrado son dos propiedades distintas.

## 2. Feature engineering

- El FE agrega **trayectoria**: año anterior, cambios, repitencias, sobreedad y comparación con la escuela.
- **Regla de fuga:** solo vale lo que se conoce al momento de predecir, es decir, años anteriores del mismo alumno o datos del mismo año t. `esc_asist` del mismo año está bien; `esc_ab` del mismo año sería fuga (por eso existe `esc_ab_lag`).
- **NaN y 0 no son lo mismo.** NaN es "no hay fila el año anterior"; 0 es "hay fila, pero es un retirado".
- **El NaN también informa.** Un lag vacío con `anios_previos > 0` delata un hueco, o sea, que antes dejó la escuela. La logística lo borra al imputar la mediana; los árboles lo aprovechan.
- **`anios_previos` deriva con el calendario:** en train vale 1–2 y en test 4. Los árboles no extrapolan.
- Ideas que no están en el FE de la clase: abandonos previos, NaN en lugar de 0 para retirados, y variables relativas al año.

## 3. Training strategy

- Todo es un **backtest** con saltos de un año: train aprende, validate elige, test califica.
- Elegir con el test infla el número que se reporta (sesgo de selección).
- **train_final** suma validate una vez tomadas las decisiones. Para el desafío: el diseño se hace con un backtest que evalúa en 2020 y el modelo final entrena con 2015–2020.
- Una partición temporal **muestra** los problemas de producción; una al azar los esconde.

## 4. Resultados (AUC en test 2018)

| Escalón | AUC |
|---|---|
| Regla: asistencia | 0,759 |
| Logística / + FE | 0,842 / 0,898 |
| LightGBM / + FE | 0,870 / 0,916 |
| + HT / + train_final | 0,921 / **0,924** |

- **Primero las variables, después el modelo:** la logística con FE le gana a LightGBM sin FE. La búsqueda de hiperparámetros suma 0,005.
- **Importancia no es efecto.** Que el modelo use la asistencia no dice que subirla baje el abandono.

## 5. Los años raros

| | 2018 | 2019 (estallido) | 2020 (pandemia) |
|---|---|---|---|
| Tasa de abandono | 1,57 % | 1,96 % | 1,22 % |
| AUC modelo quieto (8) | 0,924 | 0,906 | 0,843 |
| AUC reentrenado (9) | 0,924 | 0,909 | 0,863 |
| AUC de la asistencia sola | 0,759 | 0,758 | 0,678 |

- Mi apuesta (2018 > 2019 ≈ 2019 > 9 en 2020 > 8 en 2020) se cumplió.
- El estallido afectó solo dos meses y casi no cambió la señal. La pandemia sí: en 2020 el 44 % de los que abandonaron tenía asistencia ≥ 90 %.
- **Calibración:** en 2018 está bien. En 2019 el modelo fue optimista (predijo 1,62 %, fue 1,96 %). En 2020 fue pesimista, pero **no de forma pareja**: en el grupo de 10–20 % predijo 14,6 % y fue 7,3 %. Reentrenar mejora el orden, no la calibración.
- Por qué pasó en 2020:
  - Los de asistencia baja abandonaron la mitad de lo normal: la asistencia remota significaba otra cosa.
  - Reprobados y retirados fueron menos, pero abandonaron más: solo quedaron los casos graves.
  - La etiqueta mide matrícula, no compromiso.

## 6. Lo que esto implica para repartir el millón

- Abandonos evitados esperados = Σ p · efecto(m). Si todas las p se multiplican por el mismo factor, **el reparto óptimo no cambia**. Lo que importa son las **proporciones** entre alumnos.
- La distorsión de 2020 no es proporcional: el modelo cree que los de arriba son 21× más riesgosos que los del medio, y en la realidad son 8,5×. Sin corregir, **concentraría de más arriba**.
- Recalibrar solo el nivel no cambia nada; hay que corregir la **forma** (por ejemplo, con isotónica).
- **No uso la tasa real de 2022:** es un dato externo del MINEDUC y además es información del futuro. En cambio armo **escenarios** (2022 como 2019, como 2020 o normal) y mido cuánto cambia el reparto.
- La hipótesis del ministerio ("la plata solo rinde en quien iba a abandonar") es un supuesto causal: predecir no es intervenir.

## Preguntas abiertas para el profe

1. ¿`abandono_t1` cuenta la educación de adultos como "estar en una escuela"?
2. Cuando un alumno tiene varios registros en el año (`n_reg > 1`), ¿qué fila queda en el recorte?
