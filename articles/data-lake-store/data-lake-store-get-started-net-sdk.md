---
title: Gérer un compte Azure Data Lake Storage Gen1 avec .NET
description: Découvrez comment utiliser le Lit de développement logiciel (SDK) .NET pour les opérations de gestion de compte d’Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-csharp
ms.openlocfilehash: c5b2364328fc4e0103b969f94a4c877a388edb7c
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103609"
---
# <a name="account-management-operations-on-azure-data-lake-storage-gen1-using-net-sdk"></a>Opérations de gestion des comptes sur Azure Data Lake Storage Gen1 à l’aide du SDK .NET
> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET](data-lake-store-get-started-net-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

Dans cet article, vous allez découvrir comment réaliser des opérations de gestion des comptes sur Azure Data Lake Storage Gen1 avec le Kit de développement logiciel (SDK) .NET. Les opérations de gestion des comptes incluent la création d’un compte Data Lake Storage Gen1, la création d’une liste des comptes dans un abonnement Azure, la suppression de comptes, etc.

Pour obtenir des instructions sur l’exécution des opérations de gestion des données sur Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET, consultez la section relative aux [opérations du système de fichiers sur Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-data-operations-net-sdk.md).

## <a name="prerequisites"></a>Prérequis
* **Visual Studio 2013 ou version ultérieure**. Les instructions ci-dessous reposent sur Visual Studio 2019.

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="create-a-net-application"></a>Créer une application .NET
1. Dans Visual Studio, sélectionnez le menu **Fichier**, puis **Nouveau** et **Projet**.
2. Choisissez **Application console (.NET Framework)** , puis sélectionnez **Suivant**.
3. Dans **Nom du projet**, entrez `CreateADLApplication`, puis sélectionnez **Créer**.

4. Ajoutez les packages NuGet à votre projet.

   1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nom du projet, puis cliquez sur **Gérer les packages NuGet**.
   2. Dans l’onglet **Gestionnaire de package NuGet**, vérifiez que **Source du package** a la valeur **nuget.org** et que la case **Inclure la version préliminaire** est cochée.
   3. Recherchez et installez les packages NuGet suivants :

      * `Microsoft.Azure.Management.DataLake.Store` - Ce didacticiel utilise v2.1.3-preview.
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` - Ce didacticiel utilise v2.2.12.

        ![Ajouter une source NuGet](./media/data-lake-store-get-started-net-sdk/data-lake-store-install-nuget-package.png "Créer un compte Azure Data Lake")
   4. Fermez le **Gestionnaire de package NuGet**.
5. Ouvrez **Program.cs**, supprimez le code existant, puis insérez les instructions suivantes pour ajouter des références aux espaces de noms.

    ```csharp
    using System;
    using System.IO;
    using System.Linq;
    using System.Text;
    using System.Threading;
    using System.Collections.Generic;
    using System.Security.Cryptography.X509Certificates; // Required only if you are using an Azure AD application created with certificates
                
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.Management.DataLake.Store;
    using Microsoft.Azure.Management.DataLake.Store.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

6. Déclarez les variables et fournissez les valeurs des espaces réservés. En outre, assurez-vous que le chemin d’accès local et le nom de fichier que vous fournissez existent sur l’ordinateur.

    ```csharp
    namespace SdkSample
    {
        class Program
        {
            private static DataLakeStoreAccountManagementClient _adlsClient;
                
            private static string _adlsAccountName;
            private static string _resourceGroupName;
            private static string _location;
            private static string _subId;

            private static void Main(string[] args)
            {
                _adlsAccountName = "<DATA-LAKE-STORAGE-GEN1-NAME>.azuredatalakestore.net"; 
                _resourceGroupName = "<RESOURCE-GROUP-NAME>"; 
                _location = "East US 2";
                _subId = "<SUBSCRIPTION-ID>";                    
            }
        }
    }
    ```

Dans les sections suivantes de cet article, vous pouvez découvrir comment utiliser les méthodes .NET pour effectuer des opérations telles que l’authentification des utilisateurs et le chargement de fichiers.

## <a name="authentication"></a>Authentification

* Pour en savoir plus sur l’authentification des utilisateurs accédant à votre application, consultez la section relative à [l’authentification de l’utilisateur avec Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-net-sdk.md).
* Pour en savoir plus sur l’authentification entre les services dans le cadre de votre application, consultez la section relative à [l’authentification entre les services avec Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) .NET](data-lake-store-service-to-service-authenticate-net-sdk.md).

## <a name="create-client-object"></a>Créer un objet client
L’extrait de code suivant crée l’objet client du compte Data Lake Storage Gen1, utilisé pour émettre des demandes de gestion de compte auprès du service, pour créer un compte, en supprimer un, etc.

```csharp
// Create client objects and set the subscription ID
_adlsClient = new DataLakeStoreAccountManagementClient(armCreds) { SubscriptionId = _subId };
```
    
## <a name="create-a-data-lake-storage-gen1-account"></a>Créer un compte Data Lake Storage Gen1
L’extrait de code suivant crée un compte Data Lake Storage Gen1 dans l’abonnement Azure que vous avez fourni lors de la création de l’objet client du compte Data Lake Storage Gen1.

```csharp
// Create Data Lake Storage Gen1 account
var adlsParameters = new DataLakeStoreAccount(location: _location);
_adlsClient.Account.Create(_resourceGroupName, _adlsAccountName, adlsParameters);
```

## <a name="list-all-data-lake-storage-gen1-accounts-within-a-subscription"></a>Répertorier tous les comptes Data Lake Storage Gen1 d’un abonnement
Ajoutez la méthode suivante à votre définition de classe. L’extrait de code suivant répertorie tous les comptes Data Lake Storage Gen1 d’un abonnement Azure donné.

```csharp
// List all Data Lake Storage Gen1 accounts within the subscription
public static List<DataLakeStoreAccountBasic> ListAdlStoreAccounts()
{
    var response = _adlsClient.Account.List(_adlsAccountName);
    var accounts = new List<DataLakeStoreAccountBasic>(response);

    while (response.NextPageLink != null)
    {
        response = _adlsClient.Account.ListNext(response.NextPageLink);
        accounts.AddRange(response);
    }

    return accounts;
}
```

## <a name="delete-a-data-lake-storage-gen1-account"></a>Supprimer un compte Data Lake Storage Gen1
L’extrait de code suivant supprime le compte Data Lake Storage Gen1 que vous avez créé précédemment.

```csharp
// Delete Data Lake Storage Gen1 account
_adlsClient.Account.Delete(_resourceGroupName, _adlsAccountName);
```

## <a name="see-also"></a>Voir aussi
* [Opérations de système de fichiers dans Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) .NET](data-lake-store-data-operations-net-sdk.md)
* [Référence SDK .NET Data Lake Storage Gen1](/dotnet/api/overview/azure/data-lake-store?view=azure-dotnet)

## <a name="next-steps"></a>Étapes suivantes
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)