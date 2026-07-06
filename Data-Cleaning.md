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




```sql
```

### DataCo - Tokenized Access Logs

- **Date** : Different Timestamp Formatting with mm/dd/yyyy, yyyy/mm/dd

## Data Cleaning

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

4. Les caractères corrompus (``)

    order_state, order_country, order_city

    Le problème : C'est un problème d'encodage (généralement du UTF-8 lu comme du ISO-8859-1).

    Best Practice : On utilise la fonction REGEXP_REPLACE() de PostgreSQL dans dbt pour nettoyer ces caractères fantômes ou les remplacer par une chaîne vide.

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
