Parfait 👍 voici un **TP complet** avec **énoncé + solution détaillée** pour un exercice de formation :

---

# 📝 Exercice pratique : Construire un plan de pilotage + tableau de bord projet Big Data

---

## 📌 ÉNONCÉ

Vous êtes **chef de projet Big Data** dans une entreprise fictive.
Votre mission est de définir un **plan de pilotage opérationnel** et un **tableau de bord projet**.

Choisissez **un cas d’entreprise** parmi :

1. 🏦 **Banque FinTrust** : mise en place d’une plateforme Big Data de **détection de fraude en temps réel**.

   * Objectif : latence < 2 secondes, 95 % des fraudes détectées.

2. 🛒 **Retail ShopNow** : mise en place d’un **moteur de recommandations** personnalisées omni-canal (web + app + magasins).

   * Objectif : +15 % panier moyen, latence API < 300 ms.

3. 🏭 **Industrie SmartFactory** : mise en place d’une solution de **maintenance prédictive** basée sur l’IoT.

   * Objectif : réduire de 20 % les arrêts machines, SLA alerte < 5 min.

---

### **Travail demandé :**

1. Construire un **plan de pilotage** indiquant :

   * Roadmap (phases principales, durée, jalons).
   * Livrables attendus.
   * Principaux risques identifiés et parades.

2. Proposer un **tableau de bord projet** avec :

   * 3 à 4 **indicateurs de qualité des données**.
   * 3 à 4 **indicateurs de performance technique**.
   * 2 à 3 **indicateurs d’adoption métier**.
   * Suivi budget & risques.

Durée : **45 min de travail en groupe** + **15 min restitution**.

---

## ✅ SOLUTION PROPOSÉE (exemple Banque FinTrust)

### 1️⃣ Plan de pilotage

**Roadmap (6 mois)**

* **Mois 1** : cadrage + business case (jalon : validation sponsor).
* **Mois 2–3** : conception architecture + POC détection temps réel (jalon : POC validé < 2 s).
* **Mois 4** : implémentation pipelines ingestion Kafka + stockage Lakehouse (jalon : ingestion stable 20k events/s).
* **Mois 5** : intégration modèle ML + dashboards BI (jalon : modèle en scoring batch + dashboard risques).
* **Mois 6** : mise en production + monitoring + formation utilisateurs (jalon : 80 % métiers formés).

**Livrables principaux**

* Cahier de cadrage + cartographie données.
* Schéma d’architecture Big Data validé.
* POC détection en temps réel.
* Pipelines ingestion (Kafka → Delta Lake).
* Dashboard BI fraude (Power BI/Tableau).
* Modèle ML scoring en production (via API).
* Documentation + plan de conduite du changement.

**Principaux risques & parades**

* **Scalabilité Kafka** (surcharge) → tests de charge + autoscaling.
* **Qualité données clients** (incomplètes) → règles de validation + Steward.
* **Adoption faible métiers** → quick win : dashboard fraude simple dès M3.
* **RGPD (PII)** → anonymisation données sensibles, audit logs.

---

### 2️⃣ Tableau de bord projet

**KPI – Qualité des données**

* Complétude des transactions ≥ 98 %.
* Taux d’anomalies < 2 %.
* Fraîcheur données (fraude) < 5 secondes.
* Cohérence entre sources (CB vs clients) > 99 %.

**KPI – Performance technique**

* Latence moyenne pipeline (ingestion → décision) ≤ 1,5 s.
* Débit soutenu ≥ 20k events/s.
* SLA disponibilité ≥ 99,9 %.
* Temps de réponse API scoring ≤ 200 ms (p95).

**KPI – Adoption métier**

* % analystes connectés chaque semaine > 80 %.
* Satisfaction utilisateur (enquête) > 8/10.
* Réduction pertes liées à la fraude : -15 % en 6 mois.

**Budget & risques**

* Budget consommé : suivi mensuel vs prévisionnel (+/- 10 %).
* Alertes coûts cloud si dépassement > +15 %.
* KRI (Key Risk Indicators) :

  * Taux de faux positifs > 5 % → alerte.
  * Latence > 3 s → alerte critique.

---

## 🧑‍🏫 Variante pour atelier

* **Groupes** : chaque groupe choisit **Banque**, **Retail**, ou **Industrie**.
* **Livrable** : tableau synthétique (roadmap + dashboard).
* **Restitution** : 5 min par groupe → discussion croisée (comparer différences selon secteur).

---

👉 Résultat attendu : les apprenants comprennent comment **structurer le pilotage d’un projet Big Data** avec une roadmap claire, des livrables identifiés, et un **tableau de bord multi-niveaux (data, technique, métier, budget)**.

---

Veux-tu que je prépare aussi des **modèles visuels (roadmap Gantt + tableau KPI)** prêts à être projetés en formation ou distribués comme gabarits ?
