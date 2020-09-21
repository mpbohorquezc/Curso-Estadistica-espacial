Pulimiento de medianas
================

# Simulación de un campo aleatorio.

## Cargar librerias

Lista de librerías con link a la documentación.

  - [gstat](https://cran.r-project.org/web/packages/gstat/gstat.pdf)
  - [sp](https://cran.r-project.org/web/packages/sp/sp.pdf)

Lista de librerías con link a la documentación.

## Grilla de las ubicaciones espaciales.

``` r
n_x <- 4
n_y <- 6
x <- seq(0, 1, len = n_x)
y <- seq(0, 1, len = n_y)
coordenadas <- as.data.frame(expand.grid(x, y))
names(coordenadas) <- c("X", "Y")
```

Encabezado coordenadas

|         X |   Y |
| --------: | --: |
| 0.0000000 | 0.0 |
| 0.3333333 | 0.0 |
| 0.6666667 | 0.0 |
| 1.0000000 | 0.0 |
| 0.0000000 | 0.2 |
| 0.3333333 | 0.2 |

## Definición de objeto VGM

Esto define un objeto vgm que se usa a lo largo del paquete gstats, los
modelos que se definen pueden ser más complejos y combinados. Explorar
la
    documentación.

  - [vgm](https://cran.r-project.org/web/packages/gstat/gstat.pdf#page=73)

<!-- end list -->

``` r
vario <- vgm(10, # Punto de silla
             "Exp", # Modelo, ver documentación
             0.5)  # Rango
print(vario)
```

    ##   model psill range
    ## 1   Exp    10   0.5

## Matriz de varianza dadas coordenadas.

  - [vgmArea](https://cran.r-project.org/web/packages/gstat/gstat.pdf#page=78)
  - [coordinates](https://cran.r-project.org/web/packages/sp/sp.pdf#page=16)

<!-- end list -->

``` r
coordinates(coordenadas) <- ~X + Y
class(coordenadas) # Cambio de objedto dataframe a sp
```

    ## [1] "SpatialPoints"
    ## attr(,"package")
    ## [1] "sp"

``` r
cov_mat <- vgmArea(coordenadas, # Matriz de ubiaciones SP
        vgm = vario) # VGM object

print(dim(cov_mat))
```

    ## [1] 24 24

## Simulación.

Simulación dada la media y la matriz de varianza

``` r
mu  <- rep(0, n_x * n_y) # Media del proceso
simu <- rmvnorm(1,
                mean = mu,
                sigma = cov_mat)
print(simu[1:5])
```

    ## [1]  4.830103  3.034525  1.138179 -1.873857  2.060435

# Pulimiento de medianas

Unir las coordenadas con la columna de simulación

``` r
data <- as.data.frame(cbind(coordenadas@coords,
                            Simula = t(simu)))
names(data) <- c("X", "Y", "Var")
print(head(data))
```

|         X |   Y |         Var |
| --------: | --: | ----------: |
| 0.0000000 | 0.0 |   4.8301027 |
| 0.3333333 | 0.0 |   3.0345251 |
| 0.6666667 | 0.0 |   1.1381794 |
| 1.0000000 | 0.0 | \-1.8738568 |
| 0.0000000 | 0.2 |   2.0604349 |
| 0.3333333 | 0.2 |   0.2945418 |

Reshape para matriz, esto lo transforma la Data en forma de tabla

``` r
tabla <- reshape2::dcast(data,
                         X ~ Y,
                         value.var = "Var")
rownames(tabla) <- tabla[, 1]
tabla <- tabla[, c(-1)]
print(tabla)
```

|                   |          0 |         0.2 |         0.4 |       0.6 |      0.8 |          1 |
| :---------------- | ---------: | ----------: | ----------: | --------: | -------: | ---------: |
| 0                 |   4.830103 |   2.0604349 | \-0.9225624 | 0.5634194 | 1.092045 | \-1.300585 |
| 0.333333333333333 |   3.034525 |   0.2945418 |   4.3566390 | 4.2564992 | 4.567231 |   2.020777 |
| 0.666666666666667 |   1.138179 | \-0.3603120 |   0.8336506 | 2.5359436 | 4.159528 |   3.367247 |
| 1                 | \-1.873857 |   1.2181301 |   2.7622138 | 5.5076557 | 3.472742 |   1.070395 |

Pulimiento de mediandas de la tabla

``` r
med <- medpolish(tabla)
```

    ## 1: 29.05233
    ## 2: 25.79825
    ## 3: 25.51355
    ## Final: 25.51355

``` r
geo_data <- reshape2::melt(med$residuals)
print(med)
```

    ## 
    ## Median Polish Results (Dataset: "tabla")
    ## 
    ## Overall: 1.745649
    ## 
    ## Row Effects:
    ##                 0 0.333333333333333 0.666666666666667                 1 
    ##        -2.1776530         1.2569077        -0.5097646         0.5097646 
    ## 
    ## Column Effects:
    ##           0         0.2         0.4         0.6         0.8           1 
    ## -0.03286843 -1.31674008  0.05228308  1.27700074  1.54436155 -0.92518049 
    ## 
    ## Residuals:
    ##                           0      0.2      0.4       0.6       0.8         1
    ## 0                  5.294975  3.80918 -0.54284 -0.281577 -0.020312  0.056599
    ## 0.333333333333333  0.064837 -1.39127  1.30180 -0.023058  0.020312 -0.056599
    ## 0.666666666666667 -0.064837 -0.27946 -0.45452  0.023058  1.379282  3.056543
    ## 1                 -4.096402  0.27946  0.45452  1.975241 -0.327033 -0.259839

Reshape de los datos, con efecto de la fila y la columna

``` r
tabla_residuales <- as.data.frame(med$residuals)
names(tabla_residuales) <- med$col
rownames(tabla_residuales) <- med$row
geo_data <- reshape2::melt(as.matrix(tabla_residuales))

geo_data <- cbind(data,
                  geo_data,
                  med$overall)
names(geo_data) <- c("X",
                     "Y",
                     "Var",
                     "Efecto fila",
                     "Efecto columa",
                     "Residual",
                     "Efecto Global")
print(geo_data)
```

|         X |   Y |         Var | Efecto fila | Efecto columa |    Residual | Efecto Global |
| --------: | --: | ----------: | ----------: | ------------: | ----------: | ------------: |
| 0.0000000 | 0.0 |   4.8301027 | \-2.1776530 |   \-0.0328684 |   5.2949750 |      1.745649 |
| 0.3333333 | 0.0 |   3.0345251 |   1.2569077 |   \-0.0328684 |   0.0648367 |      1.745649 |
| 0.6666667 | 0.0 |   1.1381794 | \-0.5097646 |   \-0.0328684 | \-0.0648367 |      1.745649 |
| 1.0000000 | 0.0 | \-1.8738568 |   0.5097646 |   \-0.0328684 | \-4.0964022 |      1.745649 |
| 0.0000000 | 0.2 |   2.0604349 | \-2.1776530 |   \-1.3167401 |   3.8091788 |      1.745649 |
| 0.3333333 | 0.2 |   0.2945418 |   1.2569077 |   \-1.3167401 | \-1.3912749 |      1.745649 |
| 0.6666667 | 0.2 | \-0.3603120 | \-0.5097646 |   \-1.3167401 | \-0.2794564 |      1.745649 |
| 1.0000000 | 0.2 |   1.2181301 |   0.5097646 |   \-1.3167401 |   0.2794564 |      1.745649 |
| 0.0000000 | 0.4 | \-0.9225624 | \-2.1776530 |     0.0522831 | \-0.5428416 |      1.745649 |
| 0.3333333 | 0.4 |   4.3566390 |   1.2569077 |     0.0522831 |   1.3017991 |      1.745649 |
| 0.6666667 | 0.4 |   0.8336506 | \-0.5097646 |     0.0522831 | \-0.4545169 |      1.745649 |
| 1.0000000 | 0.4 |   2.7622138 |   0.5097646 |     0.0522831 |   0.4545169 |      1.745649 |
| 0.0000000 | 0.6 |   0.5634194 | \-2.1776530 |     1.2770007 | \-0.2815775 |      1.745649 |
| 0.3333333 | 0.6 |   4.2564992 |   1.2569077 |     1.2770007 | \-0.0230583 |      1.745649 |
| 0.6666667 | 0.6 |   2.5359436 | \-0.5097646 |     1.2770007 |   0.0230583 |      1.745649 |
| 1.0000000 | 0.6 |   5.5076557 |   0.5097646 |     1.2770007 |   1.9752412 |      1.745649 |
| 0.0000000 | 0.8 |   1.0920452 | \-2.1776530 |     1.5443615 | \-0.0203125 |      1.745649 |
| 0.3333333 | 0.8 |   4.5672308 |   1.2569077 |     1.5443615 |   0.0203125 |      1.745649 |
| 0.6666667 | 0.8 |   4.1595280 | \-0.5097646 |     1.5443615 |   1.3792820 |      1.745649 |
| 1.0000000 | 0.8 |   3.4727422 |   0.5097646 |     1.5443615 | \-0.3270331 |      1.745649 |
| 0.0000000 | 1.0 | \-1.3005853 | \-2.1776530 |   \-0.9251805 |   0.0565990 |      1.745649 |
| 0.3333333 | 1.0 |   2.0207773 |   1.2569077 |   \-0.9251805 | \-0.0565990 |      1.745649 |
| 0.6666667 | 1.0 |   3.3672472 | \-0.5097646 |   \-0.9251805 |   3.0565432 |      1.745649 |
| 1.0000000 | 1.0 |   1.0703947 |   0.5097646 |   \-0.9251805 | \-0.2598386 |      1.745649 |

Validación de la descomposición

``` r
valida <- cbind(geo_data$Var,
                geo_data[["Efecto fila"]] +
                geo_data[["Efecto columa"]] +
                geo_data[["Residual"]] +
                geo_data[["Efecto Global"]])
names(valida) <- c("datos", "suma")
print(valida)
```

    ## Warning in kable_pipe(x = structure(c("4.8301027", "3.0345251", "1.1381794", : The table should have a header (column
    ## names)

|             |             |
| ----------: | ----------: |
|   4.8301027 |   4.8301027 |
|   3.0345251 |   3.0345251 |
|   1.1381794 |   1.1381794 |
| \-1.8738568 | \-1.8738568 |
|   2.0604349 |   2.0604349 |
|   0.2945418 |   0.2945418 |
| \-0.3603120 | \-0.3603120 |
|   1.2181301 |   1.2181301 |
| \-0.9225624 | \-0.9225624 |
|   4.3566390 |   4.3566390 |
|   0.8336506 |   0.8336506 |
|   2.7622138 |   2.7622138 |
|   0.5634194 |   0.5634194 |
|   4.2564992 |   4.2564992 |
|   2.5359436 |   2.5359436 |
|   5.5076557 |   5.5076557 |
|   1.0920452 |   1.0920452 |
|   4.5672308 |   4.5672308 |
|   4.1595280 |   4.1595280 |
|   3.4727422 |   3.4727422 |
| \-1.3005853 | \-1.3005853 |
|   2.0207773 |   2.0207773 |
|   3.3672472 |   3.3672472 |
|   1.0703947 |   1.0703947 |
