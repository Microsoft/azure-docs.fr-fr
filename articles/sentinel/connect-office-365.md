---
title: Connecter des journaux Office 365 à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de journal Office 365 pour apporter des informations sur les activités de l’utilisateur et de l’administrateur en cours dans Exchange, Teams et SharePoint, OneDrive inclus.
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
ms.date: 08/30/2020
ms.author: yelevin
ms.openlocfilehash: 4d3e3615ede7406a3b581171ae759ec0ec53f13e
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655368"
---
# <a name="connect-office-365-logs-to-azure-sentinel"></a>Connecter des journaux Office 365 à Azure Sentinel

Le connecteur de journal d’activité [Office 365](/office/) fournit des informations Azure Sentinel sur les activités de l’utilisateur et de l’administrateur en cours dans **Exchange** et **SharePoint** (y compris **OneDrive**), ainsi que, maintenant, dans **Teams**. Ces informations comportent des détails relatifs aux actions telles que le téléchargement de fichiers, les demandes d’accès envoyées, les changements apportés aux événements de groupe, les opérations de boîtes aux lettres, les événements Teams (par ex. concernant la conversation, l’équipe, les membres et les canaux), ainsi que les détails de l’utilisateur qui a effectué les actions. En connectant les journaux Office 365 à Azure Sentinel, vous pouvez afficher et analyser ces données dans vos classeurs, les interroger pour créer des alertes personnalisées et les incorporer pour améliorer votre processus d’investigation, ce qui vous donne plus d’informations sur la sécurité d’Office 365.

> [!IMPORTANT]
> L’extension du connecteur  **du journal Office 365 pour les journaux Microsoft Teams** est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel.

- Vous devez être un administrateur général ou un administrateur de la sécurité sur ce locataire.

- Votre déploiement Office 365 doit se trouver sur le même locataire que votre espace de travail Azure Sentinel.

> [!IMPORTANT]
> - Pour permettre au connecteur d’accéder aux données par le biais de l’API Activité de gestion Office 365, vous devez avoir activé la **journalisation d’audit unifiée** sur votre déploiement Office 365. Selon le type de licence Office 365 / Microsoft 365 dont vous disposez, elle peut être activée par défaut ou non. Consultez le [Centre de conformité et de sécurité Office 365](/office365/servicedescriptions/office-365-platform-service-description/office-365-securitycompliance-center) pour vérifier l’état de la journalisation d’audit unifiée en fonction de votre type de licence.
> - Vous pouvez également activer, désactiver et vérifier manuellement l’état actuel de la journalisation d’audit unifiée Office 365. Pour obtenir des instructions, voir [Activer ou désactiver la recherche dans un journal d’audit Office 365](/office365/securitycompliance/turn-audit-log-search-on-or-off).
> - Pour plus d’informations, consultez la [Référence de l’API Activité de gestion Office 365](/office/office-365-management-api/office-365-management-activity-api-reference).


   > [!NOTE]
   > Comme indiqué ci-dessus, et comme vous le constaterez dans la page du connecteur sous **Types de données**, le connecteur Office 365 Azure Sentinel prend actuellement en charge l’ingestion des journaux d’audit uniquement depuis Microsoft Exchange et SharePoint (y compris OneDrive) **ainsi que, désormais, également de Teams**. Cela étant, il existe des solutions externes si vous souhaitez placer d’[autres données Office](https://techcommunity.microsoft.com/t5/azure-sentinel/ingesting-office-365-alerts-with-graph-security-api/ba-p/984888) dans Azure Sentinel. 

## <a name="enable-the-office-365-log-connector"></a>Activer le connecteur de journal d’activité Office 365

### <a name="instructions-tab"></a>Onglet Instructions

1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**.

1. Dans la galerie **Connecteurs de données**, sélectionnez **Office 365**, puis sélectionnez **Ouvrir la page du connecteur** dans le volet de visualisation.

1. Sous la section intitulée **Configuration**, activez les cases à cocher correspondant aux journaux d’activité Office 365 que vous souhaitez connecter à Azure Sentinel, puis cliquez sur **Appliquer les modifications**. 

   > [!NOTE]
   > Si vous avez précédemment connecté plusieurs locataires à Azure Sentinel, à l’aide d’une ancienne version du connecteur Office 365 le prenant en charge, vous serez en mesure d’afficher et de modifier les journaux collectés à partir de chaque locataire. Vous ne serez pas en mesure d’ajouter des locataires supplémentaires, mais vous pouvez supprimer les locataires ajoutés précédemment.

### <a name="next-steps-tab"></a>Onglet Étapes suivantes

- Consultez les classeurs, les exemples de requêtes et les modèles de règle d’analytique recommandés qui sont fournis avec le connecteur de journal **Office 365** pour obtenir des insights sur les données de votre journal SharePoint, OneDrive, Exchange et Teams.

- Pour interroger manuellement les données du journal Office 365 dans les **Journaux**, entrez `OfficeActivity` sur la première ligne de la fenêtre de requête.
   - Pour filtrer la requête pour un type de journal spécifique, entrez `| where OfficeWorkload == "<logtype>"` sur la deuxième ligne de la requête, où *\<logtype\>* est `SharePoint`, `OneDrive`, `Exchange` ou `MicrosoftTeams`.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Office 365 à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Commencez à détecter les menaces avec Azure Sentinel à l’aide de règles [intégrées](tutorial-detect-threats-built-in.md) ou [personnalisées](tutorial-detect-threats-custom.md).