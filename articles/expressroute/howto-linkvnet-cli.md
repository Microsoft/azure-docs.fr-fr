---
title: 'Tutoriel : Lier un réseau virtuel à un circuit ExpressRoute – Azure CLI'
description: Ce tutoriel montre comment lier des réseaux virtuels à des circuits ExpressRoute en utilisant le modèle de déploiement Resource Manager et Azure CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.openlocfilehash: b9dda384e2ef30808559d10012dea2909b2af0fd
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206932"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit-using-cli"></a>Tutoriel : Connecter un réseau virtuel à un circuit ExpressRoute à l’aide de l’interface CLI

Ce tutoriel montre comment lier des réseaux virtuels (VNets) à des circuits Azure ExpressRoute à l’aide d’Azure CLI. Pour pouvoir être liés avec l’interface Azure CLI, les réseaux virtuels doivent être créés à l’aide du modèle de déploiement Resource Manager. Ils peuvent appartenir au même abonnement ou faire partie d’un autre abonnement. Si vous souhaitez utiliser une autre méthode pour connecter votre réseau virtuel à un circuit ExpressRoute, vous pouvez sélectionner un article dans la liste suivante :

> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-linkvnet-classic.md)
> 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Connecter un réseau virtuel du même abonnement à un circuit
> - Connecter un réseau virtuel d’un autre abonnement à un circuit
> - Modifier une connexion de réseau virtuel
> - Configurer ExpressRoute FastPath

## <a name="prerequisites"></a>Prérequis

* Vous avez besoin de la dernière version de l’interface de ligne de commande (CLI). Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).
* Avant de commencer la configuration, examinez les [conditions préalables](expressroute-prerequisites.md), la [configuration requise pour le routage](expressroute-routing.md) et les [flux de travail](expressroute-workflows.md).
* Vous devez disposer d’un circuit ExpressRoute actif. 
  * Suivez les instructions permettant de [créer un circuit ExpressRoute](howto-circuit-cli.md) et faites-le activer par votre fournisseur de service de connectivité. 
  * Vérifiez que le peering privé Azure est configuré pour votre circuit. Pour obtenir des instructions de routage, consultez l'article sur la [configuration du routage](howto-routing-cli.md) . 
  * Assurez-vous que le peering privé Azure est configuré. L’homologation BGP entre votre réseau et Microsoft doit être établie pour que vous puissiez activer la connectivité de bout en bout.
  * Vérifiez qu’un réseau virtuel et une passerelle de réseau virtuel ont été créés et entièrement approvisionnés. Suivez les instructions fournies dans l’article [Configurer une passerelle de réseau virtuel pour ExpressRoute](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md). Veillez à utiliser `--gateway-type ExpressRoute`.
* Vous pouvez lier jusqu’à 10 réseaux virtuels à un circuit ExpressRoute standard. Tous les réseaux virtuels doivent figurer dans la même région géopolitique lors de l’utilisation d’un circuit ExpressRoute standard. 
* Un réseau virtuel unique peut être lié à quatre circuits ExpressRoute maximum. Pour créer un objet connexion pour chaque circuit ExpressRoute auquel vous vous connectez, procédez comme suit. Les circuits ExpressRoute peuvent être dans le même abonnement, dans des abonnements différents ou dans une combinaison des deux.
* Si vous activez le module complémentaire ExpressRoute Premium, vous pouvez lier des réseaux virtuels à l’extérieur de la région géopolitique du circuit ExpressRoute. Le module complémentaire Premium vous permet également de connecter plus de 10 réseaux virtuels à votre circuit ExpressRoute en fonction de la bande passante choisie. Pour plus d’informations sur le module complémentaire Premium, consultez le [FAQ](expressroute-faqs.md) .

## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>Connecter un réseau virtuel du même abonnement à un circuit

Vous pouvez connecter une passerelle de réseau virtuel à un circuit ExpressRoute à l’aide de l’exemple suivant. Assurez-vous que la passerelle de réseau virtuel est créée et prête pour la liaison avant d’exécuter la commande.

```azurecli-interactive
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit
```

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>Connecter un réseau virtuel d’un autre abonnement à un circuit

Vous pouvez partager un circuit ExpressRoute entre plusieurs abonnements. La figure suivante montre un schéma simple sur le fonctionnement du partage de circuits ExpressRoute entre plusieurs abonnements.

Chacun des petits clouds dans le cloud principal est utilisé pour représenter les abonnements appartenant à différents services au sein d’une organisation. Les départements au sein de l’organisation utilisent leur propre abonnement pour déployer leurs services, mais ils peuvent partager un même circuit ExpressRoute pour se reconnecter à votre réseau local. Un seul service (dans cet exemple : le service informatique) peut détenir le circuit ExpressRoute. D’autres abonnements au sein de l’organisation peuvent utiliser le circuit ExpressRoute.

> [!NOTE]
> Les frais de connectivité et de bande passante pour le circuit dédié s’appliquent au propriétaire du circuit ExpressRoute. Tous les réseaux virtuels partagent la même bande passante.
> 
> 

![Connectivité entre abonnements](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### <a name="administration---circuit-owners-and-circuit-users"></a>Administration : propriétaires de circuit et utilisateurs du circuit

Le « propriétaire du circuit » est un utilisateur avancé autorisé de la ressource de circuit ExpressRoute. Le propriétaire du circuit peut créer des autorisations utilisables par les « utilisateurs du circuit ». Les utilisateurs du circuit sont propriétaires de passerelles de réseau virtuel qui ne figurent pas dans le même abonnement que le circuit ExpressRoute. Les utilisateurs du circuit peuvent utiliser des autorisations (une seule autorisation par réseau virtuel).

Le propriétaire du circuit a le pouvoir de modifier et de révoquer les autorisations à tout moment. Lorsqu’une autorisation est révoquée, toutes les connexions sont supprimées de l’abonnement dont l’accès a été révoqué.

### <a name="circuit-owner-operations"></a>Opérations du propriétaire du circuit

**Création d’une autorisation**

Le propriétaire du circuit crée une autorisation, ce qui entraîne la création d’une clé d’autorisation dont un utilisateur du circuit peut se servir pour connecter ses passerelles de réseau virtuel au circuit ExpressRoute. Une autorisation n’est valide que pour une seule connexion.

L’exemple suivant montre comment créer une autorisation :

```azurecli-interactive
az network express-route auth create --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization
```

La réponse contient la clé et l’état d’autorisation :

```output
"authorizationKey": "0a7f3020-541f-4b4b-844a-5fb43472e3d7",
"authorizationUseStatus": "Available",
"etag": "W/\"010353d4-8955-4984-807a-585c21a22ae0\"",
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/authorizations/MyAuthorization1",
"name": "MyAuthorization1",
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup"
```

**Vérification des autorisations**

Le propriétaire du circuit peut vérifier toutes les autorisations émises pour un circuit donné en exécutant l’exemple suivant :

```azurecli-interactive
az network express-route auth list --circuit-name MyCircuit -g ExpressRouteResourceGroup
```

**Ajout d’autorisations**

Le propriétaire du circuit peut ajouter des autorisations à l’aide de l’exemple suivant :

```azurecli-interactive
az network express-route auth create --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization1
```

**Suppression d’autorisations**

Le propriétaire du circuit peut révoquer/supprimer les autorisations accordées à l’utilisateur en exécutant l’exemple suivant :

```azurecli-interactive
az network express-route auth delete --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization1
```

### <a name="circuit-user-operations"></a>Opérations de l’utilisateur du circuit

L’utilisateur du circuit a besoin de l’ID de pair et d’une clé d’autorisation de la part du propriétaire du circuit. La clé d'autorisation est un GUID.

```azurecli-interactive
az network express-route show -n MyCircuit -g ExpressRouteResourceGroup
```

**Réclamation d’une autorisation de connexion**

L’utilisateur du circuit peut exécuter l’applet de commande suivante pour utiliser une autorisation de liaison :

```azurecli-interactive
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit --authorization-key "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**Libération d’une autorisation de connexion**

Vous pouvez libérer une autorisation en supprimant la connexion qui lie le circuit ExpressRoute et le réseau virtuel.

## <a name="modify-a-virtual-network-connection"></a>Modifier une connexion de réseau virtuel
Vous pouvez mettre à jour certaines propriétés d’une connexion de réseau virtuel. 

**Mise à jour du poids attribué à la connexion**

Votre réseau virtuel peut être connecté à plusieurs circuits ExpressRoute. Vous pouvez recevoir le même préfixe à partir de plusieurs circuits ExpressRoute. Pour choisir la connexion pour envoyer le trafic destiné à ce préfixe, vous pouvez modifier le *RoutingWeight* d’une connexion. Le trafic est envoyé sur la connexion avec le *RoutingWeight*.le plus élevé.

```azurecli-interactive
az network vpn-connection update --name ERConnection --resource-group ExpressRouteResourceGroup --routing-weight 100
```

La plage de *RoutingWeight* est de 0 à 32000. La valeur par défaut est 0.

## <a name="configure-expressroute-fastpath"></a>Configurer ExpressRoute FastPath 
Vous pouvez activer [ExpressRoute FastPath](expressroute-about-virtual-network-gateways.md) si votre passerelle de réseau virtuel est Très hautes performances ou ErGw3AZ. FastPath améliore le niveau de performance de chemin d’accès de données comme le nombre de paquets et de connexions par seconde entre votre réseau local et votre réseau virtuel. 

**Configurer FastPath sur une nouvelle connexion**

```azurecli-interactive
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --express-route-gateway-bypass true --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit
```

**Mise à jour d’une connexion existante pour activer FastPath**

```azurecli-interactive
az network vpn-connection update --name ERConnection --resource-group ExpressRouteResourceGroup --express-route-gateway-bypass true
```
## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin de la connexion ExpressRoute, à partir de l’abonnement où se trouve la passerelle, utilisez la commande `az network vpn-connection delete` pour supprimer le lien entre la passerelle et le circuit.

```azurecli-interactive
az network vpn-connection delete --name ERConnection --resource-group ExpressRouteResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a montré comment connecter un réseau virtuel à un circuit dans le même abonnement et dans un autre abonnement. Pour plus d’informations sur la passerelle ExpressRoute, consultez : 

> [!div class="nextstepaction"]
> [À propos des passerelles de réseau virtuel ExpressRoute](expressroute-about-virtual-network-gateways.md)