---
title: Créer Azure Files StorageClass sur Azure Red Hat OpenShift 4
description: Découvrez comment créer Azure Files StorageClass sur Azure Red Hat OpenShift
ms.service: container-service
ms.topic: article
ms.date: 10/16/2020
author: grantomation
ms.author: b-grodel
keywords: aro, openshift, az aro, red hat, cli, azure file
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: fe80698b71ae0ba808991d79b423d49abfacdf7c
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825904"
---
# <a name="create-an-azure-files-storageclass-on-azure-red-hat-openshift-4"></a>Créer Azure Files StorageClass sur Azure Red Hat OpenShift 4

Dans cet article, vous allez créer un StorageClass pour Azure Red Hat OpenShift 4 qui approvisionne dynamiquement le stockage ReadWriteMany (RWX) à l’aide d’Azure Files. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer les composants requis et installer les outils nécessaires
> * Créer un StorageClass Azure Red Hat OpenShift 4 avec l’approvisionneur Azure Files

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Avant de commencer

Déployez un cluster Azure Red Hat OpenShift 4 dans votre abonnement, voir [Créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).


### <a name="set-up-azure-storage-account"></a>Configurer un compte de stockage Azure

Cette étape permet de créer un groupe de ressources en dehors du groupe de ressources du cluster Azure Red Hat OpenShift (ARO). Ce groupe de ressources contiendra les partages Azure Files créés par l’approvisionneur dynamique d’Azure Red Hat OpenShift.

```bash
AZURE_FILES_RESOURCE_GROUP=aro_azure_files
LOCATION=eastus

az group create -l $LOCATION -n $AZURE_FILES_RESOURCE_GROUP

AZURE_STORAGE_ACCOUNT_NAME=aroazurefilessa

az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_NAME \
    --resource-group $AZURE_FILES_RESOURCE_GROUP \
    --kind StorageV2 \
    --sku Standard_LRS
```

## <a name="set-permissions"></a>Définir des autorisations
### <a name="set-resource-group-permissions"></a>Définir les autorisations du groupe de ressources

Le principal du service ARO requiert l’autorisation « listKeys » sur le groupe de ressources du nouveau compte de stockage Azure. Attribuez le rôle « Contributeur » pour y parvenir.

```bash
ARO_RESOURCE_GROUP=aro-rg
CLUSTER=cluster
ARO_SERVICE_PRINCIPAL_ID=$(az aro show -g $ARO_RESOURCE_GROUP -n $CLUSTER --query servicePrincipalProfile.clientId -o tsv)

az role assignment create --role Contributor -–assignee $ARO_SERVICE_PRINCIPAL_ID -g $AZURE_FILES_RESOURCE_GROUP
```

### <a name="set-aro-cluster-permissions"></a>Définir les autorisations de cluster ARO

Le compte de service Binder de volume persistant OpenShift nécessitera la capacité à lire des secrets. Créez et attribuez un rôle de cluster OpenShift pour y parvenir.
```bash
ARO_API_SERVER=$(az aro list --query "[?contains(name,'$CLUSTER')].[apiserverProfile.url]" -o tsv)

oc login -u kubeadmin -p $(az aro list-credentials -g $ARO_RESOURCE_GROUP -n $CLUSTER --query=kubeadminPassword -o tsv) $APISERVER

oc create clusterrole azure-secret-reader \
    --verb=create,get \
    --resource=secrets

oc adm policy add-cluster-role-to-user azure-secret-reader system:serviceaccount:kube-system:persistent-volume-binder
```

## <a name="create-storageclass-with-azure-files-provisioner"></a>Créer StorageClass avec l’approvisionneur Azure Files

Cette étape permet de créer un StorageClass avec un approvisionneur Azure Files. Dans le manifeste StorageClass, les détails du compte de stockage sont requis afin que le cluster ARO sache qu’il doit examiner un compte de stockage en dehors du groupe de ressources actuel.

```bash
cat << EOF >> azure-storageclass-azure-file.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azure-file
provisioner: kubernetes.io/azure-file
parameters:
  location: $LOCATION
  skuName: Standard_LRS
  storageAccount: $AZURE_STORAGE_ACCOUNT_NAME
  resourceGroup: $AZURE_FILES_RESOURCE_GROUP
reclaimPolicy: Delete
volumeBindingMode: Immediate
EOF

oc create -f azure-storageclass-azure-file.yaml
```

## <a name="change-the-default-storageclass-optional"></a>Modifier le StorageClass par défaut (facultatif)

Le StorageClass par défaut sur ARO s’appelle managed-premium et utilise l’approvisionneur azure-disk. Modifiez ceci en émettant des commandes patch sur les manifestes StorageClass.

```bash
oc patch storageclass managed-premium -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

oc patch storageclass azure-file -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## <a name="verify-azure-file-storageclass-optional"></a>Vérifier Azure File StorageClass (facultatif)

Créez une nouvelle application et attribuez-lui le stockage.

```bash
oc new-project azfiletest
oc new-app –template httpd-example

#Wait for the pod to become Ready
curl $(oc get route httpd-example -n azfiletest -o jsonpath={.spec.host})

oc set volume dc/httpd-example --add --name=v1 -t pvc --claim-size=1G -m /data

#Wait for the new deployment to rollout
export POD=$(oc get pods --field-selector=status.phase==Running -o jsonpath={.items[].metadata.name})
oc exec $POD -- bash -c "mkdir ./data"
oc exec $POD -- bash -c "echo 'azure file storage' >> /data/test.txt"

oc exec $POD -- bash -c "cat /data/test.txt"
azure file storage
```
Le fichier .txt de test sera également visible via Explorateur Stockage dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un stockage persistant dynamique à l’aide de Microsoft Azure Files et d’Azure Red Hat OpenShift 4. Vous avez appris à :

> [!div class="checklist"]
> * Créer un compte de stockage
> * Configurer un StorageClass sur un cluster Azure Red Hat OpenShift 4 à l’aide de l’approvisionneur Azure Files

Passez à l’article suivant pour en savoir plus sur les ressources prises en charge par Azure Red Hat OpenShift 4.

* [Stratégie de prise en charge d’Azure Red Hat OpenShift](support-policies-v4.md)
