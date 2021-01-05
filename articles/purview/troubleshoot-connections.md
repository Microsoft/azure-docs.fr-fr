---
title: Résoudre les problèmes liés à vos connexions dans Azure Purview
description: Cet article explique les étapes à suivre pour résoudre les problèmes liés à vos connexions dans Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: f76a05757f86308785d4ca678675b87b3fa9d63e
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550227"
---
# <a name="troubleshoot-your-connections-in-azure-purview"></a>Résoudre les problèmes liés à vos connexions dans Azure Purview

Cet article explique comment résoudre les erreurs de connexion lors de la configuration d’analyses sur des sources de données dans Azure Purview.

## <a name="permission-the-credential-on-the-data-source"></a>Autoriser les informations d’identification sur la source de données

Si vous utilisez une identité managée ou un principal de service comme méthode d’authentification pour les analyses, vous devez autoriser ces identités à avoir accès à votre source de données.

Il existe des instructions spécifiques pour chaque type de source :

- [Stockage Blob Azure](register-scan-azure-blob-storage-source.md#setting-up-authentication-for-a-scan)
- [Azure Cosmos DB](register-scan-azure-cosmos-database.md#setting-up-authentication-for-a-scan)
- [Explorateur de données Azure](register-scan-azure-data-explorer.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen1](register-scan-adls-gen1.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen2](register-scan-adls-gen2.md#setting-up-authentication-for-a-scan)
- [Azure SQL Database](register-scan-azure-sql-database.md)
- [Azure SQL Database Managed Instance](register-scan-azure-sql-database-managed-instance.md#setting-up-authentication-for-a-scan)
- [Azure Synapse Analytics](register-scan-azure-synapse-analytics.md#setting-up-authentication-for-a-scan)
- [SQL Server](register-scan-on-premises-sql-server.md#setting-up-authentication-for-a-scan)
- [Power BI](register-scan-power-bi-tenant.md)

## <a name="storing-your-credential-in-your-key-vault-and-using-the-right-secret-name-and-version"></a>Stockage de vos informations d’identification dans votre coffre de clés et utilisation du nom et de la version appropriés pour le secret

Vous devez également stocker vos informations d’identification dans votre instance Azure Key Vault et utiliser le nom et la version appropriés pour le secret.

Pour ce faire, procédez comme suit :

1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres** > **Secrets**.
1. Sélectionnez le secret que vous utilisez pour vous authentifier auprès de votre source de données à des fins d’analyse.
1. Sélectionnez la version que vous prévoyez d’utiliser et vérifiez que le mot de passe ou la clé de compte est correct en cliquant sur **Afficher la valeur du secret**. 

## <a name="verify-permissions-for-the-purview-managed-identity-on-your-azure-key-vault"></a>Vérifier les autorisations de l’identité managée Purview sur votre coffre Azure Key Vault

Vérifiez que les autorisations appropriées ont été configurées pour que l’identité managée Purview accède à votre coffre Azure Key Vault.

Pour cela, procédez comme suit :

1. Accédez à votre coffre de clés et à la section **Stratégies d’accès**.
1. Vérifiez que votre identité managée Purview s’affiche sous la section *Stratégies d’accès actuelles* avec au minimum des autorisations **get** sur les secrets.

Si vous ne voyez pas votre identité managée Purview dans la liste, suivez les étapes décrites dans [Créer et gérer des informations d’identification pour les analyses](manage-credentials.md) afin de pouvoir l’ajouter. 

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
