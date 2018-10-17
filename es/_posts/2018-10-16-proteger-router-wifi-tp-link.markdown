---
published: true
title: Proteger Router WIFI TP Link
layout: post
---
Los ataques a los modem routers que las proveedoras de servicio colocan en muchos casos estan expuestos a ataques. Son el eslabon debil donde logran escalar a todos los dispositivos conectados (telefonos, computadoras, tablets, camaras ip, televisiones, etc). 

Esto siempre sucedio, pero con la popularizacion de la compra online, y el agregado de mineria de criptomonedas a las herramientas de los bot malignos, los analisis y ataques se han industrializado, donde se intenta no ser detectados y cada nodo infectado intenta expandiendirse por si mismo. 

https://www.darkreading.com/attacks-breaches/100000-plus-home-routers-hijacked-in-campaign-to-steal-banking-credentials/d/d-id/1332946

https://www.scmagazine.com/home/security-news/ghostdns-hijacking-campaign-steps-up-attacks-on-brazilians-100k-devices-compromised/

Es importante por lo tanto comenzar por establecer una contraseña segura, pero como paso adicional le sigue verificar cuan expuesto esta el router (y por lo tanto nuestros datos) hacia internet. 

Para lograr esto, primero es necesario conocer la IP externa que nos corresponde.  En la interfase de administracion del router, buscar la sección de status y donde diga WAN va a mencionar una IP ( que no comienza con 10, 172.16 o 192.168).

Para realizar el escaneo de los puertos en Linux esto es relativamente simple con nmap. Por ejemplo, si nuestra ip fuera 45.33.32.156 (scanme.nmap.org)

```b
nmap -p0- -v -A -T4 45.33.32.156
```

El resultado de puertos puede incluir alguno que esta disponible solo en la red local, por lo tanto es necesario verificar cada uno de ellos a través de un servicio online, como por ejemplo http://www.t1shopper.com/tools/port-scan/

Servicios online similares pueden encontrarse con "port scan online" en google. 

Al confirmar los puertos disponibles, ya los pasos son muy especificos para cada router. Para el caso del TP-Link TD-W8901N se configura desde Access Management -> ACL, donde si esta desactivado permite trafico a la interfase de administracion del router desde la red interna como desde internet. Al activar ACL, todo acceso será bloqueado por lo que hay que agregar una regla para autorizar a la red interna "Interface LAN".

![1539710479563](/images/1539710479563.png)

Con este paso pueden haberse bloqueado varios puertos (por ejemplo el 80). Para bloquear el tráfico external hacia el resto de los puertos detectados, en este modelo de router se establecen filtros (Access Management -> Filter). Es importante notar la opcion para rule unmatched, con valores "Forward" o "Next". Con Next, al fallar la regla continua con la siguiente. 

![1539710779454](/images/1539710779454.png)

Mayor información del router puede encontrarse en https://www.tp-link.com/ar/download/TD-W8901N.html
