# bresenham
Bresenham line algorithm in 6809 assembly

Il s'agit d'une implémentation de [l'algorithme de bresenham](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm) en assembleur 6809.

Cet algorithme permet de tracer des lignes en utilisant uniquement des nombres entiers. Il convient donc parfaitement au codage en assembleur sur un processeur 8 bits tel que le [motorola 6809](https://fr.wikipedia.org/wiki/Motorola_6809).

Pour le tester, il faut un ordinateur 8 bits avec ce processeur. Nous choisirons le [Thomson TO8](https://fr.wikipedia.org/wiki/Thomson_TO8).

# Fonctionnement
Le principe de l'algo est de trouver les points les plus proche de l'equation de droite formée par les coordonnées en entrées (X0,Y0)-(X1,Y1).
A chaque itération, une coordonnée (x,y) est calculée est les pixels correspondants sont allumés.

Dans un premier temps, le plus simple était une implémentation Basic façon assembleur ; en utilisant des variables globales et des sous procédures.
  
```
dsqdqs
```
