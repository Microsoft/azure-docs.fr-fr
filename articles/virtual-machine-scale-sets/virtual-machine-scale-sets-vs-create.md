---
title: Déployer un jeu de mise à l’échelle de machines virtuelles à l’aide de Visual Studio
description: Déployer des jeux de mise à l'échelle de machines virtuelles à l'aide de Visual Studio et d’un modèle Resource Manager
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: visual-studio
ms.date: 09/09/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 2db281f1b3278e37deca8486971a7f7a83f85ccf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86503079"
---
# <a name="how-to-create-a-virtual-machine-scale-set-with-visual-studio"></a>Création d’un jeu de mise à l’échelle de machines virtuelles avec Visual Studio

Cet article explique comment déployer un groupe de machines virtuelles identiques Azure à l’aide d’un déploiement de groupe de ressources Visual Studio.

Les [groupes de machines virtuelles identiques Azure](https://azure.microsoft.com/blog/azure-vm-scale-sets-public-preview/) constituent une ressource de calcul Azure utile pour déployer et gérer une collection de machines virtuelles similaires avec mise à l’échelle automatique et équilibrage de charge. Vous pouvez configurer et déployer des groupes de machines virtuelles identiques à l’aide de [modèles Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates). Les modèles Azure Resource Manager peuvent être déployés à l’aide d’Azure CLI, de PowerShell et de REST, ainsi que directement à partir de Visual Studio. Visual Studio fournit une série d’exemples de modèles que vous pouvez déployer dans le cadre d’un projet de déploiement de groupe de ressources Azure.

Les déploiements de groupe de ressources Azure vous permettent de regrouper et de publier un ensemble de ressources Azure connexes dans une même opération de déploiement. Pour plus d’informations, voir [Création et déploiement de groupes de ressources Azure à l’aide de Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer le déploiement de groupes de machines virtuelles identiques dans Visual Studio, les conditions préalables suivantes doivent être réunies :

* Visual Studio 2013 ou une version ultérieure
* Kit de développement logiciel (SDK) Azure 2.7, 2.8 ou 2.9

>[!NOTE]
>Cet article utilise Visual Studio 2019 avec le [Kit de développement logiciel (SDK) Azure 2.8](https://azure.microsoft.com/blog/announcing-the-azure-sdk-2-8-for-net/).

## <a name="create-a-project"></a>Créer un projet <a name="creating-a-project"></a> 

1. Ouvrez Visual Studio et sélectionnez **Créer un projet**.

1. Dans **Créer un projet**, choisissez **groupe de ressources Azure** pour C#, puis sélectionnez **Suivant**.

1. Dans **Configurer votre nouveau projet**, entrez un nom, puis sélectionnez **Créer**.

    ![Nommer et créer votre projet](media/virtual-machine-scale-sets-vs-create/configure-azure-resource-group.png)

1. Dans la liste des modèles, choisissez le modèle **Groupe de machines virtuelles identiques Windows** ou **Groupe de machines virtuelles identiques Linux**. Sélectionnez **OK**.

   ![Sélectionner un modèle de machine virtuelle](media/virtual-machine-scale-sets-vs-create/select-vm-template.png)

Une fois votre projet créé, l’**Explorateur de solutions** contient un script de déploiement PowerShell, un modèle Azure Resource Manager et un fichier de paramètres pour le groupe de machines virtuelles identiques.

## <a name="customize-your-project"></a>Personnalisation de votre projet

Vous pouvez maintenant modifier le modèle pour le personnaliser en fonction des besoins de votre application. Vous pouvez ajouter des propriétés d’extension de machine virtuelle ou modifier les règles d’équilibrage de charge. Par défaut, les modèles de groupe de machines virtuelles identiques sont configurés pour déployer l’extension **AzureDiagnostics** qui permet d’ajouter très facilement des règles de mise à l’échelle automatique. Le modèle déploie également un équilibreur de charge avec une adresse IP publique, configurée avec des règles NAT de trafic entrant.

L’équilibreur de charge vous permet de vous connecter aux instances de machine virtuelle avec le protocole SSH (Linux) ou RDP (Windows). La plage de ports frontaux commence à 50000. Pour Linux, si vous utilisez SSH pour le port 50000, l’équilibrage de charge vous achemine vers le port 22 de la première machine virtuelle du groupe identique. La connexion au port 50001 est routée vers le port 22 de la deuxième machine virtuelle et ainsi de suite.

 Un bon moyen de modifier vos modèles avec Visual Studio consiste à utiliser la **Structure JSON**. Vous pouvez organiser les paramètres, les variables et les ressources. En comprenant le schéma, Visual Studio peut signaler d’éventuelles erreurs dans votre modèle avant le déploiement de celui-ci.

![JSON Explorer](media/virtual-machine-scale-sets-vs-create/json-explorer.png)

## <a name="deploy-the-project"></a>Déployer le projet

Déployez le modèle Azure Resource Manager pour créer la ressource du groupe de machines virtuelles identiques :

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Déployer** > **Nouveau**.

    ![Déployez votre projet](media/virtual-machine-scale-sets-vs-create/deploy-new-project.png)

1. Dans **Déployer vers le groupe de ressources**, choisissez l’abonnement à utiliser, puis sélectionnez un groupe de ressources. Si nécessaire, vous pouvez créer un groupe de ressources.

1. Ensuite, sélectionnez **Modifier les paramètres** pour entrer des paramètres transmis à votre modèle.

   ![Entrer un abonnement et un groupe de ressources](media/virtual-machine-scale-sets-vs-create/deploy-to-resource-group.png)

1. Entre le nom d’utilisateur et le mot de passe pour le système d’exploitation. Ces valeurs sont requises pour créer le déploiement. Si les outils PowerShell Tools for Visual Studio ne sont pas installés, sélectionnez **Enregistrer les mots de passe** afin d’éviter une invite de commandes PowerShell masquée, ou utilisez la [prise en charge du coffre de clés](https://azure.microsoft.com/blog/keyvault-support-for-arm-templates/). Cliquez sur **Save** (Enregistrer) pour continuer.

    ![Modifiez les paramètres de déploiement](media/virtual-machine-scale-sets-vs-create/edit-deployment-parameters.png)

1. Dans **Déployer vers le groupe de ressources**, sélectionnez **Déployer**. L’action exécute le script **Deploy-AzureResourceGroup.ps1**. La fenêtre **Sortie** affiche la progression du déploiement.

   ![La sortie affiche les résultats](media/virtual-machine-scale-sets-vs-create/deployment-output.png)

## <a name="explore-your-virtual-machine-scale-set"></a>Explorer de votre groupe de machines virtuelles identiques <a name="exploring-your-virtual-machine-scale-set"></a>

Sélectionnez **Afficher** > **Cloud Explorer** pour afficher le nouveau groupe de machines virtuelles identiques. Si nécessaire, utilisez **Actualiser tout**.

![Cloud Explorer](media/virtual-machine-scale-sets-vs-create/cloud-explorer.png)

**Cloud Explorer** vous permet de gérer des ressources Azure dans Visual Studio lors du développement d’applications. Vous pouvez également afficher votre groupe identique de machines virtuelles dans le [portail Azure](https://portal.azure.com) et [l’Explorateur de ressources Azure](https://resources.azure.com/).

 Le portail est le meilleur moyen de gérer votre infrastructure Azure à l’aide d’un navigateur web. Azure Resource Explorer est un moyen facile d’explorer et de déboguer des ressources Azure. Azure Resource Explorer offre la vue d’instance et affiche des commandes PowerShell pour les ressources que vous examinez.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez réussi à déployer Virtual Machine Scale Sets via Visual Studio, vous pouvez encore personnaliser votre projet pour répondre aux besoins de votre application. Par exemple, configurez la mise à l’échelle automatique en ajoutant une ressource **Insights**. Vous pouvez ajouter à votre modèle une infrastructure telle que des machines virtuelles autonomes, ou déployer des applications à l’aide de l’extension de script personnalisé. Vous trouverez de bons exemples de modèles dans le dépôt GitHub [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates) (Modèles de démarrage rapide Azure). Recherchez `vmss`.
