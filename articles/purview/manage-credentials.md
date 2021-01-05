---
title: Créer et gérer des informations d'identification pour les analyses
description: Cet article décrit les étapes à suivre pour créer et gérer des informations d'identification dans Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: 30004306d9ff44df04a26640a2bd7a09256fce25
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516677"
---
# <a name="credentials-for-source-authentication-in-azure-purview"></a>Informations d'identification pour l'authentification des sources dans Azure Purview

Cet article explique comment créer des informations d'identification dans Azure Purview afin de réutiliser et d'appliquer rapidement les informations d'authentification enregistrées à vos analyses de source de données.

## <a name="prerequisites"></a>Prérequis

* Un coffre de clés Azure. Pour savoir comment en créer un, consultez [Démarrage rapide : Créer un coffre de clés avec le portail Azure](../key-vault/general/quick-create-portal.md).

## <a name="introduction"></a>Introduction
Les informations d'identification permettent à Azure Purview de s'authentifier auprès des sources de données que vous avez enregistrées. Un objet Informations d'identification peut être créé pour différents types de scénarios d'authentification (tels que l'authentification de base nécessitant un nom d'utilisateur/mot de passe). Celui-ci capturera les informations spécifiques requises en fonction du type de méthode d'authentification choisi. Les informations d'identification utilisent vos secrets Azure Key Vault existants pour récupérer des informations d'authentification sensibles pendant le processus de création des informations d'identification.

## <a name="using-purview-managed-identity-to-set-up-scans"></a>Utilisation de l'identité managée Purview pour configurer des analyses
Si vous utilisez l'identité managée Purview pour configurer des analyses, vous n'aurez pas à créer explicitement d'informations d'identification et à lier votre coffre de clés à Purview pour les stocker. Pour obtenir des instructions détaillées sur l'ajout de l'identité managée Purview afin d'accéder à l'analyse de vos sources de données, reportez-vous aux sections d'authentification spécifiques aux sources de données ci-dessous :

- [Stockage Blob Azure](register-scan-azure-blob-storage-source.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen1](register-scan-adls-gen1.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen2](register-scan-adls-gen2.md#setting-up-authentication-for-a-scan)
- [Azure SQL Database](register-scan-azure-sql-database.md)
- [Azure SQL Database Managed Instance](register-scan-azure-sql-database-managed-instance.md#setting-up-authentication-for-a-scan)
- [Azure Synapse Analytics](register-scan-azure-synapse-analytics.md#setting-up-authentication-for-a-scan)

## <a name="create-azure-key-vaults-connections-in-your-azure-purview-account"></a>Créer des connexions Azure Key Vault sur votre compte Azure Purview

Avant de pouvoir créer des informations d'identification, vous devez d'abord associer une ou plusieurs de vos instances Azure Key Vault existantes à votre compte Azure Purview.

1. Dans le menu de navigation Azure Purview, accédez au centre de gestion, puis aux informations d'identification.

2. Sur la barre de commandes Informations d'identification, sélectionnez Gérer les connexions Key Vault.

    :::image type="content" source="media/manage-credentials/manage-kv-connections.png" alt-text="Gérer les connexions AKV":::

3. Sélectionnez +Nouveau dans le volet Gérer les connexions Key Vault. 

4. Renseignez les informations demandées, puis sélectionnez Créer.

5. Confirmez que votre coffre de clés Key Vault a bien été associé à votre compte Azure Purview.

    :::image type="content" source="media/manage-credentials/view-kv-connections.png" alt-text="Afficher les connexions AKV":::

## <a name="grant-the-purview-managed-identity-access-to-your-azure-key-vault"></a>Autoriser l'identité managée Purview à accéder à votre coffre de clés Azure Key Vault

Accédez à votre coffre de clés -> Stratégies d'accès -> Ajouter une stratégie d'accès. Accordez l'autorisation Get dans la liste déroulante Secrets, et sélectionnez Sélectionner le principal comme MSI Purview. 

:::image type="content" source="media/manage-credentials/add-msi-to-akv.png" alt-text="Ajouter des MSI Purview à AKV":::


:::image type="content" source="media/manage-credentials/add-access-policy.png" alt-text="Ajouter une stratégie d’accès":::


:::image type="content" source="media/manage-credentials/save-access-policy.png" alt-text="Enregistrer la stratégie d'accès":::

## <a name="create-a-new-credential"></a>Créer de nouvelles informations d'identification

Type d'informations d'identification pris en charge dans Purview aujourd'hui :
* Authentification de base : Vous ajouterez le **mot de passe** comme secret dans le coffre de clés
* Principal de service : Vous ajouterez la **clé principale du service** comme secret dans le coffre de clés 
* Authentification SQL : Vous ajouterez le **mot de passe** comme secret dans le coffre de clés
* Clé de compte : Vous ajouterez la **clé de compte** comme secret dans le coffre de clés

Informations supplémentaires sur l'ajout de secrets à un coffre de clés : (insérer article sur les coffres de clés)

Après avoir stocké vos secrets dans votre coffre de clés, créez vos nouvelles informations d'identification en sélectionnant +Nouveau sur la barre de commandes Informations d'identification. Fournissez les informations requises, notamment en sélectionnant la méthode d'authentification et l'instance de Key Vault à partir de laquelle sélectionner un secret. Une fois tous les détails renseignés, cliquez sur Créer.

:::image type="content" source="media/manage-credentials/new-credential.png" alt-text="Nouvelles informations d'identification":::

Vérifiez que vos nouvelles informations d'identification apparaissent dans l'affichage Liste des informations d'identification et qu'elles sont prêtes à être utilisées.

:::image type="content" source="media/manage-credentials/view-credentials.png" alt-text="Afficher les informations d'identification":::

## <a name="manage-your-key-vault-connections"></a>Gérer vos connexions Key Vault

1. Rechercher des connexions Key Vault par nom

    :::image type="content" source="media/manage-credentials/search-kv.png" alt-text="Rechercher le coffre de clés":::

1. Supprimer une ou plusieurs connexions Key Vault
 
    :::image type="content" source="media/manage-credentials/delete-kv.png" alt-text="Supprimer un coffre de clés":::

## <a name="manage-your-credentials"></a>Gérer vos informations d'identification

1. Rechercher des informations d'identification par nom
  
2. Sélectionner et mettre à jour des informations d'identification existantes

3. Supprimer des informations d'identification
