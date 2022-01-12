# TP Mongo DB

Mise en place d'une base de donnée Mongo DB dans le cadre d'une application de fidélisation client pour une chaine de restaurant.

## Proposition Commerciale

Mise en place d'un système de compte client pour permettre l'historisation des commandes par clients en vue d'un marketing ciblé.
On pourra inciter les clients à se créer un compte client en offrant des promotions par mail pour les clients inscrits.
Passé un certain nombre de clients inscrits il sera possible de faire des statistiques permettant de ciblé différents types de clientèle afin d'affiner le marketing pour fidéliser un type de clientèle ou attirer un nouveau type de clientèle. 

## Choix de Mongo DB

Mongo DB ne possédant pas de schéma fixe, contrairement aux SGBD basés sur le SQL, il possèdent l'avantage de pouvoir facilement être modifiable. On pourra donc aisément ajouté de nouveaux champs à nos collections si par exemple le marketing désire conserver une nouvelle donnée sur les clients.

La notion d'Objet et de Sous-Objet propre à Mongo DB est également un avantage par rapport aux SGBD relationnels classiques puisqu'elle élimine le besoin en table de jointure, par exemple les adresses divisées en "Libellé/Ville/CodePostal". Cela simplifie grandement la structure de la base de données et donc aussi les requête et leur efficacité.

![exemple document client](/Img_README/client.png)

Comme nous pouvons le voir l'exemple de document possède un objet *Adresse* divisé en Sous-Objets pour stocker les différents champs de l'adresse.
Une application pertinente des objets Mongo dans notre cas serai de stocker les notes reçu par un restaurant.

![exemple document restaurant](/Img_README/ex_restau.png)

Les notes sont stockées les unes après les autres au sein de l'objet *Grades*. Cela évite la création d'une collection GRADES avec une jointure nécessaire lors de la consultation des notes d'un restaurant.

Mongo DB peut également stocker un grand nombre de données différentes: chaîne de caractères, nombre entier ou décimal, booléen, date et heure et des script JS. src: https://docs.mongodb.com/manual/reference/bson-types/

Pour les fichiers inférieurs à 16Mo il est possible de les stocker en base 64 mais pour les autres on peut utiliser GridFS.
GridFS est un processus permettant de stocker les fichiers en les découpant en morceaux (*chunks*) de 4Mo et en les stockant dans une collection GridFS dédiée à cela. src: https://docs.mongodb.com/manual/core/gridfs/

Enfin Mongo DB étant un SGBD populaire il possède une documentation bien fournie et une communauté en ligne active permettant d'obtenir assez rapidement de l'aide sur les éventuels problèmes techniques rencontrés.

avantage des outils mongo db (mongo chart)

alternative à mongo

## Structure de la base de donnée

![structure bdd](/Img_README/struct_db.png)

Comme nous pouvons le voir nous partons sur une base de données composées de 4 collections:
- 1 collection RESTAURANTS listant les restaurants et leurs adresses.
- 1 collection CLIENTS listant les comptes clients et toutes les informations personnelles que nous pouvons collecter.
- 1 collection PRODUITS listant la carte de nos enseignes.
- 1 collection COMMANDES qui fait le lien entre les 3 collections précédentes en indiquant également la date et l'heure de la commande et en indiquant également les éventuelles réductions.

La collection commande aurait pu être un objet au sein de la collection CLIENTS, voire de la collection RESTAURANTS, mais nous avons préféré choisir cette solution pour simplifier la structure de la base. De plus certaines statistiques ne dépendront pas des clients, par exemple: quels sont les restaurants les plus actifs le week-end? ou quels sont les produits les plus commandés le soir?

Une amélioration possible serait de créer une collection REDUCTION listant les détails des différentes promotions proposées dans nos enseignes. Cela permettrait de faire des statistiques sur le succès de ces différentes promotions afin de quantifier l'impact de telle ou telle campagne publicitaire.

## Partie Technique

### Index

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

Dans notre application nous pourrons mettre dans un premier temps des index sur les champs suivant:
-CLIENTS: sexe, date de naissance.
-COMMANDES: restaurant_id, client_id, date, produit.

Ces index accélererons le processus pour les statistiques les plus importantes d'un point de vue marketing (tri des clients par sexe et tranche d'age, statistique de consommation de ces groupes, popularité des produits, différence de fréquentation des restaurants).

Cet exemple nous permet également d'aborder la fonction **explain** de Mongo DB. Cette fonction permet de renvoyer des informations concernant le plan d'exécution d'une requête ou sur l'exécution en elle-même de la requête. Cette fonction aide grandement pour optimiser la base. Dans notre cas elle permet de justifier l'utilité des index pour notre base.
src: https://docs.mongodb.com/manual/reference/operator/meta/explain/

### Fonctions Update et Delete

- Update
La fonction update() permet de modifier une collection.
Elle est divisée en 2 fonctions updateOne() et updateMany() qui permettent respectivement de modifier un ou plusieurs documents avec une requête.
exemple: 

![requete find client](/Img_README/find_ronnica.png)

On  peut voir que le champ libellé de l'adresse est vide.

![requete update client](/Img_README/update_ronnica.png)

Avec la fonction updateOne() on a mis à jour le champ libellé du document.

On peut passer des options à la fonction si par exemple l'on veut faire de l'upsert (créer le document s'il n'existe pas).
UpdateOne() met à jour le premier document correspondant aux critères de sélection alors que updateMany() va mettre à jour tous les documents correspondants aux critères.
src:https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/ ; https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/

-Delete
La fonction delete() permet de supprimer des documents au sein d'une collection.
Elle est également divisée en 2 fonctions deleteMany() et deleteOne().
exemple: 
![requete delete client](/Img_README/delete_ronnica.png)

On voit que le document précédement modifié a été supprimé de la collection.

utilité requete géospatiale, discuter structure geoJson, illustré avec exemple geochart


## Utilisation

agrégations

nombre de clients différents par restaurants
nombre de commandes par restaurant par periode donnée
produit les plus commandés par périodes
divisions des clients en tranche d'age et sexe
liste des restaurants dans un rayon par rapport à un client
