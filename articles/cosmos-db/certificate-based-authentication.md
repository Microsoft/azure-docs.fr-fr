---
title: Authentification basée sur les certificats avec Azure Cosmos DB et Azure Active Directory
description: Apprenez comment configurer une identité Azure AD pour l’authentification basée sur certificat pour accéder aux clés depuis Azure Cosmos DB.
author: voellm
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/11/2019
ms.author: tvoellm
ms.reviewer: sngun
ms.openlocfilehash: e0913351d40cd75da17d16cca119b4ad5ce20de0
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334699"
---
# <a name="certificate-based-authentication-for-an-azure-ad-identity-to-access-keys-from-an-azure-cosmos-db-account"></a>Authentification basée sur les certificats pour une identité Azure AD pour accéder aux clés d’un compte Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L’authentification par certificat permet d’authentifier votre application client en utilisant Azure Active Directory (Azure AD) avec un certificat client. Vous pouvez effectuer une authentification basée sur un certificat sur une machine sur laquelle vous avez besoin d’une identité, telle qu’une machine locale ou une machine virtuelle dans Azure. Votre application peut alors lire les clés Azure Cosmos DB sans avoir les clés directement dans l’application. Cet article décrit comment créer un exemple d’application Azure AD, la configurer pour l’authentification par certificat, se connecter à Azure en utilisant la nouvelle identité d’application, puis récupérer les clés de votre compte Azure Cosmos. Cet article utilise Azure PowerShell pour configurer les identités et fournit un exemple d’application C# qui authentifie et accède aux clés depuis votre compte Azure Cosmos.  

## <a name="prerequisites"></a>Prérequis

* Installez la [version la plus récente](/powershell/azure/install-az-ps) d’Azure PowerShell.

* Si vous n’avez pas [d’abonnement Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

## <a name="register-an-app-in-azure-ad"></a>Inscrire une application dans Azure AD

Dans cette étape, vous inscrirez un exemple d’application web dans votre compte Azure AD. Cette application est ensuite utilisée pour lire les clés de votre compte Azure Cosmos DB. Suivez cette procédure pour inscrire une application : 

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez le volet Azure **Active Directory** , accédez au volet **Inscriptions d’applications** , puis sélectionnez **Nouvelle inscription**. 

   :::image type="content" source="./media/certificate-based-authentication/new-app-registration.png" alt-text="Nouvelle inscription d’application dans Active Directory":::

1. Remplissez le formulaire **Inscrire une application** , notamment les détails suivants :  

   * **Nom**  : entrez un nom pour votre application. Cela peut être n’importe quel nom, tel que « sampleApp ».
   * **Types de comptes pris en charge**  : choisissez **Comptes dans cet annuaire organisationnel uniquement (répertoire par défaut)** afin de permettre aux ressources de votre répertoire actuel d’accéder à cette application. 
   * **URL de redirection**  : choisissez l’application de type **Web** et fournissez une URL pointant vers l’emplacement où votre application est hébergée. Il peut s’agir de n’importe quelle URL. Pour cet exemple, vous pouvez fournir une URL de test de type `https://sampleApp.com`, même si l’application n’existe pas.

   :::image type="content" source="./media/certificate-based-authentication/register-sample-web-app.png" alt-text="Inscription d’un exemple d’application web":::

1. Une fois le formulaire rempli, sélectionnez **Inscrire**.

1. Une fois l’application inscrite, notez l’ **ID (du client) d’application** et l’ **ID d’objet**. Ces informations vous seront utiles plus tard. 

   :::image type="content" source="./media/certificate-based-authentication/get-app-object-ids.png" alt-text="Obtenir les ID de l’application et de l’objet":::

## <a name="install-the-azuread-module"></a>Installer le module AzureAD

Dans cette étape, vous allez installer le module Azure AD PowerShell. Ce module est nécessaire pour obtenir l’ID de l’application que vous avez inscrite à l’étape précédente et associer un certificat autosigné à cette application. 

1. Ouvrez Windows PowerShell ISE avec des droits d’administrateur. Si vous ne l’avez pas déjà fait, installez le module AZ PowerShell et connectez-vous à votre abonnement. Si vous avez plusieurs abonnements, vous pouvez définir le contexte de votre abonnement actuel, comme indiqué dans les commandes suivantes :

   ```powershell
   Install-Module -Name Az -AllowClobber
   Connect-AzAccount

   Get-AzSubscription
   $context = Get-AzSubscription -SubscriptionId <Your_Subscription_ID>
   Set-AzContext $context 
   ```

1. Installer et importer le module [AzureAD](/powershell/module/azuread/?view=azureadps-2.0&preserve-view=true)

   ```powershell
   Install-Module AzureAD
   Import-Module AzureAD 
   ```

## <a name="sign-into-your-azure-ad"></a>Connexion à votre compte Azure AD

Connectez-vous à votre compte AD Azure où vous avez inscrit l’application. Utilisez la commande Connect-AzureAD pour vous connecter à votre compte, entrez les informations d’identification de votre compte Azure dans la fenêtre contextuelle. 

```powershell
Connect-AzureAD 
```

## <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Ouvrez une autre instance de Windows PowerShell ISE et exécutez les commandes suivantes pour créer un certificat autosigné et lire la clé associée au certificat :

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "Cert:\CurrentUser\My" -Subject "CN=sampleAppCert" -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData()) 
```

## <a name="create-the-certificate-based-credential"></a>Créer les informations d’identification basées sur le certificat 

Ensuite, exécutez les commandes suivantes pour obtenir l’ID d’objet de votre application et créer les informations d’identification basées sur le certificat. Dans cet exemple, nous définissons l’expiration du certificat après un an. Mais de votre côté, vous pouvez la définir à n’importe quelle date de fin.

```powershell
$application = Get-AzureADApplication -ObjectId <Object_ID_of_Your_Application>

New-AzureADApplicationKeyCredential -ObjectId $application.ObjectId -CustomKeyIdentifier "Key1" -Type AsymmetricX509Cert -Usage Verify -Value $keyValue -EndDate "2020-01-01"
```

La commande ci-dessus donne une sortie similaire à la capture d’écran ci-dessous :

:::image type="content" source="./media/certificate-based-authentication/certificate-based-credential-output.png" alt-text="Sortie de la création d’informations d’identification basées sur un certificat":::

## <a name="configure-your-azure-cosmos-account-to-use-the-new-identity"></a>Configurez votre compte Azure Cosmos pour utiliser la nouvelle identité

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Accédez à votre compte Azure Cosmos, ouvrez le panneau **Contrôle d’accès (IAM)** .

1. Sélectionnez **Ajouter** , puis **Ajouter une attribution de rôle**. Ajoutez le sampleApp que vous avez créé à l’étape précédente avec le rôle **Contributeur** , comme indiqué dans la capture d’écran suivante :

   :::image type="content" source="./media/certificate-based-authentication/configure-cosmos-account-with-identify.png" alt-text="Configurer le compte Azure Cosmos pour utiliser la nouvelle identité":::

1. Une fois le formulaire rempli, sélectionnez **Enregistrer**

## <a name="register-your-certificate-with-azure-ad"></a>Inscrire votre certificat dans Azure AD

Vous pouvez associer les informations d’identification basées sur un certificat à l’application cliente dans Azure AD via le Portail Azure. Pour associer les informations d’identification, vous devez télécharger le fichier de certificat en procédant comme suit :

Dans l’inscription d’application Azure pour l’application cliente :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez le volet Azure **Active Directory** , accédez au volet **Inscriptions d’applications** , puis ouvre l’exemple d’application que vous avez créé à l’étape précédente. 

1. Sélectionnez **Certificats et secrets** , puis **Télécharger un certificat**. Parcourez jusqu’au fichier de certificat que vous avez créé à l’étape précédente pour le télécharger.

1. Sélectionnez **Ajouter**. Une fois le certificat téléchargé, les valeurs d’empreinte, de date de début et d’expiration s’affichent.

## <a name="access-the-keys-from-powershell"></a>Accéder aux clés à partir de PowerShell

Dans cette étape, vous vous connecterez à Azure en utilisant l’application et le certificat que vous avez créé et accéderez aux clés de votre compte Azure Cosmos. 

1. Tout d’abord, effacez les informations d’identification de compte Azure que vous avez utilisées pour vous connecter à votre compte. Vous pouvez effacer les informations d’identification à l’aide de la commande suivante :

   ```powershell
   Disconnect-AzAccount -Username <Your_Azure_account_email_id> 
   ```

1. Vérifiez ensuite que vous pouvez vous connecter au portail Azure avec les informations d’identification de l’application et accéder aux clés Azure Cosmos DB :

   ```powershell
   Login-AzAccount -ApplicationId <Your_Application_ID> -CertificateThumbprint $cert.Thumbprint -ServicePrincipal -Tenant <Tenant_ID_of_your_application>

   Get-AzCosmosDBAccountKey `
      -ResourceGroupName "<Resource_Group_Name_of_your_Azure_Cosmos_account>" `
      -Name "<Your_Azure_Cosmos_Account_Name>" `
      -Type "Keys"
   ```

La commande précédente affichera les clés primaire et secondaire de votre compte Azure Cosmos. Vous pouvez consulter le journal d’activité de votre compte Azure Cosmos pour valider la réussite de la requête d’obtention des clés, ainsi que le déclenchement de l’événement par l’application « sampleApp ».

:::image type="content" source="./media/certificate-based-authentication/activity-log-validate-results.png" alt-text="Valider l’appel d’obtention des clés dans Azure AD":::

## <a name="access-the-keys-from-a-c-application"></a>Accéder aux clés à partir d’une application C# 

Vous pouvez également valider ce scénario en accédant aux clés à partir d’une application C#. Par exemple, l’application de console C# suivante peut accéder aux clés de base de données Azure Cosmos DB en utilisant l’application inscrite dans Active Directory. Avant d’exécuter le code, mettez à jour les détails du compte tenantId, clientID, certName, nom du groupe de ressources, ID d’abonnement, Nom du compte Azure Cosmos. 

```csharp
using System;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Linq;
using System.Net.Http;
using System.Security.Cryptography.X509Certificates;
using System.Threading;
using System.Threading.Tasks;
 
namespace TodoListDaemonWithCert
{
    class Program
    {
        private static string aadInstance = "https://login.windows.net/";
        private static string tenantId = "<Your_Tenant_ID>";
        private static string clientId = "<Your_Client_ID>";
        private static string certName = "<Your_Certificate_Name>";
 
        private static int errorCode = 0;
        static int Main(string[] args)
        {
            MainAync().Wait();
            Console.ReadKey();
 
            return 0;
        } 
 
        static async Task MainAync()
        {
            string authContextURL = aadInstance + tenantId;
            AuthenticationContext authContext = new AuthenticationContext(authContextURL);
            X509Certificate2 cert = ReadCertificateFromStore(certName);
 
            ClientAssertionCertificate credential = new ClientAssertionCertificate(clientId, cert);
            AuthenticationResult result = await authContext.AcquireTokenAsync("https://management.azure.com/", credential);
            if (result == null)
            {
                throw new InvalidOperationException("Failed to obtain the JWT token");
            }
 
            string token = result.AccessToken;
            string subscriptionId = "<Your_Subscription_ID>";
            string rgName = "<ResourceGroup_of_your_Cosmos_account>";
            string accountName = "<Your_Cosmos_account_name>";
            string cosmosDBRestCall = $"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/listKeys?api-version=2015-04-08";
 
            Uri restCall = new Uri(cosmosDBRestCall);
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Remove("Authorization");
            httpClient.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
            HttpResponseMessage response = await httpClient.PostAsync(restCall, null);
 
            Console.WriteLine("Got result {0} and keys {1}", response.StatusCode.ToString(), response.Content.ReadAsStringAsync().Result);
        }
 
        /// <summary>
        /// Reads the certificate
        /// </summary>
        private static X509Certificate2 ReadCertificateFromStore(string certName)
        {
            X509Certificate2 cert = null;
            X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
            store.Open(OpenFlags.ReadOnly);
            X509Certificate2Collection certCollection = store.Certificates;
 
            // Find unexpired certificates.
            X509Certificate2Collection currentCerts = certCollection.Find(X509FindType.FindByTimeValid, DateTime.Now, false);
 
            // From the collection of unexpired certificates, find the ones with the correct name.
            X509Certificate2Collection signingCert = currentCerts.Find(X509FindType.FindBySubjectName, certName, false);
 
            // Return the first certificate in the collection, has the right name and is current.
            cert = signingCert.OfType<X509Certificate2>().OrderByDescending(c => c.NotBefore).FirstOrDefault();
            store.Close();
            return cert;
        }
    }
}
```

Ce script génère les clés primaire et secondaire, comme le montre la capture d’écran suivante :

:::image type="content" source="./media/certificate-based-authentication/csharp-application-output.png" alt-text="Sortie de l’application csharp":::

Tout comme dans la section précédente, vous pouvez consulter le journal d’activité de votre compte Azure Cosmos pour valider le déclenchement de l’événement de la requête d’obtention des clés par l’application « sampleApp ». 


## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser les clés Azure Cosmos à l’aide d’Azure Key Vault](access-secrets-from-keyvault.md)

* [Base de référence de sécurité pour Azure Cosmos DB](security-baseline.md)