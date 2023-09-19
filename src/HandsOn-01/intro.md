# HandsOn 01: Balancejant la càrrega

## Problema

Els clients de la empresa tradicional (*HandsOn00*) han reportat que la seva web presenta problemes de rendiment, com una càrrega lenta o fins i tot indisponibilitat en algunes ocasions. Això suggereix que el servidor web actual no està gestionant adequadament la càrrega de tràfic en línia. Amb l'increment de la presència en línia de l'empresa, és vital abordar aquest problema de rendiment per garantir una millor experiència per als usuaris i mantenir la reputació de la marca.

## Solució Proposada

Per millorar el rendiment i garantir la disponibilitat del lloc web, es proposa una solució basada en **l'ús de rèpliques del servidor web** i la **implementació d'un balancejador de càrrega per repartir les peticions**. Aquesta solució té diversos avantatges que ajudaran a abordar el problema actual:

### Afegir Rèpliques del Servidor Web

Es recomana crear rèpliques del servidor web existent per gestionar millor la càrrega de tràfic. Amb més servidors, es pot distribuir la càrrega de manera més equitativa i respondre ràpidament a les peticions dels usuaris.

### Balanceig de Càrrega

Implementar un balancejador de càrrega que distribuirà les peticions entre les rèpliques del servidor web. Això optimitzarà l'ús dels recursos i millorarà el rendiment global del lloc web.