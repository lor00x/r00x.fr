+++
date = "2014-05-04T15:03:24+02:00"
draft = true
title = "r00x.fr: création d'un blog"
slug = "r00x-fr-creation-dun-blog"

+++

Bonjour

Après une courte présentation du blog, voici mon premier vrai article ! J'explique ici les étapes de mise en place de ce blog. Parce qu'il faut bien commencer par quelque chose, j'ai dû faire des choix sur lesquels je reviendrai peut-être un jour ;-)


### TLDR (Too Long, Didn't Read):
Pour les impatients et les pressés voici le résumé:


- blog: [Ghost](https://ghost.org/)
- serveur: [NodeJs](http://nodejs.org/)
- commentaires: [Disqus](http://disqus.com/)
- container: [Docker](https://www.docker.io/)
- reverse proxy: [Nginx](http://nginx.org/)
- certificat SSL: [CaCert](https://www.cacert.org/)
- config TLS/SSL: [conf Mozilla](https://wiki.mozilla.org/Security/Server_Side_TLS)
- hébergeur: [OVH Kimsufi dédié](http://www.kimsufi.com/fr/)

### Point de départ, le blog: Ghost

[Ghost](https://ghost.org/) est une plate-forme de blogging très récente: Quickstarter en Avril 2013, version 0.3 sortie en septembre 2013. L'auteur a voulu créer un blog très simple, par opposition à [Wordpress](http://fr.wordpress.org/) qui offre un (trop) grand nombre de fonctionnalités. De nombreux [thèmes](http://marketplace.ghost.org/) sont disponibles, les articles s'affichent très bien sur les supports mobiles, tout comme l'interface de rédaction. Cette dernière utilise le langage de balisage léger [Markdown](http://fr.wikipedia.org/wiki/Markdown) pour un formatage facile du texte. Elle offre également une vue séparée: édition à gauche, rendu final à droite:

![L'interface de rédaction de ghost](/images/2014/May/ghost_redaction_article.jpg)


Le projet est encore jeune et de nombreux [bugs et requests](https://github.com/TryGhost/Ghost/issues?state=open) ont été levés, espérons que les corrections arrivent bientôt, même si la version courante est déjà correcte :-) 


### Le serveur: NodeJS
Ghost fonctionne grâce à un serveur [NodeJS](http://nodejs.org/), qui est basé sur [v8](https://code.google.com/p/v8/), le moteur Javascript utilisé par Chrome. Sa programmation événementielle et asynchrone (appels non-bloquants) le rend très performant tout en ayant une empreinte mémoire et une consommation CPU faibles. C'est un candidat idéal au [problème du C10k](http://fr.wikipedia.org/wiki/C10k_problem). Le serveur étant codé en javascript, l'échange de données avec le client s'en trouve simplifié, en peut même envisager d'inclure la même librairie JS côté client et serveur.


### Le système de commentaires: Disqus
Autoriser les commentaires sur son blog permet aux lecteurs d'apporter leurs points de vue ou corrections aux articles postés. Le revers de la médaille, c'est qu'il autorise aussi les spams et les trolls. J'ai décidé d'activer les commentaires en passant par un service tiers: [Disqus](https://disqus.com/).

Le service s'intègre très bien dans les blogs les plus courants (Wordpress...) mais également dans Ghost au moyen d'un [simple ajout](http://help.disqus.com/customer/portal/articles/1454924-ghost-installation-instructions) dans un template. Il comprend un filtre anti-spam, et l'interface d'admin est très pratique: modération des commentaires, blacklist, statistiques, on peut même voir [les commentaires](https://disqus.com/r00xfr/) qu'un utilisateur a fait sur tous les blogs utilisant Disqus... attention vous êtes tracés ! Ca permet surtout de découvrir facilement d'autres blogs traitant de sujets connexes.

Pour poster un commentaire les utilisateurs doivent utiliser leur compte Tweeter, Google, Facebook ou Disqus. Donc pas de base d'utilisateurs à gérer en local ! 

Le service est en [freemium](http://fr.wikipedia.org/wiki/Freemium). Il est utilisé par des sites à fort traffic comme CNN. Selon leur [blog](http://blog.disqus.com/post/50374065365/whats-cooler-than-a-billion-monthly-uniques), Disqus a passé le milliard de visiteurs uniques en Mai 2013 ! 

![Un milliard de visiteurs uniques !](http://media.tumblr.com/8965b650133e48864aa6cb50285304d2/tumblr_inline_mmpkreqXKn1qz4rgp.png)

### Container: Docker

Avoir un blog qui fonctionne c'est bien, mais c'est encore mieux si on le met en place en songeant aux problèmes que l'on rencontrera tôt ou tard:

- piratage
- surcharge
- défaillance du serveur physique
 
... et leurs conséquences directes:

- perte de données
- migration de serveur
- réinstallation du serveur et du blog

Un moyen d'adresser ces problématiques et d'utiliser un [serveur dédié virtuel](http://fr.wikipedia.org/wiki/Serveur_d%C3%A9di%C3%A9_virtuel) pour héberger le blog. Les avantages sont multiples:

- isolation: le processus du blog et ses données sont isolés du reste du système qui l'héberge. Cela permet de réduire ou retarder les dégâts que causerait un éventuel pirate en cas d'intrusion. Bien sûr le risque zéro n'existe pas, et un attaquant compétent et déterminé arrivera toujours à ses fins.
- sauvegarde: il est facile de sauvegarder une image du serveur virtuel. Il est ensuite aisé de restaurer le blog après un crash serveur ou un piratage, mais aussi de le déplacer sur une autre machine plus puissante.

L'inconvénient majeur d'un serveur d'un serveur virtuelle






