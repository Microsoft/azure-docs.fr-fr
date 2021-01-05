---
title: Configurer Azure Functions
description: Ce tutoriel vous explique comment créer une application de fonction Azure et comment la configurer pour qu’elle fonctionne avec des fournisseurs personnalisés Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: 6e11e08d861538ddf395f522495798546e04ca7c
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91948950"
---
# <a name="set-up-azure-functions-for-azure-custom-providers"></a>Configurer Azure Functions pour les fournisseurs personnalisés Azure

Un fournisseur personnalisé implique qu’un contrat soit passé entre Azure et un point de terminaison. Les fournisseurs personnalisés vous permettent de modifier des workflows dans Azure. Ce tutoriel montre comment configurer une application de fonction Azure pour qu’elle fonctionne comme un point de terminaison de fournisseur personnalisé.

## <a name="create-the-azure-function-app"></a>Créer une application de fonction Azure

> [!NOTE]
> Dans ce tutoriel, vous allez créer un point de terminaison de service simple qui utilise une application de fonction Azure. Toutefois, un fournisseur personnalisé peut utiliser n’importe quel point de terminaison accessible publiquement. Les alternatives sont notamment Azure Logic Apps, la gestion des API Azure et la fonctionnalité Web Apps d’Azure App Service.

Pour démarrer ce tutoriel, vous devez d’abord suivre le tutoriel [Créer votre première application de fonction à l’aide du portail Azure](../../azure-functions/functions-create-first-azure-function.md). Ce tutoriel crée une fonction webhook .NET Core qui peut être modifiée dans le portail Azure. C’est également la base du tutoriel actuel.

## <a name="install-azure-table-storage-bindings"></a>Installer les liaisons du stockage Table Azure

Pour installer les liaisons du stockage Table Azure :

1. Accédez à l’onglet **Intégrer** pour HttpTrigger.
1. Sélectionnez **+Nouvelle entrée**.
1. Sélectionnez **Stockage Table Azure**.
1. Installez l’extension Microsoft.Azure.WebJobs.Extensions.Storage si celle-ci n’est pas déjà installée.
1. Dans la zone **Nom du paramètre de table**, entrez **tableStorage**.
1. Dans la zone**Nom de la table**, entrez **myCustomResources**.
1. Sélectionnez **Enregistrer** pour enregistrer le paramètre d’entrée mis à jour.

![Vue d’ensemble du fournisseur personnalisé montrant les liaisons de table](./media/create-custom-provider/azure-functions-table-bindings.png)

## <a name="update-restful-http-methods"></a>Mettre à jour les méthodes HTTP RESTful

Pour configurer la fonction Azure de sorte qu’elle comprenne les méthodes de requête RESTful du fournisseur personnalisé :

1. Accédez à l’onglet **Intégrer** pour HttpTrigger.
1. Sous **Méthodes HTTP sélectionnées**, sélectionnez **GET**, **POST**, **DELETE** et **PUT**.

![Vue d’ensemble du fournisseur personnalisé montrant les méthodes HTTP](./media/create-custom-provider/azure-functions-http-methods.png)

## <a name="add-azure-resource-manager-nuget-packages"></a>Ajouter des packages NuGet Azure Resource Manager

> [!NOTE]
> Si votre fichier projet C# ne se trouve pas dans le répertoire du projet, vous pouvez l’y ajouter manuellement. Sinon, il apparaîtra après l’installation de l’extension Microsoft.Azure.WebJobs.Extensions.Storage dans l’application de fonction.

Ensuite, mettez à jour le fichier projet C# en y ajoutant des bibliothèques NuGet. Ces bibliothèques facilitent l’analyse des requêtes entrantes envoyées par les fournisseurs personnalisés. Suivez les étapes pour [ajouter des extensions à partir du portail](../../azure-functions/functions-bindings-register.md) et mettez à jour le fichier projet C# pour y ajouter les références de package suivantes :

```xml
<PackageReference Include="Microsoft.Azure.WebJobs.Extensions.Storage" Version="3.0.4" />
<PackageReference Include="Microsoft.Azure.Management.ResourceManager.Fluent" Version="1.22.2" />
<PackageReference Include="Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator" Version="1.1.*" />
```

L’élément XML suivant est un exemple de fichier projet C# :

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <WarningsAsErrors />
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.WebJobs.Extensions.Storage" Version="3.0.4" />
    <PackageReference Include="Microsoft.Azure.Management.ResourceManager.Fluent" Version="1.22.2" />
    <PackageReference Include="Microsoft.Azure.WebJobs.Script.ExtensionsMetadataGenerator" Version="1.1.*" />
  </ItemGroup>
</Project>
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré une application de fonction Azure pour qu’elle fonctionne comme un point de terminaison de fournisseur personnalisé Azure.

Pour savoir comment créer un point de terminaison de fournisseur personnalisé RESTful, consultez [Tutoriel : Création d’un point de terminaison de fournisseur personnalisé RESTful](./tutorial-custom-providers-function-authoring.md).