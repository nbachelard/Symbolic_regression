####Discussion le 15/11/2024####

Partant de la liste d'éléments (numériques et symboliques) pon produit un vecteur token numérique (de grande dimension). Choisir sa taille (dimension du plongement).

Mécanisme du transformer (attention is all you need) :
ex : le chat est blanc, il s'appelle Edgar. On détermine que le fait de s'appeler Edgar est attribué au chat (qui par ailleurs est blanc). Portée variable, ce peut être le suivi d'un personnage d'un bout à l'autre d'un livre. Ici c'est un opérateur qui porte sur plusieurs sous-assemblages récursifs de symboles.

Multicouche.

Encodage, décodage.

Rq fondatrice : au début de la régression symbolique à NN on utilise de simple réseaux denses, remplaçant les fonctions d'activation par les différents opérateurs que l'on autorise dans l'expression de f. Mais des fois le gradient est trop petit, la division. Aussi, on doit entraîner un nouveau réseau à chaque fois que l'on change autre chose que les constantes multiplicatives (ie. à chauqe fois que l'on miodifie le squelette du réseau).
On peut néanmoins essayer ça en réduisant le dico.

Encodage de la position : l'ordre compte : a/b et b/a c'est pas pareil. Par contre les couples d'entrée sont permutables entre eux.

Minimisation ensuite d'une fonction de cout, par exemple Xentropy. Quantifier overfitting.

Lien des tutos :
<https://arxiv.org/pdf/1610.02995> pour fitter un pendule simple (!)
<https://medium.com/@bavalpreetsinghh/transformer-from-scratch-using-pytorch-28a5d1b2e033> transformer un peu bourrin
<https://buomsoo-kim.github.io/attention/2020/04/21/Attention-mechanism-19.md/> transformer pas à pas

