---
title: Cloner une application avec PowerShell
description: Découvrez comment cloner votre application App Service vers une nouvelle application à l’aide de PowerShell. Différents scénarios de clonage sont abordés, dont celui de l’intégration de Traffic Manager.
ms.assetid: f9a5cfa1-fbb0-41e6-95d1-75d457347a35
ms.topic: article
ms.date: 01/14/2016
ms.custom: seodec18
ms.openlocfilehash: 92e893fe9f74d51856faf39d3e4800dd5a8155db
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88815383"
---
# <a name="azure-app-service-app-cloning-using-powershell"></a>Clonage de l’application Azure App Service à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avec la publication de Microsoft Azure PowerShell version 1.1.0, une nouvelle option a été ajoutée à `New-AzWebApp`, qui permet à l’utilisateur de cloner une application App Service existante vers une application qui vient d’être créée dans une autre région ou dans la même région. Cette option permet aux utilisateurs de déployer de nombreuses applications dans différentes régions.

Le clonage d’application est pris en charge par les plans App Service Standard, Premium, Premium V2 et Isolé. Cette nouvelle fonctionnalité utilise les mêmes limitations que la fonctionnalité de sauvegarde App Service. Consultez [Sauvegarder une application dans Azure App Service](manage-backup.md).

## <a name="cloning-an-existing-app"></a>Clonage d’une application existante
Scénario : Vous souhaitez cloner le contenu d’une application de la région USA Centre Sud vers une nouvelle application de la région USA Centre Nord. Cela peut être effectué à l’aide de la version Azure Resource Manager de l’applet de commande PowerShell pour créer une application avec l’option `-SourceWebApp`.

En connaissant le nom du groupe de ressources qui contient l’application source, vous pouvez utiliser la commande PowerShell suivante pour obtenir les informations de l’application source (dans ce cas nommée `source-webapp`) :

```powershell
$srcapp = Get-AzWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

Pour créer un plan App Service, vous pouvez utiliser la commande `New-AzAppServicePlan` comme dans l’exemple suivant

```powershell
New-AzAppServicePlan -Location "North Central US" -ResourceGroupName DestinationAzureResourceGroup -Name DestinationAppServicePlan -Tier Standard
```

À l’aide de la commande `New-AzWebApp`, vous pouvez créer la nouvelle application dans la région USA Centre Nord et la lier à un plan App Service existant. Par ailleurs, vous pouvez utiliser le même groupe de ressources que l’application source ou en définir un nouveau, comme le montre la commande suivante :

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp
```

Pour cloner une application existante, notamment tous les emplacements de déploiement associés, vous devez utiliser le paramètre `IncludeSourceWebAppSlots`.  Notez que le paramètre `IncludeSourceWebAppSlots` est pris en charge uniquement pour le clonage d’une application entière, y compris tous ses emplacements. La commande PowerShell suivante illustre l’utilisation de ce paramètre avec la commande `New-AzWebApp` :

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots
```

Pour cloner une application existante dans la même région, vous devez créer un groupe de ressources et un plan de service d’application dans la même région, puis utiliser la commande PowerShell suivante pour cloner l’application :

```powershell
$destapp = New-AzWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan NewAppServicePlan -SourceWebApp $srcapp
```

## <a name="cloning-an-existing-app-to-an-app-service-environment"></a>Clonage d’une application existante vers un App Service Environment
Scénario : Vous souhaitez cloner le contenu d’une application de la région USA Centre Sud vers une nouvelle application d’un environnement App Service Environment (ASE) existant.

En connaissant le nom du groupe de ressources qui contient l’application source, vous pouvez utiliser la commande PowerShell suivante pour obtenir les informations de l’application source (dans ce cas nommée `source-webapp`) :

```powershell
$srcapp = Get-AzWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

En connaissant le nom de l’environnement ASE et le nom du groupe de ressources auquel l’ASE appartient, vous pouvez créer l’application dans l’environnement ASE, comme le montre la commande suivante :

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "North Central US" -AppServicePlan DestinationAppServicePlan -ASEName DestinationASE -ASEResourceGroupName DestinationASEResourceGroupName -SourceWebApp $srcapp
```

Le paramètre `Location` est requis en raison de l’héritage, mais il est ignoré lorsque vous créez l’application dans un environnement ASE. 

## <a name="cloning-an-existing-app-slot"></a>Clonage d’un emplacement d’application existant
Scénario : Vous souhaitez cloner un emplacement de déploiement d’une application vers une nouvelle application ou un nouvel emplacement. La nouvelle application peut résider dans la même région que l’emplacement d’application d’origine ou dans une autre région.

En connaissant le nom du groupe de ressources qui contient l’application source, vous pouvez utiliser la commande PowerShell suivante pour obtenir les informations de l’emplacement d’application source (dans ce cas nommé `source-appslot`) lié à `source-app` :

```powershell
$srcappslot = Get-AzWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-app -Slot source-appslot
```

La commande suivante illustre la création d’un clone de l’application source vers une nouvelle application :

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-app -Location "North Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot
```

## <a name="configuring-traffic-manager-while-cloning-an-app"></a>Configuration de Traffic Manager lors du clonage d’une application
La création d’applications sur plusieurs régions et la configuration d’Azure Traffic Manager pour router le trafic vers toutes les applications constituent un scénario contribuant largement à garantir que les applications des clients sont hautement disponibles. Au moment de cloner une application existante, vous avez la possibilité de connecter les deux applications à un nouveau profil de gestionnaire de trafic ou à un profil existant. Notez que seule la version Azure Resource Manager de Traffic Manager est prise en charge.

### <a name="creating-a-new-traffic-manager-profile-while-cloning-an-app"></a>Création d’un profil Traffic Manager lors du clonage d’une application
Scénario : Vous souhaitez cloner une application vers une autre région tout en configurant un profil de gestionnaire de trafic Azure Resource Manager qui inclut les deux applications. La commande suivante illustre la création d’un clone de l’application source vers une nouvelle application tout en configurant un nouveau profil Traffic Manager :

```powershell
$destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile
```

### <a name="adding-new-cloned-app-to-an-existing-traffic-manager-profile"></a>Ajout d’une nouvelle application clonée à un profil Traffic Manager existant
Scénario : Vous disposez déjà d’un profil de gestionnaire de trafic Azure Resource Manager auquel vous souhaitez ajouter les deux applications comme points de terminaison. Pour ce faire, vous devez tout d’abord assembler l’ID du profile Traffic Manager existant. Vous devez disposer de l’ID d’abonnement, du nom de groupe de ressources et du nom du profil Traffic Manager existant.

```powershell
$TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"
```

Une fois l’ID de gestionnaire de trafic obtenu, la commande suivante illustre la création d’un clone de l’application source vers une nouvelle application tout en les ajoutant à un profil Traffic Manager existant :

```powershell
$destapp = New-AzWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "South Central US" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID
```
> [!NOTE]
> Si vous recevez une erreur indiquant que « La validation SSL sur le nom d’hôte Traffic Manager échoue », nous vous suggérons d’utiliser l’attribut -IgnoreCustomHostNames dans l’applet de commande PowerShell pendant l’opération de clonage, ou d’utiliser le portail.

## <a name="current-restrictions"></a>Restrictions actuelles
Voici les restrictions connues du clonage d’application :

* Les paramètres de mise à l’échelle automatique ne sont pas clonés.
* Les paramètres de planification de sauvegarde ne sont pas clonés.
* Les paramètres de réseau virtuel ne sont pas clonés.
* App Insights n’est pas automatiquement configuré dans l’application de destination
* Les paramètres d’authentification simple ne sont pas clonés.
* Les paramètres d’extension Kudu ne sont pas clonés.
* Les règles TiP ne sont pas clonées.
* Les contenus de la base de données ne sont pas clonés
* Les adresses IP sortantes changent si vous clonez vers une unité d’échelle différente
* Non disponible pour les applications Linux

### <a name="references"></a>References
* [Clonage de l’application Azure App Service ](app-service-web-app-cloning.md)
* [Sauvegarder une application dans Azure App Service](manage-backup.md)
* [Prise en charge d’Azure Resource Manager pour la version préliminaire d’Azure Traffic Manager](../traffic-manager/traffic-manager-powershell-arm.md)
* [Présentation de l'environnement App Service](environment/intro.md)
* [Utilisation d'Azure PowerShell avec Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md)

