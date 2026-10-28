# expcolab-2026-undersamp-9A
TS 9A  Estabilidad del modelo predictivo frente al Undersampling

Experimento colaborativo #09, Grupo A: **undersampling de CONTINUA**.
Laboratorio de Implementación I, Maestría en Ciencia de Datos, Universidad Austral, 2026.

Integrantes: Juan Pressacco · Alexandra Cremydas

> Estado: esperando aprobaciòn. Corridas pendientes.

## Hipótesis

El recorte de CONTINUA está en el z620 solamente por velocidad. Buscar los hiperparámetros con una fracción de CONTINUA al azar, y todas las bajas, se compara contra el control (`training_pct` = 1), que es el que ve toda la CONTINUA.

## Diseño

Base: workflow `z620`, Analista Junior, Grupo A. `param_final` se arma con `modifyList(fijos, mejores_hiperparametros)`. Con `c()` se duplican num_leaves, min_data_in_leaf y feature_fraction y LightGBM se queda con el fijo.

| Aspecto | Nivel | Qué es |
|---|---|---|
| training_pct | 1 | Control. Entra toda la CONTINUA |
| | 0,5 | Esa fracción de CONTINUA al azar, y todas las bajas |
| | 0,25 | Igual, con 0,25 de la CONTINUA |
| | 0,1 | Igual, con 0,1 de la CONTINUA |

El recorte entra solo en la búsqueda. Validación y entrenamiento final completos.

Períodos, iguales en los cuatro niveles: training 201901–202105, validación 202107, entrenamiento final 201901–202107, future 202109.

El grupo B repite el mismo experimento con futuro 202107 y ganancia local.

### Las 40 corridas

| Porcentaje | Carpetas |
|---|---|
| 1 | 9101–9110 |
| 0,5 | 9201–9210 |
| 0,25 | 9301–9310 |
| 0,1 | 9401–9410 |

Cómo se lee el código: 9 (problema) · bloque 1–4 (1 = pct 1, 2 = 0,5, 3 = 0,25, 4 = 0,1) · corrida 01–10. Por ejemplo, 9307 = pct 0,25 · séptima corrida de ese bloque.

Cuarenta corridas: cuatro porcentajes por 10 semillas. Cada integrante corre sus 5 semillas primigenias y entre los dos se arma el vector de 10. Las mismas 10 en los 4 niveles. Primeras 5 de cada bloque un integrante, últimas 5 el otro. La misma posición del bloque es la misma semilla: 9101 con 9201, 9301 y 9401.

En cada corrida se vuelve a hacer el grid search completo del z620, los 50 modelos.

### Protocolo

1. Grid search completo en cada corrida: los 50 modelos del z620.
2. Cada integrante corre sus 5 semillas primigenias. Las mismas 10 en los cuatro porcentajes.
3. Cortes de envío de 1800 a 2400 de a 100. Para cada semilla, el promedio del Public Score de esos cortes, no el mejor corte.
4. Comparación: Wilcoxon pareado sobre los 10 valores contra el control. No se compara por el promedio.
5. Si el óptimo de una perilla queda en el borde, se amplía el rango y se vuelve a correr esa grilla.

## Estructura del repositorio

```
(raíz)            z620_exp09_grupoA.ipynb: la copia del z620.
                  El original de la cátedra no se pisa.
                  Antes de cada corrida se cambian semilla_primigenia,
                  experimento y training_pct.
salidas/<código>/ salidas chicas de cada corrida: modelo.txt,
                  log del grid search y ganancia por semilla y corte
resultados/       tabla consolidada de ganancias, test de Wilcoxon y gráficos
```

Las carpetas `salidas/` y `resultados/` se agregan a medida que haya resultados.

## Archivos para Kaggle

No se suben al repositorio: son 7 por corrida. Quedan en la VM de quien corrió la corrida, en `/content/buckets/b1/exp/WF<código>/kaggle/`.

Nombre de cada archivo: `KA<código>_<envíos>.csv`, por ejemplo `KA9101_1800.csv`. En Kaggle, la descripción de cada envío dice `envios=<corte> semilla=<semilla>`.



- Slides del experimento: [link pendiente]
- Bitácora: [link pendiente]
