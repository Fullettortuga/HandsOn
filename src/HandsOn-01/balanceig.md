### Balancejador de càrrega

El balanceig de càrrega és una tècnica utilitzada en l'administració de sistemes i xarxes per distribuir les peticions de servei o càrrega de treball entre múltiples servidors o recursos. Aquesta pràctica assegura que els recursos estiguin utilitzats de manera equitativa i eficient, evitant sobrecàrregues en un sol servidor i millorant la disponibilitat i el rendiment global.

* Un balancejador de càrrega actua com un intermediari entre els clients i els servidors d'aplicacions o recursos. Quan un client fa una petició, aquesta arriba primer al balancejador.

* El balancejador decideix a quin servidor enviar la petició, utilitzant diferents algorismes com Round Robin, Least Connections, o basant-se en mètriques de rendiment.

* La petició és redirigida al servidor seleccionat, que processa la sol·licitud i retorna la resposta al client.

### Avantatges del Balanceig de Càrrega

* **Tolèrancia a fallades**: La distribució de la càrrega entre múltiples servidors redueix el risc de caigudes i assegura una major disponibilitat dels serveis. En cas de fallada d'un servidor, el tràfic es redirigeix als servidors actius.
   
* **Optimització del Rendiment**_
El balanceig de càrrega permet a les organitzacions millorar el rendiment dels seus serveis web i aplicacions, ja que distribueix la càrrega entre múltiples servidors i redueix la càrrega en cada un d'ells.

* Escalabilitat: A mesura que augmenta la càrrega, es poden afegir nous servidors fàcilment per acomodar-la.

## Pas 1: Creació d'un Màquina Virtual

1. Crear una màquina virtual amb la plantilla de Rocky.
2. Configureu els següents recursos: **1GB de RAM, i 0,25 CPU**.
3. Assigneu la plantilla a la xarxa *Shared Network*.
4. Anomeneu la màquina **H01-BC-NGINX**.

## Pas 2: Instal·lació de NGINX

**[NGINX](https://www.nginx.com/)** és un servidor web i un servidor de proxy invers altament popular. Es caracteritza per la seva eficiència i escalabilitat, oferint un alt rendiment en l'atenció de peticions *HTTP* i *HTTPS*. Per obtenir més informació sobre la seva configuració podeu consultar [unit](https://unit.nginx.org/).


* Actualitza el sistema:

```sh
dnf update -y
```

* Instal·la VIM per editar els fitxers de configuració:

```sh
~ dnf install vim -y
```

* Instal·la NGINX:

```sh
~ dnf install nginx -y
```

* Inicia el servei NGINX i habilital
per a l'arrencada automàtica:

```sh
~ systemctl enable  --now nginx
```

Una característica de NGINX a més de reenviar les peticions als servidors individuals, NGINX també pot reenviar les peticions entrants a grups de servidors anomenats upstreams. Un upstream és un grup de servidors que conformen una única entitat lògica i es poden utilitzar com a destinació per a les peticions entrants en un listener o ruta.

Un upstream ha de definir un objecte servidors que llisti adreces i noms de servidors.

D'aquesta manera NGINX distribueix les peticions entre els servidors de l'upstream de forma circular, actuant com a balancejador de càrrega.

```bash
vim /etc/nginx/conf.d/upstreams.conf
# Definir els upstreams
# Modifica <IP1> i <IP2> amb les adreces IP de les instàncies de WordPress i <Port> amb el port en què s'escolta WordPress.
#Guarda i tanca l'arxiu. Per defecte, el port és el 80.
```

```nginx
upstream wordpress_servers {
    server <IP1>:<Port>;
    server <IP2>:<Port>;
}
```

Cada objecte de servidor pot establir un pes numèric per ajustar la quota de peticions que rep a través de l'upstream. 

```nginx
upstream wordpress_servers {
    server <IP1>:<Port>;
    server <IP2>:<Port>: {
                    "weight": 0.5
                };
}
```

En l'exemple anterior, el primer servidor IP1 rep el doble de peticions que el segon IP2.

## Pas 3: Configurem el servidor web

Edita el fitxer de configuració de NGINX per al lloc web:

```sh
vim /etc/nginx/conf.d/wordpress.conf
# copia el contingut
# modifica server amb la teva ip
# guarda i surt
```

```sh
server {
    listen 80;
    server_name server;

    location / {
        proxy_pass http://wordpress_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Analitzem la configuració:

* **server { ... }**: Aquesta és la directiva principal que defineix la configuració per a un servidor virtual.

* **listen 80**: Aquesta línia especifica que el servidor escoltarà les peticions al port 80.

* **server_name server**: Especifica el nom de domini associat amb aquest servidor.

* **location / { ... }**: Defineix la configuració per a la ubicació **/** (ruta principal).

* **proxy_pass http://wordpress_servers**: Aquesta línia és clau per al balanceig de càrrega. Quan es rep una sol·licitud a la ruta principal, aquesta és passada als servidors especificats a *wordpress_servers* mitjançant un **proxy**. Això permet reenviar la sol·licitud a una agrupació de servidors (upstream).

* **proxy_set_header Host $host**: Aquesta capçalera és important per informar als servidors  a quin domini  fa referència la sol·licitud.

* **proxy_set_header X-Real-IP $remote_addr**: El servidor web necessita conèixer l'adreça IP real de l'usuari quan es passa a través d'un proxy.

* **proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for**: Permet conservar les adreces IP originals dels clients.

Aquesta configuració permet a NGINX actuar com a servidor de balanceig de càrrega, redirigint les sol·licituds de la ruta principal a una agrupació de servidors especificada (**wordpress_servers**) per gestionar la càrrega i millorar el rendiment.

## Pas 4: Verificació de la Configuració 

NGINX ens proporciona una eina per verificar la sintaxi dels fitxers dconfiguració:

```sh
nginx -t
```

Finalment reiniciem NGINX per aplicar els canvis:

```sh
~ systemctl restart nginx
```