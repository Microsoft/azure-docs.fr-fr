---
title: Déplacer une application Service Fabric Mesh vers une autre région
description: Vous pouvez déplacer des ressources Service Fabric Mesh en déployant une copie de votre modèle actuel vers une nouvelle région Azure.
author: erikadoyle
ms.author: edoyle
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: c842a065f108a924c6bffd70d6c2edbbd31b6dff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260158"
---
# <a name="move-a-service-fabric-mesh-application-to-another-azure-region"></a>Déplacer une application Service Fabric Mesh vers une autre région Azure

Cet article décrit comment déplacer votre application Service Fabric Mesh et ses ressources vers une autre région Azure. Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, en réponse à des pannes, pour obtenir des fonctionnalités ou des services disponibles dans des régions spécifiques uniquement, pour répondre à des exigences de stratégie et de gouvernance internes, ou en réponse à des exigences de planification de la capacité.

 [Service Fabric Mesh ne prend pas en charge](../azure-resource-manager/management/region-move-support.md#microsoftservicefabricmesh) la possibilité de déplacer directement des ressources dans des régions Azure. Pour autant, vous pouvez déplacer des ressources indirectement en déployant une copie de votre modèle Azure Resource Manager actuel vers la nouvelle région cible, puis en redirigeant le trafic d'entrée et les dépendances vers l'application Service Fabric Mesh nouvellement créée.

## <a name="prerequisites"></a>Prérequis

* Contrôleur d’entrée (par exemple, [Application Gateway](../application-gateway/index.yml)) faisant office d’intermédiaire pour acheminer le trafic entre les clients et votre application Service Fabric Mesh
* Disponibilité de Service Fabric Mesh (préversion) dans la région Azure cible (`westus`, `eastus` ou `westeurope`)

## <a name="prepare"></a>Préparation

1. Prenez un « instantané » de l’état actuel de votre application Service Fabric Mesh en exportant le modèle Azure Resource Manager et les paramètres du plus récent déploiement. Pour ce faire, suivez les étapes décrites dans [Exporter un modèle après déploiement](../azure-resource-manager/templates/export-template-portal.md#export-template-after-deployment) à l’aide du portail Azure. Vous pouvez également utiliser [Azure CLI](../azure-resource-manager/management/manage-resource-groups-cli.md#export-resource-groups-to-templates), [Azure PowerShell](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates) ou l'[API REST](/rest/api/resources/resourcegroups/exporttemplate).

2. Le cas échéant, [exportez d’autres ressources dans le même groupe de ressources](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource-group) à des fins de redéploiement dans la région cible.

3. Examinez (et modifiez, si nécessaire) le modèle exporté pour vous assurer que les valeurs de propriété existantes correspondent à celles que vous souhaitez utiliser dans la région cible. Le nouvel `location` (région Azure) correspond à un paramètre que vous fournirez lors du redéploiement.

## <a name="move"></a>Déplacer

1. Créez un groupe de ressources (ou utilisez un groupe existant) dans la région cible.

2. Avec votre modèle exporté, suivez les étapes décrites dans [Déployer des ressources à partir d’un modèle personnalisé](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template) à l’aide du portail Azure. Vous pouvez également utiliser [Azure CLI](../azure-resource-manager/templates/deploy-cli.md), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) ou l'[API REST](../azure-resource-manager/templates/deploy-rest.md).

3. Pour plus d'informations sur le déplacement de ressources connexes, telles que les [comptes Stockage Azure](../storage/common/storage-account-move.md), reportez-vous aux conseils correspondant aux services individuels répertoriés dans la rubrique [Déplacement des ressources Azure entre les régions](../azure-resource-manager/management/move-region.md).

## <a name="verify"></a>Vérifier

1. Une fois le déploiement terminé, testez le(s) point(s) de terminaison de l'application afin d'en vérifier la fonctionnalité.

2. Vous pouvez également vérifier l’état de votre application en utilisant [az mesh app show](/cli/azure/ext/mesh/mesh/app?view=azure-cli-latest#ext-mesh-az-mesh-app-show) et en examinant les journaux d'application et les commandes ([az mesh code-package-log](/cli/azure/ext/mesh/mesh/code-package-log?view=azure-cli-latest)) à l'aide de l'[interface de ligne de commande Azure Service Fabric Mesh](./service-fabric-mesh-quickstart-deploy-container.md#set-up-service-fabric-mesh-cli).

## <a name="commit"></a>Commit

Après avoir vérifié la fonctionnalité équivalente de votre application Service Fabric Mesh dans la région cible, configurez votre contrôleur d’entrée (par exemple, [Application Gateway](../application-gateway/redirect-overview.md)) pour rediriger le trafic vers la nouvelle application.

## <a name="clean-up-source-resources"></a>Nettoyer les ressources sources

Pour terminer le déplacement de l'application Service Fabric Mesh, [supprimez l’application source et/ou le groupe de ressources parent](../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Étapes suivantes

* [Déplacer des ressources Azure entre les régions](../azure-resource-manager/management/move-region.md)
* [Prise en charge du déplacement des ressources Azure entre les régions](../azure-resource-manager/management/region-move-support.md)
* [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Prise en charge de l’opération de déplacement pour les ressources](../azure-resource-manager/management/move-support-resources.md
)
