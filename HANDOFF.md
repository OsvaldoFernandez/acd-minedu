# Handoff · desafío del millón (ACD clase 6)

Última sesión: 2026-11-16. Entrega: **2026-11-21**.

## Dónde quedamos

El notebook `desafio.ipynb` corre completo hasta la sección 9 (unos 6–7 minutos con `jupyter nbconvert --to notebook --execute --inplace desafio.ipynb`). Falta **una sola decisión** y la entrega.

**🟨 10 resuelto (2026-11-16):** minimax → reparto "sin E · normal"; `entrega.csv` generado y validado; README final escrito. Anexo `anexos/reparto_como_2020.csv` agregado (avisarle al profe por escrito que no es una segunda entrega). Pendiente: Osvaldo relee README y notebook; después, presentación para exponer en clase **10 minutos máximo**, con el estilo del deck del profe (índice de 14 láminas propuesto, hay que achicarlo). No citar la tasa real 2021 (1,47 %) que aparece en la lámina 15 del profe.

**Git:** commits locales sin pushear desde `c357f1e`. El push lo hace Osvaldo.

## Decisiones tomadas (y por qué)

| Decisión | Por qué |
|---|---|
| Criterio: maximizar abandonos evitados, sin restricciones | Decisión de negocio inicial |
| Sin datos externos (ni tasas reales de 2022) | Regla de la consigna y fuga del futuro; en su lugar, escenarios |
| Features: las de la clase + **C** (percentiles en el año) | C suma en pandemia (+0,009 AUC) y aguanta cambios de nivel |
| A, B, D afuera | A y B ≈ 0. D resta en pandemia, y en 2021 hay escuelas anómalas (deltas que nunca se vieron) que los árboles no saben tratar |
| `anios_previos` queda | Los que entran al panel fuera de 1° básico abandonan 24,6 % (vs. 1,2 %) |
| E (lag2) afuera | No cambia el reparto (87 % de solapamiento, arrepentimiento < 3 %) |
| Backtest de pandemia pesa más | 2021 se parece más a 2020 que a 2018 (validación adversarial: 0,686 vs. 0,840) |
| Escenario "como 2019" descartado | En la zona de la plata (riesgo ≥ 11 %), 2019 se comportó como un año normal. Su sorpresa estuvo en chicos de riesgo bajo |
| Dos mundos: normal y "como 2020" | La distorsión de 2020 cae donde va la plata y no es pareja (isotónica aprendida en el backtest 2015–2019 → 2020) |

## Números clave

- Modelo final (2015–2020 → foto 2021): tasa media predicha 1,23 %.
- **Reparto normal:** 6.419 alumnos, riesgo ≥ 11 %, 688 abandonos evitados esperados.
- **Reparto como 2020:** 8.815 alumnos, riesgo ≥ 6,6 %. Más plata arriba y abajo, menos al grupo 20–40 %.
- **Matriz 2 × 2:** el reparto normal pierde 5,6 (1,3 %) si el mundo es como 2020; el reparto 2020 pierde 13,0 (1,9 %) si el mundo es normal.
- Minimax y promedio eligen lo mismo: **"sin E · normal"**.

## Lo que falta

1. Osvaldo confirma la regla (🟨 10).
2. Sección 10: escribir el CSV `id_alumno,monto` con validaciones (solo foto 2021, montos ≥ 0, suma ≤ US$ 1.000.000, egresados con 0).
3. README final con todas las decisiones, incluidas:
   - El abandono "invisible" de 2019: chicos de riesgo bajo, que ningún reparto alcanza.
   - El óptimo plano: repartos con chicos bastante distintos (73 % de solapamiento) rinden casi lo mismo.
   - Las aproximaciones: el mapeo isotónico viene de un modelo de backtest y se aplica al modelo final; el "efecto" de la plata es un supuesto causal del ministerio.
4. Actualizar `reflexion.md` con lo aprendido en el desafío (escenarios, arrepentimiento, factor parejo).

## Pendientes de estudio (no bloquean la entrega)

- Bloque 4: preguntas sin responder (rendimientos decrecientes, el FE que falla en 2020, la importancia repartida).
- Bloques 5 (AutoGluon, láminas 17–18) y 6 (láminas 19–20).
- Pregunta abierta: `asistencia_delta1` en 2021 mira contra 2020 (asistencia inflada). ¿Cómo queda y lo vio el modelo al entrenar?
- Preguntas para el profe: ¿`abandono_t1` cuenta adultos como escuela? ¿Qué fila queda si `n_reg > 1`?

## Cómo retomar

Leer este archivo, abrir `desafio.ipynb` en la sección 9 (las salidas ya están guardadas) y arrancar por la 🟨 10.

## Presentación (2026-11-19)

Hecha: 8 láminas, unos 5 minutos, con notas del orador y tiempos. Artifact privado: https://claude.ai/artifact/K2ErUmHgpn7YjSuDkosogL (se comparte desde Share). Copia en el repo: `presentacion/presentacion.html`, con las fuentes en `presentacion/fuente/`. Integrantes: Carlos Canova, Javier Cescon, Osvaldo Fernandez. Pendiente: que el grupo la revise.
