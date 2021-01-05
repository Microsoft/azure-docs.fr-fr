---
title: 'Tutoriel : Configurer un réseau virtuel pour Azure AD Domain Services | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment utiliser le portail Azure afin de créer et configurer un sous-réseau de réseau virtuel Azure ou un appairage de réseaux pour un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: ddb8f360304bdb41ae359f293af4d10b0afc6558
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618414"
---
# <a name="tutorial-configure-virtual-networking-for-an-azure-active-directory-domain-services-managed-domain"></a>Tutoriel : Configurer un réseau virtuel pour un domaine managé Azure Active Directory Domain Services

Pour fournir la connectivité nécessaire aux utilisateurs et aux applications, un domaine managé Azure Active Directory Domain Services (Azure AD DS) est déployé dans un sous-réseau de réseau virtuel Azure. Ce sous-réseau de réseau virtuel doit être utilisé uniquement pour les ressources du domaine managé qui sont fournies par la plateforme Azure.

Lorsque vous créez vos propres machines virtuelles et applications, celles-ci ne doivent pas être déployées dans le même sous-réseau de réseau virtuel. Créez et déployez vos applications dans un sous-réseau de réseau virtuel séparé, ou dans un autre réseau virtuel qui est appairé avec le réseau virtuel Azure AD DS.

Ce tutoriel vous montre comment créer et configurer un sous-réseau de réseau virtuel dédié ou comment appairer un réseau différent avec le réseau virtuel du domaine managé Azure AD DS.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Utiliser correctement les options de connectivité de réseau virtuel pour les ressources jointes au domaine Azure AD DS
> * Créer une plage d’adresses IP et un sous-réseau supplémentaire dans le réseau virtuel Azure AD DS
> * Configurer l’appairage du réseau virtuel avec un réseau séparé d’Azure AD DS

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Vous devez disposer des privilèges d’*Administrateur global* dans votre locataire Azure AD pour configurer Azure AD DS.
* Vous avez besoin de privilèges de *Contributeur* dans votre abonnement Azure pour créer les ressources Azure AD DS nécessaires.
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, le premier tutoriel [crée et configure un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans ce tutoriel, vous créez et configurez le domaine managé avec le portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="application-workload-connectivity-options"></a>Options de connectivité pour les charges de travail des applications

Dans le tutoriel précédent, vous avez créé un domaine managé en utilisant certaines options de configuration par défaut pour le réseau virtuel. Ces options par défaut ont créé un réseau virtuel et un sous-réseau de réseau virtuel Azure. Les contrôleurs de domaine Azure AD DS qui fournissent les services du domaine managé sont connectés à ce sous-réseau de réseau virtuel.

Quand vous créez et exécutez des machines virtuelles qui doivent accéder au domaine managé, vous devez leur fournir la connectivité réseau nécessaire. Cette connectivité réseau peut être fournie de l’une des manières suivantes :

* Créez un sous-réseau de réseau virtuel supplémentaire sur le réseau virtuel du domaine managé. Ce sous-réseau supplémentaire est l’endroit où vous créez et connectez vos machines virtuelles.
    * Comme ces machines virtuelles font partie du même réseau virtuel, elles peuvent automatiquement résoudre les noms et communiquer avec les contrôleurs de domaine Azure AD DS.
* Configurez l’appairage de réseaux virtuels Azure entre le réseau virtuel du domaine managé et un ou plusieurs réseaux virtuels distincts. Vous créez et connectez vos machines virtuelles dans ces réseaux virtuels séparés.
    * Quand vous configurez l’appairage de réseaux virtuels, vous devez également configurer les paramètres DNS pour utiliser la résolution de noms par les contrôleurs de domaine Azure AD DS.

En règle générale, vous utilisez une seule de ces options de connectivité réseau. Le choix est souvent dicté par la façon dont vous souhaitez gérer vos différentes ressources Azure.

* Si vous souhaitez gérer les machines virtuelles connectées et Azure AD DS dans un même groupe de ressources, créez un sous-réseau de réseau virtuel supplémentaire pour les machines virtuelles.
* Si vous préférez les gérer séparément, utilisez l’appairage de réseaux virtuels.
    * Vous pouvez également choisir d’utiliser l’appairage de réseaux virtuels pour fournir la connectivité réseau aux machines virtuelles dans votre environnement Azure qui sont déjà connectées à un réseau virtuel existant.

Dans ce tutoriel, vous devez configurer une seule de ces options de connectivité de réseau virtuel.

Pour plus d’informations sur la planification et la configuration du réseau virtuel, consultez [Considérations relatives au réseau pour Azure Active Directory Domain Services][network-considerations].

## <a name="create-a-virtual-network-subnet"></a>Créer un sous-réseau de réseau virtuel

Par défaut, le réseau virtuel Azure créé avec le domaine managé contient un seul sous-réseau de réseau virtuel. Ce sous-réseau doit être utilisé uniquement par la plateforme Azure qui fournit les services du domaine managé. Pour créer et utiliser vos propres machines virtuelles dans ce réseau virtuel Azure, créez un sous-réseau supplémentaire.

Pour créer un sous-réseau de réseau virtuel dédié aux machines virtuelles et aux charges de travail des applications, effectuez les étapes suivantes :

1. Dans le portail Azure, sélectionnez le groupe de ressources de votre domaine managé, par exemple *myResourceGroup*. Dans la liste des ressources, choisissez le réseau virtuel par défaut, comme *aadds-vnet* ici.
1. Dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Espace d’adressage**. Le réseau virtuel est créé avec l’espace d’adressage unique *10.0.2.0/24*, qui est utilisé par le sous-réseau par défaut.

    Ajoutez une plage d’adresses IP supplémentaire au réseau virtuel. La taille de cette plage d’adresses et la plage d’adresses IP réelle à utiliser dépendent des autres ressources réseau qui sont déjà déployées. La plage d’adresses IP ne doit pas chevaucher les plages d’adresses existantes dans votre environnement Azure ou local. Assurez-vous d’utiliser une plage d’adresses IP suffisamment grande pour le nombre de machines virtuelles que vous prévoyez de déployer dans le sous-réseau.

    Dans l’exemple suivant, la plage d’adresses IP supplémentaire *10.0.3.0/24* est ajoutée. Quand vous êtes prêt, sélectionnez **Enregistrer**.

    ![Ajouter une plage d’adresses IP de réseau virtuel supplémentaire dans le portail Azure](./media/tutorial-configure-networking/add-vnet-address-range.png)

1. Ensuite, dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Sous-réseaux**, puis choisissez **+ Sous-réseau** pour ajouter un sous-réseau.
1. Entrez un nom pour le sous-réseau, par exemple, *workloads* (charges de travail). Si nécessaire, mettez à jour la **plage d’adresses** si vous souhaitez utiliser un sous-ensemble de la plage d’adresses IP qui a été configurée pour le réseau virtuel lors des étapes précédentes. Pour le moment, conservez les valeurs par défaut des options telles que le groupe de sécurité réseau, la table de routage et les points de terminaison de service.

    Dans l’exemple suivant, le sous-réseau *workloads* qui est créé utilise la plage d’adresses IP *10.0.3.0/24* :

    ![Ajouter un sous-réseau de réseau virtuel supplémentaire dans le portail Azure](./media/tutorial-configure-networking/add-vnet-subnet.png)

1. Lorsque vous êtes prêt, sélectionnez **OK**. Il faut quelques instants pour créer le sous-réseau de réseau virtuel.

Quand vous créez une machine virtuelle qui doit utiliser le domaine managé, veillez à sélectionner ce sous-réseau de réseau virtuel. Ne créez pas vos machines virtuelles dans le sous-réseau *aadds-subnet* par défaut. Si vous sélectionnez un autre réseau virtuel, vous ne fournissez pas la connectivité réseau et la résolution DNS nécessaires pour l’accès au domaine managé, sauf si vous configurez le peering de réseaux virtuels.

## <a name="configure-virtual-network-peering"></a>Configurer le peering de réseaux virtuels

Vous souhaitez peut-être utiliser un réseau virtuel Azure existant pour les machines virtuelles, ou conserver votre réseau virtuel séparé du domaine managé. Pour utiliser le domaine managé, les machines virtuelles situées dans d’autres réseaux virtuels doivent pouvoir communiquer avec les contrôleurs de domaine Azure AD DS. Il est possible de fournir cette connectivité par l’appairage de réseaux virtuels Azure.

L’appairage de réseaux virtuels Azure vous permet de connecter deux réseaux virtuels sans avoir besoin d’un périphérique de réseau privé virtuel (VPN). Vous pouvez ainsi connecter rapidement des réseaux virtuels et définir les flux de trafic dans votre environnement Azure.

Pour plus d’informations sur l’appairage, consultez cette [vue d’ensemble de l’appairage de réseaux virtuels Azure][peering-overview].

Pour appairer un réseau virtuel avec le réseau virtuel du domaine managé, effectuez les étapes suivantes :

1. Choisissez le réseau virtuel par défaut créé pour votre domaine managé, nommé *aadds-vnet*.
1. Dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Appairages**.
1. Pour créer un appairage, sélectionnez **+ Ajouter**. Dans l’exemple suivant, le réseau virtuel par défaut *aadds-vnet* est appairé avec un autre réseau virtuel nommé *myVnet*. Configurez les paramètres suivants avec vos propres valeurs :

    * **Nom du peering de aadds-vnet avec un réseau virtuel distant** : identificateur descriptif des deux réseaux, par exemple *aadds-vnet-to-myvnet*
    * **Type de déploiement de réseau virtuel** : *Resource Manager*
    * **Abonnement**: abonnement du réseau virtuel avec lequel vous souhaitez effectuer l’appairage, par exemple *Azure*
    * **Réseau virtuel** : réseau virtuel à appairer, par exemple *myVnet*
    * **Nom du peering de myVnet à aadds-vnet** : identificateur descriptif des deux réseaux, par exemple *myvnet-to-aadds-vnet*

    ![Configurer l’appairage de réseaux virtuels dans le portail Azure](./media/tutorial-configure-networking/create-peering.png)

    Conservez les autres paramètres par défaut de l’accès au réseau virtuel ou du trafic transféré, sauf si votre environnement a des exigences spécifiques, puis sélectionnez **OK**.

1. Il faut quelques instants pour créer l’appairage entre le réseau virtuel Azure AD DS et le réseau virtuel que vous avez sélectionné. Une fois l’appairage terminé, la colonne **État d’appairage** indique *Connecté*, comme dans l’exemple ci-dessous :

    ![Réseaux connectés après leur appairage dans le portail Azure](./media/tutorial-configure-networking/connected-peering.png)

Pour que les machines virtuelles du réseau virtuel appairé puissent utiliser le domaine managé, vous devez configurer les serveurs DNS avec la résolution de noms correcte.

### <a name="configure-dns-servers-in-the-peered-virtual-network"></a>Configurer les serveurs DNS dans le réseau virtuel appairé

Pour permettre aux machines virtuelles et aux applications déployées sur le réseau virtuel appairé de communiquer correctement avec le domaine managé, vous devez mettre à jour les paramètres DNS. Les adresses IP des contrôleurs de domaine Azure AD DS doivent être configurées en tant que serveurs DNS sur le réseau virtuel appairé. Il existe deux façons de configurer les contrôleurs de domaine comme serveurs DNS dans le réseau virtuel appairé :

* Configurez les serveurs DNS du réseau virtuel Azure pour qu’ils utilisent les contrôleurs de domaine Azure AD DS.
* Configurez le serveur DNS existant actuellement utilisé sur le réseau virtuel appairé afin qu’il transfère les requêtes au domaine managé à l’aide de la redirection DNS conditionnelle. Ces étapes varient selon le serveur DNS existant qui est utilisé.

Dans ce tutoriel, vous configurez les serveurs DNS du réseau virtuel Azure afin qu’ils transfèrent toutes les requêtes aux contrôleurs de domaine Azure AD DS.

1. Dans le portail Azure, sélectionnez le groupe de ressources du réseau virtuel appairé, *myResourceGroup*, par exemple. Dans la liste des ressources, choisissez le réseau virtuel appairé, *myVnet* ici.
1. Dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Serveurs DNS**.
1. Par défaut, un réseau virtuel utilise les serveurs DNS intégrés fournis par Azure. Choisissez d’utiliser des serveurs DNS **Personnalisés**. Entrez les adresses IP des contrôleurs de domaine Azure AD DS, qui sont habituellement *10.0.2.4* et *10.0.2.5*. Vérifiez ces adresses IP dans la fenêtre **Vue d’ensemble** de votre domaine managé dans le portail.

    ![Configurer les serveurs DNS du réseau virtuel pour qu’ils utilisent les contrôleurs de domaine Azure AD DS](./media/tutorial-configure-networking/custom-dns.png)

1. Quand vous êtes prêt, sélectionnez **Enregistrer**. Il faut quelques instants pour mettre à jour les serveurs DNS pour le réseau virtuel.
1. Pour appliquer les nouveaux paramètres DNS aux machines virtuelles, redémarrez les machines virtuelles qui sont connectées au réseau virtuel appairé.

Quand vous créez une machine virtuelle qui doit utiliser le domaine managé, veillez à sélectionner ce réseau virtuel appairé. Si vous sélectionnez un autre réseau virtuel, vous ne fournissez pas la connectivité réseau et la résolution DNS nécessaires pour l’accès au domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser correctement les options de connectivité de réseau virtuel pour les ressources jointes au domaine Azure AD DS
> * Créer une plage d’adresses IP et un sous-réseau supplémentaire dans le réseau virtuel Azure AD DS
> * Configurer l’appairage du réseau virtuel avec un réseau séparé d’Azure AD DS

Pour voir ce domaine managé en action, créez et joignez une machine virtuelle au domaine.

> [!div class="nextstepaction"]
> [Joindre une machine virtuelle Windows Server à votre domaine managé](join-windows-vm.md)

<!-- INTERNAL LINKS --> 
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[peering-overview]: ../virtual-network/virtual-network-peering-overview.md
[network-considerations]: network-considerations.md
