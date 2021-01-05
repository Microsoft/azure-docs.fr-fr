---
title: Connecter Zimperium Mobile Threat Defense à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter Zimperium Mobile Threat Defense à Azure Sentinel.
services: sentinel
author: yelevin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/20/2020
ms.author: yelevin
ms.openlocfilehash: 86854fa22a49f09e5d3d2fc5fdb53c245850fbac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77587938"
---
# <a name="connect-your-zimperium-mobile-threat-defense-to-azure-sentinel"></a>Connectez votre Zimperium Mobile Threat Defense à Azure Sentinel


> [!IMPORTANT]
> Le connecteur de données Zimperium Mobile Threat Defense dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).



Le connecteur Zimperium Mobile Threat Defense vous permet de connecter le journal des menaces Zimperium avec Azure Sentinel pour afficher des tableaux de bord, créer des alertes personnalisées et améliorer l’investigation. Cela vous donne plus d’informations sur le paysage des menaces mobiles de votre organisation et améliore vos capacités d’opération de sécurité.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-zimperium-mobile-threat-defense"></a>Configurer et connecter Zimperium Mobile Threat Defense

Zimperium Mobile Threat Defense peut intégrer et exporter des journaux directement dans **Azure Sentinel**.

1. Dans le portail Azure Sentinel, cliquez Connecteurs de données, puis sélectionnez **Zimperium Mobile Threat Defense**.
2. Sélectionnez **Ouvrir la page du connecteur**.
3. Suivez les instructions de la page du connecteur **Zimperium Mobile Threat Defense**, qui se résument comme suit.
 1. Dans zConsole, cliquez sur **Gérer** dans la barre de navigation.
 2. Cliquez sur l’onglet **Integrations** .
 3. Cliquez sur le bouton **Rapports sur les menaces**, puis sur le bouton **Ajouter des intégrations**.
 4. Créez l’intégration en sélectionnant **Microsoft Azure Sentinel** parmi les intégrations disponibles, puis entrez l’ID d’espace de travail et la clé primaire pour la connexion à Azure Sentinel.
 5. Une option permettant de sélectionner un niveau de filtre pour les données de menace à transmettre à Azure Sentinel est également disponible. 

4. Pour plus d’informations, voir le [portail du service clientèle de Zimperium](https://support.zimperium.com).


## <a name="find-your-data"></a>Rechercher vos données

Après l’établissement d’une connexion réussie, les données s’affichent dans Log Analytics sous CustomLogs ZimperiumThreatLog_CL et ZimperiumMitigationLog_CL.

Pour utiliser le schéma approprié pour Zimperium Mobile Threat Defense dans Log Analytics, recherchez ZimperiumThreatLog_CL et ZimperiumMitigationLog_CL.


## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Zimperium Mobile Threat Defense à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).

- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).

- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.

Pour en savoir plus sur Zimperium, voir les rubriques suivantes :

- [Zimperium](https://zimperium.com)

- [Blog Zimperium Mobile Security](https://blog.zimperium.com)

- [Portail du service clientèle de Zimperium](https://support.zimperium.com)

