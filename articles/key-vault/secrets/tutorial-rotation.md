---
title: Tutoriel sur la rotation pour des ressources avec un seul jeu d’informations d’authentification stocké dans Azure Key Vault
description: Utilisez ce tutoriel pour découvrir comment automatiser la rotation d’un secret pour des ressources qui utilisent un seul jeu d’informations d’authentification.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 01/26/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp
ms.openlocfilehash: 5e61510965693e123c724d7b40d2fa6071fdd94c
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97824812"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-use-one-set-of-authentication-credentials"></a>Automatiser la rotation d’un secret pour des ressources qui utilisent un seul jeu d’informations d’authentification

La meilleure façon de s’authentifier auprès des services Azure consiste à utiliser une [identité managée](../general/authentication.md). Il existe toutefois des scénarios dans lesquels cette approche est inappropriée. Dans ces cas-là, des clés d’accès ou des secrets sont utilisés. Vous devez permuter régulièrement les clés d’accès ou les secrets.

Ce tutoriel montre comment automatiser la rotation régulière des secrets pour les bases de données et les services qui utilisent un seul jeu d’informations d’authentification. Plus précisément, ce tutoriel permute les mots de passe SQL Server stockés dans Azure Key Vault à l’aide d’une fonction déclenchée par une notification Azure Event Grid :


:::image type="content" source="../media/rotate-1.png" alt-text="Diagramme de la solution de permutation":::

1. Trente jours avant la date d’expiration d’un secret, Key Vault publie l’événement « Expiration proche » sur Event Grid.
1. Event Grid vérifie les abonnements aux événements et utilise HTTP POST pour appeler le point de terminaison de l’application de fonction abonné à l’événement.
1. L’application de fonction reçoit les informations du secret, génère un nouveau mot de passe aléatoire et crée une version pour le secret avec le nouveau mot de passe dans Key Vault.
1. L’application de fonction met à jour SQL Server avec le nouveau mot de passe.

> [!NOTE]
> Il peut y avoir un décalage entre les étapes 3 et 4. Pendant ce temps, le secret dans Key Vault ne peut pas s’authentifier auprès de SQL Server. En cas de d’échec de l’une des étapes, Event Grid procède à de nouvelles tentatives pendant deux heures.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Azure Key Vault
* SQL Server

Vous pouvez utiliser le lien de déploiement ci-dessous, si vous n’avez pas de coffre de clés existant ni SQL Server :

[![Image représentant un bouton intitulé « Déployer sur Azure »](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FInitial-Setup%2Fazuredeploy.json)

1. Sous **Groupe de ressources**, sélectionnez **Créer**. Nommez le groupe **akvrotation**.
1. Sous **Nom de connexion de l’administrateur SQL**, tapez nom de connexion de l’administrateur SQL. 
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Créer**

:::image type="content" source="../media/rotate-2.png" alt-text="Créer un groupe de ressources":::

Vous disposez maintenant d’un coffre de clés et d’une instance SQL Server. Vous pouvez vérifier cette configuration dans l’interface de ligne de commande (CLI) Azure en exécutant la commande suivante :

```azurecli
az resource list -o table -g akvrotation
```

Le résultat doit ressembler à la sortie suivante :

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation      eastus      Microsoft.Sql/servers/databases
akvrotation-sql2         akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql2/master  akvrotation      eastus      Microsoft.Sql/servers/databases
```

## <a name="create-and-deploy-sql-server-password-rotation-function"></a>Créer et déployer la fonction de rotation de mot de passe SQL Server
> [!IMPORTANT]
> Le modèle ci-dessous nécessite que Key Vault, SQL Server et Azure Function se trouvent dans le même groupe de ressources

Ensuite, créez une application de fonction avec une identité managée par le système en plus des autres composants nécessaires, et déployez les fonctions de rotation de mot de passe SQL Server.

L’application de fonction nécessite les composants suivants :
- Un plan Azure App Service
- Une application de fonction avec des fonctions de rotation de mot de passe SQL, avec déclencheur d’événement et déclencheur HTTP 
- Un compte de stockage requis pour la gestion des déclencheurs de l’application de fonction
- Une stratégie d’accès pour que l’identité Function App accède aux secrets dans Key Vault
- Un abonnement aux événements EventGrid pour l’événement **SecretNearExpiry**

1. Sélectionnez le lien de déploiement d’un modèle Azure : 

   [![Image représentant un bouton intitulé « Déployer sur Azure »](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FFunction%2Fazuredeploy.json)

1. Dans la liste **Groupe de ressources**, sélectionnez **akvrotation**.
1. Dans **Nom du serveur SQL Server**, tapez le nom du serveur SQL Server avec le mot de passe à permuter.
1. Dans **Nom du coffre de clés**, tapez le nom du coffre de clés.
1. Dans **Nom de l’application de fonction**, tapez le nom de l’application de fonction.
1. Dans **Nom du secret**, tapez le nom du secret où le mot de passe sera stocké.
1. Dans **URL du référentiel**, tapez l’emplacement GitHub du code de fonction ( **https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp.git** ).
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

:::image type="content" source="../media/rotate-3.png" alt-text="Sélectionnez Vérifier + créer.":::
  

Après avoir effectué les étapes précédentes, vous disposez d’un compte de stockage, d’une batterie de serveurs et d’une application de fonction. Vous pouvez vérifier cette configuration dans l’interface de ligne de commande (CLI) Azure en exécutant la commande suivante :

```azurecli
az resource list -o table -g akvrotation
```

Le résultat doit ressembler à la sortie suivante :

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation       eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation       eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation       eastus      Microsoft.Sql/servers/databases
cfogyydrufs5wazfunctions akvrotation       eastus      Microsoft.Storage/storageAccounts
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/serverFarms
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/sites
akvrotation-fnapp        akvrotation       eastus      Microsoft.insights/components
```

Pour plus d’informations sur la création d’une application de fonction et sur le recours à une identité managée pour accéder à Key Vault, consultez [Création d’une application de fonction à partir du Portail Azure](../../azure-functions/functions-create-function-app-portal.md), [Guide pratique pour utiliser une identité managée pour App Service et Azure Functions](../../app-service/overview-managed-identity.md) et [Attribution d’une stratégie d'accès Key Vault à l’aide du Portail Azure](../general/assign-access-policy-portal.md).

### <a name="rotation-function"></a>Fonction de permutation
Déployée dans l’étape précédente, la fonction utilise un événement pour déclencher la permutation d’un secret en mettant à jour Key Vault et la base de données SQL. 

#### <a name="function-trigger-event"></a>Événement de déclencheur de fonction

Cette fonction lit les données d’événement et exécute la logique de permutation :

```csharp
public static class SimpleRotationEventHandler
{
   [FunctionName("AKVSQLRotation")]
   public static void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
   {
      log.LogInformation("C# Event trigger function processed a request.");
      var secretName = eventGridEvent.Subject;
      var secretVersion = Regex.Match(eventGridEvent.Data.ToString(), "Version\":\"([a-z0-9]*)").Groups[1].ToString();
      var keyVaultName = Regex.Match(eventGridEvent.Topic, ".vaults.(.*)").Groups[1].ToString();
      log.LogInformation($"Key Vault Name: {keyVaultName}");
      log.LogInformation($"Secret Name: {secretName}");
      log.LogInformation($"Secret Version: {secretVersion}");

      SecretRotator.RotateSecret(log, secretName, keyVaultName);
   }
}
```

#### <a name="secret-rotation-logic"></a>Logique de permutation du secret
Cette méthode de permutation lit les informations de la base de données à partir du secret, crée une version du secret et met à jour la base de données avec le nouveau secret :

```csharp
    public class SecretRotator
    {
        private const string CredentialIdTag = "CredentialId";
        private const string ProviderAddressTag = "ProviderAddress";
        private const string ValidityPeriodDaysTag = "ValidityPeriodDays";

        public static void RotateSecret(ILogger log, string secretName, string keyVaultName)
        {
            //Retrieve Current Secret
            var kvUri = "https://" + keyVaultName + ".vault.azure.net";
            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());
            KeyVaultSecret secret = client.GetSecret(secretName);
            log.LogInformation("Secret Info Retrieved");

            //Retrieve Secret Info
            var credentialId = secret.Properties.Tags.ContainsKey(CredentialIdTag) ? secret.Properties.Tags[CredentialIdTag] : "";
            var providerAddress = secret.Properties.Tags.ContainsKey(ProviderAddressTag) ? secret.Properties.Tags[ProviderAddressTag] : "";
            var validityPeriodDays = secret.Properties.Tags.ContainsKey(ValidityPeriodDaysTag) ? secret.Properties.Tags[ValidityPeriodDaysTag] : "";
            log.LogInformation($"Provider Address: {providerAddress}");
            log.LogInformation($"Credential Id: {credentialId}");

            //Check Service Provider connection
            CheckServiceConnection(secret);
            log.LogInformation("Service  Connection Validated");
            
            //Create new password
            var randomPassword = CreateRandomPassword();
            log.LogInformation("New Password Generated");

            //Add secret version with new password to Key Vault
            CreateNewSecretVersion(client, secret, randomPassword);
            log.LogInformation("New Secret Version Generated");

            //Update Service Provider with new password
            UpdateServicePassword(secret, randomPassword);
            log.LogInformation("Password Changed");
            log.LogInformation($"Secret Rotated Successfully");
        }
}
```
Le code complet est disponible sur [GitHub](https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp).

## <a name="add-the-secret-to-key-vault"></a>Ajouter le secret à Key Vault
Définissez votre stratégie d’accès pour accorder des autorisations *Gérer les secrets* aux utilisateurs :

```azurecli
az keyvault set-policy --upn <email-address-of-user> --name akvrotation-kv --secret-permissions set delete get list
```

Créez un secret avec des étiquettes qui contiennent l’ID de ressource SQL Server, le nom de connexion SQL Server et la période de validité du secret, en jours. Spécifiez le nom du secret, le mot de passe initial de la base de données SQL (dans notre exemple « Simple123 ») et ajoutez une date d’expiration définie sur demain.

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name sqlPassword --vault-name akvrotation-kv --value "Simple123" --tags "CredentialId=sqlAdmin" "ProviderAddress=<sql-database-resource-id>" "ValidityPeriodDays=90" --expires $tomorrowDate
```

La création d’un secret avec une date d’expiration proche entraîne la publication d’un événement `SecretNearExpiry` dans un délai de 15 minutes, qui déclenche à son tour la fonction de rotation du secret.

## <a name="test-and-verify"></a>Tester et vérifier

Pour vérifier que le secret a permuté, accédez à **Key Vault** > **Secrets** :

:::image type="content" source="../media/rotate-8.png" alt-text="Accéder à Secrets":::

Ouvrez le secret **sqlPassword**, et visualisez la version d’origine et la version permutée :

:::image type="content" source="../media/rotate-9.png" alt-text="Accéder à Secrets":::

### <a name="create-a-web-app"></a>Créer une application web

Pour vérifier les informations d’identification SQL, créez une application web. Cette application web obtient le secret à partir de Key Vault, extrait les informations de la base de données SQL et les informations d’identification du secret, puis teste la connexion à SQL Server.

L’application web nécessite les composants suivants :
- Une application web avec identité managée par le système
- Une stratégie d’accès pour accéder aux secrets dans Key Vault par le biais d’une identité managée d’application web

1. Sélectionnez le lien de déploiement d’un modèle Azure : 

   [![Image représentant un bouton intitulé « Déployer sur Azure »](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp-WebApp%2Fmain%2FARM-Templates%2FWeb-App%2Fazuredeploy.json)

1. Sélectionnez le groupe de ressources **akvrotation**.
1. Dans **Nom du serveur SQL Server**, tapez le nom du serveur SQL Server avec le mot de passe à permuter.
1. Dans **Nom du coffre de clés**, tapez le nom du coffre de clés.
1. Dans **Nom du secret**, tapez le nom du secret où le mot de passe est stocké.
1. Dans **URL du dépôt**, tapez l’emplacement GitHub du code de l’application web ( **https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp-WebApp.git** ).
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).


### <a name="open-the-web-app"></a>Ouvrir l’application web

Accédez à l’URL de l’application déployée :
 
https://akvrotation-app.azurewebsites.net/

Lorsque l’application s’ouvre dans le navigateur, vous voyez la **Valeur du secret généré** et une **Base de données connectée** dont la valeur est *true*.

## <a name="learn-more"></a>En savoir plus

- Tutoriel : [Rotation pour les ressources avec deux jeux d’informations d’identification](tutorial-rotation-dual.md)
- Présentation : [Supervision de Key Vault avec Azure Event Grid](../general/event-grid-overview.md)
- Procédure : [Recevoir un e-mail en cas de changement d’un secret de coffre de clés](../general/event-grid-logicapps.md)
- [Schéma des événements Azure Event Grid pour Azure Key Vault](../../event-grid/event-schema-key-vault.md)