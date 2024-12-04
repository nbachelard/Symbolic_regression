###Fabrication dataset###

-vérifier quelle sont les conditions de rejet des expressions : en plus de l'échec de l'évaluation en un des points du dataset, il y aurait un critère sur la longueur (en temps ?) de l'évaluation/simplification par sympy.

-voir quelles sont les conséquences, dans le cas d'échec d'évaluation de la fonction en un des points tirés, de juste rejeter le point au lieu de rejeter la fonction.

-au début, on peut simplement virer les opérations menant à des échecs : racine, log.

-déterminer les stats pertinentes sur le jeu de données

-essayer une méthode récursive simple de génération d'arbres, et montrer qu'elle est biaisée. les exemples donnés : biais droite/gauche (compter le nombre de branches binaire-binaire droites sur celui de branches binaire-binaire gauches) ; profondeur vs. largeur.
ATTENTION Lample 2019 prétend qu'à nombre de binaires donné, celui full gauche, celui full droite, le plus profond et le plus large doivent avoir la même proba. Ça me semble assez peu clair, et serait là encore à étayer par des données.

-choix des opérateurs. Il semble pertinent là encore de regarder un jeu de données de référence.
Il faudra aussi voir comment ce choix se traduit dans les arbres finalement retenus (ex : soustraction).
Noter que Kamienny prend séparément ^2, ^3, tandis que chez Lample la puissance est un opérateur binaire. La racine est dans tous les cas isolée.
Essayer de remplacer l'opposé par -1*.

###FNCC###
-trouver le code
-trouver des datasets (on peut commencer par ceux du papier qui est assez détaillé quant aux poids du réseau une fois optimisé).

###Tranformer###
-positional encoding est crucial au sein de l'expression de f, pas dans le dataset de départ (tenir compte de la permutabilité des points d'entrée).
-Validation  : facteurs de difficulté : nombre d'opérateurs binaires, nombre d'unaires, taille de l'entrée, nombre de points indicateurs (dans N/4, N/2, 3N/4, N), extrapolation performance, (variance de l'entére excède 1), Robustness to noise (varying the multiplicative noise added to the labels).
-dépendance de complexité en la taille des phrases

DONE list :

-y aurait une histoire dans l'argument des opérateurs unaires (genre il y aurait cos72 mais pas cos72x+34). Un pb à l'étape d'insertion des unaires.

-nature de l'out de sympy (en faire un string ou une liste en notation polonaise).
Sur ce point, j'ai de plus en plus l'impression que la non-performance de scipy nous fait, au global, perdre du temps. Au-delà de la possibilité d'obtenir de jolies expressions pour les afficher au tableau, je crois que l'intérêt de ce truc est surtout de réaliser de sstatistiques sur la représentation des différents opérateurs/formes d'expressiosn (ex typique : extraire les puissances entières des successions de produits).
En fait il existe surement un tradeoff entre simplification ciblée et longueur des évaluations individuelles. Comme on travaillera en basse dimension (?) je suppose qu'il sera plus rapide de juste faire des évaluations débiles/.
