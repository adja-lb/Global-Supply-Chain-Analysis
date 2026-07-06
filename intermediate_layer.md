# 🏛️ Couche Intermediate - Normalisation 3NF (Silver Layer)

Ce dossier contient la deuxième étape de notre pipeline dbt. L'objectif de cette couche est d'éliminer la redondance informationnelle du dataset plat d'origine en appliquant les principes de la **Troisième Forme Normale (3NF)**. Les données sont éclatées en entités indépendantes pour garantir l'intégrité référentielle avant l'exposition décisionnelle.

---

## 📐 Architecture de Normalisation (3NF)

Le gros tableau dénormalisé issu du Staging a été fragmenté en **5 modèles distincts**. Les dépendances transitives ont été supprimées (par exemple, les attributs d'une catégorie dépendent de l'ID catégorie, et non de l'ID produit).



### 👥 1. Entité Clients (`int_customers`)
* **Rôle :** Centralise le référentiel unique des acheteurs.
* **Clé Primaire :** `customer_id`
* **Attributs :** Profil du segment, adresse normalisée (rue, ville, état, pays, zipcode) et coordonnées de géolocalisation (`latitude`, `longitude`).

### 📦 2. Entité Produits (`int_products`)
* **Rôle :** Contient le catalogue des articles vendus.
* **Clé Primaire :** `product_id` (dérivé de `product_card_id`)
* **Clés Étrangères :** `category_id`, `department_id`
* **Attributs :** Nom du produit et prix unitaire théorique (`product_price`).

### 🏷️ 3. Entité Catégories (`int_categories`)
* **Rôle :** Table de référence isolant le libellé des catégories pour éviter leur duplication dans la table produit.
* **Clé Primaire :** `category_id`
* **Attributs :** `category_name`

### 🏢 4. Entité Départements (`int_departments`)
* **Rôle :** Représente la structure organisationnelle interne de l'entreprise associée aux produits.
* **Clé Primaire :** `department_id`
* **Attributs :** `department_name`

### 📊 5. Table Centrale des Transactions (`int_orders_items`)
* **Rôle :** Point central de l'architecture stockant les faits transactionnels à la maille "ligne de commande" (Order Item) et portant les clés de relations 3NF.
* **Clé Primaire :** `order_item_id`
* **Clés Étrangères :** `order_id`, `customer_id`, `product_id`
* **Métriques Clefs :** * *Temporel & Logistique :* Dates de commande et d'expédition, délais réels vs planifiés, statuts et risques de livraison.
  * *Financier :* Ventes brutes (`gross_sales`), ventes nettes (`net_sales`), remises, taux de réduction, profit unitaire et global.

---

## 🛠️ Règles de Typage & d'Intégrité Appliquées

1. **Précision Décimale Strict :** Toutes les métriques financières ont été castées en `decimal(10,2)` ou `decimal(5,4)` pour les ratios, évitant ainsi les dérives de calculs propres au type `float`.
2. **Préservation des Codes Postaux :** Les codes postaux des clients (`customer_zipcode_clean`) ont été explicitement typés en `VARCHAR` pour empêcher PostgreSQL de tronquer les zéros non significatifs au début des chaînes.
3. **Suppression du Bruit :** Les colonnes inutiles pour le décisionnel ou non conformes (ex: `product_image`, `customer_password`, `customer_email`) ont été définitivement écartées de cette couche.

---

## 📊 Matérialisation dbt

Tous les modèles de ce dossier sont configurés pour se compiler sous forme de **tables physiques** (`table`) afin de matérialiser proprement le cœur de notre Data Warehouse (Silver Layer) et d'optimiser les performances des jointures futures.
