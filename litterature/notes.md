####Notes sur la littérature####

$$$Kamienny et al 2022$$$

Note : SRBench

**Intro**
pour y = f(x) deux familles d'approches :
-statistiques paramétriques : simple mais impose guess sur f
-ML : sujet à overfitting, problèmes d'interprétabilité

SR prend le meilleur des deux. Usuellement décomposé en deux étapes : choix d'un squelette composant des fonctions d'un certain ensemble. Puis ajustement des constantes (BFGS algorithm).

Typical approach = GP. At each iteration consider a range of candidtates, apply BFGS to each of them them select the fittest and make it mutate. Accurate but slow to optimize, and no learning.

Seminal ML approaches to SR problems conserve this 2-step decomposition. Which is an issue in teerms of performance as two skelettons can be closer one to another than a unique one with two sets of constants. Also issue of optimisation (loss potentially nonconvex). ML tend to overfit (see fig 11). Here is performed E2E (end-to-end) regression, at the cost of using a hybrid symbolic-numeric vocabulary (-> so I guess this means a LOT of tokens). Can then refine constants with BFGS ("mitigating nonlinear optimisation issues").

Fig 1 et 11 compare les différentes approches. Sachant que c'est un peu de la triche de compter ML qui ne fait pas vraiment de la régression symbolique (la solution n'en n'est pas interprétable).

Tests : robustness to noise, extrapolation capabilities.

**Construction d'un jeu de données**

Méthode Lample. On ne prendra pas la division, moins fréquente que les 3 autres opérations, mais plutôt l'opérateur unaire inv.
Leur truc contraignant le nombre d'opérateurs binaires par celui de variables de la fonction est un peu bizarre, il doit être justifié par une caractéristique des datasets "réalistes". A creuser en tout cas, on peut pas juste faire comme eux. A noter qu'ils font pas comme Lample pour le co-sampling des binary et unary, là aussi leur truc est tordu. Ils prennent toutes les premières variables (?? c'est pas clair) mais c'est pas grave car au pire on peu set des préfacteurs à 0.
Idem pour les stats sur la géométrie des arbres. A un moment ou à un autre il nous faudra des jeux de données réelles, jsp ce que Feynman vaut par exemple. Aller voir SRbench.
Y compris pour les biais de symétrie, en fait on n'est pas Dieu donc tout ce que l'on peut faire c'est prendre des jeux existants. Mais du coup le truc se mord la queue potentiellement (en matière d'extrapolation).

Pour le jeu de données on jette f dès que l'une des valeurs samplées est hors de son domaine de définition.

Choix des points : choisit k (dans [|0, kmax]|) centroides selon normale de centre 0, variance 1. Puis pour chaque sélectionne une variance. Et ensuite tire selon chacune une certaine fraction des points (pour que la somme sur les k centroides fasse à peu près un certain nombre N). Applique à chacune une transformation de Haar. Enfin whiten, ie. enlève la moyenne et fixe std=1 (puisque les variables interviendront sous la forme ax_i+b).

Tokenization/encoding : represent numbers with triplet sign-mantissa-exponent. Insert this in reversed polish notation (as in Lample), without any distinctive particularity.

Rk : encoder will only process numeric tokens. The embedding is a priori not shared, included for numeric tokens, btw encoder and decoder.

**Méthode**

Embedding : obtient N.(3D+1) tokens. Que tu embed en N.(3Dmax+1)d_emb avec un padding puis passage dans l'espace des embeddings. Tu passe ça dans un ReLU-FF à deux couches qui sort un truc de taille Nd_emb (pas critique donc autant faire ça).

Transformer : prend 16 têtes d'attention, d_emb=512. Encodeur à 4 couches, décodeur à 16 couches. Invariance par permutation des N points : pas d'importance de position.

Entrainement : le learning rate augmente puis diminue. On batch ensemble les données de N similaire pour éviter un padding excessif.

Inference : for accuracy E2E+BFGS/model > E2E > E2E+BFGS/random >~ skeleton+BFGS
POur travailler avec des données comparables à celles du dataset d'entrainement on scale à l'entrée de sorte à ce que la moyenne soit 0 et l'écart-type 1. On rescale accordingly en sortie.
Si >N points, bagging. Donne p candidats. Ne dit pas comment inférer un bon candidat depuis ceux-ci.

**Résultats**

Validation dans le domaine :
Facteurs de difficulté : nombre d'opérateurs binaires, nombre d'unaires, taille de l'entrée, nombre de points indicateurs (dans N/4, N/2, 3N/4, N), extrapolation performance, (variance de l'entére excède 1), Robustness to noise (varying the multiplicative noise added to the labels).

Évalue ensuite l'écart du guess sur N points, métriques : 
- R^2 qui est le pourcentage de la variance expliuqée (pour chaque fonction). Problème, c'est pas borné, donc des déconnants peuvent casser la somme. On peut donc en retirer les pire x%
- Acc_e, qui indique pour chaque fonction si oui ou non le plus grand écart est inférieur à e : attention c'est sensible aux y_i aberrants (à voir d'ailleurs comment on traite les exceptions), et donc fonction du nombre de y_i. On peut donc là encore choisir de retirer y% des pires fonctions.
ATTENTION selon la valeur de e les performances de différents modèles ne se classent pas dans le même ordre !

**App A**

"The probabilities of the unary operators were selected to match the natural frequencies appearing in the Feynman dataset" :
add:1, sub:1, mul:1
inv:5, abs:1, sqr:3, sqrt:3, sin:1, cos:1, tan:0.2, atan:0.2, log:0.2, exp:1
Métriques :
Input dimension (0-10)
Number of input pairs (0-200)
Number of binary ops (0-35 ??)
Binary operators (hist)
Number of unary ops (hist)
Unary operators (0-5)
C'est fait avec le cul mais retenir qu'il faut aller chercher de quoi comparer dans SRbench, ou Feynman. Il se passe des trucs bizarres, par exemple - apparait 3* plus rarement que +. Noter que les expressions échouant une évaluation sont supprimées, mais pas remplacées (on pourrait également mettre en place un quota par taille de l'entrée).
Pour les unaires : ^2>>sqrt>inv>cos>arctan>abs>^3>exp>sin>tan>log.

À noter que les pyuissnaces entières peuvent être obtenues avec de sbrancehs de multiuplication successives. Il faut donc considérer dans la proportion de leur apparition la correction de celles déjà générées par simple construction de l'arbre.


$$$Lample 2019$$$

**2.3 : generating random expressions**
La méthode naïve de création récursive d'arbre en donnant à chaque feuille une proba fixée de donner un nouveau noeud ou bien un unaire ou bien une feuille tendraient à favoriser les arbres profonds devant ceux larges ; à favoriser les left- or right-leaning.
-> Le montrer.
Soi-disant qu'à nombre de binaires donné, celui full gauche, celui full droite, le plus profond et le plus large doivent avoir la même proba.
-> Le montrer.

Formule donnant le nombre d'arbres contenant p1 unaires, p2 binaires, L feuilles.

Inconvénient du seq2seq : lors du décodage rien n'empêche le modèle de renvoyer une expression qui n'est pas un arbre :  Dyer et al. (2016) donne un moyen de biaiser le décodage pour éviter cela. Lample note cependant que de tels cas sont rares et peuvent en pratique être simplement ignorés.

Dans son cas les nombres sont des entiers, de la forme - 3 4 ou + 5 6 7 .

$$$Alnuqaydan 2023$$$

Complexité du transformer (?) scale quadratiquement avec la longueur des séquences manipulées (entrée ou sortie ?). Sachant que la dépendance de la taille du vecteur token en la qlongueur de la séquence n'est pas forcément linéaire non plus. Elle constitue un intéressant hyperparamètre.
Deux références sur des architectures présentant de meilleurs scalings :

- Zaheer M, Guruganesh G, Dubey A, Ainslie J, Alberti C, Ontanon S, Pham P, Ravula A, Wang Q, Yang Li and Ahmed A 2020 Big
bird: transformers for longer sequences (arXiv:2007.14062)
- Beltagy I, Peters M E and Cohan A 2020 Longformer: the long-document transformer (arXiv:2004.05150)

$$$Martiuus et al 2016 (operators-activated FCNN)$$$

L'objectif principal ici est l'extrapolation : on eentraînera un nouveau modèle pour chaque système physique. 

Note : au-delà de la science, fabriquer des opérations réalistes permet de généraliser avec pertinence hors du domaine connu (réponse à pourquoi ne se contente-t-on pas d'interpoler). Say for practical applications, eg robotics/.

Integrate typical physical operators as activation functions to allow extrapolation. Try sin, cos first (aloso sigmoid because canonical but will be disabled at learning phase since uncommon in physics (?)). Could also have used log, sqrt but definition. Radial basis functions ??
Linear NN can perform additions, (with PN constants), but not multiplications, which we allow separately. Only allow product of 2 since as this will allow conytrolling the max size of polynomials through network depth. (power laws < 2**d are handled by allowing 2-products) Also, way of limiting complexity. Allow linear activation so that can have less nonlineariies than the size of network.

Optimizer : SGD or Adam ? Minimize L2 loss + L1 regularization (favors small number of terms). Ajouter la norme L1 peut mener à verrouiller les valeurs des poids, voire les sous-estimer globalement. on mitige ceci en ne contraignant pas la norme L1 dans le premier quart du temps, puis en contraignant L0 et non plus L1 dans le dernier 20e (ainsi les plus petites valeurs de w_ij vont à 0 exactement).
Faire plusierus runs depuis plusieurs initialisations pour contourner le risque d'optima locaux non-globaux.

Construction du réseau. Pour chaque couche ou choisit un nombre de multiplications dans {1, 3, 5} (j'imagine donc qu'il est important qu'il soit impair), et 4 fois plus de cellules unaires : donc on équireprésente la multiplication, le sin, cos, sigmoide, linéarité (rq : le réseau est complètement différentiable sur le spoids des matrices et biais correspondant à chaque couche).
