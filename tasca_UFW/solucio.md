# Solució pràctica UFW - Tasca Firewall

A continuació es detalla pas a pas la realització de les activitats de la pràctica de tallafocs amb UFW a Zorin. S'inclouen captures de pantalla i explicacions per a cada punt, seguint els criteris d'avaluació.

## Preparació de l’entorn

### Configuració de xarxa a VirtualBox

La màquina virtual disposa de dos adaptadors de xarxa:

- **Adaptador 1 (NAT)**: per a l'accés a Internet.
- **Adaptador 2 (Host-Only)**: per a la comunicació amb l'amfitrió.

![Configuració adaptador NAT](/tasca_UFW/img_ufw/captura1.png)

*Captura 1: Adaptador 1 configurat en mode NAT.*

![Configuració adaptador Host-Only](/tasca_UFW/img_ufw/captura2.png)

*Captura 2: Adaptador 2 configurat en mode Host-Only.*

### Comprovació d’interfícies i IPs

Executem `ip a` per veure les adreces assignades.

![Resultat de ip a](/tasca_UFW/img_ufw/captura3.png)

*Captura 3: La interfície `enp0s8` té IP `192.168.56.131` (xarxa host-only). L’amfitrió normalment és `192.168.56.1`.*

### Actualització del sistema i instal·lació de serveis

Actualitzem els repositoris i instal·lem SSH i Nginx.

![apt update i upgrade](/tasca_UFW/img_ufw/captura4.png)

*Captura 4: Actualització del sistema.*

### Verificació dels serveis SSH i Nginx

Comprovem que ambdós serveis estan actius.

![Estat del servei SSH](/tasca_UFW/img_ufw/captura5.png)

*Captura 5: `ssh.service` està actiu (running).*

![Estat del servei Nginx](/tasca_UFW/img_ufw/captura6.png)

*Captura 6: `nginx.service` està actiu.*

## Activitat 1: Comprovar l’estat del firewall i habilitar-lo

Primer comprovem si UFW està instal·lat i el seu estat.

![which ufw i ufw status](/tasca_UFW/img_ufw/captura7.png)

*Captura 7: UFW és a `/usr/sbin/ufw`, inicialment `inactivo`. Després de `sudo ufw enable` passa a `activo`.*

## Activitat 2: Mostrar les regles definides i explicar les regles per defecte

Executem `ufw status verbose` per veure la política per defecte.

![ufw status verbose](/tasca_UFW/img_ufw/captura8.png)

*Captura 8: `Predeterminado: deny (entrantes), allow (salientes), disabled (enrutados)`. Això significa:*
- **deny entrantes**: tot trànsit que intenti entrar a la màquina és denegat tret que hi hagi una regla `allow` explícita.
- **allow salientes**: tot trànsit que surt de la màquina està permès per defecte.
- **disabled enrutados**: no es fa forwarding entre interfícies.

Comprovem si existeix alguna regla personalitzada.

![ufw status numbered (buit)](/tasca_UFW/img_ufw/captura9.png)

*Captura 9: No hi ha regles definides, només el comportament per defecte.*

## Activitat 3: Comprovar la regla `deny` per defecte al trànsit d’entrada (SSH)

Des de l’amfitrió (PowerShell) intentem connectar per SSH a la IP `192.168.56.131`.

![Intent de SSH fallit](/tasca_UFW/img_ufw/captura10.png)

*Captura 10: `Connection timed out`. La política `deny incoming` bloqueja el paquet de SYN del SSH, per tant no es pot establir la connexió.*

## Activitat 4: Aplicar regla per defecte `deny` al trànsit de sortida i provar ping a Google

Canviem la política de sortida a `deny`.

![default deny outgoing](/tasca_UFW/img_ufw/captura11.png)

*Captura 11: `ufw default deny outgoing` canvia la política. Ara `Predeterminado: deny (entrantes), deny (salientes)`.*

Provem ping a google.com.

![ping fallit per deny outgoing](/tasca_UFW/img_ufw/captura12.png)

*Captura 12: `ping: google.com: Fallo temporal en la resolución del nombre`. En denegar tot el trànsit sortint, tampoc funcionen les consultes DNS ni els paquets ICMP.*

## Activitat 5: Aplicar regla per defecte `allow` al trànsit de sortida i comprovar ping

Revertim la política de sortida a `allow` i tornem a fer ping.

![default allow outgoing i ping correcte](/tasca_UFW/img_ufw/captura13.png)

*Captura 13: `ufw default allow outgoing` restaura la política inicial. El ping a google.com ara funciona correctament (0% packet loss).*

## Activitat 6: Crear una regla per prohibir el trànsit cap a `capgros.com`

Primer obtenim l’adreça IP del lloc web. Utilitzem `host` (l’enunciat deia `capgros.com`, però a les captures es fa servir `capgros.elnacional.cat`).

![host capgros.elnacional.cat](/tasca_UFW/img_ufw/captura14.png)

*Captura 14: Les IPs són `104.18.9.104` i `104.18.8.104`.*

Comprovem que tenim connectivitat abans de bloquejar.

![ping abans de bloquejar](/tasca_UFW/img_ufw/captura15.png)

*Captura 15: Ping funcionant a `104.18.8.104`.*

Afegim regles `deny out` per a cada IP.

![deny out a 104.18.8.104](/tasca_UFW/img_ufw/captura16.png)
![deny out a 104.18.9.104](/tasca_UFW/img_ufw/captura17.png)

*Captures 16 i 17: S’afegeixen les regles de denegació de trànsit sortint cap a les dues IPs.*

Comprovem les regles creades.

![ufw status numbered amb regles deny](/tasca_UFW/img_ufw/captura18.png)

*Captura 18: Ara hi ha dues regles `DENY OUT` per a les IPs.*

Provem de fer ping de nou.

![ping fallit després de bloquejar](/tasca_UFW/img_ufw/captura20.png)

*Captura 20: Ping a `capgros.elnacional.cat` resol a `104.18.9.104` i el paquet es descarta, resultat en `100% packet loss`.*

## Activitat 7: Habilita el trànsit d’entrada pel servei nginx des de l’amfitrió (IP 192.168.56.1)

Afegim una regla que permeti connexions TCP al port 80 (HTTP) només des de l’IP de l’amfitrió.

![allow from 192.168.56.1 to port 80](/tasca_UFW/img_ufw/captura21.png)

*Captura 21: `ufw allow from 192.168.56.1 to any port 80 proto tcp`.*

Des de l’amfitrió obrim un navegador i accedim a `http://192.168.56.131`.

![navegador accedint](/tasca_UFW/img_ufw/captura22.png)
![pàgina Welcome to nginx](/tasca_UFW/img_ufw/captura23.png)

*Captures 22 i 23: El servidor nginx respon correctament mostrant la pàgina per defecte.*

## Activitat 8: Mostra el conjunt de regles definides

Finalment, visualitzem totes les regles actives numerades.

![ufw status numbered final](/tasca_UFW/img_ufw/captura24.png)

*Captura 24: Es mostren les dues regles de denegació de sortida cap a `capgros.elnacional.cat` i la regla d’entrada per a nginx des de l’amfitrió.*

## Conclusió

S’ha demostrat el funcionament d’UFW com a tallafocs d’equip, aplicant polítiques per defecte, regles específiques d’entrada i sortida, i verificant el comportament amb proves de connectivitat (SSH, ping, HTTP). Tots els criteris d’avaluació s’han cobert:

- Es mostren i expliquen les regles per defecte.
- Es comprova la denegació d’entrada (SSH) i de sortida (ping a Google).
- S’aplica i comprova el canvi de polítiques de sortida.
- Es bloqueja l’accés a un lloc web específic.
- S’habilita l’accés al servidor web des de l’amfitrió.
- Es mostren finalment totes les regles creades.

### Autor
Guillem Barjuan
