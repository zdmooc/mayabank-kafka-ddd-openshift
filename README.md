# MayaBank Kafka DDD OpenShift

Projet de référence **Architecte Solution** de Maya Interlink Solutions autour d'une plateforme bancaire fictive de paiements temps réel.

L'objectif n'est pas d'installer Kafka pour installer Kafka. Le projet part du besoin métier et suit le chemin :

```text
Besoin métier
  -> DDD / Bounded Contexts
  -> Event Storming
  -> Architecture fonctionnelle
  -> Choix synchrone / asynchrone
  -> Décision Kafka vs alternatives
  -> Event-Driven Architecture
  -> OpenShift
  -> Sécurité / Sizing / HA / PRA / Observabilité
  -> GitOps
  -> DAT / Comité d'architecture
```

## Dépôt principal

Ce dépôt est la source de vérité du **cas d'usage MayaBank** : besoins, DDD, décisions d'architecture, contrats d'événements, code des services, tests et manifests de lab permettant les validations locales.

Le dépôt `zdmooc/Maya-gitops` sera utilisé plus tard pour l'industrialisation GitOps/Argo CD, après stabilisation du lab.

## Cas d'usage

1. **MayaBank Instant Payment Platform** — priorité actuelle.
2. **MayaBank International Payments Platform** — phase suivante.
3. **Maya Trading Event Streaming Platform** — phase ultérieure.

Aucune donnée confidentielle BPCE ou autre client réel ne doit être ajoutée. Les scénarios sont fictifs et utilisent uniquement des connaissances génériques ou publiques.

## Anciennes bases Kafka auditées

- `zdmooc/kafka-complete-masterclass`
- `zdmooc/kafka-socle-gouvernance-lab`
- `zdmooc/kafka-data-engineer-kafkaops-topic-service-crc`
- `zdmooc/kafka-expert`

Ces dépôts sont des **sources à auditer**, pas des modèles à recopier automatiquement.

## Statut

### Itération 00 — Audit des dépôts Kafka existants

Statut : **en cours / documentation créée**

Livrables :

- `docs/audit/KAFKA_REPOSITORY_AUDIT.md`
- `docs/audit/KAFKA_DUPLICATION_MATRIX.md`
- `docs/audit/KAFKA_TECH_DEBT.md`
- `docs/audit/KAFKA_MODERNIZATION_PLAN.md`

Aucune installation Kafka n'est réalisée pendant cette itération.

## Principes techniques actuels

- Kafka moderne : **KRaft**, pas ZooKeeper pour une nouvelle architecture.
- OpenShift : lab local sur CRC, distinct de la référence de production.
- Red Hat Streams for Apache Kafka / Strimzi : utiliser les API et versions réellement supportées au moment de l'implémentation.
- Les nouveaux Custom Resources Kafka doivent partir sur l'API stable `kafka.strimzi.io/v1` lorsque la version retenue la supporte.
- Aucun secret réel dans Git.
- Toute configuration de production doit être justifiée par les exigences et le sizing, jamais copiée depuis un lab.

## Méthode de travail

Chaque itération doit produire un état Git récupérable, documenté et testable.

```text
Architecture -> Commit -> git pull -> Test local -> Résultat -> Correction -> Validation
```

Ne pas passer à l'itération suivante tant que l'itération technique courante n'a pas été validée.