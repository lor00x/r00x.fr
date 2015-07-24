+++
date = "2014-09-13T23:57:32+02:00"
draft = false
title = "Golang"
slug = "go-go-go"

+++

Bonjour à tous !

Je réfléchissais récemment aux différents langages de programmation que j'ai eu l'occasion de pratiquer, et je souhaitais vous faire part du petit nouveau, que j'espère bientôt rajouter sur mon CV ^_^.

# Ils sont légions 

Avez-vous déjà réfléchi au nombre de langages de programmation auxquels vous avez déjà eu affaire ? Pour ma part cela se résume ainsi:

* J'ai appris le *BASIC* dès l'âge de 6 ans (hé oui !), ce qui a grandement joué sur mon avenir...
* En école d'informatique, on m'a formé aux langages "courants": *Assembleur, C, C++, Java*.
* Une fois débarqué en milieu professionel j'ai dû apprendre sur le tas d'autres langages, principalement: *Perl, PHP, HTML, CSS, Javascript*.

Et de nombreux autres à des degrés de connaissance variés: *68000, Scheme, Lisp, Python, Groovy, Ruby, Powershell, Bash, Matlab...*

![](/images/2014/Sep/mini_bd.jpg)
# A la recherche du langage

Après plusieurs années de PHP je souhaitais me refaire la main sur un autre langage mais je ressentais une certaine frustration vis à vis de ce que proposaient les autres: arithmétique des pointeurs, gestion de la mémoire, fichier headers (.h) ainsi qu'un syntaxe complexe pour le C et le C++. Quand à Java, le côté verbeux, l'empreinte mémoire et la machine virtuelle me rebutaient. Ces raisons sont discutables et constituent uniquement mon ressenti global. 

Exit donc les langages classiques, il me fallait quelque chose de neuf...

# Go !

C'est là que j'ai découvert [Golang](http://en.wikipedia.org/wiki/Go_%28programming_language%29) (Go pour les intimes). Créé par Google, et annoncé officiellement en 2009, ce langage a pioché les idées dans divers des langages cités plus haut. Il en offre, selon moi, un très bon compromis:

 * La compilation produit un binaire unique sans dépendance. La phase d'édition de liens se fait donc de manière statique. La compilation s'effectue en un temps record grâce à une syntaxe simplifiée.
 
 * La syntaxe est très simple, à l'opposé du *"More than one way to do it"* du Perl ou des syntaxes multiples d'autres langages. Par exemple, Golang propose un mot-clé unique `for` pour les boucles (pas de `foreach`, `while`, `do while`, `do until` ...). Les points-virgules `;` en fin de ligne sont absents (en fait, ils sont ajoutés automatiquement à la compilation), tout comme le signe `$` cher à PHP et Perl devant les variables. Les accolades `{ }` délimitent toutefois les blocs de code, mais les parenthèses `( )` ne sont pas obligatoires pour les clauses `if`.
 
 * Le langage est fortement typé. Les types proposés de base sont: entier signé/non-signé de 8 à 64 bits `uint8`, `int8`... `int64`, réel `float32`, `float64`, booleen `bool`, chaine `string`, rune `rune` (charactère UTF-8), tableau (taille statique, par ex: `[5]int`) et "slice" (taille dynamique, par ex: `[]int`). A noter aussi en natif deux types permettant de manipuler les nombres complexes `complex32`, `complex64`. Les types composés sont déclarés au moyen de `struct` similaires au C/C++. La plupart des erreurs de typage sont donc détectées à la compilation.
 
 * La programmation objet est également très simplifiée: déclaration de types, d'interfaces et de méthodes... et c'est tout ! Pas de classe abstraites, pas d'héritage (simple ni multiple), pas de surcharge de méthodes. Fichtre ! Mais comment vais-je donc mettre en oeuvre mes design pattern ? Haha ! Pas d'inquiétude, on peut très bien s'en sortir :-) Golang permet ainsi l'enfouissement de type, ce qui permet une forme d'héritage.
 
 * La gestion de la mémoire: les pointeurs sont autorisés (`*int, &var`) mais pas leur arithmétique, les variables doivent être déclarées, les tableaux peuvent être déclarés avec une taille statique ou dynamique (slices). L'allocation de la mémoire se fait donc automatiquement tout comme la libération qui est gérée par un ramasse-miettes, mais les pointeurs nous laissent décider du passage par valeur ou par référence.

On part donc sur de bonnes bases, mais ce n'est pas tout !

# Les petits plus

Golang présente nativement des fonctionalités que d'autres langages permettent seulement au prix d'outils ou de librairies annexes:

 * Go intègre nativement la programmation concurrente. Un `go mafunction()` permet de lancer une fonction de manière asynchrone. Pour communiquer entre les "goroutines", on utilise les "channels" qui sont des canaux de communication fortement typés. Cela permet de faire de la programmation asynchrone de manière intuitive et lisible (pas d'utilisation de callback comme avec Nodejs).
 
 * La ligne de commande permet de formater un programme Golang selon un standard unique lisible par tous, proche du [style K&R](http://fr.wikipedia.org/wiki/Style_d%27indentation#Style_K.26R). D'autres langages ou IDE le proposent mais cette fonctionnalité prend tout son sens avec la syntaxe simplifiée: le code est concis et lisible par tous, même si certains grogneront à cause de l'accolade à la ligne ;-)
 
 * On peut exécuter directement un fichier golang, par exemple: `go run test.go`. La compilation sera effectuée implicitement. Voilà qui fera plaisir à ceux qui ont l'habitude des langages de script.
 
 * Une commande `godoc` permet de générer la documentation à partir du code et même de lancer un serveur web pour la consulter directement.
 
 * Une commande permet `go test` de lancer les tests unitaires que vous n'aurez pas manqué d'écrire ;-) Un ajout de l'option `-test.bench="Benchmark*"` permet de lancer un benchmark sur toutes les fonctions dont le nom correspond au schéma spécifié.
 * Goland intègre également un outil de profiling. Cet outil dispose d'un affichage du graphes des appels de fonctions, ce qui permet en un coup d'oeil de se faire une idée du flux d'exécution et de détecter les goulots d'étranglement (voir [ici](http://blog.golang.org/profiling-go-programs)).
 
La liste n'est pas exhaustive, des astuces de programmation rendent la vie plus facile. Par exemple, bien que l'on puisse déclarer des interfaces, le mot-clé `implement` n'existe pas, car tout type dont la signature des méthodes correspond à une interface va automatiquement implémenter cette interface. Autre cas pratique: une méthode peut être appelée indifféremment sur un type ou sur un pointeur de type, Golang traduira automatiquement l'appel sans qu'on ait à jouer avec `*` ou `&`.
 
# Pour aller plus loin

Pour ceux qui ne connaissaient pas Golang, j'espère avoir titillé votre curiosité. Je vous conseille d'aller jeter un coup d'oeil sur le [site](http://golang.org/) et le [blog](http://blog.golang.org/) officiels.

De nombreux projets Golang voient le jour sur Github et autres plate-formes de partage de code. Je citerais parmi eux [Docker](https://github.com/docker/docker), un système de virtualisation par containers que j'utilise d'ailleurs pour héberger ce blog.

# Place à l'action

Tout ça c'est bien beau, mais la maîtrise d'un langage ne s'acquiert que par la pratique. Il me fallait donc un projet pour débuter la programmation en Golang. Un de mes amis souhaitant lui aussi se lancer dans cette aventure, nous avons décidé de nous pencher sur le protocole LDAP et de le réimplémenter chacun de notre côté.

Mais ceci fera l'objet du prochain article ;-)




