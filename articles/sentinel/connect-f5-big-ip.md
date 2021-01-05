---
title: Connecter des données F5 BIG-IP à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données F5 Big-IP à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d068223f-395e-46d6-bb94-7ca1afd3503c
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: ae361c74b261bdd6a5673040f868392282b573ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77588278"
---
# <a name="connect-your-f5-big-ip-appliance"></a>Connecter votre appliance F5 BIG-IP 

> [!IMPORTANT]
> Le connecteur de données F5 BIG-IP dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur de données F5 BIG-IP vous permet de connecter facilement tous vos journaux F5 BIG-IP à Azure Sentinel, de consulter des classeurs, de créer des alertes personnalisées et d’améliorer les enquêtes. Cela vous donne plus d’informations sur le réseau de votre organisation et améliore vos capacités d’opération de sécurité. L’intégration entre F5 BIG-IP et Azure Sentinel utilise l’API REST.


> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-f5-big-ip"></a>Configurer et connecter F5 BIG-IP 

F5 BIG-IP peut intégrer et exporter des journaux d’activité directement dans Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Data Connectors** (Connecteurs de données) et sélectionnez **F5 BIG-IP**, puis **Open Connector Page** (Ouvrir la page du connecteur). 
1. Pour connecter votre F5 BIG-IP, vous devez publier une déclaration JSON dans le point de terminaison d’API du système. Pour savoir comment procéder, consultez [Intégration de F5 BIG-IP avec Azure Sentinel](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel).
8. Dans la page du connecteur F5 BIG-IP, copiez l’ID d’espace de travail et la clé primaire, puis collez-les conformément aux instructions sous [Diffusion de données en continu vers Azure Log Analytics](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel#streaming-data-to-azure-log-analytics).
1. Une fois la procédure relative à F5 BIG-IP terminée, les types de données connectés apparaissent dans la page du connecteur Azure Sentinel.
1. Afin d’utiliser le schéma approprié dans Log Analytics pour les événements F5 BIG-IP, recherchez **F5Telemetry_LTM_CL**, **F5Telemetry_system_CL** et **F5Telemetry_ASM_CL**.


## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 



## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter F5 BIG-IP à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.


