# Mini Projet Course de tortue

Etudiant: POITEVIN Louis
Numero : 11410541

# Description

Le programme python est découpé chronologiquement de la manière suivante : 
* Une partie acquisition des données, on recupère les données de la course avec un interval ~ 3sec (utilisation de sleep) sur le serveur distant
* Une partie nettoyage et modélisation de nos données avec mise en forme sous type de DataFrame (pandas)
* Une partie visualisation pour voir les tendances de nos données ainsi que voir les points de rattage de top.
* Une partie typage qui décrit les fonctions qui vont permettre d'identifier les différentes tortues selon leurs caractéristiques de vitesse et d'accélération
* Une partie detection qui traite nos données avec les fonctions de types


# Requirements

Mon programme à été conçu en python3, il est donc nécéssaire que nos variables d'environnement soient compatibles avec l'environnement de Spark. Ajoutez dans ~/.bashrc les lignes suivantes :

* <code> export SPARK_HOME=/chemin/vers/spark-3.0.1-bin-hadoop2.7 </code>
* <code> export PYSPARK_PYTHON=/usr/bin/python3 </code>

Puis faites un <code> source .bashrc </code>

Pour executer le programme : 
<code> spark-submit mini_projet.py </code>

Si cela ne fonctionne pas pour quelconque problème de version, il est possible de l'executer via :
<code> python3 mini_projet.py </code> 

Ou  :
* <code> cd /chemin/du/projet </code>
* <code> jupyter notebook </code>

### Le nombre de requêtes pour l'acquisition des données est fixée au nombre de 40.
C'est la variable N_request dans le code.

# Description fonction detection et principe d'algorithmique 

### detect_regular : Permet de detecter une tortue régulière.
Algorithme : Grâce à l'acceleration des tortues retrouvés grâce aux vitesses, nous regardons d'abord si la valeur la plus répétée (frequence max) est "0". 
Si c'est le cas alors c'est potentiellement une tortue régulière. 
Pour chaque valeur du tableau d'acceleration :  
    Si il y'a ratage de top entre deux requêtes, alors on prend pas cette valeur en consideration.
    Sinon on verifie qu'elle est bien égale à zero.

Paramètres retournées si c'est bien une tortue régulière : 
* Vitesse en nombre de pas entre deux tops : On prend la différence de vitesse entre deux tops


### detect_tired : Permet de detecter une tortue fatiguée.
Algorithme : Ici j'applique la valeur absolue sur le tableau d'acceleration des tortues et je verifie si dans un premier temps la valeur la plus fréquente n'est pas zero (pour eliminer le cas si c'est une tortue reguliere). 
Ensuite pour chaque valeur du tableau d'acceleration : 
    Si la valeur est égale a la valeur la plus fréquente alors on continue
    Sinon on regarde si cette valeur correspond à l'arret de la tortue ou sa fin de réaccélération de la manière suivante :
        Si c'est dû à un ratage de top, alors on continue
        Si la valeur à gauche est ou la valeur à droite est égale a la valeur la plus fréquente alors on continue
        Sinon ce n'est pas une tortue fatiguée

Paramètres retournées si c'est bien une tortue fatiguée :
* Vitesse initiale : On prend la premiere valeur du tableau de vitesse
* Rythme de (dé)croissance : Si c'est une tortue fatigué, on prend la valeur absolue de l'acceleration qui revient le plus fréquemment (les autres valeurs sont sensées êtres des moment de l’arrêt de la tortue)


    

### detect_cycle : Permet de detecter une tortue cyclique.
Algorithme : Tout d'abord on verifie que 0 n'est pas l'élément le plus fréquent dans le tableau d'accélération pour éliminer le cas si c'est une tortue régulière.
Ensuite on prend le tableau des vitesse et pour chaque valeur : 
Si on retombe sur notre premiere valeur du tableau alors c'est un cycle potentiel 
    Sauvegarde du cycle potentiel 
    Si les valeurs du cycle potentiel coincident avec le reste des valeurs alors c'est une tortue cyclique
    Sinon ce n'est pas une tortue cyclique


Paramètres retournées si c'est bien une tortue cyclique 
* Tableau des vitesses dans la fenetre : on stocke l'indice a laquelle on a trouvé notre début et fin de cycle. Puis on retourne l'ensemble des vitesse comprises (inclusivement avec index_debut_cycle et index_fin_cycle)

### detect_distraite : Permet de detecter une tortue distraite.
Algorithme : Si pour chaque tortue, aucune des trois detections précédentes n'a renvoyer True alors c'est une tortue distraite
Paramètres retournées si c'est bien une tortue distraite :
* MaxVitesse : On retourne le maximum du tableau de vitesse
* MinVitesse : On retourne le minimum du tableau de vitesse



# Fonction detection "rattage" de top 

Ces deux fonctions permettent de detecter un "rattage" de top dans les tableaux de vitesse et d'accélération des tortues. Elles s'ajustent au tableau récapitulatif des "rattages" de top sauvegardés lors de l'acquisition de données par le biais des requêtes.
Si une top(requeteN) - top(requeteN-1) > 1 c'est que la requête N est un rattage de "top"

Donc si on a un "rattage" de top survenu a l'indice N dans le tableau des positions des tortues alors :

* detectMissTopVitesse : detecte l'indice d'une valeur abérante survenu dans le tableau de vitesse d'une tortue à l'indice n-1. 

* detectMissTopAcceleration : detecte l'indice d'une valeur abérante dans le tableau d'acceleration d'une tortue à l'indice n-2.

# Format d'analyse 

J'utilise pandas pour stocker dans un dataframe l'ensemble des valeurs de vitesse et d'acceleration de chaque tortue
Je rajoute ensuite 4 colonne qui correspondent au types des tortues
A la suite du traitement par la fonction analyse_detection(), ces 4 colonnes sont remplies par False ou True grâce aux fonctions de detections implémentées.
La dataframe est ensuite converti en fichier csv "resultat.csv" pour stocker l'ensemble de nos résultats concernant uniquement les typages. Sinon le dataframe resultant est entierement stocké dans "dataframe_apres_traitement.csv"
Remarque : Lorsque la valeur est True pour une catégorie, le paramètre est aussi stocké dans la colonne pour l'id concerné. Par manque de temps, je n'ai pas pu trouver une solution adaptée à mon code pour ajouter une colonne avec les parametres concernés pour le type de tortue/

# Remarques

La detection de tortue cyclique ne marche pas correctement si un "rattage" de top a lieu à chaque fois au milieu du vraie cycle. 
Ceci vient du fait que tant que mon algorithme est à la recherche d'un cycle potentiel et qu'un "rattage" de top survient (donc valeur abérante), il re-initialise le cycle potentiel qu'il été en train de constitué pour en redémarrer un autre.
Dès lors qu'il trouve un cycle potentiel, il sait s'adapter pour l'évaluation de la comparaison du cycle potentiel avec avec la suite des valeurs du tableaux meme en cas de "rattage" de top. 

