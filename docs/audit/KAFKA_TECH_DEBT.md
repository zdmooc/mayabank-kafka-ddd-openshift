# Dette technique Kafka — registre priorisé

Ce document liste les dettes techniques identifiées dans les anciens dépôts et leur traitement prévu.

## Priorité P0 — bloquant pour une nouvelle base

### TD-001 — Manifests ZooKeeper encore présents

**Dépôts concernés**

- `kafka-complete-masterclass`
- `kafka-data-engineer-kafkaops-topic-service-crc`

**Problème**

Les manifests de cluster utilisent encore une section ZooKeeper.

**Risque**

Copier ces manifests dans un nouveau projet conduirait à une architecture historique qui ne correspond pas à la cible Kafka moderne sur OpenShift.

**Action**

- ne pas copier ;
- réécrire en KRaft ;
- utiliser `KafkaNodePool` ;
- conserver les anciens fichiers uniquement comme historique/migration si nécessaire.

**Décision** : `REWRITE`.

---

### TD-002 — API Strimzi `v1beta2`

**Dépôts concernés**

- `kafka-complete-masterclass`
- `kafka-data-engineer-kafkaops-topic-service-crc`

**Problème**

Plusieurs `Kafka`, `KafkaTopic`, `KafkaUser`, `KafkaConnect` utilisent `kafka.strimzi.io/v1beta2`.

**Contexte actuel**

Red Hat Streams for Apache Kafka 3.2 introduit l'API stable `v1` et documente la migration de `v1beta2` vers `v1`.

**Action**

Tous les nouveaux manifests MayaBank doivent être écrits pour l'API réellement supportée par l'Operator installé, avec `v1` comme base attendue.

**Décision** : `UPDATE`.

---

### TD-003 — `inter.broker.protocol.version` dans un cluster Kafka 4.1.1

**Dépôt concerné**

- `kafka-socle-gouvernance-lab`

**Problème**

Les profils local et HA configurent :

```yaml
inter.broker.protocol.version: "4.1"
```

Apache Kafka a supprimé cette configuration en version 4.0. En KRaft moderne, la `metadataVersion` sert à gérer le niveau de fonctionnalités/metadata.

**Action**

Supprimer ce paramètre des futurs manifests et revalider la stratégie de version/upgrade.

**Décision** : `DELETE` dans la nouvelle cible.

---

## Priorité P1 — important avant déploiement CRC

### TD-004 — Version Kafka codée en dur sans découverte Operator

**Exemples existants**

- Kafka 3.7.0 dans les anciens manifests ;
- Kafka 4.1.1 dans le socle gouvernance.

**Problème**

La version disponible dépend de la version du Cluster Operator/Red Hat Streams réellement installée sur CRC.

**Action**

Avant déploiement :

1. détecter version OpenShift ;
2. détecter PackageManifest/Subscription ;
3. détecter version Operator ;
4. lire les versions Kafka supportées par ce Cluster Operator ;
5. sélectionner explicitement la version.

**Décision** : `REWRITE deployment discovery`.

---

### TD-005 — Script OperatorHub trop optimiste

**Dépôt concerné**

- `kafka-data-engineer-kafkaops-topic-service-crc`

**Problèmes**

Le script d'installation :

- suppose un `CHANNEL=stable` ;
- suppose `SOURCE=redhat-operators` ;
- prend `.items[0]` comme CSV à surveiller ;
- ne sort pas en erreur explicite si aucun CSV n'atteint `Succeeded` après la boucle.

**Action**

Créer un futur script CRC robuste qui :

- liste les packages disponibles ;
- sélectionne explicitement le package ;
- vérifie source/channel ;
- suit la Subscription installée ;
- contrôle le CSV exact ;
- échoue sur timeout ;
- affiche version Operator et Kafka supportée.

**Décision** : `REWRITE`.

---

### TD-006 — PLAINTEXT présent dans des profils servant de référence

**Dépôt concerné**

- `kafka-socle-gouvernance-lab`

Le profil HA expose à la fois un listener plaintext et un listener TLS.

**Action**

- lab CRC : plaintext éventuellement autorisé uniquement si clairement justifié et isolé ;
- architecture production bancaire : TLS obligatoire par défaut ;
- authentification et autorisation définies par ADR.

**Décision** : `UPDATE`.

---

### TD-007 — Stockage éphémère dans plusieurs labs

**Contexte**

L'éphémère est cohérent pour un lab, mais dangereux s'il est présenté sans distinction comme référence de production.

**Action**

Chaque manifest doit porter une classification explicite :

```text
LAB ONLY
```

ou

```text
PRODUCTION REFERENCE
```

La production utilisera persistent storage dimensionné.

**Décision** : `KEEP for lab / NEVER copy to prod`.

---

## Priorité P2 — architecture et qualité

### TD-008 — Sizing Kafka générique non relié à une volumétrie

**Dépôt concerné**

- `kafka-expert`

Le DDL propose des valeurs CPU/RAM/disque pour controllers et brokers sans calcul métier démontré.

**Action**

Le projet MayaBank doit construire un calcul basé sur :

```text
transactions/s
peak factor
events/transaction
average event bytes
retention
replication
compression
growth
```

Puis seulement proposer un nombre de brokers et des ressources.

**Décision** : `REWRITE sizing methodology`.

---

### TD-009 — Secrets factices en dur dans un DDL

**Dépôt concerné**

- `kafka-expert`

Exemples : `password`, `secure_password`.

**Action**

N'utiliser dans le nouveau dépôt que :

- variables d'environnement ;
- références à Secrets OpenShift ;
- placeholders explicites du type `${SECRET_NAME}` ;
- fichiers `.env.example` sans valeur sensible.

**Décision** : `UPDATE standard`.

---

### TD-010 — Code pédagogique avec TODO

**Dépôt concerné**

- `kafka-expert/python/consumers/basic_consumer.py`

**Action**

Ne pas utiliser comme service de paiement prêt à l'emploi. Les nouveaux services auront :

- traitement métier explicite ;
- gestion d'erreur ;
- idempotence ;
- tests ;
- observabilité.

**Décision** : `KEEP as sample only`.

---

## Priorité P2 — duplication organisationnelle

### TD-011 — Trop de dépôts couvrent les mêmes concepts

Kafka fundamentals, sécurité, observabilité, runbooks et opérations sont répétés.

**Action**

Ne pas fusionner physiquement tout le contenu.

Attribuer un rôle clair à chaque dépôt :

- masterclass = pédagogie ;
- gouvernance = plateforme/RUN ;
- kafkaops-crc = tooling/self-service/CRC ;
- kafka-expert = HLD/DDL technique ;
- mayabank = architecture solution métier + implémentation du cas bancaire.

**Décision** : `KEEP separated with clear responsibilities`.

---

## Priorité P0 fonctionnelle — manque principal

### TD-012 — Absence de chaîne métier → Kafka

Aucun ancien dépôt ne couvre de bout en bout :

```text
Business Requirement
-> DDD
-> Bounded Context
-> Event Storming
-> Interaction Decision
-> Kafka Decision
-> Integration Event
-> Topic
-> Consumer
```

**Action**

C'est la priorité du nouveau dépôt.

**Décision** : `BUILD`.

---

### TD-013 — Absence de Transactional Outbox

Aucun ancien dépôt audité ne fournit un parcours complet Transactional Outbox pour résoudre le dual-write application DB + Kafka.

**Action**

Concevoir puis implémenter dans MayaBank :

```text
Payment Service
-> transaction DB
-> payment + outbox
-> CDC
-> Kafka
```

avec tests de panne et d'idempotence.

**Décision** : `BUILD`.

---

## Critères de sortie de dette pour le nouveau dépôt

Avant le premier vrai déploiement Kafka sur CRC :

- [ ] aucun ZooKeeper dans la nouvelle cible ;
- [ ] aucun `v1beta2` dans les nouveaux manifests si l'Operator retenu supporte `v1` ;
- [ ] aucune configuration Kafka 4.x supprimée ;
- [ ] version Kafka découverte/validée ;
- [ ] lab clairement séparé de production ;
- [ ] aucun secret en clair ;
- [ ] script de validation CRC disponible ;
- [ ] tests smoke reproductibles.