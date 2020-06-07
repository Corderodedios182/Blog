---
title: "Análisis exploratorio, filtrado colaborativo y aplicación Shiny con R."
categories:
  - R
  - Inteligencia Artificial
tags:
  - Inteligencia Artificial
  - R
date: 2019-06-01T14:37:42
---

¿Alguna vez te has preguntado qué libro leer?, Las recomendaciones de libros son un tema fascinante.

Este conjunto de datos externo nos permite profundizar en las recomendaciones de libros basados en datos.

El reporte se divide en tres partes:

- Parte I: explora el conjunto de datos para encontrar algunas ideas interesantes.
- Parte II: presenta y demuestra el filtrado colaborativo.
- Parte III: defino la REAS, entorno de trabajo.
- Parte IV: Implementación de aplicación con Shiny R (para recomendar algunos libros (para mí y para usted :-)).

Puede acceder (simplemente haga clic en (https://philippsp.shinyapps.io/BookRecommendation/), subí el código a un servidor para aplicaciones Shiny pero de igual manera comparto el código comentado.

## I. Exploración de los datos

Ha habido buenos conjuntos de datos para películas (Netflix, Movielens) y recomendaciones de música (Million Songs), pero no para libros.

Trabajaré con 3 conjuntos de datos:

- books: Este conjunto de datos contiene calificaciones de diez mil libros populares.
- ratings: Ratings que han colocado 53424 usuarios.
- tags: Etiquetas de genero que han colocado los usuarios.

Vienen a la mente muchas preguntas.

```R
books <- fread('books.csv')
ratings <- fread('ratings.csv')
tags <- fread('tags.csv')
book_tags <- fread('book_tags.csv')
```

```R
cat("23 columna en el dataset : " , names(books))
#Las más importantes para nuestro analísis.
head(select(books, book_id, authors, original_publication_year, language_code, image_url))
```

    23 columna en el dataset :  id book_id best_book_id work_id books_count isbn isbn13 authors original_publication_year original_title title language_code average_rating ratings_count work_ratings_count work_text_reviews_count ratings_1 ratings_2 ratings_3 ratings_4 ratings_5 image_url small_image_url


<table>
<caption>A data.table: 6 × 5</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>authors</th><th scope=col>original_publication_year</th><th scope=col>language_code</th><th scope=col>image_url</th></tr>
	<tr><th scope=col></th><th scope=col></th><th scope=col></th><th scope=col></th><th scope=col></th></tr>
</thead>
<tbody>
	<tr><td> 2767052</td><td>Suzanne Collins            </td><td>2008</td><td>eng  </td><td>https://images.gr-assets.com/books/1447303603m/2767052.jpg </td></tr>
	<tr><td>       3</td><td>J.K. Rowling, Mary GrandPré</td><td>1997</td><td>eng  </td><td>https://images.gr-assets.com/books/1474154022m/3.jpg       </td></tr>
	<tr><td>   41865</td><td>Stephenie Meyer            </td><td>2005</td><td>en-US</td><td>https://images.gr-assets.com/books/1361039443m/41865.jpg   </td></tr>
	<tr><td>    2657</td><td>Harper Lee                 </td><td>1960</td><td>eng  </td><td>https://images.gr-assets.com/books/1361975680m/2657.jpg    </td></tr>
	<tr><td>    4671</td><td>F. Scott Fitzgerald        </td><td>1925</td><td>eng  </td><td>https://images.gr-assets.com/books/1490528560m/4671.jpg    </td></tr>
	<tr><td>11870085</td><td>John Green                 </td><td>2012</td><td>eng  </td><td>https://images.gr-assets.com/books/1360206420m/11870085.jpg</td></tr>
</tbody>
</table>

```R
#Los ratings los cruzaremos con books, tenemos 53424 usuarios unicos que cruzaremos con la base books.
head(ratings)
```

<table>
<caption>A data.table: 6 × 3</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>user_id</th><th scope=col>rating</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td> 314</td><td>5</td></tr>
	<tr><td>1</td><td> 439</td><td>3</td></tr>
	<tr><td>1</td><td> 588</td><td>5</td></tr>
	<tr><td>1</td><td>1169</td><td>4</td></tr>
	<tr><td>1</td><td>1185</td><td>4</td></tr>
	<tr><td>1</td><td>2077</td><td>4</td></tr>
</tbody>
</table>

```R
#Tenemos los generos de los libros
cat("Generos que los usuarios le han colocado a los libros : ", dim(tags))
tags[25:30,]
```

    Generos que los usuarios le han colocado a los libros :  34252 2


<table>
<caption>A data.table: 6 × 2</caption>
<thead>
	<tr><th scope=col>tag_id</th><th scope=col>tag_name</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>24</td><td>-england-  </td></tr>
	<tr><td>25</td><td>-fiction   </td></tr>
	<tr><td>26</td><td>-fictional </td></tr>
	<tr><td>27</td><td>-fictitious</td></tr>
	<tr><td>28</td><td>-football- </td></tr>
	<tr><td>29</td><td>-george    </td></tr>
</tbody>
</table>

### Limpieza del conjunto de datos

Al igual que con casi cualquier conjunto de datos de la vida real, primero debemos hacer un poco de limpieza.

```R
#Los datos de ratings contienen casi 1 millón de filas.
cat("Filas y columnas : ", dim(ratings))
```

    Filas y columnas :  981756 3

Noté que había libros que tenían más de una evaluación por el mismo usuarios, para estos casos promedie los ratings.

```R
#Ejemplo de un usuario que voto más de una vez por libro.
filter(ratings, book_id == 849 & user_id == 11283)

#tmp <- ratings %>% group_by(book_id, user_id) %>% summarise(Conteo = n())
#head(filter(tmp,Conteo > 2))

ratings <- ratings %>% group_by(book_id, user_id) %>% summarise(rating = mean(rating))
```

<table>
<caption>A data.frame: 3 × 3</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>user_id</th><th scope=col>rating</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>849</td><td>11283</td><td>3</td></tr>
	<tr><td>849</td><td>11283</td><td>4</td></tr>
	<tr><td>849</td><td>11283</td><td>4</td></tr>
</tbody>
</table>

```R
#Ejemplo del usuario con más de 3 votaciones en un libro
filter(ratings, book_id == 849 & user_id == 11283)
```

<table>
<caption>A grouped_df: 1 × 3</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>user_id</th><th scope=col>rating</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>849</td><td>11283</td><td>3.666667</td></tr>
</tbody>
</table>

Encontré usuarios que votaron menos de 3 veces, decidí eliminarlos ya que afectaban el modelo de recomendación.

```R
ratings[, N := .N, .(user_id)]

cat('Número de usuarios que calificaron menos de 3 libros: ', uniqueN(ratings[N <= 2, user_id]))

ratings <- ratings[N > 2]
```

    Número de usuarios que calificaron menos de 3 libros:  8302

En cuanto a las bases books y tags se encuentran limpias.

### Comenzemos la exploración

###### ¿Comó se distribuyen las calificaciones?

Vemos que las personas tienden a dar calificaciones bastante positivas a los libros.

La mayoría de las clasificaciones están en el rango 3-5, mientras que muy pocas clasificaciones están en el rango 1-2.


```R
options(repr.plot.width = 5, repr.plot.height = 3)
ratings %>%
  ggplot(aes(x = rating, fill = factor(rating))) +
  geom_bar(color = "grey20") + scale_fill_brewer(palette = "YlGnBu") + guides(fill = FALSE) +
  ggtitle("mayoría de ratings entre 3-5 \nmuy pocos ratings 1-2")
```

![quantifiers](/images/Recomendador/output_16_0.png)

###### Número de valoraciones por usuario

A medida que filtramos nuestras calificaciones, todos los usuarios tienen al menos 3 calificaciones.

Sin embargo, también podemos ver que hay algunos usuarios con muchas calificaciones.

Esto es interesante, porque luego podemos examinar si los evaluadores frecuentes califican los libros de manera diferente a los evaluadores menos frecuentes.

Vamos a volver a esto más adelante.

```R
ratings %>%
  group_by(user_id) %>%
  summarize(number_of_ratings_per_user = n()) %>%
  ggplot(aes(number_of_ratings_per_user)) +
  geom_bar(fill = "cadetblue3", color = "blue") + coord_cartesian(c(3, 200)) +
  ggtitle("Min: 3 , 1°Quantil 5: , Mean: 21,\n 3°Quantil: 26, Max: 197") +
  labs(x = "numero de ratings por usuario", y = "Conteo" )
```

![quantifiers](/images/Recomendador/output_18_0.png)

###### Distribución de valoraciones medias de usuarios

Las personas tienen diferentes tendencias para calificar los libros.

Algunos ya le dan 5 estrellas a un libro mediocre, mientras que otros no le dan 5 estrellas a menos que sea el libro perfecto para ellos.

Tales tendencias se pueden ver en la gráfica a continuación.

En el lado derecho hay un aumento de los usuarios con una calificación media de 5, lo que indica que realmente les gustaron todos los libros (o solo calificaron los libros que realmente les gustan ...).

También podemos ver que casi no hay notorios votantes que califican a todos los libros con un 1.

Estas tendencias serán importantes para el filtrado colaborativo más adelante, y generalmente se restan restando la calificación promedio del usuario de sus calificaciones.

```R
ratings %>%
  group_by(user_id) %>%
  summarize(mean_user_rating = mean(rating)) %>%
  ggplot(aes(mean_user_rating)) +
  geom_histogram(fill = "cadetblue3", color = "grey20") +
  ggtitle("Min: 1 , 1°Quantil 3.5: , Mean: 3.8,\n 3°Quantil: 4.2, Max: 5") +
  labs(x = "Usuarios ratings")
```

    `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.


![quantifiers](/images/Recomendador/output_20_1.png)

###### Número de calificaciones por libro

Podemos ver que en el conjunto de datos subconjunto, la mayoría de los libros tienen alrededor de 18-20 clasificaciones.

```R
ratings %>%
  group_by(book_id) %>%
  summarize(number_of_ratings_per_book = n()) %>%
  ggplot(aes(number_of_ratings_per_book)) +
  geom_bar(fill = "orange", color = "grey20", width = 1) + coord_cartesian(c(0,40)) +
  ggtitle("Min: 1 , 1°Quantil 16: , Mean: 18,\n 3°Quantil: 22, Max: 37") +
  labs(x = "Numero de ratings por libro")
```

![quantifiers](/images/Recomendador/output_22_0.png)

###### Distribución de calificaciones medias de libros

Las calificaciones medias de los libros no revelan ninguna peculiaridad.

```R
ratings %>%
  group_by(book_id) %>%
  summarize(mean_book_rating = mean(rating)) %>%
  ggplot(aes(mean_book_rating)) + geom_histogram(fill = "orange", color = "grey20") + coord_cartesian(c(1,5)) +
  ggtitle("Min: 1.8 , 1°Quantil 3.6: , Mean: 3.8,\n 3°Quantil: 4.1, Max: 5")
```

    `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![quantifiers](/images/Recomendador/output_24_1.png)

###### Distribución de géneros
Extraer los géneros de los libros no es trivial ya que los usuarios asignan etiquetas autoelegidas a los libros,
que pueden o no ser los mismos que los géneros definidos por goodreads.

Como forma pragmática, elegí solo las etiquetas que coinciden con las proporcionadas por goodbooks.

La mayoría de los libros son libros de "Fantasía", "Romance" o "Misterio",mientras que no hay muchos "Libros de cocina" en la base de datos.

```R
genres <- str_to_lower(c("Art", "Biography", "Business", "Chick Lit", "Children's", "Christian", "Classics", "Comics", "Contemporary", "Cookbooks", "Crime", "Ebooks", "Fantasy", "Fiction", "Gay and Lesbian", "Graphic Novels", "Historical Fiction", "History", "Horror", "Humor and Comedy", "Manga", "Memoir", "Music", "Mystery", "Nonfiction", "Paranormal", "Philosophy", "Poetry", "Psychology", "Religion", "Romance", "Science", "Science Fiction", "Self Help", "Suspense", "Spirituality", "Sports", "Thriller", "Travel", "Young Adult"))

exclude_genres <- c("fiction", "nonfiction", "ebooks", "contemporary")
genres <- setdiff(genres, exclude_genres)

available_genres <- genres[str_to_lower(genres) %in% tags$tag_name]
available_tags <- tags$tag_id[match(available_genres, tags$tag_name)]

tmp <- book_tags %>%
  filter(tag_id %in% available_tags) %>%
  group_by(tag_id) %>%
  summarize(n = n()) %>%
  ungroup() %>%
  mutate(sumN = sum(n), percentage = n / sumN) %>%
  arrange(-percentage) %>%
  left_join(tags, by = "tag_id")

tmp %>%
  ggplot(aes(reorder(tag_name, percentage), percentage, fill = percentage)) + geom_bar(stat = "identity") + coord_flip() + scale_fill_distiller(palette = 'YlOrRd') + labs(y = 'Percentage', x = 'Genre') +
  ggtitle("Distribucion de generos libros")
```

![quantifiers](/images/Recomendador/output_26_0.png)

###### Idiomas diferentes

En la base books tenemos una columna que nos indica el lenguaje.

Esto es interesante porque goodreads el sitio donde descargué los datos es de habla inglesa.

Sin embargo, el conjunto de datos contiene algunos libros en diferentes idiomas.

La razón es que normalmente hay varias ediciones de un libro (tanto en el mismo idioma como en diferentes idiomas).

Para este conjunto de datos, parece que se incluyó la edición más popular, que para algunos libros es su idioma original.


```R
p1 <- books %>%
  mutate(language = factor(language_code)) %>%
  group_by(language) %>%
  summarize(number_of_books = n()) %>%
  arrange(-number_of_books) %>%
  ggplot(aes(reorder(language, number_of_books), number_of_books, fill = reorder(language, number_of_books))) +
  geom_bar(stat = "identity", color = "grey20", size = 0.35) + coord_flip() +
  labs(x = "language", title = "english included") + guides(fill = FALSE)

p2 <- books %>%
  mutate(language = factor(language_code)) %>%
  filter(!language %in% c("en-US", "en-GB", "eng", "en-CA", "")) %>%
  group_by(language) %>%
  summarize(number_of_books = n()) %>%
  arrange(-number_of_books) %>%
  ggplot(aes(reorder(language, number_of_books), number_of_books, fill = reorder(language, number_of_books))) +
  geom_bar(stat = "identity", color = "grey20", size = 0.35) + coord_flip() +
  labs(x = "", title = "english excluded") + guides(fill = FALSE)

grid.arrange(p1,p2, ncol=2)
```

![quantifiers](/images/Recomendador/output_28_0.png)

###### Los 10 libros mejor calificados

Es evidente que a los usuarios les gusta
- a) Calvin y Hobbes en general.
- b) compilaciones de libros.

Esto tiene sentido intuitivamente ya que las personas no se interesarán en una compilación completa si no les gustan los libros individuales.


```R
books %>%
  mutate(image = paste0('<img src="', small_image_url, '"></img>')) %>%
  arrange(-average_rating) %>%
  top_n(10,wt = average_rating) %>%
  select(title, ratings_count, average_rating)
```

<table>
<caption>A data.frame: 11 × 3</caption>
<thead>
	<tr><th scope=col>title</th><th scope=col>ratings_count</th><th scope=col>average_rating</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Complete Calvin and Hobbes                                   </td><td> 28900</td><td>4.82</td></tr>
	<tr><td>Words of Radiance (The Stormlight Archive, #2)                   </td><td> 73572</td><td>4.77</td></tr>
	<tr><td>Harry Potter Boxed Set, Books 1-5 (Harry Potter, #1-5)           </td><td> 33220</td><td>4.77</td></tr>
	<tr><td>ESV Study Bible                                                  </td><td>  8953</td><td>4.76</td></tr>
	<tr><td>Mark of the Lion Trilogy                                         </td><td>  9081</td><td>4.76</td></tr>
	<tr><td>It's a Magical World: A Calvin and Hobbes Collection             </td><td> 22351</td><td>4.75</td></tr>
	<tr><td>Harry Potter Boxset (Harry Potter, #1-7)                         </td><td>190050</td><td>4.74</td></tr>
	<tr><td>There's Treasure Everywhere: A Calvin and Hobbes Collection      </td><td> 16766</td><td>4.74</td></tr>
	<tr><td>Harry Potter Collection (Harry Potter, #1-6)                     </td><td> 24618</td><td>4.73</td></tr>
	<tr><td>The Authoritative Calvin and Hobbes: A Calvin and Hobbes Treasury</td><td> 16087</td><td>4.73</td></tr>
	<tr><td>The Indispensable Calvin and Hobbes                              </td><td> 14597</td><td>4.73</td></tr>
</tbody>
</table>

###### Los 10 mejores libros más populares

Al mirar los libros que fueron calificados con mayor frecuencia podemos tener una impresión de la popularidad de un libro.

Podemos ver los 10 mejores libros populares en la tabla a continuación.

```R
books %>%
  mutate(image = paste0('<img src="', small_image_url, '"></img>')) %>%
  arrange(-ratings_count) %>%
  top_n(10,wt = ratings_count) %>%
  select(title, ratings_count, average_rating)
```

<table>
<caption>A data.frame: 10 × 3</caption>
<thead>
	<tr><th scope=col>title</th><th scope=col>ratings_count</th><th scope=col>average_rating</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Hunger Games (The Hunger Games, #1)                 </td><td>4780653</td><td>4.34</td></tr>
	<tr><td>Harry Potter and the Sorcerer's Stone (Harry Potter, #1)</td><td>4602479</td><td>4.44</td></tr>
	<tr><td>Twilight (Twilight, #1)                                 </td><td>3866839</td><td>3.57</td></tr>
	<tr><td>To Kill a Mockingbird                                   </td><td>3198671</td><td>4.25</td></tr>
	<tr><td>The Great Gatsby                                        </td><td>2683664</td><td>3.89</td></tr>
	<tr><td>The Fault in Our Stars                                  </td><td>2346404</td><td>4.26</td></tr>
	<tr><td>The Hobbit                                              </td><td>2071616</td><td>4.25</td></tr>
	<tr><td>The Catcher in the Rye                                  </td><td>2044241</td><td>3.79</td></tr>
	<tr><td>Pride and Prejudice                                     </td><td>2035490</td><td>4.24</td></tr>
	<tr><td><span style=white-space:pre-wrap>Angels &amp; Demons  (Robert Langdon, #1)                   </span></td><td>2001311</td><td>3.85</td></tr>
</tbody>
</table>


###### ¿Qué influye en la calificación de un libro?
A continuación, podemos ver si podemos encontrar asociaciones de características con la calificación de un libro.

Para un vistazo rápido, primero grafiquemos la matriz de correlación entre el promedio de libros y algunas variables.

```R
tmp <- books %>%
  select(one_of(c("books_count","original_publication_year","ratings_count", "work_ratings_count", "work_text_reviews_count", "average_rating"))) %>%
  as.matrix()

corrplot(cor(tmp, use = 'pairwise.complete.obs'), type = "lower")
```

![quantifiers](/images/Recomendador/output_34_0.png)

En resumen, solo vemos pequeñas correlaciones entre las características y la calificación promedio (última fila),
lo que indica que no hay relaciones sólidas entre la calificación que recibe un libro y las metavariables
(como la calificación cuenta, etc.).

Esto significa que la calificación depende más de otras características (por ejemplo, la calidad de los libros en sí).

###### ¿Existe una relación entre el número de calificaciones y la calificación promedio?

Teóricamente, podría ser que la popularidad de un libro (en términos de la cantidad de calificaciones que recibe)
está asociada con la calificación promedio que recibe, de modo que una vez que un libro se está volviendo popular,
obtiene mejores calificaciones.


```R
get_cor <- function(df){
  m <- cor(df$x,df$y, use="pairwise.complete.obs");
  eq <- substitute(italic(r) == cor, list(cor = format(m, digits = 2)))
  as.character(as.expression(eq));                 
}

books %>%
  filter(ratings_count < 1e+5) %>%
  ggplot(aes(ratings_count, average_rating)) + stat_bin_hex(bins = 50) + scale_fill_distiller(palette = "Spectral") +
  stat_smooth(method = "lm", color = "orchid", size = 2) +
  annotate("text", x = 85000, y = 2.7, label = get_cor(data.frame(x = books$ratings_count, y = books$average_rating)), parse = TRUE, color = "orchid", size = 7) +
  ggtitle("Numero de calificaciones vs rating promedio")
```

![quantifiers](/images/Recomendador/output_37_0.png)

Sin embargo, nuestros datos muestran que esto es cierto solo en un grado muy pequeño.

La correlación entre estas variables es de solo 0.045.

###### Múltiples ediciones de cada libro.
El conjunto de datos contiene información sobre cuántas ediciones de un libro están disponibles en book_count.

Estas pueden ser ediciones diferentes en el mismo idioma o también traducciones del libro a diferentes idiomas.

Entonces, uno podría suponer que cuanto mejor sea el libro, más ediciones deberían estar disponibles.


```R
books %>% filter(books_count <= 500) %>% ggplot(aes(books_count, average_rating)) + stat_bin_hex(bins = 50) + scale_fill_distiller(palette = "Spectral") +
  stat_smooth(method = "lm", color = "orchid", size = 2) +
  annotate("text", x = 400, y = 2.7, label = get_cor(data.frame(x = books$books_count, y = books$average_rating)), parse = TRUE, color = "orchid", size = 7) +
  ggtitle("Ediciones de un libro vs rating promedio")
```

![quantifiers](/images/Recomendador/output_40_0.png)

De hecho, los datos muestran exactamente el patrón opuesto: cuantas más ediciones tenga un libro,
menor será la calificación promedio.

La dirección causal de esta asociación, por supuesto, no está clara aquí.

###### ¿Los evaluadores frecuentes califican de manera diferente?

Es posible que los usuarios que califican más libros (calificadores frecuentes)
califiquen los libros de manera diferente a los calificadores menos frecuentes.

La siguiente figura explora esta posibilidad.


```R
tmp <- ratings %>%
  group_by(user_id) %>%
  summarize(mean_rating = mean(rating), number_of_rated_books = n())

tmp %>% filter(number_of_rated_books <= 100) %>%
  ggplot(aes(number_of_rated_books, mean_rating)) + stat_bin_hex(bins = 50) + scale_fill_distiller(palette = "Spectral") + stat_smooth(method = "lm", color = "orchid", size = 2, se = FALSE) +
  annotate("text", x = 80, y = 1.9, label = get_cor(data.frame(x = tmp$number_of_rated_books, y = tmp$mean_rating)), color = "orchid", size = 7, parse = TRUE) +
  ggtitle("Usuarios que califican más de 100 libros")
```

![quantifiers](/images/Recomendador/output_43_0.png)


Parece que los evaluadores frecuentes tienden a dar calificaciones más bajas a los libros, tal vez sean / se vuelvan más críticos a medida que leen y califican. Eso es interesante.

###### Serie de libros

Los datos contienen información en la columna del título sobre si cierto libro es parte de una serie (por ejemplo, la trilogía de El señor de los anillos).

```R
books %>%
  filter(str_detect(str_to_lower(title), '\\(the lord of the rings')) %>%
  select(book_id, title, average_rating)
```

<table>
<caption>A data.frame: 4 × 3</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>title</th><th scope=col>average_rating</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>   34</td><td>The Fellowship of the Ring (The Lord of the Rings, #1)</td><td>4.34</td></tr>
	<tr><td>15241</td><td>The Two Towers (The Lord of the Rings, #2)            </td><td>4.42</td></tr>
	<tr><td>18512</td><td>The Return of the King (The Lord of the Rings, #3)    </td><td>4.51</td></tr>
	<tr><td>   33</td><td>The Lord of the Rings (The Lord of the Rings, #1-3)   </td><td>4.47</td></tr>
</tbody>
</table>


Dado esto, podemos extraer la serie del título, ya que siempre se da entre paréntesis, puede calcular características como el número de volúmenes en una serie, y así sucesivamente.

A continuación, examino si los libros que forman parte de una serie más grande reciben una calificación más alta.
De hecho, cuanto más volúmenes hay en una serie, mayor es la calificación promedio.

```R
books <- books %>%
  mutate(series = str_extract(title, "\\(.*\\)"),
         series_number = as.numeric(str_sub(str_extract(series, ', #[0-9]+\\)$'),4,-2)),
         series_name = str_sub(str_extract(series, '\\(.*,'),2,-2))

tmp <- books %>%
  filter(!is.na(series_name) & !is.na(series_number)) %>%
  group_by(series_name) %>%
  summarise(number_of_volumes_in_series = n(), mean_rating = mean(average_rating))

tmp %>%
  ggplot(aes(number_of_volumes_in_series, mean_rating)) + stat_bin_hex(bins = 50) + scale_fill_distiller(palette = "Spectral") +
  stat_smooth(method = "lm", se = FALSE, size = 2, color = "orchid") +
  annotate("text", x = 35, y = 3.95, label = get_cor(data.frame(x = tmp$mean_rating,  y = tmp$number_of_volumes_in_series)), color = "orchid", size = 7, parse = TRUE) +
  ggtitle("Calificaciones en series de libros")
```

![quantifiers](/images/Recomendador/output_48_0.png)

###### ¿La secuela es mejor que la original?

También podemos ver que dentro de una serie, de hecho, la secuela tiene una calificación ligeramente mejor que la original.

```R
books %>%
  filter(!is.na(series_name) & !is.na(series_number) & series_number %in% c(1,2)) %>%
  group_by(series_name, series_number) %>%
  summarise(m = mean(average_rating)) %>%
  ungroup() %>%
  group_by(series_name) %>%
  mutate(n = n()) %>%
  filter(n == 2) %>%
  ggplot(aes(factor(series_number), m, color = factor(series_number))) +
  geom_boxplot() + coord_cartesian(ylim = c(3,5)) + guides(color = FALSE) + labs(x = "Volume of series", y = "Average rating") +
  ggtitle("Original (rojo), contra Secuela (azul)")
```

![quantifiers](/images/Recomendador/output_50_0.png)

###### ¿Cuánto debe durar un título?

Si eres autor, una de las opciones más importantes es el título de un libro.

Por supuesto, el contenido del título es importante.
Sin embargo, también podría importar cuánto dura el título.

Por lo tanto, trazo la calificación promedio en función de la longitud del título (en palabras).

Podemos ver que, de hecho, hay alguna variación en la calificación promedio dependiendo de la duración del título.


```R
books <- books %>%
  mutate(title_cleaned = str_trim(str_extract(title, '([0-9a-zA-Z]| |\'|,|\\.|\\*)*')),
         title_length = str_count(title_cleaned, " ") + 1)

tmp <- books %>%
  group_by(title_length) %>%
  summarize(n = n()) %>%
  mutate(ind = rank(title_length))

books %>%
  ggplot(aes(factor(title_length), average_rating, color=factor(title_length), group=title_length)) +
  geom_boxplot() + guides(color = FALSE) + labs(x = "Title length") + coord_cartesian(ylim = c(2.2,4.7)) + geom_text(aes(x = ind,y = 2.25,label = n), data = tmp) +
  ggtitle("3 o 7 palabras parecen ligeramente mejores calificaciones")
```

![quantifiers](/images/Recomendador/output_52_0.png)

###### ¿Tener un subtítulo mejora la calificación del libro?

Vemos que los libros que tienen un subtítulo obtienen una calificación ligeramente más alta que los libros sin subtítulo.


```R
books <- books %>%
  mutate(subtitle = str_detect(books$title, ':') * 1, subtitle = factor(subtitle))

books %>%
  ggplot(aes(subtitle, average_rating, group = subtitle, color = subtitle)) +
  geom_boxplot() + guides(color = FALSE) +
  ggtitle("Con subtitulo (rojo), Sin subtitulo (azul)")
```

![quantifiers](/images/Recomendador/output_54_0.png)


###### ¿Importa el número de autores?
Todos conocemos el dicho: "demasiados cocineros estropean el caldo".

¿Es esto también cierto para los libros? Mirando la trama a continuación,parece ser exactamente lo contrario: cuantos más autores tenga un libro, mayor será su calificación promedio.


```R
books <- books %>%
  group_by(book_id) %>%
  mutate(number_of_authors = length(str_split(authors, ",")[[1]]))

books %>% filter(number_of_authors <= 10) %>%
  ggplot(aes(number_of_authors, average_rating)) + stat_bin_hex(bins = 50) + scale_fill_distiller(palette = "Spectral") +
  stat_smooth(method = "lm", size = 2, color = "orchid", se = FALSE) +
  annotate("text", x = 8.5, y = 2.75, label = get_cor(data.frame(x = books$number_of_authors, y = books$average_rating)), color = "orchid", size = 7, parse = TRUE) +
  ggtitle("más autores en un libro, mayor calificación promedio.")
```

![quantifiers](/images/Recomendador/output_56_0.png)

## Resumen - Parte I

Identificamos algunos aspectos interesantes de los conjuntos de datos de libros y usuarios.

En resumen, los efectos observados en la calificación del libro son bastante pequeños,
lo que sugiere que la calificación del libro se debe principalmente a otros aspectos,
con la esperanza de incluir la calidad del libro en sí.

# Parte II: Filtrado colaborativo

El filtrado colaborativo es un método estándar para recomendaciones de productos.

Para obtener la idea general, veamos un ejemplo:

Queremos leer un libro nuevo, pero no sabemos cuál vale la pena leer.
Tenemos cierto amigo, con quien hablamos sobre libros y por lo general he tenido una opinión bastante similar sobre esos libros.
Entonces sería una buena idea preguntarle a este amigo si leyó y le gustaron algunos libros que aún no conoce.

Estos serían buenos candidatos para un próximo libro.

Lo que describí anteriormente es exactamente la idea principal del llamado filtrado colaborativo basado en el usuario.

Funciona de la siguiente manera:

###### 1 . Primero identificamos a otros usuarios similares al usuario actual en términos de sus calificaciones en el mismo conjunto de libros.

Por ejemplo, si le gustaron todos los libros de “El señor de los anillos”, identifiqué a los usuarios que también les gustaron esos libros.

###### 2. Se encuentra usuarios similares, tomo su calificación promedio de libros que el usuario actual aún no ha leído ...

Entonces, ¿cómo calificaron esos amantes del "Señor de los anillos" otros libros?
Tal vez calificaron "El Hobbit" muy alto.

###### 3. Recomiendo aquellos libros con la calificación promedio más alta para él.

En consecuencia, "El Hobbit" tiene una calificación promedio alta y podría recomendarlo.

### Estos tres pasos se pueden traducir fácilmente en un algoritmo.

Sin embargo, antes de que podamos hacer eso, tenemos que reestructurar nuestros datos.

Para el filtrado colaborativo, los datos generalmente se estructuran para que cada fila corresponda a un usuario
y cada columna corresponda a un libro.

Esto podría verse así, por ejemplo, para 3 usuarios y 5 libros, tenemos 3 filas y 5 columnas.

Tenga en cuenta que no todos los usuarios calificaron cada libro.

Por ejemplo, el usuario 1 solo calificó el libro 3, mientras que el usuario 2 calificó el libro 1 y el libro 2.


```R
head(ratings)
```

<table>
<caption>A data.table: 6 × 4</caption>
<thead>
	<tr><th scope=col>book_id</th><th scope=col>user_id</th><th scope=col>rating</th><th scope=col>N</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td> 1169</td><td>4</td><td>187</td></tr>
	<tr><td>1</td><td> 1185</td><td>4</td><td>190</td></tr>
	<tr><td>1</td><td> 3662</td><td>4</td><td>185</td></tr>
	<tr><td>1</td><td>12471</td><td>5</td><td>192</td></tr>
	<tr><td>1</td><td>13282</td><td>5</td><td>182</td></tr>
	<tr><td>1</td><td>13544</td><td>5</td><td>183</td></tr>
</tbody>
</table>


```R
#Para reestructurar nuestros datos de calificación de este conjunto de datos de la misma manera,
#podemos hacer lo siguiente:

dimension_names <- list(user_id = sort(unique(ratings$user_id)), book_id = sort(unique(ratings$book_id)))
ratingmat <- spread(select(ratings, book_id, user_id, rating), book_id, rating) %>% select(-user_id)

ratingmat <- as.matrix(ratingmat)
dimnames(ratingmat) <- dimension_names
```


```R
ratingmat[1:5, 1:5]
cat("Matriz con filas (usuarios) y columnas (libros) : ", str(ratingmat))
```


<table>
<caption>A matrix: 5 × 5 of type int</caption>
<thead>
	<tr><th></th><th scope=col>1</th><th scope=col>2</th><th scope=col>3</th><th scope=col>4</th><th scope=col>5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>2</th><td>NA</td><td>NA</td><td>NA</td><td>NA</td><td>NA</td></tr>
	<tr><th scope=row>4</th><td>NA</td><td>NA</td><td>NA</td><td>NA</td><td>NA</td></tr>
	<tr><th scope=row>5</th><td>NA</td><td>NA</td><td>NA</td><td>NA</td><td>NA</td></tr>
	<tr><th scope=row>9</th><td>NA</td><td>NA</td><td>NA</td><td>NA</td><td>NA</td></tr>
	<tr><th scope=row>10</th><td>NA</td><td>NA</td><td>NA</td><td>NA</td><td>NA</td></tr>
</tbody>
</table>



     int [1:9003, 1:9999] NA NA NA NA NA NA NA NA NA NA ...
     - attr(*, "dimnames")=List of 2
      ..$ user_id: chr [1:9003] "2" "4" "5" "9" ...
      ..$ book_id: chr [1:9999] "1" "2" "3" "4" ...
    Matriz con filas (usuarios) y columnas (libros) :

Vemos que nuestra matriz de calificación tiene 9003 filas x 9999 columnas, que son muchos elementos de matriz.

Ok, ahora tenemos los datos.

###### Es hora de seguir nuestros 3 pasos:

###### Paso 1: busca usuarios similares

Para este paso seleccionamos usuarios que tienen en común calificaciones en los mismos libros.

Para hacerlo más fácil, seleccione un usuario de ejemplo "David" (user_id: 17329).

Primero seleccionamos a los usuarios que calificaron al menos un libro que David también calificó.


```R
current_user <- "17329"
#Seleccion de libros de un usuario
rated_items <- which(!is.na((as.data.frame(ratingmat[current_user, ]))))
#Usuario con al menos un libro seleccionado por David
selected_users <- names(which(apply(!is.na(ratingmat[ ,rated_items]), 1, sum) >= 2))
head(selected_users, 40) #Encabezados con libro en comun con David
```


<ol class=list-inline>
	<li>'35'</li>
	<li>'153'</li>
	<li>'158'</li>
	<li>'202'</li>
	<li>'343'</li>
	<li>'368'</li>
	<li>'958'</li>
	<li>'1169'</li>
	<li>'1185'</li>
	<li>'1339'</li>
	<li>'1449'</li>
	<li>'1456'</li>
	<li>'1464'</li>
	<li>'1518'</li>
	<li>'1571'</li>
	<li>'1634'</li>
	<li>'1677'</li>
	<li>'1759'</li>
	<li>'2166'</li>
	<li>'2218'</li>
	<li>'2347'</li>
	<li>'2421'</li>
	<li>'2467'</li>
	<li>'2619'</li>
	<li>'3050'</li>
	<li>'3075'</li>
	<li>'3246'</li>
	<li>'3263'</li>
	<li>'3399'</li>
	<li>'3580'</li>
	<li>'3641'</li>
	<li>'3662'</li>
	<li>'3757'</li>
	<li>'3796'</li>
	<li>'4005'</li>
	<li>'4204'</li>
	<li>'4242'</li>
	<li>'4276'</li>
	<li>'4289'</li>
	<li>'4489'</li>
</ol>




```R
cat("En total hay 440 usuarios que tienen al menos un libro en común.", str(selected_users))
```

     chr [1:440] "35" "153" "158" "202" "343" "368" "958" "1169" "1185" "1339" ...
    En total hay 440 usuarios que tienen al menos un libro en común.

Para estos usuarios, podemos calcular la similitud de sus calificaciones con las calificaciones de "David".

Hay varias opciones para calcular la similitud.

Por lo general, se utilizan la similitud del coseno o el coeficiente de correlación de Pearson.

Aquí, elegí la correlación de pearson.
Ahora revisaremos todos los usuarios seleccionados y calcularemos la similitud entre sus calificaciones y las de David.

A continuación hago esto para 2 usuarios (user_ids: 1339 y 21877) a modo de ilustración.
Podemos ver que la similitud es mayor para el usuario 1339 que para el usuario 21877



```R
#Usuario actual 17329
user1 <- data.frame(item=colnames(ratingmat),rating=ratingmat[current_user,]) %>% filter(!is.na(rating))
#Usuario 1329, almenos un libro en comun
user2 <- data.frame(item=colnames(ratingmat),rating=ratingmat["1339",]) %>% filter(!is.na(rating))
tmp<-merge(user1, user2, by="item")
tmp

cat("Coeficiente de correlacion con el usuario 1329 : ", cor(tmp$rating.x, tmp$rating.y, use="pairwise.complete.obs"))
```


<table>
<caption>A data.frame: 3 × 3</caption>
<thead>
	<tr><th scope=col>item</th><th scope=col>rating.x</th><th scope=col>rating.y</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1258</td><td>4</td><td>5</td></tr>
	<tr><td>1662</td><td>4</td><td>5</td></tr>
	<tr><td>1757</td><td>3</td><td>3</td></tr>
</tbody>
</table>



    Coeficiente de correlacion con el usuario 1329 :  1


```R
#Usuario 21877
user2 <- data.frame(item = colnames(ratingmat), rating = ratingmat["21877", ]) %>% filter(!is.na(rating))
tmp <- merge(user1, user2, by="item")
tmp

cat("Coeficiente de correlacion con el usuario 21877 : ",cor(tmp$rating.x, tmp$rating.y, use="pairwise.complete.obs"))
```


<table>
<caption>A data.frame: 4 × 3</caption>
<thead>
	<tr><th scope=col>item</th><th scope=col>rating.x</th><th scope=col>rating.y</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>105 </td><td>4</td><td>3</td></tr>
	<tr><td>1365</td><td>3</td><td>4</td></tr>
	<tr><td>1584</td><td>1</td><td>5</td></tr>
	<tr><td>318 </td><td>4</td><td>4</td></tr>
</tbody>
</table>



    Coeficiente de correlacion con el usuario 21877 :  -0.8660254

Podemos calcular la similitud de todos los demás usuarios con David y clasificarlos según la mayor similitud.


```R
rmat <- ratingmat[selected_users, ]
user_mean_ratings <- rowMeans(rmat,na.rm=T)
rmat <- rmat - user_mean_ratings

similarities <- cor(t(rmat[rownames(rmat)!=current_user, ]), rmat[current_user, ], use = 'pairwise.complete.obs')
sim <- as.vector(similarities)
names(sim) <- rownames(similarities)
res <- sort(sim, decreasing = TRUE)
head(sort(res, decreasing = TRUE), 5)
```

    Warning message in cor(t(rmat[rownames(rmat) != current_user, ]), rmat[current_user, :
    “the standard deviation is zero”


<dl class=dl-horizontal>
	<dt>1339</dt>
		<dd>1</dd>
	<dt>1449</dt>
		<dd>1</dd>
	<dt>2347</dt>
		<dd>1</dd>
	<dt>6290</dt>
		<dd>1</dd>
	<dt>8682</dt>
		<dd>1</dd>
</dl>



Ahora podemos seleccionar los 4 usuarios más similares: 1339, 1449, 2347, 6290

##### Visualizando similitudes entre usuarios
Las similitudes entre los usuarios se pueden visualizar utilizando el paquete qpraph.

El ancho de los bordes del gráfico corresponde a la similitud (azul para correlaciones positivas, rojo para correlaciones negativas).


```R
sim_mat <- cor(t(rmat), use = 'pairwise.complete.obs')
random_users <- selected_users[1:20]
qgraph(sim_mat[c(current_user, random_users), c(current_user, random_users)], layout = "spring", vsize = 5, theme = "TeamFortress", labels = c(current_user, random_users))
```

    Warning message in cor(t(rmat), use = "pairwise.complete.obs"):
    “the standard deviation is zero”Warning message in qgraph(sim_mat[c(current_user, random_users), c(current_user, :
    “Non-finite weights are omitted”

![quantifiers](/images/Recomendador/output_75_1.png)

###### Paso 2: Predicciones para otros libros
Para obtener recomendaciones para nuestro usuario, tomaríamos los usuarios más similares (por ejemplo, 4)
y promediaríamos sus calificaciones para los libros que David aún no ha calificado.

Para que estos promedios sean más confiables, también podemos incluir solo elementos que hayan sido calificados
por varios otros usuarios similares.


```R
similar_users <- names(res[1:4])

similar_users_ratings <- data.frame(item = rep(colnames(rmat), length(similar_users)), rating = c(t(as.data.frame(rmat[similar_users,])))) %>% filter(!is.na(rating))

current_user_ratings <- data.frame(item = colnames(rmat), rating = rmat[current_user,]) %>% filter(!is.na(rating))

predictions <- similar_users_ratings %>%
  filter(!(item %in% current_user_ratings$item)) %>%
  group_by(item) %>% summarize(mean_rating = mean(rating))

predictions <- predictions[order(predictions$mean_rating, decreasing = TRUE),]
head(predictions,10)
```


<table>
<caption>A tibble: 10 × 2</caption>
<thead>
	<tr><th scope=col>item</th><th scope=col>mean_rating</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1031</td><td>1.754386</td></tr>
	<tr><td>2004</td><td>1.754386</td></tr>
	<tr><td>3934</td><td>1.754386</td></tr>
	<tr><td>5524</td><td>1.754386</td></tr>
	<tr><td>7239</td><td>1.754386</td></tr>
	<tr><td>1005</td><td>1.528846</td></tr>
	<tr><td>378 </td><td>1.528846</td></tr>
	<tr><td>779 </td><td>1.528846</td></tr>
	<tr><td>857 </td><td>1.528846</td></tr>
	<tr><td>867 </td><td>1.528846</td></tr>
</tbody>
</table>



######  Paso 3: Recomiende las 5 mejores predicciones
Dados los resultados, clasificaríamos las predicciones con respecto a su calificación promedio y recomendaríamos los libros mejor calificados a David.

En nuestro caso serían los libros 1031, 2004, 3934, 5524, 7239.

Tengamos en cuenta que normalizamos la base, para evitar sesto y centrar mejor los datos.


```R
predictions %>%
  arrange(-mean_rating) %>%
  top_n(5, wt = mean_rating) %>%
  mutate(book_id = as.numeric(as.character(item))) %>%
  left_join(select(books, authors, title, book_id), by = "book_id") %>%
  select(-item)
```


<table>
<caption>A tibble: 5 × 4</caption>
<thead>
	<tr><th scope=col>mean_rating</th><th scope=col>book_id</th><th scope=col>authors</th><th scope=col>title</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1.754386</td><td>1031</td><td>NA</td><td>NA</td></tr>
	<tr><td>1.754386</td><td>2004</td><td>NA</td><td>NA</td></tr>
	<tr><td>1.754386</td><td>3934</td><td>NA</td><td>NA</td></tr>
	<tr><td>1.754386</td><td>5524</td><td>NA</td><td>NA</td></tr>
	<tr><td>1.754386</td><td>7239</td><td>NA</td><td>NA</td></tr>
</tbody>
</table>



Y eso ya es todo.

Este es el principio básico del filtrado colaborativo basado en el usuario.

Ahora podemos ser más prácticos y evaluar y comparar algunos algoritmos de recomendación.

#### Usando recomenderlab

Recomenderlab es un paquete R. Proporciona una infraestructura de investigación para probar y desarrollar algoritmos de recomendación que incluyen UBCF, IBCF, FunkSVD y algoritmos basados en reglas de asociación.

El paquete admite conjuntos de datos de clasificación (p. Ej., 1-5 estrellas) y unarios (0-1).
Los algoritmos admitidos son:

- Filtrado colectivo basado en el usuario (UBCF)
- Filtrado colectivo basado en elementos (IBCF)
- SVD con imputación media de columna (SVD)
- Funk SVD (SVDF)
- Mínimos cuadrados alternos (ALS)
- Factorización de matrices con LIBMF (LIBMF)
- Asociación recomendada basada en reglas (AR)
- Artículos populares (POPULARES)
- Elementos elegidos al azar para la comparación (ALEATORIO)
- Vuelva a recomendar los artículos que le gustaron (RECOMENDAR)
- Recomendaciones híbridas (HybridRecommender)

Para la evaluación, el marco admite protocolos dados-n y todo-pero-x con

Muchos algoritmos ya están implementados en el paquete, y podemos usar los disponibles para ahorrar algo de
esfuerzo de codificación, o agregar algoritmos personalizados y usar la infraestructura (por ejemplo, validación cruzada).

Hay un aspecto importante con respecto a la representación de nuestra matriz de calificación.

Como ya pudimos ver arriba, faltan la mayoría de los valores en la matriz de calificación,porque cada usuario acaba de calificar algunos de los 10000 libros.

Esto nos permite representar esta matriz en un formato escaso para ahorrar memoria.


```R
ratingmat0 <- ratingmat
ratingmat0[is.na(ratingmat0)] <- 0
sparse_ratings <- as(ratingmat0, "sparseMatrix")
rm(ratingmat0)
gc()
```


<table>
<caption>A matrix: 2 × 6 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>used</th><th scope=col>(Mb)</th><th scope=col>gc trigger</th><th scope=col>(Mb)</th><th scope=col>max used</th><th scope=col>(Mb)</th></tr>
</thead>
<tbody>
	<tr><th scope=row>Ncells</th><td> 2423528</td><td>129.5</td><td>  4703850</td><td> 251.3</td><td>  4703850</td><td> 251.3</td></tr>
	<tr><th scope=row>Vcells</th><td>60347225</td><td>460.5</td><td>277139965</td><td>2114.5</td><td>330762405</td><td>2523.6</td></tr>
</tbody>
</table>




```R
#Recommendedderlab utiliza como una variante especial de matrices dispersas,
#por lo que primero convertimos a esta clase.

real_ratings <- new("realRatingMatrix", data = sparse_ratings)
real_ratings
```


    9003 x 9999 rating matrix of class ‘realRatingMatrix’ with 191512 ratings.


Ejecutar un algoritmo en Recomenderlab es realmente fácil.

Todo lo que tenemos que hacer es utilizar la función Recomender() y pasar los datos,
seleccionar un método ("UBCF" - filtrado colaborativo basado en el usuario) y pasar algunos parámetros (por ejemplo, el método para calcular la similitud, por ejemplo, Pearson, y el número de usuarios similares utilizados para las predicciones, nn, p. ej. 4).


```R
model <- Recommender(real_ratings, method = "UBCF", param = list(method = "pearson", nn = 4))
```

La creación de predicciones también es sencilla.

Simplemente pasando a la función a predict() el modelo, las calificaciones para el usuario para el que deseamos predecir las calificaciones y un parámetro para indicarle a la función las calificaciones pronosticadas.


```R
#Predicciones
prediction <- predict(model, real_ratings[current_user, ], type = "ratings")
```


```R
#Echemos un vistazo a las mejores predicciones para David:

as(prediction, 'data.frame') %>%
  arrange(-rating) %>% .[1:5,] %>%
  mutate(book_id = as.numeric(as.character(item))) %>%
  left_join(select(books, authors, title, book_id), by = "book_id") %>%
  select(-item)
```


<table>
<caption>A data.frame: 5 × 5</caption>
<thead>
	<tr><th scope=col>user</th><th scope=col>rating</th><th scope=col>book_id</th><th scope=col>authors</th><th scope=col>title</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>17329</td><td>4.079354</td><td> 523</td><td>NA</td><td>NA</td></tr>
	<tr><td>17329</td><td>3.850361</td><td>1031</td><td>NA</td><td>NA</td></tr>
	<tr><td>17329</td><td>3.850361</td><td>2004</td><td>NA</td><td>NA</td></tr>
	<tr><td>17329</td><td>3.850361</td><td>3934</td><td>NA</td><td>NA</td></tr>
	<tr><td>17329</td><td>3.850361</td><td>5524</td><td>NA</td><td>NA</td></tr>
</tbody>
</table>



Vemos que las recomendaciones son casi las mismas en comparación con nuestro algoritmo descrito anteriormente.

Los 4 mejores libros recomendados son exactamente los mismos.

## Resumen Parte II

En el filtrado colaborativo, la idea principal es usar calificaciones de usuarios similares para crear recomendaciones.


# Parte III: Definición REAS, entorno de trabajo.

- Rendimiento: Recomendaciones adecuadas para tomar en cuenta.
- Entorno: Calificar libros
- Actuadores: Filtrado colaborativo
- Sensores: Matriz usuario y libros

###### Entorno de Trabajo

- Totalmente Observable: Los sensores detectan todos los aspectos relevantes.
- Determinista: El siguiente estado está totalmente determinado por su estado actual.
- Secuencial: Una desición presente afecta desiciones futuras.
- Dinámico: El entorno cambia.
- Continuo: No es posible enumerar los estados.
- Agente individual: Un solo agente resolviendo el problema.

###### Ventajas del agente

- Busca la similitud de un usuario para recomendar
- Nos ayuda a conocer el comportamiento de usuarios

###### Desventajas del agente

- Es necesario tener una buena base de datos, estructurada y grande
- Entre más datos tengamos mayor computo necesitamos para los algoritmos



# Parte IV: Implementación de aplicación con Shiny R.

##### Consideraciones Generales

La aplicación se encuentra en la carpeta RecomendacionLibros.Los archivos principales de la aplicación son:

- server.R, Script con el motor de la aplicación (Llama a los script cf_algoritms y similarity_measures.R)

    1. cf_algorithm.R, contiene le algoritmo de filtrado colaborativo y la función de predicción.

    2. simililarity_measure.R contiene funciones para calcular matrices de similitud.

- uir.R, Script con el diseño de la aplicacioón (Llama al script helpesr.R)

    1. helpesr.R contiene estilos css.

###### Aplicación Shiny

Para crear la aplicación shiny, tomé la implementación de filtrado colaborativo descrito arriba.

Los datos deben proporcionarse como una matriz de libro x usuario (en contraste con la matriz de usuario x libro que utilicé anteriormente).

También submuestreé solo el 20% de los datos de calificación para aumentar aún más la velocidad.

##### Obteniendo las predicciones

Se toman las calificaciones de los nuevos usuarios (como un ejemplo del usuario actual), ejecuta la predicción y luego organiza los resultados de la predicción como las 20 recomendaciones principales.

###### Construyendo la aplicación

La construcción y el despliegue reales de la aplicación también son realmente fáciles, esta alojada en un servidor web que R tienen para aplicaciones.

Todo ocurre con los archivos ui.R y server.R.

La interfaz de IU estática, que es básicamente un tablero con dos cuadros grandes (uno para las calificaciones de los usuarios y otro para las recomendaciones).

server.R primero llena el cuadro de calificación del usuario con libros que se pueden calificar,luego calcula las recomendaciones cuando el usuario presiona el botón y, en tercer lugar,llena el cuadro de las recomendaciones.

Simplemente creé una cuenta en shinyapps.io configure mi Rstudio para publicar mi aplicación.
Aquí como subir aplicaciones shiny a la web.

(https://docs.rstudio.com/shinyapps.io/getting-started.html)

## Resumen Parte IV

Eso es. Cuando comencé este proyecto, no había hecho demasiado filtrado colaborativo, pero después de ver este conjunto de datos,me fascinó crear una aplicación de recomendación de libros de trabajo.

Todavía hay margen de mejora.

si necesitamos conocer que libor leer sabemos a donder ir :-):

(https://philippsp.shinyapps.io/BookRecommendation/)

###### Bibliografías:

- R for data sciences (https://r4ds.had.co.nz/)

- Advanced Machine Learning with R por Cory Lesmeister (Autor), Chinnamgari (Autor)
