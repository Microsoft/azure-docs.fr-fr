---
title: Connecter des données d’activité Azure à Azure Sentinel | Microsoft Docs
description: Diffusez des journaux d’activité Azure dans Azure Sentinel en un seul clic. Le journal d’activité enregistre et affiche les événements au niveau de l’abonnement dans Azure.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 8c25baa8-b93b-41da-9e6c-15bb7b5c5511
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/22/2020
ms.author: yelevin
ms.openlocfilehash: 88f363406ec40dfa439a52ad351501d1c64dce95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85564491"
---
# <a name="connect-data-from-azure-activity-log"></a>Connecter des données à partir du journal d’activité Azure

Vous pouvez diffuser des journaux dans Azure Sentinel en un seul clic à partir du [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Le journal d’activité est un journal d’abonnement qui enregistre et affiche les événements au niveau de l’abonnement qui se produisent dans Azure, des données opérationnelles d’Azure Resource Manager aux mises à jour des événements Service Health. À l’aide du journal d’activité, vous pouvez déterminer « quoi, qui et quand » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur les ressources de votre abonnement. Vous pouvez également connaître l’état de l’opération et d’autres propriétés pertinentes. Le journal d’activité n’inclut pas d’opérations de lecture (GET) ni d’opérations pour les ressources qui utilisent le modèle Classic/« RDFE ». 

## <a name="prerequisites"></a>Prérequis

- Votre utilisateur doit disposer d’autorisations de contributeur pour l’espace de travail Log Analytics.
- Votre utilisateur doit disposer d’autorisations de lecteur pour tous les abonnements dont vous souhaitez diffuser les journaux dans Azure Sentinel.

## <a name="set-up-the-azure-activity-connector"></a>Configurer le connecteur Activity

1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**. Dans la liste des connecteurs, cliquez sur **Azure Activity**, puis sur le bouton **Ouvrir la page du connecteur** dans le coin inférieur droit.

2. Sous l’onglet **Instructions**, cliquez sur le lien **Configurer les journaux d’activité Azure >** .

3. Dans le volet **Journal d’activité Azure**, sélectionnez les abonnements dont vous souhaitez diffuser les journaux dans Azure Sentinel. 

4. Dans le volet Abonnement qui s’ouvre sur la droite, cliquez sur **Se connecter**.

5. Pour utiliser le schéma approprié dans Log Analytics pour les alertes d’activité Azure, saisissez `AzureActivity`dans la fenêtre de requête.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter le journal d’activité Azure à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Commencez à détecter les menaces avec Azure Sentinel à l’aide de règles [intégrées](tutorial-detect-threats-built-in.md) ou [personnalisées](tutorial-detect-threats-custom.md).
