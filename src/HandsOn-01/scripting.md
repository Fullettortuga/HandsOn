# Scripting

Aquesta secció té com a objectiu demostrar com **NGINX** reparteix les peticions entre els servidors configurats. Farem servir un script de *Bash* per generar peticions als servidors balancejats i mostrarem en temps real com es distribueix la càrrega entre ells.

## Pas 1: Creació del Script

Crearem un script en el llenguatge **Bash** que automatitzarà les peticions. L'objectiu és registrar en el *log* del nostre balancejador la destinació de les peticions i, posteriorment, analitzar aquest fitxer mitjançant **Bash**.

Per tal de registrar aquesta informació en el *log*, implementarem un fitxer de *log* personalitzat per evitar interferir amb el *log* habitual del servidor.

```sh
vim /etc/nginx/nginx.conf
# Afegir aquest contingut dins de les claus http
```

Dins del fitxer de configuració **nginx.conf**, afegirem el següent contingut dins de la secció **http**:

```python
log_format custom_log '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for" '
                          'Server: $upstream_addr';
access_log /var/log/nginx/custom_access.log custom_log;
```

Aquest codi defineix un nou format de log anomenat **custom_log**. Aquest format inclou diversos elements com l'adreça remota, l'usuari remot, la data i l'hora de la sol·licitud, el tipus de petició, l'estat de la resposta, a més del servidor al qual s'està redirigint la petició.

Un cop realitzat aquest pas, reiniciarem el servidor per aplicar els canvis de configuració.

```sh
systemctl restart nginx
```

**cURL** (anomenat com "Client for URLs") és una eina de línia de comandes amplament utilitzada per fer peticions a diferents protocols de xarxa, com ara HTTP, HTTPS, FTP, SCP, SFTP i molts altres. Està dissenyat per recuperar o enviar dades a un servidor amb facilitat i eficàcia.

Per exemple:

```sh
curl https://www.google.com
# Per guardar la resposta en un fitxer
curl -o google.html https://www.google.com
```

Aquesta comanda farà una sol·licitud **HTTP GET** a la pàgina principal de Google i mostrarà la resposta a la consola. Recollirà la pàgina HTML i la mostrarà directament en la terminal.

Ara utilitzarem l'eina per comprovar que el fitxer de *log* funciona correctament.

```sh
curl -s -o /dev/null  192.168.101.106/wordpress
cat /var/log/nginx/custom_access.log
```

Observeu que:
* **-s**: Aquesta opció indica a curl que funcioni en mode silenciós (silent). No mostra la sortida de l'operació a la pantalla. És útil si només vols realitzar l'acció sense mostrar cap informació a la sortida estàndard.

* **-o /dev/null**: Aquesta opció especifica que la sortida de la petició realitzada amb cURL s'ha de redirigir a **/dev/null**. **/dev/null** és un dispositiu especial de sistema a la majoria de sistemes UNIX-like que descarta tot el que s'hi escriu. Bàsicament, aquesta opció suprimeix la sortida, la qual cosa significa que no veuràs res a la pantalla.

**Nota**: 192.168.101.106 és la direcció IP del balacejador.

Si tot ha funcionat ```cat``` us mostrarà la següent informació a la pantalla:

192.168.101.106 - - [19/Sep/2023:08:38:35 +0000] "GET /wordpress HTTP/1.1" 301 241 "-" "curl/7.61.1" "-" Server: 192.168.101.105:80

Aquesta línia indica que el balancejador amb **IP: 192.168.101.106** ha reenviat la petició al servidor web amb **IP: 192.168.101.105** al port 80.

Així, podem utilitzar l'eina **cURL** per implementar un script que faci 100 peticions a una IP determinada (el nostre balancejador de càrrega) i que compti quantes són gestionades pel servidor amb *IP1* i quantes pel servidor amb *IP2*.

```bash
#!/bin/bash

# IP del balancejador
BALANCER_URL="http://192.168.101.106/wordpress"
# IPs dels servidors
SERVER1="192.168.101.77"
SERVER2="192.168.101.105"
# Número de peticions a fer
NUM_REQUESTS=100
# Fitxer de log personalitzat
LOG_FILE="/var/log/nginx/custom_access.log"

echo "Sending $NUM_REQUESTS requests to $BALANCER_URL..."

for ((i=1; i<=$NUM_REQUESTS; i++)); do
  curl -s -o /dev/null $BALANCER_URL
done

# Comptem les peticions per a cada servidor
SERVER1_COUNT=$(grep -c "$SERVER1" "$LOG_FILE")
SERVER2_COUNT=$(grep -c "$SERVER2" "$LOG_FILE")

# Mostrem el recompte
echo "Peticions a $SERVER1: $SERVER1_COUNT"
echo "Peticions a $SERVER2: $SERVER2_COUNT"

# Esborrem el contingut del fitxer de log
truncate -s 0 $LOG_FILE

```

**Explicació detallada**:

1. **#!/bin/bash**: Indica que l'script s'executarà amb l'interprete de Bash (/bin/bash).
2. Variables:
     * *BALANCER_URL*: Conté l'adreça URL del balancejador de càrrega.
     * *SERVER1* i *SERVER2*: Contenen les adreces IP dels servidors.
     * *NUM_REQUESTS*: És el nombre de peticions que volem fer.
     * *LOG_FILE*: Ruta del fitxer de log personalitzat.
3. **echo "Sending $NUM_REQUESTS requests to $BALANCER_URL...":** Utilitza echo per mostrar un missatge a la consola que indica quantes peticions s'enviaran al balancejador (*com printf a c*).
4. Implementa un bucle (for) de NUM_REQUESTS. A cada iteració, s'utilitza l'eina curl per enviar una petició al nostre servidor.
5. Comptatge de peticions:
     * **grep -c "$SERVER1" "$LOG_FILE"**: Compta quantes vegades apareix SERVER1 en el fitxer de log ($LOG_FILE).
     * **grep -c "$SERVER2" "$LOG_FILE"**: Compta quantes vegades apareix SERVER2 en el fitxer de log ($LOG_FILE).
6. Esborrar contingut del fitxer de log:
    * **truncate -s 0 $LOG_FILE**: Esborra el contingut del fitxer de log ($LOG_FILE) sense eliminar-lo, simplement estalviant espai i mantenint el fitxer.

Per a executar-lo, necessitem:

* Donar permissos d'execució al script:
  
  ```sh
  chmod +x check.sh
  ```

* Executa script:

  ```sh
  ./check.sh
  ``` 

Això ens mostrarà com 50 peticions són dirigides a cada servidor de manera alternada, una per una.