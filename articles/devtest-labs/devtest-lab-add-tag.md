---
title: Ajouter des balises à un laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Apprenez à créer des balises personnalisées dans Azure DevTest Labs, et à utilisez des balises pour catégoriser les ressources. Vous pouvez voir toutes les ressources de votre abonnement qui ont une balise.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a4d2b328626f0d0f096082fa88bb170894ccf3e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88270731"
---
# <a name="add-tags-to-a-lab-in-azure-devtest-labs"></a>Ajouter des balises à un laboratoire dans Azure DevTest Labs

Vous pouvez créer des balises personnalisées et les appliquer à vos ressources DevTest Labs pour classer vos ressources logiquement. Vous pouvez ensuite rapidement et facilement voir toutes les ressources de votre abonnement avec cette balise. Les balises sont utiles si vous devez organiser les ressources à des fins de facturation ou de gestion.

Les ressources prises en charge par les balises comprennent les

* Machines virtuelles de calcul
* Cartes réseau
* Adresses IP
* Équilibreurs de charge
* Comptes de stockage
* Disques managés

Vous pouvez appliquer des balises quand vous souhaitez [Créer un laboratoire](devtest-lab-create-lab.md), puis les gérer via le panneau Balises dans Configuration et paramètres.

Chaque balise est composée d’une paire **nom**/**valeur**. Par exemple, vous pouvez créer une balise avec le nom *costcenter* et lui attribuer la valeur *34543*. Une telle balise peut à l’avenir vous aider à identifier les ressources de laboratoire facturables dans cette zone spécifique de votre organisation. Vous devez choisir les noms et les valeurs qui ont un sens en fonction de la manière dont vous souhaitez organiser votre abonnement.

## <a name="steps-to-manage-tags-in-an-existing-lab"></a>Étapes de gestion des balises dans un laboratoire existant

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Si nécessaire, sélectionnez **Autres services**, puis **DevTest Labs** dans la liste. Votre laboratoire peut déjà être affiché sur le Tableau de bord dans **Toutes les ressources**.
1. Dans la liste des laboratoires, sélectionnez le laboratoire dans lequel vous souhaitez ajouter ou gérer vos balises.
1. Dans la zone **Vue d’ensemble** du laboratoire, sélectionnez **Configuration et stratégies**.

    ![Bouton Configuration et stratégies](./media/devtest-lab-add-tag/devtestlab-config-and-policies.png)

1. Sur la gauche, dans **GÉRER**, sélectionnez **Balises**.
1. Pour créer une nouvelle étiquette pour ce laboratoire, entrez une paire **Nom**/**Valeur** et sélectionnez **Enregistrer**. Vous pouvez également sélectionner une balise existante dans la liste pour afficher ou gérer les ressources associées à cette balise.

    ![Gérer les balises](./media/devtest-lab-add-tag/devtestlab-manage-tags.png)

> [!NOTE]
> Les balises créées au niveau du labo sont transmises à toutes les ressources facturables que le labo prépare dans votre abonnement. Par exemple, les balises de niveau labo sont transmises aux machines virtuelles de calcul sous-jacentes des machines virtuelles du labo. Vous pouvez utiliser des balises dans le contexte de la gestion des coûts. Les balises de niveau labo apparaissent dans le filtre par balise pour la gestion des coûts.

## <a name="understanding-limitations-to-tags"></a>Présentation des limitations des balises

Les limites suivantes s’appliquent aux balises :

* Chaque ressource ou groupe de ressources peut inclure un maximum de 15 paires nom/valeur de balise. Cette limitation s’applique uniquement aux balises directement appliquées au groupe de ressources ou à la ressource. Un groupe de ressources peut contenir de nombreuses ressources qui ont chacune 15 paires nom/valeur de balise.
* Le nom de balise est limité à 512 caractères, et la valeur de balise à 256 caractères. Pour les comptes de stockage, le nom de balise est limité à 128 caractères, et la valeur de balise à 256 caractères.
* Les ressources d’un groupe de ressources n’héritent pas des balises appliquées à ce groupe de ressources.

[Utiliser des balises pour organiser vos ressources Azure](../azure-resource-manager/management/tag-resources.md) fournit des détails supplémentaires sur l’utilisation de balises dans Azure, y compris la gestion des balises à l’aide de PowerShell ou d’Azure CLI.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
* Vous pouvez appliquer des restrictions et des conventions sur votre abonnement avec des stratégies personnalisées. La stratégie que vous définissez peut exiger que toutes les ressources aient une valeur pour une balise en particulier. Pour plus d’informations, consultez [Définir des stratégies et la planification](devtest-lab-set-lab-policy.md).
* Explorez la [Galerie de modèles de démarrage rapide d’Azure Resource Manager DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
