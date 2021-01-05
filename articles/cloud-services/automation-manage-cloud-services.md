---
title: Gestion d’Azure Cloud Services à l’aide d’Azure Automation | Microsoft Docs
description: Découvrez comment le service Azure Automation peut être utilisé pour gérer les services de cloud computing Azure à grande échelle.
services: cloud-services, automation
author: jodoglevy
manager: timlt
editor: ''
ms.assetid: 3789810a-2892-4eef-bf29-c781c1b5af48
ms.service: cloud-services
ms.topic: article
ms.date: 06/20/2016
ms.author: timlt
ms.openlocfilehash: 67830f8c00d9f74f62883e0714ffe1c2bbbd6903
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075618"
---
# <a name="managing-azure-cloud-services-using-azure-automation"></a>Gestion d'Azure Cloud Services à l'aide d'Azure Automation
Ce guide vous présente le service Azure Automation et la manière de l'utiliser pour gérer plus simplement vos services de cloud computing Azure.

## <a name="what-is-azure-automation"></a>Qu'est-ce qu'Azure Automation ?
[Azure Automation](https://azure.microsoft.com/services/automation/) est un service Azure destiné à simplifier la gestion du cloud via l'automatisation des processus. Azure Automation permet d'automatiser les tâches fastidieuses, manuelles, propices aux erreurs et répétitives, afin d'augmenter la fiabilité, l'efficacité et le retour sur investissement pour votre organisation.

Azure Automation fournit un moteur d'exécution de workflow hautement fiable et hautement disponible, qui s'adapte à vos besoins au fur et à mesure que votre entreprise se développe. Dans Azure Automation, les processus peuvent être lancés manuellement, par des systèmes tiers ou à des intervalles planifiés, afin que les tâches interviennent exactement au moment opportun.

Réduisez les coûts opérationnels et dégagez du temps pour votre personnel informatique/DevOps afin de lui permettre de se concentrer sur des tâches générant une valeur ajoutée pour l'entreprise, en configurant Azure Automation pour exécuter automatiquement vos tâches de gestion de cloud.

## <a name="how-can-azure-automation-help-manage-azure-cloud-services"></a>Comment Azure Automation peut-il aider à gérer des services de cloud computing Azure ?
Les services cloud Azure peuvent être gérés dans Azure Automation à l'aide des applets de commande PowerShell qui sont disponibles dans les [outils Azure PowerShell](/powershell/). Azure Automation dispose dès le départ de ces applets de commande PowerShell de services de cloud computing, de sorte que vous pouvez effectuer toutes vos tâches de gestion de services de cloud computing au sein du service. Dans Azure Automation, vous pouvez également associer ces applets de commande à des applets de commande d'autres services Azure, afin d'automatiser des tâches complexes entre des services Azure et des systèmes tiers.

Voici quelques exemples qui utilisent Azure Automation pour gérer Azure Cloud Services :

* [Déploiement continu d’un service cloud si cscfg ou cspkg est mis à jour dans Azure Blob Storage](https://gallery.technet.microsoft.com/scriptcenter/Continuous-Deployment-of-A-eeebf3a6)
* [Redémarrage des instances de service cloud en parallèle, un domaine de mise à niveau à la fois](https://gallery.technet.microsoft.com/scriptcenter/Reboot-Cloud-Service-PaaS-b337a06d)

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les bases d'Azure Automation et la manière de l'utiliser pour gérer les services de cloud computing Azure, cliquez sur ces liens pour en savoir plus sur Azure Automation.

* [Vue d’ensemble de Microsoft Azure Automation](../automation/automation-intro.md)
* [Mon premier Runbook](../automation/learn/automation-tutorial-runbook-graphical.md)