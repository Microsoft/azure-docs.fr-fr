---
title: Migrer d’Azure Red Hat OpenShift 3.11 vers Azure Red Hat OpenShift 4
description: Migrer d’Azure Red Hat OpenShift 3.11 vers Azure Red Hat OpenShift 4
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 08/13/2020
keywords: migration, aro, openshift, red hat
ms.openlocfilehash: 322c0cf5ece2a9c950e71b947e2aa6088a165cb8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89469743"
---
# <a name="migrate-from-azure-red-hat-openshift-311-to-azure-red-hat-openshift-4"></a>Migrer d’Azure Red Hat OpenShift 3.11 vers Azure Red Hat OpenShift 4

Azure Red Hat OpenShift sur OpenShift 4 apporte Kubernetes 1.16 sur le système d’exploitation Red Hat Core, des clusters privés, la prise en charge des fonctionnalités Apportez votre propre réseau virtuel et le rôle complet d’administrateur de cluster. De plus, de nombreuses nouvelles fonctionnalités sont désormais disponibles, telles que la prise en charge de l’infrastructure des opérateurs, l’opérateur Hub et le maillage de service OpenShift.

Pour réussir la transition d’Azure Red Hat OpenShift 3.11 à Azure Red Hat OpenShift 4, veillez à passer en revue les [différences en matière de stockage, de mise en réseau, de journalisation, de sécurité et de surveillance](https://docs.openshift.com/container-platform/4.4/migration/migrating_3_4/planning-migration-3-to-4.html).

Dans cet article, nous allons montrer comment migrer d’un cluster Azure Red Hat OpenShift 3.11 à un cluster Azure Red Hat 4.

> [!NOTE]
> Les outils de migration Red Hat OpenShift tels que l’outil d’assistance à la migration du plan de contrôle et l’outil de migration d’application en cluster (CAM) ne peuvent pas être utilisés avec les clusters Azure Red Hat OpenShift 3.11.

## <a name="before-you-begin"></a>Avant de commencer

Cet article part du principe que vous disposez d’un cluster Azure Red Hat OpenShift 3.11 existant.

## <a name="create-a-target-azure-red-hat-openshift-4-cluster"></a>Créer un cluster Azure Red Hat OpenShift 4 cible

Tout d’abord, [créez le cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md) que vous souhaitez utiliser comme cluster cible. Ici, nous allons utiliser la configuration de base. Si vous êtes intéressé par les différents paramètres, consultez le [didacticiel Créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).

Créez un réseau virtuel avec deux sous-réseaux vides pour le nœud principal et les nœuds Worker.

```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
```

Utilisez ensuite la commande suivante pour créer le cluster.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

## <a name="configure-the-target-openshift-4-cluster"></a>Configurer le cluster OpenShift 4 cible

### <a name="authentication"></a>Authentification

Pour que les utilisateurs puissent interagir avec Azure Red Hat OpenShift, ils doivent d’abord s’authentifier auprès du cluster. La couche d’authentification identifie l’utilisateur associé aux requêtes auprès de l’API Azure Red Hat OpenShift. La couche d’autorisation utilise ensuite les informations concernant l’utilisateur demandeur pour déterminer si la demande est autorisée.

Lorsqu’un cluster Azure Red Hat OpenShift 4 est créé, un utilisateur administratif temporaire est créé. [Connectez-vous à votre cluster](tutorial-connect-cluster.md), ajoutez des utilisateurs et des groupes et [configurez les autorisations appropriées](https://docs.openshift.com/aro/4/authentication/understanding-authentication.html) pour les deux.

### <a name="networking"></a>Mise en réseau

Azure Red Hat OpenShift 4 utilise différents opérateurs pour configurer le réseau dans votre cluster : [opérateur de réseau du cluster](https://docs.openshift.com/aro/4/networking/cluster-network-operator.html#nw-cluster-network-operator_cluster-network-operator), [opérateur DNS](https://docs.openshift.com/aro/4/networking/dns-operator.html) et [opérateur d’entrée](https://docs.openshift.com/aro/4/networking/ingress-operator.html). Pour plus d’informations sur la configuration de la mise en réseau dans un cluster Azure Red Hat OpenShift 4, consultez le [schéma de mise en réseau](concepts-networking.md) et la page [Understanding Networking](https://docs.openshift.com/aro/4/networking/understanding-networking.html) (Comprendre la mise en réseau).

### <a name="storage"></a>Stockage
Azure Red Hat OpenShift 4 prend en charge les plug-ins PersistentVolume suivants :

- AWS Elastic Block Store (EBS)
- Azure Disk
- Fichier Azure
- GCE Persistent Disk
- HostPath
- iSCSI
- Local volume
- NFS
- Red Hat OpenShift Container Storage

Pour plus d’informations sur la configuration de ces types de stockage, consultez la page [Configuring persistent storage](https://access.redhat.com/documentation/azure_red_hat_openshift/4/html/storage/configuring-persistent-storage) (Configuration du stockage persistant).

### <a name="registry"></a>Registre

Azure Red Hat OpenShift 4 peut générer des images à partir de votre code source, les déployer et gérer leur cycle de vie. Pour ce faire, Azure Red Hat OpenShift 4 fournit un [registre d’images conteneur intégré interne](https://docs.openshift.com/aro/4/registry/registry-options.html) qui peut être déployé dans votre environnement Azure Red Hat OpenShift pour gérer localement les images.

Si vous utilisez des registres externes tels qu’[Azure Container Registry](../container-registry/index.yml), des [registres Red Hat Quay](https://docs.openshift.com/aro/4/registry/registry-options.html#registry-quay-overview_registry-options) ou un [registre Red Hat avec authentification](https://docs.openshift.com/aro/4/registry/registry-options.html#registry-authentication-enabled-registry-overview_registry-options), suivez les étapes pour fournir des informations d’identification au cluster afin de lui permettre d’accéder aux référentiels.

### <a name="monitoring"></a>Surveillance

Azure Red Hat OpenShift comprend une pile d’analyse préconfigurée, préinstallée et à mise à jour automatique qui est basée sur le projet open source Prometheus et son écosystème plus étendu. Il permet d’analyser les composants de cluster et inclut un ensemble d’alertes permettant de notifier immédiatement l’administrateur du cluster de tout problème survenu, ainsi qu’un ensemble de tableaux de bord Grafana. La pile d’analyse de cluster est uniquement prise en charge pour l’analyse des clusters Azure Red Hat OpenShift. Pour plus d’informations, consultez [Cluster monitoring for Azure Red Hat OpenShift](https://docs.openshift.com/aro/4/monitoring/cluster_monitoring/about-cluster-monitoring.html) (Analyse des clusters pour Azure Red Hat OpenShift).

Si vous avez utilisé [Azure Monitor pour conteneurs pour Azure Red Hat OpenShift 3.11](../azure-monitor/insights/container-insights-azure-redhat-setup.md), vous pouvez également activer Azure Monitor pour conteneurs pour les [clusters Azure Red Hat OpenShift 4](../azure-monitor/insights/container-insights-azure-redhat4-setup.md) et continuer à utiliser le même espace de travail Log Analytics.

## <a name="move-your-dns-or-load-balancer-configuration-to-the-new-cluster"></a>Déplacer votre configuration DNS ou d’équilibrage de charge vers le nouveau cluster

Si vous utilisez Azure Traffic Manager, ajoutez des points de terminaison pour faire référence à votre cluster cible et classez ces points de terminaison par ordre de priorité.

## <a name="deploy-application-to-your-target-cluster"></a>Déployer l’application sur votre cluster cible

Une fois votre cluster cible correctement configuré pour votre charge de travail, [connectez-vous à votre cluster](tutorial-connect-cluster.md) et créez les applications, composants ou services nécessaires pour vos projets. Azure Red Hat OpenShift vous permet de les créer à partir de Git, d’images conteneur, du Catalogue des développeurs Red Hat, d’un fichier Dockerfile, d’une définition YAML/JSON ou en sélectionnant un service de base de données dans le Catalogue.

## <a name="delete-your-source-cluster"></a>Supprimer votre cluster source
Une fois que vous avez confirmé que votre cluster Azure Red Hat OpenShift 4 est correctement configuré, supprimez votre cluster Azure Red Hat OpenShift 3.11.

```
az openshift delete --name $CLUSTER_NAME
                    --resource-group $RESOURCE_GROUP
                    [--no-wait]
                    [--subscription]
                    [--yes]
```
## <a name="next-steps"></a>Étapes suivantes
Consultez la documentation Azure Red Hat OpenShift fournie par Red Hat [ici](https://docs.openshift.com/aro/4/welcome/index.html).