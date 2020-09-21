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

    ## [1] -1.347102 -1.149417 -8.618555 -3.077666  3.023889

# Pulimiento de medianas

Unir las coordenadas con la columna de simulación

``` r
data <- as.data.frame(cbind(coordenadas@coords,
                            Simula = t(simu)))
names(data) <- c("X", "Y", "Var")
print(head(data))
```

|         X |   Y |        Var |
| --------: | --: | ---------: |
| 0.0000000 | 0.0 | \-1.347102 |
| 0.3333333 | 0.0 | \-1.149417 |
| 0.6666667 | 0.0 | \-8.618555 |
| 1.0000000 | 0.0 | \-3.077666 |
| 0.0000000 | 0.2 |   3.023889 |
| 0.3333333 | 0.2 |   1.682274 |

Reshape para matriz, esto lo transforma la Data en forma de tabla

``` r
tabla <- reshape2::dcast(data,
                         X ~ Y,
                         value.var = "Var")
rownames(tabla) <- tabla[, 1]
tabla <- tabla[, c(-1)]
print(tabla)
```

|                   |          0 |        0.2 |         0.4 |         0.6 |       0.8 |           1 |
| :---------------- | ---------: | ---------: | ----------: | ----------: | --------: | ----------: |
| 0                 | \-1.347102 |   3.023889 |   0.8124928 | \-0.0532752 | 0.3013796 | \-1.6932062 |
| 0.333333333333333 | \-1.149417 |   1.682274 |   1.0359056 |   2.8293628 | 4.8222992 |   3.0159529 |
| 0.666666666666667 | \-8.618555 | \-4.554644 | \-2.7003418 |   4.4378744 | 3.7340327 |   2.1978153 |
| 1                 | \-3.077666 | \-5.570104 | \-6.6392507 |   0.6051340 | 0.3713920 | \-0.0435759 |

Pulimiento de mediandas de la tabla

``` r
med <- medpolish(tabla)
```

    ## 1: 42.34408
    ## Final: 42.34408

``` r
geo_data <- reshape2::melt(med$residuals)
print(med)
```

    ## 
    ## Median Polish Results (Dataset: "tabla")
    ## 
    ## Overall: -0.3768296
    ## 
    ## Row Effects:
    ##                 0 0.333333333333333 0.666666666666667                 1 
    ##        -0.1255664         2.2540844         0.1255664        -1.5011337 
    ## 
    ## Column Effects:
    ##         0       0.2       0.4       0.6       0.8         1 
    ## -2.113187 -1.943560 -1.645214  1.717603  2.597200  1.486543 
    ## 
    ## Residuals:
    ##                          0     0.2      0.4      0.6      0.8        1
    ## 0                  1.26848  5.4698  2.96010 -1.26848 -1.79342 -2.67735
    ## 0.333333333333333 -0.91348  1.7486  0.80386 -0.76549  0.34784 -0.34784
    ## 0.666666666666667 -6.25410 -2.3598 -0.80386  2.97153  1.38810  0.96254
    ## 1                  0.91348 -1.7486 -3.11607  0.76549 -0.34784  0.34784

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
| 0.0000000 | 0.0 | \-1.3471017 | \-0.1255664 |    \-2.113187 |   1.2684818 |   \-0.3768296 |
| 0.3333333 | 0.0 | \-1.1494170 |   2.2540844 |    \-2.113187 | \-0.9134843 |   \-0.3768296 |
| 0.6666667 | 0.0 | \-8.6185550 |   0.1255664 |    \-2.113187 | \-6.2541043 |   \-0.3768296 |
| 1.0000000 | 0.0 | \-3.0776664 | \-1.5011337 |    \-2.113187 |   0.9134843 |   \-0.3768296 |
| 0.0000000 | 0.2 |   3.0238892 | \-0.1255664 |    \-1.943560 |   5.4698455 |   \-0.3768296 |
| 0.3333333 | 0.2 |   1.6822745 |   2.2540844 |    \-1.943560 |   1.7485801 |   \-0.3768296 |
| 0.6666667 | 0.2 | \-4.5546438 |   0.1255664 |    \-1.943560 | \-2.3598202 |   \-0.3768296 |
| 1.0000000 | 0.2 | \-5.5701038 | \-1.5011337 |    \-1.943560 | \-1.7485801 |   \-0.3768296 |
| 0.0000000 | 0.4 |   0.8124928 | \-0.1255664 |    \-1.645214 |   2.9601027 |   \-0.3768296 |
| 0.3333333 | 0.4 |   1.0359056 |   2.2540844 |    \-1.645214 |   0.8038647 |   \-0.3768296 |
| 0.6666667 | 0.4 | \-2.7003418 |   0.1255664 |    \-1.645214 | \-0.8038647 |   \-0.3768296 |
| 1.0000000 | 0.4 | \-6.6392507 | \-1.5011337 |    \-1.645214 | \-3.1160735 |   \-0.3768296 |
| 0.0000000 | 0.6 | \-0.0532752 | \-0.1255664 |      1.717603 | \-1.2684818 |   \-0.3768296 |
| 0.3333333 | 0.6 |   2.8293628 |   2.2540844 |      1.717603 | \-0.7654946 |   \-0.3768296 |
| 0.6666667 | 0.6 |   4.4378744 |   0.1255664 |      1.717603 |   2.9715349 |   \-0.3768296 |
| 1.0000000 | 0.6 |   0.6051340 | \-1.5011337 |      1.717603 |   0.7654946 |   \-0.3768296 |
| 0.0000000 | 0.8 |   0.3013796 | \-0.1255664 |      2.597200 | \-1.7934242 |   \-0.3768296 |
| 0.3333333 | 0.8 |   4.8222992 |   2.2540844 |      2.597200 |   0.3478446 |   \-0.3768296 |
| 0.6666667 | 0.8 |   3.7340327 |   0.1255664 |      2.597200 |   1.3880961 |   \-0.3768296 |
| 1.0000000 | 0.8 |   0.3713920 | \-1.5011337 |      2.597200 | \-0.3478446 |   \-0.3768296 |
| 0.0000000 | 1.0 | \-1.6932062 | \-0.1255664 |      1.486543 | \-2.6773529 |   \-0.3768296 |
| 0.3333333 | 1.0 |   3.0159529 |   2.2540844 |      1.486543 | \-0.3478446 |   \-0.3768296 |
| 0.6666667 | 1.0 |   2.1978153 |   0.1255664 |      1.486543 |   0.9625358 |   \-0.3768296 |
| 1.0000000 | 1.0 | \-0.0435759 | \-1.5011337 |      1.486543 |   0.3478446 |   \-0.3768296 |

Validación de la descomposición

``` r
valida <- cbind(geo_data$Var,
                geo_data[["Efecto fila"]] +
                geo_data[["Efecto columa"]] +
                geo_data[["Residual"]] +
                geo_data[["Efecto Global"]])
valida <- as.data.frame(valida)
names(valida) <- c("datos", "suma")
print(valida)
```

|       datos |        suma |
| ----------: | ----------: |
| \-1.3471017 | \-1.3471017 |
| \-1.1494170 | \-1.1494170 |
| \-8.6185550 | \-8.6185550 |
| \-3.0776664 | \-3.0776664 |
|   3.0238892 |   3.0238892 |
|   1.6822745 |   1.6822745 |
| \-4.5546438 | \-4.5546438 |
| \-5.5701038 | \-5.5701038 |
|   0.8124928 |   0.8124928 |
|   1.0359056 |   1.0359056 |
| \-2.7003418 | \-2.7003418 |
| \-6.6392507 | \-6.6392507 |
| \-0.0532752 | \-0.0532752 |
|   2.8293628 |   2.8293628 |
|   4.4378744 |   4.4378744 |
|   0.6051340 |   0.6051340 |
|   0.3013796 |   0.3013796 |
|   4.8222992 |   4.8222992 |
|   3.7340327 |   3.7340327 |
|   0.3713920 |   0.3713920 |
| \-1.6932062 | \-1.6932062 |
|   3.0159529 |   3.0159529 |
|   2.1978153 |   2.1978153 |
| \-0.0435759 | \-0.0435759 |
