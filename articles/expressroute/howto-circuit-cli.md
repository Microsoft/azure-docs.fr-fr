---
title: 'Démarrage rapide : Créer et modifier un circuit ExpressRoute : Azure CLI'
description: Ce démarrage rapide montre comment créer, approvisionner, vérifier, mettre à jour, supprimer et déprovisionner un circuit ExpressRoute à l’aide d’Azure CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: quickstart
ms.date: 10/05/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: a1d50c3f8f94fbfd7dbcb9b25e051b7f2951c518
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91969073"
---
# <a name="quickstart-create-and-modify-an-expressroute-circuit-using-azure-cli"></a>Démarrage rapide : Créer et modifier un circuit ExpressRoute à l’aide d’Azure CLI

Cet démarrage rapide décrit comment créer un circuit Azure ExpressRoute à l’aide de l’interface de ligne de commande (CLI). Cet article vous montre également comment vérifier l'état, mettre à jour ou supprimer et déprovisionner un circuit.

## <a name="prerequisites"></a>Prérequis

* Examinez les [conditions préalables](expressroute-prerequisites.md) et les [flux de travail](expressroute-workflows.md) avant de commencer la configuration.
* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Installez la dernière version des commandes CLI (version 2.0 ou ultérieure). Pour plus d’informations sur l’installation des commandes CLI, consultez [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli) et [Prise en main d’Azure CLI](/cli/azure/get-started-with-azure-cli).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Créer et approvisionner un circuit ExpressRoute

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Vous connecter à votre compte Azure et sélectionner votre abonnement

Pour commencer votre configuration, connectez-vous à votre compte Azure. Si vous utilisez l’option « Essayer » de Cloud Shell, vous êtes connecté automatiquement. Utilisez les exemples suivants pour faciliter votre connexion :

```azurecli-interactive
az login
```

Vérifiez les abonnements associés au compte.

```azurecli-interactive
az account list
```

Sélectionnez l’abonnement pour lequel vous souhaitez créer un circuit ExpressRoute.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

### <a name="get-the-list-of-supported-providers-locations-and-bandwidths"></a>Récupérer la liste des fournisseurs, des emplacements et des bandes passantes pris en charge

Avant de créer un circuit ExpressRoute, vous avez besoin d’une liste des fournisseurs de services, des emplacements et des options de bande passante pris en charge. La commande d’interface CLI `az network express-route list-service-providers` retourne ces informations que vous utilisez dans les étapes ultérieures :

```azurecli-interactive
az network express-route list-service-providers
```

La réponse ressemble à ce qui suit :

```output
[
  {
    "bandwidthsOffered": [
      {
        "offerName": "50Mbps",
        "valueInMbps": 50
      },
      {
        "offerName": "100Mbps",
        "valueInMbps": 100
      },
      {
        "offerName": "200Mbps",
        "valueInMbps": 200
      },
      {
        "offerName": "500Mbps",
        "valueInMbps": 500
      },
      {
        "offerName": "1Gbps",
        "valueInMbps": 1000
      },
      {
        "offerName": "2Gbps",
        "valueInMbps": 2000
      },
      {
        "offerName": "5Gbps",
        "valueInMbps": 5000
      },
      {
        "offerName": "10Gbps",
        "valueInMbps": 10000
      }
    ],
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/expressRouteServiceProviders/",
    "location": null,
    "name": "AARNet",
    "peeringLocations": [
      "Melbourne",
      "Sydney"
    ],
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "tags": null,
    "type": "Microsoft.Network/expressRouteServiceProviders"
  },
```

Vérifiez la réponse pour voir si votre fournisseur de connectivité est référencé. Prenez note des éléments suivants, car vous en aurez besoin lors de la création d’un circuit :

* Nom
* PeeringLocations
* BandwidthsOffered

Vous êtes maintenant prêt à créer un circuit ExpressRoute.

### <a name="create-an-expressroute-circuit"></a>Création d’un circuit ExpressRoute

> [!IMPORTANT]
> Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Effectuez cette opération quand le fournisseur de connectivité est prêt à approvisionner le circuit.
>
>

Si vous n’avez pas déjà un groupe de ressources, vous devez en créer un avant de créer votre circuit ExpressRoute. Vous pouvez créer un groupe de ressources à l’aide de la commande suivante :

```azurecli-interactive
az group create -n ExpressRouteResourceGroup -l "West US"
```

L’exemple suivant montre comment créer un circuit ExpressRoute de 200 Mb/s par le biais d’Equinix dans la Silicon Valley. Si vous utilisez un autre fournisseur et des paramètres différents, remplacez ces informations quand vous créez votre requête.

Assurez-vous que vous spécifiez le niveau de référence (SKU) et la famille de références corrects :

* Le niveau de référence (SKU) détermine si un circuit ExpressRoute est [Local](expressroute-faqs.md#expressroute-local), Standard ou [Premium](expressroute-faqs.md#expressroute-premium). Vous pouvez spécifier *Local*, *Standard ou *Premium*. Vous ne pouvez pas modifier la référence (SKU) de *Standard/Premium* en *Local*.
* La famille de références détermine le type de facturation. Vous pouvez spécifier *Metereddata* pour un forfait de données limité, et *Unlimiteddata* pour un forfait de données illimitées. Vous pouvez modifier le type de facturation *Metereddata* en *Unlimiteddata*, mais que vous ne pouvez pas le modifier de *Unlimiteddata* en *Metereddata*. Un circuit *Local* est uniquement *Unlimiteddata*.


Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Voici un exemple de demande pour une nouvelle clé de service :

```azurecli-interactive
az network express-route create --bandwidth 200 -n MyCircuit --peering-location "Silicon Valley" -g ExpressRouteResourceGroup --provider "Equinix" -l "West US" --sku-family MeteredData --sku-tier Standard
```

La réponse contient la clé de service.

### <a name="list-all-expressroute-circuits"></a>Répertorier tous les circuits ExpressRoute

Pour obtenir la liste de tous les circuits ExpressRoute que vous avez créés, exécutez la commande `az network express-route list`. Vous pouvez récupérer ces informations à tout moment à l’aide de cette commande. Pour répertorier tous les circuits, effectuez l’appel sans paramètres.

```azurecli-interactive
az network express-route list
```

Votre clé de service apparaît dans le champ *ServiceKey* de la réponse.

```output
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

Vous pouvez obtenir une description détaillée de tous les paramètres en exécutant la commande avec le paramètre « -h ».

```azurecli-interactive
az network express-route list -h
```

### <a name="send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>Envoyer la clé de service à votre fournisseur de connectivité pour l’approvisionnement

« ServiceProviderProvisioningState » fournit des informations sur l’état actuel de l’approvisionnement du côté du fournisseur de service. Le statut indique l’état du côté Microsoft. Pour plus d’informations, consultez l’article sur les [workflows](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Quand vous créez un circuit ExpressRoute, ce circuit affiche l’état suivant :

```output
"serviceProviderProvisioningState": "NotProvisioned"
"circuitProvisioningState": "Enabled"
```

Le circuit passe à l’état suivant quand le fournisseur de connectivité l’active pour vous :

```output
"serviceProviderProvisioningState": "Provisioning"
"circuitProvisioningState": "Enabled"
```

Pour être utilisé, le circuit ExpressRoute doit être dans l’état suivant :

```output
"serviceProviderProvisioningState": "Provisioned"
"circuitProvisioningState": "Enabled
```

### <a name="periodically-check-the-status-and-the-state-of-the-circuit-key"></a>Vérifier régulièrement le statut et l’état de la clé du circuit

La vérification du statut et de l’état de la clé du service vous permet de savoir quand votre fournisseur a approvisionné votre circuit. Une fois le circuit configuré, *ServiceProviderProvisioningState* a la valeur *Approvisionné*, comme le montre l’exemple suivant :

```azurecli-interactive
az network express-route show --resource-group ExpressRouteResourceGroup --name MyCircuit
```

La réponse ressemble à ce qui suit :

```output
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

### <a name="create-your-routing-configuration"></a>Créer votre configuration de routage

Pour obtenir des instructions pas à pas, consultez l’article [Configuration du routage des circuits ExpressRoute](howto-routing-cli.md) pour créer et modifier des peerings de circuit.

> [!IMPORTANT]
> Ces instructions s’appliquent seulement aux circuits créés avec des fournisseurs de services proposant des services de connectivité de couche 2. Si vous utilisez un fournisseur de services proposant des services gérés de couche 3 (généralement un VPN IP, comme MPLS), votre fournisseur de connectivité configure et gère le routage pour vous.
>
>

### <a name="link-a-virtual-network-to-an-expressroute-circuit"></a>Lier un réseau virtuel à un circuit ExpressRoute

Maintenant, vous devez lier un réseau virtuel à votre circuit ExpressRoute. Utilisez l’article sur la [liaison de réseaux virtuels à des circuits ExpressRoute](howto-linkvnet-cli.md).

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Modification d’un circuit ExpressRoute

Vous pouvez modifier certaines propriétés d'un circuit ExpressRoute sans affecter la connectivité. Vous pouvez apporter les modifications suivantes sans temps d'arrêt :

* Vous pouvez activer ou désactiver le module complémentaire ExpressRoute Premium pour votre circuit ExpressRoute. La modification de la référence SKU de *Standard/Premium* en *Local* n’est pas prise en charge.
* Vous pouvez augmenter la bande passante de votre circuit ExpressRoute à condition que la capacité disponible sur le port le permette. Cependant, la rétrogradation de la bande passante d’un circuit n’est pas prise en charge.
* Vous pouvez remplacer le plan de mesure Données limitées par Données illimitées. Cependant, la modification du plan de limitation Données illimitées en Données limitées n’est pas pris en charge.
* Vous pouvez activer et désactiver *Autoriser les opérations classiques*.

Pour plus d’informations sur les limites et les limitations, reportez-vous au [FAQ ExpressRoute](expressroute-faqs.md).

### <a name="to-enable-the-expressroute-premium-add-on"></a>Pour activer le module complémentaire ExpressRoute Premium

Vous pouvez activer le module complémentaire ExpressRoute Premium pour votre circuit existant à l’aide de la commande suivante :

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Premium
```

Les fonctionnalités du module complémentaire ExpressRoute Premium sont ainsi activées pour votre circuit. Nous commençons à vous facturer la fonctionnalité du module complémentaire Premium dès que la commande a été exécutée avec succès.

### <a name="to-disable-the-expressroute-premium-add-on"></a>Pour désactiver le module complémentaire ExpressRoute Premium

> [!IMPORTANT]
> Cette opération peut échouer si vous utilisez des ressources supérieures à ce qui est autorisé pour le circuit standard.
>
>

Avant de désactiver le module complémentaire ExpressRoute Premium, comprenez les critères suivants :

* Avant de passer du niveau Premium au niveau Standard, vous devez vérifier que le nombre de réseaux virtuels liés au circuit est inférieur à 10. Si vous ne le faites pas, votre demande de mise à jour échoue et nous appliquons les tarifs Premium.
* Tous les réseaux virtuels dans d’autres régions géopolitiques doivent préalablement être dissociés. Si vous ne supprimez pas le lien, votre demande de mise à jour échoue et nous continuons de vous facturer aux tarifs Premium.
* Pour le peering privé, votre table de routage doit comporter moins de 4 000 routages. Si elle contient plus de 4 000 routages, la session BGP est abandonnée. Cette session ne sera réactivée qu’une fois le nombre de préfixes publiés repassé sous la barre de 4 000.

Vous pouvez désactiver le module complémentaire ExpressRoute Premium dans votre circuit existant à l’aide de l’exemple suivant :

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Standard
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>Pour mettre à jour la bande passante d’un circuit ExpressRoute

Pour connaître les options de bande passante prises en charge par votre fournisseur, consultez le [FAQ ExpressRoute](expressroute-faqs.md). Vous pouvez choisir n'importe quelle taille supérieure à la taille de votre circuit existant.

> [!IMPORTANT]
> Vous devrez peut-être recréer le circuit ExpressRoute si la capacité sur le port existant est inappropriée. Vous ne pouvez pas mettre le circuit à niveau si aucune capacité supplémentaire n’est disponible à cet emplacement.
>
> Vous ne pouvez pas réduire la bande passante d’un circuit ExpressRoute sans interrompre le service. Le fait de passer à un niveau inférieur de bande passante vous oblige à déprovisionner le circuit ExpressRoute, puis à réapprovisionner un nouveau circuit ExpressRoute.
>

Une fois que vous avez décidé de la taille dont vous avez besoin, utilisez la commande suivante pour redimensionner votre circuit :

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --bandwidth 1000
```

Votre circuit sera mis à niveau côté Microsoft. Vous devrez ensuite contacter votre fournisseur de connectivité pour qu’il mette à jour les configurations de son côté afin de refléter cette modification. Une fois cette modification apportée, nous commençons à vous facturer pour l’option de bande passante mise à jour.

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>Pour modifier la référence de limité à illimité

Vous pouvez modifier la référence SKU d'un circuit ExpressRoute à l'aide de l'exemple suivant :

```azurecli-interactive
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-family UnlimitedData
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>Pour contrôler l'accès aux environnements classique et Resource Manager

Consultez les instructions dans [Transférer des circuits ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager](expressroute-howto-move-arm.md).

## <a name="deprovisioning-an-expressroute-circuit"></a><a name="delete"></a>Déprovisionnement d’un circuit ExpressRoute

Pour déprovisionner et supprimer un circuit ExpressRoute, veillez à bien comprendre les critères suivants :

* Vous devez dissocier tous les réseaux virtuels du circuit ExpressRoute. Si cette opération échoue, vérifiez si certains de vos réseaux virtuels sont liés au circuit.
* Si l’état d’approvisionnement du fournisseur de services du circuit ExpressRoute est **En cours d’approvisionnement** ou **Approvisionné**, vous devez vous mettre en relation avec votre fournisseur de services pour annuler l’approvisionnement du circuit de son côté. Nous continuons à réserver des ressources et à vous facturer jusqu’à ce que le fournisseur de services termine le désapprovisionnement du circuit et nous en avertisse.
* Si le fournisseur de services a déprovisionné le circuit, ce qui signifie que l’état d’approvisionnement du fournisseur de services est défini sur **Non approvisionné**, vous pouvez supprimer le circuit. La facturation du circuit s’arrêtera.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer votre circuit ExpressRoute en exécutant la commande suivante :

```azurecli-interactive
az network express-route delete  -n MyCircuit -g ExpressRouteResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez créé votre circuit et l’avez approvisionné avec votre fournisseur, passez à l’étape suivante pour configurer l’homologation :

> [!div class="nextstepaction"]
> [Créer et modifier le routage le routage pour votre circuit ExpressRoute](howto-routing-cli.md)
