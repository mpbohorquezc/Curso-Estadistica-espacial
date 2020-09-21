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
modelos que se definen pueden ser más compljos y combinados. Explorar la
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

    ## [1] -4.0682601 -1.4198806 -2.8682008 -0.6751372 -3.3157826

# Pulimiento de mediandas

Unir las coordenadas con la columna de simulación

``` r
data <- as.data.frame(cbind(coordenadas@coords,
                            Simula = t(simu)))
names(data) <- c("X", "Y", "Var")
print(head(data))
```

|         X |   Y |         Var |
| --------: | --: | ----------: |
| 0.0000000 | 0.0 | \-4.0682601 |
| 0.3333333 | 0.0 | \-1.4198806 |
| 0.6666667 | 0.0 | \-2.8682008 |
| 1.0000000 | 0.0 | \-0.6751372 |
| 0.0000000 | 0.2 | \-3.3157826 |
| 0.3333333 | 0.2 | \-3.5416547 |

Reshape para matriz, esto lo transforma la Data en forma de tabla

``` r
tabla <- reshape2::dcast(data,
                         X ~ Y,
                         value.var = "Var")
rownames(tabla) <- tabla[, 1]
tabla <- tabla[, c(-1)]
print(tabla)
```

|                   |           0 |         0.2 |        0.4 |        0.6 |         0.8 |          1 |
| :---------------- | ----------: | ----------: | ---------: | ---------: | ----------: | ---------: |
| 0                 | \-4.0682601 | \-3.3157826 | \-3.964449 | \-1.865695 |   0.2984491 |   3.756817 |
| 0.333333333333333 | \-1.4198806 | \-3.5416547 | \-1.178037 |   3.457028 |   1.8517466 |   2.158261 |
| 0.666666666666667 | \-2.8682008 | \-2.5784630 |   1.156072 |   1.411423 |   1.1135553 | \-2.366930 |
| 1                 | \-0.6751372 | \-0.2091974 | \-2.262302 | \-2.058672 | \-1.7594986 | \-3.485401 |

Pulimiento de mediandas de la tabla

``` r
med <- medpolish(tabla)
```

    ## 1: 33.91125
    ## 2: 33.35282
    ## Final: 33.35282

``` r
print(med)
```

    ## 
    ## Median Polish Results (Dataset: "tabla")
    ## 
    ## Overall: -1.617855
    ## 
    ## Row Effects:
    ##                 0 0.333333333333333 0.666666666666667                 1 
    ##        -0.6030266         1.6962823         0.6030266        -0.9881111 
    ## 
    ## Column Effects:
    ##          0        0.2        0.4        0.6        0.8          1 
    ## -1.6728436 -1.3292682 -0.4564005  1.4867726  1.9508511  0.6001988 
    ## 
    ## Residuals:
    ##                          0      0.2      0.4      0.6      0.8       1
    ## 0                 -0.17454  0.23437 -1.28717 -1.13159  0.56848  5.3775
    ## 0.333333333333333  0.17454 -2.29081 -0.80006  1.89183 -0.17753  1.4796
    ## 0.666666666666667 -0.18053 -0.23437  2.62730  0.93948  0.17753 -1.9523
    ## 1                  3.60367  3.72604  0.80006 -0.93948 -1.10438 -1.4796
