---
title: 'ML Studio (classique) : Modules PowerShell - Azure'
description: Utilisez PowerShell pour créer et gérer des espaces de travail Azure Machine Learning Studio (classique), des expériences, des services web, etc.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: likebupt
ms.author: keli19
ms.date: 04/25/2019
ms.openlocfilehash: f66363ec8b64dea1a076f81f4fc89bafe5ca4151
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322323"
---
# <a name="powershell-modules-for-azure-machine-learning-studio-classic"></a>Modules PowerShell pour Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Grâce aux modules PowerShell, vous pouvez gérer par programmation vos ressources Studio (classique) et des ressources telles que les espaces de travail, les jeux de données et les services web.

Vous pouvez interagir avec les ressources Studio (classiques) à l’aide de trois modules PowerShell :

* [Azure PowerShell Az](#az-rm), publié en 2018, inclut toutes les fonctionnalités d’AzureRM, mais avec des noms cmdlets différents
* [AzureRM](#az-rm) publiée en 2016, remplacé par PowerShell Az
* [Azure Machine Learning PowerShell classique](#classic) publié en 2016

Même si ces modules PowerShell ont des similitudes, chacun d’eux est conçu pour certains scénarios particuliers. Cet article décrit les différences entre les modules PowerShell et vous aide à décider lequel choisir.  

Consultez le [tableau de prise en charge](#support-table) ci-dessous pour connaître les ressources prises en charge par chaque module. 

## <a name="azure-powershell-az-and-azurerm"></a><a name="az-rm"></a> Azure PowerShell Az et AzureRM

Az est maintenant le module PowerShell prévu pour interagir avec Azure et inclut toutes les fonctionnalités précédentes d’AzureRM. AzureRM continue de bénéficier de correctifs de bogues, mais ne recevra aucune nouvelle cmdlet ni fonctionnalité.  Az et AzureRM gèrent les solutions déployées à l’aide du modèle de déploiement **Azure Resource Manager**. Ces ressources incluent des espaces de travail Studio (classique) et de « nouveaux » services web Studio (classique). 

PowerShell Classic peut être installé en même temps que Az ou AzureRM pour couvrir les deux types de ressources « nouveau » et « classique ». Il n’est toutefois pas recommandé d’installer Az et AzureRM en même temps. Pour choisir entre Az et AzureRM, Microsoft recommande Az pour tous les déploiements futurs.  Apprenez-en davantage sur Az / AzureRM et le chemin de migration dans la [présentation d’Azure PowerShell Az](/powershell/azure/new-azureps-module-az).

Pour vous familiariser avec Az, suivez les [instructions d’installation pour Azure Az](/powershell/azure/install-az-ps).

## <a name="powershell-classic"></a><a name="classic"></a> PowerShell classique

Le [module PowerShell](https://aka.ms/amlps) Studio (classique) vous permet de gérer les ressources déployées à l’aide du **modèle de déploiement classique**. Ces ressources incluent des ressources utilisateur Studio (classique), des services web « classiques » et des points de terminaison de service web « classique ».

Microsoft vous recommande toutefois d’utiliser le modèle de déploiement Resource Manager pour les futures ressources afin de simplifier le déploiement et la gestion des ressources. Si vous souhaitez en savoir plus sur les modèles de déploiement, consultez l’article [Déploiement Azure Resource Manager et déploiement Classic](../../azure-resource-manager/management/deployment-models.md).

Pour bien démarrer avec PowerShell classique, téléchargez le [package de la version](https://github.com/hning86/azuremlps/releases) dans GitHub et suivez les [instructions d’installation](https://github.com/hning86/azuremlps/blob/master/README.md). Les instructions indiquent comment débloquer la DLL téléchargée/décompressée et l’importer dans votre environnement PowerShell.

PowerShell Classic peut être installé en même temps que Az ou AzureRM pour couvrir les deux types de ressources « nouveau » et « classique ».

## <a name="powershell-support-table"></a><a name="support-table"></a> Tableau de prise en charge PowerShell


| Tâche | **Az** |  **PowerShell Classic** |
| --- | --- | --- |
| Créer/supprimer des espaces de travail | [Modèles Microsoft Azure Resource Manager](./deploy-with-resource-manager-template.md) |  |
| Gérer les plans d’engagement d’espace de travail | [New-AzMlCommitmentPlan](/powershell/module/az.machinelearning/new-azmlcommitmentplan) | |
| Gérer les utilisateurs de l’espace de travail |  | [Add-AmlWorkspaceUsers](https://github.com/hning86/azuremlps#add-amlworkspaceusers)|
| Gérer des services web | [New-AzMlWebService](/powershell/module/az.machinelearning/new-azmlwebservice) <br>(« nouveaux » services web)| [New-AmlWebService](https://github.com/hning86/azuremlps#manage-classic-web-service) <br>(services web « classiques ») |
| Gérez les clés/points de terminaison des services web |  [Get-AzMlWebServiceKey](/powershell/module/az.machinelearning/get-azmlwebservicekey)|  [Add-AmlWebServiceEndpoint](https://github.com/hning86/azuremlps#manage-classic-web-servcie-endpoint)|
| Gérer les jeux de données/modèles formés d’un utilisateur| | [Get-AmlDataset](https://github.com/hning86/azuremlps#manage-user-assets-dataset-trained-model-transform) |
| Gérer les expériences utilisateur |  | [Start-AmlExperiment](https://github.com/hning86/azuremlps#manage-experiment) |
| Gérer les modules personnalisés | | [New-AmlCustomModule](https://github.com/hning86/azuremlps#manage-custom-module) |


## <a name="next-steps"></a>Étapes suivantes
Consultez la documentation complète sur ces module PowerShell :
* [PowerShell Classic](https://aka.ms/amlps)
* [Azure PowerShell Az](/powershell/module/az.machinelearning/#machine_learning)