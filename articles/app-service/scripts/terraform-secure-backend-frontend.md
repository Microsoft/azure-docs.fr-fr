---
title: 'Terraform : Déployer une application web frontale et une application web principale en toute sécurité avec une intégration au réseau virtuel et un point de terminaison privé'
description: Découvrez comment le fournisseur Terraform pour App Service permet de déployer deux applications web connectées en toute sécurité avec un point de terminaison privé et une intégration au réseau virtuel
author: ericgre
ms.assetid: 3e5d1bbd-5581-40cc-8f65-bc74f1802156
ms.topic: sample
ms.date: 08/10/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: c1de8ebbd9ad381628cfeb19413baa295b42b3db
ms.sourcegitcommit: a07a01afc9bffa0582519b57aa4967d27adcf91a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91739831"
---
# <a name="create-two-web-apps-connected-securely-with-private-endpoint-and-vnet-integration"></a>Créer deux applications web connectées en toute sécurité avec un point de terminaison privé et une intégration au réseau virtuel

Cet article présente un exemple d’utilisation d’un [point de terminaison privé](../networking/private-endpoint.md) et d’une [intégration au réseau virtuel](../web-sites-integrate-with-vnet.md) régional pour connecter deux applications web (frontale et principale) en toute sécurité en procédant comme suit :
- Déployer un réseau virtuel.
- Créer le premier sous-réseau pour l’intégration.
- Créer le deuxième sous-réseau pour le point de terminaison privé. Vous devez définir un paramètre spécifique pour désactiver les stratégies réseau.
- Déployer un plan App Service de type PremiumV2, ou PremiumV3, nécessaire pour la fonctionnalité de point de terminaison privé
- Créer l’application web frontale avec des paramètres d’application spécifiques pour utiliser la zone DNS privée. [En savoir plus](../web-sites-integrate-with-vnet.md#azure-dns-private-zones).
- Connecter l’application web frontale au sous-réseau d’intégration.
- Créer l’application web principale.
- Créer la zone DNS privée avec le nom de la zone de liaison privée pour l’application web privatelink.azurewebsites.net.
- Lier cette zone au réseau virtuel.
- Créer le point de terminaison privé pour l’application web principale dans le sous-réseau du point de terminaison, puis enregistrer les noms DNS (site web et SCM) dans la zone privée DNS créée précédemment.

## <a name="how-to-use-terraform-in-azure"></a>Comment utiliser Terraform dans Azure

Pour savoir comment utiliser Terraform avec Azure, consultez la [documentation d’Azure](/azure/developer/terraform/).

## <a name="the-complete-terraform-file"></a>Fichier Terraform complet

Pour utiliser ce fichier, vous devez modifier la propriété de nom pour les ressources frontwebapp et backwebapp (le nom d’application web doit être un nom DNS unique dans le monde entier). 

```hcl
provider "azurerm" {
  version = "~>2.0"
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "appservice-rg"
  location = "francecentral"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "integrationsubnet" {
  name                 = "integrationsubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
  delegation {
    name = "delegation"
    service_delegation {
      name = "Microsoft.Web/serverFarms"
    }
  }
}

resource "azurerm_subnet" "endpointsubnet" {
  name                 = "endpointsubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.2.0/24"]
  enforce_private_link_endpoint_network_policies = true
}

resource "azurerm_app_service_plan" "appserviceplan" {
  name                = "appserviceplan"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  sku {
    tier = "Premiumv2"
    size = "P1v2"
  }
}

resource "azurerm_app_service" "frontwebapp" {
  name                = "frontwebapp20200810"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.appserviceplan.id

  app_settings = {
    "WEBSITE_DNS_SERVER": "168.63.129.16",
    "WEBSITE_VNET_ROUTE_ALL": "1"
  }
}

resource "azurerm_app_service_virtual_network_swift_connection" "vnetintegrationconnection" {
  app_service_id  = azurerm_app_service.frontwebapp.id
  subnet_id       = azurerm_subnet.integrationsubnet.id
}

resource "azurerm_app_service" "backwebapp" {
  name                = "backwebapp20200810"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.appserviceplan.id
}

resource "azurerm_private_dns_zone" "dnsprivatezone" {
  name                = "privatelink.azurewebsites.net"
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_private_dns_zone_virtual_network_link" "dnszonelink" {
  name = "dnszonelink"
  resource_group_name = azurerm_resource_group.rg.name
  private_dns_zone_name = azurerm_private_dns_zone.dnsprivatezone.name
  virtual_network_id = azurerm_virtual_network.vnet.id
}

resource "azurerm_private_endpoint" "privateendpoint" {
  name                = "backwebappprivateendpoint"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  subnet_id           = azurerm_subnet.endpointsubnet.id

  private_dns_zone_group {
    name = "privatednszonegroup"
    private_dns_zone_ids = [azurerm_private_dns_zone.dnsprivatezone.id]
  }

  private_service_connection {
    name = "privateendpointconnection"
    private_connection_resource_id = azurerm_app_service.backwebapp.id
    subresource_names = ["sites"]
    is_manual_connection = false
  }
}
```




## <a name="next-steps"></a>Étapes suivantes


> [En savoir plus sur l’utilisation de Terraform dans Azure](/azure/developer/terraform/)