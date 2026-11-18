# SOBRIETE.md · PriceWatch

Groupe : IRIS AYIVODJI
Machine de mesure (OS, processeur, RAM allouée à Docker) : Windows 11 Pro, Docker Desktop (WSL2), processeur : ntel Core i7-1185G7 @ 3.00 GHz (16 Go RAM), 7,6 Go alloués à Docker

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

-- **01** : la version de départ, sans aucune limite. Le job écrit 8400 lignes, que j'ai copiées dans `data/out/reference.csv`. Il occupe un cœur entier (ratio de 0.99) et monte jusqu'à 1567 Mo, soit 6 fois plus que les 256 Mo visés.
  
- **02a** : le conteneur s'arrête sans afficher d'erreur. Avec `docker inspect`, on voit le code 137 (128 + 9, donc un SIGKILL) et `OOMKilled=true`. Le noyau l'a tué parce qu'il dépassait sa limite de mémoire.
- **02b** : le job va jusqu'au bout et donne le même résultat, mais il met 31,9 s au lieu de 17,1 s, presque deux fois plus. Le temps CPU ne change pas (16,8 s contre 16,9 s) : donc le travail est le même. Le ratio de 0.53 montre qu'il n'a eu droit qu'à un demi-cœur, il a donc été bridé.
  - Il y a une vraie différence entre les deux ressources. Sans assez de RAM, le conteneur est tué. Sans assez de CPU, il est ralenti.
  - On peut dire que c'est donc la mémoire qui pose problème.

### Dimensionnement retenu

Commande de lancement finale :

    docker run ...

Justification (pic mesuré, marge choisie) :

### Ce que nous en retenons (3 lignes maximum)

-
