---
title: 'Tutoriel : Configurer des filtres de routage pour l’homologation Microsoft – Azure CLI'
description: Ce tutoriel décrit comment configurer des filtres de routage pour l’homologation Microsoft à l’aide d’Azure CLI
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: ac7fc5af21f11699331d41a074e88ae757170664
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975992"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-azure-cli"></a>Tutoriel : Configurer des filtres de routage pour le peering Microsoft : Azure CLI

> [!div class="op_single_selector"]
> * [Portail Azure](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [Azure CLI](how-to-routefilter-cli.md)
> 

Les filtres de routage permettent d’utiliser un sous-ensemble de services pris en charge via le peering Microsoft. Les étapes décrites dans cet article vous aident à configurer et à gérer des filtres de routage pour les circuits ExpressRoute.

Les services Microsoft 365, comme Exchange Online, SharePoint Online et Skype Entreprise sont accessibles par le biais de l’appairage Microsoft. Quand l’homologation Microsoft est configurée dans un circuit ExpressRoute, tous les préfixes liés à ces services sont publiés via les sessions BGP établies. Une valeur de communauté BGP est attachée à chaque préfixe pour identifier le service qui est proposé par le biais du préfixe. Pour obtenir la liste de valeurs de communauté BGP et des services auxquels elles sont mappées, consultez les [communautés BGP](expressroute-routing.md#bgp).

La connectivité à tous les services Azure et Microsoft 365 entraîne la publication d’un grand nombre de préfixes via le protocole BGP. Le grand nombre de préfixes augmente considérablement la taille des tables de routage gérées par les routeurs au sein de votre réseau. Si vous envisagez d’utiliser uniquement un sous-ensemble des services offerts par le biais du peering Microsoft, vous pouvez réduire la taille de vos tables de routage de deux manières. Vous pouvez :

* Filtrer les préfixes indésirables en appliquant des filtres de routage sur les communautés BGP. Le filtrage du routage est une pratique de mise en réseau standard courante.

* Définir des filtres de routage et les appliquer à votre circuit ExpressRoute. Un filtre de routage est une ressource qui vous permet de sélectionner la liste des services que vous envisagez d’utiliser via le peering Microsoft. Les routeurs ExpressRoute envoient uniquement la liste des préfixes qui appartiennent aux services identifiés dans le filtre de routage.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - obtenir les valeurs de communauté BGP ;
> - créer un filtre de routage et une règle de filtre ;
> - associer le filtre de routage à un circuit ExpressRoute.

### <a name="about-route-filters"></a><a name="about"></a>À propos des filtres de routage

Quand l’homologation Microsoft est configurée sur votre circuit ExpressRoute, les routeurs Microsoft Edge établissent une paire de sessions BGP avec vos routeurs de périphérie via votre fournisseur de connectivité. Aucun routage n’est publié sur votre réseau. Pour activer les annonces de routage sur votre réseau, vous devez associer un filtre de routage.

Un filtre de routage vous permet d’identifier les services que vous souhaitez utiliser via le peering Microsoft de votre circuit ExpressRoute. Il s’agit essentiellement de la liste autorisée de toutes les valeurs de communauté BGP. Une fois qu’une ressource de filtre de routage est définie et associée à un circuit ExpressRoute, tous les préfixes qui mappent aux valeurs de communauté BGP sont publiés sur votre réseau.

Pour associer des filtres de routage à des services Microsoft 365, vous devez être autorisé à utiliser les services Microsoft 365 par le biais d’ExpressRoute. Si vous n’êtes pas autorisé à utiliser les services Microsoft 365 par le biais d’ExpressRoute, l’association des filtres de routage échoue. Pour plus d’informations sur le processus d’autorisation, consultez [Azure ExpressRoute pour Microsoft 365](/microsoft-365/enterprise/azure-expressroute).

> [!IMPORTANT]
> Le peering Microsoft des circuits ExpressRoute configurés avant le 1er août 2017 entraînera la publication de tous les préfixes de service via le peering Microsoft, même si les filtres de routage ne sont pas définis. Le peering Microsoft des circuits ExpressRoute qui sont configurés le 1er août 2017 ou après n’entraînera la publication d’aucun préfixe tant qu’un filtre de routage n’aura pas été attaché au circuit.
> 

## <a name="prerequisites"></a>Prérequis

Pour vous connecter aux services via une homologation Microsoft, vous devez suivre les étapes de configuration suivantes :

* Vous devez disposer d’un circuit ExpressRoute actif où le peering Microsoft est provisionné. Vous pouvez utiliser les instructions suivantes pour accomplir ces tâches :
  * [Créez un circuit ExpressRoute](howto-circuit-cli.md) et faites-le activer par votre fournisseur de connectivité avant de poursuivre. Le circuit ExpressRoute doit être approvisionné et activé.
  * [Créez le peering Microsoft](howto-routing-cli.md) si vous gérez la session BGP directement. Sinon, demandez à votre fournisseur de connectivité de configurer le peering Microsoft pour votre circuit.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

Si vous choisissez d’installer et d’utiliser l’interface CLI en local, ce guide de démarrage nécessite Azure CLI version 2.0.28 ou ultérieure. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Vous connecter à votre compte Azure et sélectionner votre abonnement

Pour commencer votre configuration, connectez-vous à votre compte Azure. Si vous l’utilisez l’option « Essayer », vous êtes automatiquement connecté et pouvez ignorer l’étape de connexion. Utilisez les exemples suivants pour faciliter votre connexion :

```azurecli
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

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Obtenir la liste des préfixes et des valeurs de communauté BGP

1. Pour obtenir la liste des valeurs de communauté BGP et préfixes liés aux services accessibles via une homologation Microsoft, utilisez la cmdlet suivante :

    ```azurecli-interactive
    az network route-filter rule list-service-communities
    ```

1. Dressez la liste des valeurs de communauté BGP que vous souhaitez utiliser dans le filtre de routage.

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Créer un filtre de routage et une règle de filtre

Un filtre de routage ne peut avoir qu’une seule règle, et cette règle doit être de type « Autoriser ». Cette règle peut être associée à une liste des valeurs de communauté BGP. La commande `az network route-filter create` crée uniquement une ressource de filtre de routage. Après avoir créé la ressource, vous devez créer une règle et la joindre à l’objet de filtre de routage.

1. Pour créer une ressource de filtre de routage, utilisez la commande suivante :

    ```azurecli-interactive
    az network route-filter create -n MyRouteFilter -g MyResourceGroup
    ```

1. Pour créer une règle de filtre de routage, utilisez la commande suivante :
 
    ```azurecli-interactive
    az network route-filter rule create --filter-name MyRouteFilter -n CRM --communities 12076:5040 --access Allow -g MyResourceGroup
    ```

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Joindre le filtre de routage à un circuit ExpressRoute

Exécutez la commande suivante pour joindre le filtre de routage au circuit ExpressRoute :

```azurecli-interactive
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroupName --name MicrosoftPeering --route-filter MyRouteFilter
```

## <a name="common-tasks"></a><a name="tasks"></a>Tâches courantes

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Obtenir les propriétés d’un filtre de routage

Pour obtenir les propriétés d’un filtre de routage, utilisez la commande suivante :

```azurecli-interactive
az network route-filter show -g ExpressRouteResourceGroupName --name MyRouteFilter 
```

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Mettre à jour les propriétés d’un filtre de routage

Si le filtre de routage est déjà associé à un circuit, les mises à jour de la liste de communautés BGP propagent automatiquement les modifications de publication de préfixe via la session BGP établie. Vous pouvez mettre à jour la liste de communautés BGP de votre filtre de routage à l’aide de la commande suivante :

```azurecli-interactive
az network route-filter rule update --filter-name MyRouteFilter -n CRM -g ExpressRouteResourceGroupName --add communities '12076:5040' --add communities '12076:5010'
```

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Détacher un filtre de routage d’un circuit ExpressRoute

Une fois qu’un filtre de routage est détaché du circuit ExpressRoute, aucun préfixe n’est publié via la session BGP. Vous pouvez détacher un filtre de routage d’un circuit ExpressRoute à l’aide de la commande suivante :

```azurecli-interactive
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroupName --name MicrosoftPeering --remove routeFilter
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous ne pouvez supprimer un filtre de routage que s’il n’est associé à aucun circuit. Assurez-vous que le filtre de routage n’est pas associé à un circuit avant de tenter de le supprimer. Vous pouvez supprimer un filtre de routage à l’aide de la commande suivante :

```azurecli-interactive
az network route-filter delete -n MyRouteFilter -g MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les exemples de configuration de routeur, consultez :

> [!div class="nextstepaction"]
> [Exemples de configuration de routeur pour configurer et gérer le routage](expressroute-config-samples-routing.md)
