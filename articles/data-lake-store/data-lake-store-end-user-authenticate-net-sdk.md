---
title: Authentification des utilisateurs finaux - .NET avec Data Lake Storage Gen1 - Azure
description: Découvrez comment authentifier les utilisateurs finaux auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory et du SDK .NET.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-csharp
ms.openlocfilehash: 67ba4f12aec9e987d79109b7197d03301bf40650
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89004780"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-net-sdk"></a>Authentification des utilisateurs finaux auprès d’Azure Data Lake Storage Gen1 à l’aide du SDK .NET
> [!div class="op_single_selector"]
> * [À l’aide de Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Utilisation du kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Utilisation de Python](data-lake-store-end-user-authenticate-python.md)
> * [Utilisation de l'API REST](data-lake-store-end-user-authenticate-rest-api.md)
> 
>  

Dans cet article, vous allez apprendre à utiliser le SDK .NET pour authentifier les utilisateurs finaux auprès d’Azure Data Lake Storage Gen1. Pour plus d’informations sur l’authentification de service à service auprès de Data Lake Storage Gen1 à l’aide du SDK .NET, voir [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide du SDK .NET](data-lake-store-service-to-service-authenticate-net-sdk.md).

## <a name="prerequisites"></a>Prérequis
* **Visual Studio 2013 ou version ultérieure**. Les instructions ci-dessous reposent sur Visual Studio 2019.

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Créez une application « native » Azure Active Directory**. Vous devez avoir suivi la procédure dans [End-user authentication with Data Lake Storage Gen1 using Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md) (Authentification des utilisateurs finaux avec Data Lake Storage Gen1 à l’aide d’Azure Active Directory).

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

5. Ouvrez **Program.cs**.
6. Remplacez les instructions using par les lignes suivantes :

    ```csharp
    using System;
    using System.IO;
    using System.Linq;
    using System.Text;
    using System.Threading;
    using System.Collections.Generic;
            
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.Management.DataLake.Store;
    using Microsoft.Azure.Management.DataLake.Store.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```     

## <a name="end-user-authentication"></a>Authentification des utilisateurs finaux
Ajoutez cet extrait de code dans votre application cliente .NET. Remplacez les valeurs d’espace réservé par les valeurs récupérées à partir d’une application native Azure AD (répertoriée comme prérequis). Cet extrait de code vous permet d’authentifier votre application **de manière interactive** auprès de Data Lake Storage Gen1, ce qui signifie que vous êtes invité à entrer vos informations d’identification Azure.

Pour simplifier l’utilisation, l’extrait de code suivant utilise les valeurs par défaut de l’ID client et l’URI de redirection qui sont valides avec n’importe quel abonnement Azure. Dans l’extrait de code suivant, il vous suffit de fournir la valeur de votre ID de locataire. Vous pouvez récupérer l’ID de locataire en suivant les instructions fournies dans la section [Obtenir l’ID de locataire](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    
- Remplacez la fonction Main() par le code suivant :

    ```csharp
    private static void Main(string[] args)
    {
        //User login via interactive popup
        string TENANT = "<AAD-directory-domain>";
        string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
        System.Uri ARM_TOKEN_AUDIENCE = new System.Uri(@"https://management.core.windows.net/");
        System.Uri ADL_TOKEN_AUDIENCE = new System.Uri(@"https://datalake.azure.net/");
        string MY_DOCUMENTS = System.Environment.GetFolderPath(System.Environment.SpecialFolder.MyDocuments);
        string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");
        var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
        var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
        var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
    }
    ```

Voici quelques informations utiles concernant l’extrait de code précédent :

* L’extrait de code précédent utilise les fonctions d’assistance `GetTokenCache` et `GetCreds_User_Popup`. Le code de ces fonctions d'assistance est disponible [ici sur GitHub](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options#gettokencache).
* Pour vous aider à effectuer le didacticiel plus rapidement, l’extrait de code utilise un ID client d’application native disponible par défaut pour tous les abonnements Azure. Vous pouvez donc **utiliser cet extrait de code en l’état dans votre application**.
* Cependant, si vous souhaitez utiliser votre propre ID client application et domaine Azure AD, vous devez créer une application native Azure AD, puis utiliser l’ID locataire, l’ID client et l’URI de redirection Azure AD de l’application que vous avez créée. Pour obtenir des instructions, voir la page [Créer une application Active Directory pour l’authentification des utilisateurs finaux auprès de Data Lake Storage Gen1](data-lake-store-end-user-authenticate-using-active-directory.md).

  
## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à vous servir de l’authentification des utilisateurs finaux auprès d’Azure Data Lake Storage Gen1 avec le SDK .NET. Vous pouvez maintenant consulter les articles ci-après qui expliquent comment utiliser le SDK .NET pour travailler avec Azure Data Lake Storage Gen1.

* [Opérations de gestion du compte sur Data Lake Storage Gen1 à l’aide du SDK .NET](data-lake-store-get-started-net-sdk.md)
* [Opérations sur les données dans Data Lake Storage Gen1 à l’aide du SDK .NET](data-lake-store-data-operations-net-sdk.md)

