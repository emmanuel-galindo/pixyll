---
published: true
title: Blog en GitHub Pages con Jekyll, parte 2 
summary: Vamos a agregarle comentarios, cambiar el formato, y más!
layout: post
---

### En esta continuación de [Blog en GitHub, gratis y rápido con TinyPress][3] te cuento como agregarle comentarios, hacerlo mas ancho, etc

### Agregar comentarios
1. Generar cuenta en Disqus
+ Lo único que necesitás es asignar un short name y acordartelo
+ Te va a dejar generar un código HTML universal para copiar en el blog, únicamente actualizando el shortname que seleccionaste antes.
2. Agregá lo siguiente al archivo "_layout/default.html", aproximadamente en la linea 41, abajo de `{ content }` :
{% highlight text %}
{% raw %}
{% include comments.html %}
{% endraw %}
{% endhighlight %}

### Hacer que el blog sea mas ancho

1. Agregar un archivo custom.css en el dir "_css"
2. Agregar este contenido (1 rem aprox 16px, 16x62=992px)

{% highlight css %}
@charset "UTF-8";

.measure {
  max-width: 62rem;
}
{% endhighlight %}

### Solo mostrar barra de desplazamiento horizontal (scrollbar) cuando sea necesario
En el mismo custom.css:
{% highlight css %}
pre {
 overflow-x: auto;
}
{% endhighlight %}

### Modificar el color de la caja para codigo fuente###
Para cambiar el color de fondo de los bloques de codigo, por ej en [Yarssr is not loading the feed, how to fix it][2]:

1. Abrir default.html en "_layout"
2. Modificar la línea:

{% highlight html %}
    <link rel="stylesheet" href="{{ "/css/solarized-dark.css" | prepend: site.baseurl }}" type="text/css">
{% endhighlight %}
por
{% highlight html %}
    <link rel="stylesheet" href="{{ "/css/solarized-light.css" | prepend: site.baseurl }}" type="text/css">
{% endhighlight %}

O viceversa...



[1]: https://raw.githubusercontent.com/emmanuel-galindo/emmanuel-galindo.github.io/master/_includes/comments.html
[2]: http://emmanuel-galindo.github.io/2016/04/14/yarssr-is-not-loading-the-feed-how-to-fix-it.html]
[3]: http://emmanuel-galindo.github.io/2016/04/15/blog-en-github-gratis-y-r-pido-con-tinypress.html
