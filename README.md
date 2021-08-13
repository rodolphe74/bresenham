# Bresenham
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

Il ne reste plus qu'à convertir ce programme basic en [assembleur 6809](https://github.com/rodolphe74/bresenham/blob/main/bresenham.ass).


## Comment coder le DRAWXY en assembleur sur un TO8 ?
Il faut savoir que :
- le bit 0 de l'octet &HE7C3 permet de commuter le mode forme/couleur de la mémoire écran, lorsqu'il est à 1, le contrôleur vidéo est en mode forme.
- la mémoire vidéo commence à l'adresse &H4000 sur 8000 octets
- en mode forme, l'écran est divisé en 25 lignes de 40*8 colonnes soit 320*200 pixels.
Pour tracer un point il faut:
- activer le mode forme du controleur vidéo.
- calculer l'adresse du bloc de 8 pixels correspondant aux coordonnées (x,y).
- rechercher la valeur de l'octet correspondant au pixel dans le bloc de 8 (c'est facile, c'est le reste R de la division par 8 de la coordonnées X, puis on fait 2^R pour connaitre la valeur de l'octet).
- faire un OR avec le bloc de 8 pixels existant à cette adresse afin de ne pas effacer les pixels déjà allumés.

Par exemple, je souhaite allumer le pixel (203,52):
```
203/8=25 -> R=3 -> 2^R = 8
l'octet cible de la mémoire video -> Y*40 + X/8 -> 52*40+25 = 2105
auquel on ajoute 2^R -> 2105 + 8 = 2113
il reste à faire le OR avec l'octet existant à cette adresse
```

En assembleur 6809, cela donne:

```
    JSR DRAWFORME
    JSR DRAWXY

DRAWFORME
* passage en mode forme
    PSHS X
    LDX COMMUTATEUR
    LDA ,X
    ORA #1
    STA ,X
    PULS X
    RTS

DRAWXY
    PSHS D
    PSHS X
* calcul de l'adresse ecran : V=INT(X/8)+Y*40+&H4000
    LDD VARX
    STD VARDIVNUM
    JSR DIV_BY_EIGHT        * quotient dans VARDIVQ - reste dans VARDIVR (version optimisee)
    LDD VARY
    LDA #40
    MUL                     * Y*40 dans reg D (Y < 200 : tient dans B)
    LDX #VARDIVQ
    ADDD ,X                 * X/8 + Y*40 dans reg D
    LDX #BASESCRADDR        * base ecran TO
    ADDD ,X                 * base ecran TO + X/8 + Y*40 dans reg D
    STD SCRADDR             * sauvegarde adresse
    LDX SCRADDR             * adresse dans X

    LDY #PXDATA
    LDD VARDIVR             * reste div 8 dans D donc dans B
    EXG A,B                 * reste div 8 dans A
    LEAY A,Y                * decalage du reste de la div dans Y
    LDB ,Y                  * valeur adresse Y dans B
    STB ORVAL               * sauvegarde de B
    LDA ,X                  * memoire courante dans reg A
    ORA ORVAL               * normalement c'est bon dans A
    STA ,X                  * ecriture memoire ecran
    PULS X
    PULS D
    RTS
  
PXDATA
    FCB 128,64,32,16,8,4,2,1
BASESCRADDR
    FDB $4000
COMMUTATEUR
    FDB $E7C3               * bit 0 a 1 pour passer en mode forme
ORVAL
    FCB 0
VARX
    FDB 0
VARY
    FDB 0
VARDIVR
    FDB 0
VARDIVQ
    FDB 0
```
