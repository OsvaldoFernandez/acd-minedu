# acd-minedu · Un millón de dólares contra el abandono escolar

Desafío de la clase 6 de Aplicaciones de Ciencia de Datos (UNO, 2026): repartir US$ 1.000.000 entre los alumnos de la foto 2021 para evitar la mayor cantidad posible de abandonos en 2022.

**Estado:** en progreso. Ya está el feature engineering; faltan el modelo final, los escenarios de calibración y el reparto.

## Contenido

| Archivo | Qué es |
|---|---|
| `desafio.ipynb` | El ejercicio: FE, backtests, modelo, escenarios y reparto |
| `reflexion.md` | Resumen de lo aprendido en la clase 6 |

## Cómo correrlo

Los datos no están en el repo (licencia CC BY-NC y tamaño). El notebook los lee desde `../predicciones/clase6/desafio_ministerio/`.

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace desafio.ipynb
```

## Decisiones tomadas

- **Criterio:** maximizar los abandonos evitados, sin restricciones adicionales de reparto.
- **Sin datos externos:** el escenario de 2022 se elige comparando escenarios construidos con los datos del curso.

Fuente: Centro de Estudios MINEDUC, datos abiertos (CC BY-NC 2.0).
