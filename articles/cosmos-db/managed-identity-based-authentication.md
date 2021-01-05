---
title: Comment utiliser une identité managée de machine virtuelle Linux attribuée par le système pour accéder aux données Azure Cosmos DB
description: Découvrez comment configurer une identité managée affectée par le système (Managed Service Identity) Azure Active Directory (Azure AD) pour accéder à des clés à partir d’Azure Cosmos DB.
author: j-patrick
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 03/20/2020
ms.author: justipat
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: cfef6ce0fb38f074f854d5ceb77677843e44b91b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96345727"
---
# <a name="use-system-assigned-managed-identities-to-access-azure-cosmos-db-data"></a>Utiliser des identités managées affectées par le système pour accéder aux données Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans cet article, vous allez configurer une solution *robuste et indépendante de la rotation des clés* pour accéder aux clés d’Azure Cosmos DB à l’aide d’[identités managées](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md). L’exemple de cet article utilise Azure Functions, mais vous pouvez utiliser n’importe quel service qui prend en charge les identités managées. 

Vous allez apprendre à créer une application de fonction qui peut accéder aux données Azure Cosmos DB sans avoir à copier les clés d’Azure Cosmos DB. L’application de fonction s’activera toutes les minutes et enregistrera la température actuelle d’un aquarium. Pour savoir comment configurer une application de fonction déclenchée par un minuteur, consultez l’article [Créer une fonction dans Azure déclenchée par un minuteur](../azure-functions/functions-create-scheduled-function.md).

Pour simplifier le scénario, un paramètre [Durée de vie](./time-to-live.md) est déjà configuré pour nettoyer les documents de température plus anciens. 

## <a name="assign-a-system-assigned-managed-identity-to-a-function-app"></a>Attribuer une identité managée affectée par le système à une application de fonction

Dans cette étape, vous allez attribuer une identité managée affectée par le système à votre application de fonction.

1. Dans le [Portail Azure](https://portal.azure.com/), ouvrez le volet **Fonction Azure** et accédez à votre application de fonction. 

1. Ouvrez l’onglet **Identité** >  **des fonctionnalités de plateforme** : 

   :::image type="content" source="./media/managed-identity-based-authentication/identity-tab-selection.png" alt-text="Capture d’écran montrant les fonctionnalités de la plateforme et les options d’identité pour l’application de fonction.":::

1. Sous l’onglet **Identité**, définissez l’**état** de l’identité du système sur **Activé** et sélectionnez **Enregistrer**. Le volet **Identité** doit se présenter comme suit :  

   :::image type="content" source="./media/managed-identity-based-authentication/identity-tab-system-managed-on.png" alt-text="Capture d’écran montrant l’état de l’identité du système défini sur Activé.":::

## <a name="grant-access-to-your-azure-cosmos-account"></a>Accorder l’accès à votre compte Azure Cosmos

Dans cette étape, vous allez attribuer un rôle à l’identité managée affectée par le système de votre application de fonction. Azure Cosmos DB a plusieurs rôles intégrés que vous pouvez attribuer à l’identité managée. Pour cette solution, vous allez utiliser les deux rôles suivants :

|Rôle intégré  |Description  |
|---------|---------|
|[Contributeur de compte DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)|Gérer des comptes Azure Cosmos DB. Autorise la récupération des clés en lecture/écriture. |
|[Rôle de lecteur de compte Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)|Lire les données de comptes Azure Cosmos DB. Autorise la récupération des clés de lecture. |

> [!IMPORTANT]
> La prise en charge du contrôle d’accès en fonction du rôle dans Azure Cosmos DB s’applique uniquement aux opérations du plan de contrôle. Les opérations du plan de données sont sécurisées par le biais de clés primaires ou de jetons de ressources. Pour en savoir plus, consultez l’article [Sécuriser l’accès aux données](secure-access-to-data.md).

> [!TIP] 
> Lorsque vous attribuez des rôles, attribuez uniquement l’accès nécessaire. Si votre service nécessite uniquement la lecture des données, attribuez le rôle **Lecteur de compte Cosmos DB** à l’identité managée. Pour plus d’informations sur l’importance de l’accès avec un privilège minimum, consultez l’article [Exposition réduite des comptes privilégiés](../security/fundamentals/identity-management-best-practices.md#lower-exposure-of-privileged-accounts).

Dans ce scénario, l’application de fonction lira la température de l’aquarium, puis mettra à jour ces données dans un conteneur d’Azure Cosmos DB. Étant donné que l’application de fonction doit écrire les données, vous devez attribuer le rôle **Contributeur de compte DocumentDB**. 

### <a name="assign-the-role-using-azure-portal"></a>Assigner le rôle à l’aide du Portail Azure

1. Connectez-vous au Portail Azure et accédez à votre compte Azure Cosmos DB. Ouvrez le volet **Contrôle d’accès (IAM)** , puis l’onglet **Attributions de rôle** :

   :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab.png" alt-text="Capture d’écran montrant le volet Contrôle d’accès et l’onglet Attributions de rôles.":::

1. Sélectionnez **+ Ajouter** > **Ajouter une attribution de rôle**.

1. Le volet **Ajouter une attribution de rôle** s’ouvre à droite :

   :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab-add-role-pane.png" alt-text="Capture d’écran montrant le volet Ajouter une attribution de rôle.":::

   * **Rôle** : sélectionnez **Contributeur de compte DocumentDB**.
   * **Attribuer l’accès à** : sous la sous-section **Sélectionner l’identité managée affectée par le système**, sélectionnez **Application de fonction**.
   * **Select** : le volet sera rempli avec toutes les applications de fonction de votre abonnement qui ont une **Identité de système managée**. Dans ce cas, sélectionnez l’application de fonction **FishTankTemperatureService** : 

      :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab-add-role-pane-filled.png" alt-text="Capture d’écran montrant le volet Ajouter une attribution de rôle rempli avec des exemples.":::

1. Après avoir sélectionné votre application de fonction, sélectionnez **Enregistrer**.

### <a name="assign-the-role-using-azure-cli"></a>Assigner le rôle à l’aide de l’Interface de ligne de commande Azure

Pour attribuer le rôle à l’aide d’Azure CLI, ouvrez Azure Cloud Shell et exécutez les commandes suivantes :

```azurecli-interactive

scope=$(az cosmosdb show --name '<Your_Azure_Cosmos_account_name>' --resource-group '<CosmosDB_Resource_Group>' --query id)

principalId=$(az webapp identity show -n '<Your_Azure_Function_name>' -g '<Azure_Function_Resource_Group>' --query principalId)

az role assignment create --assignee $principalId --role "DocumentDB Account Contributor" --scope $scope
```

## <a name="programmatically-access-the-azure-cosmos-db-keys"></a>Accéder par programme aux clés d’Azure Cosmos DB

À présent, nous disposons d’une application de fonction dotée d’une identité managée affectée par le système avec le rôle **Contributeur de compte DocumentDB** dans les autorisations d’Azure Cosmos DB. Le code d’application de fonction suivant obtient les clés d’Azure Cosmos DB, crée un objet CosmosClient, obtient la température de l’aquarium, puis enregistre cette dernière dans Cosmos DB.

Cet exemple utilise l’[API Répertorier les clés](/rest/api/cosmos-db-resource-provider/2020-04-01/databaseaccounts/listkeys) pour accéder à vos clés de compte Azure Cosmos DB.

> [!IMPORTANT] 
> Si vous voulez attribuer le rôle [Lecteur de compte Cosmos DB](#grant-access-to-your-azure-cosmos-account), vous allez devoir utiliser l’[API Répertorier les clés en lecture seule](/rest/api/cosmos-db-resource-provider/2020-04-01/databaseaccounts/listreadonlykeys). Seules les clés en lecture seule seront remplies.

L’API Répertorier les clés retourne l’objet `DatabaseAccountListKeysResult`. Ce type n’est pas défini dans les bibliothèques C#. Le code suivant montre l'implémentation de cette classe :  

```csharp 
namespace Monitor 
{
  public class DatabaseAccountListKeysResult
  {
      public string primaryMasterKey {get;set;}
      public string primaryReadonlyMasterKey {get; set;}
      public string secondaryMasterKey {get; set;}
      public string secondaryReadonlyMasterKey {get;set;}
  }
}
```

L’exemple utilise également un document simple appelé « TemperatureRecord », qui est défini comme suit :

```csharp
using System;

namespace Monitor
{
    public class TemperatureRecord
    {
        public string id { get; set; } = Guid.NewGuid().ToString();
        public DateTime RecordTime { get; set; }
        public int Temperature { get; set; }

    }
}
```

Vous allez utiliser la bibliothèque [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) pour obtenir le jeton de l’identité managée affectée par le système. Pour en savoir plus sur les autres façons d’obtenir le jeton et sur la bibliothèque `Microsoft.Azure.Service.AppAuthentication`, consultez l’article [Authentification de service à service](../key-vault/general/service-to-service-authentication.md).


```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Azure.Cosmos;
using Microsoft.Azure.Services.AppAuthentication;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

namespace Monitor
{
    public static class FishTankTemperatureService
    {
        private static string subscriptionId =
        "<azure subscription id>";
        private static string resourceGroupName =
        "<name of your azure resource group>";
        private static string accountName =
        "<Azure Cosmos DB account name>";
        private static string cosmosDbEndpoint =
        "<Azure Cosmos DB endpoint>";
        private static string databaseName =
        "<Azure Cosmos DB name>";
        private static string containerName =
        "<container to store the temperature in>";

        [FunctionName("FishTankTemperatureService")]
        public static async Task Run([TimerTrigger("0 * * * * *")]TimerInfo myTimer, ILogger log)
        {
            log.LogInformation($"Starting temperature monitoring: {DateTime.Now}");

            // AzureServiceTokenProvider will help us to get the Service Managed token.
            var azureServiceTokenProvider = new AzureServiceTokenProvider();

            // Authenticate to the Azure Resource Manager to get the Service Managed token.
            string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");

            // Setup the List Keys API to get the Azure Cosmos DB keys.
            string endpoint = $"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/listKeys?api-version=2019-12-12";

            // Setup an HTTP Client and add the access token.
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            // Post to the endpoint to get the keys result.
            var result = await httpClient.PostAsync(endpoint, new StringContent(""));

            // Get the result back as a DatabaseAccountListKeysResult.
            DatabaseAccountListKeysResult keys = await result.Content.ReadAsAsync<DatabaseAccountListKeysResult>();

            log.LogInformation("Starting to create the client");

            CosmosClient client = new CosmosClient(cosmosDbEndpoint, keys.primaryMasterKey);

            log.LogInformation("Client created");

            var database = client.GetDatabase(databaseName);
            var container = database.GetContainer(containerName);

            log.LogInformation("Get the temperature.");

            var tempRecord = new TemperatureRecord() { RecordTime = DateTime.UtcNow, Temperature = GetTemperature() };

            log.LogInformation("Store temperature");

            await container.CreateItemAsync<TemperatureRecord>(tempRecord);

            log.LogInformation($"Ending temperature monitor: {DateTime.Now}");
        }

        private static int GetTemperature()
        {
            // Fake the temperature sensor for this demo.
            Random r = new Random(DateTime.UtcNow.Second);
            return r.Next(0, 120);
        }
    }
}
```

Vous êtes maintenant prêt à [déployer votre application de fonction](../azure-functions/create-first-function-vs-code-csharp.md).

## <a name="next-steps"></a>Étapes suivantes

* [Authentification basée sur les certificats avec Azure Cosmos DB et Azure Active Directory](certificate-based-authentication.md)
* [Sécuriser les clés Azure Cosmos DB à l’aide d’Azure Key Vault](access-secrets-from-keyvault.md)
* [Base de référence de sécurité pour Azure Cosmos DB](security-baseline.md)