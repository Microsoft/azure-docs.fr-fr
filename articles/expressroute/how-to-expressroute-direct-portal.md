---
title: 'Azure ExpressRoute : Configurer ExpressRoute Direct avec le portail'
description: Cette page vous permet de configurer ExpressRoute Direct à l’aide du portail.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 09/29/2020
ms.author: duau
ms.openlocfilehash: 56638f14565f76b0a2fc252b81dba3dae9e53dd8
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289438"
---
# <a name="create-expressroute-direct-using-the-azure-portal"></a>Créer une ressource ExpressRoute Direct à l’aide du portail Azure

Cet article vous montre comment créer une ressource ExpressRoute Direct à l’aide du portail Azure.
ExpressRoute Direct vous permet de vous connecter directement au réseau global de Microsoft à des emplacements de peering stratégiquement répartis dans le monde entier. Pour plus d’informations, consultez [A propos d’ExpressRoute Direct](expressroute-erdirect-about.md).

## <a name="before-you-begin"></a><a name="before"></a>Avant de commencer

Vérifiez que le fournisseur de ressources **Microsoft.Network** est enregistré dans votre abonnement. L’inscription d’un fournisseur de ressources configure votre abonnement pour travailler avec le fournisseur de ressources.

1. Accédez aux paramètres de votre abonnement comme décrit dans [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md).
1. Dans votre abonnement, pour **Fournisseurs de ressources** , vérifiez que le fournisseur **Microsoft.Network** affiche un état **Inscrit**. Si le fournisseur de ressources Microsoft.Network ne figure pas dans la liste des fournisseurs inscrits, ajoutez-le.

## <a name="create-expressroute-direct"></a><a name="create-erdir"></a>Créer une ressource ExpressRoute Direct

1. Dans le menu du [portail Azure](https://portal.azure.com) ou dans la page **Accueil** , sélectionnez **Créer une ressource**.

1. Dans la page **Nouveau** , dans le champ **_Rechercher dans la Place de marché_ *_, tapez _* ExpressRoute Direct** , puis sélectionnez **Entrée** pour obtenir les résultats de la recherche.

1. Dans les résultats, sélectionnez **ExpressRoute Direct**.

1. Dans la page **ExpressRoute Direct** , sélectionnez **Créer** pour ouvrir la page **Créer une ressource ExpressRoute Direct**.

1. Commencez par renseigner les champs de la page **De base**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/basics.png" alt-text="Page De base":::

    * **Abonnement** : abonnement Azure que vous voulez utiliser pour créer une ressource ExpressRoute Direct. La ressource ExpressRoute Direct et les circuits ExpressRoute doivent se trouver dans le même abonnement.
    * **Groupe de ressources** : groupe de ressources Azure dans lequel la nouvelle ressource ExpressRoute Direct sera créée. Si vous n’avez pas de groupe de ressources, vous pouvez en créer un.
    * **Région**  : Région publique Azure dans laquelle est créée la ressource.
    * **Name**  : nom de la nouvelle ressource ExpressRoute Direct.

1. Ensuite, renseignez les champs de la page **Configuration**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration.png" alt-text="Capture d’écran montrant la page « Créer une ressource ExpressRoute Direct » avec l’onglet « Configuration » sélectionné.":::

    * **Emplacement du peering**  : emplacement du peering où vous allez vous connecter à la ressource ExpressRoute Direct. Pour plus d’informations sur les emplacements de peering, consultez [Emplacements ExpressRoute](expressroute-locations-providers.md).
   * **Bande passante**  : bande passante de paire de ports que vous voulez réserver. ExpressRoute Direct prend en charge les deux options de bande passante de 10 Go et de 100 Go. Si la bande passante souhaitée n’est pas disponible à l’emplacement de peering spécifié, [ouvrez une demande de support dans le portail Azure](https://aka.ms/azsupt).
   * **Encapsulation**  : ExpressRoute Direct prend en charge l’encapsulation QinQ et Dot1Q.
     * Si l’encapsulation QinQ est sélectionnée, chaque circuit ExpressRoute se voit affecter dynamiquement un S-Tag et est unique sur l’ensemble de la ressource ExpressRoute Direct.
     *  Chaque C-Tag sur le circuit doit être unique sur le circuit, mais pas sur ExpressRoute Direct.
     * Si l’encapsulation Dot1Q est sélectionnée, vous devez gérer l’unicité de C-Tag (VLAN) sur l’ensemble de la ressource ExpressRoute Direct.
     >[!IMPORTANT]
     >ExpressRoute Direct ne peut être que d’un seul type d’encapsulation. L’encapsulation ne peut pas être modifiée après la création d’ExpressRoute Direct.
     >

1. Spécifiez des étiquettes de ressource, puis sélectionnez **Vérifier + créer** pour valider les paramètres de ressource ExpressRoute Direct.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/validate.png" alt-text="Capture d’écran montrant la page « Créer une ressource ExpressRoute » avec l’onglet « Vérifier + créer » sélectionné.":::

1. Sélectionnez **Create** (Créer). Un message vous informe que votre déploiement est en cours. L’état s’affiche sur cette page à mesure que les ressources sont créées. 

## <a name="generate-the-letter-of-authorization-loa"></a><a name="authorization"></a>Générer la lettre d’autorisation

La génération de la lettre d’autorisation n’est pas disponible à partir du portail pour l’instant. Utilisez **[Azure PowerShell](expressroute-howto-erdirect.md#authorization)** pour obtenir la lettre d’autorisation.

## <a name="change-admin-state-of-links"></a><a name="state"></a>Changer l’état Administrateur de liens

Ce processus doit être utilisé pour effectuer un test de la couche 1, en s’assurant que chaque connexion croisée est correctement reliée dans chaque routeur principal et secondaire.

1. Dans la page **Vue d’ensemble** des ressources ExpressRoute Direct, dans la section **Liens** , sélectionnez **link1**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/link.png" alt-text="Lien 1" lightbox="./media/how-to-expressroute-direct-portal/link-expand.png":::

1. Basculez le paramètre **État d’administration** sur **Activé** , puis sélectionnez **Enregistrer**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/state.png" alt-text="État d’administration":::

    >[!IMPORTANT]
    >La facturation commence quand l’état d’administration est activé sur l’un des liens.
    >

1. Répétez le même processus pour **link2**.

## <a name="create-a-circuit"></a><a name="circuit"></a>Créer un circuit

Par défaut, vous pouvez créer 10 circuits dans l’abonnement où se trouve la ressource ExpressRoute Direct. Cette valeur peut être augmentée par le support. Vous êtes responsable du suivi de la bande passante approvisionnée et utilisée. La bande passante provisionnée est la somme de la bande passante de tous les circuits de la ressource ExpressRoute Direct. La bande passante utilisée est l’utilisation physique des interfaces physiques sous-jacentes.

* D’autres bandes passantes de circuit peuvent être utilisées sur ExpressRoute Direct, uniquement pour prendre en charge les scénarios décrits ci-dessus. Ces règles sont les suivantes : 40 Gbits/s et 100 Gbits/s.

* La valeur de SkuTier peut être Local, Standard ou Premium.

* Le paramètre SkuFamily ne peut avoir que la valeur MeteredData. Un nombre illimité n’est pas pris en charge sur ExpressRoute Direct.

Les étapes suivantes vous permettent de créer un circuit ExpressRoute à partir du workflow ExpressRoute Direct. Si vous préférez, vous pouvez également créer un circuit à l’aide du workflow de circuit normal, même s’il n’y a aucun avantage à utiliser les étapes de workflow de circuit normales pour cette configuration. Consultez [Créer et modifier un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md).

1. Dans la section **Paramètres** d’ExpressRoute Direct, sélectionnez **Circuits** , puis **+Ajouter**. 

    :::image type="content" source="./media/how-to-expressroute-direct-portal/add.png" alt-text="Capture d’écran montrant les paramètres ExpressRoute avec les Circuits sélectionnés et Ajouter en surbrillance." lightbox="./media/how-to-expressroute-direct-portal/add-expand.png":::

1. Configurez les paramètres de la page **Configuration**.

   :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration2.png" alt-text="Page de configuration – ExpressRoute Direct":::

1. Spécifiez des étiquettes de ressource, puis sélectionnez **Vérifier + créer** afin de valider les valeurs avant de créer la ressource.

   :::image type="content" source="./media/how-to-expressroute-direct-portal/review.png" alt-text="Vérifier et créer ExpressRoute Direct":::

1. Sélectionnez **Create** (Créer). Un message vous informe que votre déploiement est en cours. L’état s’affiche sur cette page à mesure que les ressources sont créées. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur ExpressRoute, consultez la [Vue d’ensemble](expressroute-erdirect-about.md).
