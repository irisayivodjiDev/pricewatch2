# SOBRIETE.md · PriceWatch

Groupe :
Machine de mesure (OS, processeur, RAM allouée à Docker) : Windows 11 Pro, Docker Desktop (WSL2), processeur : à compléter, 7,6 Go alloués à Docker

## Séance 2 · Le job sous contrainte

Cible : 256 Mo de RAM, 1 CPU, moins de 30 s, résultat identique.

| Étape | Limites (RAM · CPU) | Pic de RAM (Mo) | Temps réel (s) | Temps CPU (s) | Ratio CPU / réel | Code de sortie | OOMKilled |
|---|---|---|---|---|---|---|---|
| 01. Ligne de base | aucune | 1567 | 17.1 | 16.9 | 0.99 | 0 | false |
| 02a. Sous contrainte | 256m · 1 | — (tué) | — | — | — | 137 | true |
| 02b. CPU réduit | aucune · 0.5 | 1567 | 31.9 | 16.8 | 0.53 | 0 | false |
| 03. Optimisation RAM | 256m · 1 | | | | | | |
| 04. Optimisation CPU | 256m · 1 | | | | | | |
| 05. Dimensionné | ? · ? | | | | | | |
| Bonus | | | | | | | |

### Ce que nous avons changé, et pourquoi (une ligne par changement)

- **01. Ligne de base** : version de départ, sans limite. 8400 lignes écrites, copiées dans `data/out/reference.csv`. Le job utilise un cœur entier (ratio 0.99) et monte à 1567 Mo, soit 6 fois la cible de 256 Mo.
  - Mesures refaites après avoir arrêté mes autres conteneurs. Une première série, prise pendant qu'ils tournaient, donnait 22,0 s de temps réel, ce qui montre que les autres conteneurs faussaient le temps.
- **02a. 256 Mo sans swap, 1 CPU** : le conteneur est tué sans aucun message. `docker inspect` donne le code 137 (128 + 9 = SIGKILL) et `OOMKilled=true` : le noyau l'a tué parce qu'il dépassait sa limite mémoire.
- **02b. Sans limite RAM, 0.5 CPU** : le job finit avec le même résultat, mais en 31,9 s au lieu de 17,1 s, presque deux fois plus long. Le temps CPU ne change pas (16,8 s contre 16,9 s) : le job fait le même travail, mais le ratio de 0.53 montre qu'il n'a eu qu'un demi-cœur (throttling).
  - La RAM est une ressource **incompressible** : quand elle manque, le conteneur est tué. Le CPU est **compressible** : quand il manque, le conteneur est seulement ralenti.
  - Ici, la ressource limitante est donc la **mémoire**.

### Dimensionnement retenu

Commande de lancement finale :

    docker run ...

Justification (pic mesuré, marge choisie) :

### Ce que nous en retenons (3 lignes maximum)

-
