---
title: 'Démarrage rapide : Créer et modifier un circuit avec ExpressRoute - portail Azure'
description: Dans ce guide de démarrage rapide, vous découvrez comment créer, provisionner, vérifier, mettre à jour, supprimer et déprovisionner un circuit ExpressRoute à l’aide du portail Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: quickstart
ms.date: 10/21/2020
ms.author: duau
ms.openlocfilehash: b0fd844021e7398e061072d7939b782616580c1c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185196"
---
# <a name="quickstart-create-and-modify-an-expressroute-circuit"></a>Démarrage rapide : Création et modification d’un circuit ExpressRoute

Ce guide de démarrage rapide vous montre comment créer un circuit ExpressRoute à l’aide du portail Azure et du modèle de déploiement Azure Resource Manager. Vous pouvez également vérifier l’état d’un circuit, le mettre à jour, le supprimer ou le déprovisionner.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Examinez les [conditions préalables](expressroute-prerequisites.md) et les [flux de travail](expressroute-workflows.md) avant de commencer la configuration.
* Vous pouvez [visualiser une vidéo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit) avant de commencer afin de mieux comprendre les étapes.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Créer et approvisionner un circuit ExpressRoute

### <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

### <a name="create-a-new-expressroute-circuit"></a>Créez un circuit ExpressRoute

> [!IMPORTANT]
> Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Effectuez cette opération seulement quand le fournisseur de connectivité prêt à approvisionner le circuit.

Vous pouvez créer un circuit ExpressRoute en sélectionnant l'option permettant de créer une ressource. 

1. Dans le menu du portail Azure, sélectionnez **Créer une ressource**. Sélectionnez **Réseaux** > **ExpressRoute**, comme illustré dans l’image suivante :

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/create-expressroute-circuit-menu.png" alt-text="Création d’un circuit ExpressRoute":::

2. Après avoir sélectionné **ExpressRoute**, vous voyez s’afficher la page **Créer ExpressRoute**. Indiquez le **groupe de ressources**, la **région** et le **nom** du circuit. Ensuite, sélectionnez **Next: Configuration >** .

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-create-basic.png" alt-text="Configurer le groupe de ressources et la région":::

3. Lorsque vous entrez les valeurs dans cette page, veillez à spécifier le niveau de référence (SKU) approprié (Local, Standard ou Premium) et le modèle de facturation des données (Illimitées ou Limitées).

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-create-configuration.png" alt-text="Configurer le circuit":::
    
    * **Type de port** détermine si vous vous connectez à un fournisseur de services ou directement au réseau global de Microsoft dans un emplacement de peering.
    * **Créer ou importer à partir d’un classique** détermine si un circuit est en cours de création ou si vous migrez un circuit classique vers Azure Resource Manager.
    * Le **fournisseur** est le service Internet à partir duquel vous allez demander votre service.
    * L’**Emplacement de peering** est l’emplacement physique où vous effectuez le peering auprès de Microsoft.

    > [!IMPORTANT]
    > L’emplacement de peering indique [l’emplacement physique](expressroute-locations.md) où vous effectuez le peering auprès de Microsoft. Cet emplacement n’est **pas** lié à la propriété « Emplacement », qui fait référence à la zone géographique où se trouve le fournisseur de ressources réseau Azure. Bien que ces éléments ne soient pas liés, nous vous conseillons de choisir un fournisseur de ressources réseau géographiquement proche de l’emplacement de peering du circuit.

    * La **référence SKU** détermine si un module complémentaire ExpressRoute local, ExpressRoute standard ou ExpressRoute premium est activé. Vous pouvez spécifier **Local** pour obtenir la référence SKU locale, **Standard** pour obtenir la référence SKU standard ou **Premium** pour le module complémentaire Premium. Vous pouvez changer la référence SKU pour activer le module complémentaire Premium.
    > [!IMPORTANT]
    > Vous ne pouvez pas remplacer la référence SKU de **Standard/Premium** par celle de **Local**.
    
    * Le **modèle de facturation** détermine le type de facturation. Vous pouvez spécifier **Limité** pour un forfait de données limité, et **Illimité** pour un forfait de données illimité. Vous pouvez changer le type de facturation de **Limité** à **Illimité**.

    > [!IMPORTANT]
    > En revanche, vous ne pouvez pas modifier le type de **Illimité** à **Limité**.

    * **Autoriser les opérations classiques** permet de lier des réseaux virtuels classiques au circuit.

### <a name="view-the-circuits-and-properties"></a>Afficher les circuits et les propriétés

**Afficher tous les circuits**

Vous pouvez afficher tous les circuits que vous avez créés en sélectionnant **Tous les services > Mise en réseau > circuit ExpressRoute** dans le menu de gauche.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-menu.png" alt-text="Menu du circuit ExpressRoute":::

Tous les circuits ExpressRoute créés dans l’abonnement sont listés ici.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-list.png" alt-text="Liste du circuit ExpressRoute":::

**Afficher les propriétés**

Vous pouvez afficher les propriétés du circuit en sélectionnant celui-ci. Dans la page **Vue d’ensemble** de votre circuit, la clé de service s’affiche dans le champ du même nom. Faites référence à la clé de service de votre circuit, puis transmettez-la au fournisseur de services pour terminer le processus d’approvisionnement. La clé de service est spécifique à votre circuit.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-overview.png" alt-text="Afficher les propriétés":::

### <a name="send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>Envoyer la clé de service à votre fournisseur de connectivité pour l’approvisionnement

Dans cette page, **Statut du fournisseur** vous donne des informations sur l’état actuel du provisionnement du côté fournisseur de services. **Statut du circuit** vous indique l’état du côté Microsoft. Pour plus d’informations sur les états d’approvisionnement des circuits, consultez l’article [Flux de travail](expressroute-workflows.md#expressroute-circuit-provisioning-states) .

Quand vous créez un circuit ExpressRoute, ce circuit affiche l’état suivant :

État du fournisseur : **Non approvisionné**<BR>
État du circuit : **Activé**

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-overview-provisioning-state.png" alt-text="Démarre le processus de provisionnement":::

Le circuit passe à l’état suivant quand le fournisseur de connectivité l’active pour vous :

État du fournisseur : **Approvisionnement**<BR>
État du circuit : **Activé**

Pour être utilisé, le circuit ExpressRoute doit être dans l’état suivant :

État du fournisseur : **Approvisionné**<BR>
État du circuit : **Activé**

### <a name="periodically-check-the-status-and-the-state-of-the-circuit-key"></a>Vérifier régulièrement le statut et l’état de la clé du circuit

Vous pouvez afficher les propriétés du circuit qui vous intéressent en le sélectionnant. Vérifiez le **Statut du fournisseur** et vérifiez qu’il est passé à **Approvisionné** avant de continuer.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/provisioned.png" alt-text="Statut du circuit et du fournisseur":::

### <a name="create-your-routing-configuration"></a>Créer votre configuration de routage

Pour obtenir des instructions pas à pas, consultez l’article [Configuration du routage des circuits ExpressRoute](expressroute-howto-routing-portal-resource-manager.md) pour créer et modifier des peerings de circuit.

> [!IMPORTANT]
> Ces instructions s’appliquent seulement aux circuits créés avec des fournisseurs de services proposant des services de connectivité de couche 2. Si vous utilisez un fournisseur de services proposant des services gérés de couche 3 (généralement un VPN IP, comme MPLS), votre fournisseur de connectivité configure et gère le routage pour vous.

### <a name="link-a-virtual-network-to-an-expressroute-circuit"></a>Lier un réseau virtuel à un circuit ExpressRoute

Maintenant, vous devez lier un réseau virtuel à votre circuit ExpressRoute. Utilisez l’article [Liaison de réseaux virtuels à des circuits ExpressRoute](expressroute-howto-linkvnet-arm.md) quand vous travaillez avec le modèle de déploiement Resource Manager.

## <a name="getting-the-status-of-an-expressroute-circuit"></a><a name="status"></a>Récupération de l’état d’un circuit ExpressRoute

Vous pouvez afficher l’état d’un circuit en sélectionnant celui-ci et en affichant la page Vue d’ensemble.

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Modification d’un circuit ExpressRoute

Vous pouvez modifier certaines propriétés d'un circuit ExpressRoute sans affecter la connectivité. Vous pouvez modifier la bande passante, la référence (SKU), le modèle de facturation et autoriser les opérations classiques sur la page **Configuration**. Pour plus d’informations sur les limites et limitations, voir le [FAQ ExpressRoute](expressroute-faqs.md).

Vous pouvez effectuer les tâches suivantes sans entraîner de temps d’arrêt :

* Activer ou désactiver un module complémentaire ExpressRoute Premium pour votre circuit ExpressRoute.

> [!IMPORTANT]
  > Le passage de la référence SKU de **Standard/Premium** à **Local** n’est pas pris en charge.

* Augmenter la bande passante de votre circuit ExpressRoute à condition que la capacité disponible sur le port le permette.

  > [!IMPORTANT]
  > La rétrogradation de la bande passante d'un circuit n'est pas prise en charge.

* Modifiez le plan de mesure de *Données limitées* en *Données illimitées*.

  > [!IMPORTANT]
  > La modification du plan de limitation de **Données illimitées** à **Données limitées** n’est pas prise en charge.

* Vous pouvez activer et désactiver *Autoriser les opérations classiques*.
  > [!IMPORTANT]
  > Vous devrez peut-être recréer le circuit ExpressRoute si la capacité sur le port existant est inappropriée. Vous ne pouvez pas mettre le circuit à niveau si aucune capacité supplémentaire n’est disponible à cet emplacement.
  >
  > Même si vous pouvez aisément mettre à niveau la bande passante, vous ne pouvez pas réduire la bande passante d’un circuit ExpressRoute sans interrompre le service. Le fait de passer à un niveau inférieur de bande passante vous oblige à annuler l’approvisionnement du circuit ExpressRoute, puis à réapprovisionner un nouveau circuit ExpressRoute.
  >
  > La désactivation de l’opération du module complémentaire Premium peut échouer si vous utilisez davantage de ressources que ce qui est autorisé pour le circuit standard.

Pour modifier un circuit ExpressRoute, sélectionnez **Configuration**.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-configuration.png" alt-text="Modifier le circuit":::

## <a name="deprovisioning-an-expressroute-circuit"></a><a name="delete"></a>Déprovisionnement d’un circuit ExpressRoute

Si l’état d’approvisionnement du fournisseur de services du circuit ExpressRoute est **En cours d’approvisionnement** ou **Approvisionné**, vous devez vous mettre en relation avec votre fournisseur de services pour annuler l’approvisionnement du circuit de son côté. Nous continuons à réserver des ressources et à vous facturer jusqu’à ce que le fournisseur de services termine le désapprovisionnement du circuit et nous en avertisse.

> [!NOTE]
>* Vous devez dissocier *tous les réseaux virtuels* du circuit ExpressRoute avant le déprovisionnement. Si cette opération échoue, vérifiez si certains de vos réseaux virtuels sont liés au circuit.
>* Si le fournisseur de services a annulé l’approvisionnement du circuit (l’état d’approvisionnement du fournisseur de services affiche la valeur **Non approvisionné**), vous pouvez supprimer le circuit. Cette opération arrête la facturation du circuit.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer votre circuit ExpressRoute en sélectionnant l’icône **Supprimer**. Avant de continuer, assurez-vous que l’état du fournisseur est *Non approvisionné*.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-delete.png" alt-text="Supprimer le circuit":::

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé votre circuit, passez à l’étape suivante :

> [!div class="nextstepaction"]
> [Créer et modifier le routage le routage pour votre circuit ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
