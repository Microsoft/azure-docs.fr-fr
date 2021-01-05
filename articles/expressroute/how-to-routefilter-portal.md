---
title: 'Tutoriel : Configurer des filtres de routage pour l’homologation Microsoft – Portail Azure'
description: Ce tutoriel décrit comment configurer des filtres de routage pour l’homologation Microsoft à l’aide du portail Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: d142028d197f9e279b5f1e05757946dc40d2c153
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109134"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-using-the-azure-portal"></a>Tutoriel : Configurer des filtres de routage pour l’homologation Microsoft à l’aide du portail Azure

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

Quand l’homologation Microsoft est configurée sur votre circuit ExpressRoute, les routeurs de périphérie Microsoft établissent une paire de sessions BGP avec vos routeurs de périphérie via votre fournisseur de connectivité. Aucun routage n’est publié sur votre réseau. Pour activer les annonces de routage sur votre réseau, vous devez associer un filtre de routage.

Un filtre de routage vous permet d’identifier les services que vous souhaitez utiliser via le peering Microsoft de votre circuit ExpressRoute. Il s’agit essentiellement de la liste autorisée de toutes les valeurs de communauté BGP. Une fois qu’une ressource de filtre de routage est définie et associée à un circuit ExpressRoute, tous les préfixes qui mappent aux valeurs de communauté BGP sont publiés sur votre réseau.

Pour associer des filtres de routage à des services Microsoft 365, vous devez être autorisé à utiliser les services Microsoft 365 par le biais d’ExpressRoute. Si vous n’êtes pas autorisé à utiliser les services Microsoft 365 par le biais d’ExpressRoute, l’association des filtres de routage échoue. Pour plus d’informations sur le processus d’autorisation, consultez [Azure ExpressRoute pour Microsoft 365](/microsoft-365/enterprise/azure-expressroute).

> [!IMPORTANT]
> Le peering Microsoft des circuits ExpressRoute configurés avant le 1er août 2017 entraînera la publication de tous les préfixes de service via le peering Microsoft, même si les filtres de routage ne sont pas définis. Le peering Microsoft des circuits ExpressRoute qui sont configurés le 1er août 2017 ou après n’entraînera la publication d’aucun préfixe tant qu’un filtre de routage n’aura pas été attaché au circuit.
> 

## <a name="prerequisites"></a>Prérequis

- Examinez les [conditions préalables](expressroute-prerequisites.md) et les [flux de travail](expressroute-workflows.md) avant de commencer la configuration.

- Vous devez disposer d’un circuit ExpressRoute actif où le peering Microsoft est provisionné. Vous pouvez utiliser les instructions suivantes pour accomplir ces tâches :
  - [Créez un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) et demandez à votre fournisseur de connectivité de l’activer avant de poursuivre. Le circuit ExpressRoute doit être approvisionné et activé.
  - [Créez le peering Microsoft](expressroute-howto-routing-portal-resource-manager.md) si vous gérez la session BGP directement. Sinon, demandez à votre fournisseur de connectivité de configurer le peering Microsoft pour votre circuit.

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Obtenir la liste des préfixes et des valeurs de communauté BGP

### <a name="get-a-list-of-bgp-community-values"></a>Obtenir la liste des valeurs de communauté BGP

Les valeurs de communauté BGP associées aux services accessibles via le peering Microsoft sont disponibles sur la page [Configuration requise pour le routage ExpressRoute](expressroute-routing.md).

### <a name="make-a-list-of-the-values-that-you-want-to-use"></a>Dresser la liste des valeurs que vous souhaitez utiliser

Dressez la liste des [valeurs de communauté BGP](expressroute-routing.md#bgp) que vous souhaitez utiliser dans le filtre de routage. 

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Créer un filtre de routage et une règle de filtre

Un filtre de routage ne peut avoir qu’une seule règle, et cette règle doit être de type « Autoriser ». Cette règle peut être associée à une liste des valeurs de communauté BGP.

1. Sélectionnez **Créer une ressource**, puis recherchez *Filtre de routage* comme illustré dans l’image suivante :

    :::image type="content" source="./media/how-to-routefilter-portal/create-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

1. Placez le filtre de routage dans un groupe de ressources. Vérifiez que l’emplacement est le même que celui du circuit ExpressRoute. Sélectionnez **Vérifier + créer**, puis **Créer**.

    :::image type="content" source="./media/how-to-routefilter-portal/create-route-filter-basic.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

### <a name="create-a-filter-rule"></a>Créer une règle de filtre

1. Pour ajouter et mettre à jour des règles, sélectionnez l’onglet Gérer la règle pour votre filtre de routage.

    :::image type="content" source="./media/how-to-routefilter-portal/manage-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

1. Sélectionnez dans la liste déroulante les services auxquels vous souhaitez vous connecter et enregistrer la règle lorsque vous avez terminé.

    :::image type="content" source="./media/how-to-routefilter-portal/add-route-filter-rule.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Joindre le filtre de routage à un circuit ExpressRoute

Associez le filtre de routage à un circuit en sélectionnant le bouton **+ Ajouter un circuit** et en sélectionnant le circuit ExpressRoute dans la liste déroulante.

:::image type="content" source="./media/how-to-routefilter-portal/add-circuit-to-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

Si le fournisseur de connectivité configure l’homologation pour votre circuit ExpressRoute, actualisez le circuit dans la page Circuit ExpressRoute avant de sélectionner le bouton **+ Ajouter un circuit**.

:::image type="content" source="./media/how-to-routefilter-portal/refresh-express-route-circuit.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

## <a name="common-tasks"></a><a name="tasks"></a>Tâches courantes

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Obtenir les propriétés d’un filtre de routage

Vous pouvez afficher les propriétés d’un filtre de routage lorsque vous ouvrez la ressource dans le portail.

:::image type="content" source="./media/how-to-routefilter-portal/view-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Mettre à jour les propriétés d’un filtre de routage

1. Vous pouvez mettre à jour la liste des valeurs de communauté BGP associées à un circuit en sélectionnant le bouton **Gérer la règle**.

    :::image type="content" source="./media/how-to-routefilter-portal/update-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

1. Sélectionnez les communautés de service de votre choix, puis **Enregistrer**.

    :::image type="content" source="./media/how-to-routefilter-portal/add-route-filter-rule.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Détacher un filtre de routage d’un circuit ExpressRoute

Pour dissocier un circuit du filtre de routage, cliquez avec le bouton droit sur le circuit, puis sélectionnez **Dissocier**.

:::image type="content" source="./media/how-to-routefilter-portal/detach-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::


## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer un filtre de routage en sélectionnant le bouton **Supprimer**. Avant de procéder, assurez-vous que le filtre de routage n’est associé à aucun circuit.

:::image type="content" source="./media/how-to-routefilter-portal/delete-route-filter.png" alt-text="Capture d’écran montrant la page Filtre de routage":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les exemples de configuration de routeur, consultez :

> [!div class="nextstepaction"]
> [Exemples de configuration de routeur pour configurer et gérer le routage](expressroute-config-samples-routing.md)
