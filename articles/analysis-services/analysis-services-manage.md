---
title: Gérer Azure Analysis Services | Microsoft Docs
description: Cet article décrit les outils utilisés pour gérer les tâches d’administration et de gestion pour un serveur Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 10/28/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 91ffcac98b2b919a8fc131d235e699aad4fa215d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93078898"
---
# <a name="manage-analysis-services"></a>Gérer Analysis Services
Une fois que vous avez créé un serveur Analysis Services dans Azure, vous devrez peut-être effectuer certaines tâches d’administration et de gestion immédiatement ou un peu plus tard. Par exemple, exécuter le traitement d’actualisation des données, contrôler qui peut accéder aux modèles sur votre serveur ou surveiller l’intégrité de votre serveur. Certaines tâches de gestion ne peuvent être effectuées que dans le portail Azure, d’autres dans SQL Server Management Studio (SSMS), et certaines tâches encore peuvent être effectuées dans les deux.

## <a name="azure-portal"></a>Portail Azure
Le [portail Azure](https://portal.azure.com/) est l’endroit où vous pouvez créer et supprimer des serveurs, surveiller les ressources du serveur, modifier la taille et gérer qui a accès à vos serveurs.  Si vous rencontrez des problèmes, vous pouvez également envoyer une demande de support.

![Capture d’écran montrant le portail Azure sur lequel vous pouvez créer et supprimer des serveurs, surveiller les ressources des serveurs, modifier leur taille et gérer qui a accès à vos serveurs.](./media/analysis-services-manage/aas-manage-portal.png)

## <a name="sql-server-management-studio"></a>SQL Server Management Studio
La connexion à votre serveur dans Azure revient à vous connecter à une instance de serveur dans votre entreprise. À partir de SSMS, vous pouvez effectuer la plupart des tâches identiques comme traiter des données ou créer un script de traitement, gérer des rôles et utiliser PowerShell.
  
![SQL Server Management Studio](./media/analysis-services-manage/aas-manage-ssms.png)

### <a name="download-and-install-ssms"></a>Téléchargement et installation de SSMS
Pour obtenir toutes les dernières fonctionnalités et bénéficier d’une expérience parfaitement fluide lors de la connexion à votre serveur Azure Analysis Services, veillez à utiliser la version la plus récente de SSMS. 

[Téléchargez SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).


### <a name="to-connect-with-ssms"></a>Pour se connecter avec SSMS
 Quand vous utilisez SSMS, avant de vous connecter à votre serveur pour la première fois, vérifiez que votre nom d’utilisateur figure dans le groupe Administrateurs Analysis Services. Pour plus d’informations, consultez [Administrateurs de serveur et utilisateurs de base de données](#server-administrators-and-database-users) plus loin dans cet article.

1. Avant de vous connecter, vous devez obtenir le nom du serveur. Dans **Portail Azure** > Serveur > **Présentation** > **Nom du serveur** , copiez le nom du serveur.
   
    ![Obtenir le nom du serveur dans Azure](./media/analysis-services-deploy/aas-deploy-get-server-name.png)
2. Dans SSMS > **Explorateur d’objets** , cliquez sur **Se connecter** > **Analysis Services**.
3. Dans la boîte de dialogue **Se connecter au serveur** , copiez le nom du serveur puis, dans **Authentification** , choisissez l’un des types d’authentification suivants :   
    > [!NOTE]
    > Pour le type d’authentification, **Active Directory - Authentification universelle avec prise en charge de MFA** est recommandé.

    > [!NOTE]
    > Si vous vous connectez avec un compte Microsoft, Live ID, Yahoo, Gmail, etc., laissez le champ de mot de passe vide. Vous êtes invité à saisir un mot de passe après avoir cliqué sur Se connecter.

    **Authentification Windows** pour utiliser vos informations de domaine, nom d’utilisateur et mot de passe Windows.

    **Authentification par mot de passe Active Directory** pour utiliser un compte professionnel. Par exemple, lors d’une connexion à partir d’un ordinateur non joint à un domaine.

    **Active Directory - Authentification universelle avec prise en charge de MFA** pour utiliser une [authentification non interactive ou multifacteur](../azure-sql/database/authentication-mfa-ssms-overview.md). 
   
    ![Se connecter dans SSMS](./media/analysis-services-manage/aas-manage-connect-ssms.png)

## <a name="server-administrators-and-database-users"></a>Administrateurs de serveur et utilisateurs de base de données
Dans Azure Analysis Services, il existe deux types d’utilisateur : les administrateurs de serveur et les utilisateurs de base de données. Les deux types d’utilisateur doivent exister dans Azure Active Directory et être spécifiés par adresse de messagerie d’organisation ou UPN. Pour en savoir plus, consultez [Authentification et autorisations utilisateur](analysis-services-manage-users.md).


## <a name="troubleshooting-connection-problems"></a>Résolution des problèmes de connexion
Si vous rencontrez des problèmes lors d’une connexion à l’aide de SSMS, vous devrez peut-être effacer le cache de connexion. Rien n’est mis en cache sur le disque. Pour effacer le cache, fermez et redémarrez le processus de connexion. 

## <a name="next-steps"></a>Étapes suivantes
Si vous n’avez pas déjà déployé un modèle tabulaire sur votre nouveau serveur, c’est le moment de le faire. Pour en savoir plus, voir [Déployer sur Azure Analysis Services](analysis-services-deploy.md).

Si vous avez déployé un modèle sur votre serveur, vous êtes prêt à vous connecter à celui-ci à l’aide d’un client ou d’un navigateur. Pour en savoir plus, voir [Obtenir les données du serveur Azure Analysis Services](analysis-services-connect.md).