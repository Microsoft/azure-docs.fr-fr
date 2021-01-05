---
title: 'Inscrire une connexion Peering Service - Azure PowerShell '
description: Dans ce tutoriel, découvrez comment inscrire une connexion Peering Service avec PowerShell.
services: peering-service
author: derekolo
ms.service: peering-service
ms.topic: tutorial
ms.date: 05/18/2020
ms.author: derekol
Customer intent: Customer wants to measure their connection telemetry per prefix to Microsoft services with Azure Peering Service .
ms.openlocfilehash: a12bfa042c8741814cedc70f1dcb67dedbfd331e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91400432"
---
# <a name="tutorial-register-a-peering-service-connection-using-azure-powershell"></a>Tutoriel : Inscrire une connexion Peering Service en utilisant Azure PowerShell

Dans ce tutoriel, découvrez comment inscrire Peering Service en utilisant Azure PowerShell.

Azure Peering Service est un service réseau qui améliore la connectivité entre le client et les services cloud Microsoft, comme Microsoft 365, Dynamics 365, les services SaaS, Azure ou les services Microsoft accessibles via l’Internet public. Cet article explique comment inscrire une connexion Peering Service en utilisant Azure PowerShell.

Si vous n’avez pas d’abonnement Azure, créez un [compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) maintenant.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous décidez plutôt d’installer et d’utiliser PowerShell en local, vous devez utiliser le module Azure PowerShell version 1.0.0, ou une version ultérieure, pour les besoins de ce guide de démarrage rapide. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. Pour plus d’informations sur l’installation et la mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

Enfin, si vous exécutez PowerShell en local, vous devez également exécuter `Connect-AzAccount`. Cette commande crée une connexion avec Azure.

Utilisez le module Azure PowerShell pour inscrire et gérer Peering Service. Vous pouvez inscrire ou gérer Peering Service à partir de la ligne de commande PowerShell ou à l’aide de scripts.


## <a name="prerequisites"></a>Prérequis  
Vous devez disposer des éléments suivants :

### <a name="azure-account"></a>Compte Azure

Vous devez avoir un compte Microsoft Azure actif et valide. Sinon, vous ne pourrez pas configurer la connexion Peering Service. Peering Service est une ressource fournie dans les abonnements Azure.

### <a name="connectivity-provider"></a>Fournisseur de connectivité

Vous pouvez faire appel à un fournisseur de services Internet ou un partenaire d’échange Internet pour obtenir une connectivité Peering Service qui vous permettra de connecter votre réseau au réseau Microsoft.

Assurez-vous que les fournisseurs de connectivité sont des partenaires de Microsoft.

### <a name="register-a-subscription-with-the-resource-provider-and-feature-flag"></a>Inscrire un abonnement auprès du fournisseur de ressources et l’indicateur de fonctionnalité

Avant de passer aux étapes d’inscription de Peering Service, inscrivez votre abonnement auprès du fournisseur de ressources et de l’indicateur de fonctionnalité à l’aide d’Azure PowerShell. Les commandes Azure PowerShell sont spécifiées ici :

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName AllowPeeringService ProviderNamespace Microsoft.Peering 

Register-AzResourceProvider -ProviderNamespace Microsoft.Peering 

```

### <a name="fetch-the-location-and-service-provider"></a>Obtenir l’emplacement et le fournisseur de services 

Exécutez les commandes suivantes dans Azure PowerShell pour obtenir l’emplacement et le fournisseur de services sur lesquels Peering Service doit être activé. 

Obtenir les emplacements Peering Service :

```azurepowershell-interactive
# Gets a list of available countries
Get-AzPeeringServiceCountry 
# Gets a list of metro locations serviced by country
Get-AzPeeringServiceLocation -Country "United States"
```

Obtenir les fournisseurs Peering Service :
              
```azurepowershell-interactive
Get-AzPeeringServiceProvider
```

### <a name="register-the-peering-service-connection"></a>Inscrire la connexion Peering Service

Inscrivez la connexion Peering Service à l’aide de l’ensemble de commandes suivant dans Azure PowerShell. Cet exemple inscrit la connexion Peering Service nommée myPeeringService.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
```

### <a name="register-the-peering-service-prefix"></a>Inscrire le préfixe Peering Service

Inscrivez le préfixe fourni par le fournisseur de connectivité en exécutant les commandes suivantes dans Azure PowerShell. Cet exemple inscrit le préfixe nommé myPrefix.

```azurepowershell-interactive
$loc = "Washington"
$provider = "TestPeer1"
$resourceGroup = "MyResourceGroup"
$name = “myPeeringService”
$peeringService = New-AzPeeringService -ResourceGroupName $resourceGroup -Name $name -PeeringLocation $loc -PeeringServiceProvider $provider
$prefixName = "myPrefix"
$prefix = “192.168.1.0/24”
$serviceKey = "6f48cdd6-2c2e-4722-af89-47e27b2513af"
$prefixService = $peeringService | New-AzPeeringServicePrefix -Name $prefixName -Prefix $prefix -ServiceKey $serviceKey
```

### <a name="list-all-the-peering-services-connections"></a>Lister toutes les connexions Peering Service

Pour afficher la liste de toutes les connexions Peering Service, exécutez la commande suivante :

```azurepowershell-interactive
$peeringService = Get-AzPeeringService
```

### <a name="list-all-the-peering-service-prefixes"></a>Lister tous les préfixes Peering Service

Pour afficher la liste de tous les préfixes Peering Service, exécutez la commande suivante :

```azurepowershell-interactive
 $prefixName = "myPrefix"
```

```azurepowershell-interactive
$prefix = Get-AzPeeringServicePrefix -PeeringServiceName "myPeeringService" -ResourceGroupName "MyResourceGroup" -Name "myPrefix"
```

### <a name="remove-the-peering-service-prefix"></a>Supprimer le préfixe Peering Service

Pour supprimer le préfixe Peering Service, exécutez la commande suivante :

```azurepowershell-interactive
Remove-AzPeeringServicePrefix -ResourceGroupName  "MyResourceGroup" -Name "myPrefix" -PeeringServiceName "myPeeringService"
```

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).
- Pour inscrire une connexion Peering Service par le biais du Portail Azure, consultez [Inscrire une connexion Peering Service - Portail Azure](azure-portal.md).
- Pour inscrire une connexion Peering Service à l’aide d’Azure CLI, consultez [Inscrire une connexion Peering Service - Azure CLI](cli.md).
