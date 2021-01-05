---
title: Connecter des données CyberArk EPV à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de données CyberArk Enterprise Password Vault (EPV) pour extraire ses journaux dans Azure Sentinel. Affichez les données CyberArk EPV dans des classeurs, créez des alertes et améliorez les enquêtes.
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
ms.openlocfilehash: c375595951eb760d5341db424c5572719b97046a
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102664"
---
# <a name="connect-cyberark-enterprise-password-vault-epv-to-azure-sentinel"></a>Connecter CyberArk Enterprise Password Vault (EPV) à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données CyberArk Enterprise Password Vault (EPV) dans Azure Sentinel est actuellement disponible en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur CyberArk Syslog vous permet de connecter facilement les journaux de votre solution de sécurité CyberArk à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L’intégration entre CyberArk et Azure Sentinel utilise le connecteur de données CEF pour analyser et afficher correctement les messages CyberArk Syslog.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-cyberark-epv"></a>Configuration et connexion de CyberArk EPV

Les journaux CyberArk EPV sont envoyés de Vault à un serveur de transfert de journaux basés sur Linux (exécutant rsyslog ou syslog-ng) sur lequel l’agent Log Analytics est installé, qui exporte les journaux vers Azure Sentinel. Si vous n’avez pas de serveur de transfert de journaux de ce type, consultez [ces instructions](connect-cef-agent.md) pour en obtenir un.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** , sélectionnez **CyberArk Enterprise Password Vault (EPV) Events (Preview)** , puis **Ouvrir la page du connecteur**.

1. Suivez les instructions CyberArk EPV pour configurer l’envoi de données syslog au serveur de transfert de journaux.

1. Validez votre connexion et vérifiez l’ingestion des données à l’aide de [ces instructions](connect-cef-verify.md). Environ 20 minutes peuvent être nécessaires avant que vos journaux commencent à apparaître dans Log Analytics.

## <a name="find-your-data"></a>Recherche de données

Après l’établissement d’une connexion réussie, les données s’affichent dans **Journaux** , sous la section **Azure Sentinel** dans le tableau *CommonSecurityLog*.

Pour interroger les journaux CyberArk EPV dans Log Analytics, entrez `CommonSecurityLog` en haut de la fenêtre de requête.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter des journaux CyberArk Enterprise Password Vault à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
