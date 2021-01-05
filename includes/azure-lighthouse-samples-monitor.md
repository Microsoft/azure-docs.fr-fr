---
title: Fichier include
description: Fichier include
services: lighthouse
author: JnHs
ms.service: lighthouse
ms.topic: include
ms.date: 12/11/2020
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: 0056e18b6cb3aad2a4504bbe20b3b3421793489e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356263"
---
Ces exemples montrent comment utiliser Azure Monitor pour créer des alertes avec des abonnements qui ont été intégrés pour la gestion des ressources déléguées Azure.

| **Modèle** | **Description** |
|---------|---------|
| [monitor-delegation-changes](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/monitor-delegation-changes) | Interroge le dernier jour d’activité dans un locataire gérant et [crée des rapports sur les délégations ajoutées ou supprimées](../articles/lighthouse/how-to/monitor-delegation-changes.md) (ou sur les tentatives ayant échoué).|
| [alert-using-actiongroup](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/alert-using-actiongroup) | Ce modèle crée une alerte Azure et se connecte à un groupe d’actions existant.|
| [multiple-loganalytics-alerts](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/multiple-loganalytics-alerts) | Crée plusieurs alertes de journal basées sur des requêtes Kusto.|
| [delegation-alert-for-customer](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegation-alert-for-customer) | Déploie une alerte dans un locataire lorsqu’un utilisateur délègue un abonnement à un locataire gérant.|
| [workbook-activitylogs-by-domain](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain) | Affiche les journaux d’activité Azure dans les abonnements avec une option pour les filtrer par nom de domaine. |
