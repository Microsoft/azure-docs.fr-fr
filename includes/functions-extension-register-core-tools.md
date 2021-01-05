---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 08/07/2020
ms.author: glenga
ms.openlocfilehash: a4f03223b5067d18f5d6e747b3bb630a86031b8f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020344"
---
Si vous ne pouvez pas utiliser les packs d’extensions, vous pouvez utiliser Azure Functions Core Tools localement pour installer les packs d’extensions spécifiques requis par votre projet.

> [!IMPORTANT]
> Vous ne pouvez pas installer explicitement des extensions dans une application de fonction qui utilise des bundles d’extension. Supprimez la section `extensionBundle` dans *host.json* avant d’installer des extensions de manière explicite.

Les éléments suivants décrivent certaines raisons pour lesquelles vous devrez peut-être installer des extensions manuellement :

* Vous devez accéder à une version spécifique d’une extension qui n’est pas disponible dans un bundle.
* Vous devez accéder à une extension personnalisée qui n’est pas disponible dans un bundle.
* Vous devez accéder à une combinaison spécifique d’extensions non disponibles dans un seul bundle.

> [!NOTE]
> Pour installer manuellement des extensions avec Core Tools, le [kit SDK .NET Core 2.x](https://dotnet.microsoft.com/download) doit être installé. Azure Functions Core Tools installe le kit SDK .NET Core pour installer des extensions à partir de NuGet. Vous n’avez pas besoin de connaître .NET pour utiliser les extensions Azure Functions.

Lorsque vous installez explicitement des extensions, un fichier de projet .NET nommé extensions.csproj est ajouté à la racine de votre projet. Ce fichier définit l’ensemble des packages NuGet requis par vos fonctions. Bien que vous puissiez utiliser les [références de package NuGet](/nuget/consume-packages/package-references-in-project-files) dans ce fichier, Core Tools vous permet d’installer des extensions sans avoir à modifier manuellement le fichier.

Il existe plusieurs façons d’utiliser les outils de base pour installer les extensions requises dans votre projet local. 

#### <a name="install-all-extensions"></a>Installer toutes les extensions 

Utilisez la commande suivante pour ajouter automatiquement tous les packages d’extensions utilisés par les liaisons dans votre projet local :

```dotnetcli
func extensions install
```
La commande lit le fichier *function.json* pour voir les packages dont vous avez besoin, les installe et regénère le projet des extensions (extensions.csproj). Il ajoute les nouvelles liaisons à la version actuelle, mais ne met pas à jour les liaisons existantes. Utilisez l’option `--force` pour mettre à jour les liaisons existantes vers la dernière version pendant les nouvelles installations.

Si votre application de fonction utilise des liaisons que les outils Core ne reconnaissent pas, vous devez installer manuellement l’extension spécifique.

#### <a name="install-a-specific-extension"></a>Installer une extension spécifique

Utilisez la commande suivante pour installer un package d’extension spécifique à une version spécifique, dans ce cas l’extension Storage :

```dotnetcli
func extensions install --package Microsoft.Azure.WebJobs.Extensions.Storage --version 4.0.2
```
