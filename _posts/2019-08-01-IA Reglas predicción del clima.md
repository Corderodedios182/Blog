---
title: "IA Reglas predicción del clima"
categories:
  - Intelligence Artificielle
  - R
tags:
  - Intelligence Artificielle
  - R
date: 2019-08-01T14:37:42
---

## Reglas del sistema de predicción del clima.

En ocasiones no necesitamos de un modelo complejo.

El siguiente cuaderno es un ejemplo de un buen análisis para encontrar reglas lógicas que podemos integrar en nuestro código.

#### El interés del análisis es encontrar una regla para determinar en qué condiciones un niño puede salir a jugar.

```R
#Paqueterías a usar en el siguiente script
library(dplyr)
library(tidyr)
library(ggplot2)

options(repr.plot.width=10, repr.plot.height=8)
```

Nuestro conjunto de datos tiene cuenta con 4 variables Perspectiva, Temperatura, Humedad, Viento.

### Busquemos las condiciones en que los niños salen a jugar.


```R
#Importación de la base de datos
Clima <- read.csv("~/Documentos/2_Facultad/IA/Proyecto_1/Clima.csv")
Clima
```


<table>
<caption>A data.frame: 14 × 5</caption>
<thead>
	<tr><th scope=col>Perspectiva</th><th scope=col>Temperatura</th><th scope=col>Humedad</th><th scope=col>Viento</th><th scope=col>Jugar</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Soleado </td><td>29.0</td><td>85</td><td>No</td><td>No</td></tr>
	<tr><td>Soleado </td><td>27.0</td><td>90</td><td>Si</td><td>No</td></tr>
	<tr><td>Nublado </td><td>28.0</td><td>86</td><td>No</td><td>Si</td></tr>
	<tr><td>Lluvioso</td><td>21.0</td><td>96</td><td>No</td><td>Si</td></tr>
	<tr><td>Lluvioso</td><td>20.0</td><td>80</td><td>No</td><td>Si</td></tr>
	<tr><td>Lluvioso</td><td>18.0</td><td>70</td><td>Si</td><td>No</td></tr>
	<tr><td>Nublado </td><td>17.0</td><td>65</td><td>Si</td><td>Si</td></tr>
	<tr><td>Soleado </td><td>22.0</td><td>95</td><td>No</td><td>No</td></tr>
	<tr><td>Soleado </td><td>20.5</td><td>70</td><td>No</td><td>Si</td></tr>
	<tr><td>Lluvioso</td><td>24.0</td><td>80</td><td>No</td><td>Si</td></tr>
	<tr><td>Soleado </td><td>24.0</td><td>70</td><td>Si</td><td>Si</td></tr>
	<tr><td>Nublado </td><td>22.0</td><td>90</td><td>Si</td><td>Si</td></tr>
	<tr><td>Nublado </td><td>27.0</td><td>75</td><td>No</td><td>Si</td></tr>
	<tr><td>Lluvioso</td><td>21.6</td><td>91</td><td>Si</td><td>No</td></tr>
</tbody>
</table>



Antes de contestar preguntas particulares ¿Cuándo se considera una alta temperatura "hace calor" y cuándo se tiene una alta humedad?

## Primero veamos cómo tenemos dividos nuestros datos

¿Cuántos Si y No tengo en la variable jugar?


```R
Clima %>%
  group_by(Jugar) %>%
  summarise(Conteo = n())
```


<table>
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>Jugar</th><th scope=col>Conteo</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>No</td><td>5</td></tr>
	<tr><td>Si</td><td>9</td></tr>
</tbody>
</table>



Ahora veamos si existe un patrón para salir a jugar, tomando en cuenta sólo la variable perspectiva.


```R
Conteo = Clima %>%
  group_by(Jugar, Perspectiva) %>%
  summarise(Conteo = n())

Conteo
```


<table>
<caption>A grouped_df: 5 × 3</caption>
<thead>
	<tr><th scope=col>Jugar</th><th scope=col>Perspectiva</th><th scope=col>Conteo</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>No</td><td>Lluvioso</td><td>2</td></tr>
	<tr><td>No</td><td>Soleado </td><td>3</td></tr>
	<tr><td>Si</td><td>Lluvioso</td><td>3</td></tr>
	<tr><td>Si</td><td>Nublado </td><td>4</td></tr>
	<tr><td>Si</td><td>Soleado </td><td>2</td></tr>
</tbody>
</table>



Podemos ver que nublado es un buen clima para salir a jugar.


```R
ggplot(Clima, aes(x = Jugar, fill = Perspectiva)) + geom_bar() +
  labs(title = 'Casos en los que se juega',subtitle = 'En los días nublados siempre se juega')
```

![pyregex example]({{ '/images/reglas_clima/output_9_0.png' | absolute_url }}){: .align-center}

Comencemos a vizualizar la variable temperatura y humedad para encontrar un patrón.

Veamos estadísticas descriptivas.


```R
summary(Clima)
```


       Perspectiva  Temperatura       Humedad      Viento Jugar
     Lluvioso:5    Min.   :17.00   Min.   :65.00   No:8   No:5  
     Nublado :4    1st Qu.:20.62   1st Qu.:71.25   Si:6   Si:9  
     Soleado :5    Median :22.00   Median :82.50                
                   Mean   :22.94   Mean   :81.64                
                   3rd Qu.:26.25   3rd Qu.:90.00                
                   Max.   :29.00   Max.   :96.00                


Podemos ver que la temperatura mínima es 17 con una máxima de 29 y la humedad mínima de 65 con máxima de 96.

Visualizando la temperatura y humedad de forma individual podemos comenzar a observar que si la temperatura es mayor a 25 grados los niños no salen a jugar, de igual forma, si la humedad es mayor a 85.


```R
ggplot(Clima, aes(x = Temperatura, y = Perspectiva)) +
  geom_point(aes(color = Jugar)) +
  labs(title = "Temperatura Mínima de 17 y Máxima de 29",subtitle = "¿Qué pasará con temperaturas bajas, influye el viento?")

ggplot(Clima, aes(x = Humedad, y = Perspectiva)) +
  geom_point(aes(color = Jugar)) +
  labs(title = "Humedad Mínima de 65 y Máxima de 96",subtitle = "¿Qué pasa con humedades altas, influye el viento?")
```

![pyregex example]({{ '/images/reglas_clima/output_13_0.png' | absolute_url }}){: .align-center}

![pyregex example]({{ '/images/reglas_clima/output_13_1.png' | absolute_url }}){: .align-center}

Veamos la relación de todas las variables, temperatura, humedad y viento.

```R
ggplot(Clima, aes(x = Temperatura, y = Humedad, size = Viento)) +
  geom_point(aes(color = Perspectiva)) +
  facet_wrap(~Jugar) +
  labs(title = 'Humedad, Temperatura y Viento',subtitle = 'Los niños salen a jugar cuando está nublado')
```

    Warning message:
    “Using size for a discrete variable is not advised.”

![pyregex example]({{ '/images/reglas_clima/output_15_1.png' | absolute_url }}){: .align-center}

Podemos concluir que los niños siempre salen cuando está nublado.

Ahora sólo enfoquemonos en los días soleados y lluviosos.

1. Juegan cuando está lluvioso y no hay viento aunque la humedad sea mayor a 90

2. Juegan cuando la temperatura es menor a 27 grados y la humedad es menor que 95


```R
#Enfoquemonos en los días Soleados y Lluviosos,
ggplot(filter(Clima,Perspectiva != 'Nublado'), aes(x = Temperatura, y = Humedad, size = Viento)) +
  geom_point(aes(color = Perspectiva)) +
  facet_wrap(~Jugar) +
  labs(title = 'Soleado y Lluvioso',subtitle = '')
```

    Warning message:
    “Using size for a discrete variable is not advised.”

![pyregex example]({{ '/images/reglas_clima/output_18_1.png' | absolute_url }}){: .align-center}

¿Cuándo la humedad es alta?

No podemos determinar cuándo se considera una humedad alta, la variable viento parece tener relevancia al momento de decidir, ya que en días soleados la humedad mayor a 94 se considera alta.

¿A qué temperatura se considera como calor?

Aunque tengamos viento, si la temperatura es mayor a 27 grados no salen a jugar.

Podemos decir que a más de 27 grados tenemos temperaturas altas.

```R
#Hemos creados las reglas para salir a jugar para este conjunto de datos.
Clima[Clima$Perspectiva == 'Nublado',]
Clima[Clima$Perspectiva == 'Lluvioso' & Clima$Viento == 'No',]
Clima[Clima$Perspectiva == 'Soleado' & Clima$Temperatura < 27 & Clima$Humedad < 95,]
```

<table>
<caption>A data.frame: 4 × 5</caption>
<thead>
	<tr><th></th><th scope=col>Perspectiva</th><th scope=col>Temperatura</th><th scope=col>Humedad</th><th scope=col>Viento</th><th scope=col>Jugar</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>3</th><td>Nublado</td><td>28</td><td>86</td><td>No</td><td>Si</td></tr>
	<tr><th scope=row>7</th><td>Nublado</td><td>17</td><td>65</td><td>Si</td><td>Si</td></tr>
	<tr><th scope=row>12</th><td>Nublado</td><td>22</td><td>90</td><td>Si</td><td>Si</td></tr>
	<tr><th scope=row>13</th><td>Nublado</td><td>27</td><td>75</td><td>No</td><td>Si</td></tr>
</tbody>
</table>


<table>
<caption>A data.frame: 3 × 5</caption>
<thead>
	<tr><th></th><th scope=col>Perspectiva</th><th scope=col>Temperatura</th><th scope=col>Humedad</th><th scope=col>Viento</th><th scope=col>Jugar</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>4</th><td>Lluvioso</td><td>21</td><td>96</td><td>No</td><td>Si</td></tr>
	<tr><th scope=row>5</th><td>Lluvioso</td><td>20</td><td>80</td><td>No</td><td>Si</td></tr>
	<tr><th scope=row>10</th><td>Lluvioso</td><td>24</td><td>80</td><td>No</td><td>Si</td></tr>
</tbody>
</table>


<table>
<caption>A data.frame: 2 × 5</caption>
<thead>
	<tr><th></th><th scope=col>Perspectiva</th><th scope=col>Temperatura</th><th scope=col>Humedad</th><th scope=col>Viento</th><th scope=col>Jugar</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>9</th><td>Soleado</td><td>20.5</td><td>70</td><td>No</td><td>Si</td></tr>
	<tr><th scope=row>11</th><td>Soleado</td><td>24.0</td><td>70</td><td>Si</td><td>Si</td></tr>
</tbody>
</table>

Al introducir todo en una misma sentencia de código hemos creado una regla estándar.

```R
#Todo junto
Clima$test <- ifelse((Clima$Perspectiva == 'Nublado') |
                        (Clima$Perspectiva == 'Lluvioso' & Clima$Viento == 'No') |
                        (Clima$Perspectiva == 'Soleado' & Clima$Temperatura < 27 & Clima$Humedad < 95),'SI','NO')

Clima
```

<table>
<caption>A data.frame: 14 × 6</caption>
<thead>
	<tr><th scope=col>Perspectiva</th><th scope=col>Temperatura</th><th scope=col>Humedad</th><th scope=col>Viento</th><th scope=col>Jugar</th><th scope=col>test</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Soleado </td><td>29.0</td><td>85</td><td>No</td><td>No</td><td>NO</td></tr>
	<tr><td>Soleado </td><td>27.0</td><td>90</td><td>Si</td><td>No</td><td>NO</td></tr>
	<tr><td>Nublado </td><td>28.0</td><td>86</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Lluvioso</td><td>21.0</td><td>96</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Lluvioso</td><td>20.0</td><td>80</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Lluvioso</td><td>18.0</td><td>70</td><td>Si</td><td>No</td><td>NO</td></tr>
	<tr><td>Nublado </td><td>17.0</td><td>65</td><td>Si</td><td>Si</td><td>SI</td></tr>
	<tr><td>Soleado </td><td>22.0</td><td>95</td><td>No</td><td>No</td><td>NO</td></tr>
	<tr><td>Soleado </td><td>20.5</td><td>70</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Lluvioso</td><td>24.0</td><td>80</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Soleado </td><td>24.0</td><td>70</td><td>Si</td><td>Si</td><td>SI</td></tr>
	<tr><td>Nublado </td><td>22.0</td><td>90</td><td>Si</td><td>Si</td><td>SI</td></tr>
	<tr><td>Nublado </td><td>27.0</td><td>75</td><td>No</td><td>Si</td><td>SI</td></tr>
	<tr><td>Lluvioso</td><td>21.6</td><td>91</td><td>Si</td><td>No</td><td>NO</td></tr>
</tbody>
</table>

Wow!! hemos creado Inteligencia Artificial apartir de entender el comportamiento descriptivo de nuestros datos.
