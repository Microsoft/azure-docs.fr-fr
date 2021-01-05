---
title: Connecter des données Cloud App Security à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur Microsoft Cloud App Security (MCAS) pour diffuser en continu des alertes et des journaux Cloud Discovery entre MCAS et Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/24/2020
ms.author: yelevin
ms.openlocfilehash: 9827e93f793ef47a835aa0a5fb2005e2830a0824
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655712"
---
# <a name="connect-data-from-microsoft-cloud-app-security"></a>Connecter des données à partir de Microsoft Cloud App Security 

Le connecteur [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) (MCAS) vous permet de diffuser en continu des alertes et des [journaux Cloud Discovery](/cloud-app-security/tutorial-shadow-it) entre MCAS et Azure Sentinel. Vous bénéficiez ainsi d’une visibilité accrue sur vos applications, de fonctionnalités d’analytique sophistiquées pour identifier et combattre les cybermenaces, et contrôlez le déplacement de vos données.

## <a name="prerequisites"></a>Prérequis

- Votre utilisateur doit disposer des autorisations de lecture et d’écriture pour l’espace de travail.
- Votre utilisateur doit disposer des autorisations d'administrateur général ou d'administrateur de la sécurité sur le locataire de l’espace de travail.
- Pour diffuser des journaux Cloud Discovery dans Azure Sentinel, [activez Azure Sentinel comme SIEM dans Microsoft Cloud App Security](/cloud-app-security/siem-sentinel).

> [!IMPORTANT]
> L’ingestion des journaux Cloud Discovery est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
 
## <a name="connect-to-cloud-app-security"></a>Connexion à Cloud App Security

Si vous avez déjà Cloud App Security, assurez-vous qu’il est [activé sur votre réseau](/cloud-app-security/getting-started-with-cloud-app-security).
Si Cloud App Security est déployé et ingère vos données, les données d’alerte peuvent facilement être envoyés à Azure Sentinel.


1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**. Dans la liste des connecteurs, cliquez sur la mosaïque **Microsoft Cloud App Security**, puis sur le bouton **Ouvrir la page du connecteur** dans le coin inférieur droit.

1. Pour sélectionner les journaux que vous souhaitez diffuser en continu dans Azure Sentinel, vous pouvez sélectionner **Alertes** et **Journaux Cloud Discovery** (préversion). 

1. Cliquez sur **Appliquer les modifications**.

1. Vous pouvez décider si les alertes d’Azure Defender génèrent automatiquement des incidents dans Azure Sentinel. Sous **Créer des incidents**, sélectionnez **Activé** pour activer la règle analytique par défaut qui crée automatiquement des incidents à partir d’alertes. Vous pouvez ensuite modifier cette règle sous **Analytique** sous l’onglet **Règles actives**.

1. Pour utiliser le schéma approprié dans Log Analytics pour les alertes Cloud App Security, entrez `SecurityAlert` dans la fenêtre de requête. Pour le schéma des journaux Cloud Discovery, entrez `McasShadowItReporting`.

> [!NOTE]
> Cloud Discovery permet de détecter et d’identifier les tendances en agrégeant les données de connexion des utilisateurs sous-jacentes dans les applications cloud.
>
> Les données Cloud Discovery étant agrégées sur une base quotidienne, les données les plus récentes peuvent mettre jusqu'à 24 heures à apparaître dans Azure Sentinel. Si un examen de bas niveau requiert des données plus immédiates, il doit être effectué directement dans le service ou l’appliance source où résident les données brutes.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Microsoft Cloud App Security à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Commencez à détecter les menaces avec Azure Sentinel à l’aide de règles [intégrées](./tutorial-detect-threats-built-in.md) ou [personnalisées](tutorial-detect-threats-custom.md).