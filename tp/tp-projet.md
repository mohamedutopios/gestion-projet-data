génial, voici un **scénario d’audit opérationnel** où chaque phase (cadrage → industrialisation) est volontairement incomplète ou bancale. L’objectif pédagogique : vos apprenants doivent **détecter les failles**, **expliquer pourquoi c’est un problème**, **proposer une correction**, et **prioriser** (critique/majeur/mineur).

---

# Scénario (à remettre aux apprenants) — “Opération ORION”

## Contexte rapide

**NovaRetail**, enseigne e-commerce FR/ES, veut piloter en temps quasi-réel le **taux de retours**, la **fraude**, et la **performance logistique** avant **le pic de fin d’année**. Un socle data est lancé : events web/app (Kafka), batch ERP/WMS/CRM, data lake (S3), entrepôt analytique, dashboards BI, et une API ML “fraude”.

**Calendrier cible** : lancement 1er “MVP” dans **8 semaines**. Équipe : PO Data (Claire), 2 Data Engineers (vous + Léo), 1 BI (Maya), 1 MLE (Imane), 1 Data Steward (Rami). Côté gouvernance : DPO (Sofia), RSSI (Victor). Budget annoncé par la DSI “dans les standards”.

> Votre mission : **piloter opérationnellement** et **challenger** ce qui suit.
> Pour chaque phase : repérez ce qui cloche / manque, expliquez l’impact, proposez un correctif priorisé.

---

## 1) Cadrage (extraits livrés par l’équipe projet)

* **Objectifs**

  * “Réduire les retours et la fraude rapidement.”
  * “Avoir de meilleurs dashboards.”
  * “Mettre une API ML pour la fraude.”
* **KPIs**

  * “Taux de retours : à suivre.”
  * “Fraude : chargebacks à surveiller.”
  * “Logistique : mieux livrer.”
* **Périmètre**

  * Sources : ERP, WMS, CRM, Web events, passerelle de paiement.
  * Hors périmètre : “rien pour l’instant”.
* **Hypothèses**

  * “Les équipes métiers sont disponibles.”
  * “Les fournisseurs d’API tierces ne changeront pas leurs schémas.”
* **ROI (slide unique)**

  * “Économies estimées : élevées. Coûts : dans le budget.”
  * Détail chiffré non fourni (à venir).
* **Planification**

  * 2 sprints de cadrage/conception (2 semaines), 2 sprints d’implémentation (2 semaines), 1 sprint de validation/déploiement (1 semaine), 1 sprint “buffer” (1 semaine).
  * “Vacances d’équipe non prises en compte” (note interne).

> **Livrables joints** : un cahier des charges de 3 pages, un tableau des tâches non priorisé, aucun risque listé.

---

## 2) Conception (extraits)

* **Architecture dessinée au tableau** (photo floue jointe) :

  ```
  App/Web -> Kafka -> Spark -> S3 (Bronze/Silver/Gold) -> BI
                                 -> API ML (fraude)
  ```

  * Pas de détails sur **IAM**, **réseaux**, **sécurité**.
  * “Chiffrement S3 activé par défaut ?” (post-it).
  * “Pseudonymisation à vérifier avec DPO” (post-it).
* **Modèle de données Gold**

  * Tables listées : `orders`, `returns`, `payments`, `customers`.
  * Dictionnaire partiel (quelques colonnes décrites, pas d’owner).
* **RACI**

  * “Tout le monde **R** (responsable) sur les pipelines et la sécu. PO **A** (accountable) sur tout.”
  * DPO non mentionné, RSSI non mentionné.
* **Plan RGPD/Sécu**

  * “Consentement déjà géré par l’équipe web.”
  * “Logs d’accès à mettre plus tard.”
  * Rétention “par défaut S3”.

---

## 3) Implémentation (extraits)

* **Pipelines**

  * Streaming Spark : consommation d’un topic `events` (tous types d’événements mélangés).
  * Pas de **contrat de schéma** formel ; “on parse au vol”.
  * Mode `append`, pas de **dedup**, pas de **DLQ**.
  * Checkpoint local sur VM de dev “en attendant la prod”.
* **Batch**

  * Import ERP/WMS/CRM via scripts Python “temporaires” (cron à venir).
  * DBT évoqué “si on a le temps”.
* **Data Lake**

  * S3 avec `/bronze`, `/silver`, `/gold`.
  * Partitionnement : “on verra selon les requêtes BI”.
  * Nommage : mix FR/EN (ex : `/retours_par_mois/` et `/returns_daily/`).
* **Terraform**

  * Fichiers répartis dans `/infra/tmp/`.
  * Pas de backend de state (local).
  * Providers non versionnés.
  * Variables “à mettre dans le code pour aller plus vite”.
  * Aucun tag projet / environnement.

---

## 4) Validation (extraits)

* **Tests**

  * “Unitaires : à écrire.”
  * “Rejeu : pas nécessaire si tout passe du premier coup.”
  * Perf : “on verra en prod avec le vrai trafic.”
  * Sécu : “IAM par défaut du compte cloud, ok.”
* **Qualité des données**

  * “Les champs principaux ont l’air remplis.”
  * Aucune métrique de complétude, unicité, conformité.
  * Pas de seuils ni d’alertes.
* **Recette métier**

  * Réunion prévue “après le Black Friday” (calendrier chargé).

---

## 5) Déploiement (extraits)

* **Stratégie**

  * “Cutover” un dimanche soir.
  * Pas de **rollback** documenté.
  * “On bascule les dashboards au moment du go-live.”
* **API ML Fraude**

  * Endpoint non documenté ; “Imane a une version locale.”
  * AuthN “à ouvrir au réseau interne pour commencer.”
* **Communication**

  * Aucun plan d’annonce / support.
  * Pas d’astreinte ni d’on-call défini.

---

## 6) Industrialisation (extraits)

* **Monitoring**

  * “On installera Grafana après la 1re semaine de prod.”
  * Prometheus “si nécessaire”.
  * Pas de métriques clés définies (lag Kafka, latence API, SLA rafraîchissement tables).
* **Runbooks / Playbooks**

  * “On saura réagir, l’équipe est expérimentée.”
  * Pas de procédures écrites (incident, RGPD, escalade).
* **Coûts / Capacity**

  * “Surveillance des coûts à faire par la DSI” (non planifiée).
  * Pas de prévision de volumétrie, pas d’auto-scaling défini.

---

### Gabarit de réponse attendu (à remplir par équipe)

Pour chaque phase :

1. **Anomalie / manque** (fait précis, citation si utile)
2. **Pourquoi c’est un problème** (impact métier/tech/sécu/rgpd/coûts/délais)
3. **Correction proposée** (solution concrète)
4. **Priorité** (Critique / Majeur / Mineur)
5. **Propriétaire & délai** (RACI + date)

> Format conseillé (copiez-collez) :

```
[Phase] [Anomalie] — [Impact] — [Correction] — [Priorité] — [Owner] — [Échéance]
```

---

# Guide formateur (ne pas donner aux apprenants)

## 1) Cadrage — problèmes à identifier (exemples attendus)

* Objectifs non SMART (pas de cible chiffrée, ni délai, ni population).
* KPIs vagues (pas de définition ni formule : ex. “taux de retours” sans numérateur/dénominateur).
* Périmètre sans exclusions (risque d’**scope creep**).
* Aucune **cartographie des risques** (fournisseurs API, pic trafic, vacances, dépendances IT).
* ROI non chiffré, pas d’hypothèses → **business case** invérifiable.
* Capacité/charge non prise en compte (vacances, charges transverses).
* Pas d’engagement métiers (validation continue absente).
* Pas de contraintes non fonctionnelles (SLO latence API, SLA rafraîchissement BI).
* Aucune **rétention**/légalité mentionnée (RGPD).
* **Action** : formuler objectifs SMART, lister KPIs avec définitions, matrice risques, jalons avec validations métiers, hypothèses ROI documentées, décisions Go/No-Go.

## 2) Conception — problèmes à identifier

* Architecture trop high-level : pas de **réseau privé**, pas de **NACL/SG**, pas de **IAM RBAC**, pas de **KMS**, pas de **Secrets Manager**.
* Pas de **catalogue de données** (owner, sensibilité, SLA).
* Dictionnaire de données incomplet (pas de types, clés, SCD).
* RACI incohérent (tout le monde R, PO A sur tout, DPO/RSSI absents).
* RGPD flou : consentement supposé, pas de **pseudonymisation** hors prod, pas de **registre** de traitement.
* Rétention “par défaut” → non-conforme.
* **Action** : diagrammes détaillés (réseau/IAM/chiffrement/observabilité), RACI réaliste, PIA/AIPD, plan de rétention & minimisation, data catalog + SLA.

## 3) Implémentation — problèmes à identifier

* Topic unique `events` (anti-pattern) → mélange de schémas, couplage fort.
* Absence de **contrats de schéma** (Avro/JSON Schema + registry) → casse silencieuse.
* Pas de **DLQ**, pas de **dedup**, pas d’**idempotence** → doublons/pertes.
* Checkpoints locaux → non reproductible / pas de HA.
* Partitionnement “plus tard” → perf & coûts dégradés.
* Nommage incohérent → dette opérationnelle/BI.
* Terraform sans **backend** distant, sans **version pinning**, **secrets en dur**, **no tags** → risque d’état perdu, dérive, sécurité.
* CI/CD absent (lint, fmt, plan, apply, approvals).
* **Action** : séparer topics par domaine, schema registry & compat policy, DLQ + retry/backoff, exactly-once/idempotence, checkpoints gérés (S3/HDFS), conventions de nommage, partitionnement par date/pays, TF backend (remote), pin providers, variables/TFVARS + secrets manager, pipelines CI/CD.

## 4) Validation — problèmes à identifier

* Tests inexistants (unitaires/intégration/charge/chaos/reprise).
* “Perf en prod” → inacceptable (risque de panne lors du pic).
* IAM “par défaut” → **principe du moindre privilège** violé.
* Qualité sans métriques ni seuils → dashboards faux sans le savoir.
* Recette métier **après** pic → gouvernance inversée.
* **Action** : plan de tests complet, critères d’acceptation, tests de charge (lag Kafka, throughput), tests de reprise (broker down), data quality (complétude/conformité/unicité) avec seuils et **alertes**, revues métiers itératives.

## 5) Déploiement — problèmes à identifier

* Stratégie “big bang”, pas de **blue/green** ni **canary**, pas de **rollback**.
* Basculer BI au go-live sans flag → risque d’arrêt décisionnel.
* API ML sans **contrat** ni **authN/Z** → dette + faille sécu.
* Comms/Support absents → métiers pris de court, N1/N2 non prêts.
* Pas d’astreinte → MTTA/MTTR élevés.
* **Action** : runbook go-live, plan de rollback, blue/green, feature flags BI, contrat OpenAPI + auth (OIDC/JWT), quotas, SLOs (p95), planning communication, astreinte.

## 6) Industrialisation — problèmes à identifier

* Monitoring “après” prod → inverse.
* Métriques clés absentes (consumer lag, latence API, taux d’erreur, SLA rafraîchissement, coûts).
* Pas de playbooks (Kafka dégradé, schéma cassé, DLQ qui grossit, fuite PII, escalade RGPD).
* FinOps absent (budgets/alertes de coût, tags).
* Capacity planning et auto-scaling non définis.
* **Action** : Prometheus exporters (Kafka, Spark, API, infra), dashboards Grafana, alerting (seuils + durée), runbooks versionnés, FinOps (budgets/alerts, tags), revues hebdo de capacité.

---

## Barème d’évaluation (suggestion)

* **Exhaustivité** des anomalies (40 %)
* **Pertinence** des impacts (20 %)
* **Qualité & réalisme** des corrections (30 %)
* **Priorisation & RACI** (10 %)

---

## Fiche de relevé d’anomalies (modèle à distribuer)

```
Phase | Anomalie précise | Impact | Correction proposée | Priorité | Owner | Échéance
----- | ---------------- | ------ | ------------------- | -------- | ----- | --------
Cadrage | KPI non défini pour "taux de retours" | Décisions BI faussées | Définir formule: retours validés / commandes livrées, seuil 10%, périodicité J+1 | Critique | Data Steward | 72h
...
```

---

Souhaite-tu que je te génère **les supports prêts à imprimer** (brief apprenant en 2 pages + grille de correction formateur + fiche anomalies en .docx/.xlsx) ?
