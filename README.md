#  BâtiStock Pro Dashboard — Power BI BTP Analytics

> **Dashboard pédagogique Power BI** · Analyse commerciale complète d'un distributeur de matériaux de construction · Projet conçu pour former des apprenants Data à la BI en contexte BTP.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-003E54?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![BTP](https://img.shields.io/badge/Secteur-BTP-E76F51?style=for-the-badge)

---

##  Contexte

Projet pédagogique conçu pour illustrer un **cas métier BTP end-to-end** : analyse commerciale complète d'un distributeur de matériaux de construction multi-agences.

**Entreprise fictive** : **BâtiStock Pro** — négociant en matériaux BTP avec 10 agences réparties en France (urbaines et rurales), 59 références produits dans 10 familles, 1 200 clients segmentés (Artisans / Entreprises BTP / Particuliers), 3 ans de données (2024-2026).

**Pourquoi ce cas** : le secteur BTP a des **spécificités fortes** (saisonnalité marquée, segments clients hétérogènes, marges serrées, gestion stocks lourde) qui en font un excellent cas pédagogique pour former à l'analyse BI en contexte industriel réel.

---

## 📊 Dashboard livré

Dashboard interactif **3 pages** construit avec Power BI Desktop :

### Page 1 — Pilotage Commercial COMEX
- 5 KPI cards : CA Total, Marge, Taux de Marge, Panier Moyen, Nombre de Ventes
- Courbe mensuelle sur 3 ans avec saisonnalité BTP
- Top 10 familles produits
- Répartition CA par segment client (Artisans / Entreprises BTP / Particuliers)
- Carte / barres par région

### Page 2 — Analyse Performance Agences
- Classement des 10 agences par CA avec RANKX
- Comparaison Urbaine vs Rurale (CA + Marge)
- Évolution mensuelle par type de zone
- Slicers interactifs : Région + Type d'agence

### Page 3 — Analyse Produits & Segments Clients
- Donut CA par famille produit
- Top 10 produits par CA
- Tableau performance par segment client (CA, Nb clients, Panier moyen)
- Saisonnalité trimestrielle par année

---

## 🏗️ Architecture technique

**Modèle en étoile** avec 1 table de faits + 5 dimensions :

```
                       Calendrier (Date Table)
                             |
                             v
   Produits  ---->  VENTES (fait)  <---- Clients
      |              ^      ^
      |              |      |
      v              |      |
  Fournisseurs    Agences
```

**Tables** :
- `Ventes` (fait) : 11 870 lignes de transactions sur 3 ans
- `Produits` (dim) : 59 références dans 10 familles
- `Agences` (dim) : 10 points de vente (6 Rurales + 4 Urbaines)
- `Clients` (dim) : 1 200 clients segmentés
- `Fournisseurs` (dim) : 15 partenaires (Lafarge, Saint-Gobain, Rockwool…)
- `Calendrier` (dim) : 1 096 jours (2024-2026)

---

## 📐 Les 15 mesures DAX implémentées

### Mesures de base
```dax
Nb_Ventes = COUNTROWS ( Ventes )

CA_Total = 
SUMX ( Ventes,
    Ventes[quantite] * RELATED ( Produits[prix_unitaire] ) * ( 1 - Ventes[remise_pct] )
)

Cout_Total = 
SUMX ( Ventes, Ventes[quantite] * RELATED ( Produits[cout_unitaire] ) )

Marge = [CA_Total] - [Cout_Total]

Taux_Marge = DIVIDE ( [Marge], [CA_Total], 0 )

Panier_Moyen = DIVIDE ( [CA_Total], [Nb_Ventes], 0 )

Nb_Clients_Actifs = DISTINCTCOUNT ( Ventes[id_client] )
```

### Mesures Time Intelligence
```dax
CA_YTD = TOTALYTD ( [CA_Total], Calendrier[date] )

CA_N_1 = CALCULATE ( [CA_Total], SAMEPERIODLASTYEAR ( Calendrier[date] ) )

Evolution_YoY = DIVIDE ( [CA_Total] - [CA_N_1], [CA_N_1], 0 )

CA_T = TOTALQTD ( [CA_Total], Calendrier[date] )
```

### Mesures avancées
```dax
Rank_Agence = RANKX ( ALL ( Agences ), [CA_Total], , DESC, DENSE )

Rank_Famille = RANKX ( ALL ( Produits[famille] ), [Marge], , DESC, DENSE )

CA_Segment_Pct = 
DIVIDE ( [CA_Total], CALCULATE ( [CA_Total], ALL ( Clients[segment] ) ), 0 )

Marge_Famille_Pct = 
DIVIDE ( [Marge], CALCULATE ( [Marge], ALL ( Produits[famille] ) ), 0 )
```

---

##  Insights business identifiés

### 5 insights stratégiques issus du dashboard

**1. Saisonnalité BTP très marquée**
Les trimestres T2 (avril-juin) et T3 (juillet-septembre) représentent **~65% du CA annuel** (saison chantiers). Les mois d'hiver (T1 + T4) chutent à ~35% du CA. Impact direct sur la gestion de trésorerie et des stocks.

**2. Les Entreprises BTP dominent le CA (65%)**
Seulement **21% des clients** (253 Entreprises BTP sur 1 200) génèrent **65% du CA (18,29 M€)**, avec un **panier moyen de 7 496 €** — 15× supérieur à celui des Particuliers (488 €). Les Entreprises BTP sont le cœur du business.

**3. L'Outillage électroportatif = famille la plus rentable**
L'outillage représente **23,8% du CA** et affiche le **meilleur taux de marge** (~42%) grâce à des produits à valeur ajoutée (Perceuses, Visseuses, Rabots). Levier d'upsell majeur.

**4. Rurales performantes malgré l'intuition**
Les 6 agences Rurales génèrent **14,6 M€** vs 13,4 M€ pour les 4 Urbaines. Mais à **CA par agence**, les Urbaines dominent (3,35 M€ vs 2,43 M€, +38%) — à prendre en compte dans la stratégie d'expansion.

**5. Marges serrées mais stables**
Taux de marge global de **21,2%**, avec faible variation entre agences (20,3% à 22,2%). Caractéristique du secteur BTP où les volumes compensent les marges unitaires faibles.

---

##  Structure du repo

```
batistock-pro-dashboard/
├── README.md                          # ce fichier
├── LICENSE                            # MIT
├── dataset/
│   └── Projet_BatiStock_Pro.xlsx      # dataset source (6 tables)
├── dashboard/
│   └── Dashboard_BatiStock_Pro.pdf    # export PDF du dashboard final
└── docs/
    ├── Projet_BatiStock_Enonce.pdf    # énoncé pédagogique
    └── Projet_BatiStock_Correction.pdf # correction avec DAX commenté
```

---

##  Comment reproduire le projet

### Pré-requis
- Power BI Desktop (gratuit) · [Télécharger](https://powerbi.microsoft.com/desktop)
- Excel ou Office 365

### Installation

1. Cloner le repo :
```bash
git clone https://github.com/laurentdjiam/batistock-pro-dashboard.git
cd batistock-pro-dashboard
```

2. Ouvrir Power BI Desktop → importer `dataset/Projet_BatiStock_Pro.xlsx`

3. Suivre la correction DAX dans `docs/Projet_BatiStock_Correction.pdf` pour reconstruire les 15 mesures

4. Construire les 3 pages selon le dashboard de référence (`dashboard/Dashboard_BatiStock_Pro.pdf`)

---

##  Usage pédagogique

Ce projet est conçu pour :
- **Formateurs Data** : support de cours Power BI secteur BTP (12-15h)
- **Apprenants Data** : cas pratique sur un secteur industriel réel
- **Recruteurs** : démonstration de compétences BI en contexte sectoriel

**Compétences démontrées** :
- Modélisation en étoile avec 6 tables
- DAX avancé : Time Intelligence (TOTALYTD, SAMEPERIODLASTYEAR), classements (RANKX)
- Analyses sectorielles BTP (saisonnalité, segments B2B/B2C)
- Design dashboard pro avec thème personnalisé
- Documentation pédagogique complète

**Prolongements possibles** :
- Ajout d'une page Prévisionnel avec forecasting
- Analyse rotation stocks par agence
- Modèle de churn clients inactifs

---

##  Auteur

**Laurent NZWENJA DJIAM**
Data Analyst · Analytics Engineer · Consultant Power BI
3+ ans d'expérience · Ingénieur ECE Paris · MSc Data Engineer & IA

- 🔗 [LinkedIn](https://linkedin.com/in/laurent-nzwenja-djiam)
- 🌐 [Portfolio Notion](https://excited-reindeer-05e.notion.site)
- 💻 [GitHub](https://github.com/laurentdjiam)
- 📧 laurentdjiam@gmail.com

---

## 🔗 Projets associés

-  [BoutiqueParis Dashboard](https://github.com/laurentdjiam/boutiqueparis-dashboard) — Dashboard Power BI retail prêt-à-porter
-  [HR Analytics SQL](https://github.com/laurentdjiam/hr-analytics-sql) — 27 requêtes SQL RH analytiques

---

##  Licence

Ce projet est sous licence **MIT** — libre d'utilisation, modification et redistribution pour vos besoins pédagogiques ou professionnels.

Voir [LICENSE](LICENSE) pour les détails.

---

## Si ce projet vous est utile

** Star le repo** pour le partager avec la communauté Data BTP francophone !


*Projet publié en octobre 2026 · Dashboard construit et documenté par Laurent NZWENJA DJIAM*
