# Projet NoSQL - Dashboard MongoDB Atlas

## Présentation
Ce projet a été développé dans le cadre du cours de NoSQL de Monsieur Touati. Il s'agit d'un tableau de bord interactif qui se connecte à une base de données MongoDB Atlas pour visualiser et analyser les données d'événements sportifs des Jeux Olympiques.

## Fonctionnalités
- Connexion à une base de données MongoDB Atlas
- Visualisation des statistiques clés : événements, utilisateurs, réservations, revenus
- Graphiques interactifs pour l'analyse des données
- TOP 5 des événements les plus populaires
- Répartition des revenus par type de sport
- Statut des réservations (confirmé, en attente, annulé)
- Rafraîchissement automatique des données

## Démonstration des concepts NoSQL
Ce projet démontre plusieurs concepts NoSQL vus en cours :
- **Requêtes d'agrégation MongoDB** : utilisation de pipelines d'agrégation pour les statistiques
- **Lookup** : jointures pour associer les événements et les réservations
- **Structure de données orientée document** : exploitation de la flexibilité du modèle MongoDB
- **Déploiement cloud** : utilisation de MongoDB Atlas comme service managé
- **Évolutivité horizontale** : connexion à un cluster répliqué pour la haute disponibilité

## Instructions d'exécution

Cette application est fournie sous forme d'image Docker pour faciliter son déploiement sans configuration.

### Prérequis
- Docker installé sur votre machine

### Étapes d'exécution

1. **Téléchargez l'image Docker** depuis ce dépôt (fichier `dashboard-olympicgames.tar.gz`)

2. **Décompressez l'image** :
   ```bash
   # Sur Linux/Mac
   gzip -d dashboard-olympicgames.tar.gz
   
   # Sur Windows avec 7-Zip
   # Clic droit sur dashboard-olympicgames.tar.gz → 7-Zip → Extraire ici
   ```

3. **Chargez l'image dans Docker** :
   ```bash
   docker load -i dashboard-olympicgames.tar
   ```

4. **Exécutez le conteneur** :
   ```bash
   docker run -p 8050:8050 dashboard-olympicgames
   ```

5. **Accédez au tableau de bord** dans votre navigateur à l'adresse :
   ```
   http://localhost:8050
   ```

## Structure de la base de données
L'application se connecte à une base de données `olympicGames` contenant les collections suivantes :

1. **events** : Événements sportifs
   ```javascript
   {
     _id: ObjectId,
     nom: String,
     sport: String,
     date: Date,
     lieu: String,
     capacite: Number
   }
   ```

2. **users** : Utilisateurs
   ```javascript
   {
     _id: ObjectId,
     nom: String,
     prenom: String,
     email: String,
     pays: String
   }
   ```

3. **reservations** : Réservations de billets
   ```javascript
   {
     _id: ObjectId,
     user_id: ObjectId,
     event_id: ObjectId,
     date_reservation: Date,
     statut: String, // "confirmé", "en attente", "annulé"
     billets: [
       {
         categorie: String,
         prix: Number,
         quantite: Number
       }
     ]
   }
   ```

## Exemples de pipelines d'agrégation

Voici quelques-uns des pipelines d'agrégation utilisés dans l'application :

### TOP 5 des événements par réservations
```javascript
[
  {"$lookup": {
    "from": "events",
    "localField": "event_id",
    "foreignField": "_id",
    "as": "event_info"
  }},
  {"$unwind": "$event_info"},
  {"$group": {
    "_id": "$event_info._id",
    "nom": {"$first": "$event_info.nom"},
    "sport": {"$first": "$event_info.sport"},
    "count": {"$sum": 1},
    "revenue": {"$sum": {"$sum": "$billets.prix"}}
  }},
  {"$sort": {"count": -1}},
  {"$limit": 5}
]
```

### Revenus par sport
```javascript
[
  {"$lookup": {
    "from": "events",
    "localField": "event_id",
    "foreignField": "_id",
    "as": "event_info"
  }},
  {"$unwind": "$event_info"},
  {"$unwind": "$billets"},
  {"$group": {
    "_id": "$event_info.sport",
    "total": {"$sum": "$billets.prix"}
  }},
  {"$sort": {"total": -1}}
]
```

## Remarques pour l'évaluation

Ce projet démontre les compétences suivantes :
- Modélisation de données NoSQL
- Requêtes complexes avec MongoDB
- Création d'un tableau de bord analytique
- Connexion à MongoDB Atlas (cloud)
- Conteneurisation avec Docker

La connexion à la base de données est déjà configurée dans l'image Docker pour faciliter l'évaluation.
