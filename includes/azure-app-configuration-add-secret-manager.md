---
ms.openlocfilehash: c3c5b8ef94b507cad433e587c9ebfc2ec16c0ff9
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91440414"
---
## <a name="add-secret-manager"></a>Ajouter Secret Manager

Un outil appelé Secret Manager stocke les données sensibles pour les travaux de développement à l’extérieur de l’arborescence de votre projet. Cette approche empêche le partage accidentel des secrets d’une application au sein du code source. Procédez comme suit pour activer l’utilisation de Secret Manager dans le projet ASP.NET Core :

#### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

Accédez au répertoire racine du projet, puis exécutez la commande suivante pour activer le stockage des secrets dans le projet :

```dotnetcli
dotnet user-secrets init
```

Un élément `UserSecretsId` contenant un GUID est ajouté au fichier *.csproj* :

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    
    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
    </PropertyGroup>

</Project>
```

#### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

1. Ouvrez le fichier  *.csproj*.

1. Ajoutez un élément `UserSecretsId` au fichier *.csproj* comme indiqué ici. Vous pouvez utiliser le même GUID ou remplacer cette valeur par votre propre valeur.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">
    
        <PropertyGroup>
            <TargetFramework>netcoreapp2.1</TargetFramework>
            <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
        </PropertyGroup>
    
        <ItemGroup>
            <PackageReference Include="Microsoft.AspNetCore.App" />
            <PackageReference Include="Microsoft.AspNetCore.Razor.Design" Version="2.1.2" PrivateAssets="All" />
        </ItemGroup>
    
    </Project>
    ```
    
1. Enregistrez le fichier  *.csproj*.

---

> [!TIP]
> Pour en savoir plus sur Secret Manager, consultez [Stockage sécurisé des secrets d’application en développement dans ASP.NET Core](/aspnet/core/security/app-secrets).
