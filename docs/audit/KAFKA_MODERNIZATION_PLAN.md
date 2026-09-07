# Plan de modernisation Kafka

Ce plan définit comment passer des anciens dépôts Kafka au nouveau projet MayaBank sans recopier la dette technique existante.

## Principe directeur

```text
Ancien contenu
  -> Audit
  -> Validation actuelle
  -> Sélection
  -> Adaptation au besoin métier
  -> Test
  -> Intégration dans MayaBank
```

Le nouveau dépôt est un projet **Architecte Solution**, pas une fusion des anciens labs.

## Phase M0 — Geler les responsabilités des dépôts

### kafka-complete-masterclass

Rôle futur : **formation Kafka générale**.

Actions :

- conserver concepts et labs pédagogiques ;
- marquer les manifests ZooKeeper/v1beta2 comme historiques ou les moderniser séparément ;
- ne pas utiliser ce repo comme source de vérité OpenShift production.

### kafka-socle-gouvernance-lab

Rôle futur : **plateforme Kafka / gouvernance / RUN**.

Actions :

- conserver KRaft et `KafkaNodePool` ;
- supprimer `inter.broker.protocol.version` des profils Kafka 4.x ;
- revalider version/metadataVersion ;
- distinguer explicitement lab et production ;
- durcir listeners et sécurité du profil HA.

### kafka-data-engineer-kafkaops-topic-service-crc

Rôle futur : **KafkaOps / CRC / Topic-as-a-Service**.

Actions :

- garder CLI, FastAPI, Ansible, catalogue et runbooks ;
- remplacer le cluster ZooKeeper/v1beta2 par KRaft/v1 ;
- rendre l'installation OperatorHub déterministe ;
- renforcer les vérifications et timeouts.

### kafka-expert

Rôle futur : **HLD/DDL technique et Data Engineering**.

Actions :

- conserver HLD/DDL comme base pédagogique ;
- supprimer les exemples de secrets en dur ;
- remplacer le sizing générique par une méthode calculée ;
- distinguer samples Python de composants production-ready.

## Phase M1 — Construire ce qui manque

Dans `mayabank-kafka-ddd-openshift` :

1. besoin métier Instant Payment ;
2. acteurs et parcours de paiement ;
3. DDD / Bounded Contexts ;
4. Event Storming ;
5. commandes, événements, agrégats, policies ;
6. architecture fonctionnelle sans présupposer Kafka ;
7. matrice synchrone/asynchrone ;
8. Kafka vs REST vs MQ vs gRPC ;
9. ADR de décision ;
10. événements d'intégration ;
11. topic strategy ;
12. partition strategy ;
13. idempotence ;
14. Transactional Outbox / CDC ;
15. OpenShift lab ;
16. sécurité ;
17. observabilité ;
18. résilience ;
19. sizing ;
20. HA/PRA ;
21. DAT et comité d'architecture.

## Phase M2 — Baseline technologique OpenShift/Kafka

Avant toute installation locale :

```text
crc status
oc version
oc get clusterversion
oc get nodes -o wide
oc adm top nodes
oc get storageclass
oc get catalogsource -n openshift-marketplace
oc get packagemanifest | find Kafka/Streams operator
```

Objectif : découvrir ce que le CRC expose réellement.

Aucune version Kafka ou Operator ne doit être supposée à partir des anciens repos.

## Phase M3 — Nouvelle base Kafka CRC

Cible conceptuelle du lab :

```text
OpenShift Local / CRC
  -> Red Hat Streams / Strimzi Operator
  -> KafkaNodePool dual-role
       controller + broker
  -> Kafka cluster KRaft
  -> ephemeral ou petit PVC selon capacité
  -> KafkaTopic
  -> KafkaUser si sécurité activée
```

Le lab minimal ne représente pas la production.

### Contraintes obligatoires

- API CR : `kafka.strimzi.io/v1` si supportée par l'Operator retenu ;
- KRaft ;
- aucune propriété Kafka retirée en 4.x ;
- aucun secret réel ;
- script de vérification ;
- cleanup/redeploy reproductible.

## Phase M4 — Référence production

Architecture séparée du CRC :

```text
Controllers: 3+
Brokers:     3+
Persistent block storage
Replication factor: défini selon criticité
min.insync.replicas: cohérent avec RF
TLS
Authentication
Authorization
Topology awareness
Observability
Capacity planning
```

Les chiffres CPU/RAM/disque seront issus du sizing, pas copiés depuis `kafka-expert`.

## Phase M5 — DDD + cohérence transactionnelle

La première implémentation métier doit résoudre explicitement le dual-write.

Cible à évaluer :

```text
Payment Service
   |
   | transaction locale
   v
Payment DB
  |- payments
  `- outbox
       |
       v
      CDC
       |
       v
     Kafka
```

À comparer avec :

- publication directe ;
- polling publisher ;
- transaction Kafka ;
- orchestration synchrone.

La décision sera consignée dans une ADR.

## Phase M6 — Sécurité bancaire de référence

Principes :

- chiffrement en transit ;
- identité des producers/consumers ;
- moindre privilège ;
- séparation par topic/group ;
- NetworkPolicies ;
- gestion de secrets OpenShift ;
- pas de PLAINTEXT en référence production ;
- audit des accès ;
- rotation des secrets/certificats.

## Phase M7 — Observabilité et SLO

Mesurer au minimum :

- broker/controller health ;
- partition state ;
- ISR ;
- under-replicated partitions ;
- offline partitions ;
- producer error/latency ;
- consumer lag ;
- throughput ;
- CPU/RAM ;
- stockage ;
- saturation.

Les alertes seront reliées aux SLO métier quand ces derniers seront définis.

## Phase M8 — Résilience et PRA

Tester localement ce qui est testable :

- redémarrage broker ;
- consumer indisponible ;
- producer indisponible ;
- doublon ;
- replay ;
- mauvais schema ;
- backlog/lag.

Concevoir séparément la production :

- perte broker ;
- perte worker ;
- perte zone ;
- perte cluster/site ;
- RPO/RTO ;
- réplication inter-cluster si nécessaire.

## Roadmap projet après Itération 00

### Iteration 01

**Kafka Enterprise Use Cases**

Livrables prévus :

- `docs/kafka/KAFKA_ENTERPRISE_USE_CASES.md`
- `docs/kafka/WHEN_NOT_TO_USE_KAFKA.md`
- `docs/kafka/KAFKA_VS_ALTERNATIVES.md`

Aucune installation.

### Iteration 02

**MayaBank Instant Payment — Business Context**

### Iteration 03

**DDD / Bounded Contexts**

### Iteration 04

**Event Storming**

### Iteration 05

**Architecture fonctionnelle et interactions**

### Iteration 06

**Kafka Decision / ADR**

### Iteration 07+

Event design, Outbox/CDC, puis seulement OpenShift/Kafka lab.

## Critère de succès

La modernisation est réussie lorsque nous pouvons expliquer et démontrer :

```text
Pourquoi Kafka ici ?
Pourquoi pas Kafka ailleurs ?
Quel événement ?
Quel topic ?
Quelle clé ?
Quelle cohérence ?
Quelle sécurité ?
Quelle résilience ?
Quel sizing ?
Comment le déployer et le tester sur OpenShift ?
```

sans dépendre d'un ancien manifest copié aveuglément.