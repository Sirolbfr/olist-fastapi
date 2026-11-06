# API des indicateurs de ventes Olist

> À compléter le J2 (mission 12). Remplace chaque « … ». Ce fichier s'adresse à un développeur
> qui découvre le projet : il doit pouvoir installer et lancer l'API sans te poser de question.

## Objet

Api pour accéder à la table ventes d'Olist

## Prérequis

- Python 3.10 ou plus récent
- ?

## Installation

1. cloner le repo
2. installer les libraries : `py -m pip install -r requirements.txt`

## Configuration

Confgiguration des routes dans main.py

## Lancement

`py -m uvicorn main:app --reload`

## Routes

**`GET /sante`**
- Rôle : Check si le serveur est up et sur quelle source il se base
- Paramètres : aucun
- Exemple de réponse : {
  "statut": "ok",
  "source": "postgresql",
  "nb_lignes": 110197
}

**`GET /indicateurs`**
- Rôle : Résume la table ventes en qq chiffres
- Paramètres : aucun
- Exemple de réponse : {
  "nb_lignes": 110197,
  "nb_commandes": 96478,
  "ca_total": 15424463.65,
  "panier_moyen": 159.88
}

**`GET /ventes/mensuelles`**
- Rôle : Vérifie le nb de commandes et le ca par mois
- Paramètres : début et fin
- Exemple d'appel : http://127.0.0.1:8000/ventes/mensuelles?debut=2017-01&fin=2017-03

**`GET /categories/top`**
- Rôle : Retourne les n premières catégories avec leur ca et part_ca respectifs
- Paramètres : n
- Exemple d'appel : http://127.0.0.1:8000/categories/top?n=10

**`GET /etats/{code_etat}`**
- Rôle : Retourne un état avec son ca et sa part_ca
- Paramètres : code_etat
- Exemple d'appel : http://127.0.0.1:8000/etats/RR

## Codes de réponse

- `200` : OK — La requête a réussi et le serveur renvoie les données demandées (ex. la liste des ventes, les indicateurs ou l'état).
- `400` : Bad Request — La requête est mal formée ou contient des paramètres invalides côté client (ex. un paramètre de filtre incorrect)
- `404` : Not Found — La ressource demandée n'existe pas (ex. un code d'État brésilien inconnu ou une route inexistante).
- `422` : Unprocessable Entity — Erreur de validation FastAPI / Pydantic (ex. le format du paramètre de chemin ne respecte pas la regex ^[A-Za-z]{2}$ ou un type de donnée est incorrect).
- `503` : Service Unavailable — Le service est temporairement indisponible (ex. impossible de se connecter à la base de données SQL analytique.ventes ou problème avec la source de données).

## Sécurité

- `%(code)s` pour éviter des injections sql
- Gestion des calculs dans PostgreSQL

## Données

Olist, *Brazilian E-Commerce Public Dataset* (Kaggle), licence CC BY-NC-SA 4.0. …
