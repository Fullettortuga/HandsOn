# Instruccions per contribuïr

* **Fork el repositori**: Ves al repositori del projecte al qual vols contribuir i fes clic al botó "Fork" a la part superior dreta de la pàgina. Això crearà una còpia del repositori al teu compte de GitHub.

* **Clona el teu repositori a la teva màquina**: Utilitza Git per clonar el teu repositori a la teva màquina local. Pots fer-ho amb la comanda següent, reemplaçant <URL_DEL_TEU_REPO> per l'URL del teu repositori:

```bash
git clone <URL_DEL_TEU_REPO>
```

* **Crea una branca (branch) nova**: Abans de fer canvis, crea una branca nova on faràs les teves modificacions. Això ajuda a mantenir les coses ordenades. Utilitza la comanda següent:

```bash
git checkout -b nom_de_la_branca
```

* Fes els canvis: Realitza les modificacions necessàries en els fitxers del projecte.

* Afegeix i commiteja els canvis: Utilitza els següents comandos per afegir els canvis i fer un commit:

```bash
git add .
git commit -m "Descripció dels canvis"
```

* Puja els canvis al teu repositori a GitHub: Pots fer-ho amb la comanda següent, reemplaçant <nom_de_la_branca> pel nom de la branca on has fet els canvis:

```bash
git push origin nom_de_la_branca
```

* **Crea una PR**: Vés al teu repositori a GitHub i selecciona la branca on has fet els canvis. Apareixerà un missatge destacat dient que has fet una nova branca. Fes clic a "Compare & pull request" per començar la PR.

* Proporciona una descripció detallada dels canvis que has fet. A més, pots afegir captures de pantalla o informació addicional per ajudar els revisors a entendre els teus canvis.

* **Envia la PR**: Un cop hagis omplert tota la informació, fes clic al botó "Create pull request" per enviar la PR al projecte original.
