# 6) Checklists de conception (prêtes à l’emploi)

**Collecte**

* [ ] SLA de latence défini (batch/stream)
* [ ] Stratégie schéma (Registry, compatibilité)
* [ ] Idempotence + DLQ + replays documentés
* [ ] Sécurité (TLS, ACL, secrets) & *PII tagging*

**Stockage**

* [ ] Zones Bronze/Silver/Gold & règles d’entrée/sortie
* [ ] Format Parquet + table ACID (Delta/Iceberg/Hudi)
* [ ] Partitionnement & politique de **compaction**
* [ ] Catalog central + politiques RLS/CLS

**Traitement**

* [ ] Orchestration (Airflow/Dagster), *retry*, *backfill*
* [ ] Tests data (GE/Deequ), SCD/CDC/UPSERT définis
* [ ] Tuning (AQE, joins, partitions) & coûts surveillés
* [ ] Observabilité (latence, throughput, erreurs)

**Analyse & restitution**

* [ ] Modèles sémantiques, KPI *single source of truth*
* [ ] RLS/CLS, masquage PII, audit
* [ ] Stratégie de refresh (batch vs direct)
* [ ] Alerting & diffusion (emails, webhooks, reverse ETL)
