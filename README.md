# 🏗️ Global Supply Chain Analysis

<img width="75%" height="75%" alt="logistique_supply_chain" src="https://github.com/user-attachments/assets/ad11cf27-a286-462d-b2c1-16b4aedb795b" />


## 📌 Présentation du Projet

---

## 🏗️ Architecture du Pipeline & Flux de Données


---

## 🎯 Problématique Business & Questions Clés

---

## 🛠️ Stack Technique
* **Infrastructure :** Docker & PostgreSQL
* **IDE / Gestion DB :** DataGrip
* **Modélisation :** SQL (Création de Vues et Tables Dimensionnelles)
* **Visualisation :** Power BI (Modélisation en étoile, Mesures DAX)

---

## 📐 Modélisation Dimensionnelle (Méthodologie Kimball)

Pour casser la complexité des jointures d'origine (Flocon étendu), le schéma `analytics` centralise la donnée autour d'un **Grain unique : Une transaction de location**.

### Le Schéma en Étoile mis en œuvre :
* **Table de Faits :** `fact_` 
* **Tables Dimensions :**
  * `dim_customers` :
  * `dim_product` :
  * `dim_` : 
  * `dim_date` : 

---

## 🚀 Structure du Repository & Avancement

- [x] **Étape 1 :** Data Preparation & Data Cleaning
- [] **Étape 2 :** Écriture des scripts de transformation SQL et création du schéma `analytics`.
- [] **Étape 3 :** Connexion optimisée à Power BI et configuration du modèle sémantique.
- [] **Étape 4 :** Design du Dashboard, KPIs et calcul du ROI fictif.

---

## 📈 Impact Business & Livrables Visés (KPIs)
Le dashboard final mettra en évidence :

