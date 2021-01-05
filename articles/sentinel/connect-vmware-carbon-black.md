---
title: Connecter des données VMware Carbon Black Cloud Endpoint Standard à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données VMware Carbon Black Cloud Endpoint Standard à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: e13b251c9bd95a5b52d63d8ea1bbf265c9c46fd8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023872"
---
# <a name="connect-your-vmware-carbon-black-cloud-endpoint-standard-to-azure-sentinel-with-azure-function"></a>Connecter VMware Carbon Black Cloud Endpoint Standard à Azure Sentinel avec Azure Functions

> [!IMPORTANT]
> Le connecteur de données VMware Carbon Black Cloud Endpoint Standard dans Azure Sentinel est disponible en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur VMware Carbon Black Cloud Endpoint Standard vous permet de connecter facilement tous les journaux de votre solution de sécurité [VMware Carbon Black Endpoint Standard](https://www.carbonblack.com/products/endpoint-standard/) à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer l’investigation. L’intégration entre VMware Carbon Black et Azure Sentinel utilise Azure Functions pour extraire des données de journal à l’aide de l’API REST.


> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-vmware-carbon-black-cloud-endpoint-standard"></a>Configurer et connecter VMware Carbon Black Cloud Endpoint Standard

Azure Functions peut intégrer et extraire des événements et des journaux directement à partir de VMware Carbon Black Cloud Endpoint Standard et les transmettre à Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** et sélectionnez le connecteur **VMware Carbon Black**.
2. Sélectionnez **Ouvrir la page du connecteur**.
3. Suivez les instructions de la page **VMware Carbon Black**.


## <a name="find-your-data"></a>Recherche de données

Une fois la connexion correctement établie, les données apparaissent dans Log Analytics sous les tableaux **CarbonBlackAuditLogs_CL**, **CarbonBlackNotifications_CL** et ****CarbonBlackEvents_CL****.

## <a name="validate-connectivity"></a>Valider la connectivité
Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 


## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter VMware Carbon Black Cloud Endpoint Standard à Azure Sentinel à l’aide d’applications de fonction Azure. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.

