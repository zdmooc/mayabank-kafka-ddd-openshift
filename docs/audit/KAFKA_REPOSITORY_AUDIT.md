# Audit des dépôts Kafka existants

Date de référence : septembre 2026.

## Périmètre

Dépôts audités :

1. `zdmooc/kafka-complete-masterclass`
2. `zdmooc/kafka-socle-gouvernance-lab`
3. `zdmooc/kafka-data-engineer-kafkaops-topic-service-crc`
4. `zdmooc/kafka-expert`

Objectif : identifier ce qui peut être réutilisé pour `mayabank-kafka-ddd-openshift`, ce qui doit être modernisé et ce qui ne doit pas être recopié.

## Synthèse

| Dépôt | Valeur principale | État | Décision |
|---|---|---|---|
| `kafka-complete-masterclass` | pédagogie Kafka générale, labs, Strimzi, Topics/Users/Connect | utile mais plusieurs manifests OpenShift anciens | **KEEP + UPDATE** |
| `kafka-socle-gouvernance-lab` | meilleure base plateforme : KRaft, `KafkaNodePool`, gouvernance, RUN, sécurité, observabilité, DR | base la plus moderne, mais quelques configurations à corriger/revalider | **KEEP + UPDATE** |
| `kafka-data-engineer-kafkaops-topic-service-crc` | très bon parcours CRC/KafkaOps/Topic-as-a-Service, scripts, Ansible, CLI | déploiement Kafka OpenShift encore basé sur ancien modèle ZooKeeper/v1beta2 | **KEEP + REWRITE OPENSHIFT KAFKA** |
| `kafka-expert` | HLD/DDL, KRaft, Ansible, Python, Elastic/OpenSearch, MCO | bonne matière architecture technique, mais sizing générique et exemples à durcir | **KEEP + EXTRACT + UPDATE** |

## 1. kafka-complete-masterclass

### À conserver

- concepts Kafka : topics, partitions, offsets, consumer groups ;
- ISR, acks, idempotence, transactions ;
- rétention et compaction ;
- sécurité TLS/SASL/ACL ;
- exploitation et troubleshooting ;
- Kafka Connect et ressources Strimzi ;
- labs pédagogiques locaux.

### Problèmes constatés

Le manifest `k8s/strimzi/20-kafka-cluster.yaml` utilise encore :

- `apiVersion: kafka.strimzi.io/v1beta2` ;
- Kafka `3.7.0` ;
- `spec.kafka.replicas` ;
- ZooKeeper ;
- stockage éphémère ;
- listener PLAINTEXT uniquement.

Ce manifest est donc une **référence historique/pédagogique**, pas une base pour notre nouvelle architecture OpenShift.

Les ressources `KafkaTopic`, `KafkaUser` et `KafkaConnect` du même lab utilisent aussi `v1beta2`.

### Décision

- conserver le dépôt comme formation générale ;
- ne pas copier ses manifests Kafka dans le nouveau projet ;
- prévoir sa migration vers `kafka.strimzi.io/v1` + `KafkaNodePool` + KRaft ;
- conserver Redpanda uniquement comme lab Kafka-compatible local, jamais comme substitut implicite à la cible Red Hat Streams en entreprise.

## 2. kafka-socle-gouvernance-lab

### À conserver en priorité

Ce dépôt contient actuellement la base technique la plus proche de la cible :

- `KafkaNodePool` ;
- KRaft ;
- profil local 1 nœud dual-role ;
- profil HA 3 controllers + 3 brokers ;
- stockage persistant dans le profil HA ;
- Topic Operator / User Operator ;
- métriques ;
- Kafka Exporter ;
- gouvernance ;
- sécurité ;
- observabilité ;
- runbooks ;
- DR ;
- reporting.

### Bon pattern de lab

Le profil local :

```text
1 KafkaNodePool
  roles:
    - controller
    - broker
  storage: ephemeral
```

est cohérent pour un **lab CRC contraint**, à condition de le documenter explicitement `LAB ONLY`.

### Bon pattern de production

Le profil HA sépare :

```text
controllers: 3
brokers:     3
```

avec stockage persistant. Cette topologie constitue une bonne base de discussion d'architecture mais ne représente pas un sizing universel.

### Problème concret à corriger

Les manifests Kafka `4.1.1` contiennent :

```yaml
inter.broker.protocol.version: "4.1"
```

Cette configuration a été supprimée d'Apache Kafka 4.0. En KRaft moderne, la compatibilité est pilotée par la metadata version. Ce paramètre ne doit donc pas être repris.

### Points à revalider

- version Kafka exacte : doit correspondre à la version supportée par l'Operator installé ;
- `metadataVersion` : doit être alignée avec le broker version/support matrix ;
- annotations Strimzi héritées : conserver uniquement celles requises par la version réellement installée ;
- listener PLAINTEXT : acceptable éventuellement en lab interne, à supprimer de la référence production ;
- ressources CPU/RAM absentes du manifest HA : à dimensionner à partir du besoin et non à deviner.

### Décision

Ce dépôt devient notre **source technique prioritaire** pour les patterns KRaft / NodePool / gouvernance, après correction et revalidation.

## 3. kafka-data-engineer-kafkaops-topic-service-crc

### À conserver

Très bonne valeur pédagogique et opérationnelle :

- parcours OpenShift Local / CRC ;
- scripts `00-prereqs.sh` à `50-verify.sh` ;
- Topic-as-a-Service ;
- FastAPI ;
- KafkaOps CLI ;
- catalogue déclaratif de topics ;
- Ansible ;
- observabilité ;
- runbooks ;
- smoke tests.

### Problème majeur

Le manifest actuel `openshift/manifests/kafka/kafka-cluster.yaml` est encore basé sur :

- `kafka.strimzi.io/v1beta2` ;
- Kafka `3.7.0` ;
- `spec.kafka.replicas` ;
- ZooKeeper ;
- stockage éphémère.

Il ne doit pas être utilisé tel quel pour notre nouveau lab.

### Risque dans le script OperatorHub

Le script `10-install-strimzi-operator.sh` utilise des valeurs génériques :

```text
CHANNEL=stable
SOURCE=redhat-operators
```

et suppose le premier CSV du namespace pour décider que l'installation est terminée.

À améliorer :

- découvrir réellement PackageManifest/Subscription disponibles ;
- cibler le bon package/operator ;
- vérifier le CSV correspondant à cette Subscription ;
- échouer explicitement après timeout ;
- afficher la version réellement installée.

### Décision

- conserver Topic-as-a-Service, CLI, Ansible, runbooks et logique de vérification ;
- réécrire la partie `openshift/manifests/kafka/` en `v1` + KRaft + `KafkaNodePool` ;
- réécrire le script d'installation afin qu'il détecte le catalogue CRC réel plutôt que supposer un channel/source.

## 4. kafka-expert

### À conserver

- HLD Kafka ;
- DDL Kafka ;
- architecture KRaft ;
- séparation controllers/brokers ;
- sécurité TLS / SASL / mTLS / ACL ;
- Prometheus / Grafana ;
- Ansible ;
- scripts de santé ;
- intégration Elastic/OpenSearch ;
- exemples producer/consumer Python.

### Point positif

Le HLD décrit déjà une architecture KRaft avec séparation des rôles en production :

```text
3 controllers dédiés
3 à 5 brokers (exemple)
```

C'est une base de réflexion valable.

### Problème de sizing

Le DDL contient un sizing générique :

```text
Controller : 4 CPU / 8 Go / 50 Go
Broker     : 8-16 CPU / 32-64 Go / 2-5 To
```

Ces chiffres ne doivent pas être présentés comme une recommandation universelle. Notre nouveau projet construira le sizing depuis :

- transactions/s ;
- pic transactions/s ;
- événements/transaction ;
- taille moyenne d'événement ;
- compression ;
- rétention ;
- réplication ;
- nombre de consumers ;
- croissance.

### Problème sécurité/documentation

Le DDL contient des exemples avec :

```text
ssl_truststore_password => "password"
password => "secure_password"
```

Même s'il s'agit d'exemples, le nouveau dépôt public standardisera :

- variables d'environnement ;
- Secrets OpenShift ;
- `.env.example` sans secret ;
- aucun password en dur dans un exemple production-like.

### Code partiellement pédagogique

Le consumer Python contient encore un `TODO` de logique de traitement. Il doit être considéré comme exemple pédagogique et non comme composant prêt pour une architecture de paiement.

### Décision

Réutiliser les concepts HLD/DDL, mais reconstruire les livrables autour du cas métier MayaBank au lieu de reprendre une architecture Data Analytics générique.

## 5. Ce qui manque dans les quatre dépôts

Les recherches n'ont pas trouvé de véritable implémentation structurée de :

- DDD ;
- Bounded Contexts ;
- Event Storming ;
- Transactional Outbox ;
- décision synchrone/asynchrone flux par flux ;
- matrice Kafka vs REST/MQ/gRPC ;
- architecture métier de paiement instantané ;
- contrats d'intégration basés sur des événements métier ;
- idempotence métier d'un paiement ;
- architecture de cohérence entre base de données et Kafka.

C'est le rôle principal de `mayabank-kafka-ddd-openshift`.

## 6. Référence technologique actuelle

À la date de cet audit, Red Hat Streams for Apache Kafka 3.2 documente :

- l'API stable `kafka.strimzi.io/v1` ;
- `KafkaNodePool` pour les brokers/controllers ;
- KRaft ;
- la migration depuis `v1beta2` ;
- Kafka 4.2.0 dans les exemples Getting Started.

Références :

- https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/3.2/
- https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/3.2/html/getting_started_with_streams_for_apache_kafka_on_openshift/
- https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/3.2/html/deploying_and_managing_streams_for_apache_kafka_on_openshift/

La version réellement installable sur le CRC de test devra néanmoins être découverte depuis son OperatorHub au moment du lab.

## 7. Conclusion de l'audit

Nous ne devons pas créer un cinquième cours Kafka.

Le nouveau dépôt doit devenir le projet de référence pour la chaîne :

```text
Paiement métier
  -> DDD
  -> Event Storming
  -> Architecture fonctionnelle
  -> Interaction Design
  -> Kafka Decision
  -> Event-Driven Architecture
  -> Outbox / CDC
  -> OpenShift / Kafka
  -> Sécurité / HA / PRA / Observabilité
```

La meilleure stratégie est de **réutiliser sélectivement** les anciens dépôts après modernisation, pas de les fusionner physiquement.