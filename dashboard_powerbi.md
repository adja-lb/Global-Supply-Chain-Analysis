# KPI

📊 Onglet 1 : Performance Commerciale (fact_sales)

Cet onglet s'adresse au Directeur Financier et au Directeur Commercial.

    Les KPIs Flash : * Chiffre d'Affaires Brut (gross_sales) vs Chiffre d'Affaires Net (net_sales).

        Taux de remise moyen (order_item_discount_rate).

        Marge totale et taux de profitabilité moyen.

    Les Visuels Clés :

        Graphique en barres : Top 10 des produits les plus rentables (en croisant avec dim_products).

        Courbe d'évolution : Tendances des ventes nettes par mois/trimestre (en croisant avec dim_date).

        Treemap : Répartition des ventes par segment de clientèle (dim_customers).

🚚 Onglet 2 : Efficacité Supply Chain (fact_shipping)

Cet onglet s'adresse au Directeur Logistique.

    Les KPIs Flash :

        Taux de commandes en retard (basé sur late_delivery_risk).

        Délai moyen de livraison réel (days_for_shipping_real).

        Écart moyen entre le prévu et le réel (shipping_delay_days).

    Les Visuels Clés :

        Carte de chaleur (Map) : Visualisation des pays/états qui subissent le plus de retards de livraison (en utilisant la géolocalisation de fact_shipping ou dim_customers).

        Graphique en secteurs (Donut) : Répartition des statuts de livraison (Closed, Pending, Suspected Fraud).

        Matrice de performance : Délai de livraison moyen par mode d'expédition (shipping_mode).


Si tu utilises AVERAGE(fact_shipping[days_for_shipping_real]) sur un graphique par mois : Tu verras le délai moyen des commandes passées en janvier (même si elles ont été expédiées en février).

Si tu utilises Avg Shipping Days (by Shipping Date) : Tu verras le délai moyen des commandes physiquement expédiées au mois de janvier.
