---
title: Connecter des données Squadra Technologies secRMM à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Squadra Technologies secRMM à Azure Sentinel.
services: sentinel
author: yelevin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/20/2020
ms.author: yelevin
ms.openlocfilehash: d904e51321870fb4b61a237c23e425034b76dc0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77588108"
---
# <a name="connect-your-squadra-technologies-secrmm-data-to-azure-sentinel"></a>Connecter vos données Squadra Technologies secRMM à Azure Sentinel 

> [!IMPORTANT]
> Le connecteur de données Squadra Technologies secRMM (Security Removable Media Manager) dans Azure Sentinel est en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Le connecteur Squadra Technologies secRMM vous permet de connecter facilement les journaux de la solution de sécurité Squadra Technologies secRMM à Azure Sentinel. Il vous permet de voir des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. Ce connecteur vous donne des insights sur les événements liés au stockage amovible USB. L’intégration entre Squadra Technologies secRMM et Azure Sentinel est basée sur l’utilisation de l’API REST.


> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-squadra-technologies-secrmm"></a>Configurer et connecter Squadra Technologies secRMM 

La solution Squadra Technologies secRMM peut intégrer et exporter des journaux directement vers Azure Sentinel.
1. Dans le portail Azure Sentinel, cliquez sur Data Connectors (Connecteurs de données), sélectionnez Squadra Technologies secRMM, puis Open Connector Page (Ouvrir la page du connecteur).

2. Suivez les étapes décrites dans le [guide d’intégration de Squadra Technologies pour Azure Sentinel](http://www.squadratechnologies.com/StaticContent/ProductDownload/secRMM/9.9.0.0/secRMMAzureSentinelAdministratorGuide.pdf) afin de récupérer des données Squadra secRMM dans Azure Sentinel.   


## <a name="find-your-data"></a>Rechercher vos données

Une fois la connexion correctement établie, les données apparaissent dans l’espace de travail Log Analytics sous CustomLogs secRMM_CL.
Si vous souhaitez utiliser le schéma approprié dans l’espace de travail Log Analytics pour Squadra Technologies secRMM, recherchez secRMM_CL.

## <a name="validate-connectivity"></a>Valider la connectivité
Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 


## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Squadra Technologies secRMM à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.

