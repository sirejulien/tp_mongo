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

Mongo DB peut également stocker un grand nombre de données différentes: chaîne de caractères, nombre entier ou décimal, booléen, date et heure et des script JS. On peut également stocké des coordonnées géographiques grâce au type geoJson afin d'effectuer des requête concernant, par exemple, la proximité des clients avec les restaurants qu'ils fréquentent. 
src: https://docs.mongodb.com/manual/reference/bson-types/ ; https://docs.mongodb.com/manual/reference/geojson/

Pour les fichiers inférieurs à 16Mo il est possible de les stocker en base 64 mais pour les autres on peut utiliser GridFS.
GridFS est un processus permettant de stocker les fichiers en les découpant en morceaux (*chunks*) de 4Mo et en les stockant dans une collection GridFS dédiée à cela. src: https://docs.mongodb.com/manual/core/gridfs/

Enfin Mongo DB étant un SGBD populaire il possède une documentation bien fournie et une communauté en ligne active permettant d'obtenir assez rapidement de l'aide sur les éventuels problèmes techniques rencontrés.

Un avantage indéniable de Mongo DB est la mise à disposition d'outils efficaces pour profiter au mieux des possibilités offertes par Mongo DB.
Pour la mise en place de la base nous utilisons **Atlas** permettant de facilement créer le cluster et de s'y connecter. Pour consulter et agir sur la base nous utilisons **MongoDB Compass**. Cet un logiciel qui fournit un interface graphique utilisateur simple d'utilisation pour tester des requêtes ou administrer la base de données. De plus on y trouve également une console en ligne de commande **MongoSH** pour agir sur la base. Nous pouvons également cité **Mongo Chart** qui permet de facilement créer des graphiques à partir des données stockées.

Une alternative à Mongo DB aurait été une base de donnée relationnelle basé sur du SQL comme Oracle ou SQL Server de Microsoft. Ces bases de données se base sur des tableaux à 2 dimensions (*tables*) ayant des relations logiques entre eux afin d'organiser les données. Ces bases de données sont privilégiées pour des bases complexes mais nécessite une vision d'ensemble en amont car elles sont assez peu évolutives (*scalable*) puisqu'elle se base sur un schéma fixe. De plus il est plus compliqué d'augmenter le nombre de serveur sur lesquelles est déployée la base. Ainsi nous choisissons une base Mongo DB pour cette application car elle sera amenée à évoluer au fur et à mesure des besoins du marketing et sera potentiellement amenée à être deployé sur de nombreux serveurs.

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

Cet exemple nous permet également d'aborder la fonction *explain()* de Mongo DB. Cette fonction permet de renvoyer des informations concernant le plan d'exécution d'une requête ou sur l'exécution en elle-même de la requête. Cette fonction aide grandement pour optimiser la base. Dans notre cas elle permet de justifier l'utilité des index pour notre base.
src: https://docs.mongodb.com/manual/reference/operator/meta/explain/

### Fonctions Basiques Mongo DB (Find, Insert, Update et Delete)

- Find
Nous avons déjà montré la fonction *find()* lors de la partie précédente sur les index. Elle permet de récupérer des documents dans une collection selon les filtres mis en paramètres. Dans notre exemple nous recherchions tous les clients masculins (SEXE:"H"), cependant la console n'a pas affiché les documents trouvés puisque nous avons utilisé la fonction *explain()* pour récupérer le temps d'exécution de la requête.

- Insert
La fonction *insert()* permet de créer un document dans la collection selon les paramètres données.

![requete insert](/Img_README/insert.png)

![resultat insert](/Img_README/result_insert.png)

On observe que le document a bien été créé avec les valeurs données en paramètre pour chaque champs.

Cette fonction est divisée en 2 sous-fonctions *insertOne()* et *insertMany()* permettant d'insérer un ou plusieurs documents dans la collection.

- Update
La fonction update() permet de modifier une collection.
Elle est divisée en 2 fonctions updateOne() et updateMany() qui permettent respectivement de modifier un ou plusieurs documents avec une requête.
exemple: 

![requete find client](/Img_README/find_ronnica.png)

On  peut voir que le champ libellé de l'adresse est vide.

![requete update client](/Img_README/update_ronnica.png)

Avec la fonction updateOne() on a mis à jour le champ libellé du document.

On peut passer des options à la fonction si par exemple l'on veut faire de l'upsert (créer le document s'il n'existe pas).
A la différence de l'insert, l'upsert va vérifier si un document correspond aux critères de sélection et va le modifier le cas échéant alors que l'insert, n'ayant pas de critères de sélection va créer le document sans se soucier des autres documents présents dans la collection. Cela peut entrainer des doublons dans la base si les requêtes sont mal faites. 

*updateOne()* met à jour le premier document correspondant aux critères de sélection alors que *updateMany()* va mettre à jour tous les documents correspondants aux critères.
src:https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/ ; https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/

- Delete
La fonction delete() permet de supprimer des documents au sein d'une collection.
Elle est également divisée en 2 fonctions deleteMany() et deleteOne().
exemple: 

![requete delete client](/Img_README/delete_ronnica.png)

On voit que le document précédement modifié a été supprimé de la collection.

## Utilisation

### Requête Géospatiale

![requete geospatiale](/Img_README/query_geo.png)

Avec une telle requête nous pouvons recherché les restaurants à proximité de l'adresse d'un client. Cela nécessite bien évidemment de traduire les adresses en coordonnées géographiques.
Dans cet exemple, grâce à la fonction *$nearSphere*, Mongo DB va nous retourner les restaurants triés selon leur proximité avec la position du client.

Cela peut servir pour envoyer des promotions ciblés géographiquement au client par rapport à son addresse ou alors, dans le cadre d'une application mobile, se servir des données GPS du téléphone du client pour lui indiquer le restaurant le plus proche de sa position actuelle.

### Agrégations

- Divisions des clients par tranche d'âge et de sexe:

![requete age/sexe](/Img_README/query_age_sexe.png)

Cette requête permet de retourner tous les clients masculins agé de 20 à 30 ans inclus. En faisant varier les critères de sélection il est facilement possible de diviser la clientèle en groupe en vue d'un marketing ciblée.

![count age/sexe](/Img_README/count_age_sexe.png)

Cette requête permet de compter le nombre de clients au sein de ce groupe en combinant les fonctions *$match* et *$count*. On appelle cet exécution de tâche à la suite une agrégation. C'est un avantage de Mongo DB car elle peut rendre des requêtes compliquées plus lisibles qu'en SQL.

- Nombre de commandes par client et par restaurant:

![count cmd/client/restau](/Img_README/nbre_commandes_client.png)

Cette requête récupère toutes les commandes liées à un restaurant par rapport à son *ObjectId* dans le *$match*.
Par la suite elle groupe les commandes restantes par rapport à leur *CLIENT_ID* et renvoie le nombre de commande par *CLIENT_ID*.
Cette requête permet donc d'identifier les clients les plus fidèles par restaurants. Cette informations peut être utilisée pour savoir quelle type de clientèle est la plus fidèles ou pour savoir quels sont les restaurants réussissant le mieux à fidèliser leur clientèle.

Bien d'autres applications peuvent être faites des agrégations Mongo: quelles sont les produits les plus populaires au sein d'un groupe de client? quelle est la différence d'affluence des restaurants selon les jours de la semaine ou les périodes de l'année?...

De plus, grâce au *scale up* possible de Mongo DB il sera facilement possible de rajouter des données pour des statistiques différentes ou plus précises.

## Conclusion

Nous vous proposons donc une solution basée sur une base de données Mongo DB pour son évolutivité et pour sa simplicité de mise en place et d'utilisation, notamment grâce aux outils Mongo DB.
De plus nous avons vu que les agrégations Mongo permettent de rechercher tous les statistiques pertinentes pour un marketing efficace.

Ainsi une solution logiciel pertinentes serait de créer une application en 2 parties, une partie pour récupérer les informations (clients grâce à un compte fidélité et restaurants grâce à un outil d'administration de la chaîne d'enseigne) et une seconde partie permettant au service marketing d'obtenir toutes les informations qu'il désire pour mettre en place de campagnes publicitaires. Ce second outil peut également être un plus pour le management afin de voir l'impact de telle ou telle décision sur le profit d'un restaurant. 

## Annexe

### Exemples de requête

Requête pour afficher tous les documents dans les restaurants de la collection

$match: {
    }

Requête pour afficher les champs restaurant_id, name, borough et cuisine pour tous les documents de la collection restaurant.
$project: {
        restaurant_id: 1,
        name: 1,
        borough: 1,
        cuisine: 1,
    }
 
Requête pour afficher les champs restaurant_id, name, borough et cuisine, mais exclure le champ \_id pour tous les documents de la collection restaurant. 

$project: {
        	restaurant_id: 1,
        	name: 1,
        	borough: 1,
  	cuisine: 1,
	\_id :0,
    }
 
Requête pour afficher les champs restaurant_id, nom, arrondissement et code postal, mais excluez le champ \_id pour tous les documents de la collection restaurant.

 $project: {
        restaurant_id: 1,
        name: 1,
        borough: 1,
        "address.zipcode": 1,
        _id: 0,
    }

Requête pour afficher tous les restaurants qui sont dans l'arrondissement du Bronx. 
 
 $match: {
        borough: "Bronx",
    }

Requête pour afficher les 5 premiers restaurants qui se trouvent dans le quartier du Bronx.

[{
    $match: {
        borough: "Bronx",
    }
}, {
    $limit: 5
}]
 
Requête pour afficher les 5 restaurants suivants après avoir sauté les 5 premiers qui sont dans le quartier du Bronx.

[{
    $skip: 5
}, {
    $match: {
        borough: "Bronx",
    }
}, {
    $limit: 5
}]
 
Requête pour trouver les restaurants qui ont obtenu un score supérieur à 90. 

 [{
$match: {
"grades.0.score": {
$gt: 90
}
}
}]

Requête MongoDB pour trouver les restaurants qui ont obtenu un score supérieur à 80 mais inférieur à 100. 

[{
    $match: {
        "grades.0.score": {
            $gt: 80,
            $lt: 100
        }
    }
}]
 
Requête MongoDB pour trouver les restaurants qui se situent dans une latitude inférieure à -95,754168.

 [{
    $match: {
        "address.coord.1": {
            $lt: -95.754168
        }
    }
}]
