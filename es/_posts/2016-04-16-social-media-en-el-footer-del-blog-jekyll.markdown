---
published: true
title: Social media en el footer del Blog Jekyll
summary: Agregar íconos SVG para Linkedin y GitHub
layout: post
---
### Te muestro como agregar al footer de tu blog links a tus sitios de social media. Los ejemplos son sólamente para GitHub y Linkedin... pero usando SVG!!!! 

### En esta continuación de [GitHub pages desde linea de comando][2] vamos a agregar iconos y links al footer de social media


1. Especificar tus usuarios a _config.yml, por ejemplo los mios son:
{% highlight text %}
github_username:  emmanuel-galindo
linkedin_username: emmanuelgalindo
{% endhighlight %}

2. Dimensionar los SVG en tu CSS customizado (css/custom.css en mi caso), y otra clase mas que uso en mi ejemplo asi no se ve tan feo

{% highlight text %}
/** Icons */
.icon > svg {display: inline-block;width: 16px;height: 16px;vertical-align: middle;}
.icon > svg path { fill: #828282; }

.social-media-list {list-style: none;margin-left: 0;}
{% endhighlight %}

3. Modificar _includes/footer.html. Por ejemplo, el que se usa en este blog actualmente es
<https://raw.githubusercontent.com/emmanuel-galindo/emmanuel-galindo.github.io/master/_includes/footer.html>

Otra alternativa a los SVG, es usar la libreria de [Font Awesome][1]

[1]: http://fontawesome.io/examples/
[2]: http://emmanuel-galindo.github.io/2016/04/15/github-pages-desde-linea-de-comando.html
