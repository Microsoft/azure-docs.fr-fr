---
title: Comment analyser Azure Synapse Analytics
description: Ce guide pratique explique en détail comment analyser Azure Synapse Analytics.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/22/2020
ms.openlocfilehash: a0b0dc8c29bcdb51f7b348dd62e3d27796819a7d
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550246"
---
# <a name="register-and-scan-azure-synapse-analytics"></a>Inscrire et analyser Azure Synapse Analytics

Cet article explique comment inscrire et analyser une instance d’Azure Synapse Analytics (anciennement SQL DW) dans Purview.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Azure Synapse Analytics (anciennement SQL DW) prend en charge les analyses complètes et incrémentielles pour capturer les métadonnées et le schéma. Ces analyses permettent également de classer les données automatiquement selon des règles de classification système et personnalisées.

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Créer un compte Azure Purview](create-catalog-portal.md).
- Vous devez être administrateur de la source de données Azure Purview.
- Accès réseau entre le compte Purview et Azure Synapse Analytics.
 
## <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Il existe trois façons de configurer l’authentification pour Stockage Blob Azure :

- Identité managée
- Authentification SQL
- Principal de service

    > [!Note]
    > Seule la connexion principale au niveau du serveur (créée par le processus de configuration) ou les membres du rôle de base de données `loginmanager` dans la base de données master peuvent créer des connexions. Cela nécessite environ 15 minutes une fois l’autorisation accordée. Le compte Purview doit disposer des autorisations appropriées pour pouvoir analyser la ou les ressources.

### <a name="managed-identity-recommended"></a>Identité managée (recommandé) 
   
Votre compte Purview possède sa propre identité managée, qui est fondamentalement votre nom Purview lorsque vous l’avez créé. Vous devez créer un utilisateur Azure AD dans Azure Synapse Analytics (anciennement SQL DW) avec le nom d’identité managée exact de Purview en suivant les conditions préalables et le didacticiel sur [Créer des utilisateurs Azure AD avec des applications Azure AD](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-service-principal-tutorial).

Exemple de syntaxe SQL pour créer l’utilisateur et accorder l’autorisation :

```sql
CREATE USER [PurviewManagedIdentity] FROM EXTERNAL PROVIDER
GO

EXEC sp_addrolemember 'db_owner', [PurviewManagedIdentity]
GO
```

L’authentification doit avoir l’autorisation d’obtenir des métadonnées pour la base de données, les schémas et les tables. Elle doit également être en mesure d’interroger les tables à échantillonner pour la classification. Il est recommandé d’attribuer l’autorisation `db_owner` à l’identité.

### <a name="service-principal"></a>Principal de service

Si vous souhaitez appliquer l’authentification par principal de service pour les analyses, vous pouvez utiliser un principal de service existant ou en créer un. 

> [!Note]
> Si vous devez créer un principal de service, procédez comme suit :
> 1. Accédez au [portail Azure](https://portal.azure.com).
> 1. Sélectionnez **Azure Active Directory** dans le menu de gauche.
> 1. Sélectionnez **Inscriptions d’applications**.
> 1. Sélectionnez **+ Nouvelle inscription d’application**.
> 1. Entrez un nom pour **l’application** (nom du principal de service).
> 1. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.
> 1. Pour l’URI de redirection, sélectionnez **Web** et entrez l’URL de votre choix. Il n’est pas nécessaire qu’elle soit réelle ni qu’elle fonctionne.
> 1. Sélectionnez ensuite **Inscription**.

Il est nécessaire de récupérer l’ID d’application et le secret du principal de service :

1. Accédez à votre principal de service sur le [Portail Azure](https://portal.azure.com).
1. Copiez les valeurs **ID d’application (client)** dans **Vue d’ensemble** et **Clé secrète client** dans **Certificats et secrets**.
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer**, puis entrez le **Nom** de votre choix et la **Valeur** comme **Clé secrète client** de votre principal de service.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse. 

#### <a name="granting-the-service-principal-access-to-your-azure-synapse-analytics-formerly-sql-dw"></a>Octroi de l’accès du principal de service à votre instance Azure Synapse Analytics (anciennement SQL DW)

En outre, vous devez créer un utilisateur Azure AD dans Azure Synapse Analytics en suivant les conditions préalables et le didacticiel sur [Créer des utilisateurs Azure AD avec des applications Azure AD](https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-service-principal-tutorial). Exemple de syntaxe SQL pour créer l’utilisateur et accorder l’autorisation :

```sql
CREATE USER [ServicePrincipalName] FROM EXTERNAL PROVIDER
GO

EXEC sp_addrolemember 'db_owner', [ServicePrincipalName]
GO
```

> [!Note]
> Purview a besoin de l’**ID d’application (client)** et du **secret client** pour effectuer l’analyse.

### <a name="sql-authentication"></a>Authentification SQL

Vous pouvez suivre les instructions dans [CREATE LOGIN](https://docs.microsoft.com/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-current&preserve-view=true#examples-1) pour créer une connexion pour Azure Synapse Analytics (anciennement SQL DW) si vous n’en avez pas déjà une.

Lorsque la méthode d’authentification sélectionnée est **Authentification SQL**, vous devez vous procurer votre mot de passe et le stocker dans le coffre de clés :

1. Obtenir le mot de passe pour votre connexion SQL
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer** et entrez le **Nom** et la **Valeur** comme *mot de passe* pour votre connexion SQL.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide de la clé pour configurer votre analyse.

## <a name="register-an-azure-synapse-analytics-instance-formerly-sql-dw"></a>Inscrire une instance Azure Synapse Analytics (anciennement SQL DW)

Pour inscrire un nouveau serveur Azure Synapse Analytics dans votre Data Catalog, procédez comme suit :

1. Accédez à votre compte Purview.
1. Sélectionnez **Sources** dans le volet de navigation de gauche.
1. Sélectionnez **Inscrire**.
1. Sur **Inscrire des sources**, sélectionnez **Azure Synapse Analytics (anciennement SQL DW)**
1. Sélectionnez **Continue** (Continuer)

Sur l’écran **Inscrire des sources (Azure Synapse Analytics)** , procédez comme suit :

1. Entrez le **Nom** sous lequel la source de données apparaîtra dans le catalogue.
1. Choisissez la façon dont vous souhaitez pointer vers le compte de stockage de votre choix :
   1. Sélectionnez **À partir de l’abonnement Azure**, puis sélectionnez l’abonnement approprié dans la zone de liste déroulante **Abonnement Azure** et le serveur approprié dans la zone de liste déroulante **Nom du serveur**.
   1. Vous pouvez également sélectionner **Entrer manuellement** et entrer le **Nom du serveur**.
1. Sélectionnez **Terminer** pour inscrire la source de données.

:::image type="content" source="media/register-scan-azure-synapse-analytics/register-sources.png" alt-text="Options d’inscription des sources" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)

