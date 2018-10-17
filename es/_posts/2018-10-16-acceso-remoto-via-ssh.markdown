---
published: true
title: Acceso remoto via SSH
layout: post
---
Una forma segura de acceder a nuestros recursos cuando no estamos en la red local es usando SSH. Esto incluye la ejecución de aplicaciones desde dentro de la red local (p2p, movie and music players, etc).

Para instalar el server sshd, en linux:

```bash
sudo aptitude install openssh-server
```



La forma segura de configurar esta comunicación es realizar una regla de port forwarding con un puerto no  comun (i.e. 46022). Luego se deberá habilitar ese puerto en el modem router hogareño. 

Una vez probada la comunicación usando usuario y password local, el siguiente paso es desalentar los ataques por fuerza bruta. Para ello, hay que agregar una directiva en la configuración de SSH para que sólo permita autenticarse con usuario y password en la red local. Siendo para los accesos de internet solo posible usando clave privada. 

```
PasswordAuthentication no
ChallengeResponseAuthentication no

Match Address 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
    PasswordAuthentication yes
```

Notar que el bloque Match debe ser agregado al final del archivo, de otra manera todas las directivas siguientes van a ser matcheadas hasta el proximo bloque Match. Agregar en el lugar incorrecto del archivo esta instuccion puede ocasionar imposibilidad para conectarse. 



Para generar la clave privada, ejecutar

```
ssh-keygen -t rsa -b 4096
```

Al consultar por una clave privada, es posible dejarlo vacio pero se recomienda establecer una clave segura. Esta clave sera solicitada unicamente al momento de cargar la clave privada en el cliente. 

Si el resto de las opciones se dejan como estaban, en ~/.ssh/ estarán la clave privada y la publica en formato PEM y openssh respectivamente.

La publica debe cargarse al archivo ~/.ssh/authorized_keys

```
cat id.pub >> ~/.ssh/authorized_keys
```

La clave privada hay que instalarla en los clientes. 

Para el caso de putty en windows, hay que convertir la clave a Putty Private Key usando la utilidad putty-gen, e importando la clave PEM. Luego salvar como ppk y ya será posible usarla desde putty o winscp. 