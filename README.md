# HACKATHON QUAI D'ORSAY NUMERIQUE 2017

## Préparer son environnement

### Installer PostgreSQL et PostGIS (partie spatiale)

Sur Ubuntu voir https://www.postgresql.org/download/linux/ubuntu/

    # touch /etc/apt/sources.list.d/pgdg.list
    # echo "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" > /etc/apt/sources.list.d/pgdg.list

Sur Debian 8 voir https://www.postgresql.org/download/linux/debian/

    # touch /etc/apt/sources.list.d/pgdg.list
    # echo "deb http://apt.postgresql.org/pub/repos/apt/ jessy-pgdg main" > /etc/apt/sources.list.d/pgdg.list

    # apt-get update

Installer les paquets:

    $ sudo apt-get install postgresql-9.5 postgresql-9.5-postgis-2.3 postgis postgis-gui postgresql-9.5-postgis-2.3-scripts

Sur desktop, installer 'application d'administration de postgres PgAdmin III:

    $ sudo apt-get install pgadmin3

## Compte d'accès aux données: carto

    $ sudo su - postgres
    # psql -c "create user carto password 'bidon';"

## Chargement données

### Référentiels DFAE, CEF et VISA

La procédure reste la même dans les trois cas:

Se connecter en tant que superutilisateur de PostgreSQL avec l'utilisateur `postgres`

    $ sudo su - postgres

Créer chacunes des bases:

    # psql -c "CREATE DATABASE cef;"
    # psql -c "CREATE DATABASE visa;"
    # psql -c "CREATE DATABASE dfae;"

Charger l'extension PostGIS dans chacune des bases:

    # psql cef -c "CREATE EXTENSION postgis;"
    # psql visa -c "CREATE EXTENSION postgis;"
    # psql dfae -c "CREATE EXTENSION postgis;"

Ajouter les droits d'accès au compte "carto":

    # psql cef -c "GRANT ALL PRIVILEGES ON DATABASE cef TO carto;"
    # psql cef -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO carto;"

    # psql visa -c "GRANT ALL PRIVILEGES ON DATABASE visa TO carto;"
    # psql visa -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO carto;"

    # psql dfae -c "GRANT ALL PRIVILEGES ON DATABASE dfae TO carto;"
    # psql dfae -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO carto;"

Créer les tables:

Aller dans le répertoire où se trouve les scripts `sql` puis exécuter la commande suivante où `nom_base` est la base d'import:

    # psql <nom_base> < create_01.sql.txt

Charger les données dans les différentes tables (ne pas oublier de changer les chemins et de vérifier les droits sur les fichiers de données).

    # psql <nom_base> < copy.sql

Les données corrigées se trouvent sur l'espace de stockage à l'adresse suivante:

    ftp://forge.dsinet.diplomatie.gouv.fr/hackarto2017/Datas/


#### Création des tables et chargement de la base VISA

    psql visa < VISA/create_01.sql.txt
    psql visa < VISA/create_02.sql.txt

    psql visa < VISA/copy_01.sql.txt
    psql visa < VISA/copy_02.sql.txt


#### Création des tables et chargement de la base CEF

    psql cef < CEF/create_01.sql.txt

    psql cef < CEF/copy_01.sql.txt

#### Chargement des données SIG pays

Lorsqu'on utilise PostGIS, un utilitaire nommé `shp2pgsql` est automatiquement installé.

**Création du SQL depuis le shp**

On crée un fichier SQL intermédiaire depuis la donnée cartographique dite "shp" (ou "shapefile").

    shp2pgsql -d -s 4326 ne_10m_admin_0_countries.shp ne_10m_admin_0_countries > /tmp/countries.sql

L'option `-d` veut dire supprimer la table et la créer à nouveau, la peupler.

L'option `-s 4326` veut dire utilise la projection ayant pour code EPSG 4326 (cf <http://epsg.io/4326>)

Ensuite, on passe le chemin du fichier puis le nom de la table `ne_10m_admin_0_countries` (pour un shp, le nom est le même que le nom du fichier sans l'extension)

On redirige vers la sortie le SQL généré.

**Chargement en base**

Se connecter en tant que superutilisateur avec

    sudo su - postgres

Charger en base

    psql cef < /tmp/countries.sql

**Version courte**

On peut charger la donnée en une seule ligne via la syntaxe ci-dessous

    shp2pgsql -d -s 4326 ne_10m_admin_0_countries.shp ne_10m_admin_0_countries | psql cef


### Référentiels externes OCDE

Se connecter en tant que superutilisateur de PostgreSQL avec l'utilisateur `postgres`

    $ sudo su - postgres

Créer chacunes des bases:

    # psql -c "CREATE DATABASE ocde;"

Charger l'extension PostGIS:

    # psql ocde -c "CREATE EXTENSION postgis;"

Ajouter les droits d'accès au compte "carto":

    # psql ocde -c "GRANT ALL PRIVILEGES ON DATABASE ocde TO carto;"
    # psql ocde -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO carto;"

## Analyse

### Visa

* Nombre de Visas délivré par nationalité  (Fait coté donnée)
* Réfugié / Non réfugié
* Tri par date
* Couverture géographique des visas délivrés (Schengen, Métropole, Dom-Tom)
* Tranche d'âge (du demandeur)
* Sexe (du demandeur)
* Activité consulat (via table msv_service, msv_calendrier) - Délai de traitement
* Motif du visa (rapprochement fam,..) / durée du visa
* Nb Visas par type (court séjour, long séjour, ...)

### CEF

* Nombre des utilisateurs ayant fait une demande / pays  (Fait coté donnée)
* Catégoriser par cursus (niveau d'études) / matières (spécialisations)
* Flux (si dossier accepté)
* Maitrise de la langue
* Temps de traitement du dossier
* Vue globale par année / domaine
* Distribution etudes principale par pays via "camembert"

### REPRESENTATION UI

 * Intégration des données OCDE en base (OK)
 * Données OCDE (Compte Annee et Pays) (OK)
 * Données OCDE (Ranking France) (OK)
 * Mise en place du slider temps (changer l'année en cours) (OK)
 * Fonction de mise à jour (representation année / indicateur --> carte et pays) (OK)
 * Higlight pays sélectionné + affichage en bannière de l'année / pays  (OK)
 * Story déplacements [Odyssey.js]: https://cartodb.github.io/odyssey.js/ (OK)
 * Géocoder les points de déplacements (en Python) (OK)

 ### TODO

 * Titre dans les graphes
 * Taille texte dans Treemap
 * Tooltip sur Treemap (OK)
 * Fixer une couleur par sexe sur le "camembert" (OK)
 * Cacher la pyramide des âges mondiale lors du click sur un pays / ajout bouton pour la réafficher (OK)
 * Tooltip sur le polygon pour les visas
 * Tooltip sur les flux (OK)
 * Tooltip sur les camemberts (OK)
 * Visa / année sur la carte (OK)
 * pyramide des âges par année / pays
