---
title: .NET – Authentification de service à service – Data Lake Storage Gen1
description: Découvrez comment procéder à une authentification de service à service auprès d’Azure Data Lake Storage Gen1 en utilisant Azure Active Directory et le Kit de développement logiciel (SDK) .NET.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 19b4ac619ec3e72c787efc8e9f043f42dbd8b09b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010297"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-net-sdk"></a>Authentification de service à service auprès d’Azure Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) .NET
> [!div class="op_single_selector"]
> * [À l’aide de Java](data-lake-store-service-to-service-authenticate-java.md)
> * [Utilisation du kit de développement logiciel (SDK) .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [Utilisation de Python](data-lake-store-service-to-service-authenticate-python.md)
> * [Utilisation de l'API REST](data-lake-store-service-to-service-authenticate-rest-api.md)
>
>

Dans cet article, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) .NET pour effectuer une authentification de service à service auprès d’Azure Data Lake Storage Gen1. Pour plus d’informations sur l’authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 à l’aide du Kit de développement logiciel (SDK) .NET, consultez la rubrique [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec le Kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-net-sdk.md).

## <a name="prerequisites"></a>Prérequis
* **Visual Studio 2013 ou version ultérieure**. Les instructions ci-dessous reposent sur Visual Studio 2019.

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Créez une application « web » Azure Active Directory**. Vous devez avoir suivi la procédure indiquée dans [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md).

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

## <a name="service-to-service-authentication-with-client-secret"></a>Authentification de service à service avec une clé secrète client
Ajoutez cet extrait de code dans votre application cliente .NET. Remplacez les valeurs d’espace réservé par les valeurs récupérées à partir d’une application web Azure AD (répertoriée comme condition préalable). Cet extrait de code permet d’authentifier votre application **de façon non interactive** auprès de Data Lake Storage Gen1 à l’aide de la clé secrète client/clé de l’application web Azure AD.

```csharp
private static void Main(string[] args)
{
    // Service principal / application authentication with client secret / key
    // Use the client ID of an existing AAD "Web App" application.
    string TENANT = "<AAD-directory-domain>";
    string CLIENTID = "<AAD_WEB_APP_CLIENT_ID>";
    System.Uri ARM_TOKEN_AUDIENCE = new System.Uri(@"https://management.core.windows.net/");
    System.Uri ADL_TOKEN_AUDIENCE = new System.Uri(@"https://datalake.azure.net/");
    string secret_key = "<AAD_WEB_APP_SECRET_KEY>";
    var armCreds = GetCreds_SPI_SecretKey(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, secret_key);
    var adlCreds = GetCreds_SPI_SecretKey(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, secret_key);
}
```

L’extrait de code précédent utilise la fonction d’assistance `GetCreds_SPI_SecretKey`. Le code de cette fonction d’assistance est disponible [ici sur GitHub](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options#getcreds_spi_secretkey).

## <a name="service-to-service-authentication-with-certificate"></a>Authentification de service à service avec un certificat

Ajoutez cet extrait de code dans votre application cliente .NET. Remplacez les valeurs d’espace réservé par les valeurs récupérées à partir d’une application web Azure AD (répertoriée comme condition préalable). Cet extrait de code permet d’authentifier votre application **de façon non interactive** auprès de Data Lake Storage Gen1 à l’aide du certificat d’une application web Azure AD. Pour plus d’informations sur la création d’une application Azure AD, consultez [Créer un principal du service avec un certificat](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate).

```csharp
private static void Main(string[] args)
{
    // Service principal / application authentication with certificate
    // Use the client ID and certificate of an existing AAD "Web App" application.
    string TENANT = "<AAD-directory-domain>";
    string CLIENTID = "<AAD_WEB_APP_CLIENT_ID>";
    System.Uri ARM_TOKEN_AUDIENCE = new System.Uri(@"https://management.core.windows.net/");
    System.Uri ADL_TOKEN_AUDIENCE = new System.Uri(@"https://datalake.azure.net/");
    var cert = new X509Certificate2(@"d:\cert.pfx", "<certpassword>");
    var armCreds = GetCreds_SPI_Cert(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, cert);
    var adlCreds = GetCreds_SPI_Cert(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, cert);
}
```

L’extrait de code précédent utilise la fonction d’assistance `GetCreds_SPI_Cert`. Le code de cette fonction d’assistance est disponible [ici sur GitHub](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options#getcreds_spi_cert).

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à utiliser l’authentification de service à service auprès d’Azure Data Lake Storage Gen1 avec le Kit de développement logiciel (SDK) .NET. Vous pouvez à présent consulter les articles ci-après, qui expliquent comment utiliser le Kit de développement logiciel (SDK) .NET pour travailler avec Data Lake Storage Gen1.

* [Opérations de gestion du compte sur Data Lake Storage Gen1 à l’aide du SDK .NET](data-lake-store-get-started-net-sdk.md)
* [Opérations sur les données dans Data Lake Storage Gen1 à l’aide du SDK .NET](data-lake-store-data-operations-net-sdk.md)
