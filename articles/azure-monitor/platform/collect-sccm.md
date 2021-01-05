---
title: Connexion de Configuration Manager à Azure Monitor | Microsoft Docs
description: Cet article décrit comment connecter Configuration Manager à l’espace de travail dans Azure Monitor et commencer à analyser des données.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/28/2019
ms.openlocfilehash: 20d55de7ef0f7b7f49b922409f8cb61874fae0f6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008971"
---
# <a name="connect-configuration-manager-to-azure-monitor"></a>Connexion de Configuration Manager à Azure Monitor
Vous pouvez connecter votre environnement Microsoft Endpoint Configuration Manager à Azure Monitor pour synchroniser les données de regroupement d’appareils et faire référence à ces regroupements dans Azure Monitor et Azure Automation.  

## <a name="prerequisites"></a>Conditions préalables requises

Azure Monitor prend en charge Configuration Manager (Current Branch), version 1606 et ultérieure.

>[!NOTE]
>La fonctionnalité de connexion de Configuration Manager à un espace de travail Log Analytics est facultative et n’est pas activée par défaut. Vous devez activer cette fonctionnalité avant de l’utiliser. Pour plus d’informations, consultez [Activation de fonctionnalités facultatives de mises à jour](/configmgr/core/servers/manage/install-in-console-updates#bkmk_options).

## <a name="configuration-overview"></a>Présentation de la configuration

Les étapes suivantes récapitulent les étapes de configuration de l’intégration de Configuration Manager avec Azure Monitor.  

1. Dans Azure Active Directory, inscrivez Configuration Manager en tant qu’application web et/ou application API web, et assurez-vous de disposer de l’ID et de la clé secrète client résultant de l’inscription à partir d’Azure Active Directory. Pour plus d’informations sur cette étape, consultez [Utiliser le portail pour créer une application et un principal du service Active Directory pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

2. Dans Azure Active Directory, [attribuez à Configuration Manager (l’application web inscrite) l’autorisation d’accéder à Azure Monitor](#grant-configuration-manager-with-permissions-to-log-analytics).

3. Dans Configuration Manager, ajoutez une connexion à l’aide de l’Assistant **Services Azure**.

4. [Téléchargez et installez l’agent Log Analytics pour Windows](#download-and-install-the-agent) sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager. L’agent envoie les données de Configuration Manager vers l’espace de travail Log Analytics dans Azure Monitor.

5. Dans Azure Monitor, [importez les collections de Configuration Manager](#import-collections) en tant que groupes d’ordinateurs.

6. Dans Azure Monitor, consultez les données de Configuration Manager en tant que [groupes d’ordinateurs](computer-groups.md).

## <a name="grant-configuration-manager-with-permissions-to-log-analytics"></a>Accorder à Configuration Manager les autorisations d’accès à Log Analytics

Dans la procédure suivante, vous attribuez le rôle *Contributeur* dans votre espace de travail Log Analytics à l’application et au principal de service AD que vous avez créés précédemment pour Configuration Manager. Si vous ne disposez pas déjà d’un espace de travail, consultez [Créer un espace de travail dans Azure Monitor](../learn/quick-create-workspace.md) avant de continuer. Cela permet à Configuration Manager de s’authentifier et de se connecter à votre espace de travail Log Analytics.  

> [!NOTE]
> Vous devez spécifier des autorisations dans l’espace de travail Log Analytics pour Configuration Manager. Sinon, vous recevez un message d’erreur quand vous utilisez l’Assistant Configuration dans Configuration Manager.
>

1. Dans le portail Azure, cliquez sur **Tous les services** en haut à gauche. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Log Analytics**.

2. Dans votre liste d’espaces de travail Log Analytics, sélectionnez l’espace de travail à modifier.

3. Dans le volet gauche, sélectionnez **Contrôle d’accès (IAM)** .

4. Dans la page Contrôle d'accès (IAM), cliquez sur **Ajouter une attribution de rôle** pour afficher le volet **Ajouter une attribution de rôle**.

5. Dans le volet **Ajouter une attribution de rôle**, ouvrez la liste déroulante **Rôle** et sélectionnez le rôle **Contributeur**.  

6. Dans la liste déroulante **Attribuer l’accès à**, sélectionnez l’application Configuration Manager créée précédemment dans AD, puis cliquez sur **OK**.  

## <a name="download-and-install-the-agent"></a>Téléchargement et installation de l’agent

Consultez l’article [Connecter des ordinateurs Windows au service Azure Monitor dans Azure](agent-windows.md) afin de mieux comprendre les méthodes disponibles pour l’installation de l’agent Log Analytics pour Windows sur l’ordinateur qui héberge le rôle de système de site de point de connexion de service de Configuration Manager.  

## <a name="connect-configuration-manager-to-log-analytics-workspace"></a>Connecter Configuration Manager à l’espace de travail Log Analytics

>[!NOTE]
> Pour que vous puissiez ajouter une connexion Log Analytics, il faut que votre environnement Configuration Manager ait un [point de connexion de service](/configmgr/core/servers/deploy/configure/about-the-service-connection-point) configuré pour le mode en ligne.

> [!NOTE]
> Vous devez connecter le site de niveau supérieur de votre hiérarchie à Azure Monitor. Si vous connectez un site principal autonome à Azure Monitor, puis que vous ajoutez un site d’administration centrale à votre environnement, vous devez supprimer la connexion et la recréer au sein de la nouvelle hiérarchie.

1. Dans l’espace de travail **Administration** de Configuration Manager, sélectionnez **Services cloud**, puis **Services Azure**. 

2. Cliquez avec le bouton droit sur **Services Azure**, puis sélectionnez **Configurer les services Azure**. La page **Configurer les services Azure** s’affiche. 
   
3. Dans l’écran **Général**, vérifiez que vous avez effectué les actions suivantes et que vous disposez des détails de chaque élément, puis sélectionnez **Suivant**.

4. Dans la page Services Azure de l’Assistant Services Azure :

    1. Spécifiez un **Nom** pour l’objet dans Configuration Manager.
    2. Spécifiez une **description** facultative pour vous aider à identifier le service.
    3. Sélectionnez le **connecteur OMS** du service Azure.

    >[!NOTE]
    >OMS est désormais appelée Log Analytics, qui est une fonctionnalité d’Azure Monitor.

5. Sélectionnez **Suivant** pour passer à la page des propriétés de l’application Azure de l’Assistant Services Azure.

6. Dans la page **Application** de l’Assistant Services Azure, sélectionnez l’environnement Azure dans la liste, puis cliquez sur **Importer**.

7. Dans la page **Importer des applications**, spécifiez les informations suivantes :

    1. Spécifiez le **nom du locataire Azure AD** pour l’application.

    2. Spécifiez le locataire Azure AD en guise d’**ID de locataire Azure AD**. Vous pouvez trouver ces informations dans la page **Propriétés** d’Azure Active Directory. 

    3. Spécifiez le nom de l’application en guise de **Nom d’application**.

    4. Pour **ID client**, spécifiez l’ID d’application de l’application Azure AD créée.

    5. Pour **Clé secrète**, spécifiez la clé secrète client de l’application Azure AD créée.

    6. Pour **Expiration de la clé secrète**, spécifiez la date d’expiration de votre clé.

    7. Pour **URI ID d’application**, spécifiez l’URI ID d’application de l’application Azure AD créée.

    8. Sélectionnez **Vérifier** ; à droite, les résultats doivent indiquer **Vérification effectuée**.

8. Dans la page **Configuration**, passez en revue les informations pour vérifier que les champs **Abonnements Azure**, **Groupe de ressources Azure** et **Espace de travail Operations Management Suite** sont préremplis, indiquant que l’application Azure AD dispose des autorisations suffisantes dans le groupe de ressources. Si les champs sont vides, cela signifie que votre application ne dispose pas des droits requis. Sélectionnez les regroupements d’appareils à collecter et à transférer à l’espace de travail, puis sélectionnez **Ajouter**.

9. Passez en revue les options de la page **Confirmer les paramètres**, puis sélectionnez **Suivant** pour commencer à créer et à configurer la connexion.

10. Une fois la configuration terminée, la page **Fin** s’affiche. Sélectionnez **Fermer**. 

Après avoir lié Configuration Manager à Azure Monitor, vous pouvez ajouter ou supprimer des regroupements, et afficher les propriétés de la connexion.

## <a name="update-log-analytics-workspace-connection-properties"></a>Mettre à jour les propriétés de connexion de l’espace de travail Log Analytics

En cas d’expiration ou de perte de mot de passe ou de clé secrète du client, vous devez mettre à jour manuellement les propriétés de connexion Log Analytics.

1. Dans l’espace de travail **Administration** de Configuration Manager, accédez à **Services cloud**, puis sélectionnez **Connecteur OMS** pour ouvrir la page **Propriétés de connexion OMS**.
2. Dans cette page, cliquez sur l’onglet **Azure Active Directory** pour afficher vos informations **Client**, **ID client** et **Expiration de clé secrète client**. **Vérifiez** votre **Clé secrète client** pour voir si elle a expiré.

## <a name="import-collections"></a>Importer des regroupements

Après l’ajout d’une connexion Log Analytics à Configuration Manager et l’installation de l’agent sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager, l’étape suivante consiste à importer des regroupements de Configuration Manager dans Azure Monitor en tant que groupes d’ordinateurs.

Après la configuration initiale de l’importation des regroupements d’appareils à partir de votre hiérarchie, les informations du regroupement sont récupérées toutes les 3 heures dans l’optique de maintenir l’appartenance à jour. Vous pouvez désactiver cette fonction à tout moment.

1. Dans le portail Azure, cliquez sur **Tous les services** en haut à gauche. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Espaces de travail Log Analytics**.
2. Dans la liste des espaces de travail Log Analytics, sélectionnez l’espace de travail avec lequel Configuration Manager est inscrit.  
3. Sélectionnez **Paramètres avancés**.
4. Sélectionnez **Groupes d’ordinateurs**, puis **SCCM**.  
5. Sélectionnez **Importer les appartenances aux regroupements Configuration Manager**, puis cliquez sur **Enregistrer**.  
   
    ![Capture d’écran des paramètres avancés des groupes d’ordinateurs pour SCCM, contenant l’option Importer les appartenances aux regroupements Configuration Manager.](./media/collect-sccm/sccm-computer-groups01.png)

## <a name="view-data-from-configuration-manager"></a>Afficher les données de Configuration Manager

Après l’ajout d’une connexion Log Analytics à Configuration Manager et l’installation de l’agent sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager, les données de l’agent sont envoyées à l’espace de travail Log Analytics dans Azure Monitor. Dans Azure Monitor, vos regroupements Configuration Manager apparaissent sous la forme de [groupes d’ordinateurs](./computer-groups.md). Vous pouvez afficher les groupes à partir de la page **Configuration Manager**, sous **Paramètres\Groupes d’ordinateurs**.

Une fois les regroupements importés, vous pouvez voir combien d’ordinateurs avec des appartenances à des regroupements ont été détectés. Vous pouvez également voir le nombre de regroupements importés.

![Capture d’écran des paramètres avancés des groupes d’ordinateurs pour SCCM, montrant l’option Importer les appartenances aux regroupements Configuration Manager sélectionnée.](./media/collect-sccm/sccm-computer-groups02.png)

Quand vous cliquez sur l’un deux, l’éditeur de requête de journal s’ouvre, affichant tous les groupes importés ou tous les ordinateurs appartenant à chaque groupe. À l’aide de [Recherche dans les journaux](../log-query/log-query-overview.md), vous pouvez effectuer une analyse approfondie des données d’appartenance au regroupement.

## <a name="next-steps"></a>Étapes suivantes

Utilisez [Recherche de journal](../log-query/log-query-overview.md) pour afficher des informations détaillées sur vos données Configuration Manager.

