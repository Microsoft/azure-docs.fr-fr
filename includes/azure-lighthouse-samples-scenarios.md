---
title: Fichier include
description: Fichier include
services: lighthouse
author: JnHs
ms.service: lighthouse
ms.topic: include
ms.date: 04/24/2020
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: 4a6d9ef04a33c84d68dff1429fb39a193a249280
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82204539"
---
Ces exemples illustrent différentes tâches qui peuvent être effectuées dans des scénarios de gestion interlocataires.

| **Modèle** | **Description** |
|---------|---------|
| [create-keyvault-secret](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/create-keyvault-secret) | Crée un coffre de clés dans le locataire du client et crée des stratégies d’accès.
| [cross-rg-deployment](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/cross-rg-deployment) | Déploie des comptes de stockage dans deux différents groupes de ressources.|
| [deploy-azure-mgmt-services](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-mgmt-services) | Crée des services de gestion Azure, les relie et déploie des solutions supplémentaires. Pour un déploiement de bout en bout, utilisez le modèle **rgWithAzureMgmt.json**. |
| [deploy-azure-security-center](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-security-center) | Active et configure Azure Security Center dans l’abonnement Azure ciblé. |
| [deploy-azure-sentinel](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-sentinel) | Déploie et active Azure Sentinel sur un espace de travail Log Analytics existant dans un abonnement délégué. |
| [deploy-log-analytics-vm-extensions](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-log-analytics-vm-extensions) | Ces modèles vous permettent de déployer des extensions de machine virtuelle Log Analytics sur vos machines virtuelles Windows et Linux, en les connectant à l’espace de travail Log Analytics désigné. |
