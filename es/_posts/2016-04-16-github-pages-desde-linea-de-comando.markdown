---
published: true
title: GitHub pages desde linea de comando
summary: Ya con tantos cambios, lo mejor es desarrollar localmente y publicar los cambios usando Git. 
layout: post
---
### En esta continuación de [Blog en GitHub Pages con Jekyll, parte 2][1] ya tratamos de realizar lo mas posible desde linea de comando

### Los ejemplos son para Linux Debian, pero los pasos son los mismos ###


1. Instalar git (como root): 

    `# aptitude install git`

2. Ir a tu repositorio de la pagina en GitHub y copiar la url (aparece entre HTTPS y el botón Download ZIP)
3. Localmente, ir al directorio donde querés trabajar
4. Clonar el repositorio (como usuario normal) usando la url del paso 2) 

    `$ git clone https://github.com/emmanuel-galindo/emmanuel-galindo.github.io.git`

5. Modificar algun archivo como prueba
6. Ejecutar: 
    
    `$ git commit -a -m "Commit"; git push origin master`

7. Va a solicitarte usuario y password. Para evitar cargarlo en cada push, se lo puede cachear con dos comandos:

    `$ git config --global credential.helper cache`

    `$ git config --global credential.helper 'cache --timeout=3600'`


8. En el caso de que agregues un nuevo post desde la interface de tinypress, podes actualizar tu repo local con:

    *Quizas lo mejor sea hacer un backup...*

    `$ git pull`

[1]: http://emmanuel-galindo.github.io/2016/04/15/github-pages-jekyll-y-tinypress.html

