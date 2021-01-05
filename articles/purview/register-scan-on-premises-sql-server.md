---
title: Inscrire et analyser un serveur SQL local
description: Ce tutoriel explique comment analyser un serveur SQL local à l’aide d’un runtime d’intégration auto-hébergé.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 09/18/2020
ms.openlocfilehash: a807c76a08bf075973cd0f358c67b61b10b40ba0
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550240"
---
# <a name="register-and-scan-an-on-premises-sql-server"></a>Inscrire et analyser un serveur SQL local

Cet article explique comment inscrire une source de données de serveur SQL dans Purview et configurer une analyse sur celle-ci.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

La source de données locale de serveur SQL prend en charge les fonctionnalités suivantes :

- **Analyses complètes et incrémentielles** pour capturer des métadonnées et une classification dans un réseau local ou un serveur SQL installé sur une machine virtuelle Azure.

- **Traçabilité** entre les ressources de données pour les activités de copie/dataflow d’ADF.

La source de données locale du serveur SQL prend en charge :

- toutes les versions, de SQL Server 2019 à SQL Server 2000 ;

- méthode d’authentification Authentification SQL

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Créer un compte Azure Purview](create-catalog-portal.md).

- Configurez un [runtime d’intégration auto-hébergé](manage-integration-runtimes.md) pour analyser la source de données.

## <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Il n’existe qu’une seule façon de configurer l’authentification pour un serveur SQL local :

- Authentification SQL

### <a name="sql-authentication"></a>Authentification SQL

L’identité SQL doit avoir accès à la base de données primaire. Il s’agit de l’emplacement où `sys.databases` est stocké. Le scanneur Purview doit énumérer `sys.databases` pour rechercher toutes les instances de base de données SQL dans le serveur.

#### <a name="using-an-existing-server-administrator"></a>Utilisation d’un administrateur de serveur existant

Si vous envisagez d’utiliser un utilisateur administrateur de serveur (sa) existant pour analyser votre serveur SQL local, vérifiez les points suivants :

1. `sa` n’est pas le type d’authentification Windows.

2. L’utilisateur au niveau du serveur que vous envisagez d’utiliser doit avoir les rôles de serveur public et sysadmin. Vous pouvez vérifier cela en accédant à SQL Server Management Studio (SSMS), en vous connectant au serveur, en accédant à la sécurité, en sélectionnant la connexion que vous prévoyez d’utiliser, en cliquant avec le bouton droit sur **Propriétés**, puis en sélectionnant **Rôles de serveur**.

   :::image type="content" source="media/register-scan-on-premises-sql-server/server-level-login.png" alt-text="Connexion au niveau du serveur.":::

3. Les bases de données sont mappées à un utilisateur qui dispose au moins d’un accès de niveau db_datareader sur chaque base de données.

   :::image type="content" source="media/register-scan-on-premises-sql-server/user-mapping-sa.png" alt-text="mappage de l’utilisateur pour sa.":::

#### <a name="creating-a-new-login-and-user"></a>Création d’une connexion et d’un utilisateur

Si vous souhaitez créer une connexion et un utilisateur pour pouvoir analyser votre serveur SQL, procédez comme suit :

1. Accédez à SQL Server Management Studio (SSMS), connectez-vous au serveur, accédez à la sécurité, cliquez avec le bouton droit sur connexion, puis créez une connexion. Veillez à sélectionner l’authentification SQL.

   :::image type="content" source="media/register-scan-on-premises-sql-server/create-new-login-user.png" alt-text="Créer une connexion et un utilisateur.":::

2. Sélectionnez les rôles de serveurs dans le volet de navigation de gauche, puis sélectionnez public et sysadmin.

3. Sélectionnez un mappage d’utilisateur dans le volet de navigation de gauche, puis sélectionnez toutes les bases de données dans le mappage.

   :::image type="content" source="media/register-scan-on-premises-sql-server/user-mapping.png" alt-text="mappage d’utilisateur.":::

4. Cliquez sur OK pour enregistrer.

5. Accédez de nouveau à l’utilisateur que vous avez créé en cliquant avec le bouton droit, puis en sélectionnant **Propriétés**. Entrez un nouveau mot de passe et confirmez-le. Sélectionnez « Spécifier l’ancien mot de passe », puis entrez l’ancien mot de passe. **Vous devez modifier votre mot de passe dès que vous créez une connexion.**

   :::image type="content" source="media/register-scan-on-premises-sql-server/change-password.png" alt-text="modifier un mot de passe.":::

#### <a name="storing-your-sql-login-password-in-a-key-vault-and-creating-a-credential-in-purview"></a>Stockage de votre mot de passe de connexion SQL dans un coffre de clés et création d’informations d’identification dans Purview

1. Accédez à votre coffre de clés dans le portail Azure.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer**, puis entrez le **Nom** et la **Valeur** en tant que *mot de passe* de votre connexion au serveur SQL.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez de nouvelles informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du **nom d’utilisateur** et du **mot de passe** pour configurer votre analyse.

## <a name="register-a-sql-server-data-source"></a>Inscrire une source de données de serveur SQL

1. Accédez à votre compte Purview.

1. Sous sources et analyse dans le volet de navigation de gauche, sélectionnez les **runtimes d’intégration**. Assurez-vous qu’un runtime d’intégration auto-hébergé est configuré. Si ce n’est pas le cas, suivez les étapes mentionnées [ici](manage-integration-runtimes.md) afin de créer un runtime d’intégration auto-hébergé pour les analyses sur une machine virtuelle locale ou Azure qui a accès à votre réseau local.

1. Sélectionnez **Sources** dans le volet de navigation de gauche.

1. Sélectionnez **Inscrire**.

1. Sélectionnez **SQL Server**, puis **Continuer**.

   :::image type="content" source="media/register-scan-on-premises-sql-server/set-up-sql-data-source.png" alt-text="Configurer la source de données SQL.":::

5. Fournissez un nom convivial et un point de terminaison de serveur, puis sélectionnez **Terminer** pour inscrire la source de données. Si, par exemple, votre nom de domaine complet SQL Server est **foobar.database.windows.net**, entrez *foobar* en tant que point de terminaison de serveur.

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
