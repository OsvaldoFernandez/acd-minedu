# acd-minedu · Un millón de dólares contra el abandono escolar

Desafío de la clase 6 de Aplicaciones de Ciencia de Datos (UNO, 2026): repartir US$ 1.000.000 entre los alumnos de la foto 2021 para evitar la mayor cantidad posible de abandonos en 2022.

**Entrega:** `entrega.csv`. Le da plata a 6.419 alumnos, entre US$ 0,03 y US$ 431,87, y suma US$ 999.967,94.

## Contenido

| Archivo | Qué es |
|---|---|
| `entrega.csv` | La entrega: `id_alumno,monto` |
| `desafio.ipynb` | El código que arma la entrega: FE, backtests, modelos, escenarios y reparto |
| `reflexion.md` | Resumen de lo aprendido en la clase 6 |

## Cómo correrlo

Los datos no están en el repo (licencia CC BY-NC y tamaño). El notebook los lee desde `../predicciones/clase6/desafio_ministerio/`. Tarda unos 7 minutos.

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace desafio.ipynb
```

## Cómo decidimos a quién darle y cuánto

**1. Un modelo que ordena y estima la probabilidad de abandonar.** Es el LightGBM de la clase con sus hiperparámetros, entrenado con las fotos 2015–2020. Usa las variables de la clase más percentiles de asistencia y promedio dentro del año (ver abajo).

**2. Un reparto con rendimientos decrecientes.** Cada dólar va al alumno donde más rinde: p · 0,003 · e^(−m/200). El resultado: los que reciben plata terminan con el mismo rendimiento en el último dólar, y la plata llega solo a quienes tienen un riesgo predicho de al menos 11 %. Cuanto más riesgo, más plata.

**3. Escenarios en lugar de datos externos.** No sabemos cómo se va a comportar 2022, y no usamos tasas reales. Lo que miramos es **cómo se equivocó el modelo en los años raros con etiqueta**:
- Si todas las probabilidades se multiplican por el mismo factor, **el reparto no cambia**. Importa la *forma* de la distorsión, no el nivel.
- **2019 (estallido):** en la zona donde va la plata, el modelo acertó como en un año normal. Su sorpresa (21 % más abandonos) estuvo entre alumnos de riesgo bajo, que ningún reparto alcanza. Se descarta como escenario.
- **2020 (pandemia):** el modelo exageró justo en la zona de la plata, y no de forma pareja (el grupo 20–40 % a menos de la mitad). Se construye el mundo "como 2020" con una corrección isotónica aprendida en el backtest 2015–2019 → 2020.

**4. Elegir con una matriz de arrepentimiento.** Calculamos el reparto óptimo para cada mundo y lo evaluamos en el otro (abandonos evitados esperados que se pierden contra lo mejor posible):

| | Mundo normal | Mundo como 2020 | Peor caso |
|---|---|---|---|
| Reparto normal | 0 | 5,6 | **5,6 (1,3 %)** |
| Reparto como 2020 | 13,0 | 0 | 13,0 (1,9 %) |

Entregamos el **reparto normal**: es el que menos pierde en el peor caso (y también en promedio). Los dos repartos comparten solo el 73 % de los alumnos, pero rinden casi lo mismo: el óptimo es plano.

## Features: qué probamos y qué descartamos

Probamos cinco grupos nuevos sobre el FE de la clase, con dos backtests (normal 2015–17 → 2018 y pandemia 2015–19 → 2020), midiendo AUC y la captura del 5 % más riesgoso. Pesó más el backtest de pandemia, porque 2021 se parece más a 2020 que a un año normal (validación adversarial con variables del alumno: AUC 0,686 contra 2020 y 0,840 contra 2018).

| Grupo | Resultado | Por qué |
|---|---|---|
| **C** · percentiles en el año | ✅ entra | +0,009 de AUC en pandemia. Mira la posición dentro del año, así que aguanta cambios de nivel como los de 2020 y 2021 |
| **A** · retirados con NaN en vez de 0 | ❌ | Δ ≈ 0 |
| **B** · huecos y abandonos previos | ❌ | Δ ≈ 0. `anios_previos` con el lag vacío ya marca el hueco del año anterior |
| **D** · cambios de la escuela | ❌ | Resta en pandemia. En 2021 el 3,5 % de los alumnos está en escuelas cuya asistencia cayó más de 30 puntos (antes, como mucho 0,4 %): valores que el modelo nunca vio |
| **E** · asistencia y promedio de t−2 | ❌ | La ablación no podía juzgarlo: en 2021, `lag1` es 2020 (asistencia inflada) y `lag2` es 2019, una situación que ningún backtest reproduce. Entrenamos las dos variantes hasta el final y las comparamos **en el reparto**: comparten el 87 % de los alumnos y ninguna combinación pierde más de 3 %. No cambia la decisión, así que gana la versión simple |
| `anios_previos` | ✅ queda | Parecía riesgosa (deriva con el calendario), pero los que aparecen por primera vez fuera de 1° básico abandonan 24,6 %, contra 1,2 % con historia continua |

## Supuestos y límites

- **Sin datos externos:** ni tasas reales de 2022 ni otras fuentes del MINEDUC.
- **El mapeo "como 2020"** se aprende con un modelo de backtest y se aplica al modelo final. Suponemos que la distorsión se parece entre los dos.
- **El efecto de la plata** es el supuesto del ministerio. Predecir quién abandona no dice qué pasa si se interviene: eso es inferencia causal.
- **Lo esperado:** unos 688 abandonos evitados, *si* 2022 se comporta como un año normal *y* el efecto es el supuesto.

Fuente: Centro de Estudios MINEDUC, datos abiertos (CC BY-NC 2.0).
