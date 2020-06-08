---
title: "Dashbord alojado en Heroku y desarrollado con Python-Plotly."
categories:
  - Python
  - visualization
tags:
  - visualization
  - Python
date: 2020-02-05T14:37:42
---

# Dashbord alojado en Heroku y desarrollado con Python-Plotly.

Me cuestionaba mucho, ¿Que tan dificil es realizar un Dashbord de forma gratuita y libre?, ¿Donde lo alojaría?, ¿Cual sería la Arquitectura?.

En el siguiente link puedes ver el resultado: (https://unam-derecho-animal.herokuapp.com/)

Alojado en Heroku: (https://www.heroku.com/)

Y hecho con Plotly: (https://plotly.com/)

### ¿Que es Heroku?

Si en tu día a día te encuentras desarrollando apps, debes ser consciente que el impacto inicia cuando comienzas a tener usuarios, por esto es importante lanzar tu aplicación sin tener complicaciones de infraestructura, administrar servidores, tus bases de datos y la seguridad que estos deben de tener entre otras cosas.

En el mercado existen una serie de alternativas conocidas como PaaS (Platform as a Service) o “Plataformas como Servicios” que, además de ser la evolución de las IaaS (Infraestructura como Servicio), como EC2 de Amazon donde te dan un servidor y tu te encargas de provisionarlos y administrarlos con PaaS, te puedes olvida de todas estas cuestiones de administración, pues utilizas directamente una plataforma que lo hace por tí.

Heroku es uno de los PaaS más utilizados en la actualidad en entornos empresariales por su fuerte enfoque en resolver el despliegue de una aplicación. Ademas te permite manejar los servidores y sus configuraciones, escalamiento y la administración. A Heroku solo le dices qué lenguaje de backend estás utilizando o qué base de datos vas a utilizar y te preocupas únicamente por el desarrollo de tu aplicación.

### ¿Quién utiliza Heroku?

Heroku tiene su clientela bien definida: empresas que quieren dejar de preocuparse por cuestiones de infraestructura y sólo enfocarse en el desarrollo. Por lo general estas suelen ser empresas grandes o startups que prefieren no invertir en un equipo de operaciones cuando están en una etapa temprana, y su prioridad debe ser hacer un producto que las personas quieran.

Heroku tiene dos tiers, o niveles, para personas interesadas en aprender: una versión gratuita similar a la de now.sh, que entra en modo “sleep” cada 30 minutos sin tráfico, y otra de 7 USD que compite con el servicio básico de 5 USD al mes que ofrece Digital Ocean, pero agregar las ventajas de que nuestros servidores sean administrados por nosotros.

### ¿Que es Plotly?

Plotly es una empresa de informática técnica con sede en Montreal, Quebec, que desarrolla herramientas de análisis y visualización de datos en línea.
Plotly proporciona herramientas de gráficos, análisis y estadísticas en línea para individuos y colaboración, así como bibliotecas de gráficos científicos para Python, R, MATLAB, Perl, Julia, Arduino y REST.

##### Todo el código lo puedes encontrar en: https://github.com/Corderodedios182/Dashbord_UNAM_DerechoAnimal

### Ventajas:

    -Código libre.
    -Gran variedad de gráficas que dificilmente se pueden crear en herramientas de visualización.
    -Versatilidad y alta escalabilidad.
    -Heroku es muy amigable para desplegar las aplicaciones.

### Desventajas:

    -El diseño con Plotly es laborioso.
    -Buenos conocimientos de HTML, CSS y visualización de datos.
    -Con la versión gratis de Heroku el Dashbord en ocasiones es lento.

### Conclusión:

Si quieres mostrar gráficas impresionantes del rendimiento de tus algoritmos, Plotly es una excelente opción.

Para realizar gráficas sencillas y un Dashbord más empresarial preferiría DataStudio ó PowerBI.
