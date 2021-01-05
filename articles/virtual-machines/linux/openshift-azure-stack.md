---
title: Déployer OpenShift dans Azure Stack
description: Déployez OpenShift dans Azure Stack.
author: haroldwongms
manager: joraio
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: 3729dbbc54d15d4641f5e514582f1437e790ef19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87374035"
---
# <a name="deploy-openshift-container-platform-or-okd-in-azure-stack"></a>Déployer OpenShift Container Platform ou OKD dans Azure Stack

OpenShift peut être déployé dans Azure Stack. En raison de quelques différences importantes entre Azure et Azure Stack, le déploiement et certaines fonctionnalités diffèrent légèrement.

Le fournisseur de cloud Azure ne fonctionne pas dans Azure Stack. Vous ne pouvez donc pas utiliser l’attachement de disque pour le stockage persistant dans Azure Stack. À la place, vous pouvez configurer d’autres options de stockage, telles que NFS, iSCSI, GlusterFS, etc. Une alternative est d’activer CNS et d’utiliser GlusterFS pour le stockage persistant. Si CNS est activé, trois nœuds supplémentaires sont déployés avec un stockage supplémentaire dédié à GlusterFS.

Plusieurs méthodes sont possibles pour déployer OpenShift Container Platform ou OKD dans Azure Stack :

- Vous pouvez déployer manuellement les composants d’infrastructure Azure nécessaires, puis suivre la [documentation OpenShift Container Platform](https://docs.openshift.com/container-platform) ou la [documentation OKD](https://docs.okd.io).
- Vous pouvez également utiliser un [modèle Resource Manager](https://github.com/Microsoft/openshift-container-platform/) existant qui simplifie le déploiement du cluster OpenShift Container Platform.
- Vous pouvez également utiliser un [modèle Resource Manager](https://github.com/Microsoft/openshift-origin) existant qui simplifie le déploiement du cluster OKD.

Si vous utilisez le modèle Resource Manager, sélectionnez la branche appropriée (azurestack-version-3.x). Les modèles pour Azure ne fonctionnent pas, car les versions d’API diffèrent entre Azure et Azure Stack. La référence d’image RHEL est actuellement codée en dur en tant que variable dans le fichier azuredeploy.json et doit être changée pour correspondre à votre image.

```json
"imageReference": {
    "publisher": "Redhat",
    "offer": "RHEL-OCP",
    "sku": "7-4",
    "version": "latest"
}
```

Pour toutes les options, vous devez avoir un abonnement Red Hat. Pendant le déploiement, l’instance Red Hat Enterprise Linux est inscrite dans l’abonnement Red Hat et associée à l’ID du pool contenant les droits pour OpenShift Container Platform.
Vérifiez que vous avez un nom d’utilisateur, un mot de passe et un ID de pool RHSM (Red Hat Subscription Manager) valides. Vous pouvez aussi utiliser une clé d’activation, un ID d’organisation et un ID de pool.  Vous pouvez vérifier ces informations en vous connectant au site https://access.redhat.com.

## <a name="azure-stack-prerequisites"></a>Prérequis pour Azure Stack

Une image RHEL (OpenShift Container Platform) ou une image CentOS (OKD) doit être ajoutée à votre environnement Azure Stack en vue du déploiement d’un cluster OpenShift. Contactez votre administrateur Azure Stack afin d’ajouter ces images. Les instructions sont disponibles ici :

- https://docs.microsoft.com/azure/azure-stack/azure-stack-add-vm-image
- https://docs.microsoft.com/azure/azure-stack/azure-stack-marketplace-azure-items
- https://docs.microsoft.com/azure/azure-stack/azure-stack-redhat-create-upload-vhd

## <a name="deploy-by-using-the-openshift-container-platform-or-okd-resource-manager-template"></a>Déployer à l’aide du modèle Resource Manager pour OpenShift Container Platform ou OKD

Pour déployer à l’aide du modèle Resource Manager, vous utilisez un fichier de paramètres pour fournir les paramètres d’entrée. Pour personnaliser davantage le déploiement, dupliquez (fork) le dépôt GitHub et changez les éléments appropriés.

Voici une liste non exhaustive d’options de personnalisation courantes :

- Taille de machine virtuelle bastion (variable dans azuredeploy.json)
- Conventions de nommage (variables dans azuredeploy.json)
- Caractéristiques de cluster OpenShift, modifiées dans le fichier hosts (deployOpenShift.sh)
- Référence d’image RHEL (variable dans azuredeploy.json)

Pour connaître les étapes du déploiement avec Azure CLI, consultez la section correspondante dans la section [OpenShift Container Platform](./openshift-container-platform-3x.md) ou la section [OKD](./openshift-okd.md).

## <a name="next-steps"></a>Étapes suivantes

- [Tâches de post-déploiement](./openshift-container-platform-3x-post-deployment.md)
- [Résoudre les problèmes de déploiement d’OpenShift dans Azure](./openshift-container-platform-3x-troubleshooting.md)