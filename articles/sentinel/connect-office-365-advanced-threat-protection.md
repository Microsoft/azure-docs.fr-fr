---
title: Connecter des données Microsoft Defender pour Office 365 (anciennement Office 365 ATP) à Azure Sentinel | Microsoft Docs
description: Ingérez les données Microsoft Defender pour Office 365 dans Azure Sentinel pour gagner en visibilité et créer des scénarios de réponse automatisée.
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
ms.date: 09/06/2020
ms.author: yelevin
ms.openlocfilehash: d5140bcd85606213f00185d4ba1f50dbe70dba63
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655492"
---
# <a name="connect-alerts-from-microsoft-defender-for-office-365"></a>Connecter les alertes de Microsoft Defender pour Office 365 

> [!IMPORTANT]
>
> - **Microsoft Defender pour Office 365** était anciennement connu sous le nom **Office 365 Advanced Threat Protection (ATP)** .
>
>     Il se peut que vous constatiez que l’ancien nom sera encore utilisé dans le produit (notamment son connecteur de données dans Azure Sentinel) pendant un certain temps.
>
> - L’ingestion des alertes de Microsoft Defender pour Office 365 est actuellement disponible en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service et n’est pas recommandée pour des charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
 
[Microsoft Defender pour Office 365](/office365/servicedescriptions/office-365-advanced-threat-protection-service-description) protège votre organisation contre les failles zero-day et autres menaces avancées que représentent les programmes malveillants inconnus dans les e-mails, les liens URL malveillants et les outils de collaboration. En ingérant des alertes Microsoft Defender pour Office 365 dans Azure Sentinel, vous pourrez utiliser des informations sur les menaces relatives aux e-mails, partages de fichiers et URL dans vos opérations de sécurité. Vous pouvez ensuite analyser de manière plus détaillée les événements de sécurité se produisant dans votre organisation et générer des playbooks afin d’y répondre immédiatement et efficacement.

Le connecteur importe les alertes suivantes :

- Détection d’un clic d’URL potentiellement malveillant 

- Suppression des e-mails contenant des programmes malveillants après leur livraison

- Suppression des e-mails contenant des URL de hameçonnage après leur livraison 

- E-mail signalé par l’utilisateur comme étant un programme malveillant ou un hameçonnage 

- Détection de modèles d’envoi d’e-mails suspects 

- Utilisateur non autorisé à envoyer des e-mails 

Ces alertes peuvent être consultées par les clients d’Office dans le **centre de sécurité et de conformité d’Office**.

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel lorsque vous activez le connecteur.

- Vous devez disposer du rôle Administrateur général ou Administrateur de la sécurité sur le locataire de l’espace de travail Azure Sentinel.

- Vous devez disposer d’une licence valide pour [Office 365 ATP Plan 2](/microsoft-365/security/office-365-security/office-365-atp#office-365-atp-plan-1-and-plan-2) (inclus avec les licences Office 365 E5, Office 365 A5 et Microsoft 365 E5, et disponible à l’achat séparément). 

## <a name="connect-to-microsoft-defender-for-office-365"></a>Se connecter à Microsoft Defender pour Office 365

Si Microsoft Defender pour Office 365 est déployé et que des stratégies ont été configurées, les alertes peuvent facilement être ingérées dans Azure Sentinel.

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données** dans le menu de navigation.

1. Sélectionnez **Microsoft Defender pour Office 365** (peut toujours être appelé *Office 365 Advanced Threat Protection*) dans la galerie de connecteurs, puis sélectionnez **Ouvrir la page des connecteurs**.

1. Dans la section **Configuration**, cliquez sur **Se connecter**. 

1. Dans la section **Créer des incidents** , cliquez sur **Activer**.

1. Pour utiliser le schéma approprié pour interroger des alertes Office 365 ATP, recherchez **SecurityAlert** et spécifiez **OATP** pour **Nom du fournisseur**.

1. Sélectionnez l’onglet **Étapes suivantes** pour afficher et utiliser les exemples de requête et les modèles de règle d’analyse fournis avec le connecteur Microsoft Defender pour Office 365.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Microsoft Defender pour Office 365 à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).