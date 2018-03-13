Gérer les collisions avec Pygame
================================

Voici un petit tuto qui explique comment gérer les collisions avec pygame pour un jeu de type _plateforme_.

Pygame offre quelques fonctions de collisions avec les objets `Rect`. Les difficultés apparaissent dès qu'on veut réagir à une collision. Ce petit tuto va tenter d'expliquer comment détecter une collision et comment y réagir.

Détection des collisions
------------------------

Nous ne traiterons que le cas simple de rectangles qui entrent en collision avec d'autres rectangles. Ces rectangles sont orientés selon les axes. Autrement dit ils ne sont pas _penchés_. Pour savoir s'il y a collision, on peut soit utiliser la méthode `colliderect` d'un objet `Rect` en lui passant en argument un autre objet `Rect`. La méthode retourne `True` s'il y a collision, `False` sinon.

Il est aussi possible de facilement faire cette détection par nous-même en utilisant le théorème de la séparation des axes. Le principe est très simple. Considérons 2 rectangles A et B.

![Séparation des axes][SAT]

Si le rectangle B est à gauche du rectangle A, on voit que peu importe sa hauteur, il n'entrera jamais en intersection avec A. De même si B est au-dessus de A, peu importe s'il est à gauche ou à droite, il n'entrera jamais en intersection avec A, Le même raisonnement s'applique si B est en-dessous ou à droite.

On détermine le premier cas où B est à gauche de A en vérifiant que `B.right < A.left`. On fera une opération similaire pour les autres cas.

La chose étonnante est que si on n'est dans aucun des 4 cas précédents, alors **forcément** il y a intersection entre A et B. Ainsi on pourrait écrire une fonction de détection de collision entre 2 `Rect` de cette manière :

```
def collision(rectA, rectB):
    if rectB.right < rectA.left:
        # rectB est à gauche
        return False
    if rectB.bottom < rectA.top:
        # rectB est au-dessus
        return False
    if rectB.left > rectA.right:
        # rectB est à droite
        return False
    if rectB.top > rectA.bottom:
        # rectB est en-dessous
        return False
    # Dans tous les autres cas il y a collision
    return True
```

Squelette de notre jeu de plateforme
------------------------------------

C'est déjà bien de savoir comment détecter des collisions. Mais ça ne nous dit pas quoi faire lorsqu'il y a collision ! Dans le cas de notre jeu de plateforme, notre personnage peut bouger à gauche, à droite, il peut sauter et il est soumis à la pesanteur. Ainsi il tente de bouger constament vers le bas. C'est le sol sous ses pieds qui l'empêche de tomber.

Voici tout d'abord le squelette de notre jeu sans s'occuper des collisions. Le personnage peut aller à gauche et à droite. Et on peut sauter. Il tombe aussi mais on limite sa chute à une hauteur prédéfinie, sans quoi il disparaîtrait rapidement de l'écran.

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
 
import pygame
from pygame.locals import *
 
# On initialise pygame
pygame.init()
taille_fenetre = (600, 400)
fenetre_rect = pygame.Rect((0, 0), taille_fenetre)
screen_surface = pygame.display.set_mode(taille_fenetre)
 
BLEU_NUIT = (  5,   5,  30)
VERT      = (  0, 255,   0)
JAUNE     = (255, 255,   0)
 
timer = pygame.time.Clock()
 
joueur = pygame.Surface((25, 25))
joueur.fill(JAUNE)
# Position du joueur
x, y = 25, 100
# Vitesse du joueur
vx, vy = 0, 0
# Gravité vers le bas donc positive
GRAVITE = 2
 
mur = pygame.Surface((25, 25))
mur.fill(VERT)
 
niveau = [
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    ]
 
def dessiner_niveau(surface, niveau):
    """Dessine le niveau sur la surface donnée.
 
    Utilise la surface `mur` pour dessiner les cases de valeur 1
    """
    for j, ligne in enumerate(niveau):
        for i, case in enumerate(ligne):
            if case == 1:
                surface.blit(mur, (i*25, j*25))
 
# Boucle événementielle
continuer = True
while continuer:
    for event in pygame.event.get():
        if event.type == QUIT:
            continuer = False
        elif event.type == KEYDOWN:
            if event.key == K_SPACE:
                vy = -20
 
    timer.tick(30)
    keys_pressed = pygame.key.get_pressed()
    vx = (keys_pressed[K_RIGHT] - keys_pressed[K_LEFT]) * 5
    vy += GRAVITE
    vy = min(20, vy) # On limite la vitesse de la chute à un max de 20
    # Chaque frame (1/30 sec) on avance de la vitesse vx et vy. C'est comme
    # si on admettait que le temps unitaire était de 1/30 de sec.
    x += vx
    y += vy
    # Limitons pour le moment jusqu'où le personnage peut tomber
    y = min(300, y)
 
    screen_surface.fill(BLEU_NUIT)
    dessiner_niveau(screen_surface, niveau)
    screen_surface.blit(joueur, (x, y))
    pygame.display.flip()
 
pygame.quit()
```

On voit notre personnage tomber et s'arrêter à la hauteur de 300 px. On peut aller à gauche, à droite et on peut sauter. A chaque frame on calcule sa vitesse `vx` en regardant si les flèches gauche et droite sont enfoncées. Si oui `keys_pressed[K_RIGHT]` vaudra 1 et on le multiplie par 5, ce qui donne un déplacement de 5 px par frame. Si c'est `keys_pressed[K_LEFT]` qui est enfoncée, la valeur sera aussi de 1, mais on retire cette valeur, donc ce sera -1 si on enfonce que cette touche, et on multiplie aussi par 5. Si on enfonce les deux flèches, les vitesses s'annulent et on ne va nul part.

Pour la gravité, c'est très simple. On ajoute à la vitesse `vy` une accélération constante de 2 px par frame. Donc à la première frame, `vy` vaut 0 et on ajoute 2. Donc vy vaut 2. Ensuite à la frame suivante on ajoute encore 2, et `vy` devient 4. Et ainsi de suite. On limite toutefois la vitesse à 20 px / frame, juste pour que la chute n'atteigne pas des vitesses vertigineuses.

Détecter les collisions avec le décor
-------------------------------------

Pour détecter si notre personnage entre en collision avec le décor, on doit vérifier si notre niveau contient des valeurs de 1. Là où se trouve notre personnage. La position de notre personne x, y définit en réalité son coin supérieur gauche. Voici un exemple de sa position à un moment donné.

![Collision avec un mur][pos in grid]

On voit que puisque la taille du joueur est la même que la taille des cases de notre niveau, le joueur ne peut à tout moment entrer en contact qu'avec 4 cases : la case contenant son coin supérieur gauche, la case à droite, la case en dessous et la case en-dessous et à droite. Il nous faut donc une fonction qui pour une position `x, y` nous dise dans quel case `i, j` on se trouve. Il nous faut aussi une autre fonction qui nous retourne une liste de `Rect` représentant les murs de notre niveau dans les 4 cases en partant de la position `i, j`. C'est donc la case contenant le coin supérieur gauche de notre personnage.

Pour la première fonction, on obtient ceci :

```
def from_coord_to_grid(pos):
    """Retourne la position dans le niveau en indice (i, j)
 
    `pos` est un tuple contenant la position (x, y) du coin supérieur gauche.
    On limite i et j à être positif.
    """
    x, y = pos
    i = max(0, int(x / 25))
    j = max(0, int(y / 25))
    return i, j
```

Cette fonction ne prend qu'un seul argument qui sera par exemple un tuple avec la position `x, y`. On récupère les deux éléments de notre argument dans les variables `x` et `y`. Voir [l'unpacking](http://sametmax.com/quest-ce-que-lunpacking-en-python-et-a-quoi-ca-sert/). Puisque nos bloques de décor ont une largeur / hauteur de 25, il suffit de diviser la position par 25. On s'assure ensuite de ne garder que la partie entière. Et on limite le résulat à un nombre positif. On ne veut pas retourner d'indices négatifs, car ça aurait des effets indésirables quand on les utilisera dans des slices.

Pour la deuxième fonction :

```
def get_neighbour_blocks(niveau, i_start, j_start):
    """Retourne la liste des rectangles autour de la position (i_start, j_start).
 
    Vu que le personnage est dans le carré (i_start, j_start), il ne peut
    entrer en collision qu'avec des blocks dans sa case, la case en-dessous,
    la case à droite ou celle en bas et à droite. On ne prend en compte que
    les cases du niveau avec une valeur de 1.
    """
    blocks = list()
    for j in range(j_start, j_start+2):
        for i in range(i_start, i_start+2):
            if niveau[j][i] == 1:
                topleft = i*25, j*25
                blocks.append(pygame.Rect((topleft), (25, 25)))
    return blocks
```

On se créé d'abord une liste viste qui va contenir nos `Rect`. On parcourt ensuite notre liste de listes en visitant les 4 cases partant de `i_start, j_start`. Si la valeur trouvée dans cette case est un 1, il y a un mur et on créé un `Rect` avec les coordonnées adéquates, à savoir un coin supérieur gauche égal à la position `i, j` multiplié par 25, et une largeur et hauteur de 25. On retourne cette liste de rectangles.

A présent on peut détecter si on entre en collision avec notre décor. Mais que faire dans ce cas ?

Une solution naïve est de garder un copie de notre position lors de la frame précédente, de faire avancer notre personnage selon sa vitesse, de vérifier s'il y a collision avec le décor, et si oui, de le replacer à son ancienne position.

Il suffit de modifier la boucle principale comme ceci, en ajoutant une fonction pour détecter s'il y a collision :

```
def collision(niveau, pos):
    rect = pygame.Rect(pos, (25, 25))
    i, j = from_coord_to_grid(pos)
    for block in get_neighbour_blocks(niveau, i, j):
        if rect.colliderect(block):
            return True
    return False
 
# Boucle événementielle
continuer = True
while continuer:
    for event in pygame.event.get():
        if event.type == QUIT:
            continuer = False
        elif event.type == KEYDOWN:
            if event.key == K_SPACE:
                vy = -20
 
    timer.tick(30)
    keys_pressed = pygame.key.get_pressed()
    # Sauvegarde de l'ancienne position
    old_x, old_y = x, y
    vx = (keys_pressed[K_RIGHT] - keys_pressed[K_LEFT]) * 5
    vy += GRAVITE
    vy = min(20, vy) # vy ne peut pas dépasser 25 sinon effet tunnel...
    x += vx
    y += vy
    if collision(niveau, (x, y)):
        x, y = old_x, old_y
 
    screen_surface.fill(BLEU_NUIT)
    dessiner_niveau(screen_surface, niveau)
    screen_surface.blit(joueur, (x, y))
    pygame.display.flip()
 
pygame.quit()
```

La fonction `collision` prend en argument notre niveau et la position du personnage. Elle crée un `Rect` à la position du personnage, détermine les coordonnées `i, j` où se trouve notre personnage et itère sur tous les `Rect` retournés par notre fonction `get_neighbour_blocks`. Elle vérifie s'il y a collision et retourne True ou False selon.

Dans notre boucle principale, on garde une copie de notre ancienne position dans `old_x, old_y`. On vérifie s'il y a collision et si oui, on replace notre personnage à l'ancienne position.

On voit notre personnage tomber, puis il semble bloqué. Si on le fait sauter, il se débloque mais se retrouve de nouveau coincé. Aussi il ne tombe jamais _contre_ le bloque en-dessous de lui.

La raison de ce comportement est due au fait qu'à chaque frame, notre personnage est en chute libre. `vy` vaut probablement 20 et donc la nouvelle position du personnage entre en collision avec le bloque sous ses pieds. En cas de collision on l'empêche de bouger, donc il ne sait rien faire d'autre.

Distance de pénétration
-----------------------

Pour résoudre notre problème, il faudrait réagir différement à la collision. Si le personnage entre en collision, il devrait pouvoir se déplacer jusqu'à arriver contre l'obstacle. Pour ce faire, il va falloir déterminer de combien il a pénétré l'obstacle pour le faire reculer d'autant. Facile à dire ! Mais comment faire ce calcul ?

Prenons le cas du notre personnage qui tombe. A la frame `$ N $`, il était encore au-dessus du bloque sous ses pieds. A la frame `$ N+1 $`, il entre en collision avec.

![Pénétration][penetration]

On voit que le joueur devrait être reculé d'une distance `dy_correction` pour se retrouver en contact avec le bloque. Cette distance est simplement egale à `dy_correction = block.top - perso_rect.bottom`.

On ne doit faire ce calcul que lorsqu'il y a collision. Une manière de procéder est de comparer la position de `perso_rect.bottom` avant le déplacement et après. Si avant le déplacement il était au-dessus de `block.top` et qu'après déplacement il est en-dessous de `block.top` ça veut dire que c'est son côté bas qui vient d'entrer en collision. Et dans ce cas, on peut calculer le `dy_correction`. On peut faire de même pour les 4 faces.

Voici la fonction qui nous retourne les distances de pénétration sur les 2 axes :

```
def compute_penetration(block, old_rect, new_rect):
    """Calcule la distance de pénétration du `new_rect` dans le `block` donné.
 
    `block`, `old_rect` et `new_rect` sont des pygame.Rect.
    Retourne les distances `dx_correction` et `dy_correction`.
    """
    dx_correction = dy_correction = 0.0
    if old_rect.bottom <= block.top < new_rect.bottom:
        dy_correction = block.top  - new_rect.bottom
    elif old_rect.top >= block.bottom > new_rect.top:
        dy_correction = block.bottom - new_rect.top
    if old_rect.right <= block.left < new_rect.right:
        dx_correction = block.left - new_rect.right
    elif old_rect.left >= block.right > new_rect.left:
        dx_correction = block.right - new_rect.left
    return dx_correction, dy_correction
```

Elle prend en argument notre `Rect` du mur, et deux autres `Rect` représentant la position du personnage avant et après son déplacement. On reçoit en retour les corrections sur l'axe des X et Y.

On peut à présent se créer un fonction qui gère les collisions de notre personnage et nous retourne sa position finale, après résolution des collisions.

```
def bloque_sur_collision(niveau, old_pos, new_pos, vx, vy):
    """Tente de déplacer old_pos vers new_pos dans le niveau.
 
    S'il y a collision avec les éléments du niveau, new_pos sera ajusté pour
    être adjacents aux éléments avec lesquels il entre en collision.
 
    La fonction retourne la position modifiée pour new_pos.
    """
    old_rect = pygame.Rect(old_pos, (25, 25))
    new_rect = pygame.Rect(new_pos, (25, 25))
    i, j = from_coord_to_grid(new_pos)
    blocks = get_neighbour_blocks(niveau, i, j)
    for block in blocks:
        if not new_rect.colliderect(block):
            continue
 
        dx_correction, dy_correction = compute_penetration(block, old_rect, new_rect)
        new_rect.top += dy_correction
        new_rect.left += dx_correction
 
    x, y = new_rect.topleft
    return x, y
 
# Boucle événementielle
continuer = True
while continuer:
    for event in pygame.event.get():
        if event.type == QUIT:
            continuer = False
        elif event.type == KEYDOWN:
            if event.key == K_SPACE:
                vy = -20
 
    timer.tick(30)
    keys_pressed = pygame.key.get_pressed()
    # Sauvegarde de l'ancienne position
    old_x, old_y = x, y
    vx = (keys_pressed[K_RIGHT] - keys_pressed[K_LEFT]) * 5
    vy += GRAVITE
    vy = min(20, vy) # vy ne peut pas dépasser 25 sinon effet tunnel...
    x += vx
    y += vy
    x, y = bloque_sur_collision(niveau, (old_x, old_y), (x, y))
 
    screen_surface.fill(BLEU_NUIT)
    dessiner_niveau(screen_surface, niveau)
    screen_surface.blit(joueur, (x, y))
    pygame.display.flip()
 
pygame.quit()
```

On voit le personnage tomber jusqu'à toucher le sol. Hourra ! On se déplace à droite puis à gauche, et le personnage est bloqué ! Arrghh !!!

Pourquoi le personnage bloque ?
-------------------------------

Pour comprendre pourquoi le personnage bloque, il faut comprendre ce qu'il se passe lorsqu'on tente d'aller à gauche.

![Correction minimum][min correction]

Le personnage possède une vitesse `vx` et `vy`. Sa nouvelle position est donc en collision avec les deux bloques sous lui. On teste les collisions dans l'ordre donné sur le diagramme : de gauche à droite et du haut vers le bas. Donc on vérifie les collisions dans la case 3 avant la case 4. Pour la case 3, il y a collision à la fois avec le bord inférieur et avec le bord gauche de notre personnage. La fonction `compute_penetration` nous retournera un `dx_correction` et un `dy_correction` que nous appliquons à la position du personnage et il se retrouve dans sa position initiale. Donc il est bloqué !

En réalité il ne faut corriger que par la plus petite distance pour sortir hors de l'obstacle. Malgré tout, dans cet exemple on voit bien que ça ne fonctionne pas, car la plus petite distance est `dx_correction`. L'autre problème est **l'ordre** dans lequel on considère les bloques. Si on considérait la case 4 avant la 3, on verrait que notre personnage n'entre en collision qu'avec le bas. Et il n'y a qu'une correction `dy_correction`. Ensuite en regardant avec la case 3, il n'y a plus de collision puisque la case 4 nous a repoussé vers le haut.

Mais quel ordre devons-nous choisir ? Il faut commencer par résoudre les cases dont la correction ne s'applique que sur un seul axe, soit X ou Y. On garde les autres bloques pour plus tard. Une fois qu'on a résolu toutes ces collisions, on reprend les bloques qu'on avait gardés, et on résout les collision en n'appliquant que la plus petite correction entre `dx_correction` et `dy_correction`. On entend bien sûr la plus petite valeur **absolue**. Ainsi pour une correction `dx_correction = -8, dy_correction = 2`, on n'appliquerait que la correction `dy_correction`.

Notre nouvelle fonction `bloque_sur_collision` va également prendre en argument les vitesses `vx` et `vy` et elle retournera les vitesses corrigées en cas de collision. Ainsi si notre personnage tombe sur le sol, sa `vy` deviendra 0, jusqu'à la prochaine frame bien entendu...

```
def bloque_sur_collision(niveau, old_pos, new_pos, vx, vy):
    """Tente de déplacer old_pos vers new_pos dans le niveau.
 
    S'il y a collision avec les éléments du niveau, new_pos sera ajusté pour
    être adjacent aux éléments avec lesquels il entre en collision.
    On passe également en argument les vitesses `vx` et `vy`.
 
    La fonction retourne la position modifiée pour new_pos ainsi que les
    vitesses modifiées selon les éventuelles collisions.
    """
    old_rect = pygame.Rect(old_pos, (25, 25))
    new_rect = pygame.Rect(new_pos, (25, 25))
    i, j = from_coord_to_grid(new_pos)
    collide_later = list()
    blocks = get_neighbour_blocks(niveau, i, j)
    for block in blocks:
        if not new_rect.colliderect(block):
            continue
 
        dx_correction, dy_correction = compute_penetration(block, old_rect, new_rect)
        # Dans cette première phase, on n'ajuste que les pénétrations sur un
        # seul axe.
        if dx_correction == 0.0:
            new_rect.top += dy_correction
            vy = 0.0
        elif dy_correction == 0.0:
            new_rect.left += dx_correction
            vx = 0.0
        else:
            collide_later.append(block)
 
    # Deuxième phase. On teste à présent les distances de pénétrations pour
    # les blocks qui en possédaient sur les 2 axes.
    for block in collide_later:
        dx_correction, dy_correction = compute_penetration(block, old_rect, new_rect)
        if dx_correction == dy_correction == 0.0:
            # Finalement plus de pénétration. Le new_rect a bougé précédemment
            # lors d'une résolution de collision
            continue
        if abs(dx_correction) < abs(dy_correction):
            # Faire la correction que sur l'axe X (plus bas)
            dy_correction = 0.0
        elif abs(dy_correction) < abs(dx_correction):
            # Faire la correction que sur l'axe Y (plus bas)
            dx_correction = 0.0
        if dy_correction != 0.0:
            new_rect.top += dy_correction
            vy = 0.0
        elif dx_correction != 0.0:
            new_rect.left += dx_correction
            vx = 0.0
 
    x, y = new_rect.topleft
    return x, y, vx, vy
```

Comme avant on se créé les `Rect` de notre position avant et après déplacement et on calcule les coordonnées `i, j`. On se prépare une liste `collide_later` dans laquelle on placera les `Rect` pour lesquels la fonction `compute_penetration` a retourné une correction sur les 2 axes.

Pour chaque bloque dans les alentours, on vérifie s'il y a collision. Si oui on calcule les corrections `dx_correction, dy_correction`. Si la correction est nulle sur un axe, on l'applique sur l'autre axe. On en profite pour mettre la vitesse à 0, puisqu'on a heurté qqch dans cette direction. Sinon on place le `Rect` dans notre liste de rectangles qu'on considérera plus tard.

Une fois la boucle finie, on itère cette fois sur les `Rect` qui nous donnaient une correction sur les 2 axes. On re-caclule la correction qui maintenant a pu changer, puisque d'autres bloques auraient pu interagir avec le personnage. On regarde sur quel axe est la plus petite correction, et on annule l'autre correction. Finalement on applique cette correction. On retourne notre nouvelle position et la nouvelle vitesse.

Nous y sommes ! Notre personnage peut se déplacer dans le décor et s'arrêter lorsqu'il entre en contact avec un obstacle. Voici le code final :

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
 
import pygame
from pygame.locals import *
 
# On initialise pygame
pygame.init()
taille_fenetre = (600, 400)
fenetre_rect = pygame.Rect((0, 0), taille_fenetre)
screen_surface = pygame.display.set_mode(taille_fenetre)
 
BLEU_NUIT = (  5,   5,  30)
VERT      = (  0, 255,   0)
JAUNE     = (255, 255,   0)
 
timer = pygame.time.Clock()
 
joueur = pygame.Surface((25, 25))
joueur.fill(JAUNE)
# Position du joueur
x, y = 25, 100
# Vitesse du joueur
vx, vy = 0, 0
# Gravité vers le bas donc positive
GRAVITE = 2
 
mur = pygame.Surface((25, 25))
mur.fill(VERT)
 
niveau = [
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    ]
 
def dessiner_niveau(surface, niveau):
    """Dessine le niveau sur la surface donnée.
 
    Utilise la surface `mur` pour dessiner les cases de valeur 1
    """
    for j, ligne in enumerate(niveau):
        for i, case in enumerate(ligne):
            if case == 1:
                surface.blit(mur, (i*25, j*25))
 
def from_coord_to_grid(pos):
    """Retourne la position dans le niveau en indice (i, j)
 
    `pos` est un tuple contenant la position (x, y) du coin supérieur gauche.
    On limite i et j à être positif.
    """
    x, y = pos
    i = max(0, int(x // 25))
    j = max(0, int(y // 25))
    return i, j
 
def get_neighbour_blocks(niveau, i_start, j_start):
    """Retourne la liste des rectangles autour de la position (i_start, j_start).
 
    Vu que le personnage est dans le carré (i_start, j_start), il ne peut
    entrer en collision qu'avec des blocks dans sa case, la case en-dessous,
    la case à droite ou celle en bas et à droite. On ne prend en compte que
    les cases du niveau avec une valeur de 1.
    """
    blocks = list()
    for j in range(j_start, j_start+2):
        for i in range(i_start, i_start+2):
            if niveau[j][i] == 1:
                topleft = i*25, j*25
                blocks.append(pygame.Rect((topleft), (25, 25)))
    return blocks
 
def bloque_sur_collision(niveau, old_pos, new_pos, vx, vy):
    """Tente de déplacer old_pos vers new_pos dans le niveau.
 
    S'il y a collision avec les éléments du niveau, new_pos sera ajusté pour
    être adjacents aux éléments avec lesquels il entre en collision.
    On passe également en argument les vitesses `vx` et `vy`.
 
    La fonction retourne la position modifiée pour new_pos ainsi que les
    vitesses modifiées selon les éventuelles collisions.
    """
    old_rect = pygame.Rect(old_pos, (25, 25))
    new_rect = pygame.Rect(new_pos, (25, 25))
    i, j = from_coord_to_grid(new_pos)
    collide_later = list()
    blocks = get_neighbour_blocks(niveau, i, j)
    for block in blocks:
        if not new_rect.colliderect(block):
            continue
 
        dx_correction, dy_correction = compute_penetration(block, old_rect, new_rect)
        # Dans cette première phase, on n'ajuste que les pénétrations sur un
        # seul axe.
        if dx_correction == 0.0:
            new_rect.top += dy_correction
            vy = 0.0
        elif dy_correction == 0.0:
            new_rect.left += dx_correction
            vx = 0.0
        else:
            collide_later.append(block)
 
    # Deuxième phase. On teste à présent les distances de pénétrations pour
    # les blocks qui en possédaient sur les 2 axes.
    for block in collide_later:
        dx_correction, dy_correction = compute_penetration(block, old_rect, new_rect)
        if dx_correction == dy_correction == 0.0:
            # Finalement plus de pénétration. Le new_rect a bougé précédemment
            # lors d'une résolution de collision
            continue
        if abs(dx_correction) < abs(dy_correction):
            # Faire la correction que sur l'axe X (plus bas)
            dy_correction = 0.0
        elif abs(dy_correction) < abs(dx_correction):
            # Faire la correction que sur l'axe Y (plus bas)
            dx_correction = 0.0
        if dy_correction != 0.0:
            new_rect.top += dy_correction
            vy = 0.0
        elif dx_correction != 0.0:
            new_rect.left += dx_correction
            vx = 0.0
 
    x, y = new_rect.topleft
    return x, y, vx, vy
 
def compute_penetration(block, old_rect, new_rect):
    """Calcul la distance de pénétration du `new_rect` dans le `block` donné.
 
    `block`, `old_rect` et `new_rect` sont des pygame.Rect.
    Retourne les distances `dx_correction` et `dy_correction`.
    """
    dx_correction = dy_correction = 0.0
    if old_rect.bottom <= block.top < new_rect.bottom:
        dy_correction = block.top  - new_rect.bottom
    elif old_rect.top >= block.bottom > new_rect.top:
        dy_correction = block.bottom - new_rect.top
    if old_rect.right <= block.left < new_rect.right:
        dx_correction = block.left - new_rect.right
    elif old_rect.left >= block.right > new_rect.left:
        dx_correction = block.right - new_rect.left
    return dx_correction, dy_correction
 
# Boucle événementielle
continuer = True
while continuer:
    for event in pygame.event.get():
        if event.type == QUIT:
            continuer = False
        elif event.type == KEYDOWN:
            if event.key == K_SPACE:
                vy = -20
 
    timer.tick(30)
    keys_pressed = pygame.key.get_pressed()
    # Sauvegarde de l'ancienne position
    old_x, old_y = x, y
    vx = (keys_pressed[K_RIGHT] - keys_pressed[K_LEFT]) * 5
    vy += GRAVITE
    vy = min(20, vy) # vy ne peut pas dépasser 25 sinon effet tunnel...
    x += vx
    y += vy
    x, y, vx, vy = bloque_sur_collision(niveau, (old_x, old_y), (x, y), vx, vy)
 
    screen_surface.fill(BLEU_NUIT)
    dessiner_niveau(screen_surface, niveau)
    screen_surface.blit(joueur, (x, y))
    pygame.display.flip()
 
pygame.quit()
```

[SAT]: pygame_collisions_img/SAT.png "Théorème de la séparation des axes"
[pos in grid]: pygame_collisions_img/pos_in_grid.png "Collision du joueur avec un mur"
[penetration]: pygame_collisions_img/penetration.png "Pénétration des rectangles."
[min correction]: pygame_collisions_img/minimum_correction.png "Correction minimum"
