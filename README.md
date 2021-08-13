# bresenham
Bresenham line algorithm in 6809 assembly

Il s'agit d'une implémentation de [l'algorithme de bresenham](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm) en assembleur 6809.

Cet algorithme permet de tracer des lignes en utilisant uniquement des nombres entiers. Il convient donc parfaitement au codage en assembleur sur un processeur 8 bits tel que le [motorola 6809](https://fr.wikipedia.org/wiki/Motorola_6809).

Pour le tester, il faut un ordinateur 8 bits avec ce processeur. Nous choisirons le [Thomson TO8](https://fr.wikipedia.org/wiki/Thomson_TO8).

# Fonctionnement
Le principe de l'algo est de trouver les points les plus proche de l'equation de droite formée par les coordonnées en entrées (X0,Y0)-(X1,Y1).
A chaque itération, une coordonnée (x,y) est calculée est les pixels correspondants sont allumés.

Dans un premier temps, le plus simple était une implémentation en Basic façon assembleur ; en utilisant des variables globales et des sous procédures.
  
```
' 320 X 200 X 8BPP INDEXED COLOR
SCREENRES 320, 200, 8

DIM SHARED AS INTEGER X0,X1,Y0,Y1
DIM SHARED AS INTEGER DX,DY,YI,XI,HL
DIM SHARED AS INTEGER X,Y,P

' INIT
X0=319:Y0=0:X1=0:Y1=199

'DX=X1-X0
'DY=Y1-Y0
HL=ABS(Y1-Y0)-ABS(X1-X0)

SUB DRAWXY
    PSET(X,Y)
END SUB

SUB SWAPCOORDS
    DIM BUF AS INTEGER
    BUF=X0
    X0=X1
    X1=BUF
    BUF=Y0
    Y0=Y1
    Y1=BUF
END SUB

SUB UPDATE_P1_Y
    Y=Y+YI:P=P+2*DY-2*DX
END SUB

SUB UPDATE_P2_Y
    P=P+2*DY
END SUB

SUB UPDATE_P1_X
    X=X+XI:P=P+2*DX-2*DY
END SUB

SUB UPDATE_P2_X
    P=P+2*DX
END SUB

IF HL<0 THEN
    IF X0>X1 THEN
        SWAPCOORDS
    END IF
    DX=X1-X0
    DY=Y1-Y0
    YI=1
    IF DY<0 THEN
        YI=-1:DY=-DY
    END IF
    X=X0:Y=Y0
    P=2*DY-DX
    DO
        IF X>=X1 THEN EXIT DO
        IF P>=0 THEN DRAWXY:UPDATE_P1_Y ELSE DRAWXY:UPDATE_P2_Y
        X=X+1
    LOOP
ELSE
    IF Y0>Y1 THEN
        SWAPCOORDS
    END IF
    DX=X1-X0
    DY=Y1-Y0
    XI=1
    IF DX<0 THEN
        XI=-1:DX=-DX
    END IF
    X=X0:Y=Y0
    P=2*DX-DY
    DO
        IF Y>=Y1 THEN EXIT DO
        IF P>=0 THEN DRAWXY:UPDATE_P1_X ELSE DRAWXY:UPDATE_P2_X
        Y=Y+1
    LOOP  
END IF

SLEEP
```

## Comment coder le DRAWXY sur un TO8 ?
Il faut savoir que :
- le bit 0 de l'octet &HE7C3 permet de commuter le mode forme/couleur de la mémoire écran.
- en mode forme, l'écran est divisé en 25 lignes de 40*8 colonnes soit 320*200 pixels.
Pour tracer un point il faut:
- activer le mode forme du controleur vidéo.
- calculer l'adresse du bloc de 8 pixels correspondant aux coordonnées (x,y).
- rechercher la valeur de l'octet correspondant au pixel dans le bloc de 8 (c'est facile, c'est le reste R de la division par 8 de la coordonnées X, puis on fait 2^R pour connaitre la valeur de l'octet).
- faire un OR avec le bloc de 8 pixels existant à cette adresse afin de ne pas effacer les pixels déjà allumés.
