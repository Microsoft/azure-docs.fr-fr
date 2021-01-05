---
title: Créer une sauvegarde d’application de cluster Azure Red Hat OpenShift 4 à l’aide de Velero
description: Découvrez comment créer une sauvegarde de vos applications de cluster Azure Red Hat OpenShift à l’aide de Velero.
ms.service: container-service
ms.topic: article
ms.date: 06/22/2020
author: troy0820
ms.author: b-trconn
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 264778d2d6d1ee0119ad8622043b7cd3a1088ec1
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280139"
---
# <a name="create-an-azure-red-hat-openshift-4-cluster-application-backup"></a>Créer une sauvegarde d’application de cluster Azure Red Hat OpenShift 4

Dans cet article, vous allez préparer votre environnement pour créer une sauvegarde d’application de cluster Azure Red Hat OpenShift 4. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer les composants requis et installer les outils nécessaires
> * Créer une sauvegarde d’application Azure Red Hat OpenShift 4

> [!NOTE] 
> Velero ne sauvegarde pas les données de stockage clé-valeur etcd d’Azure Red Hat OpenShift. Si vous avez besoin de sauvegarder etcd, consultez [Sauvegarde de etcd](https://docs.openshift.com/container-platform/4.5/backup_and_restore/backing-up-etcd.html).

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Avant de commencer

### <a name="install-velero"></a>Installer Velero

Pour [installer](https://velero.io/docs/main/basic-install/) Velero sur votre système, suivez le processus recommandé pour votre système d’exploitation.

### <a name="set-up-azure-storage-account-and-blob-container"></a>Configurer un compte de stockage Azure et un conteneur de blobs

Cette étape permet de créer un groupe de ressources en dehors du groupe de ressources du cluster ARO.  Ce groupe de ressources permet de conserver les sauvegardes et peut restaurer les applications sur de nouveaux clusters.

```bash
AZURE_BACKUP_RESOURCE_GROUP=Velero_Backups
az group create -n $AZURE_BACKUP_RESOURCE_GROUP --location eastus

AZURE_STORAGE_ACCOUNT_ID="velero$(uuidgen | cut -d '-' -f5 | tr '[A-Z]' '[a-z]')"
az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_ID \
    --resource-group $AZURE_BACKUP_RESOURCE_GROUP \
    --sku Standard_GRS \
    --encryption-services blob \
    --https-only true \
    --kind BlobStorage \
    --access-tier Hot

BLOB_CONTAINER=velero
az storage container create -n $BLOB_CONTAINER --public-access off --account-name $AZURE_STORAGE_ACCOUNT_ID
```

## <a name="set-permissions-for-velero"></a>Définir des autorisations pour Velero

### <a name="create-service-principal"></a>Créer un principal du service

Velero a besoin d’autorisations pour effectuer des sauvegardes et des restaurations. Lorsque vous créez un principal de service, vous accordez à Velero l’autorisation d’accéder au groupe de ressources que vous avez défini à l’étape précédente. Cette étape permet d’obtenir le groupe de ressources du cluster :

```bash
export AZURE_RESOURCE_GROUP=$(az aro show --name <name of cluster> --resource-group <name of resource group> | jq -r .clusterProfile.resourceGroupId | cut -d '/' -f 5,5)
```


```bash
AZURE_SUBSCRIPTION_ID=$(az account list --query '[?isDefault].id' -o tsv)

AZURE_TENANT_ID=$(az account list --query '[?isDefault].tenantId' -o tsv)
```

```bash
AZURE_CLIENT_SECRET=$(az ad sp create-for-rbac --name "velero" --role "Contributor" --query 'password' -o tsv \
--scopes  /subscriptions/$AZURE_SUBSCRIPTION_ID)
AZURE_CLIENT_ID=$(az ad sp list --display-name "velero" --query '[0].appId' -o tsv)

```

```bash
cat << EOF  > ./credentials-velero.yaml
AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
AZURE_TENANT_ID=${AZURE_TENANT_ID}
AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET}
AZURE_RESOURCE_GROUP=${AZURE_RESOURCE_GROUP}
AZURE_CLOUD_NAME=AzurePublicCloud
EOF
```

## <a name="install-velero-on-azure-red-hat-openshift-4-cluster"></a>Installer Velero sur un cluster Azure Red Hat OpenShift 4

Cette étape installe Velero dans son propre projet et les [définitions de ressource personnalisées](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) nécessaires pour effectuer des sauvegardes et des restaurations avec Velero. Vérifiez que vous êtes correctement connecté à un cluster Azure Red Hat OpenShift v4.


```bash
velero install \
--provider azure \
--plugins velero/velero-plugin-for-microsoft-azure:v1.1.0 \
--bucket $BLOB_CONTAINER \
--secret-file ~/path/to/credentials-velero.yaml \
--backup-location-config resourceGroup=$AZURE_BACKUP_RESOURCE_GROUP,storageAccount=$AZURE_STORAGE_ACCOUNT_ID \
--snapshot-location-config apiTimeout=15m \
--velero-pod-cpu-limit="0" --velero-pod-mem-limit="0" \
--velero-pod-mem-request="0" --velero-pod-cpu-request="0"
```

## <a name="create-a-backup-with-velero"></a>Créer une sauvegarde avec Velero

Pour créer une sauvegarde d’application avec Velero, vous devez inclure l’espace de noms dans lequel se trouve cette application.  Si vous disposez d’un espace de noms `nginx-example` et souhaitez inclure toutes les ressources de cet espace de noms dans la sauvegarde, exécutez la commande suivante dans le terminal :

```bash
velero create backup <name of backup> --include-namespaces=nginx-example
```
Vous pouvez vérifier l’état de la sauvegarde en exécutant :

```bash
oc get backups -n velero <name of backup> -o yaml
```

Une sauvegarde réussie génère `phase:Completed` et les objets se trouvent dans le conteneur du compte de stockage.

## <a name="create-a-backup-with-velero-to-include-snapshots"></a>Créer une sauvegarde avec Velero pour inclure des captures instantanées

Pour créer une sauvegarde d’application avec Velero afin d’inclure les volumes persistants de votre application, vous devez inclure l’espace de noms dans lequel se trouve l’application ainsi que l’indicateur `snapshot-volumes=true` lors de la création de la sauvegarde.

```bash
velero backup create <name of backup> --include-namespaces=nginx-example --snapshot-volumes=true --include-cluster-resources=true
```

Vous pouvez vérifier l’état de la sauvegarde en exécutant :

```bash
oc get backups -n velero <name of backup> -o yaml
```

Une sauvegarde réussie génère `phase:Completed` et les objets se trouvent dans le conteneur du compte de stockage.

Pour plus d’informations sur la création de sauvegardes et de restaurations à l’aide de Velero, consultez [Sauvegarde native des ressources OpenShift](https://www.openshift.com/blog/backup-openshift-resources-the-native-way).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, une application de cluster Azure Red Hat OpenShift 4 a été sauvegardée. Vous avez appris à :

> [!div class="checklist"]
> * Créer une sauvegarde d’application de cluster OpenShift v4 à l’aide de Velero
> * Créer une sauvegarde d’application de cluster OpenShift v4 avec des captures instantanées à l’aide de Velero


Passez à l’article suivant pour apprendre à créer une restauration d’application de cluster Azure Red Hat OpenShift 4.

* [Créer une restauration d’application de cluster Azure Red Hat OpenShift 4](howto-create-a-restore.md)
* [Créer une restauration d’application de cluster Azure Red Hat OpenShift 4 avec des captures instantanées](howto-create-a-restore.md)
