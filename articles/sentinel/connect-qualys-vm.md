---
title: Connecter les données de la solution Qualys Vulnerability Management à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter les données de Qualys Vulnerability Management à Azure Sentinel.
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
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 34f2cfa06cbdbb75b8fd610cd1f76fb33dde4cde
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87059855"
---
# <a name="connect-your-qualys-vm-to-azure-sentinel-with-azure-function"></a>Connecter votre solution Qualys VM à Azure Sentinel à l’aide d’Azure Functions

> [!IMPORTANT]
> Le connecteur de données Qualys VM dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le connecteur Qualys Vulnerability Management (VM) vous permet de connecter facilement tous les journaux de votre solution de sécurité [Qualys VM](https://www.qualys.com/apps/vulnerability-management/) à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L’intégration entre Qualys VM et Azure Sentinel utilise Azure Functions pour extraire des données de journal à l’aide de l’API REST.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-qualys-vm"></a>Configuration et connexion de Qualys VM

Azure Functions peut intégrer et extraire des événements et des journaux directement à partir de Qualys VM et les transmettre à Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Connecteurs de données** et sélectionnez le connecteur **Qualys Vulnerability Management**.

1. Sélectionnez **Ouvrir la page du connecteur**.

1. Suivez les instructions de la page **Qualys Vulnerability Management**.

## <a name="find-your-data"></a>Recherche de données

Une fois la connexion établie, les données apparaissent dans Log Analytics sous la table **QualysHostDetection_CL**.

## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Qualys VM à Azure Sentinel à l’aide d’applications de fonction Azure. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
