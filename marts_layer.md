# 🏆 Couche Marts - Architecture en Étoile Multi-Faits (Gold Layer)

Ce dossier représente la couche finale (Gold) de notre entrepôt de données. L'objectif de cette couche est de dénormaliser les entités de la couche Silver (3NF) pour les structurer selon la méthodologie de **Ralph Kimball (Modèle en Étoile)**. 

Pour refléter au mieux la réalité d'un environnement d'entreprise, l'architecture a été complexifiée pour gérer un **modèle multi-faits** avec des **dimensions partagées (conformed dimensions)** et une dimension temporelle à **rôles multiples (Role-Playing Dimension)**.


                             ┌──────────────────┐
                             │   dim_customers  │
                             └────────┬─────────┘
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼                                         ▼
        ┌──────────────┐                          ┌──────────────┐
        │  fact_sales  │                          │fact_shipping │
        └───────┬──────┘                          └───────┬──────┘
                │                                         │
                │           ┌────────────────┐            │ (Active: order_date)
                ├──────────►│    dim_date    │◄───────────┤
                │           └────────────────┘            │ (Inactive: shipping_date)
                ▼                                         ▼
        ┌──────────────┐                          ┌──────────────┐
        │ dim_products │                          │ dim_products │
        └──────────────┘                          └──────────────┘

### 📈 1. Table de Faits : Performance Commerciale (`fact_sales`)
* **Processus Métier :** Analyse des flux financiers, des volumes de vente et de la rentabilité.
* **Grain :** La ligne de commande (`order_item_id`).
* **Clés de Liaisons :** `customer_id`, `product_id`, `order_date_key`.
* **Métriques Clefs :** Ventes brutes, remises, ventes nettes (`net_sales`), et marges bénéficiaires.

### 🚚 2. Table de Faits : Performance Logistique (`fact_shipping`)
* **Processus Métier :** Analyse de la Supply Chain, de l'efficacité des expéditions et des risques de retard.
* **Grain :** La ligne d'expédition (`order_item_id`).
* **Clés de Liaisons :** `customer_id`, `product_id`, `order_date_key`, `shipping_date_key`.
* **Métriques Clefs :** Jours de livraison réels vs planifiés, risque de retard (`late_delivery_risk`), statut de livraison, et calcul de l'écart d'expédition (`shipping_delay_days`).

---

## 🗓️ Focus Technique : La Dimension Temporelle & Rôles Multiples

### Génération Dynamique (`dim_date`)
La table `dim_date` est générée dynamiquement directement dans PostgreSQL via une série temporelle couvrant la période 2015-2025. Elle enrichit le calendrier avec des attributs analytiques (`calendar_year`, `month_name`, `calendar_quarter`, `day_of_week_name`, `is_weekend`).

### Gestion des Relations Actives vs Inactives (*Role-Playing*)
Le processus logistique (`fact_shipping`) interagit avec le calendrier à travers deux concepts :
1. **`order_date_key` (Relation Active) :** Le chemin par défaut utilisé par l'outil de BI pour filtrer les performances selon la date d'achat.
2. **`shipping_date_key` (Relation Inactive) :** Une relation sous-jacente (ligne pointillée). Pour analyser les volumes réels d'expéditions physiques, la relation sera explicitement activée en DAX dans Power BI via la fonction :
   `USERELATIONSHIP(dim_date[date_key], fact_shipping[shipping_date_key])`

---

## 🗃️ Dimensions Partagées & Dénormalisation

* **`dim_products` :** Fusionne (dénormalise) l'entité produit avec ses catégories et ses départements parents. Cela élimine le besoin de jointures en cascade (Flocon) dans l'outil de BI, maximisant ainsi les performances de calcul de Power BI.
* **`dim_customers` :** Projette le profil propre et géolocalisé des clients.

---

## 🏅 Module d'Analyse des Classements (Rankings Performance)

### Problématique Business
Dans le cadre du pilotage de DataCo, l'identification rapide des segments leaders et des sous-performances est cruciale. Afin d'éviter de surcharger le modèle sémantique Power BI avec des calculs DAX itératifs lourds (générant des problèmes de performance sur de grands volumes), la logique de classement (*Rankings*) a été entièrement déportée en amont dans l'architecture de données (Medallion Architecture - Couche Gold/Marts) à l'aide de **dbt** et **SQL**.

### Solution Technique (SQL & dbt)
Création d'un modèle de données agrégé mensuel (`mart_sales_rankings.sql`) s'appuyant sur les fonctions analytiques de fenêtrage (`DENSE_RANK() OVER(...)`). 

Le choix de `DENSE_RANK()` garantit une gestion stricte des ex-æquo (sans saut de rang dans le classement) et s'articule autour de trois axes analytiques :
1. **Classement des Départements :** Performance globale inter-départements réinitialisée chaque mois.
2. **Classement des Catégories :** Mix produit interne à *chaque* département pour identifier les spécificités locales.
3. **Classement des Produits :** Top produits au sein de leur propre catégorie pour isoler les "Best Sellers".

---

## 🗺️ Architecture du Modèle de Données

Plutôt que d'avoir une seule table lourde, la donnée est segmentée pour isoler deux processus métiers distincts qui partagent le même référentiel :

Nous allons concevoir deux tables de faits qui vont coexister dans ta couche Gold / Marts et qui partageront les mêmes dimensions :

    fact_sales (Processus Commercial) : Axé sur la performance financière, les quantités vendues, les remises et les marges.

    fact_shipping (Processus Logistique) : Axé sur les flux de livraison, les risques de retard, les modes de transport et l'efficacité des expéditions.

    dim_date (Dimension Temporelle) : Notre calendrier central unique pour analyser les deux processus selon les mêmes axes temporels (mois, trimestres, jours de la semaine).

<img width="989" height="753" alt="Capture d&#39;écran 2026-07-07 221034" src="https://github.com/user-attachments/assets/34bbf75a-5749-498c-a053-51d742081b6f" />

