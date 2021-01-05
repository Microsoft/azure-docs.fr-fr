---
title: Remplacement de sous-réseau Azure Traffic Manager avec Azure PowerShell | Microsoft Docs
description: Cet article va vous aider à comprendre comment le remplacement de sous-réseau Traffic Manager permet de remplacer la méthode de routage d’un profil Traffic Manager pour diriger le trafic vers un point de terminaison en fonction de l’adresse IP de l’utilisateur final via une plage d’adresses IP prédéfinie pour les mappages de point de terminaison à l’aide d’Azure PowerShell.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 01bd3b1e945ee7c9ac16af7048536c0a9e2d731a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89401586"
---
# <a name="traffic-manager-subnet-override-using-azure-powershell"></a>Remplacement de sous-réseau Traffic Manager avec Azure PowerShell

Le remplacement du sous-réseau Traffic Manager vous permet de modifier la méthode de routage d’un profil.  L’ajout d’un remplacement permet de diriger le trafic en fonction de l’adresse IP de l’utilisateur final avec une plage d’adresses IP prédéfinie pour le mappage de point de terminaison. 

## <a name="how-subnet-override-works"></a>Comment fonctionne le remplacement de sous-réseau

Quand des remplacements de sous-réseau sont ajoutés à un profil Traffic Manager, Traffic Manager vérifie d’abord s’il existe un remplacement de sous-réseau pour l’adresse IP de l’utilisateur final. Si tel est le cas, la requête DNS de l’utilisateur est dirigée vers le point de terminaison correspondant.  En l’absence de mappage, Traffic Manager revient à la méthode de routage d’origine du profil. 

Les plages d’adresses IP peuvent être spécifiées sous la forme de plages CIDR (par exemple 1.2.3.0/24) ou de plages d’adresses (par exemple 1.2.3.4-5.6.7.8). Les plages d’adresses IP associées à chaque point de terminaison doivent être uniques pour ce point de terminaison. Tout chevauchement de plages d’adresses IP entre différents points de terminaison entraîne le rejet du profil par Traffic Manager.

Il existe deux types de profil de routage qui prennent en charge les remplacements de sous-réseau :

* **Géographique** - Si Traffic Manager trouve un remplacement de sous-réseau pour l’adresse IP de la requête DNS, il route cette requête vers le point de terminaison, quel que soit son intégrité.
* **Performances** - Si Traffic Manager trouve un remplacement de sous-réseau pour l’adresse IP de la requête DNS, il route le trafic vers le point de terminaison, uniquement s’il est sain.  Traffic Manager revient à l’heuristique de routage des performances si le point de terminaison de remplacement de sous-réseau n’est pas sain.

## <a name="create-a-traffic-manager-subnet-override"></a>Créer un remplacement de sous-réseau Traffic Manager

Pour créer un remplacement de sous-réseau Traffic Manager, vous pouvez utiliser Azure PowerShell afin d’ajouter les sous-réseaux du remplacement au point de terminaison Traffic Manager.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Si vous exécutez PowerShell depuis votre ordinateur, vous avez besoin du module Azure PowerShell version 1.0.0 ou ultérieure. Vous pouvez exécuter `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également exécuter `Login-AzAccount` pour vous connecter à Azure.


1. **Récupérer le point de terminaison Traffic Manager :**

    Pour activer le remplacement de sous-réseau, récupérez le point de terminaison à ajouter au remplacement, puis stockez-le dans une variable à l’aide de [Get-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/get-aztrafficmanagerendpoint?view=azps-2.5.0).

    Remplacez les valeurs de Name, ProfileName et ResourceGroupName par les valeurs du point de terminaison que vous changez.

    ```powershell

    $TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "contoso" -ProfileName "ContosoProfile" -ResourceGroupName "ResourceGroup" -Type AzureEndpoints

    ```
2. **Ajouter la plage d’adresses IP au point de terminaison :**
    
    Pour ajouter la plage d’adresses IP au point de terminaison, vous devez utiliser [Add-AzTrafficManagerIpAddressRange](https://docs.microsoft.com/powershell/module/az.trafficmanager/add-aztrafficmanageripaddressrange?view=azps-2.5.0&viewFallbackFrom=azps-2.4.0).

    ```powershell

    ### Add a range of IPs ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "1.2.3.4" -Last "5.6.7.8"

    ### Add a subnet ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "9.10.11.0" -Scope 24

    ### Add a range of IPs with a subnet ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "12.13.14.0" -Last "12.13.14.31" -Scope 27
 
    ```
    Une fois les plages ajoutées, utilisez [Set-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/set-aztrafficmanagerendpoint?view=azps-2.5.0) pour mettre à jour le point de terminaison.

    ```powershell

    Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint

    ```
Vous pouvez supprimer la plage d’adresses IP à l’aide de [Remove-AzTrafficManagerIpAddressRange](https://docs.microsoft.com/powershell/module/az.trafficmanager/remove-aztrafficmanageripaddressrange?view=azps-2.5.0).

1.  **Récupérer le point de terminaison Traffic Manager :**

    Pour activer le remplacement de sous-réseau, récupérez le point de terminaison à ajouter au remplacement, puis stockez-le dans une variable à l’aide de [Get-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/get-aztrafficmanagerendpoint?view=azps-2.5.0).

    Remplacez les valeurs de Name, ProfileName et ResourceGroupName par les valeurs du point de terminaison que vous changez.

    ```powershell

    $TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "contoso" -ProfileName "ContosoProfile" -ResourceGroupName "ResourceGroup" -Type AzureEndpoints

    ```
2. **Supprimer la plage d’adresses IP du point de terminaison :**

    ```powershell
    
    ### Remove a range of IPs ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "1.2.3.4" -Last "5.6.7.8"

    ### Remove a subnet ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "9.10.11.0" -Scope 24

    ### Remove a range of IPs with a subnet ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "12.13.14.0" -Last "12.13.14.31" -Scope 27

    ```
     Une fois les plages supprimées, utilisez [Set-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/set-aztrafficmanagerendpoint?view=azps-2.5.0) pour mettre à jour le point de terminaison.

    ```powershell

    Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint

    ```

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les [méthodes de routage du trafic](traffic-manager-routing-methods.md)de Traffic Manager.

En savoir plus sur la [méthode de routage du trafic de sous-réseau](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods#subnet-traffic-routing-method)
