---
title: 'Tutoriel : Lier un réseau virtuel à un circuit ExpressRoute – Portail Azure'
description: Ce tutoriel explique comment à créer une connexion pour lier un réseau virtuel à un circuit Azure ExpressRoute à l’aide du portail Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 0ffc9c2ee17862497d3fd986da8e003f7a497056
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998768"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit-using-the-portal"></a>Tutoriel : Connecter un réseau virtuel à un circuit ExpressRoute à l’aide du portail

> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-linkvnet-classic.md)
> 

Ce tutoriel vous aide à créer une connexion pour lier un réseau virtuel à un circuit Azure ExpressRoute à l’aide du portail Azure. Les réseaux virtuels que vous connectez à votre circuit Azure ExpressRoute peuvent faire partie de même abonnement ou d’un autre abonnement.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - connecter un réseau virtuel à un circuit dans le même abonnement ;
> - connecter un réseau virtuel à un circuit dans un autre abonnement ;
> - supprimer le lien entre le réseau virtuel et le circuit ExpressRoute.

## <a name="prerequisites"></a>Prérequis

* Avant de commencer la configuration, examinez les [conditions préalables](expressroute-prerequisites.md), la [configuration requise pour le routage](expressroute-routing.md) et les [flux de travail](expressroute-workflows.md).

* Vous devez disposer d’un circuit ExpressRoute actif.
  * Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) et faites-le activer par votre fournisseur de service de connectivité.
  * Vérifiez que le peering privé Azure est configuré pour votre circuit. Consultez l’article [Créer et modifier le Peering pour un circuit ExpressRoute](expressroute-howto-routing-portal-resource-manager.md) pour obtenir des instructions sur le Peering et le routage.
  * Assurez-vous que l’homologation privée Azure est configurée et établit une homologation BGP entre votre réseau et Microsoft pour une connectivité de bout en bout.
  * Vérifiez qu’un réseau virtuel et une passerelle de réseau virtuel ont été créés et entièrement approvisionnés. Suivez les instructions pour [créer une passerelle de réseau virtuel pour ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Une passerelle de réseau virtuel pour ExpressRoute utilise le type de passerelle « ExpressRoute », pas de VPN.

* Vous pouvez lier jusqu’à 10 réseaux virtuels à un circuit ExpressRoute standard. Tous les réseaux virtuels doivent figurer dans la même région géopolitique lors de l’utilisation d’un circuit ExpressRoute standard.

* Un réseau virtuel unique peut être lié à quatre circuits ExpressRoute maximum. Pour créer un objet connexion pour chaque circuit ExpressRoute auquel vous vous connectez, procédez comme suit. Les circuits ExpressRoute peuvent être dans le même abonnement, dans des abonnements différents ou dans une combinaison des deux.

* Si vous activez le module complémentaire ExpressRoute Premium, vous pouvez lier des réseaux virtuels à l’extérieur de la région géopolitique du circuit ExpressRoute. Le module complémentaire Premium vous permet également de connecter plus de 10 réseaux virtuels à votre circuit ExpressRoute en fonction de la bande passante choisie. Pour plus d’informations sur le module complémentaire Premium, consultez le [FAQ](expressroute-faqs.md) .

* Vous pouvez [visualiser une vidéo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit) avant de commencer afin de mieux comprendre les étapes.

## <a name="connect-a-vnet-to-a-circuit---same-subscription"></a>Connecter un réseau virtuel à un circuit - même abonnement

> [!NOTE]
> Les informations de configuration BGP ne s’affichent pas si le fournisseur de la couche 3 a configuré vos homologations. Si votre circuit est dans l’état Approvisionné, vous pouvez créer des connexions.
>

### <a name="to-create-a-connection"></a>Pour créer une connexion

1. Assurez-vous que votre circuit ExpressRoute et le peering privé Azure ont été correctement configurés. Suivez les instructions fournies dans [Création d’un circuit ExpressRoute](expressroute-howto-circuit-arm.md) et [Créer et modifier le peering pour un circuit ExpressRoute](expressroute-howto-routing-arm.md). Votre circuit ExpressRoute doit être similaire à l’image suivante :

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/express-route-circuit.png" alt-text="Capture d’écran Circuit ExpressRoute":::

1. Vous pouvez maintenant commencer à approvisionner une connexion pour lier votre passerelle de réseau virtuel à votre circuit ExpressRoute. Sélectionnez **Connexion** > **Ajouter** pour ouvrir la page **Ajouter une connexion**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/add-connection.png" alt-text="Capture d’écran Ajouter une connexion":::

1. Entrez un nom pour la connexion, puis sélectionnez **Étape suivante : Paramètres >** .

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-basic.png" alt-text="Page des informations de base de création d’une connexion":::

1. Sélectionnez la passerelle appartenant au réseau virtuel que vous voulez lier au circuit, puis sélectionnez **Vérifier + créer**. Ensuite, une fois la validation terminée, sélectionnez **Créer**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-settings.png" alt-text="Page des paramètres de création d’une connexion":::

1. Une fois votre connexion correctement configurée, votre objet de connexion affiche les informations de la connexion.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-object.png" alt-text="Capture d’écran Objet de connexion":::

## <a name="connect-a-vnet-to-a-circuit---different-subscription"></a>Connecter un réseau virtuel à un circuit - autre abonnement

Vous pouvez partager un circuit ExpressRoute entre plusieurs abonnements. La figure suivante montre un schéma simple sur le fonctionnement du partage de circuits ExpressRoute entre plusieurs abonnements.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/cross-subscription.png" alt-text="Connectivité entre abonnements":::

Chacun des petits clouds dans le cloud principal est utilisé pour représenter les abonnements appartenant à différents services au sein d’une organisation. Les départements au sein de l’organisation utilisent leur propre abonnement pour déployer leurs services, mais ils peuvent partager un même circuit ExpressRoute pour se reconnecter à votre réseau local. Un seul service (dans cet exemple : le service informatique) peut détenir le circuit ExpressRoute. D’autres abonnements au sein de l’organisation peuvent utiliser le circuit ExpressRoute.

  > [!NOTE]
  > Les frais de connectivité et de bande passante pour le circuit dédié s’appliquent au propriétaire du circuit ExpressRoute. Tous les réseaux virtuels partagent la même bande passante.
  >

### <a name="administration---about-circuit-owners-and-circuit-users"></a>Administration : à propos des propriétaires du circuit et des utilisateurs du circuit

Le « propriétaire du circuit » est l’utilisateur avec pouvoir autorisé de la ressource de circuit ExpressRoute. Le propriétaire du circuit peut créer des autorisations utilisables par les « utilisateurs du circuit ». Les utilisateurs du circuit sont propriétaires de passerelles de réseau virtuel qui ne figurent pas dans le même abonnement que le circuit ExpressRoute. Les utilisateurs du circuit peuvent échanger des autorisations (une seule autorisation par réseau virtuel).

Le propriétaire du circuit a le pouvoir de modifier et de révoquer les autorisations à tout moment. La révocation d’une autorisation entraîne la suppression de toutes les connexions de l’abonnement dont l’accès a été révoqué.

### <a name="circuit-owner-operations"></a>Opérations du propriétaire du circuit

**Création d’une autorisation de connexion**

Le propriétaire du circuit crée une autorisation, ce qui entraîne la création d’une clé d’autorisation dont un utilisateur du circuit peut se servir pour connecter ses passerelles de réseau virtuel au circuit ExpressRoute. Une autorisation n’est valide que pour une seule connexion.

> [!NOTE]
> Chaque connexion nécessite une autorisation distincte.
>

1. Dans la page ExpressRoute, sélectionnez **Autorisations**, tapez un **nom** pour l’autorisation, puis sélectionnez **Enregistrer**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization.png" alt-text="Autorisations":::

2. Une fois la configuration enregistrée, copiez **l’ID de ressource** et la **clé d’autorisation**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization-key.png" alt-text="Clé d’autorisation":::

**Suppression d’une autorisation de connexion**

Vous pouvez supprimer une connexion en sélectionnant l’icône **Supprimer** pour la clé d’autorisation de votre connexion.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-authorization-key.png" alt-text="Supprimer une clé d’autorisation":::

Si vous voulez supprimer la connexion tout en conservant la clé d’autorisation, vous pouvez supprimer la connexion de la page connexion du circuit.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection-owning-circuit.png" alt-text="Supprimer le circuit propriétaire de la connexion":::

### <a name="circuit-user-operations"></a>Opérations de l’utilisateur du circuit

L’utilisateur du circuit a besoin de l’ID de ressource et d’une clé d’autorisation du propriétaire du circuit.

**Réclamation d’une autorisation de connexion**

1. Sélectionnez le bouton **+ Créer une ressource**. Recherchez **Connexion**, puis sélectionnez **Créer**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-new-resources.png" alt-text="Créer des ressources":::

1. Vérifiez que l’option *Type de connexion* est définie sur **ExpressRoute**. Sélectionnez le *Groupe de ressources* et l’*Emplacement*, puis sélectionnez **OK** dans la page des informations de base.

    > [!NOTE]
    > L’emplacement *doit* correspondre à l’emplacement de la passerelle de réseau virtuel pour lequel vous créez la connexion.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-basics.png" alt-text="Page De base":::

1. Dans la page **Paramètres**, sélectionnez *Passerelle de réseau virtuel*, puis cochez la case **Utiliser l’autorisation**. Entrez la *clé d’autorisation* et *l’URI du circuit appairé*, puis donnez un nom à la connexion. Sélectionnez **OK**. 
 
    > [!NOTE]
    > L'*URI du circuit homologue* est l’ID de ressource du circuit ExpressRoute (que vous pouvez trouver dans le volet des paramètres des propriétés du circuit ExpressRoute).

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-settings.png" alt-text="Page de paramètres":::

1. Passez en revue les informations contenues dans la page **Résumé**, puis sélectionnez **OK**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-summary.png" alt-text="Page de résumé":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer une connexion et annuler la liaison de votre réseau virtuel à un circuit ExpressRoute en sélectionnant l’icône **Supprimer** dans la page de votre connexion.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection.png" alt-text="Supprimer la connexion":::

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a montré comment connecter un réseau virtuel à un circuit dans le même abonnement et dans un autre abonnement. Pour plus d’informations sur la passerelle ExpressRoute, consultez : 

> [!div class="nextstepaction"]
> [À propos des passerelles de réseau virtuel ExpressRoute](expressroute-about-virtual-network-gateways.md)
