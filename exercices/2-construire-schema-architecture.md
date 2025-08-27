Génial—voici un **TP complet** avec **plusieurs énoncés** (au choix) + **corrections très détaillées**. Tout est prêt pour un atelier collaboratif (travail en sous-groupes, restitution, grille d’évaluation).

---

# Atelier collaboratif – Construire un schéma d’architecture Big Data cible

## Modalités générales (pour tous les cas)

* **Durée** : 1h30 (45’ travail en groupe + 30’ restitution + 15’ feedback)
* **Livrables attendus** :

  1. Schéma d’architecture (ASCII, Miro/Draw\.io ou tableau blanc)
  2. Justification des choix (collecte, stockage, traitement, gouvernance/sécurité)
  3. Flux de données (du “source → consommation”), latences cibles, SLA/SLO
  4. Plan de qualité données & RGPD (catalogue, PII, anonymisation)
  5. Plan d’industrialisation (orchestration, observabilité, coûts/risques)
* **Barème d’évaluation (20 pts)**

  * Pertinence fonctionnelle (5) — correspondance avec les besoins/contraintes
  * Robustesse technique (5) — scalabilité, résilience, coûts
  * Gouvernance & sécurité (4) — RGPD, PII, rôles, lineage
  * Clarté du schéma & flux (3)
  * Mesure (SLA/KPI) & plan d’industrialisation (3)

---

## Cas 1 — Banque **FinTrust** : détection de fraude < 2 s

### ÉNONCÉ

* **Besoins** : détecter les fraudes carte en **quasi temps réel** (<2 s du swipe), alimenter une **BI J+1** et tracer les décisions pour audit.
* **Sources** : transactions CB (20k msg/s pic), géoloc, device fingerprint, référentiels clients (SQL), historique 3 ans.
* **Contraintes** : **exactly-once** sur décisions, **traçabilité** (qui a décidé/quel modèle), **PII** (RGPD), 99.9% disponibilité.
* **Attendus** :

  * Schéma cible (collecte, streaming, stockage, features ML, scoring, restitution)
  * Choix Lake/Lakehouse/DW + CDC éventuel
  * Gouvernance : catalogage, masquage PII, RLS/CLS
  * KPI : latence E2E, taux de faux positifs, fraîcheur données

### CORRECTION (proposition de référence)

**Architecture (ASCII)**

```
[POS/Payment GW] --(TLS)-> [Kafka (topics: tx_raw, geo, device)]
      |                                 |
      |                      [Schema Registry - Avro]
      |                                 |
      +--> [FDS - CDC Debezium (clients, cartes) -> Kafka topics cdc_*]
                     |
[Kafka Streams/ Spark Structured Streaming]
     |  - enrichissement (join cdc_*)
     |  - features temps réel (sliding windows)
     |  - scoring modèle (gRPC/ML serving) + règles
     v
[Delta/Iceberg - Bronze (tx_raw)] --> [Silver (tx_enriched)] --> [Gold (fraud_decisions)]
     |                                                         |
     |                                                         +--> [DW/BI J+1 (dbt) : rapports risques]
     +--> [DLQ topics/tables]                 [MLflow + Feature Store + Registry modèles]
```

**Choix clés**

* **Streaming** : Kafka (+ Schema Registry) ; **Spark Structured Streaming** pour enrichissement fenêtré, agrégats, **exactly-once** via sinks ACID (Delta/Iceberg) + transactions.
* **Stockage** : **Lakehouse (Delta/Iceberg)** pour ACID, time travel, **UPSERT**; tables **Bronze/Silver/Gold**.
* **ML** : features calculées en flux + **serving** (microservice gRPC/REST) ; **MLflow** (tracking, versions).
* **BI** : réplication **Gold** vers DW (ex. BigQuery/Snowflake/Synapse) pour J+1 via **dbt**.
* **Gouvernance** : Purview/Glue/Atlas (catalogue + lineage) ; **masquage PII** (hash email, token PAN), **RLS/CLS** en BI.

**Tables & partitions**

* `tx_raw_bronze` (event\_time partition J/H)
* `tx_enriched_silver` (event\_date, country)
* `fraud_decisions_gold` (event\_date, decision) — conserver **features + scores** pour audit.

**SLA/Sécurité**

* Latence E2E **≤ 1.5 s** (buffer 500 ms pour scoring).
* Kafka TLS + SASL, ACL topics ; KMS au repos ; **RBAC/ABAC** (tags PII).
* **Audit** : journal des versions modèle utilisées + décision.

**KPI**

* Taux fraude détectée, **faux positifs** (<2%) ; disponibilité 99.9% ; fraîcheur Gold < 5 min (streaming micro-batch 1 min).

**Alternatives**

* Flink au lieu de Spark (streaming natif très faible latence).
* Feature Store managé (Vertex/Tecton).

---

## Cas 2 — Retail **ShopNow** : recommandations omni-canal + stocks

### ÉNONCÉ

* **Besoins** : recommandations produits **web + mobile** en < 300 ms, prévision de demande (J+1 et H+1), disponibilité stock magasin en temps quasi réel.
* **Sources** : navigation (clickstream 10k/s), achats (SQL/ERP), catalogue, météo/événements (API).
* **Contraintes** : coûts cloud maîtrisés, saisonnalité forte, A/B testing, RGPD (opt-in tracking).
* **Attendus** :

  * Schéma cible temps réel (reco) + batch (prévision)
  * Lake vs EDW vs Lakehouse, choix orchestrateur
  * Exposition des reco (API), cache & SLA
  * Gouvernance consentement (opt-in/opt-out)

### CORRECTION (proposition de référence)

**Architecture (ASCII)**

```
[Web/Mobile SDK] -> [Kafka: clicks]       [ERP/SQL] --(CDC)-> [Kafka: orders_cdc]
        |                         \        [Catalog API] ----> [Batch ingest -> Bronze]
[Stream proc (Spark/Flink)] -------> [Silver clickstream, sessions] 
                                     [Join orders/catalog -> Gold features]
ML:
  - Offline training (Spark/MLlib/TF) on Gold
  - Model registry (MLflow)
Serving:
  [Realtime Reco API + Redis cache]  <--- [Feature Store online]
BI & Planif:
  [DW/BI (dbt)]  <- Gold (prévisions J+1/H+1)
Governance:
  [Catalog + Consent store (opt-in/out)]
```

**Choix clés**

* **Lakehouse** (Delta/Iceberg) pour unifier clickstream/achats/catalogue.
* **Streaming** pour sessions & signaux récents → **features online** en **Redis** (faible latence) ; **API** de reco (p95 < 300 ms).
* **Batch** pour entraînement quotidien et **prévisions** (dbt + Spark).
* **Orchestration** : Airflow/Dagster ; **A/B testing** (flags, buckets).

**SLA & coûts**

* p95 reco **< 300 ms** (Redis chaud, fallback règles).
* Compaction planifiée (small files), Z-Order par `product_id`.
* Autoscaling K8s/Serverless pour pics.

**RGPD & consentement**

* **Consent store** central ; filtres de collecte si opt-out.
* Pseudonymisation ID client, minimisation champs.

**KPI**

* Uplift CTR/CR, panier moyen, stockout rate ↓, taux d’adoption reco.

---

## Cas 3 — Industrie **SmartFactory** : maintenance prédictive IoT

### ÉNONCÉ

* **Besoins** : anticiper pannes machines, alerter opérateurs en **< 5 min**, réduire arrêts.
* **Sources** : capteurs **edge** (télémétrie 50k msg/s), tickets maintenance (ITSM), référentiel machines.
* **Contraintes** : connectivité intermittente, edge computing, **souveraineté** (sites UE), stockage 5 ans.
* **Attendus** :

  * Architecture **Edge → Cloud** (buffer, agrégation)
  * Pipeline features & modèles, boucle MLOps
  * Gouvernance (catalogue, qualité capteurs, DLQ)

### CORRECTION (proposition de référence)

**Architecture (ASCII)**

```
[Capteurs] -> [Edge Gateway (MQTT)] -> (TLS/VPN) -> [Kafka central]
        \-- Local buffer (disk) w/ retry
[Stream proc (Flink/Spark)] -> [Silver signals] -> [Gold anomalies]
                   |                         \
               [Feature gen]              [Alerts -> Ops (Teams/Slack)]
ML:
  - Offline training (historique) -> model registry
  - Online scoring (stream) + thresholds adaptatifs
Storage:
  Lakehouse (Delta/Iceberg): Bronze(raw), Silver(clean), Gold(anom/alerts)
BI:
  Dashboards OEE, MTBF/MTTR (DW/BI)
Governance:
  Catalog, quality rules (missing, flatlines), DLQ
```

**Choix clés**

* **Edge gateway** (MQTT) avec **buffer** local + **retransmission**.
* **Kafka** central + **DLQ** ; **Flink** pour *event-time* strict, retard capteurs.
* **Lakehouse** pour historisation longue + ACID.
* **MLOps** : MLflow, dérive données/modèles, **shadow mode** avant go-live.

**Qualité & SLA**

* Règles capteurs : *flatline*, outliers, *stale data*.
* Latence **< 2 min** collecte→alerte ; disponibilité 99.9%.

**Sécurité**

* Certificats mutual TLS, VPN site-à-site ; chiffrement at rest ; RBAC par usine.

---

## Cas 4 — Santé **MedResearch** : data hub anonymisé pour IA

### ÉNONCÉ

* **Besoins** : créer un **hub de données** pour recherche IA (images DICOM, notes cliniques), **anonymisé**, traçable ; accès sécurisé par projet.
* **Sources** : SIH (HL7/FHIR), PACS (DICOM), labo (CSV), registres externes.
* **Contraintes** : **RGPD**, consentement, pseudonymisation réversible (sous conditions), audit complet, RLS par étude.
* **Attendus** :

  * Architecture Lake/Lakehouse + DW clinique
  * Chaîne d’anonymisation et gouvernance (catalog, lineage, accès projet)
  * Coffre de clés, procédures d’accès R\&D

### CORRECTION (proposition de référence)

**Architecture (ASCII)**

```
[SIH FHIR/HL7] [PACS DICOM] [Lab CSV]
      \           |            /
       -> [Ingestion contrôlée -> Bronze] --(Policy)--> [Anonymisation/Pseudonymisation]
                                |                         |
                        [Silver clinical, dicom meta]     |
                                \                         v
                                 --> [Gold datasets by Study/Project]
Governance & Security:
  [Data Catalog + Lineage] [KMS/HSM] [RBAC/ABAC] [PII tags] [Access workflows]
Analytics/AI:
  - DW clinique (BI qualité soins)
  - Lakehouse pour IA (images + textes), notebooks isolés (VPC)
Audit:
  - Journal complet (qui/quoi/quand), time travel
```

**Choix clés**

* **Chaîne d’anonymisation** (tokens, suppression DICOM PHI, hashing).
* **Projects workspaces** isolés réseau ; **catalogue** + **lineage** obligatoire.
* **Lakehouse** pour *time travel* et *legal hold*.
* **DW** pour indicateurs qualité des soins.

**RGPD & accès**

* DPO impliqué ; **registre des traitements** ; **Privacy by Design** ; **Data Access Committee**.
* **RLS** par étude, **contrats de données** par projet.

---

# Modèle de restitution (groupe → formateur)

1. **Contexte & besoins** (2–3 minutes)
2. **Schéma** (collecte→stockage→traitement→consommation)
3. **Décisions clés** (batch/stream, Lake vs DW, ML serving, gouvernance)
4. **SLA/SLO & KPI** (latence, fraîcheur, coûts cibles)
5. **Sécurité & RGPD** (PII, RLS/CLS, audit)
6. **Risques & parades** (small files, skew, schéma, vendor lock-in)

---

# Checklist de qualité (à fournir aux groupes)

* [ ] Latence cible, volumétrie et pics précisés
* [ ] Topics/queues + schémas + compatibilité (Registry)
* [ ] Zones **Bronze/Silver/Gold** + format (Parquet + Delta/Iceberg/Hudi)
* [ ] Orchestrateur (Airflow/Dagster), tests data (Great Expectations/Deequ)
* [ ] Observabilité (SLA fraîcheur, métriques, DLQ, logs)
* [ ] Sécurité (TLS, KMS, RBAC/ABAC, PII tags, RLS/CLS)
* [ ] BI/ML : datamarts, feature store, registry modèles
* [ ] Coûts & opérations (compaction, autoscaling, S3/ADLS policies)

---

Si tu veux, je peux convertir ces cas en **supports imprimables** (PDF/MD) + **gabarits de schémas** (ASCII) et **grilles d’évaluation** prêtes à distribuer en classe.
