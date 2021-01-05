---
title: Classeurs Azure Monitor pour les rapports | Microsoft Docs
description: Découvrez comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory.
services: active-directory
author: MarkusVi
manager: daveba
ms.assetid: 4066725c-c430-42b8-a75b-fe2360699b82
ms.service: active-directory
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: identity
ms.subservice: report-monitor
ms.date: 10/30/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 76dc7693b1773038d610f9ed42eddad2c81e3eaf
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93126632"
---
# <a name="how-to-use-azure-monitor-workbooks-for-azure-active-directory-reports"></a>Comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory

> [!IMPORTANT]
> Afin d’optimiser les requêtes sous-jacentes de ce classeur, cliquez sur « Modifier », puis sur l’icône Paramètres et sélectionnez l’espace de travail dans lequel vous souhaitez exécuter ces requêtes. Par défaut, les classeurs sélectionnent tous les espaces de travail dans lesquels vous routez vos journaux d’activité Azure AD. 

Vous voulez...

- comprendre l’impact de vos [stratégies d’accès conditionnel](../conditional-access/overview.md) sur l’expérience de connexion des utilisateurs ?

- mettre un terme aux échecs de connexion pour obtenir un meilleur aperçu de l’intégration des connexions de votre organisation, et résoudre les problèmes plus rapidement ?

- savoir qui utilise l’authentification héritée pour se connecter à votre environnement ? (en [bloquant l’authentification héritée](../conditional-access/block-legacy-authentication.md), vous pouvez améliorer la protection de votre locataire.)

- Avez-vous besoin de comprendre l’impact des stratégies d’accès conditionnel dans votre locataire ?

- Aimeriez-vous passer en revue les requêtes de journal de connexion, le classeur signalant le nombre d’utilisateurs auxquels l’accès a été accordé ou refusé, ainsi que le nombre d’utilisateurs ayant contourné les stratégies d’accès conditionnel lors de l’accès aux ressources ?

- Souhaitez-vous acquérir une compréhension plus approfondie des détails du classeur par condition, afin que l’impact d’une stratégie puisse être contextualisé par condition, y compris la plateforme de l’appareil, l’état de l’appareil, l’application cliente, le risque à la connexion, l’emplacement et l’application ?

- Acquérez une meilleure compréhension des requêtes de journal de connexion, le classeur signalant le nombre d’utilisateurs auxquels l’accès a été accordé ou refusé, ainsi que le nombre d’utilisateurs ayant contourné les stratégies d’accès conditionnel lors de l’accès aux ressources.

- Pour vous aider à résoudre ces problèmes, Azure Active Directory fournit des classeurs à des fins de supervision. Les [classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md) regroupent du texte, des requêtes Analytics, des mesures et des paramètres sous la forme de rapports interactifs riches en contenu.



Cet article :

- suppose que vous êtes familiarisé avec la procédure de [création de rapports interactifs à l’aide de classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md) et

- explique comment utiliser les classeurs Azure Monitor pour comprendre l’impact de vos stratégies d’accès conditionnel, pour résoudre les échecs de connexion et pour identifier les authentifications héritées.
 


## <a name="prerequisites"></a>Prérequis

Pour pouvoir utiliser les classeurs Azure Monitor, vous devez disposer des éléments suivants :

- Un locataire Azure Active Directory avec une licence Premium (P1 ou P2). Découvrez comment [obtenir une licence premium](../fundamentals/active-directory-get-started-premium.md).

- Un [espace de travail Log Analytics](../../azure-monitor/learn/quick-create-workspace.md).

- [Accéder](../../azure-monitor/platform/manage-access.md#manage-access-using-workspace-permissions) à l’espace de travail Log Analytics
- Rôles suivants dans Azure Active Directory (si vous accédez à Log Analytics via le portail Azure Active Directory)
    - Administrateur de sécurité
    - Lecteur de sécurité
    - Lecteur de rapport
    - Administrateur général

## <a name="roles"></a>Rôles
Vous devez être dans l’un des rôles suivants et avoir [accès à l’espace de travail Log Analytics](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions) sous-jacent pour gérer les classeurs :
-   Administrateur général
-   Administrateur de sécurité
-   Lecteur de sécurité
-   Lecteur de rapport
-   Administrateur d’application

## <a name="workbook-access"></a>Accès aux classeurs 

Pour accéder à des classeurs, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez à **Azure Active Directory** > **Supervision** > **Classeurs**. 

1. Sélectionnez un rapport ou un modèle, ou, sur la barre d’outils, sélectionnez **Ouvrir**. 

![Rechercher les classeurs Azure Monitor dans Azure AD](./media/howto-use-azure-monitor-workbooks/azure-monitor-workbooks-in-azure-ad.png)

## <a name="sign-in-analysis"></a>Analyse des connexions

Pour accéder au classeur d’analyse de la connexion, allez dans la section **Utilisation** et sélectionnez **Connexions**. 

Ce classeur montre les tendances suivantes en matière de connexion :

- Toutes les connexions

- Succès

- Action de l’utilisateur en attente

- Échec

Vous pouvez filtrer chaque tendance selon les catégories suivantes :

- Plage temporelle

- Applications

- Utilisateurs

![Analyse des connexions](./media/howto-use-azure-monitor-workbooks/43.png)


Pour chaque tendance, vous obtenez une répartition en fonction des catégories suivantes :

- Emplacement

    ![Connexions par emplacement](./media/howto-use-azure-monitor-workbooks/45.png)

- Appareil

    ![Connexions par appareil](./media/howto-use-azure-monitor-workbooks/46.png)


## <a name="sign-ins-using-legacy-authentication"></a>Connexions avec une authentification héritée 


Pour accéder au classeur afin d’identifier les connexions qui utilisent une [authentification héritée](../conditional-access/block-legacy-authentication.md), accédez à la section **Utilisation** et sélectionnez **Connexions avec une authentification existante**. 

Ce classeur montre les tendances suivantes en matière de connexion :

- Toutes les connexions

- Succès


Vous pouvez filtrer chaque tendance selon les catégories suivantes :

- Plage temporelle

- Applications

- Utilisateurs

- Protocoles

![Connexions avec une authentification héritée](./media/howto-use-azure-monitor-workbooks/47.png)


Pour chaque tendance, vous obtenez une répartition par application et par protocole.

![Connexions avec une authentification héritée via une application et un protocole](./media/howto-use-azure-monitor-workbooks/48.png)



## <a name="sign-ins-by-conditional-access"></a>Connexions par accès conditionnel 


Pour accéder au classeur afin d’identifier les connexions via des [stratégies d’accès conditionnel](../conditional-access/overview.md), accédez à la section **Accès conditionnel** , puis sélectionnez **Sign-ins by Conditional Access** (Connexions par accès conditionnel). 

Ce classeur affiche les tendances relatives aux connexions désactivées. Vous pouvez filtrer chaque tendance selon les catégories suivantes :

- Plage temporelle

- Applications

- Utilisateurs

![Connexions avec accès conditionnel](./media/howto-use-azure-monitor-workbooks/49.png)


Pour les connexions désactivées, vous obtenez une répartition en fonction de l’état de l’accès conditionnel.

![Capture d’écran des zones État de l’accès conditionnel et Connexions récentes](./media/howto-use-azure-monitor-workbooks/conditional-access-status.png)


## <a name="conditional-access-insights"></a>Insights sur l’accès conditionnel

### <a name="overview"></a>Vue d’ensemble

Les classeurs contiennent des requêtes de journal de connexion qui peuvent aider les administrateurs informatiques à surveiller l’impact des stratégies d’accès conditionnel dans leur locataire. Vous avez la possibilité de créer des rapports sur le nombre d’utilisateurs auxquels l’accès a été accordé ou refusé. Le classeur contient des informations sur le nombre d’utilisateurs qui auraient contourné les stratégies d’accès conditionnel en fonction des attributs des utilisateurs au moment de la connexion. Il contient des détails par condition, afin que l’impact d’une stratégie puisse être contextualisé par condition, y compris la plateforme de l’appareil, l’état de l’appareil, l’application cliente, le risque à la connexion, l’emplacement et l’application.

### <a name="instructions"></a>Instructions 
Pour accéder au classeur relatif aux Informations sur l’accès conditionnel, sélectionnez le classeur **Informations sur l’accès conditionnel** dans la section Accès conditionnel. Ce classeur montre l’impact attendu de chaque stratégie d’accès conditionnel dans votre locataire. Sélectionnez une ou plusieurs stratégies d’accès conditionnel dans la liste déroulante et réduisez l’étendue du classeur en appliquant les filtres suivants : 

- **Intervalle de temps**

- **Utilisateur**

- **Applications**

- **Vue de données**

![Capture d’écran du volet Accès conditionnel permettant de sélectionner une stratégie d’accès conditionnel](./media/howto-use-azure-monitor-workbooks/access-insights.png)


Le résumé de l’impact indique le nombre d’utilisateurs ou de connexions pour lesquels les stratégies sélectionnées ont un résultat particulier. « Total » représente le nombre d’utilisateurs ou de connexions pour lesquels les stratégies sélectionnées ont été évaluées dans l’intervalle de temps sélectionné. Cliquez sur une vignette pour filtrer les données du classeur en fonction de ce type de résultat. 

![Capture d’écran des vignettes permettant de filtrer les résultats, notamment Total, Réussite et Échec](./media/howto-use-azure-monitor-workbooks/impact-summary.png)

Ce classeur montre également l’impact des stratégies sélectionnées, ventilé selon chacune des six conditions : 
- **État de l’appareil**
- **Plateforme de l’appareil**
- **Applications clientes**
- **Risque à la connexion**
- **Lieu**
- **Applications**

![Capture d’écran des détails du filtre Nombre total de connexions.](./media/howto-use-azure-monitor-workbooks/device-platform.png)

Vous pouvez également examiner des connexions individuelles, filtrées par les paramètres sélectionnés dans le classeur. Recherchez des utilisateurs individuels, triés par fréquence de connexion, et affichez les événements de connexion correspondants. 

![Capture d’écran des connexions individuelles consultables](./media/howto-use-azure-monitor-workbooks/filtered.png)

## <a name="sign-ins-by-grant-controls"></a>Connexions par contrôles d’octroi

Pour accéder au classeur afin d’identifier les connexions via des [contrôles d’octroi](../conditional-access/controls.md), accédez à la section **Accès conditionnel** , puis sélectionnez **Sign-ins by Grant Control** (Connexions par contrôle d’octroi). 

Ce classeur montre les tendances suivantes concernant les connexions désactivées :

- Exiger une authentification multifacteur
 
- Exiger des conditions d’utilisation

- Exiger une déclaration de confidentialité

- Autres


Vous pouvez filtrer chaque tendance selon les catégories suivantes :

- Plage temporelle

- Applications

- Utilisateurs

![Connexions par contrôles d’octroi](./media/howto-use-azure-monitor-workbooks/50.png)


Pour chaque tendance, vous obtenez une répartition par application et par protocole.

![Répartition des connexions récentes](./media/howto-use-azure-monitor-workbooks/51.png)




## <a name="sign-ins-failure-analysis"></a>Analyse des échecs de connexion

Utilisez le classeur **Analyse des échecs de connexion** pour résoudre les erreurs à l’aide des éléments suivants :

- Connexions
- Stratégies d’accès conditionnel
- Authentification héritée 


Pour accéder aux données relatives aux connexions via l’accès conditionnel, accédez à la session **Résoudre les problèmes** , puis sélectionnez **Sign-ins using Legacy Authentication** (Connexions à l’aide d’une authentification héritée). 

Ce classeur montre les tendances suivantes en matière de connexion :

- Toutes les connexions

- Succès

- Action en attente

- Échec


Vous pouvez filtrer chaque tendance selon les catégories suivantes :

- Plage temporelle

- Applications

- Utilisateurs

![Résolution des problèmes de connexion](./media/howto-use-azure-monitor-workbooks/52.png)


Pour vous aider à résoudre les problèmes de connexion, Azure Monitor vous fournit une répartition en fonction des catégories suivantes :

- Erreurs les plus fréquentes

    ![Résumé des erreurs les plus fréquentes](./media/howto-use-azure-monitor-workbooks/53.png)

- Connexions en attente d’une action utilisateur

    ![Résumé des connexions en attente d’une action utilisateur](./media/howto-use-azure-monitor-workbooks/54.png)






## <a name="next-steps"></a>Étapes suivantes

[Créer des rapports interactifs à l’aide de classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md).