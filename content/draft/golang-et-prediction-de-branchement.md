+++
date = "2014-09-14T16:50:40+02:00"
draft = true
title = "Golang et prédiction de branchement"
slug = "golang-et-prediction-de-branchement"

+++

# Langage machine pour tous !
La programmation haut niveau (JAVA, C#, PHP) donne souvent une impression de facilité par rapport à des langages plus proches du système (assembleur, C, C++). C'est oublier que, même à haut niveau, votre code finira transformé en langage machine avant d'être exécuté. La structure de votre code va influencer le compilateur et le langage machine qu'il produit.


# L'algorithmique...

Les performances de votre programme pourront varier fortement suivant la manière dont il est codé. C'est la raison pour laquelle il existe, par exemple, de nombreux [algorithmes de tri](http://en.wikipedia.org/wiki/Sorting_algorithm) (tri rapide, tri à bulle...). Chacun ayant ses avantages (vitesse de traitement élevée en moyenne) et ses inconvénients (forte occupation mémoire, adapté à certains type d'éléments à trier, traitement infiniment lent dans le pire des cas...). 

Mais votre algorithme peut-être le meilleur, il pourra finir bon dernier s'il n'est pas exécuté sur l'architecture adéquate.

# ...ne suffit pas

Car on a tendance à oublier (ou ignorer) l'architecture de la mémoire et du microprocesseur qui vont servir à stocker et exécuter le code.

La mémoire est [hiérarchisées en niveaux](http://fr.wikipedia.org/wiki/Hi%C3%A9rarchie_de_m%C3%A9moire): registres, cache processeur, RAM, disque dur, stockage externe, réseau... Chaque niveau a Plus on s'éloigne du processeur, plus la capacité de mémoire disponible et le temps d'accès aux données augmente.

Le microprocesseur, quant à lui, utilise divers méthodes pour optimiser le traitement des instructions: [pipeline d'instructions](http://fr.wikipedia.org/wiki/Pipeline_%28informatique%29), [prédiction de branchement](http://fr.wikipedia.org/wiki/Pr%C3%A9diction_de_branchement), [exécution dans le désordre](http://fr.wikipedia.org/wiki/Ex%C3%A9cution_dans_le_d%C3%A9sordre)...

# Yes you can

Aujourd'hui, les ordinateurs sont assez rapides pour que beaucoup de programmes aient des performances acceptables sans que le programmeur ait à se soucier des détails techniques de l'architecture sous-jacente. L'optimisation est une discipline exigeante qui requiert du temps et de l'expérience.

Ceci dit même en programmant sur un langage haut niveau, vous pouvez influer sur l'exécution de votre programme pour en améliorer grandement les performances. 


 


http://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-an-unsorted-array?answertab=active#tab-top

http://stackoverflow.com/questions/9936132/why-does-the-order-of-the-loops-affect-performance-when-iterating-over-a-2d-arra


