---
title: Ajouter la prise en charge de Key Vault à votre projet ASP.NET à l’aide de Visual Studio - Azure Key Vault | Microsoft Docs
description: Ce tutoriel vous montre comment ajouter la prise en charge de Key Vault à une application web ASP.NET ou ASP.NET Core.
services: key-vault
author: ghogen
manager: jillfra
ms.service: key-vault
ms.custom: vs-azure, devx-track-csharp
ms.topic: how-to
ms.date: 08/07/2019
ms.author: ghogen
ms.openlocfilehash: 9c62534acdbfbff7fd4e718bad1f07a92c641626
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792393"
---
# <a name="add-key-vault-to-your-web-application-by-using-visual-studio-connected-services"></a>Ajouter Key Vault à votre application web à l’aide des Services connectés Visual Studio

Dans ce tutoriel, vous allez voir comment ajouter facilement tout ce dont vous avez besoin pour utiliser Azure Key Vault en vue de gérer les secrets de vos projets web dans Visual Studio, que vous utilisiez ASP.NET Core ou tout type de projet ASP.NET. Quand vous utilisez la fonctionnalité Services connectés de Visual Studio, Visual Studio peut ajouter automatiquement tous les packages NuGet et les paramètres de configuration dont vous avez besoin pour vous connecter à Key Vault dans Azure.

Pour plus d’informations sur les changements qu’apportent les Services connectés à votre projet pour l’activation de Key Vault, consultez [Key Vault Connected Service - What happened to my ASP.NET 4.7.1 project](#how-your-aspnet-framework-project-is-modified) ou [Key Vault Connected Service - What happened to my ASP.NET Core project](#how-your-aspnet-core-project-is-modified).

## <a name="prerequisites"></a>Prérequis

- **Un abonnement Azure** . Si vous n’avez pas d’abonnement, inscrivez-vous pour bénéficier d’un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).
- **Visual Studio 2019, version 16.3** ou ultérieure [Télécharger le maintenant](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs).


## <a name="add-key-vault-support-to-your-project"></a>Ajouter la prise en charge Key Vault à votre projet

Avant de commencer, vérifiez que vous êtes connecté à Visual Studio. Connectez-vous avec le même compte que celui que vous utilisez pour votre abonnement Azure. Ouvrez ensuite un projet web ASP.NET 4.7.1 ou version ultérieure, ou ASP.NET Core 2.0, puis procédez comme suit :

1. Dans **l’Explorateur de solutions** , cliquez avec le bouton de droite sur le projet auquel vous souhaitez ajouter le support Key Vault, puis choisissez **Ajouter** > **Service connecté** > **Ajouter** .
   La page Service connecté s’affiche et montre des services que vous pouvez ajouter à votre projet.
1. Dans le menu des services disponibles, choisissez **Azure Key Vault** et cliquez sur **Suivant** .

   ![Choisir « Azure Key Vault »](../media/vs-key-vault-add-connected-service/key-vault-connected-service.png)

1. Sélectionnez l’abonnement que vous souhaitez utiliser, puis choisissez un Key Vault existant et cliquez sur **Terminer** . 

   ![Sélectionnez votre abonnement](../media/vs-key-vault-add-connected-service/key-vault-connected-service-select-vault.png)

À présent, la connexion à Key Vault est établie et vous pouvez accéder à vos secrets dans le code. Les étapes suivantes sont différentes selon que vous utilisez ASP.NET 4.7.1 ou ASP.NET Core.

## <a name="access-your-secrets-in-code-aspnet-core"></a>Accéder à vos secrets dans le code (ASP.NET Core)

1. Ouvrez l’un des fichiers de page, par exemple *Index.cshtml.cs* et écrivez le code suivant :
   1. Incluez une référence à `Microsoft.Extensions.Configuration` à l’aide de cette directive :

       ```csharp
       using Microsoft.Extensions.Configuration;
       ```

   1. Ajoutez la variable de configuration.

      ```csharp
      private static IConfiguration _configuration;
      ```

   1. Ajoutez ce constructeur ou remplacez le constructeur existant par ce qui suit :

       ```csharp
       public IndexModel(IConfiguration configuration)
       {
           _configuration = configuration;
       }
       ```

   1. Mettez à jour la méthode `OnGet`. Mettez à jour la valeur d’espace réservé indiquée ici avec le nom de secret que vous avez créé dans les commandes ci-dessus.

       ```csharp
       public void OnGet()
       {
           ViewData["Message"] = "My key val = " + _configuration["<YourSecretNameStoredInKeyVault>"];
       }
       ```

   1. Pour confirmer la valeur au moment de l’exécution, ajoutez du code pour afficher `ViewData["Message"]` sur le fichier *.cshtml* pour afficher le secret dans un message.

      ```cshtml
          <p>@ViewData["Message"]</p>
      ```

Vous pouvez exécuter l’application localement pour vérifier que le secret est correctement obtenu à partir de Key Vault.

## <a name="access-your-secrets-aspnet"></a>Accéder à vos secrets (ASP.NET)
Vous pouvez paramétrer la configuration de sorte que le fichier web.config ait une valeur factice dans l’élément `appSettings` qui est remplacé par la valeur true au moment de l’exécution. Vous pouvez ensuite y accéder via la structure de données `ConfigurationManager.AppSettings`.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis sélectionnez Gérer les packages NuGet. Sous l’onglet Parcourir, recherchez et installez [Microsoft. Configuration.ConfigurationBuilders.Azure](https://www.nuget.org/packages/Microsoft.Configuration.ConfigurationBuilders.Azure/)
 
1. Ouvrez votre fichier config.web et écrivez le code suivant :
    1. Ajoutez `configSections` et `configBuilders` :
        ```xml
         <configSections>
            <section
                name="configBuilders"
                type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
                restartOnExternalChanges="false"
                requirePermission="false" />
         </configSections>
         <configBuilders>
            <builders>
            <add
                    name="AzureKeyVault"
                    vaultName="vaultname"
                    type="Microsoft.Configuration.ConfigurationBuilders.AzureKeyVaultConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Azure, Version=1.0.0.0, Culture=neutral"
                    vaultUri="https://vaultname.vault.azure.net" />
            </builders>
         </configBuilders>
        ```
    1. Recherchez la balise appSettings, ajoutez un attribut `configBuilders="AzureKeyVault"`et ajoutez une ligne :
        ```xml
         <add key="<secretNameInYourKeyVault>" value="dummy"/>
        ```

1. Modifiez la méthode `About` dans *HomeController.cs* pour afficher la valeur de confirmation.

   ```csharp
   public ActionResult About()
   {
       ViewBag.Message = "Key vault value = " + ConfigurationManager.AppSettings["<secretNameInYourKeyVault>"];
   }
   ```
1. Exécutez l’application localement sous le débogueur, basculez dans l’onglet **À propos** et vérifiez que la valeur provenant de Key Vault est affichée.

## <a name="troubleshooting"></a>Dépannage

Si votre Key Vault s’exécute sur un autre compte Microsoft que celui auquel vous êtes connecté à Visual Studio (par exemple, Key Vault s’exécute sur votre compte professionnel, alors que Visual Studio utilise votre compte privé), un message d’erreur indiquant que Visual Studio ne peut pas avoir accès à Key Vault est ajouté à votre fichier Program.cs. Pour résoudre ce problème :

1. Accédez au [Portail Azure](https://portal.azure.com) et accédez à votre coffre de clés.

1. Choisissez **Stratégies d’accès** , puis **Ajouter une stratégie d’accès** et choisissez le compte avec lequel vous êtes connecté en tant que principal.

1. Dans Visual Studio, choisissez **Fichier** > **Paramètres du compte** .
Sélectionnez **Ajouter un compte** dans la section **Tous les comptes** . Connectez-vous avec le compte que vous avez choisi comme principal de votre stratégie d’accès.

1. Choisissez **Outils** > **Options** et recherchez **Authentification du service Azure** . Sélectionnez ensuite le compte que vous venez d’ajouter à Visual Studio.

Désormais, quand vous déboguez votre application, Visual Studio se connecte au compte sur lequel se trouve votre Key Vault.

## <a name="how-your-aspnet-core-project-is-modified"></a>Modifications apportées à votre projet ASP.NET Core

Cette section identifie les changements exacts apportés à un projet ASP.NET lors de l’ajout du Service connecté Key Vault avec Visual Studio.

### <a name="added-references-for-aspnet-core"></a>Références ajoutées pour ASP.NET Core

Affecte les références .NET du fichier projet et les références de package NuGet.

| Type | Informations de référence |
| --- | --- |
| NuGet | Microsoft.AspNetCore.AzureKeyVault.HostingStartup |

### <a name="added-files-for-aspnet-core"></a>Fichiers ajoutés pour ASP.NET Core

- Ajout de `ConnectedService.json`, qui enregistre des informations sur le fournisseur et la version du service connecté, ainsi qu’un lien vers la documentation.

### <a name="project-file-changes-for-aspnet-core"></a>Modifications apportées au fichier projet pour ASP.NET Core

- Ajout de l’ItemGroup des services connectés et du fichier `ConnectedServices.json`.

### <a name="launchsettingsjson-changes-for-aspnet-core"></a>Modifications de launchsettings.json pour ASP.NET Core

- Ajout des entrées de variables d’environnement suivantes au profil IIS Express et au profil qui correspond au nom de votre projet web :

    ```json
      "environmentVariables": {
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONENABLED": "true",
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONVAULT": "<your keyvault URL>"
      }
    ```

### <a name="changes-on-azure-for-aspnet-core"></a>Modifications apportées à Azure pour ASP.NET Core

- Création d’un groupe de ressources (ou utilisation d’un groupe existant).
- Création d’un coffre de clés dans le groupe de ressources spécifié.

## <a name="how-your-aspnet-framework-project-is-modified"></a>Modifications apportées à votre projet ASP.NET Framework

Cette section identifie les changements exacts apportés à un projet ASP.NET lors de l’ajout du Service connecté Key Vault avec Visual Studio.

### <a name="added-references-for-aspnet-framework"></a>Références ajoutées pour ASP.NET Framework

Affecte les références .NET du fichier projet et `packages.config` (références NuGet).

| Type | Informations de référence |
| --- | --- |
| .NET ; NuGet | Azure.Identity |
| .NET ; NuGet | Azure.Security.KeyVault.Keys |
| .NET ; NuGet | Azure.Security.KeyVault.Secrets |

> [!IMPORTANT] 
> Par défaut, Azure.Identity 1.1.1 est installé, qui ne prend pas en charge les informations d’identification de Visual Studio. Vous pouvez mettre à jour manuellement la référence du package vers 1.2 et utiliser les informations d’identification de Visual Studio.

### <a name="added-files-for-aspnet-framework"></a>Fichiers ajoutés pour ASP.NET Framework

- Ajout de `ConnectedService.json`, qui enregistre des informations sur le fournisseur et la version du service connecté, ainsi qu’un lien vers la documentation.

### <a name="project-file-changes-for-aspnet-framework"></a>Modifications apportées au fichier projet pour ASP.NET Framework

- Ajout de l’ItemGroup des services connectés et du fichier ConnectedServices.json.
- Références aux assemblys .NET décrits dans la section [Références ajoutées](#added-references-for-aspnet-framework).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez suivi ce tutoriel, vos autorisations Key Vault sont configurées pour s’exécuter avec votre propre abonnement Azure, mais cela peut ne pas être souhaitable pour un scénario de production. Vous pouvez créer une identité managée afin de gérer l’accès à Key Vault pour votre application. Consultez [Comment s’authentifier auprès de Key Vault](./authentication.md) et [Attribuer une stratégie d’accès Key Vault](./assign-access-policy-portal.md).

Découvrez plus en détail le développement de coffres de clés en lisant le [Guide du développeur de coffre de clés](developers-guide.md).