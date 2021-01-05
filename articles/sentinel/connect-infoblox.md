---
title: Connecter les données du système d’exploitation des identités réseau Infoblox à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter les données du système d’exploitation des identités réseau Infoblox à Azure Sentinel.
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
ms.openlocfilehash: c8af1c32607e5fc7bf548b40069e18923c666303
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87089649"
---
# <a name="connect-your-infoblox-nios-to-azure-sentinel"></a>Connexion de votre système d’exploitation des identités réseau Infoblox à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Infoblox NIOS dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre appliance [du système d’exploitation des identités réseau Infoblox](https://www.infoblox.com/glossary/network-identity-operating-system-nios/) à Azure Sentinel. Le connecteur de données du système d’exploitation des identités réseau Infoblox vous permet de connecter facilement vos journaux Infoblox, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L'intégration entre le système d’exploitation des identités réseau Infoblox et Azure Sentinel utilise Syslog.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="forward-infoblox-logs-to-the-syslog-agent"></a>Transférer des journaux Infoblox à l’agent Syslog  

Configurez Infoblox pour transférer les messages Syslog à votre espace de travail Azure par le biais de l’agent Syslog.

1. Dans le portail Azure Sentinel, cliquez sur **Data Connectors** (Connecteurs de données) et sélectionnez le connecteur **Infoblox NIOS**.

1. Sélectionnez **Ouvrir la page du connecteur**.

1. Suivez les instructions sur la page **Infoblox NIOS**.

## <a name="find-your-data"></a>Recherche de données

Une fois la connexion correctement établie, les données apparaissent dans l’espace de travail Log Analytics sous Syslog.

## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter les appliances CEF à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.