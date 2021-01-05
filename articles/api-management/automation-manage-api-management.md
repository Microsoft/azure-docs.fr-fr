---
title: Gestion des API Azure avec Azure Automation
description: Découvrez comment utiliser le service Azure Automation pour la gestion des API Azure.
services: api-management, automation
documentationcenter: ''
author: vladvino
manager: eamono
editor: ''
ms.assetid: 2e53c9af-f738-47f8-b1b6-593050a7c51b
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 02/13/2018
ms.author: apimpm
ms.openlocfilehash: c808d4659b5987b099dd96d73bb8c18c08fe3c99
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86249393"
---
# <a name="managing-azure-api-management-using-azure-automation"></a>Gestion des API Azure avec Azure Automation
Ce guide vous présente le service Azure Automation et décrit comment l’utiliser pour simplifier la gestion des API Azure.

## <a name="what-is-azure-automation"></a>Qu'est-ce qu'Azure Automation ?
[Azure Automation](https://azure.microsoft.com/services/automation/) est un service Azure destiné à simplifier la gestion du cloud via l'automatisation des processus. Il automatise les tâches manuelles, répétitives, fastidieuses et sources d’erreurs pour accroître la fiabilité, l’efficacité et le retour sur investissement de votre organisation.

Azure Automation fournit un moteur d’exécution de workflows à haute fiabilité et à haute disponibilité, qui s’adapte à vos besoins. Dans Azure Automation, les processus peuvent être lancés manuellement, par des systèmes tiers ou à des intervalles planifiés, afin que les tâches interviennent exactement au moment opportun.

Réduisez les coûts opérationnels et libérez du temps pour que votre équipe d’informaticiens et de développeurs puisse se concentrer sur des tâches qui génèrent une valeur ajoutée pour l’entreprise, en déléguant l’exécution automatique de vos tâches de gestion de Cloud à Azure Automation.

## <a name="how-can-azure-automation-help-manage-azure-api-management"></a>Comment Azure Automation peut-il aider à gérer les API Azure ?
La gestion des API peut être effectuée dans Azure Automation à l'aide de l’ [API de gestion des applets de commande Windows PowerShell pour l’API Azure](/powershell/module/az.apimanagement). Azure Automation vous permet d’écrire des scripts de workflow PowerShell pour effectuer la plupart de vos tâches de gestion des API à l’aide des applets de commande. Dans Azure Automation, vous pouvez également associer ces applets de commande à des applets de commande d'autres services Azure, afin d'automatiser des tâches complexes entre des services Azure et des systèmes tiers.

Voici quelques exemples de gestion d’API avec Powershell :

* [Exemples Azure PowerShell pour Gestion des API](./powershell-samples.md)

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous connaissez les bases d’Azure Automation et que vous savez l’utiliser pour la gestion des API Azure, cliquez sur les liens ci-dessous pour en savoir plus.

* Consultez le [Didacticiel de prise en main](../automation/learn/automation-tutorial-runbook-graphical.md)d’Azure Automation.
