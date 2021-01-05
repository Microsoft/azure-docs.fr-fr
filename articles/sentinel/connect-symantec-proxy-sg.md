---
title: Connecter des données Symantec ProxySG data à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Symantec ProxySG à Azure Sentinel.
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
ms.openlocfilehash: fe49481357b542ce52de5fc9dab3059cb584c229
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87083903"
---
# <a name="connect-your-symantec-proxysg-to-azure-sentinel"></a>Connexion de votre Symantec ProxySG à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Symantec ProxySG dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre appliance [Symantec ProxySG](https://www.broadcom.com/products/cyber-security/network/gateway/proxy-sg-and-advanced-secure-gateway) à Azure Sentinel. Le connecteur de données Symantec ProxySG vous permet de connecter facilement vos journaux Symantec ProxySG à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L'intégration entre Symantec ProxySG et Azure Sentinel utilise Syslog.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="forward-symantec-proxysg-logs-to-the-syslog-agent"></a>Transférer des journaux Symantec ProxySG à l’agent Syslog  

Configurez Symantec ProxySG pour transférer les messages Syslog à votre espace de travail Azure par le biais de l’agent Syslog.

1. Dans le portail Azure Sentinel, cliquez sur **Data Connectors** (Connecteurs de données) et sélectionnez le connecteur **Symantec ProxySG**.

1. Sélectionnez **Ouvrir la page du connecteur**.

1. Suivez les instructions de la page **Symantec ProxySG**.

## <a name="find-your-data"></a>Recherche de données

Une fois la connexion correctement établie, les données apparaissent dans l’espace de travail Log Analytics sous Syslog.

## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Symantec ProxySG à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.