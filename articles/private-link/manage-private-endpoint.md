---
title: Gérer une connexion de point de terminaison privé dans Azure
description: Découvrez comment gérer une connexion de point de terminaison privé dans Azure
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.openlocfilehash: 2987cd1ff8c678f7079e13e8b9bc657817c066f1
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95531370"
---
# <a name="manage-a-private-endpoint-connection"></a>Gérer une connexion de point de terminaison privé
Azure Private Link fonctionne sur un modèle de flux d’appel d’approbation, dans lequel l’utilisateur du service Private Link peut demander une connexion au fournisseur de services pour utiliser le service. Le fournisseur de services peut alors décider s’il faut autoriser l’utilisateur à se connecter. Azure Private Link permet aux fournisseurs de services de gérer la connexion des points de terminaison privés dans leurs ressources. Cet article fournit des instructions sur la gestion des connexions de points de terminaison privés.

![Gérer les points de terminaison privés](media/manage-private-endpoint/manage-private-endpoint.png)

L’utilisateur du service Private Link peut choisir entre deux méthodes d’approbation des connexions :
- **Automatique** : Si l’utilisateur du service a des autorisations Azure RBAC sur la ressource du fournisseur de services, il peut choisir la méthode d’approbation automatique. Dans ce cas, lorsque la demande atteint la ressource du fournisseur de services, celui-ci n’a rien à faire, car la connexion est automatiquement approuvée. 
- **Manuelle** : À l’inverse, si l’utilisateur du service ne dispose pas d’autorisations Azure RBAC sur la ressource du fournisseur de services, il peut choisir la méthode d’approbation manuelle. Dans ce cas, la demande de connexion apparaît comme étant **En attente** dans les ressources du service. Le fournisseur de services doit approuver manuellement la demande avant de pouvoir établir des connexions. En cas d’approbation manuelle, l’utilisateur peut également ajouter un message à sa demande pour fournir davantage de contexte au fournisseur de services. Le fournisseur de services peut choisir parmi les options suivantes pour toutes les connexions de point de terminaison privés : **Approuver**, **Rejeter**, **Supprimer**.

Le tableau ci-dessous présente les différentes actions que peut effectuer le fournisseur de services, ainsi que les états de connexion résultants pour les points de terminaison privés.  Le fournisseur de services pourra également modifier l’état de la connexion de point de terminaison privé plus tard, sans intervention de l’utilisateur. Chaque action met à jour l’état du point de terminaison côté client. 


|Action du fournisseur de services   |État du point de terminaison privé de l’utilisateur du service   |Description   |
|---------|---------|---------|
|None    |    Pending     |    La connexion est créée manuellement et est en attente de l’approbation du propriétaire de la ressource Private Link.       |
|Approbation    |  Approved       |  La connexion a été approuvée automatiquement ou manuellement et est prête à être utilisée.     |
|Rejeter     | Rejeté        | La connexion a été rejetée par le propriétaire de la ressource Private Link.        |
|Supprimer    |  Déconnecté       | La connexion a été supprimée par le propriétaire de la ressource Private Link, le point de terminaison privé devient donc informatif et doit être supprimé dans le cadre d’un nettoyage.        |
|   |         |         |
   
## <a name="manage-private-endpoint-connections-on-azure-paas-resources"></a>Gérer les connexions de point de terminaison privé dans les ressources PaaS Azure
Il est recommandé d’utiliser le portail pour gérer les connexions de point de terminaison privé dans les ressources Azure PaaS. Ni PowerShell ni Azure CLI ne sont pris en charge pour la gestion des connexions dans les ressources Azure PaaS.
1. Connectez-vous au portail Azure sur https://portal.azure.com.
2. Accédez au centre Private Link.
3. Sous **Ressources**, sélectionnez le type de ressource pour lequel vous souhaitez gérer les connexions de point de terminaison privé.
4. Pour chaque type de ressource, vous pouvez afficher le nombre de connexions de point de terminaison privé qui lui sont associées. Vous pouvez filtrer les ressources selon vos besoins.
5. Sélectionnez les connexions de point de terminaison privé.  Dans la liste des connexions, sélectionnez la connexion que vous souhaitez gérer. 
6. Vous pouvez modifier l’état de la connexion en sélectionnant l’une des options situées en haut.

## <a name="manage-private-endpoint-connections-on-a-customerpartner-owned-private-link-service"></a>Gérer les connexions de point de terminaison privé dans un service Private Link détenu par un client ou un partenaire

Azure PowerShell et Azure CLI sont recommandés pour la gestion des connexions de point de terminaison privé dans les services des partenaires Microsoft ou ceux appartenant aux clients. Le portail n’est pas pris en charge pour la gestion des connexions dans un service Private Link.  
 
### <a name="powershell"></a>PowerShell 
  
Utilisez les commandes PowerShell suivantes pour gérer les connexions de point de terminaison privé.  
#### <a name="get-private-link-connection-states"></a>Obtenir les état des connexions Private Link 
Utilisez l’applet de commande `Get-AzPrivateLinkService` pour récupérer les connexions de point de terminaison privé et leur état.  
```azurepowershell
Get-AzPrivateLinkService -Name myPrivateLinkService -ResourceGroupName myResourceGroup 
 ```
 
#### <a name="approve-a-private-endpoint-connection"></a>Approuver une connexion de point de terminaison privé 
 
Utilisez l’applet de commande `Approve-AzPrivateEndpointConnection` pour approuver une connexion de point de terminaison privé. 
 
```azurepowershell
Approve-AzPrivateEndpointConnection -Name myPrivateEndpointConnection -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkService
```
 
#### <a name="deny-private-endpoint-connection"></a>Refuser une connexion de point de terminaison privé 
 
Utilisez l’applet de commande `Deny-AzPrivateEndpointConnection` pour rejeter une connexion de point de terminaison privé. 
```azurepowershell
Deny-AzPrivateEndpointConnection -Name myPrivateEndpointConnection -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkService 
```
#### <a name="remove-private-endpoint-connection"></a>Supprimer une connexion de point de terminaison privé 
 
Utilisez l’applet de commande `Remove-AzPrivateEndpointConnection` pour supprimer une connexion de point de terminaison privé. 
```azurepowershell
Remove-AzPrivateEndpointConnection -Name myPrivateEndpointConnection1 -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkServiceName 
```
 
### <a name="azure-cli"></a>Azure CLI 
 
Utilisez `az network private-link-service update` pour gérer vos connexions de point de terminaison privé. L’état de la connexion est spécifié dans le paramètre ```azurecli connection-status```. 
```azurecli
az network private-link-service connection update -g myResourceGroup -n myPrivateEndpointConnection1 --service-name myPLS --connection-status Approved 
```

   

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus sur les points de terminaison privés](private-endpoint-overview.md)
 
