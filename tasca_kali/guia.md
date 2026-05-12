# Pràctica d’exploració de xarxa amb Kali Linux

## Configuració inicial

La màquina virtual Kali Linux funciona amb l’adaptador de xarxa en mode pont i obté l’adreça IP per DHCP. En el meu cas, la IP assignada és `192.168.2.1`.

## Exploració amb Netdiscover

### Consulta de l’ajuda

Abans d’utilitzar l’eina, es mostra la pantalla principal busquem l'eina per útilitzar-la.

![netdiscover](/tasca_kali/img_kali/captura1.png)

![netdiscover -h](/tasca_kali/img_kali/captura2.png)

### Mode actiu

S’executa un escaneig actiu sobre el rang `192.168.1.0/24` amb la comanda:

```bash
sudo netdiscover -r 192.168.1.0/24
```

![Comanda netdiscover actiu](/tasca_kali/img_kali/captura3.png)

El resultat mostra 22 hosts detectats, amb diferents rangs d’IP i fabricants:

![Resultat netdiscover actiu](/tasca_kali/img_kali/captura4.png)

### Mode passiu

S’executa el mode passiu (només escolta, no envia paquets) amb la comanda:

```bash
sudo netdiscover -p
```

![Comanda netdiscover passiu](/tasca_kali/img_kali/captura5.png)

El resultat mostra 17 hosts detectats:

![Resultat netdiscover passiu](/tasca_kali/img_kali/captura6.png)

### Diferències entre mode actiu i passiu

- El **mode actiu** genera trànsit ARP i obté més respostes (22 hosts). És més ràpid per fer un inventari complet, però deixa rastre.
- El **mode passiu** no envia cap paquet, només captura les trames ARP existents. Detecta menys equips (17 hosts) però és completament furtiu, ideal per a una fase de reconeixement sense alertar sistemes IDS.

## Exploració amb Nmap

### Consulta de l’ajuda de Nmap (opcions avançades)

Abans d’utilitzar l’eina, es mostra la pantalla principal busquem l'eina per útilitzar-la.

Es mostren les opcions generals de Nmap i les relatives a evasió i detecció de sistema operatiu i serveis.

![Ajuda nmap](/tasca_kali/img_kali/captura7.png)

![Opcions nmap evasió i sortida](/tasca_kali/img_kali/captura8.png)

### Escaneig de descobriment d’equips (ping sweep)

S’utilitza l’opció `-sn` per esbrinar quins equips estan actius a la meva xarxa local `192.168.2.0/24`.

```bash
sudo nmap -sn 192.168.2.0/24
```

![Resultat nmap -sn](/tasca_kali/img_kali/captura9.png)

**Resultat:** es detecten 7 hosts actius. Entre ells, el meu Kali (`192.168.2.1`) i altres màquines virtuals amb adreces `192.168.2.4`, `.8`, `.12`, `.20`, `.253` i `.254`.

### Escaneig detallat d’un equip (sistema operatiu i versió de serveis)

S’aplica l’escaneig `-O -sV` a l’equip `192.168.2.8`.

```bash
sudo nmap -O -sV 192.168.2.8
```

![Escaneig nmap 192.168.2.8](/tasca_kali/img_kali/captura10.png)

**Resultat:** tots els 1000 ports escanejats apareixen tancats. No es pot determinar el sistema operatiu amb precisió (massa coincidències). La latència és de 0.0016 segons i la distància 1 salt.

### Escaneig detallat d’un altre equip

S’aplica el mateix escaneig a l’equip `192.168.2.20`.

```bash
sudo nmap -O -sV 192.168.2.20
```

![Escaneig nmap 192.168.2.20](/tasca_kali/img_kali/captura11.png)

**Resultat:** idèntic a l’anterior: tots els ports tancats i detecció d’OS no concloent.

## Conclusió

L’exploració amb **netdiscover** ha permès inventariar els dispositius de la xarxa local, destacant que el mode actiu detecta més equips però genera trànsit, mentre que el mode passiu és més furtiu. Amb **nmap** s’ha confirmat la llista d’equips actius mitjançant `-sn`, i els escaneigs `-O -sV` han mostrat que els equips `192.168.2.8` i `192.168.2.20` no tenen ports oberts. Aquestes tècniques són fonamentals en ciberseguretat tant per a tasques ofensives (reconeixement inicial) com defensives (auditoria de la pròpia xarxa).

