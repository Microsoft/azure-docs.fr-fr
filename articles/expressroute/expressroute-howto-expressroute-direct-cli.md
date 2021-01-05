---
title: 'Azure ExpressRoute : Configurer ExpressRoute Direct : Interface de ligne de commande'
description: Découvrez comment utiliser Azure CLI pour configurer Azure ExpressRoute Direct en vue de vous connecter directement au réseau mondial Microsoft.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 09/28/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7931ad9e55d62d5fa2b1828d276e56f7c8cd02e1
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93286479"
---
# <a name="configure-expressroute-direct-by-using-the-azure-cli"></a>Configurer ExpressRoute Direct à l’aide d’Azure CLI

ExpressRoute Direct vous offre la possibilité de vous connecter directement au réseau mondial Microsoft à partir d’emplacements de peering qui sont distribués stratégiquement dans le monde entier. Pour plus d’informations, consultez [À propos d’ExpressRoute Direct Connect](expressroute-erdirect-about.md).

## <a name="create-the-resource"></a><a name="resources"></a>Créer la ressource

1. Connectez-vous à Azure et sélectionnez l’abonnement qui contient ExpressRoute. La ressource ExpressRoute Direct et vos circuits ExpressRoute doivent se trouver dans le même abonnement. Dans Azure CLI, exécutez les commandes suivantes :

   ```azurecli
   az login
   ```

   Vérifiez les abonnements du compte : 

   ```azurecli
   az account list 
   ```

   Sélectionnez l’abonnement pour lequel vous souhaitez créer un circuit ExpressRoute :

   ```azurecli
   az account set --subscription "<subscription ID>"
   ```

2. Réinscrivez votre abonnement auprès de Microsoft.Network pour accéder aux API expressrouteportslocation et expressrouteport.

   ```azurecli
   az provider register --namespace Microsoft.Network
   ```
3. Listez tous les emplacements où ExpressRoute Direct est pris en charge :
    
   ```azurecli
   az network express-route port location list
   ```

   **Exemple de sortie**
  
   ```output
   [
   {
    "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
    "location": null,
    "name": "Equinix-Ashburn-DC2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "1950 N. Stemmons Freeway, Suite 1039A, DA3, Dallas, TX 75207",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Dallas-DA3",
    "location": null,
    "name": "Equinix-Dallas-DA3",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "111 8th Avenue, New York, NY 10011",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-New-York-NY5",
    "location": null,
    "name": "Equinix-New-York-NY5",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "11 Great Oaks Blvd, SV1, San Jose, CA 95119",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-SV1",
    "location": null,
    "name": "Equinix-San-Jose-SV1",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   },
   {
    "address": "2001 Sixth Ave., Suite 350, SE2, Seattle, WA 98121",
    "availableBandwidths": [],
    "contact": "support@equinix.com",
    "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Seattle-SE2",
    "location": null,
    "name": "Equinix-Seattle-SE2",
    "provisioningState": "Succeeded",
    "tags": null,
    "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ]
   ```
4. Déterminez si l’un des emplacements listés à l’étape précédente dispose de bande passante :

   ```azurecli
   az network express-route port location show -l "Equinix-Ashburn-DC2"
   ```

   **Exemple de sortie**

   ```output
   {
   "address": "21715 Filigree Court, DC2, Building F, Ashburn, VA 20147",
   "availableBandwidths": [
    {
      "offerName": "100 Gbps",
      "valueInGbps": 100
    }
   ],
   "contact": "support@equinix.com",
   "id": "/subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-DC2",
   "location": null,
   "name": "Equinix-Ashburn-DC2",
   "provisioningState": "Succeeded",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePortsLocations"
   }
   ```
5. Créez une ressource ExpressRoute Direct qui repose sur l’emplacement que vous avez choisi dans les étapes précédentes.

   ExpressRoute Direct prend en charge l’encapsulation QinQ et Dot1Q. Si vous sélectionnez QinQ, chaque circuit ExpressRoute se voit affecter dynamiquement un S-Tag et est unique dans l’ensemble de la ressource ExpressRoute Direct. Chaque C-Tag doit être unique sur le circuit, mais pas dans la ressource ExpressRoute Direct.  

   Si vous sélectionnez l’encapsulation Dot1Q, vous devez gérer l’unicité du C-Tag (VLAN) dans l’ensemble de la ressource ExpressRoute Direct.  

   > [!IMPORTANT]
   > ExpressRoute Direct peut uniquement être un seul type d’encapsulation. Vous ne pouvez pas modifier le type d’encapsulation après avoir créé la ressource ExpressRoute Direct.
   > 
 
   ```azurecli
   az network express-route port create -n $name -g $RGName --bandwidth 100 gbps  --encapsulation QinQ | Dot1Q --peering-location $PeeringLocationName -l $AzureRegion 
   ```

   > [!NOTE]
   > Vous pouvez aussi définir l’attribut **Encapsulation** sur **Dot1Q**. 
   >

   **Exemple de sortie**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Disabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "02ee21fe-4223-4942-a6bc-8d81daabc94f",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }  
   ```

## <a name="change-adminstate-for-links"></a><a name="state"></a>Modifier AdminState pour les liens

Utilisez ce processus pour effectuer un test de la couche 1. Vérifiez que chaque connexion croisée est correctement corrigée dans chaque routeur des ports principaux et secondaires.

1. Définissez les liens (links) sur **Enabled** (activé). Répétez cette étape pour définir chaque lien sur **Enabled**.

   Links[0] correspondent au port principal et Links[1] au port secondaire.

   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[0].adminState="Enabled"
   ```
   ```azurecli
   az network express-route port update -n Contoso-Direct -g Contoso-Direct-rg --set links[1].adminState="Enabled"
   ```
   **Exemple de sortie**

   ```output
   {
   "allocationDate": "Wednesday, October 17, 2018",
   "bandwidthInGbps": 100,
   "circuits": null,
   "encapsulation": "Dot1Q",
   "etag": "W/\"<etagnumber>\"",
   "etherType": "0x8100",
   "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
   "links": [
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link1",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link1",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-1",
      "type": "Microsoft.Network/expressRoutePorts/links"
    },
    {
      "adminState": "Enabled",
      "connectorType": "LC",
      "etag": "W/\"<etagnumber>\"",
      "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct/links/link2",
      "interfaceName": "HundredGigE2/2/2",
      "name": "link2",
      "patchPanelId": "PPID",
      "provisioningState": "Succeeded",
      "rackId": "RackID",
      "resourceGroup": "Contoso-Direct-rg",
      "routerName": "tst-09xgmr-cis-2",
      "type": "Microsoft.Network/expressRoutePorts/links"
    }
   ],
   "location": "westus",
   "mtu": "1500",
   "name": "Contoso-Direct",
   "peeringLocation": "Equinix-Ashburn-DC2",
   "provisionedBandwidthInGbps": 0.0,
   "provisioningState": "Succeeded",
   "resourceGroup": "Contoso-Direct-rg",
   "resourceGuid": "<resourceGUID>",
   "tags": null,
   "type": "Microsoft.Network/expressRoutePorts"
   }
   ```

   Utilisez la même procédure pour désactiver les ports en utilisant `AdminState = "Disabled"`.

## <a name="create-a-circuit"></a><a name="circuit"></a>Créer un circuit

Par défaut, vous pouvez créer 10 circuits dans l’abonnement qui contient la ressource ExpressRoute Direct. Le Support Microsoft peut relever la limite par défaut. Vous êtes chargé du suivi de la bande passante provisionnée et utilisée. La bande passante provisionnée est la somme de la bande passante de tous les circuits de la ressource ExpressRoute Direct. La bande passante utilisée est l’utilisation physique des interfaces physiques sous-jacentes.

Vous pouvez utiliser des bandes passantes de circuit supplémentaires dans ExpressRoute Direct uniquement pour prendre en charge les scénarios décrits ici. Les bandes passantes sont de 40 Gbits/s et de 100 Gbits/s.

La valeur de **SkuTier** peut être Local, Standard ou Premium.

**SkuFamily** peut uniquement être MeteredData. Un nombre illimité n’est pas pris en charge sur ExpressRoute Direct.

Créez un circuit dans la ressource ExpressRoute Direct :

  ```azurecli
  az network express-route create --express-route-port "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct" -n "Contoso-Direct-ckt" -g "Contoso-Direct-rg" --sku-family MeteredData --sku-tier Standard --bandwidth 100 Gbps
  ```

  Les autres bandes passantes sont de 5 Gbps, 10 Gbps et 40 Gbps.

  **Exemple de sortie**

  ```output
  {
  "allowClassicOperations": false,
  "allowGlobalReach": false,
  "authorizations": [],
  "bandwidthInGbps": 100.0,
  "circuitProvisioningState": "Enabled",
  "etag": "W/\"<etagnumber>\"",
  "expressRoutePort": {
    "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRoutePorts/Contoso-Direct",
    "resourceGroup": "Contoso-Direct-rg"
  },
  "gatewayManagerEtag": "",
  "id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/expressRouteCircuits/ERDirect-ckt-cli",
  "location": "westus",
  "name": "ERDirect-ckt-cli",
  "peerings": [],
  "provisioningState": "Succeeded",
  "resourceGroup": "Contoso-Direct-rg",
  "serviceKey": "<serviceKey>",
  "serviceProviderNotes": null,
  "serviceProviderProperties": null,
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "MeteredData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "stag": null,
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits"
  }  
  ```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur ExpressRoute Direct, consultez-en la [présentation](expressroute-erdirect-about.md).
