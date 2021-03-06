---
ms.openlocfilehash: b1be5e903994f0a2c4a3d457f0c2bb5572a889c3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/29/2021
ms.locfileid: "98663027"
---
## <a name="add-secret-manager"></a>Ajouter Secret Manager

Un outil appelé Secret Manager stocke les données sensibles pour les travaux de développement à l’extérieur de l’arborescence de votre projet. Cette approche empêche le partage accidentel des secrets d’une application au sein du code source. Procédez comme suit pour activer l’utilisation de Secret Manager dans le projet ASP.NET Core :

#### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

Accédez au répertoire racine du projet, puis exécutez la commande suivante pour activer le stockage des secrets dans le projet :

```dotnetcli
dotnet user-secrets init
```

Un élément `UserSecretsId` contenant un GUID est ajouté au fichier *.csproj* :

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    
    <PropertyGroup>
        <TargetFramework>net5.0</TargetFramework>
        <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
    </PropertyGroup>

</Project>
```

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
