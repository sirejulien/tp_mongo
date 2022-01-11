# TP Mongo DB

Mise en place d'une base de donnée Mongo DB dans le cadre d'une application de fidélisation client pour une chaine de restaurant.

###Proposition Commerciale

Mise en place d'un système de compte client pour permettre l'historisation des commandes par clients en vue d'un marketing ciblé.
On pourra inciter les clients à se créer un compte client en offrant des promotions par mail pour les clients inscrits.
Passé un certain nombre de clients inscrits il sera possible de faire des statistiques permettant de ciblé différents types de clientèle afin d'affiner le marketing pour fidéliser un type de clientèle ou attirer un nouveau type de clientèle. 

###Choix de Mongo DB

Mongo DB ne possédant pas de schéma fixe, contrairement aux SGBD basés sur le SQL, il possèdent l'avantage de pouvoir facilement être modifiable. On pourra donc aisément ajouté de nouveaux champs à nos collections si par exemple le marketing désire conserver une nouvelle donnée sur les clients.
La notion d'Objet et de Sous-Objet propre à Mongo DB est également un avantage par rapport aux SGBD relationnels classiques puisqu'elle élimine le besoin en table de jointure. Cela simplifie grandement la structure de la base de données.
Enfin Mongo DB étant un SGBD populaire il possède une documentation bien fournie et une communauté en ligne active permettant d'obtenir assez rapidement de l'aide sur les éventuels problèmes techniques rencontrés.

###Structure de la base de donnée

![structure bdd](/Img_README/struct_db.png)
