---
title: Connecter des données de Microsoft Defender for Endpoint à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données de Microsoft Defender for Endpoint (anciennement Microsoft Defender ATP) à Azure Sentinel.
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
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: 72b2ba0ea444fb14ef9fc1bc3ea6aea3654677df
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655525"
---
# <a name="connect-alerts-from-microsoft-defender-for-endpoint-formerly-microsoft-defender-atp"></a>Connecter des alertes de Microsoft Defender for Endpoint (anciennement Microsoft Defender ATP)

> [!IMPORTANT]
>
> - **Microsoft Defender for Endpoint** était précédemment appelé **Microsoft Defender Advanced Threat Protection**, ou **MDATP**.
>
>     Il se peut que vous constatiez que l’ancien nom sera encore utilisé dans le produit (notamment son connecteur de données dans Azure Sentinel) pendant un certain temps.
>
> - L’ingestion des alertes de Microsoft Defender pour Endpoint est actuellement disponible en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur [Microsoft Defender for Endpoint](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) vous permet de diffuser en continu des alertes de Microsoft Defender for Endpoint vers Azure Sentinel. Vous pouvez ainsi analyser de manière plus détaillée les événements de sécurité se produisant dans votre organisation et générer des playbooks afin d'y répondre immédiatement et efficacement.

> [!NOTE]
>
> Pour ingérer les nouveaux journaux de données brutes de [repérage avancé](/windows/security/threat-protection/microsoft-defender-atp/advanced-hunting-overview) de Microsoft Defender pour Endpoint, utilisez le nouveau connecteur pour Microsoft 365 Defender (anciennement Microsoft Threat Protection, [voir la documentation](./connect-microsoft-365-defender.md)).

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’une licence valide pour Microsoft Defender for Endpoint, comme décrit dans [Configurer le déploiement de Microsoft Defender for Endpoint](/windows/security/threat-protection/microsoft-defender-atp/licensing). 

- Vous devez être Administrateur général ou Administrateur de la sécurité sur le locataire Azure Sentinel.

## <a name="connect-to-microsoft-defender-for-endpoint"></a>Connexion à Microsoft Defender for Endpoint

Si Microsoft Defender for Endpoint est déployé et ingère vos données, les alertes peuvent facilement être diffusées vers Azure Sentinel.

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données**, puis **Microsoft Defender for Endpoint** (peut être encore appelé *Microsoft Defender Advanced Threat Protection*) dans la Galerie, puis sélectionnez **Ouvrir la page du connecteur**.

1. Cliquez sur **Se connecter**. 

1. Pour interroger les alertes de Microsoft Defender for Endpoint, dans **Journaux**, entrez **SecurityAlert** dans la fenêtre de requête, puis ajoutez un filtre où le **Nom du fournisseur** est **MDATP**.

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a montré comment connecter Microsoft Defender for Endpoint à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).