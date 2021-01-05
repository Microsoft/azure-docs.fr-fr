---
title: Inscrire et analyser Azure SQL Database Managed Instance
description: Ce tutoriel explique comment analyser Azure SQL Database Managed Instance.
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: 6eb17537fd64b192f64c36b38bab57e11d751328
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400775"
---
# <a name="register-and-scan-an-azure-sql-database-managed-instance"></a>Inscrire et analyser une instance Azure SQL Database Managed Instance

Cet article explique comment inscrire une source de données Azure SQL Database Managed Instance dans Purview et configurer une analyse sur celle-ci.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

La source de données Azure SQL Database Managed Instance prend en charge les fonctionnalités suivantes :

- **Analyses complètes et incrémentielles** pour capturer les métadonnées et la classification dans Azure SQL Database Managed Instance

- **Traçabilité** entre les ressources de données pour les activités de copie et de flux de données ADF

### <a name="known-limitations"></a>Limitations connues

Azure Purview ne prend pas en charge l’analyse des [vues](https://docs.microsoft.com/sql/relational-databases/views/views?view=sql-server-ver15) dans Azure SQL Managed Instance.

## <a name="prerequisites"></a>Prérequis

- Créez un compte Purview si vous n’en avez pas déjà un.

- [Configurer un point de terminaison public dans Azure SQL Managed Instance](https://docs.microsoft.com/azure/azure-sql/managed-instance/public-endpoint-configure)
    > [!Note]
    > Votre organisation doit pouvoir autoriser un point de terminaison public car le **point de terminaison privé n’est pas encore pris en charge** par Purview. Si vous utilisez un point de terminaison privé, l’analyse n’aboutira pas.

### <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Authentification pour analyser Azure SQL Database Managed Instance. Si vous avez besoin de créer une nouvelle authentification, vous devez [autoriser l’accès aux bases de données à SQL Database Managed Instance](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage). Il existe actuellement trois méthodes d’authentification prises en charge par Purview :

- Authentification SQL
- Principal de service
- Identité managée

#### <a name="sql-authentication"></a>Authentification SQL

> [!Note]
> Seule la connexion principale au niveau du serveur (créée par le processus de configuration) ou les membres du rôle de base de données `loginmanager` dans la base de données master peuvent créer des connexions. Cela nécessite environ **15 minutes** une fois l’autorisation accordée. Le compte Purview doit disposer des autorisations appropriées pour pouvoir analyser la ou les ressources.

Vous pouvez suivre les instructions fournies dans [CREATE LOGIN](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-current&preserve-view=true#examples-1) pour créer une connexion pour Azure SQL Database Managed Instance si vous n’en avez pas. Vous aurez besoin d’un **nom d’utilisateur** et d’un **mot de passe** pour les étapes suivantes.

1. Accédez à votre coffre de clés dans le portail Azure.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer** et entrez le **nom** et la **valeur** comme *mot de passe* de votre instance Azure SQL Database Managed Instance.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez de nouvelles informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du **nom d’utilisateur** et du **mot de passe** pour configurer votre analyse.

#### <a name="service-principal-and-managed-identity"></a>Principal de service et identité managée

Il existe plusieurs étapes pour autoriser Purview à utiliser le principal de service pour analyser votre instance Azure SQL Database Managed Instance.

##### <a name="create-or-use-an-existing-service-principal"></a>Créer ou utiliser un principal de service existant

> [!Note]
> Ignorez cette étape si vous utilisez une **identité managée**.

Vous pouvez utiliser un principal de service existant ou en créer un nouveau. 

> [!Note]
> Si vous devez créer un principal de service, procédez comme suit :
> 1. Accédez au [portail Azure](https://portal.azure.com).
> 1. Dans le menu de gauche, sélectionnez **Azure Active Directory**.
> 1. Sélectionnez **Inscriptions d’applications**.
> 1. Sélectionnez **+ Nouvelle inscription d’application**.
> 1. Entrez un nom pour l’**application** (nom du principal de service).
> 1. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.
> 1. Pour l’URI de redirection, sélectionnez **Web** et entrez l’URL de votre choix. Il n’est pas nécessaire qu’elle soit réelle ni qu’elle fonctionne.
> 1. Sélectionnez ensuite **Inscription**.

##### <a name="configure-azure-ad-authentication-in-the-database-account"></a>Configurer l’authentification Azure AD dans le compte de base de données

Le principal de service ou l’identité managée doivent avoir l’autorisation d’obtenir des métadonnées pour la base de données, les schémas et les tables. Ils doivent également être en mesure d’interroger les tables à échantillonner pour la classification.
- [Configurer et gérer l’authentification Azure AD avec Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure)
- Créez un utilisateur Azure AD dans Azure SQL Database Managed Instance en suivant les conditions préalables et le tutoriel [Créer des utilisateurs à relation contenant-contenu mappés à des identités Azure AD](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure?tabs=azure-powershell#create-contained-users-mapped-to-azure-ad-identities).
- Affectez l’autorisation `db_owner` (**recommandée**) à l’identité.

##### <a name="add-service-principal-to-key-vault-and-purviews-credential"></a>Ajouter le principal de service aux informations d’identification de Purview et du coffre de clés

> [!Note]
> Si vous envisagez d’utiliser une **identité managée**, vous pouvez ignorer cette étape, car l’identité par défaut de Purview figure déjà dans **Purview-MSI**.

Il est nécessaire d’obtenir l’ID d’application et le secret du principal de service :

1. Accédez à votre principal de service dans le [portail Azure](https://portal.azure.com).
1. Copiez les valeurs **ID d’application (client)** dans **Vue d’ensemble** et **Secret client** dans **Certificats et secrets**.
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer** et entrez le **nom** de votre choix et la **valeur** comme **secret client** de votre principal de service.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez de nouvelles informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse.

### <a name="firewall-settings"></a>Paramètres du pare-feu

Votre serveur de base de données doit autoriser l’activation des connexions Azure. Cela permettra à Azure Purview d’atteindre et de connecter le serveur. Vous pouvez suivre le guide pratique pour des [connexions à partir d’Azure](../azure-sql/database/firewall-configure.md#connections-from-inside-azure).

1. Accédez à votre compte de base de données.
1. Dans la page **Vue d’ensemble**, sélectionnez le nom du serveur.
1. Sélectionnez **Sécurité > Pare-feux et réseaux virtuels**.
1. Sélectionnez **Oui** pour **Autoriser les services et les ressources Azure à accéder à ce serveur**.

    :::image type="content" source="media/register-scan-azure-sql-database/sql-firewall.png" alt-text="Options d’inscription de sources" border="true":::
    
> [!Note]
> Actuellement, Azure Purview ne prend pas en charge la configuration de réseau virtuel. Par conséquent, vous ne pouvez pas utiliser de paramètres de pare-feu basés sur IP.

## <a name="register-an-azure-sql-database-managed-instance-data-source"></a>Inscrire une source de données Azure SQL Database Managed Instance

1. Accédez à votre compte Purview.

1. Sélectionnez **Sources** dans le volet de navigation de gauche.

1. Sélectionnez **Inscrire**.

1. Sélectionnez **Azure SQL Database Managed Instance** puis **Continuer**.

    :::image type="content" source="media/register-scan-azure-sql-database-managed-instance/set-up-the-sql-data-source.png" alt-text="Configurer la source de données SQL":::

1. Sélectionnez **Entrer manuellement**.

1. Spécifiez le **nom de domaine complet du point de terminaison public** et le **numéro de port**. Ensuite, sélectionnez **Terminer** pour inscrire la source de données.

    :::image type="content" source="media/register-scan-azure-sql-database-managed-instance/add-azure-sql-database-managed-instance.png" alt-text="Ajouter Azure SQL Database Managed Instance":::

    Par exemple, `foobar.public.123.database.windows.net,3342`

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

> [!NOTE]
> La suppression de votre analyse ne supprime pas vos ressources des analyses Azure SQL Database Managed Instance précédentes.

## <a name="next-steps"></a>Étapes suivantes

- [Parcourir le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Effectuer une recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
