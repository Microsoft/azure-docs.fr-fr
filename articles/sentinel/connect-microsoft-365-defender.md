---
title: Connecter des données brutes Microsoft 365 Defender à Azure Sentinel | Microsoft Docs
description: Découvrez comment ingérer des données d’événement brutes de Microsoft 365 Defender dans Azure Sentinel.
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
ms.date: 10/13/2019
ms.author: yelevin
ms.openlocfilehash: 756c245fe06ae81545a125dd98f30fb27fdff2dd
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655577"
---
# <a name="connect-data-from-microsoft-365-defender-to-azure-sentinel"></a>Connecter des données de Microsoft 365 Defender à Azure Sentinel

> [!IMPORTANT]
>
> **Microsoft 365 Defender** était anciennement connu sous le nom de **Protection Microsoft contre les menaces** ou **MTP**.
>
> **Microsoft Defender for Endpoint** était précédemment appelé **Microsoft Defender Advanced Threat Protection**, ou **MDATP**.
>
> Vous pouvez voir les anciens noms encore en cours d’utilisation pendant un certain temps.

## <a name="background"></a>Arrière-plan

Le nouveau connecteur [Microsoft 365 Defender](/microsoft-365/security/mtp/microsoft-threat-protection) vous permet de diffuser des journaux de **recherche avancée de menaces**, un type de données d’événement brutes, de Microsoft 365 Defender vers Azure Sentinel. 

Avec l’intégration de [Microsoft Defender for Endpoint (MDATP)](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) dans le cadre de la sécurité Microsoft 365 Defender, vous pouvez désormais collecter vos événements de [recherche avancée de menaces](/windows/security/threat-protection/microsoft-defender-atp/advanced-hunting-overview) Microsoft Defender for Endpoint à l’aide du connecteur Microsoft 365 Defender et les diffuser directement dans de nouvelles tables dédiées dans votre espace de travail Azure Sentinel. Ces tables sont générées sur le même schéma que celui utilisé dans le portail Microsoft 365 Defender, ce qui vous donne un accès complet à l’ensemble des journaux de recherche avancée de menaces et vous permet d’effectuer les opérations suivantes :

- Copiez facilement vos requêtes de recherche avancée de menaces Microsoft Defender ATP existantes dans Azure Sentinel.

- Utilisez les journaux des événements bruts afin de fournir des insights supplémentaires pour vos alertes, la recherche de menaces et l’investigation, et mettez en corrélation les événements avec les données provenant de sources de données supplémentaires dans Azure Sentinel.

- Stockez les journaux avec une période de conservation accrue, au-delà de la durée par défaut de 30 jours de Microsoft Defender for Endpoint ou Microsoft 365 Defender. Pour ce faire, vous pouvez configurer la période de conservation de votre espace de travail ou la conservation par table dans Log Analytics.

> [!IMPORTANT]
>
> Le connecteur Microsoft 365 Defender est actuellement en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’une licence valide pour Microsoft Defender for Endpoint, comme décrit dans [Configurer le déploiement de Microsoft Defender for Endpoint](/windows/security/threat-protection/microsoft-defender-atp/licensing). 

- Le rôle Administrateur général doit être attribué à votre utilisateur sur le locataire (dans Azure Active Directory).

## <a name="connect-to-microsoft-365-defender"></a>Se connecter à Microsoft 365 Defender

Si Microsoft Defender for Endpoint est déployé et ingère vos données, les journaux des événements peuvent facilement être diffusés vers Azure Sentinel.

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données**, **Microsoft 365 Defender (préversion)** dans la galerie, puis **Ouvrir la page du connecteur**.

1. Les types d’événements suivants peuvent être collectés à partir de leurs tables de recherche avancée de menaces correspondantes. Cochez les cases associées aux types d’événements que vous souhaitez collecter :

    | Types d’événements | Nom de la table |
    |-|-|
    | Informations sur l’ordinateur (notamment les informations sur le système d’exploitation) | DeviceInfo |
    | Propriétés réseau des machines | DeviceNetworkInfo |
    | Création de processus et événements associés | DeviceProcessEvents |
    | Connexion réseau et événements associés | DeviceNetworkInfo |
    | Création de fichier, modification et autres événements du système de fichiers | DeviceFileEvents |
    | Création et modification des entrées du Registre | DeviceRegistryEvents |
    | Connexions et autres événements d’authentification | DeviceLogonEvents |
    | Événements de chargement de DLL | DeviceImageLoadEvents |
    | Types d’événements supplémentaires | DeviceEvents |
    |

1. Cliquez sur **Appliquer les modifications**. 

1. Pour interroger les tables de recherche avancée de menaces dans Log Analytics, entrez le nom de la table de la liste ci-dessus dans la fenêtre de requête.

## <a name="verify-data-ingestion"></a>Vérifier l’ingestion de données

Le graphique de données de la page du connecteur indique que vous ingérez des données. Vous remarquerez qu’il affiche une seule ligne qui agrège le volume d’événements dans toutes les tables activées. Une fois que vous avez activé le connecteur, vous pouvez utiliser la requête KQL suivante pour générer un graphique similaire du volume d’événements pour une seule table (remplacez la table *DeviceEvents* par la table requise de votre choix) :

```kusto
let Now = now();
(range TimeGenerated from ago(14d) to Now-1d step 1d
| extend Count = 0
| union isfuzzy=true (
    DeviceEvents
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now)
)
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now)
| sort by TimeGenerated
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events")
| render timechart
```

Dans l’onglet **Étapes suivantes**, vous trouverez quelques exemples de requêtes qui ont été inclus. Vous pouvez les exécuter sur place ou les modifier et les enregistrer.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à obtenir des données d’événement brutes à partir de Microsoft Defender for Endpoint dans Azure Sentinel, à l’aide du connecteur Microsoft 365 Defender. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).