---
title: Connecter des données Alcide kAudit à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Alcide kAudit à Azure Sentinel.
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
ms.date: 06/21/2020
ms.author: yelevin
ms.openlocfilehash: cf8da1d88529a823ff4399fb955c8a5e0abbd20e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87038237"
---
# <a name="connect-your-alcide-kaudit-to-azure-sentinel"></a>Connecter votre instance Alcide kAudit à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Alcide kAudit dans Azure Sentinel est actuellement disponible en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Alcide kAudit](https://www.alcide.io/kaudit-K8s-forensics/) vous permet d’identifier les comportements Kubernetes anormaux et de vous concentrer sur les violations et les incidents rencontrés par Kubernetes, tout en réduisant le temps de détection. Cet article explique comment connecter votre solution Alcide kAudit à Azure Sentinel. Le connecteur de données Alcide kAudit vous permet de placer facilement vos données de journal kAudit dans Azure Sentinel, ce qui vous permet ensuite de les consulter dans des classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer pour améliorer l’investigation. L’intégration entre Alcide kAudit et Azure Sentinel utilise l’API REST.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel.

- Vous devez disposer d’autorisations de lecture sur les clés partagées pour accéder à l’espace de travail.

## <a name="configure-and-connect-alcide-kaudit"></a>Configurer et connecter Alcide kAudit

Alcide kAudit peut exporter les journaux directement vers Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** dans le menu de navigation.

1. Sélectionnez **Alcide kAudit**  dans la galerie, puis cliquez sur le bouton de la **page d’ouverture du connecteur**.

1. Suivez les instructions pas à pas fournies dans le [Guide d’installation Alcide kAudit](https://get.alcide.io/hubfs/Azure%20Sentinel%20Integration%20with%20kAudit.pdf).

1. Lorsque vous êtes invité à entrer l’ID de l’espace de travail et la clé primaire, vous pouvez les copier à partir de la page du connecteur de données Alcide kAudit.

    :::image type="content" source="media/connect-alcide-kaudit/alcide-workspace-id-primary-key.png" alt-text="ID d’espace de travail et clé primaire":::

## <a name="find-your-data"></a>Recherche de données

Après l’établissement d’une connexion réussie, les données s’affichent dans **Journaux** sous les types de données suivants dans **CustomLogs** :

- **alcide_kaudit_detections_1_CL** - Détections Alcide kAudit 
- **alcide_kaudit_activity_1_CL** - Journaux d’activité Alcide kAudit
- **alcide_kaudit_selections_count_1_CL** - Nombres d’activités Alcide kAudit
- **alcide_kaudit_selections_details_1_CL** - Détails des activités Alcide kAudit

Pour utiliser le schéma approprié dans les journaux pour Alcide kAudit, recherchez les types de données mentionnés ci-dessus.

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Alcide kAudit à Azure Sentinel. Pour tirer pleinement parti des fonctionnalités intégrées à ce connecteur de données, cliquez sur l’onglet **Étapes suivantes** de la page du connecteur de données. Vous y trouverez des exemples de requêtes prêtes à l’emploi pour commencer à trouver des informations utiles.

Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
