---
title: Connecter des données Microsoft Defender pour Identity (anciennement Azure ATP) à Azure Sentinel | Microsoft Docs
description: Découvrez comment diffuser des journaux à partir de Microsoft Defender pour Identity (anciennement Azure Advanced Threat Protection ou ATP) dans Azure Sentinel en un seul clic.
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
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: b0cafcbc9fa6f0fcb31f44b44dfbe4fc3e10c17d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656069"
---
# <a name="connect-data-from-microsoft-defender-for-identity-formerly-azure-advanced-threat-protection"></a>Connecter des données à partir de Microsoft Defender pour Identity (anciennement Azure Advanced Threat Protection)

> [!IMPORTANT]
> Le connecteur de données Microsoft Defender pour Identity dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Vous pouvez diffuser des journaux à partir de [Microsoft Defender pour Identity](/azure-advanced-threat-protection/what-is-atp) dans Azure Sentinel en un seul clic.

## <a name="prerequisites"></a>Prérequis

- Utilisateur, doté d’autorisations d’administrateur général ou d’administrateur de la sécurité
- Vous devez être un client de la préversion de Microsoft Defender pour Identity et activer l’intégration entre Microsoft Defender pour Identity et Microsoft Cloud App Security. Pour plus d’informations, consultez [Intégration de Microsoft Defender pour Identity](https://www.microsoft.com/microsoft-365/identity/advance-threat-protection).

## <a name="connect-to-microsoft-defender-for-identity"></a>Se connecter à Microsoft Defender pour Identity

Assurez-vous que la version de la préversion de Microsoft Defender pour Identity est [activée sur votre réseau](/azure-advanced-threat-protection/install-atp-step1).
Si Microsoft Defender pour Identity est déployé et ingère vos données, les alertes suspectes peuvent facilement être diffusées vers Azure Sentinel. Le démarrage de la diffusion en continu des alertes dans Azure Sentinel peut prendre jusqu’à 24 heures.


1. Pour connecter Microsoft Defender pour Identity à Azure Sentinel, vous devez d’abord activer l’intégration entre Microsoft Defender pour Identity et Microsoft Cloud App Security. Pour plus d’informations sur la procédure à suivre, consultez [Intégration de Microsoft Defender pour Identity](https://www.microsoft.com/microsoft-365/identity/advance-threat-protection).

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données**, puis cliquez sur la vignette **Microsoft Defender pour Identity (préversion)** .

1. Vous pouvez décider que les alertes provenant de Microsoft Defender pour Identity génèrent automatiquement des incidents dans Azure Sentinel. Sous **Créer des incidents**, sélectionnez **Activer** pour activer la règle analytique par défaut qui crée automatiquement des incidents à partir des alertes générées dans le service de sécurité connecté. Vous pouvez ensuite modifier cette règle sous **Analytique**, puis **Règles actives**.

1. Cliquez sur **Connecter**.

1. Pour utiliser le schéma pertinent dans Log Analytics pour les alertes Microsoft Defender pour Identity, recherchez **SecurityAlert**.

> [!NOTE]
> Si la taille des alertes excède 30 Ko, Azure Sentinel cesse d’afficher le champ Entités dans les alertes.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Microsoft Defender pour Identity à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).