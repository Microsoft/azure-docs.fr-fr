---
title: Tutoriel – Créer un jeu de réplicas dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment créer et utiliser des jeux de réplicas sur le portail Azure pour la résilience des services avec Azure AD Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: justinha
ms.openlocfilehash: 059f711269fd5f1eea9675f238a6003eaf1a0534
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618108"
---
# <a name="tutorial-create-and-use-replica-sets-for-resiliency-or-geolocation-in-azure-active-directory-domain-services-preview"></a>Tutoriel : Créer et utiliser des jeux de réplicas pour la résilience ou la géolocalisation dans Azure Active Directory Domain Services (préversion)

Pour améliorer la résilience d’un domaine managé Azure Active Directory Domain Services (Azure AD DS) ou pour effectuer un déploiement vers des zones géographiques supplémentaires proches de vos applications, vous pouvez utiliser des *jeux de réplicas*. Chaque espace de noms de domaine managé Azure AD DS, tel que *aaddscontoso.com*, contient un jeu de réplicas initial. La possibilité de créer des jeux de réplicas supplémentaires dans d’autres régions Azure fait bénéficier un domaine managé de la résilience géographique.

Vous pouvez ajouter un jeu de réplicas à n’importe quel réseau virtuel appairé d’une région Azure prenant en charge Azure AD DS.

Les jeux de réplicas sont une fonctionnalité en préversion publique d’Azure AD Domain Services. Ne perdez pas de vue que les fonctionnalités toujours en préversion présentent des différences sur le plan de la prise en charge. Pour plus d'informations sur les préversions, consultez [Contrat SLA des préversions Azure Active Directory](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Comprendre la configuration requise des réseaux virtuels
> * Créer un jeu de réplicas
> * Supprimer un jeu de réplicas

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services créé à l’aide d’un modèle de déploiement Azure Resource Manager et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][tutorial-create-instance].

    > [!IMPORTANT]
    > Les domaines managés créés à l’aide du modèle de déploiement classique ne peuvent pas utiliser de jeux de réplicas. Vous devez également utiliser une référence (SKU) minimale d’*Enterprise* pour votre domaine managé. Si nécessaire, [changez de référence SKU pour un domaine managé][howto-change-sku].

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans ce tutoriel, vous allez créer et gérer des jeux de réplicas à partir du portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="networking-considerations"></a>Mise en réseau - Éléments à prendre en compte

Les réseaux virtuels qui hébergent des jeux de réplicas doivent pouvoir communiquer entre eux. Les applications et les services qui dépendent d’Azure AD DS ont également besoin d’une connectivité réseau avec les réseaux virtuels hébergeant les jeux de réplicas. L’appairage de réseaux virtuels Azure doit être configuré entre tous les réseaux virtuels pour créer un réseau entièrement maillé. Ces appairages permettent une réplication intrasite efficace entre les jeux de réplicas.

Avant d’utiliser des jeux de réplicas dans Azure AD DS, prenez connaissance des exigences pour les réseaux virtuels Azure :

* Évitez les espaces d’adressage IP qui se chevauchent pour permettre l’appairage de réseaux virtuels et le routage.
* Créez des sous-réseaux avec suffisamment d’adresses IP pour prendre en charge votre scénario.
* Vérifiez qu’Azure AD DS possède son propre sous-réseau. Ne partagez pas ce sous-réseau de réseau virtuel avec les machines virtuelles et services d’application.
* Les réseaux virtuels appairés ne sont PAS transitifs.

> [!TIP]
> Quand vous créez un jeu de réplicas sur le portail Azure, les appairages entre réseaux virtuels sont créées automatiquement.
>
> Si nécessaire, vous pouvez créer un réseau virtuel et un sous-réseau au moment d’ajouter un jeu de réplicas sur le portail Azure. Vous pouvez aussi choisir des ressources de réseau virtuel existantes dans la région de destination pour un jeu de réplicas et laisser les appairages se créer automatiquement s’ils n’existent pas déjà.

## <a name="create-a-replica-set"></a>Créer un jeu de réplicas

Quand vous créez un domaine managé, tel que *aaddscontoso.com*, un jeu de réplicas initial est créé. Les jeux de réplicas supplémentaires partagent le même espace de noms et la même configuration. Les modifications apportées à Azure AD DS, notamment en ce qui concerne la configuration, l’identité de l’utilisateur et les informations d’identification, les groupes, les objets de stratégie de groupe, les objets ordinateur et autres modifications, sont appliquées à tous les jeux de réplicas du domaine managé via la réplication AD DS.

Dans ce tutoriel, vous allez créer un jeu de réplicas supplémentaire dans une région Azure différente de celle du jeu de réplicas Azure AD DS initial.

Pour créer un jeu de réplicas supplémentaire, effectuez les étapes suivantes :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**.
1. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Sur le côté gauche, sélectionnez **Jeux de réplicas (préversion)** . Chaque domaine managé comprend un jeu de réplicas initial dans la région sélectionnée, comme le montre la capture d’écran d’exemple ci-dessous :

    ![Capture d’écran d’exemple pour afficher et ajouter un jeu de réplicas sur le portail Azure](./media/tutorial-create-replica-set/replica-set-list.png)

    Pour créer un jeu de réplicas supplémentaire, sélectionnez **+ Ajouter**.

1. Dans la fenêtre *Ajouter un jeu de réplicas*, sélectionnez la région de destination, par exemple *USA Est*.

    Sélectionnez un réseau virtuel dans la région de destination, par exemple *vnet-eastus*, puis choisissez un sous-réseau, par exemple *aadds-subnet*. Si nécessaire, choisissez **Créer nouveau** pour ajouter un réseau virtuel dans la région de destination, puis **Gérer** pour créer un sous-réseau pour Azure AD DS.

    S’ils n’existent pas déjà, les appairages de réseaux virtuels Azure sont automatiquement créés entre le réseau virtuel de votre domaine managé et le réseau virtuel de destination.

    La capture d’écran d’exemple suivante illustre le processus de création d’un jeu de réplicas dans la région *USA Est* :

    ![Capture d’écran d’exemple pour créer un jeu de réplicas sur le portail Azure](./media/tutorial-create-replica-set/create-replica-set.png)

1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

Le processus de création du jeu de réplicas prend un certain temps, car les ressources sont créées dans la région de destination. Le domaine managé lui-même est ensuite répliqué via la réplication AD DS.

Le jeu de réplicas est présenté comme étant en cours de *provisionnement* pendant que le déploiement se produit, comme le montre la capture d’écran d’exemple suivante. Une fois terminé, le jeu de réplicas indique qu’il est en cours d’*exécution*.

![Capture d’écran d’exemple de l’état de déploiement d’un jeu de réplicas sur le portail Azure](./media/tutorial-create-replica-set/replica-set-provisioning.png)

## <a name="delete-a-replica-set"></a>Supprimer un jeu de réplicas

Un domaine managé est actuellement limité à quatre réplicas : le jeu de réplicas initial et trois jeux de réplicas supplémentaires. Si vous n’avez plus besoin d’un jeu de réplicas ou si vous voulez créer un jeu de réplicas dans une autre région, vous pouvez supprimer les jeux de réplicas superflus.

> [!IMPORTANT]
> Vous ne pouvez pas supprimer le dernier jeu de réplicas d’un domaine managé.

Pour supprimer un jeu de réplicas, effectuez les étapes suivantes :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**.
1. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Sur le côté gauche, sélectionnez **Jeux de réplicas (préversion)** . Dans la liste des jeux de réplicas, sélectionnez le menu contextuel **...** en regard du jeu de réplicas à supprimer.
1. Sélectionnez **Supprimer** dans le menu contextuel, puis confirmez que vous voulez supprimer le jeu de réplicas.

> [!NOTE]
> La suppression d’un jeu de réplicas peut être une opération longue.

Si vous n’avez plus besoin du réseau virtuel ou de l’appairage utilisé par le jeu de réplicas, vous pouvez aussi supprimer ces ressources. Vérifiez qu’aucune autre ressource d’application de l’autre région n’a besoin de connexions réseau avant de la supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Configurer le peering de réseaux virtuels
> * Créer un jeu de réplicas dans une autre région géographique
> * Supprimer un jeu de réplicas

Pour plus d’informations conceptuelles, découvrez comment les jeux de réplicas fonctionnent dans Azure AD DS.

> [!div class="nextstepaction"]
> [Concepts et fonctionnalités des jeux de réplicas][concepts-replica-sets]

<!-- INTERNAL LINKS -->
[replica-sets]: concepts-replica-sets.md
[tutorial-create-instance]: tutorial-create-instance-advanced.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[howto-change-sku]: change-sku.md
[concepts-replica-sets]: concepts-replica-sets.md
