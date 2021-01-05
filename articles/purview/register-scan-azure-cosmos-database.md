---
title: Guide pratique pour analyser Azure Cosmos DB (API SQL)
description: Ce guide pratique explique en détail comment analyser Azure Cosmos DB (API SQL).
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/9/2020
ms.openlocfilehash: 1aaeed1973ebd15af312b722ab61938aa4271947
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97696249"
---
# <a name="register-and-scan-azure-cosmos-database-sql-api"></a>Inscription et analyse d’Azure Cosmos DB (API SQL)

Cet article explique comment inscrire un compte Azure Cosmos DB (API SQL) dans Azure Purview et configurer une analyse.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Azure Cosmos DB (API SQL) prend en charge les analyses complètes et incrémentielles pour capturer les métadonnées et le schéma. Ces analyses permettent également de classer les données automatiquement selon des règles de classification système et personnalisées.

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Création d’un compte Azure Purview](create-catalog-portal.md).
- Vous devez être administrateur de la source de données Azure Purview.

## <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Il n’existe qu’une seule façon de configurer l’authentification pour Azure Cosmos DB (API SQL) :

- Clé de compte
 
### <a name="account-key"></a>Clé de compte

Si la méthode d’authentification sélectionnée est **Clé de compte**, vous devez récupérer votre clé d’accès et la stocker dans le coffre de clés :

1. Accédez à votre compte Cosmos DB sur le Portail Azure. 
1. Sélectionnez **Paramètres** > **Clés**. 
1. Copiez votre *clé* et enregistrez-la quelque part pour la suite.
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer**, puis entrez le **Nom** et la **Valeur** comme *clé* de votre compte de stockage.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide de la clé pour configurer votre analyse.

## <a name="register-an-azure-cosmos-database-sql-api-account"></a>Inscription d’un compte Azure Cosmos DB (API SQL)

Pour inscrire un nouveau compte Azure Cosmos DB (API SQL) dans votre catalogue de données, procédez comme suit :

1. Accédez à votre compte Purview.
1. Sélectionnez **Sources** dans le volet de navigation de gauche.
1. Sélectionnez **Inscrire**.
1. Sous **Inscrire des sources**, sélectionnez **Azure Cosmos DB (API SQL)** .
1. Sélectionnez **Continue** (Continuer)

:::image type="content" source="media/register-scan-azure-cosmos-database/register-new-data-source.png" alt-text="Inscription d’une nouvelle source de données" border="true":::

Sur l’écran **Inscrire des sources (Azure Cosmos DB (API SQL))** , procédez comme suit :

1. Entrez le **Nom** sous lequel la source de données apparaîtra dans le catalogue.
1. Choisissez la façon dont vous souhaitez pointer vers le compte de stockage de votre choix :
   1. Sélectionnez **À partir d’un abonnement Azure**, puis sélectionnez l’abonnement approprié dans la zone de liste déroulante **Abonnement Azure** et le compte Cosmos DB approprié dans la zone de liste déroulante **Nom du compte Cosmos DB**.
   1. Vous pouvez également sélectionner **Entrer manuellement** et entrer un point de terminaison de service (URL).
1. Sélectionnez **Terminer** pour inscrire la source de données.

:::image type="content" source="media/register-scan-azure-cosmos-database/register-sources.png" alt-text="Options d’inscription des sources" border="true":::


[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
