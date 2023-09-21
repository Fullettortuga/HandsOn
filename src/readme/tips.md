# Consells

## Consell 1

Podeu millorar la gestió dels servidors amb la comanda ```hostnamectl set-hostname nom_servidor``` , on *nom_servidor* hauria de ser un nom descriptiu com ara **DB_SERVER** si és el servidor de la base de dades. Això ens permet mantenir la claredat i l'ordre en entorns amb moltes terminals i servidors diferents, assegurant que sabem en tot moment a quin servidor estem treballant.

## Consell 2

És essencial ser eficients en la gestió. Si la **màquina virtual** amb *Ansible* ha de servir per automatitzar totes les màquines que crearem, no és qüestió de  copiar i enganxar la clau pública a totes les noves màquines (com vam fer a classe). Cal ser astuts. Cal afegir la clau pública de la màquina Ansible a **auth** de l'OpenNebula, juntament amb la clau del nostre pc i així totes les futures màquines que crearem ansible s'hi podrà connectar.