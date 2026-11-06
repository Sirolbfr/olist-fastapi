# Journal de l'API Olist — FastAPI

**Nom :** BRIQUET Loris
**Dates :** J1 05/11 · J2 06/11
**Fichier rendu :** api_olist_….zip

> Compétences : WCS C1.6 · C2.4 · C2.1 · C2.5 (niveau 2 · Adapter) — Simplon C6 · C7 (partiel)
> Remplis les parties **dans l'ordre**. Remplace chaque « … ». Une phrase suffit sauf mention contraire.

**Chiffres de référence (export de lundi)**
- Lignes : 110 197 · Commandes livrées : 96 478
- CA total : 15 424 463,65 R$ · Panier moyen : 159,88 R$
- Novembre 2017 : 1 154 559,53 R$ et 7 289 commandes
- SP : 5 771 713,93 R$, 40 501 commandes, 37,42 % du CA

---

# J1 — L'API sur le fichier CSV

<a id="j-m1"></a>
## M1 — Lancer l'API et la tester

- Version de Python : 3.13.16
- Commande de lancement que j'ai utilisée : py -m uvicorn main:app --reload
- Code et `nb_lignes` renvoyés par `/sante` : 200 | 110197
- Différence entre `/docs` et `/sante` ouverts dans le navigateur : `/docs` explique les routes alors que `/sante` l'appelle directement
- Problème rencontré et solution (ou « aucun ») : uv uvicorn main:app --reload qui marche pas, j'ai fait avec la commande de base

<a id="j-m2"></a>
## M2 — Lire une route complète

### Le trajet d'une requête `GET /indicateurs`, en 4 étapes

1. Le navigateur envoie GET à l'adresse 127.0.0.1:8000/indicateurs
2. FastAPI exécute la fonction indicateurs du fichier main, qui appelle la fonction indicateurs du fichier donnees_csv
3. Cette fonction lit le .csv et renvoie un dictionnaire
4. Le modèle Indicateurs vérifie le format du dictionnaire puis FastAPI le renvoie en JSON avec le code 200

### L'expérience du contrat

- Quand j'ai mis `panier_moyen` en commentaire dans le modèle, la réponse a ignoré `panier_moyen` .
- Conclusion : Le commentaire fait son job

### Pourquoi deux fichiers (`main.py` et `donnees_csv.py`) ?

- Pour pouvoir switch de source plus tard

### Comparaison avec lundi

- `nb_commandes` : pareil (lundi : 96 478)
- `ca_total` : pareil (lundi : 15 424 463,65)

<a id="j-m3"></a>
## M3 — Ventes mensuelles

### Tests

- [x] Sans paramètre : **23** mois, de `2016-09` à `2018-08`.
- [x] `debut=2017-01`, `fin=2017-12` : **12** mois ; novembre 2017 = **1154559.53** et **7289** commandes.
- [x] `debut=2018-02`, `fin=2017-01` : code **400**.
- [x] `debut=2017-13` : code **422**.

### Questions

1. Différence entre un paramètre de chemin et un paramètre de requête, avec un exemple de cette API : …
2. Qui renvoie le code 400 (toi ou FastAPI) ? Et le code 422 ? Pourquoi cette différence ? Code 400 : moi, le code 422 : FastAPI. Le code 422 est géré automatiquement alors que le code 400 est prévu dans notre script.

<a id="j-m4"></a>
## M4 — Top catégories

### Tests

- `n=3` : 
- ```json
[
  {
    "categorie": "health_beauty",
    "ca": 1412271.15,
    "part_ca": 9.16
  },
  {
    "categorie": "watches_gifts",
    "ca": 1264333.12,
    "part_ca": 8.2
  },
  {
    "categorie": "bed_bath_table",
    "ca": 1225209.26,
    "part_ca": 7.94
  }
]
```
- [x] Sans paramètre : **10** catégories, `health_beauty` en tête (**9.16** %).
- [x] Les 10 parts additionnées font environ **62,4** % (lundi : les 10 premières font 62 % du CA).
- [x] `n=0` et `n=51` : code **422**.

### Question

- Que reçoit Rafael s'il n'envoie pas `n` ? Où est-ce décidé dans le code ? Les 10 premières catégories, dans le Query du main.py

<a id="j-m5"></a>
## M5 — Chiffres d'un État

### Mon modèle `Etat` (copie le code)

```python
class Etat(BaseModel):
    etat: str
    ca: float
    nb_commandes: int
    part_ca: float
```

### Tests
- [x] `/etats/SP` : CA **5771713.93**, **40501** commandes, **37.42** %.
- [x] `/etats/sp` donne exactement la même réponse.
- [x] `/etats/XX` : code **404** avec un message lisible.
- [x] `/etats/SPX` : code **422**.

### Question

- Pourquoi un État sans vente renvoie 404 et pas une réponse vide avec le code 200 ? Car un état sans ventes ne figure pas dans la base, ce qui donne un Not Found

<a id="j-m6"></a>
## M6 — Recette complète dans /docs

Pour chaque test : le code obtenu, puis une valeur lue dans la réponse.

1. `GET /sante` (attendu 200) : 
 > 200 | `{ "statut": "ok", "source": "csv", "nb_lignes": 110197 }`

2. `GET /indicateurs` (attendu 200) :
> 200 | `{ "nb_lignes": 110197, "nb_commandes": 96478, "ca_total": 15424463.65, "panier_moyen": 159.88 }`

3. `GET /ventes/mensuelles` sans paramètre (attendu 200, 23 mois) :
> 200 | `{"mois": "2016-09", "ca": 143.46, "nb_commandes": 1 }`

4. `GET /ventes/mensuelles?debut=2018-01` (attendu 200) : 
> 200 | nombre de mois 8

5. `GET /ventes/mensuelles?debut=2018-02&fin=2017-01` (attendu 400) : 400
6. `GET /ventes/mensuelles?fin=nov` (attendu 422) : 422
7. `GET /categories/top` (attendu 200, 10 catégories) : 200
8. `GET /categories/top?n=50` (attendu 200) : dernière catégorie de la liste 200
9. `GET /categories/top?n=0` (attendu 422) : 422
10. `GET /etats/RJ` (attendu 200) : part du CA 200
11. `GET /etats/XX` (attendu 404) : 404
12. `GET /etats/SPX` (attendu 422) : 422

### Bilan du J1

1. Ce qui fonctionne : tout
2. Ce qui me reste à comprendre ou à terminer : rien

---

# J2 — L'API sur PostgreSQL

<a id="j-m7"></a>
## M7 — Charger les ventes dans PostgreSQL

- Base utilisée : olist
- Résultat du contrôle final du script : **110197**, **96478** et **15424463.65**.
- Droit donné à `lecteur_olist` sur la table, et pourquoi seulement celui-là : select pour la sécurité

<a id="j-m8"></a>
## M8 — Mettre les secrets dans `.env`

- Résultat de la commande de test : lecteur_olist
- Rôle de `.env` : sécuriser les secrets
- Rôle de `.env.exemple` : template
- Rôle de la ligne `.env` dans `.gitignore` : ignore le .env
- J'ai vérifié qu'aucun `.py` ne contient le mot de passe (oui / non) : oui

<a id="j-m9"></a>
## M9 — Réécrire les fonctions en SQL

### Résultat de `verifier_sources.py`

- Bilan affiché : … / 8
- Fonction qui m'a demandé le plus de corrections, et pourquoi : …

### Ma requête de `top_categories()` (copie-la)

```sql
…
```

### Sécurité

- Pourquoi `%(code)s` plutôt qu'une f-string ? Donne l'exemple d'une valeur dangereuse. …
- Pourquoi faire calculer PostgreSQL plutôt que Python ? …

<a id="j-m10"></a>
## M10 — Basculer l'API sur PostgreSQL

- `/sante` après la bascule : source … , lignes …
- Pourquoi il faut relancer l'API après avoir modifié `.env` : …
- Test de panne (`PGPORT=5999`) : code … , message …
- Ce qu'aurait vu Rafael sans le gestionnaire d'erreur `SourceIndisponible` : …
- Ce que Rafael a dû changer dans son code pour la bascule : …

<a id="j-m11"></a>
## M11 — Écrire le client de Rafael

- Bilan affiché par `client.py` : … / 10
- Nombre de lignes de `ca_mensuel_2017.csv`, en-tête compris : …
- Différence entre `.status_code` et `.json()` : …
- Utilité du paramètre `timeout` : …

<a id="j-m12"></a>
## M12 — README et rendu

- Le test de remise à zéro a fonctionné en suivant seulement le README (oui / non) : …
- Ce que j'ai dû corriger dans le README après ce test : …
- Contenu de mon zip, fichier par fichier : …

### Mes choix techniques (3 à 5 lignes)

- Source de données retenue et pourquoi : …
- Compte de connexion et pourquoi : …
- Gestion des erreurs : …
- Ce que j'ajouterais pour mettre l'API en production : …

### Message à Mariana (3 lignes, sans jargon)

- Ce que Rafael peut faire maintenant : …
- D'où viennent les chiffres : …
- Le chiffre qui prouve que c'est juste : …
