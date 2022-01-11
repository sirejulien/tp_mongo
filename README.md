# TP Mongo DB

Mise en place d'une base de donnée Mongo DB dans le cadre d'une application de fidélisation client pour une chaine de restaurant.

##Proposition Commerciale

Mise en place d'un système de compte client pour permettre l'historisation des commandes par clients en vue d'un marketing ciblé.
On pourra inciter les clients à se créer un compte client en offrant des promotions par mail pour les clients inscrits.
Passé un certain nombre de clients inscrits il sera possible de faire des statistiques permettant de ciblé différents types de clientèle afin d'affiner le marketing pour fidéliser un type de clientèle ou attirer un nouveau type de clientèle. 

##Choix de Mongo DB

Mongo DB ne possédant pas de schéma fixe, contrairement aux SGBD basés sur le SQL, il possèdent l'avantage de pouvoir facilement être modifiable. On pourra donc aisément ajouté de nouveaux champs à nos collections si par exemple le marketing désire conserver une nouvelle donnée sur les clients.

La notion d'Objet et de Sous-Objet propre à Mongo DB est également un avantage par rapport aux SGBD relationnels classiques puisqu'elle élimine le besoin en table de jointure, par exemple les adresses divisées en "Libellé/Ville/CodePostal". Cela simplifie grandement la structure de la base de données.

Enfin Mongo DB étant un SGBD populaire il possède une documentation bien fournie et une communauté en ligne active permettant d'obtenir assez rapidement de l'aide sur les éventuels problèmes techniques rencontrés.

avantage des différents type de données stockable

avantage des outils mongo db (mongo chart)

alternative à mongo

##Structure de la base de donnée

![structure bdd](/Img_README/struct_db.png)

Comme nous pouvons le voir nous partons sur une base de données composées de 4 collections:
- 1 collection RESTAURANTS listant les restaurants et leurs adresses.
- 1 collection CLIENTS listant les comptes clients et toutes les informations personnelles que nous pouvons collecter.
- 1 collection PRODUITS listant la carte de nos enseignes.
- 1 collection COMMANDES qui fait le lien entre les 3 collections précédentes en indiquant également la date et l'heure de la commande et en indiquant également les éventuelles réductions.

La collection commande aurait pu être un objet au sein de la collection CLIENTS, voire de la collection RESTAURANTS, mais nous avons préféré choisir cette solution pour simplifier la structure de la base. De plus certaines statistiques ne dépendront pas des clients, par exemple: quels sont les restaurants les plus actifs le week-end? ou quels sont les produits les plus commandés le soir?

Une amélioration possible serait de créer une collection REDUCTION listant les détails des différentes promotions proposées dans nos enseignes. Cela permettrait de faire des statistiques sur le succès de ces différentes promotions afin de quantifier l'impact de telle ou telle campagne publicitaire.

##Partie Technique

###Index

Les index de Mongo DB sont une structures de données dont la fonction est stocker les différentes valeurs d'un champ selon un tri croissant ou décroissant et d'y joindre la liste des documents correspondants à cette valeur.

Cela permet d'accélérer grandement le processus de requête sur ce champs car Mongo DB ira lire son index afin de récupérer les documents correspondant à la requête plutot que de scanner l'intégralité des documents pour trouver quels documents possèdent la valeur cherchée sur ce champ.

src: https://docs.mongodb.com/manual/indexes/

exemple:
- Requête pour récupérer tous les clients masculins.
![requete sexe sans index](/Img_README/query_no_index.png)

- Création de l'index sur le sexe des clients.
![creation index](/Img_README/crea_index.png)

- Même requête avec index.
![requete sexe sans index](/Img_README/query_index.png)

Dans le premier cas la requête nous renvoie 500 000 documents en 790ms alors que dans le deuxieme cas (avec l'index) la requête nous renvoie le même nombre de documents en 534ms, soit une amélioration de 33%.

utilité et utilisation des fonctions update delete read

fonctions explain, pretty

utilité requete géospatiale, discuter structure geoJson, illustré avec exemple geochart