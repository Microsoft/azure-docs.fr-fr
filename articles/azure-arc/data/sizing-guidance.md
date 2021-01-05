---
title: Guide de dimensionnement
description: Planifiez la taille d’un déploiement de services de données Azure Arc.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 6e7f2e445c3e4e8df7420c0587e156968f3a2c92
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542675"
---
# <a name="sizing-guidance"></a>Guide de dimensionnement

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="overview-of-sizing-guidance"></a>Vue d’ensemble du guide de dimensionnement

Lorsque vous planifiez le déploiement de services de données Azure Arc, vous devez planifier la quantité correcte de calcul, de mémoire et de stockage requis pour exécuter le contrôleur de données Azure Arc et le nombre de groupes de serveurs de l’instance gérée SQL et de PostgreSQL Hyperscale que vous allez déployer.  Étant donné que les services de données Azure Arc sont déployés sur Kubernetes, vous avez la possibilité d’ajouter de la capacité supplémentaire à votre cluster Kubernetes au fil du temps en ajoutant des nœuds de calcul ou du stockage.  Ce guide vous fournit des conseils sur la configuration minimale requise et des conseils sur les tailles recommandées pour certaines exigences courantes.

## <a name="general-sizing-requirements"></a>Spécifications générales de dimensionnement

> [!NOTE]
> Si vous n’êtes pas familiarisé avec les concepts de cet article, vous pouvez en apprendre davantage sur la [gouvernance des ressources Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) et la [notation de taille de Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

Le nombre de cœurs doit être une valeur entière supérieure ou égale à un.

Lors de l’utilisation d’azdata pour le déploiement, les valeurs de mémoire doivent être spécifiées dans des nombres en puissance de deux, c’est-à-dire avec les suffixes : Ki, Mi ou Gi.

Les valeurs limites doivent toujours être supérieures à la valeur de la requête, si elles sont spécifiées.

Les valeurs limites pour les cœurs sont la mesure facturable sur les groupes de serveurs d’instance gérée SQL et de PostgreSQL Hyperscale.

## <a name="minimum-deployment-requirements"></a>Configuration de déploiement requise

Une taille minimale pour le déploiement des services de données Azure Arc activé peut être considérée comme le contrôleur de données Azure Arc plus une instance SQL gérée plus un groupe de serveurs PostgreSQL Hyperscale avec deux nœuds Worker.  Pour cette configuration, vous avez besoin d’au moins 16 Go de RAM et de 4 cœurs de capacité _disponible_ sur votre cluster Kubernetes.  Vous devez vous assurer de disposer d’une taille de nœud Kubernetes minimale de 8 Go de RAM et de 4 cœurs, ainsi que d’une capacité totale de 16 Go de RAM disponibles pour l’ensemble de vos nœuds Kubernetes.  Par exemple, vous pouvez avoir 1 nœud de 32 Go de RAM et 4 cœurs, ou 2 nœuds de 16 Go de RAM et 4 cœurs chacun.

Pour plus d’informations sur le dimensionnement du stockage, consultez l’article [Configuration du stockage](storage-configuration.md).

## <a name="data-controller-sizing-details"></a>Détails de la taille du contrôleur de données

Le contrôleur de données est une collection de pods déployés sur votre cluster Kubernetes pour fournir une API, le service de contrôleur, le programme d’amorçage et les bases de données de surveillance et les tableaux de bord.  Ce tableau décrit les valeurs par défaut pour les demandes et les limites de mémoire et d’UC.

|Nom du pod|Demande UC|Demande de mémoire|Limite UC|Limite de mémoire|Notes|
|---|---|---|---|---|---|
|**bootstrapper**|100m|100Mi|200m|200Mi||
|**control**|400m|2Gi|1800m|2Gi||
|**controldb**|200m|3Gi|800m|6Gi||
|**controlwd**|10m|100Mi|100m|200Mi||
|**logsdb**|200m|1600Mi|2|1600Mi||
|**logsui**|100m|500Mi|2|2Gi||
|**metricsdb**|200m|800Mi|400m|2Gi||
|**metricsdc**|100m|200Mi|200m|300Mi|Metricsdc est un daemonset créé sur chacun des nœuds Kubernetes de votre cluster.  Les nombres indiqués dans le tableau sont _par nœud_. Si vous définissez allowNodeMetricsCollection = false dans votre fichier de profil de déploiement avant de créer le contrôleur de données, le daemonset metricsdc ne sera pas créé.|
|**metricsui**|20m|200Mi|500m|200Mi||
|**mgmtproxy**|200m|250Mi|500m|500Mi||

Vous pouvez remplacer les paramètres par défaut pour les pds controldb et control dans votre fichier de profil de déploiement ou votre fichier YAML datacontroller.  Exemple :

```yaml
  resources:
    controller:
      limits:
        cpu: "1000m"
        memory: "3Gi"
      requests:
        cpu: "800m"
        memory: "2Gi"
    controllerDb:
      limits:
        cpu: "800m"
        memory: "8Gi"
      requests:
        cpu: "200m"
        memory: "4Gi"
```

Pour plus d’informations sur le dimensionnement du stockage, consultez l’article [Configuration du stockage](storage-configuration.md).

## <a name="sql-managed-instance-sizing-details"></a>Détails de dimensionnement d’une instance gérée SQL

Chaque instance gérée SQL doit avoir les exigences de ressources minimales suivantes :
- Mémoire : 2Gi
- Cœurs : 1

Chaque pod d’instance gérée SQL créé comporte trois conteneurs :

|Nom du conteneur|Demande UC|Demande de mémoire|Limite UC|Limite de mémoire|Notes|
|---|---|---|---|---|---|
|fluentbit|100m|100Mi|Non spécifié|Non spécifié|Les requêtes de ressources de conteneur fluentbit _s’ajoutent_ aux requêtes spécifiées pour l’instance gérée SQL.||
|arc-sqlmi|Utilisateur spécifié ou non spécifié.|Utilisateur spécifié ou non spécifié.|Utilisateur spécifié ou non spécifié.|Utilisateur spécifié ou non spécifié.||
|collectd|Non spécifié|Non spécifié|Non spécifié|Non spécifié||

La taille de volume par défaut de tous les volumes persistants est de 5 Gi.

## <a name="postgresql-hyperscale-server-group-sizing-details"></a>Détails de dimensionnement du groupe de serveurs PostgreSQL Hyperscale

Chaque nœud de groupe de serveurs PostgreSQL Hyperscale doit avoir les exigences de ressources minimales suivantes :
- Mémoire : 256Mi
- Cœurs : 1

Chaque coordinateur de groupe de serveurs PostgreSQL Hyperscale ou pod de travail créé possède trois conteneurs :

|Nom du conteneur|Demande UC|Demande de mémoire|Limite UC|Limite de mémoire|Notes|
|---|---|---|---|---|---|
|fluentbit|100m|100Mi|Non spécifié|Non spécifié|Les requêtes de ressources de conteneur fluentbit _s’ajoutent_ aux requêtes spécifiées pour les nœuds de groupe de serveurs PostgreSQL Hyperscale.|
|postgres|Utilisateur spécifié ou non spécifié.|Spécifié par l’utilisateur ou 256Mi (par défaut).|Utilisateur spécifié ou non spécifié.|Utilisateur spécifié ou non spécifié.||
|telegraf|Non spécifié|Non spécifié|Non spécifié|Non spécifié||

## <a name="cumulative-sizing"></a>Dimensionnement cumulatif

La taille globale d’un environnement requis pour les services de données Azure Arc est principalement une fonction du nombre et de la taille des instances de base de données qui seront créées.  La taille globale peut être difficile à prédire, sachant que le nombre d’instances augmente et diminue et que la quantité de ressources requises pour chaque instance de base de données changera.

La taille de référence pour un environnement de services de données Azure Arc activé est la taille du contrôleur de données, qui requiert 4 cœurs et 16 Go de RAM.  À partir de là, vous pouvez ajouter le total cumulé des cœurs et de la mémoire nécessaires pour les instances de base de données.  Pour les instances gérées SQL, le nombre de pods est égal au nombre d’instances gérées SQL qui sont créées.  Pour les groupes de serveurs PostgreSQL Hyperscale, le nombre de pods est équivalent au nombre de nœuds Worker plus un pour le nœud coordinateur.  Par exemple, si vous avez un groupe de serveurs PostgreSQL avec 3 nœuds Worker, le nombre total de pods sera de 4.

En plus des cœurs et de la mémoire que vous demandez pour chaque instance de base de données, vous devez ajouter 250m de cœurs et 250Mi de RAM pour les conteneurs d’agent.

Voici un exemple de calcul de dimensionnement.

Conditions requises :

- **« SQL1 »** : 1 instance gérée SQL avec 16 Go de RAM, 4 cœurs
- **« SQL2 »** : 1 instance gérée SQL avec 256 Go de RAM, 16 cœurs
- **« Postgres1 »** : 1 groupe de serveurs PostgreSQL Hyperscale avec 4 Workers à 12 Go de RAM, 4 cœurs

Calculs de dimensionnement :

- La taille de « SQL1 » est : 1 pod * ([16 Gi RAM, 4 cœurs] + [250Mi RAM, cœurs 250m]) pour les agents par pod = 16,25 Gi RAM, 4,25 cœurs.
- La taille de « SQL2 » est : 1 pod * ([256 Gi RAM, 16 cœurs] + [250Mi RAM, cœurs 250m]) pour les agents par pod = 256,25 Gi RAM, 16,25 cœurs.
- La taille totale de SQL 1 et de SQL 2 est la suivante : (16,25 Gi + 256,25 Gi) = 272,5 Gi de RAM, (4,25 cœurs + 16,25 cœurs) = 20,5 cœurs.

- La taille de « Postgres1 » est la suivante : (4 pods Worker + 1 pod coordinateur) * ([12 Gi de RAM, 4 cœurs] + [250Mi RAM, cœurs 250m]) pour les agents par pod = 61,25 Gi de RAM, 21,25 cœurs.

- La capacité totale requise pour les instances de base de données est la suivante : 272,5 Gi de RAM, 20,5 cœurs pour SQL + 61,25 Gi de RAM, 21,25 cœurs pour PostgreSQL Hyperscale = 333,75 Gi de RAM, 42,5 cœurs.

- La capacité totale requise pour les instances de base de données et le contrôleur de données est la suivante : 333,75 Gi de RAM, 42,5 cœurs pour les instances de base de données + 16 Gi de RAM, 4 cœurs pour le contrôleur de données = 349,75 Gi de RAM, 46,5 cœurs.

Pour plus d’informations sur le dimensionnement du stockage, consultez l’article [Configuration du stockage](storage-configuration.md).

## <a name="other-considerations"></a>Autres considérations

Gardez à l’esprit qu’une requête particulière de taille d’instance de base de données pour les cœurs ou la RAM ne peut pas dépasser la capacité disponible des nœuds Kubernetes dans le cluster.  Par exemple, si le plus grand nœud Kubernetes dans votre cluster Kubernetes est de 256 Gi de RAM et de 24 cœurs, vous ne pourrez pas créer une instance de base de données avec une requête de 512 Gi de RAM et 48 cœurs.  

Il est judicieux de conserver au moins 25 % de capacité disponible sur les nœuds Kubernetes pour permettre à Kubernetes de planifier efficacement la création de pods et permettre une mise à l’échelle élastique et une croissance à la demande à long terme.  

Dans vos calculs de dimensionnement, n’oubliez pas d’ajouter les besoins en ressources des modules système Kubernetes et de toutes les autres charges de travail qui peuvent partager la capacité avec les services de données Azure Arc activés sur le même cluster Kubernetes.

Pour maintenir une haute disponibilité pendant la maintenance planifiée et la continuité d’activité, vous devez planifier l’indisponibilité d’au moins un des nœuds Kubernetes dans votre cluster à un moment donné.  Kubernetes tente de replanifier les pods qui étaient en cours d’exécution sur un nœud donné qui a été mis hors service à des fins de maintenance ou en raison d’une défaillance.  S’il n’existe aucune capacité disponible sur les nœuds restants, ces pods ne sont pas replanifiés pour la création tant que la capacité n’est pas de nouveau disponible.  Soyez extrêmement vigilant avec les grandes instances de base de données.  Par exemple, s’il n’existe qu’un seul nœud Kubernetes suffisamment grand pour répondre aux besoins en ressources d’une instance de base de données volumineuse et que ce nœud échoue, Kubernetes ne peut pas planifier ce pod d’instance de base de données sur un autre nœud Kubernetes.

Gardez les [limites maximales pour une taille de cluster Kubernetes](https://kubernetes.io/docs/setup/best-practices/cluster-large/) à l’esprit.

Votre administrateur Kubernetes peut avoir configuré des [quotas de ressources](https://kubernetes.io/docs/concepts/policy/resource-quotas/) sur votre espace de noms/projet.  Gardez cela à l’esprit lors de la planification de la taille de vos instances de base de données.
