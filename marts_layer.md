

Nous allons concevoir deux tables de faits qui vont coexister dans ta couche Gold / Marts et qui partageront les mêmes dimensions :

    fact_sales (Processus Commercial) : Axé sur la performance financière, les quantités vendues, les remises et les marges.

    fact_shipping (Processus Logistique) : Axé sur les flux de livraison, les risques de retard, les modes de transport et l'efficacité des expéditions.

    dim_date (Dimension Temporelle) : Notre calendrier central unique pour analyser les deux processus selon les mêmes axes temporels (mois, trimestres, jours de la semaine).
