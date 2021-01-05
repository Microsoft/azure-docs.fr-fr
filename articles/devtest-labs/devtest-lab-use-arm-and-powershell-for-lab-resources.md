---
title: Créer ou modifier des laboratoires à l’aide de modèles de Azure Resource Manager
description: Découvrez comment utiliser des modèles Azure Resource Manager avec PowerShell pour créer ou modifier des laboratoires automatiquement dans un DevTest Lab
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 645f1d0717514d2c7e7b16844513327127e4e1a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87272638"
---
# <a name="create-or-modify-labs-automatically-using-azure-resource-manager-templates-and-powershell"></a>Créer ou modifier des laboratoires automatiquement à l’aide de modèles Azure Resource Manager et PowerShell

DevTest Labs fournit de nombreux modèles Azure Resource Manager et scripts PowerShell qui peuvent vous aider à créer rapidement et automatiquement de nouveaux laboratoires ou à modifier des laboratoires existants, puis à déployer ces ressources.

Cet article vous guide dans le processus d’utilisation de ces modèles et scripts pour automatiser la création, la modification et le déploiement de vos laboratoires. Il vous montre également où trouver des informations supplémentaires sur l’utilisation de PowerShell pour effectuer certaines tâches courantes dans DevTest Labs.

## <a name="step-1-gather-your-templates-and-scripts"></a>Étape 1 : rassembler vos modèles et scripts
Des [modèles Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates) et [scripts PowerShell](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/Scripts) prédéfinis sont à votre disposition dans notre [référentiel GitHub](https://github.com/Azure/azure-devtestlab) public. Utilisez-les tels quels ou adaptez-les à vos besoins et stockez-les dans votre propre [référentiel Git privé](devtest-lab-add-artifact-repo.md).

## <a name="step-2-modify-your-azure-resource-manager-template"></a>Étape 2 : modifier votre modèle Azure Resource Manager
Si vous n’avez jamais créé de modèle, reportez-vous à la procédure [Créer votre premier modèle Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

En outre, les [Bonnes pratiques de création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-best-practices.md) contiennent de nombreuses directives et suggestions pour vous aider à créer des modèles Azure Resource Manager fiables et faciles à utiliser. En règle générale, vous utilisez l’une des variantes des approches ou exemples fournis et modifiez votre modèle en fonction de vos besoins.

## <a name="step-3-deploy-resources-with-powershell"></a>Étape 3 : déployer des ressources avec PowerShell
Après avoir personnalisé vos modèle et scripts, suivez les étapes nécessaires pour [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md). Cet article fournit des informations générales sur l’utilisation d’Azure PowerShell avec des modèles Azure Resource Manager pour déployer vos ressources sur Azure.


## <a name="common-tasks-you-can-perform-in-devtest-labs-using-powershell"></a>Tâches courantes que vous pouvez effectuer dans DevTest Labs à l’aide de PowerShell
Il existe de nombreuses autres tâches courantes que vous pouvez automatiser à l’aide de PowerShell. Les sections suivantes de la documentation décrivent les étapes requises pour effectuer ces tâches.

* [Créer une image personnalisée à partir d’un fichier de disque dur virtuel à l’aide de PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)
* [Charger le fichier de disque dur virtuel sur le compte de stockage du laboratoire à l’aide de PowerShell](devtest-lab-upload-vhd-using-powershell.md)
* [Ajouter un utilisateur externe à un laboratoire à l’aide de PowerShell](devtest-lab-add-devtest-user.md#add-an-external-user-to-a-lab-using-powershell)
* [Créer un rôle personnalisé de laboratoire à l’aide de PowerShell](devtest-lab-grant-user-permissions-to-specific-lab-policies.md#creating-a-lab-custom-role-using-powershell)

### <a name="next-steps"></a>Étapes suivantes
* Découvrez comment créer un [référentiel Git privé](devtest-lab-add-artifact-repo.md) dans lequel stocker vos modèles ou scripts personnalisés.
* Explorez les [modèles Azure Resource Manager de la galerie de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates).
