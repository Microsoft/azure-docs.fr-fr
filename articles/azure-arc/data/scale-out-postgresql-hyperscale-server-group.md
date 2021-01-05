---
title: Effectuer un scale-out de groupe de serveurs Azure Database pour PostgreSQL Hyperscale
description: Effectuer un scale-out de groupe de serveurs Azure Database pour PostgreSQL Hyperscale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 17bdae658c7095c44a7ae9f30fd85a6c45bf1546
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779972"
---
# <a name="scale-out-your-azure-arc-enabled-postgresql-hyperscale-server-group-by-adding-more-worker-nodes"></a>Effectuer un scale-out de votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc en ajoutant des nœuds Worker
Ce document explique comment effectuer un scale-out d’un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc. Pour ce faire, il vous guide dans un scénario. **Si vous ne souhaitez pas exécuter le scénario mais souhaitez simplement en savoir plus sur la manière d’effectuer un scale-out, allez au paragraphe [Effectuer un scale-out](#scale-out)** .

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="get-started"></a>Prise en main
Si vous êtes déjà familiarisé avec le modèle de mise à l’échelle de PostgreSQL Hyperscale activé par Azure Arc ou Azure Database pour PostgreSQL Hyperscale (Citus), vous pouvez ignorer ce paragraphe. Autrement, nous vous recommandons de commencer par lire ce modèle de mise à l’échelle dans la page de documentation d’Azure Database pour PostgreSQL Hyperscale (Citus). Azure Database pour PostgreSQL Hyperscale (Citus) est la même technologie que celle hébergée en tant que service dans Azure (plateforme en tant que service, ou Paas) au lieu d’être proposée dans le cadre des services de données compatibles Azure Arc :
- [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
- [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
- [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
- [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
- [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
- [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
- [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

> \* Dans les documents ci-dessus, ignorez les sections **Se connecter au portail Azure** et **Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

## <a name="scenario"></a>Scénario
Ce scénario fait référence au groupe de serveurs PostgreSQL Hyperscale créé à titre d’exemple dans la rubrique [Créer un groupe de serveurs PostgreSQL Hyperscale compatible avec Azure Arc](create-postgresql-hyperscale-server-group.md).

### <a name="load-test-data"></a>Charger les données de test
Le scénario utilise un exemple de données GitHub publiquement disponibles, disponibles à partir du [site web Citus Data](https://www.citusdata.com/) (Citus Data fait partie de Microsoft).

#### <a name="connect-to-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Vous connecter à votre groupe de serveurs PostgreSQL Hyperscale compatibles Azure Arc

##### <a name="list-the-connection-information"></a>Afficher la liste des informations de connexion
Connectez-vous à votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc en obtenant d’abord les informations de connexion : Le format général de cette commande est le suivant
```console
azdata arc postgres endpoint list -n <server name>
```
Exemple :
```console
azdata arc postgres endpoint list -n postgres01
```

Exemple de sortie :

```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

##### <a name="connect-with-the-client-tool-of-your-choice"></a>Connectez-vous avec l’outil client de votre choix.

Exécutez la requête suivante pour vérifier que vous disposez actuellement de deux (ou plusieurs nœuds Worker Hyperscale), chacun correspondant à un pod Kubernetes :

```sql
SELECT * FROM pg_dist_node;
```

```console
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(2 rows)
```

#### <a name="create-a-sample-schema"></a>Créer un exemple de schéma
Créez deux tables en exécutant la requête suivante :

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

JSONB est le type de données JSON sous forme binaire dans PostgreSQL. Il stocke un schéma flexible dans une seule colonne et avec PostgreSQL. Le schéma aura un index GIN sur ce schéma pour indexer chaque clé et valeur qu’il contient. Avec un index GIN, l’interrogation de plusieurs conditions directement sur cette charge utile devient rapide et facile. Nous allons donc poursuivre et créer deux index avant de charger nos données :

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

Pour partitionner des tables standard, exécutez une requête pour chaque table. Spécifiez la table à partitionner et la clé sur laquelle partitionner. Nous allons partitionner le tableau des événements et utilisateurs sur user_id :

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

#### <a name="load-sample-data"></a>Charger un exemple de données
Charger les données avec COPY... À PARTIR DU PROGRAMME :

```sql
COPY github_users FROM PROGRAM 'curl "https://examples.citusdata.com/users.csv"' WITH ( FORMAT CSV );
COPY github_events FROM PROGRAM 'curl "https://examples.citusdata.com/events.csv"' WITH ( FORMAT CSV );
```

#### <a name="query-the-data"></a>Interroger les données
Et maintenant mesurez le temps que prend une simple requête avec deux nœuds :

```sql
SELECT COUNT(*) FROM github_events;
```
Prenez note de la durée d’exécution de la requête.


## <a name="scale-out"></a>Scale-out
Le format général de la commande de scale-out est le suivant :
```console
azdata arc postgres server edit -n <server group name> -w <target number of worker nodes>
```

> [!CAUTION]
> La préversion ne prend pas en charge l’annulation de mise à l’échelle. Par exemple, il n’est pas encore possible de réduire le nombre de nœuds Worker. Si vous devez le faire, vous devez extraire/sauvegarder les données, supprimer le groupe de serveurs, créer un groupe de serveurs avec moins de nœuds Worker, puis importer les données.

Dans cet exemple, nous augmentons le nombre de nœuds Worker de 2 à 4, en exécutant la commande suivante :

```console
azdata arc postgres server edit -n postgres01 -w 4
```

Lors de l’ajout de nœuds, vous verrez un état en attente pour le groupe de serveurs. Exemple :
```console
azdata arc postgres server list
```

```console
Name        State          Workers
----------  -------------  ---------
postgres01  Pending 4/5    4
```

Une fois les nœuds disponibles, le rééquilibreur de partition Hyperscale s’exécute automatiquement, et redistribue les données aux nouveaux nœuds. L’opération de scale-out est une opération en ligne. Pendant l’ajout des nœuds et la redistribution des données entre les nœuds, les données restent disponibles pour les requêtes.

### <a name="verify-the-new-shape-of-the-server-group-optional"></a>Vérifier la nouvelle forme du groupe de serveurs (facultatif)
Utilisez l’une des méthodes ci-dessous pour vérifier que le groupe de serveurs utilise à présent les nœuds Worker que vous avez ajoutés.

#### <a name="with-azdata"></a>Avec azdata :
Exécutez la commande suivante :
```console
azdata arc postgres server list
```

Elle retourne la liste des groupes de serveurs créés dans votre espace de noms, et indique le nombre de nœuds Worker. Exemple :
```console
Name        State    Workers
----------  -------  ---------
postgres01  Ready    4
```

#### <a name="with-kubectl"></a>Avec kubectl :
Exécutez la commande suivante :
```console
kubectl get postgresql-12
```

Elle retourne la liste des groupes de serveurs créés dans votre espace de noms, et indique le nombre de nœuds Worker. Exemple :
```console
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   4/4          10.0.0.4:31066      4d20h
```
> [!NOTE]
> Si vous avez créé un groupe de serveurs PostgreSQL de la version 11 au lieu de la version 12, exécutez la commande suivante à la place : _kubectl get postgresql-11_

#### <a name="with-a-sql-query"></a>Avec une requête SQL :
Connectez-vous à votre groupe de serveurs avec l’outil client de votre choix, puis exécutez la requête suivante :

```sql
SELECT * FROM pg_dist_node;
```

```console
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      3 |       3 | pg1-3.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      4 |       4 | pg1-4.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(4 rows)
```

## <a name="return-to-the-scenario"></a>Revenir au scénario

Si vous souhaitez comparer la durée d’exécution de la requête select count avec l’exemple de jeu de données, utilisez la même requête count. Vous pouvez l’utiliser sur les quatre nœuds Worker, sans aucune modification de l’instruction SQL.

```sql
SELECT COUNT(*) FROM github_events;
```
Notez la durée d’exécution.


> [!NOTE]
> En fonction de votre environnement, par exemple si vous avez déployé votre groupe de serveurs de test avec `kubeadm` sur une machine virtuelle à nœud unique, vous pouvez constater une légère amélioration de la durée d’exécution. Pour obtenir une meilleure idée du type d’amélioration des performances que vous pouvez atteindre avec PostgreSQL Hyperscale activé par Azure Arc, regardez les courtes vidéos suivantes :
>* [HTAP haute performance avec Azure PostgreSQL Hyperscale (Citus)](https://www.youtube.com/watch?v=W_3e07nGFxY)
>* [Création d’applications HTAP avec Python & Azure PostgreSQL Hyperscale (Citus)](https://www.youtube.com/watch?v=YDT8_riLLs0)


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [mettre à l’échelle (mémoire, vCores) votre groupe de serveurs PostgreSQL Hyperscale compatibles Azure Arc](scale-up-down-postgresql-hyperscale-server-group-using-cli.md)
- Découvrez comment définir des paramètres de serveur dans votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc
- Lisez les concepts et les guides pratiques d’Azure Database pour PostgreSQL Hyperscale pour distribuer vos données sur plusieurs nœuds PostgreSQL Hyperscale et tirer parti de toute la puissance d’Azure Database pour Postgres Hyperscale. :
    * [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
    * [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
    * [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

 > \* Dans les documents ci-dessus, ignorez les sections **Se connecter au portail Azure** et **Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

- [Configuration de stockage et concepts de stockage Kubernetes](storage-configuration.md)
- [Modèle de ressources Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
