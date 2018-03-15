Les bonnes pratiques avec Tkinter
=================================

Tkinter est une interface graphique (IHM). Elle permet de pouvoir communiquer graphiquement entre l'homme et la machine afin d'exécuter
votre programme python.

Nous allons voir ici certaines bonnes pratiques utilisées afin d'éviter des déboires tant dans la conception de votre programme que dans la syntaxe.

Pour cela nous allons parler ici de

* l'importation du module
* la compatibilité de python 2.x et 3.x
* l'initialisation de la fenêtre principale
* la création d'un bouton avec sa commande
* le mot clé global
    

Importation du module
---------------------

### 1. À éviter:

Il faut éviter d'importer son module Tkinter, ni même d'autres modules d'ailleurs sous la forme ci-dessous

`from tkinter import *`

La raison est assez simple, lorsque vous importez tkinter de cette manière, vous importez l'ensemble des variables liées au module.
Mais connaissez-vous ces noms de variables ? Non ! Il est donc fort possible que vous écrasiez une de ces variables lors de l'écriture
de votre code et ainsi rendre cette variable inutilisable par le module lui-même.

### 2. La bonne manière:

Il est donc important d'importer concrètement le nom de cette variable ou mieux, d'importer tkinter et appeler une de ces variables de cette
manière:

```python
import tkinter  # super!
ou  # peut devenir vite pénible quand beaucoup de variables à importer
from tkinter import variable1, ...
```


Alors vous me direz, ouais, pas grand chose me convient, importer tkinter est assez verbeux, et l'autre solution semble plus pénible.
Pas de problème ! On peut raccourcir cette importation à l'aide du mot clé `as`.

`import tkinter as tk` et là on met tout le monde d'accord !!! C'est plus court, on peut appeler nos variables à volonté très simplement.

La compatibilité entre la version 2.x et 3.x
--------------------------------------------

Il arrive parfois que l'on souhaite que notre programme soit utilisable tant pour les utilisateurs de la version 3.x que pour les utilisateurs de
la version 2.x (même si on ne conseille plus cette version depuis un certains moment).

Pour cela on va tester une importation avec une certaine version de python, et indiquer qu'en cas d'erreur d'importation, on importe sur
l'autre version. On codera cela de la manière ci-dessous...


```python
try:
    # pour python 3.x
    import tkinter as tk
except ImportError:
    # pour python 2.x
    import Tkinter as tk
```


Initialisation de la fenêtre principale
---------------------------------------

Tkinter est une interface graphique, son initialisation est représentée par une fenêtre principale vide attendant des évènements de l'utilisateur.

Niveau code, c'est en une ligne `window = tk.Tk()`

`Tk` (avec un T majuscule) est le nom de la classe appelée (la classe est l'usine créant l'objet)
`window` est le nom de l'instance de classe (l'instance est l'objet créé)

Il est très fortement conseillé d'appeler cette classe une seule fois, surtout quand on débute... Dans le cas où vous souhaitez de multiples
fenêtres, il faudra utiliser une autre classe nommée `Toplevel`.

Création d'un bouton avec sa commande
-------------------------------------

Pour créer un bouton, il faut appeler (roulement de tambour), la classe Button.

Nous allons ici ne donner qu'un petit exemple parmi beaucoup d'autres sur l'utilisation d'un objet Button.


```python
try:
    # pour python 3.x
    import tkinter as tk
except ImportError:
    # pour python 2.x
    import Tkinter as tk

root = tk.Tk()

button = tk.Button(root, text='quitter', command=root.quit)
button.pack()

root.mainloop()
```


Le bouton `button` créé dans notre fenêtre principale `root` avec comme texte d'information 'quitter', permettra de détruire la fenêtre principale
et stopper l'exécution du programme à l'aide de la commande `root.quit()`. Remarquer l'absence de parenthèses après `root.quit`, lié au fait que
le paramètre `command` demande l'objet fonction et non l'objet appelé par la fonction.

`pack` est une méthode pour placer l'objet `button` créé dans la fenêtre `root`.

`mainloop` indique la création de la boucle événementielle et attend les évènements utilisateurs

Le mot clé global
-----------------

Ce mot clé permet entre autre, la modification d'une variable à partir d'une fonction.


```python
def test():
    global x
    x = 5

x = 2
print(x)  # affiche 2

test()
print(x)  # affiche 5
```


Sans le terme `global`, `x` aurait toujours comme valeur 2 !

Ça peut avoir un intérêt, mais lorsqu'on est expérimenté et que l'on sait exactement pourquoi on le fait... Souvent ça permet de rendre fonctionnel
(un genre de rustine) un programme où l'on avait besoin d'ajouter une variable dont celle-ci serait modifier dans une fonction.

Par contre, le mot clé `global` est en règle générale une très mauvaise pratique, car souvent très nombreux dans les codes et rendant le code
difficilement débuggable par la suite et maintenable. Si on souhaite modifier une partie du code, c'est souvent ce mot clé qui fait planter le
code par la suite, car on ne maîtrise plus les variables qui sont ou ne sont pas modifiées.

Conseil: Ne l'utilisez jamais, surtout en tant que débutant !
