---
title: Connecter des données Beyond Security beSECURE à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de données Beyond Security beSECURE pour extraire des journaux beSECURE dans Azure Sentinel. Affichez les données beSECURE dans des classeurs, créez des alertes et améliorez l’examen.
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
ms.openlocfilehash: f6fd6920fafe4c1080cb5539e4e0222d9d6e18cd
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102643"
---
# <a name="connect-your-beyond-security-besecure-to-azure-sentinel"></a>Connecter Beyond Security beSECURE à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Beyond Security beSECURE dans Azure Sentinel est en préversion publique. Cette fonctionnalité est fournie sans contrat de niveau de service. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur Beyond Security beSECURE vous permet de connecter facilement tous les journaux de votre solution de sécurité beSECURE à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer l’examen. L’intégration entre beSECURE et Azure Sentinel utilise l’API REST.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-besecure"></a>Configurer et connecter beSECURE

beSECURE peut s’intégrer à Azure Sentinel et y exporter directement des journaux.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** et sélectionnez **Beyond Security beSECURE (préversion)** , puis **Ouvrir la page du connecteur**.

1. Suivez les étapes ci-dessous pour configurer votre solution beSECURE afin d’envoyer des résultats d’analyse, l’état d’analyse et les journaux de piste d’audit à Azure Sentinel.

    **Accédez au menu Intégration :**
    1. Cliquer sur l’option de menu Plus

    1. Sélectionner un serveur

    1. Sélectionner Intégration

    1. Activer Azure Sentinel

    **Fournissez à beSECURE des paramètres Azure Sentinel.**
      - Copiez les valeurs *ID de l’espace de travail* et *Clé primaire* de la page du connecteur Azure Sentinel, collez-les dans la configuration beSECURE, puis cliquez sur **Modifier**.

## <a name="find-your-data"></a>Recherche de données

Après l’établissement d’une connexion réussie, les données s’affichent dans **Journaux** sous la section **CustomLogs** , dans une ou plusieurs des tables suivantes :
  - `beSECURE_ScanResults_CL`
  - `beSECURE_ScanEvents_CL`
  - `beSECURE_Audit_CL`

Pour interroger les journaux beSECURE dans Log Analytics, entrez l’un des noms de tables ci-dessus en haut de la fenêtre de requête.

## <a name="validate-connectivity"></a>Valider la connectivité
Jusqu’à 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter beSECURE à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
