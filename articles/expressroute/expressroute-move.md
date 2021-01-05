---
title: 'ExpressRoute : Déplacer des circuits de Classic vers Azure Resource Manager'
description: Découvrez les enjeux de la migration d’un circuit ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Azure Resource Manager.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/07/2018
ms.author: duau
ms.openlocfilehash: 649174e59c3e58b4675393ce665cf765016d5551
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331547"
---
# <a name="moving-expressroute-circuits-from-the-classic-to-the-resource-manager-deployment-model"></a>Migration de circuits ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager
Cet article décrit les enjeux de la migration d’un circuit ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager.

Un circuit ExpressRoute permet de se connecter à des réseaux virtuels déployés à la fois dans les modèles classique et Resource Manager. Quel que soit son mode de création, un circuit ExpressRoute peut désormais se relier à des réseaux virtuels dans les deux modèles de déploiement.

![Un circuit ExpressRoute qui se lie à des réseaux virtuels dans les deux modèles de déploiement](./media/expressroute-move/expressroute-move-1.png)

## <a name="expressroute-circuits-that-are-created-in-the-classic-deployment-model"></a>Circuits ExpressRoute créés dans le modèle de déploiement classique
Les circuits ExpressRoute créés dans le modèle de déploiement classique doivent d’abord être migrés vers le modèle de déploiement Resource Manager, afin d’assurer la connectivité aux modèles de déploiement classique et Resource Manager. La migration d’une connexion n’entraîne ni perte ni interruption de la connectivité. Dans le modèle de déploiement classique, toutes les liaisons d’un circuit à un réseau virtuel, que ce soit dans le même abonnement ou dans différents abonnements, sont conservées.

Une fois la migration effectuée, le circuit ExpressRoute se présente, s’exécute et se comporte comme un circuit ExpressRoute créé dans le modèle de déploiement Resource Manager. Vous pouvez maintenant créer des connexions à des réseaux virtuels dans le modèle de déploiement Resource Manager.

Une fois le circuit ExpressRoute migré vers le modèle de déploiement Resource Manager, vous ne pouvez gérer son cycle de vie qu’avec le modèle de déploiement Resource Manager. Autrement dit, les opérations telles que l’ajout, la mise à jour ou la suppression de peerings, la mise à jour des propriétés d’un circuit (bande passante, référence (SKU) et type de facturation) et la suppression de circuits ne sont possibles que dans le modèle de déploiement Resource Manager. Pour plus d’informations sur la gestion de l’accès aux deux modèles de déploiement, consultez la section ci-dessous relative aux circuits créés dans le modèle de déploiement Resource Manager.

La procédure de migration ne nécessite aucune intervention de votre fournisseur de connectivité.

## <a name="expressroute-circuits-that-are-created-in-the-resource-manager-deployment-model"></a>Circuits ExpressRoute créés dans le modèle de déploiement Resource Manager
Vous pouvez rendre les circuits ExpressRoute créés dans le modèle de déploiement Resource Manager, accessibles à partir des deux modèles de déploiement. Tout circuit ExpressRoute de votre abonnement peut être accessible à partir de ces deux modèles de déploiement.

* Par défaut, les circuits ExpressRoute créés dans le modèle de déploiement Resource Manager n’ont pas accès au modèle de déploiement classique.
* Les circuits ExpressRoute migrés du modèle de déploiement classique vers le modèle de déploiement Resource Manager sont, par défaut, accessibles à partir des deux modèles de déploiement.
* Un circuit ExpressRoute a toujours accès au modèle de déploiement Resource Manager, qu’il ait été créé dans le modèle de déploiement Resource Manager ou dans le modèle de déploiement classique. Cela signifie que vous pouvez créer des connexions à des réseaux virtuels dans le modèle de déploiement Resource Manager, en suivant les instructions concernant la [liaison à des réseaux virtuels](expressroute-howto-linkvnet-arm.md).
* L’accès au modèle de déploiement classique est régi par le paramètre **allowClassicOperations** du circuit ExpressRoute.

> [!IMPORTANT]
> Tous les quotas décrits sur la page dédiée aux [limites de service](../azure-resource-manager/management/azure-subscription-service-limits.md) s’appliquent. Par exemple, un circuit standard peut avoir jusqu’à 10 liaisons/connexions de réseau virtuel dans les modèles de déploiement classique et Resource manager.
> 
> 

## <a name="controlling-access-to-the-classic-deployment-model"></a>Contrôle de l’accès au modèle de déploiement classique
Vous pouvez autoriser un circuit ExpressRoute à se connecter à des réseaux virtuels dans les deux modèles de déploiement, en configurant son paramètre **allowClassicOperations** .

Si vous attribuez la valeur TRUE au paramètre **allowClassicOperations** , vous pouvez connecter des réseaux virtuels des deux modèles de déploiement au circuit ExpressRoute. Vous pouvez vous connecter à des réseaux virtuels dans le modèle de déploiement classique en suivant les instructions sur la [liaison de réseaux virtuels dans le modèle de déploiement classique](expressroute-howto-linkvnet-classic.md). Vous pouvez vous connecter à des réseaux virtuels dans le modèle de déploiement Resource Manager, en suivant les instructions sur la [liaison de réseaux virtuels dans le modèle de déploiement Resource Manager](expressroute-howto-linkvnet-arm.md).

Si vous attribuez la valeur FALSE au paramètre **allowClassicOperations** , le circuit n’est pas accessible à partir du modèle de déploiement classique. Toutes les liaisons de réseau virtuel dans le modèle de déploiement classique sont cependant conservées. Dans ce cas, le circuit ExpressRoute n’est pas visible dans le modèle de déploiement classique.

## <a name="supported-operations-in-the-classic-deployment-model"></a>Opérations prises en charge dans le modèle de déploiement classique
Les opérations classiques suivantes sont prises en charge sur un circuit ExpressRoute si vous attribuez la valeur TRUE au paramètre **allowClassicOperations** .

* Obtenir des informations sur le circuit ExpressRoute
* Créer/mettre à jour/obtenir/supprimer des liaisons à des réseaux virtuels classiques
* Créer/mettre à jour/obtenir/supprimer des autorisations de liaison de réseau virtuel pour une connectivité entre plusieurs abonnements

Cependant, quand **allowClassicOperations** est défini sur TRUE, vous ne pouvez pas effectuer les opérations classiques suivantes :

* Créer/mettre jour/obtenir/supprimer des peerings BGP (Border Gateway Protocol) pour les peerings privés Azure, les peerings publics Azure et les peerings Microsoft
* Supprimer des circuits ExpressRoute

## <a name="communication-between-the-classic-and-the-resource-manager-deployment-models"></a>Communication entre les modèles de déploiement classique et Resource Manager
Le circuit ExpressRoute se comporte comme un pont entre les modèles de déploiement classique et Resource Manager. Le trafic entre les machines virtuelles des réseaux virtuels dans le modèle de déploiement classique et celles des réseaux virtuels dans le modèle de déploiement Resource Manager transite via ExpressRoute si les deux réseaux virtuels sont reliés au même circuit ExpressRoute.

Le débit cumulé est limité par le débit de la passerelle du réseau virtuel. Dans ce cas, le trafic n’entre ni dans vos réseaux ni dans ceux de votre fournisseur de connectivité. Le trafic entre les réseaux virtuels reste entièrement confiné dans le réseau Microsoft.

## <a name="access-to-azure-public-and-microsoft-peering-resources"></a>Accès aux ressources de peering public Azure et Microsoft
Vous conservez l’accès aux ressources généralement accessibles via le peering public Azure et le peering Microsoft, sans interruption.  

## <a name="whats-supported"></a>Opérations prises en charge
Cette section décrit les opérations prises en charge pour les circuits ExpressRoute :

* Vous pouvez utiliser un circuit ExpressRoute pour accéder aux réseaux virtuels déployés dans les modèles classique et Resource Manager.
* Vous pouvez migrer un circuit ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager. Après sa migration, le circuit ExpressRoute se présente, s’exécute et se comporte comme n’importe quel circuit ExpressRoute créé dans le modèle de déploiement Resource Manager.
* Seul le circuit ExpressRoute peut faire l’objet d’une migration. Cette opération ne permet pas de migrer les liaisons de circuit, les réseaux virtuels et les passerelles VPN.
* Une fois le circuit ExpressRoute migré vers le modèle de déploiement Resource Manager, vous ne pouvez gérer son cycle de vie qu’avec le modèle de déploiement Resource Manager. Autrement dit, les opérations telles que l’ajout, la mise à jour ou la suppression de peerings, la mise à jour des propriétés d’un circuit (bande passante, référence (SKU) et type de facturation) et la suppression de circuits ne sont possibles que dans le modèle de déploiement Resource Manager.
* Le circuit ExpressRoute se comporte comme un pont entre les modèles de déploiement classique et Resource Manager. Le trafic entre les machines virtuelles des réseaux virtuels dans le modèle de déploiement classique et celles des réseaux virtuels dans le modèle de déploiement Resource Manager transite via ExpressRoute si les deux réseaux virtuels sont reliés au même circuit ExpressRoute.
* La connectivité entre différents abonnements est prise en charge dans les modèles de déploiement classique et Resource Manager.
* Après avoir déplacé un circuit ExpressRoute entre le modèle classique et le modèle Azure Resource Manager, vous pouvez [migrer les réseaux virtuels liés au circuit ExpressRoute](expressroute-migration-classic-resource-manager.md).

## <a name="whats-not-supported"></a>Ce qui n'est pas pris en charge
Cette section décrit les opérations non prises en charge pour les circuits ExpressRoute :

* Gestion du cycle de vie d’un circuit ExpressRoute à partir du modèle de déploiement classique.
* Prise en charge du contrôle Azure RBAC (Role-Based Access Control) dans le modèle de déploiement classique. Vous ne pouvez effectuer aucun contrôle Azure RBAC sur un circuit dans le modèle de déploiement classique. Un administrateur/coadministrateur de l’abonnement peut lier des réseaux virtuels au circuit (ou les en dissocier).

## <a name="configuration"></a>Configuration
Suivez les instructions dans [Migration d’un circuit ExpressRoute du modèle de déploiement classique vers le modèle de déploiement Resource Manager](expressroute-howto-move-arm.md).

## <a name="next-steps"></a>Étapes suivantes
* [Migrer des réseaux virtuels associés à ExpressRoute d’un déploiement classique vers Resource Manager](expressroute-migration-classic-resource-manager.md)
* Pour plus d’informations sur les workflows, consultez [Workflows d’approvisionnement du circuit ExpressRoute et états du circuit](expressroute-workflows.md).
* Pour configurer votre connexion ExpressRoute :
  
  * [Création d’un circuit ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuration du routage](expressroute-howto-routing-arm.md)
  * [Lier un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)

