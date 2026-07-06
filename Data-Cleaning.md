# Data Preparation & Cleansing

DBeaver import data as ISO-8859-1 or Windows-1252 (not utf-8) to handle accents

## Data Exploration

### DataCo - Supply Chain Dataset

- **Customer City** : Error with CA not full name
- **Customer Email** : Anonymized and Unsuable (e.g. XXXXXXX) --> Remove Column
- **Customer Password** : Anonymized and Unsuable (e.g. XXXXXXX) --> Remove Column
- <s>**Customer State** : Mainly US (CA, MI, NY) but few values with Numbers (e.g. 91732, 95758)</s>
- **Customer Lname** : Some Missing Values --> How to handle
- **Customer Zip Code** : Some Missing Values --> How to handle
- **order date (DateOrders)** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd
- **Order Zipcode** : Some Missing Values --> How to handle
- **Product Description** : Only Blanks --> Remove Column
- **Product Image** : URL link --> Remove Column
- **Product Status** : Only 0 Values, adding no information --> Remove Column
- **shipping date (DateOrders)** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd

Il y a 3 lignes d'erreur en cascade où customer city à le customer state et state le zipcode. Il faudrait décaler tout ça et mettre unknown pour la city. Logique de Réparation

Pour le décalage des 3 lignes : On va utiliser un CASE WHEN en repérant ces lignes spécifiques. Si le customer_state contient en réalité un code postal (on peut le détecter avec une expression régulière qui cherche des chiffres ^[0-9]+$), alors on décale les valeurs :

    La ville devient null (puisqu'elle contenait l'état, la vraie ville est perdue ➔ on mettra 'unknown' plus tard dans Power BI).

    L'état prend la valeur qui était dans la ville.

    Le code postal prend la valeur qui était dans l'état.


```sql
```

### DataCo - Tokenized Access Logs

- **Date** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd

## Data Cleaning

# 📂 Couche Staging - Nettoyage et Standardisation des Données (Bronze Layer)

Ce dossier contient la première étape de transformation de notre pipeline dbt. L'objectif de cette couche est de prendre les données brutes du schéma `raw`, de les nettoyer, de corriger les anomalies structurelles, et de préparer une fondation saine (1NF) avant d'entamer la normalisation en 3NF (Silver Layer).

---

## 🛠️ Actions de Nettoyage Réalisées

### 1. Normalisation Automatique des En-têtes (Automation Engine)
Le dataset d'origine comporte plus de 50 colonnes avec des syntaxes hétérogènes (espaces, majuscules, parenthèses). 
* **Action :** Utilisation d'une boucle dynamique **Jinja** combinée aux macros d'adaptation dbt pour renommer automatiquement l'intégralité des colonnes en `lowercase` et `snake_case` (ex: `Days for shipping (real)` ➔ `days_for_shipping_real`).

### 2. Redressement des Anomalies de Cascade (Data Alignment)
* **Problème :** Identification de 3 lignes corrompues dans le fichier source où les données géographiques ont glissé d'une colonne vers la droite (la ville contenait l'état `"CA"`, l'état contenait le code postal, etc.).
* **Règle métier :** Alignement chirurgical via un bloc `CASE WHEN` basé sur la condition `customer_city = 'CA'`. Les données ont été glissées vers leurs vraies colonnes respectives et la ville d'origine a été passée à `'unknown'`.

### 3. Gestion des Valeurs Manquantes & Concaténation (Data Integrity)
* **Règle Nom Client :** Fusion des colonnes `customer_fname` et `customer_lname`. Pour éviter les chaînes coupées ou les espaces vides dus aux noms de famille manquants (chaînes vides `''`), une sécurité `COALESCE` remplace le nom absent par `'UNKNOWN'` (ex: `"John UNKNOWN"`).
* **Règle Types Postgres :** Forçage des types de codes postaux (`customer_zipcode`) en `VARCHAR` pour préserver l'intégrité des données (éviter la perte des zéros non significatifs au début des codes postaux).

### 4. Harmonisation des Formats de Dates (Temporal Consistency)
* **Problème :** Les colonnes de dates (`order_date` et `shipping_date`) étaient stockées en chaînes de caractères (`VARCHAR`) avec des formats américains (`MM/DD/YYYY`) et standards (`YYYY-MM-DD`) mélangés.
* **Action :** Application de fonctions de parsing conditionnel (`LIKE` + `to_date`) pour garantir une conversion propre vers le type de données `DATE` de PostgreSQL sans générer d'erreurs d'exécution.

---

## 📊 Modèles dbt de cette Couche

| Modèle SQL | Type d'Objet Postgres | Description |
| :--- | :--- | :--- |
| `stg_dataco_supply_chain_v2` | **Vue (View)** | Modèle unique consolidé appliquant le renommage automatique Jinja et l'ensemble des 4 règles de nettoyage spécifiques. |

---

## 🧪 Tests de Qualité (Data Quality)

Pour certifier la conformité de notre nettoyage, des tests d'entrée ont été configurés dans le fichier `schema.yml` :
* **`not_null`** sur `customer_id` : S'assure qu'aucun enregistrement ne manque d'identifiant client.
* **`not_null`** sur `order_date` & `shipping_date` : Valide le succès du parsing de toutes les dates du dataset.

> **Statut actuel des tests :** `PASSED` 🟩


Stratégie de Nettoyage (Colonne par Colonne)
1. Les colonnes à supprimer d'office (DROP)

    customer_email, customer_password, product_description, product_status

    Best Practice : On ne les sélectionne tout simplement pas dans notre SELECT de Staging. Cela allège immédiatement ta base de données et optimise les performances de calcul.

2. La gestion des valeurs manquantes (NULL)

    customer_lname (Nom de famille) : S'agissant d'un champ texte textuel non critique pour les jointures, on remplace les valeurs manquantes par une chaîne standard via un COALESCE. Exemple : 'unknown'.

    customer_zip_code & order_zipcode : Le code postal est une donnée géographique. Si la valeur est manquante, la remplacer par une chaîne de caractères vide '' ou '00000' (si purement US) ou laisser NULL est préférable pour ne pas fausser des calculs de coordonnées ou de zones. Pour ton portfolio, utiliser COALESCE(colonne, 'unknown') ou garder NULL proprement est très bien.

3. Les anomalies de formats de dates

    order_date_dateorders & shipping_date_dateorders

    Best Practice : PostgreSQL dispose d'une fonction très puissante, TO_TIMESTAMP() ou CAST(... AS TIMESTAMP). Cependant, si tes chaînes mélangent nativement les formats mm/dd/yyyy et yyyy/mm/dd, PostgreSQL risque de lever une erreur de parsing. La méthode la plus robuste en SQL est d'utiliser un CASE WHEN combiné avec to_timestamp selon la structure détectée (ex: présence d'un tiret ou position des slashs).

5. Les mélanges de données (Customer State : CA vs 91732)

    Les valeurs numériques comme 91732 sont en réalité des codes postaux qui ont glissé dans la colonne de l'État (State).

    Best Practice : On applique un CASE WHEN : si la valeur est purement numérique ou contient 5 chiffres, on la catégorise comme 'unknown' (et idéalement, on pourrait essayer de la réassigner au code postal si la structure intermédiaire le demande).

When we are data cleaning we usually follow a few steps
1. check for duplicates and remove any
2. standardize data and fix errors
3. Look at null values 
4. remove any columns and rows that are not necessary - few ways

```sql
```


# Tests de Staging
models/staging/schema.yml
```yml
version: 2

models:
  - name: stg_dataco_supply_chain
    description: "Table de staging nettoyée et standardisée du flux Supply Chain"
    columns:
      - name: customer_id
        description: "Identifiant unique du client (peut contenir des doublons à ce stade)"
        tests:
          - not_null # On vérifie qu'aucun client n'a un ID manquant

      - name: order_date
        description: "Date de la commande harmonisée"
        tests:
          - not_null

      - name: shipping_date
        description: "Date d'expédition harmonisée"
        tests:
          - not_null
```
Si dbt renvoie PASS, cela signifie que notre étape de staging est saine (pas de dates manquantes, pas d'ID clients manquants). Si un test renvoie FAIL, il nous donnera exactement le nombre de lignes corrompues, et on saura précisément quoi corriger


Étape 1 : Ta table brute respecte-t-elle la 1NF (Première Forme Normale) ?

Pour valider la 1NF, un tableau doit respecter 3 règles :

    L'atomicité des cellules : Chaque case ne doit contenir qu'une seule valeur (pas de listes, pas de tableaux de données dans une seule cellule).

    Des colonnes uniques : Pas de colonnes dupliquées (ex: telephone_1, telephone_2).

    Une clé primaire : Chaque ligne doit être identifiable de façon unique.


tape 2 : Comment passe-t-on à la 2NF (Deuxième Forme Normale) ?

Pour être en 2NF, il faut :

    Être en 1NF.

    S'assurer que toutes les colonnes non-clés dépendent entièrement de la clé primaire de la table, et pas seulement d'une partie de celle-ci (ce qu'on appelle éliminer les dépendances partielles).

Le problème de ton dataset actuel :
Actuellement, ton modèle de staging est un gros tableau plat "dénormalisé". Ta ligne représente "un article de commande". Si tu as une clé composite (par exemple id_commande + id_produit), le nom_du_client dépend uniquement de l'acheteur, pas du produit ! Il y a donc une dépendance partielle.
Étape 3 : Pourquoi le Data Engineer fait-il la 2NF et la 3NF en même temps ?

En théorie académique, on sépare strictement les étapes. En ingénierie de données avec dbt, le passage de la table plate (1NF) aux tables relationnelles (2NF/3NF) se fait dans le même élan au sein de ta couche Intermediate.

Lorsque tu crées tes modèles intermédiaires, tu vas "éclater" ton gros tableau en plusieurs entités distinctes :

    int_customers.sql (Tu isoles l'entité Client. Sa clé primaire est customer_id. Le nom, la ville et le code postal dépendent uniquement de cet ID. C'est de la 2NF/3NF pure pour le bloc client).

    int_products.sql (Tu isoles l'entité Produit. L'ID produit détermine le nom et la catégorie).

    int_orders.sql (Ta table de liaison/faits, qui ne conserve que les transactions, les dates, les montants, et les clés étrangères vers tes clients et produits).
