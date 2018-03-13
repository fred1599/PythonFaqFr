Comment contribuer à Python FAQ FR ?
====================================

On peut contribuer de toute sorte de manière à Python FAQ FR. Ça peut aller de la création d'un nouvel article à l'ajout d'une section ou à la correction d'un texte.

Cloner le dépôt
---------------

La première chose à faire est de cloner le dépôt. Il vous faut un compte sur Github si ce n'est pas déjà fait. Rendez-vous ensuite sur le dépôt de Python FAQ FR à l’adresse [https://github.com/dangillet/PythonFaqFr](https://github.com/dangillet/PythonFaqFr) et cliquer en haut à droite sur l'icône **Fork**.

Depuis le _fork_ du dépôt, vous allez maintenant pouvoir le cloner. Vous trouverez le lien correct à utiliser en cliquant sur le bouton vert **Clone or download**. La commande à entrer dans votre terminal est :

```
git clone https://github.com/username/PythonFaqFr.git
```

en mettant l'adresse récupérée dans le bouton vert **Clone or download**.

Ceci créera dans le répertoire courant un dossier `PythonFaqFr`. Dedans se trouve un répertoire `doc` qui contient les articles.

Création d'un environnement virtuel
-----------------------------------

Afin d'avoir les bibliothèques requises pour visualiser la documentation offline, il est recommandé de créer un environnement virtuel. La méthode décrite ci-dessous utilise pipenv. Tout d'abord il faut l'installer :

```
pip install pipenv
```

Ensuite, depuis le dossier `PythonFaqFr` il suffit de faire :

```
pipenv install
```

pipenv va trouver le fichier `Pipfile` et installer les dépendances nécessaires dans un nouvel environement virtuel. Il faut à présent activer cet environnement virtuel en faisant :

```
pipenv shell
```

Vous êtes à présent dans l’environnement virtuel.

Pour construire la documentation, vous vous rendez dans le sous-dossier `doc` et vous faites :

```
make html
```

Un sous-dossier `_build` est créer contenant deux dossiers : `doctrees` et `html`. Dans le dossier `html` se trouve le fichier `index.html`. En ouvrant ce dernier avec votre browser préféré, vous pourrez voir la documentation offline.

Pour quitter l’environnement virtuel, il suffit de faire :

```
exit
```

Structure du projet
-------------------

La documentation est générée grâce à [sphinx](http://www.sphinx-doc.org/en/master/). Toutes les pages de la documentation se trouvent dans le dossier `doc`. La page d’accueil se trouve dans le fichier `index.rst`. Ce dernier contient la table des matières.

Pour ajouter une nouvelle page, il suffit d'ajouter un nouveau fichier avec l'extension `.rst` pour utiliser le format [reStructuredText](http://docutils.sourceforge.net/rst.html) ou l'extension `.md` pour utiliser le format [Commonmark](http://commonmark.org/) (un type de Markdown).

Ensuite il faut référencer ce nouveau fichier depuis la table des matières contenue dans `index.rst`. Il suffit d'ajouter le nom du fichier, sans son extension, dans la table des matières appropriée. Il existe une table des matières par catégorie. Il est aisé d'ajouter une nouvelle catégorie si nécessaire.

Créer un _pull request_
-----------------------

Une fois le travail de rédaction ou de correction effectué, il faut créer un _pull request_ afin de l'ajouter au projet. Toutes vos modifications doivent d'abord être ajoutées à votre dépôt. Il ne s'agit pas ici de faire un tutoriel complet sur l'utilisation de Git, mais ces quelques commandes devraient suffire.

```
git add nom_fichier1 nom_fichier2
```

`git add` permet d'ajouter des fichiers modifiés. Il suffit de donner les noms des fichiers séparés d'un espace à cette commande. Ensuite il faut faire un _commit_, c'est à dire sauvegarder l'état des fichiers.

```
git commit -m "Votre message"
```

Vous écrivez un message décrivant succinctement les modifications apportées. Finalement vous devez poussez ces changement vers votre dépôt Github :

```
git push origin master
```

Vous devez à présent vous rendre sur la page de votre dépôt Github où se trouve le _fork_ de PythonFaqFr. Cliquez dans le menu en haut sur **Pull requests**. Sur cette page, cliquez sur _create a pull request_. Le système va vous présenter les différences entre votre dépôt et le dépôt d'origine. Vous pouvez changer le texte si nécessaire. Il suffit de cliquer sur le bouton _Create pull request_.