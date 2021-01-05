---
title: Connecter des données Citrix WAF à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de données Citrix WAF pour tirer vos journaux Citrix WAF dans Azure Sentinel. Affichez les données de l’affichage Citrix WAF dans des classeurs, créez des alertes et améliorez l’investigation.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0001cad6-699c-4ca9-b66c-80c194e439a5
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/25/2020
ms.author: yelevin
ms.openlocfilehash: 475e04ad3dd8c7cc51d6345b4b51dd68cff5b597
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102663"
---
# <a name="connect-your-citrix-waf-to-azure-sentinel"></a>Connecter votre Citrix WAF à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Citrix Web Application Firewall (WAF) dans Azure Sentinel est disponible en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre appliance Citrix Web Application Firewall (WAF) à Azure Sentinel. Le connecteur de données Citrix WAF vous permet de connecter facilement vos journaux Citrix WAF à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. En connectant vos journaux d’activité Citrix WAF à Azure Sentinel, vous pouvez tirer parti de la recherche et de la corrélation, des alertes et des informations riches de Threat Intelligence pour chaque journal.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="forward-citrix-waf-logs-to-the-syslog-agent"></a>Transférer les journaux Citrix WAF à l’agent Syslog  

Citrix WAF envoie des messages Syslog au format CEF à un serveur de transfert de journaux Linux (exécutant rsyslog ou syslog-ng) sur lequel l’agent Log Analytics est installé, qui transfère les journaux à votre espace de travail Azure Sentinel.

1. Si vous n’avez pas de serveur de transfert de journaux de ce type, consultez [ces instructions](connect-cef-agent.md) pour en obtenir un.

1. Suivez les instructions de Citrix pour [configurer le WAF](https://support.citrix.com/article/CTX234174), [configurer la journalisation CEF](https://support.citrix.com/article/CTX136146) et [configurer l’envoi des journaux à votre redirecteur de journal](https://docs.citrix.com/en-us/citrix-adc/13/system/audit-logging/configuring-audit-logging.html). Veillez à envoyer les journaux vers le port TCP 514 sur l’adresse IP de l’ordinateur du redirecteur des journaux.

1. Validez votre connexion et vérifiez l’ingestion des données à l’aide de [ces instructions](connect-cef-verify.md). Environ 20 minutes peuvent être nécessaires avant que vos journaux commencent à apparaître dans Log Analytics.

## <a name="find-your-data"></a>Recherche de données

Après l’établissement d’une connexion réussie, les données s’affichent dans **Journaux** , sous la section **Azure Sentinel** dans le tableau *CommonSecurityLog*.

Pour interroger les journaux Citrix WAF dans Log Analytics, entrez `CommonSecurityLog` en haut de la fenêtre de requête.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous découvrirez comment connecter Citrix WAF à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.