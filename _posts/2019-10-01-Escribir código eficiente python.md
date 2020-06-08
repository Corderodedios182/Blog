---
title: "Escribir código eficiente con Python."
categories:
  - Python
  - Coding Best Practices
tags:
  - Coding Best Practices
  - Python
date: 2019-10-01T14:37:42
---

## Escribiendo código eficiente con Python

Como científico de datos, la mayor parte del tiempo debemos dedicarlo a obtener información procesable de los datos, sin esperar a que el código termine de ejecutarse.

Escribir un código Python eficiente puede ayudar a reducir el tiempo de ejecución y ahorrar recursos computacionales, lo que finalmente nos libera para hacer las cosas que amamos como Científico de Datos.

Revisaremos el uso de estructuras de datos, funciones y módulos integrados de Python para escribir código más limpio, más rápido y más eficiente.

Exploraremos cómo calcular el tiempo y el código de perfil para encontrar cuellos de botella.

Practicando la eliminación de estos cuellos de botella y otros patrones de diseño incorrectos, utilizando la Biblioteca estándar de Python, NumPy y pandas.


## Fundamentos para la eficiencia.

**¿Qué es eficiente?**

El código que se ejecuta rápidamente para la tarea en cuestión, minimiza la huella de memoria y sigue los principios de estilo de codificación de Python.

**Formas no Pythonic y Pythonic para recorrer una lista**

Imprimiendo los valores con longitud mayor a 6, de lista creada con enfoque no $Pythonic$


```python
names = ['Jerry', 'Kramer', 'Elaine', 'George', 'Newman']

#Forma no Pythonic
i = 0

new_list= []

while i < len(names):
    if len(names[i]) >= 6:
        new_list.append(names[i])
    i += 1

print(new_list)
```

    ['Kramer', 'Elaine', 'George', 'Newman']


Un enfoque más $Pythonic$ recorrería el contenido de names, en lugar de utilizar una variable de índice.


```python
# Imprimiendo la lista creada al recorrer el contenido de los nombres.

better_list = []

for name in names:
    if len(name) >= 6:
        better_list.append(name)

print(better_list)
```

    ['Kramer', 'Elaine', 'George', 'Newman']


La mejor forma $Pythonic$ forma de hacer esto es mediante el uso de listas por comprensión.


```python
# Imprimiendo la lista usando listas de comprension.
best_list = [name for name in names if len(name) >= 6]

print(best_list)
```

    ['Kramer', 'Elaine', 'George', 'Newman']


**Zen de Python**

EL Zen de Python escrito por Tim Peters, enumera 19 modismos que sirven como principios rectores para cualquier Pythonista.

Python tiene cientos de propuestas de mejora Python, comúnmente conocidas como PEP.

El Zen de Python es uno de estos PEP y está documentado como PEP20 .

Un pequeño huevo de Pascua en Python es la capacidad de imprimir el Zen de Python usando el comando "import this".


```python
import this
```

    The Zen of Python, by Tim Peters

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!


Tipos incorporados

- list, tuple, set, dict y otros

Funciones incorporadas

- print(), len(), range(), round(), enumerate(), map(), zip()

Modulos incorporados

- os, sys, itertools, collections, math y otros

**Uso eficiente de range()**

Permite crear secuencias de números personalizadas.


```python
#Podemos escribirlos de estas 2 maneras
list(range(0,20,2))

[*range(0,20,2)]
```




    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]



**Uso de enumerate()**

Esta función es útil para obtener una lista indexada.

Tenemos una lista de personas que llegaron a una fiesta. La lista está ordenada por llegada (Jerry fue el primero en llegar, seguido de Kramer, etc.):


```python
names = ['Jerry', 'Kramer', 'Elaine', 'George', 'Newman']

# Usando un loop con lista de comprension
indexed_names_comp = [(i,name) for i,name in enumerate(names)]
print(indexed_names_comp)

# Desempaquetando un objeto de enumeración con un índice inicial de uno
indexed_names_unpack = [*enumerate(names, 1)]
print(indexed_names_unpack)
```

    [(0, 'Jerry'), (1, 'Kramer'), (2, 'Elaine'), (3, 'George'), (4, 'Newman')]
    [(1, 'Jerry'), (2, 'Kramer'), (3, 'Elaine'), (4, 'George'), (5, 'Newman')]


**Uso de map()**

Aplica una función a cada elemento de un objeto.

Supongamos que queremos convertir las letras de cada nombre de la lista names en mayusculas.


```python
# Usando map() aplicando str.upper a cada elemento de names
names_map  = map(str.upper,names)

# Tipo de dato map
print(type(names_map))

# Desempaquetando names_map a lista
names_uppercase = [*names_map]
print(names_uppercase)
```

    <class 'map'>
    ['JERRY', 'KRAMER', 'ELAINE', 'GEORGE', 'NEWMAN']


**El poder de los arrays con Numpy**

**Me hace falta saber definir la funcion de texto**


```python
import numpy as np

nums = np.array([[1,2,3],[12,3,3]])
```


```python
welcome_guest(guest_arrivals[0][0], guest_arrivals[1][1])
```




    'Bienvenido Jerry... 17 min. tarde.'




```python
# Create a list of arrival times
arrival_times = [*range(10,60,10)]

# Convert arrival_times to an array and update the times
arrival_times_np = np.array(arrival_times)
new_times = arrival_times_np - 3

# Use list comprehension and enumerate to pair guests to new times
guest_arrivals = [(names[i],time) for i,time in enumerate(new_times)]

guest_arrivals
```




    [('Jerry', 7), ('Kramer', 17), ('Elaine', 27), ('George', 37), ('Newman', 47)]




```python
welcome_guest(guest_arrivals)
```




    "Bienvenido ('Jerry', 7)... ('Kramer', 17) min. tarde."



## Tiempo de ejecución

¿Por qué debemos cronometrar nuestro código?

Respuestas:

1. Nos permite elegir el enfoque de codificación óptimo

2. Código más rápido == código más eficiente

**¿Cómo podemos cronometrar nuestro código?**

Calcule el tiempo de ejecución con el comando mágico de IPython %timeit

Comandos mágicos: mejoras sobre la sintaxis normal de Python Predeterminado por el carácter "%"

Todos los comandos mágicos disponibles con %lsmagic


```python
# Creando una lista de comprension

%timeit nums_list_comp = [num for num in range(0,51)]

print(nums_list_comp)
```

    3.6 µs ± 183 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]



```python
# Creando una lista de comprension

%timeit nums_list_comp = [num for num in range(0,51)]

print(nums_list_comp)
```

    3.6 µs ± 183 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]


Desempacar el objeto de rango es más rápido que la comprensión de la lista.


```python
# Creando una lista, desempaquetando

%timeit nums_unpack = [*range(0,51)]

print(nums_unpack)
```

    621 ns ± 5.79 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]


La el comando %timeit tienen un número definido de ejecuciones y bucles.

Pordemos especificar el número de ejecuciones / bucles, establecer el número de ejecuciones (-r) y/o bucles (-n).


```python
#Una ejecución y 10 bucles

%timeit -r1 -n10 nums_list_comp = [num for num in range(51)]

print(nums_list_comp)

```

    4.43 µs ± 0 ns per loop (mean ± std. dev. of 1 run, 10 loops each)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]



```python
# Una ejecucion y 10 bucles

%timeit -r1 -n10 nums_unpack = [*range(0,51)]

print(nums_unpack)
```

    1.35 µs ± 0 ns per loop (mean ± std. dev. of 1 run, 10 loops each)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50]


Analisando el tiempo de ejecución de multiples linea de codito con %%timeit.


```python
%%timeit

nums = []

for x in range(10):
    nums.append(x)
```

    1.07 µs ± 19.8 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)


**Comparación de tiempos**

Las estructuras de datos de Python se pueden crear usando un nombre formal

- formal_list = list()
- formal_dict = dict()
- formal_tuple = tuple()

Las estructuras de datos de Python se pueden crear usando sintaxis literal

- literal_list = []
- literal_dict = {}
- literal_tuple = ()

Veamos los tiempos de creación de cada estructura.


```python
f_time = %timeit -o formal_dict = dict()
```

    129 ns ± 3.02 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)



```python
l_time = %timeit -o literal_dict = {}
```

    49.5 ns ± 1.5 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)



```python
diff = (f_time.average - l_time.average) * (10**9)

print('l_time mejor que f_time por {} ns'.format(diff))
```

    l_time mejor que f_time por 79.7460328998568 ns


**Guardando el tiempo en una variable**


```python
#Guardando el tiempo en una variable

times = %timeit -o rand_nums = np.random.rand(1000)

times.timings
```

    18 µs ± 504 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)





    [1.7536134300462437e-05,
     1.7378414599807003e-05,
     1.766876389956451e-05,
     1.8132931500440464e-05,
     1.882860290061217e-05,
     1.8517823400179622e-05,
     1.7674591000104555e-05]




```python
times.best
```




    1.7378414599807003e-05



**Perfiles de código para tiempo de ejecución**

 - Estadísticas detalladas sobre la frecuencia y la duración de las llamadas a funciones

 - Análisis línea por línea

 - Paquete utilizado: **line_profiler "pip install line_profiler"**


```python
heroes = ['Batman','Superman','Wonder Woman']

hts = np.array([188.0, 191.0, 183.0])
wts = np.array([ 95.0, 101.0, 74.0])

def convert_units(heroes, heights, weights):

    new_hts = [ht * 0.39370 for ht in heights]
    new_wts = [wt * 2.20462 for wt in weights]

    hero_data = {}

    for i,hero in enumerate(heroes):
        hero_data[hero] = (new_hts[i], new_wts[i])
        return hero_data

convert_units(heroes, hts, wts)
```




    {'Batman': (74.01559999999999, 209.4389)}




```python
#Uso del paquete line_profiler
%load_ext line_profiler
```

    The line_profiler extension is already loaded. To reload it, use:
      %reload_ext line_profiler


**Usando %lprun detectamos cuellos de botella**  aqui me quede


```python
#Comando mágico para tiempos línea por línea

%lprun -f convert_units
```


    Timer unit: 1e-06 s

    Total time: 0 s
    File: <ipython-input-51-53cbc6b56d73>
    Function: convert_units at line 6

    Line #      Hits         Time  Per Hit   % Time  Line Contents
    ==============================================================
         6                                           def convert_units(heroes, heights, weights):
         7                                           
         8                                               new_hts = [ht * 0.39370 for ht in heights]
         9                                               new_wts = [wt * 2.20462 for wt in weights]
        10                                               
        11                                               hero_data = {}
        12                                               
        13                                               for i,hero in enumerate(heroes):
        14                                                   hero_data[hero] = (new_hts[i], new_wts[i])
        15                                                   return hero_data



```python
%lprun -f convert_units convert_units(heroes, hts, wts)
```


    Timer unit: 1e-06 s

    Total time: 4.4e-05 s
    File: <ipython-input-51-53cbc6b56d73>
    Function: convert_units at line 6

    Line #      Hits         Time  Per Hit   % Time  Line Contents
    ==============================================================
         6                                           def convert_units(heroes, heights, weights):
         7                                           
         8         1         26.0     26.0     59.1      new_hts = [ht * 0.39370 for ht in heights]
         9         1          8.0      8.0     18.2      new_wts = [wt * 2.20462 for wt in weights]
        10                                               
        11         1          2.0      2.0      4.5      hero_data = {}
        12                                               
        13         1          4.0      4.0      9.1      for i,hero in enumerate(heroes):
        14         1          2.0      2.0      4.5          hero_data[hero] = (new_hts[i], new_wts[i])
        15         1          2.0      2.0      4.5          return hero_data
