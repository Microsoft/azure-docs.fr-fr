---
title: Utiliser Key Vault pour stocker des clés Azure Cosmos DB et y accéder
description: Utilisez Azure Key Vault pour stocker une chaîne de connexion, des clés et des points de terminaison Azure Cosmos DB et y accéder.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: how-to
ms.date: 05/23/2019
ms.reviewer: sngun
ms.openlocfilehash: a9bea0664f99a21ac734de666c802e9875ff00b5
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359318"
---
# <a name="secure-azure-cosmos-keys-using-azure-key-vault"></a>Sécuriser les clés Azure Cosmos à l’aide d’Azure Key Vault 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

>[!IMPORTANT]
> La solution recommandée pour accéder aux clés Azure Cosmos DB consiste à utiliser une [identité managée attribuée par le système](managed-identity-based-authentication.md). Si votre service ne peut pas tirer parti des identités managées, utilisez la [solution basée sur le certificat](certificate-based-authentication.md). Si la solution d'identité managée et la solution basée sur le certificat ne répondent pas à vos besoins, utilisez la solution de coffre de clés ci-dessous.

Lorsque vous utilisez Azure Cosmos DB pour vos applications, vous pouvez accéder à la base de données, aux collections et aux documents à l'aide du point de terminaison et de la clé figurant dans le fichier de configuration de l'application.  Toutefois, il est déconseillé de placer les clés et l'URL directement dans le code d'application, car elles sont disponibles au format texte clair pour tous les utilisateurs. Vous voulez vous assurer que le point de terminaison et les clés sont disponibles, mais par le biais d’un mécanisme sécurisé. C’est ici qu’Azure Key Vault entre en jeu, en vous permettant de stocker et de gérer les secrets d’application de manière sécurisée.

Les étapes suivantes sont nécessaires pour stocker et lire les clés d’accès Azure Cosmos DB à partir de Key Vault :

* Créer un coffre de clés  
* Ajouter des clés d’accès Azure Cosmos DB au coffre de clés  
* Créer une application web Azure  
* Inscrire l’application et accorder des autorisations pour lire le coffre de clés  


## <a name="create-a-key-vault"></a>Créer un coffre de clés

1. Connectez-vous au [portail Azure](https://portal.azure.com/).  
2. Sélectionnez **Créer une ressource > Sécurité > Coffre de clés**.  
3. Dans la section **Créer un coffre de clés**, renseignez les informations suivantes :  
   * **Nom :** fournissez un nom unique pour votre coffre de clés.  
   * **Abonnement :** choisissez l’abonnement à utiliser.  
   * Sous **Groupe de ressources**, choisissez **Créer** et entrez le nom du groupe de ressources.  
   * Dans le menu déroulant Emplacement, choisissez un emplacement.  
   * Conservez les valeurs par défaut des autres options.  
4. Après avoir renseigné les informations ci-dessus, sélectionnez **Créer**.  

## <a name="add-azure-cosmos-db-access-keys-to-the-key-vault"></a>Ajouter des clés d’accès Azure Cosmos DB au coffre de clés
1. Accédez au coffre de clés que vous avez créé à l’étape précédente, puis ouvrez l’onglet **Secrets**.  
2. Sélectionnez **+Générer/importer**, 

   * Sélectionnez **Manuel** pour voir les **Options de chargement**.
   * Entrez un **nom** pour votre secret.
   * Entrez la chaîne de connexion de votre compte Cosmos DB dans le champ **Valeur**. Ensuite, sélectionnez **Créer**.

   :::image type="content" source="./media/access-secrets-from-keyvault/create-a-secret.png" alt-text="Création d’une clé secrète":::

4. Une fois le secret créé, ouvrez-le et copiez l’identificateur de secret qui se présente sous la forme suivante. Vous utiliserez cet identificateur à la section suivante. 

   `https://<Key_Vault_Name>.vault.azure.net/secrets/<Secret _Name>/<ID>`

## <a name="create-an-azure-web-application"></a>Créer une application web Azure

1. Créez une application web Azure ou bien téléchargez l’application à partir du [dépôt GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/Demo/keyvaultdemo). Il s’agit d’une application MVC simple.  

2. Décompressez l’application téléchargée et ouvrez le fichier **HomeController.cs**. Mettez à jour l’ID de secret dans la ligne suivante :

   `var secret = await keyVaultClient.GetSecretAsync("<Your Key Vault’s secret identifier>")`

3. **Enregistrez** le fichier, puis **générez** la solution.  
4. Ensuite, déployez l’application sur Azure. Cliquez avec le bouton droit sur le projet et choisissez **Publier**. Créez un profil de service d’application (vous pouvez nommer l’application WebAppKeyVault1) et sélectionnez **Publier**.   

5. Une fois l’application déployée, dans le portail Azure, accédez-y et activez son **identité MSI** (Managed Service Identity).  

   :::image type="content" source="./media/access-secrets-from-keyvault/turn-on-managed-service-identity.png" alt-text="Identité MSI (Managed Service Identity)":::

Si vous exécutez l’application maintenant, vous verrez l’erreur suivante, car vous n’avez pas accordé d’autorisation pour cette application dans Key Vault.

:::image type="content" source="./media/access-secrets-from-keyvault/app-deployed-without-access.png" alt-text="Application déployée sans accès":::

## <a name="register-the-application--grant-permissions-to-read-the-key-vault"></a>Inscrire l’application et accorder des autorisations pour lire le coffre de clés

Dans cette section, vous inscrivez l’application auprès d’Azure Active Directory et accordez des autorisations pour que l’application puisse lire le coffre de clés. 

1. Accédez au portail Azure, puis ouvrez le **coffre de clés** que vous avez créé à la section précédente.  

2. Ouvrez **Stratégies d’accès**, sélectionnez **+Ajouter un nouveau**, recherchez l’application web que vous avez déployée, sélectionnez les autorisations et sélectionnez **OK**.  

   :::image type="content" source="./media/access-secrets-from-keyvault/add-access-policy.png" alt-text="Ajouter une stratégie d’accès":::

Maintenant, si vous exécutez l’application, vous pouvez lire le secret dans le coffre de clés.

:::image type="content" source="./media/access-secrets-from-keyvault/app-deployed-with-access.png" alt-text="Application déployée avec un secret":::
 
De même, vous pouvez ajouter un utilisateur pour qu’il accède au coffre de clés. Vous devez vous ajouter vous-même au coffre de clés sélectionnant **Stratégies d’accès**, puis accorder toutes les autorisations dont vous avez besoin pour exécuter l’application à partir de Visual studio. Quand cette application s’exécute à partir de votre bureau, elle prend votre identité.

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer un pare-feu pour Azure Cosmos DB, consultez l’article sur la [prise en charge du pare-feu](how-to-configure-firewall.md).
* Pour configurer le point de terminaison de service de réseau virtuel, consultez l’article [Sécuriser l’accès à l’aide du point de terminaison de service de réseau virtuel](how-to-configure-vnet-service-endpoint.md).
