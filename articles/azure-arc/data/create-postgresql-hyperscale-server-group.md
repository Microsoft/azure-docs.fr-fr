---
title: Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc
description: Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d432f29e91097491fc4719ec59a11cb96948f431
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609052"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Créer un groupe de serveurs PostgreSQL Hyperscale compatible avec Azure Arc

Ce document décrit les étapes de création d’un groupe de serveurs PostgreSQL Hyperscale sur Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Prise en main
Si vous êtes déjà familiarisé avec les rubriques ci-dessous, vous pouvez ignorer ce paragraphe.
Vous souhaiterez peut-être lire les rubriques importantes avant de procéder à la création :
- [Vue d’ensemble des services de données Azure Arc compatibles](overview.md)
- [Modes et exigences de connectivité](connectivity.md)
- [Configuration de stockage et concepts de stockage Kubernetes](storage-configuration.md)
- [Modèle de ressources Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Si vous préférez essayer d’effectuer des opérations sans configurer un environnement complet, commencez rapidement avec [Démarrage rapide Azure Arc](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) sur Azure Kubernetes service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) ou dans une machine virtuelle Azure.


## <a name="login-to-the-azure-arc-data-controller"></a>Se connecter au contrôleur de données Azure Arc

Avant de pouvoir créer une instance, vous devez d’abord vous connecter au contrôleur de données Azure Arc. Si vous êtes déjà connecté au contrôleur de données, vous pouvez ignorer cette étape.

```console
azdata login
```

Vous serez ensuite invité à entrer le nom d’utilisateur, le mot de passe et l’espace de noms système.  

> Si vous avez utilisé le script pour créer le contrôleur de données, votre espace de noms doit être **arc**

```console
Namespace: arc
Username: arcadmin
Password:
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Étape préliminaire et temporaire pour les utilisateurs OpenShift uniquement

Implémentez cette étape avant de passer à l’étape suivante. Pour déployer le groupe de serveurs PostgreSQL Hyperscale sur Red Hat OpenShift dans un projet autre que celui par défaut, vous devez exécuter les commandes suivantes sur votre cluster pour mettre à jour les contraintes de sécurité. Cette commande accorde les privilèges nécessaires aux comptes de service qui exécuteront votre groupe de serveurs PostgreSQL Hyperscale. La contrainte de contexte de sécurité (SCC) **_arc-data-scc_** est celle que vous avez ajoutée lors du déploiement du contrôleur de données Azure Arc.

```console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

_**Server-Group-Name** est le nom du groupe de serveurs que vous allez créer à l’étape suivante._
   
Pour plus d’informations sur SCC dans OpenShift, consultez la documentation [OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html).
Vous pouvez maintenant implémenter l’étape suivante.

## <a name="create-an-azure-database-for-postgresql-hyperscale-server-group"></a>Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale

Pour créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc, utilisez la commande suivante :

```console
azdata arc postgres server create -n <name> --workers <# worker nodes with #>=2> --storage-class-data <storage class name> --storage-class-logs <storage class name> --storage-class-backups <storage class name>

#Example
#azdata arc postgres server create -n postgres01 --workers 2
```

> [!NOTE]
> - **D’autres paramètres de ligne de commande sont disponibles.  Consultez la liste complète des options en exécutant `azdata arc postgres server create --help`.**
> - La classe de stockage utilisée pour les sauvegardes ( _--storage-class-backups -scb_) correspond par défaut la classe de stockage des données du contrôleur de données si elle n’est pas fournie.
> - L’unité acceptée par les paramètres --volume-size-* est une quantité de ressource Kubernetes (un entier suivi d’un de ces suffixes SI (T, G, M, K, m) ou de leurs équivalents de puissance de deux (TI, GI, mi, Ki)).
> - La longueur des noms doit être inférieure ou égale à 12 caractères et conforme aux conventions d’affectation des noms DNS.
> - Vous êtes invité à entrer le mot de passe de l’utilisateur administrateur standard _postgres_.  Vous pouvez ignorer l’invite interactive en définissant la variable d’environnement de session `AZDATA_PASSWORD` avant d’exécuter la commande create.
> - Si vous avez déployé le contrôleur de données à l’aide des variables d’environnement de session AZDATA_USERNAME et AZDATA_PASSWORD dans la même session de terminal, les valeurs d’AZDATA_PASSWORD seront également utilisées pour déployer le groupe de serveurs PostgreSQL Hyperscale. Si vous préférez utiliser un autre mot de passe, vous pouvez (1) mettre à jour la valeur d’AZDATA_PASSWORD ou (2) supprimer la variable d’environnement AZDATA_PASSWORD ou supprimer sa valeur pour être invité à entrer un mot de passe de manière interactive lorsque vous créez un groupe de serveurs.
> - Le nom de l’utilisateur administrateur par défaut pour le moteur de base de données PostgreSQL Hyperscale est _postgresql_ et ne peut pas être modifié à ce stade.
> - La création d’un groupe de serveurs PostgreSQL Hyperscale n’inscrira pas immédiatement les ressources dans Azure. Dans le cadre du processus de chargement de l’[inventaire des ressources](upload-metrics-and-logs-to-azure-monitor.md) ou des [données d’utilisation](view-billing-data-in-azure.md) à Azure, les ressources seront créées dans Azure et vous pourrez voir vos ressources dans le Portail Azure.



## <a name="list-your-azure-database-for-postgresql-server-groups-created-in-your-arc-setup"></a>Répertorier vos groupes de serveurs Azure Database pour PostgreSQL créés dans votre configuration Arc

Pour afficher les groupes de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc, utilisez la commande suivante :

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-database-for-postgresql-server-groups"></a>Obtenir les points de terminaison pour la connexion à vos groupes de serveurs Azure Database pour PostgreSQL

Pour afficher les points de terminaison d’une instance PostgreSQL, exécutez la commande suivante :

```console
azdata arc postgres endpoint list -n <server group name>
```
Exemple :
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

Vous pouvez utiliser le point de terminaison d’instance PostgreSQL pour vous connecter au groupe de serveurs PostgreSQL Hyperscale à partir de votre outil favori :  [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), [pgcli](https://www.pgcli.com/) psql, pgAdmin, etc.

Si vous utilisez une machine virtuelle Azure à tester, suivez les instructions ci-dessous :

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Remarque spéciale sur les déploiements de machines virtuelles Azure

Lorsque vous utilisez une machine virtuelle Azure, l’adresse IP du point de terminaison n’affiche pas l’adresse IP _publique_. Utilisez la commande suivante pour localiser l’adresse IP publique :

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Vous pouvez ensuite combiner l’adresse IP publique avec le port pour établir votre connexion.

Vous devrez peut-être également exposer le port du groupe de serveurs PostgreSQL Hyperscale à l’aide de la passerelle de sécurité réseau (NSG). Pour autoriser le trafic via la passerelle de sécurité réseau (NSG), vous devez ajouter une règle que vous pouvez effectuer à l’aide de la commande suivante :

Pour définir une règle, vous devez connaître le nom de votre groupe de sécurité réseau. Vous déterminez le groupe de sécurité réseau à l’aide de la commande ci-dessous :

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

Une fois que vous avez le nom du groupe de sécurité réseau, vous pouvez ajouter une règle de pare-feu à l’aide de la commande suivante. Les exemples de valeurs ci-dessous créent une règle NSG pour le port 30655 et autorisent la connexion à partir de **n’importe quelle adresse IP source** .  Il ne s’agit pas d'une meilleure pratique de sécurité.  Vous pouvez mieux verrouiller les choses en spécifiant une valeur-source-adresse-préfixes spécifique à votre adresse IP du client ou une plage d’adresses IP qui couvre les adresses IP de votre équipe ou de votre organisation.

Remplacez la valeur du paramètre --destination-port-ranges ci-dessous par le numéro de port que vous avez obtenu à partir de la commande « azdata arc postgres Server List » ci-dessus.

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30655 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="connect-with-azure-data-studio"></a>Se connecter avec Azure Data Studio

Ouvrez Azure Data Studio et connectez-vous à votre instance avec l’adresse IP et le numéro de port du point de terminaison externe, ainsi que le mot de passe que vous avez spécifié au moment de la création de l’instance.  Si PostgreSQL n’est pas disponible dans la liste déroulante *Type de connexion*, vous pouvez installer l’extension PostgreSQL en recherchant PostgreSQL sous l’onglet Extensions.

> [!NOTE]
> Vous devrez cliquer sur le bouton [Avancé] dans le panneau de connexion pour entrer le numéro de port.

N’oubliez pas que si vous utilisez une machine virtuelle Azure, vous aurez besoin de l’_adresse IP_ publique qui est accessible via la commande suivante :

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

## <a name="connect-with-psql"></a>Se connecter avec psql

Pour accéder à votre groupe de serveurs PostgreSQL Hyperscale, transmettez le point de terminaison externe du groupe de serveurs PostgreSQL Hyperscale que vous avez récupéré ci-dessus :

Vous pouvez maintenant vous connecter à psql :

```console
psql postgresql://postgres:<EnterYourPassword>@10.0.0.4:30655
```

## <a name="next-steps"></a>Étapes suivantes

- Lisez les concepts et les guides pratiques d’Azure Database pour PostgreSQL Hyperscale pour distribuer vos données sur plusieurs nœuds PostgreSQL Hyperscale et tirer parti de toute la puissance d’Azure Database pour PostgreSQL Hyperscale. :
    * [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
    * [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
    * [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* Dans les documents ci-dessus, ignorez les sections **Se connecter au portail Azure** et **Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

- [Effectuer un scale-out de votre groupe de serveurs Azure Database pour PostgreSQL Hyperscale](scale-out-postgresql-hyperscale-server-group.md)
- [Configuration de stockage et concepts de stockage Kubernetes](storage-configuration.md)
- [Développement des revendications de volume persistant](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Modèle de ressources Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)