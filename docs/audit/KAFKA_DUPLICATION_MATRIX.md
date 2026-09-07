# Matrice de duplication des dépôts Kafka

Cette matrice évite de recopier inutilement du contenu déjà présent dans les anciens dépôts.

| Sujet | complete-masterclass | socle-gouvernance | kafkaops-crc | kafka-expert | Référence retenue pour le nouveau projet |
|---|---:|---:|---:|---:|---|
| Concepts Kafka de base | X | X | X | X | `kafka-complete-masterclass` comme support pédagogique |
| Topics / partitions / offsets | X | X | X | X | masterclass + validation dans MayaBank |
| KRaft | partiel/ancien manifest | X | ancien manifest CRC | X | `kafka-socle-gouvernance-lab` après corrections |
| KafkaNodePool | non dans ancien lab | X | non dans ancien manifest | conceptuel | `kafka-socle-gouvernance-lab` |
| ZooKeeper | X | référence négative/historique | X | historique dans docs | ne pas reprendre pour nouvelle cible |
| Strimzi/OpenShift | X | X | X | faible | socle-gouvernance + kafkaops-crc |
| CRC local | faible | profil local | X | non | `kafka-data-engineer-kafkaops-topic-service-crc` pour scripts/UX, manifests à réécrire |
| HLD | partiel | architecture plateforme | partiel | X | `kafka-expert` comme inspiration, réécrit pour MayaBank |
| DDL | partiel | partiel | partiel | X | `kafka-expert`, mais sizing à reconstruire |
| Sécurité | X | X | X | X | fusion conceptuelle, nouvelle politique MayaBank |
| TLS / mTLS / SASL / ACL | X | X | partiel | X | nouvelle architecture MayaBank basée sur besoin |
| Observabilité | X | X | X | X | socle-gouvernance + kafkaops-crc |
| Consumer lag | X | X | X | X | conserver patterns, créer tests MayaBank |
| RUN / MCO | X | X | X | X | socle-gouvernance + kafkaops-crc |
| Runbooks | X | X | X | X | socle-gouvernance/kafkaops-crc ; pas de copie brute |
| Ansible | non | partiel | X | X | conserver dans anciens repos, pas prioritaire dans MayaBank |
| Topic-as-a-Service | non | gouvernance | X | X | `kafka-data-engineer-kafkaops-topic-service-crc` |
| Kafka Connect | X | X | partiel | X | à introduire seulement si use case validé |
| CDC / Debezium | mention | mention | faible | mention | nouveau projet : conception et implémentation dédiées |
| DDD | - | - | - | - | **à construire ici** |
| Bounded Contexts | - | - | - | - | **à construire ici** |
| Event Storming | - | - | - | - | **à construire ici** |
| Transactional Outbox | - | - | - | - | **à construire ici** |
| Interaction REST vs Kafka vs MQ | - | - | - | - | **à construire ici** |
| Architecture Instant Payment | - | - | - | - | **à construire ici** |
| Idempotence métier paiement | - | - | - | générique Kafka | **à construire ici** |
| Event contracts métier | - | - | - | exemples génériques | **à construire ici** |
| Sizing basé sur volumétrie métier | - | partiel | partiel | sizing générique | **à construire ici** |
| PRA paiement | - | DR plateforme | - | générique | **à construire ici** |

## Règle de consolidation

Le nouveau dépôt ne devient pas une copie des anciens.

```text
Ancien dépôt
   -> comprendre
   -> vérifier
   -> sélectionner le pattern utile
   -> adapter au besoin MayaBank
   -> documenter la décision
```

## Rôles futurs des anciens dépôts

### kafka-complete-masterclass

**Rôle : formation Kafka générale.**

À conserver pour les fondamentaux, mais les manifests OpenShift historiques ne sont pas la référence du projet MayaBank.

### kafka-socle-gouvernance-lab

**Rôle : patterns plateforme Kafka / RUN / gouvernance.**

C'est la meilleure source existante pour KRaft, NodePools, HA, monitoring et gouvernance, après correction des paramètres obsolètes.

### kafka-data-engineer-kafkaops-topic-service-crc

**Rôle : KafkaOps / CRC / self-service.**

Réutiliser les scripts, idées de vérification, Topic-as-a-Service et tooling ; moderniser les manifests Kafka.

### kafka-expert

**Rôle : architecture technique HLD/DDL / Data Engineering.**

Réutiliser les concepts d'architecture, mais remplacer les hypothèses génériques par des exigences métier MayaBank mesurables.

## Ce que `mayabank-kafka-ddd-openshift` devient

Le seul dépôt qui relie de bout en bout :

```text
Business
-> DDD
-> Event Storming
-> Architecture Solution
-> Kafka Decision
-> Implementation
-> OpenShift
-> Testing
-> Production Reference Architecture
```