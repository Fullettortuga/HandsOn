# Reptes

* Configureu el servidor activant el SELinux.

* Garantim que es comprovi si la variable **TARGET_IP** té el port de WordPress (port 80) obert i accessible. En cas contrari, l'script ha d'aturar-se i indicar que el port necessari no està disponible.

* Modifica el script **check.sh** per aconseguir estadístiques en temps real sobre les peticions als servidors. L'objectiu és mostrar una sortida semblant a:
  
    ```sh
    Estadístiques en temps real:
    ---------------------------
    Peticions a 192.168.101.77: x
    Peticions a 192.168.101.105: y
    ---------------------------
    ```

    on **x i y** s'incrementin dinàmicament en funció del fitxer de log que conté les adreces IP de les peticions. Assegura't que aquestes estadístiques s'actualitzin després de cada petició.

* Modificació de la configuració de **NGINX** per facilitar l'afegida i eliminació de servidors de manera dinàmica, sense la necessitat de reiniciar NGINX. Això millorarà la flexibilitat de l'entorn de balanceig de càrrega.

* Optimització de la configuració de NGINX per a la mitigació d'atacs de denegació de servei distribuïts (DDoS). Aquesta configuració especial ajudarà a protegir el sistema davant de possibles atacs massius.

* Desenvolupament i configuració d'un balancejador de càrrega que ofereix persistència de sessions. Aquesta característica assegura que un mateix client sempre sigui dirigit al mateix servidor, essencial per a aplicacions que requereixen mantenir l'estat de la sessió, com ara cistelles de compra en línia en la nostra botiga.

* Automatització del HandsOn01 mitjançant Ansible.