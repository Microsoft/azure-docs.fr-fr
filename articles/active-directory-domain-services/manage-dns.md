---
title: Gérer DNS pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment installer les outils du serveur DNS pour gérer le DNS et créer des redirecteurs conditionnels pour un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: afa6920a36a5a7218571239b36815004d8f2d450
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619349"
---
# <a name="administer-dns-and-create-conditional-forwarders-in-an-azure-active-directory-domain-services-managed-domain"></a>Administrer DNS et créer des redirecteurs conditionnels dans un domaine managé Azure Active Directory Domain Services

Dans Azure Active Directory Domain Services (Azure AD DS), DNS est un composant clé. Azure AD DS inclut un serveur DNS qui fournit la résolution de noms pour le domaine managé. Ce serveur DNS comprend des enregistrements DNS intégrés et des mises à jour pour les composants clés qui permettent l’exécution du service.

Lorsque vous exécutez vos propres applications et services, il se peut que vous deviez créer des enregistrements DNS pour des machines qui ne sont pas jointes au domaine, ou configurer des adresses IP virtuelles pour des équilibreurs de charge ou des redirecteurs DNS externes. Les utilisateurs qui appartiennent au groupe *Administrateurs AAD DC* bénéficient de privilèges d’administration DNS sur le domaine managé Azure AD DS, et peuvent créer et modifier des enregistrements DNS personnalisés.

Dans un environnement hybride, les zones DNS et les enregistrements configurés dans d'autres espaces de noms DNS, comme un environnement AD DS local, ne sont pas synchronisés avec le domaine managé. Pour résoudre les ressources nommées dans d’autres espaces de noms DNS, créez et utilisez des redirecteurs conditionnels pointant vers des serveurs DNS existants dans votre environnement.

Cet article explique comment installer les Outils du serveur DNS, puis utiliser la console DNS pour gérer les enregistrements et créer des redirecteurs conditionnels dans Azure AD DS.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Connectivité à partir de votre réseau virtuel Azure AD DS vers l’emplacement où vos autres espaces de noms DNS sont hébergés.
    * Cette connectivité peut être assurée moyennant une connexion [Azure ExpressRoute][expressroute] ou [Passerelle VPN Azure][vpn-gateway].
* Machine virtuelle de gestion Windows Server jointe au domaine managé.
    * Si nécessaire, suivez les étapes du tutoriel pour [créer et joindre une machine virtuelle Windows Server à un domaine managé][create-join-windows-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.

## <a name="install-dns-server-tools"></a>Installer les Outils du serveur DNS

Pour créer et modifier des enregistrements DNS dans un domaine managé, vous devez installer les outils du serveur DNS. Ces outils peuvent être installés en tant que fonctionnalité de Windows Server. Pour plus d’informations sur l’installation des outils d’administration sur un client Windows, consultez [Installer les outils d’administration de serveur distant][install-rsat].

1. Connectez-vous à votre machine virtuelle de gestion. Pour connaître les différentes étapes vous permettant de vous connecter au portail Azure, consultez la page [Se connecter à une machine virtuelle Windows Server][connect-windows-server-vm].
1. Si **Gestionnaire de serveur** ne s’ouvre pas par défaut lorsque vous vous connectez à la machine virtuelle, sélectionnez le menu **Démarrer**, puis choisissez **Gestionnaire de serveur**.
1. Dans le volet *Tableau de bord* de la fenêtre **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et des fonctionnalités**.
1. Dans la page **Avant de commencer** de l’*Assistant Ajout de rôles et de fonctionnalités*, sélectionnez **Suivant**.
1. Pour le *Type d’installation*, laissez l’option **Installation basée sur un rôle ou une fonctionnalité** cochée et sélectionnez **Suivant**.
1. Dans la page **Sélection du serveur**, choisissez la machine virtuelle actuelle dans le pool de serveurs, par exemple *myvm.aaddscontoso.com*, puis sélectionnez **Suivant**.
1. Sur la page **Rôles de serveurs**, cliquez sur **Suivant**.
1. Dans la page **Fonctionnalités**, développez le nœud **Outils d’administration de serveur distant**, puis développez le nœud **Outils d’administration de rôles**. Sélectionnez la fonctionnalité **Outils du serveur DNS** dans la liste Outils d’administration de rôles.

    ![Choisir d’installer les Outils du serveur DNS à partir de la liste des outils d’administration de rôles disponibles](./media/manage-dns/install-dns-tools.png)

1. Dans la page **Confirmation**, sélectionnez **Installer**. L’installation des outils du serveur DNS peut prendre une ou deux minutes.
1. Une fois l’installation de la fonctionnalité terminée, sélectionnez **Fermer** pour quitter l’Assistant **Ajout de rôles et de fonctionnalités**.

## <a name="open-the-dns-management-console-to-administer-dns"></a>Ouvrir la console Gestion du service DNS pour administrer le serveur DNS

Une fois les outils du serveur DNS installés, vous pouvez administrer les enregistrements DNS sur le domaine managé.

> [!NOTE]
> Pour administrer DNS dans un domaine managé, vous devez être connecté à un compte d’utilisateur membre du groupe *d’administrateurs AAD DDC*.

1. Dans l’écran d’accueil, sélectionnez **Outils d’administration**. Une liste des outils de gestion disponibles s’affiche, dont le **DNS** installé dans la section précédente. Sélectionnez **DNS** pour lancer la console Gestion du service DNS.
1. Dans la boîte de dialogue **Connexion au serveur DNS**, sélectionnez **L’ordinateur suivant**, puis entrez le nom de domaine DNS du domaine managé, par exemple *aaddscontoso.com* :

    ![Se connecter au domaine managé dans la console DNS](./media/manage-dns/connect-dns-server.png)

1. La console DNS se connecte au domaine managé spécifié. Développez **Zones de recherche directe** ou **Zones de recherche inversée** pour créer vos entrées DNS requises ou modifier des enregistrements existants en fonction des besoins.

    ![Console DNS - Administration du domaine](./media/manage-dns/dns-manager.png)

> [!WARNING]
> Lorsque vous gérez des enregistrements à l’aide des Outils du serveur DNS, veillez à ne pas supprimer ou modifier les enregistrements DNS intégrés qu’Azure AD DS utilise. Les enregistrements DNS intégrés incluent les enregistrements DNS de domaine, les enregistrements de serveur de noms et d’autres enregistrements utilisés pour le lieu du contrôleur de domaine. Si vous modifiez ces enregistrements, les services de domaine sont interrompus sur le réseau virtuel.

## <a name="create-conditional-forwarders"></a>Créer des redirecteurs conditionnels

Une zone DNS Azure AD DS doit uniquement contenir la zone et les enregistrements du domaine managé proprement dit. Ne créez pas de zones supplémentaires dans le domaine managé pour résoudre les ressources nommées dans d’autres espaces de noms DNS. Privilégiez les redirecteurs conditionnels dans le domaine managé pour indiquer au serveur DNS à quel emplacement résoudre les adresses de ces ressources.

Dans un serveur DNS, un redirecteur conditionnel est une option de configuration qui vous permet de définir un domaine DNS, par exemple *contoso.com*, vers lequel transférer les requêtes. À la différence du serveur DNS local qui tente de résoudre les requêtes pour les enregistrements de ce domaine, les requêtes DNS sont transférées au DNS configuré pour ce domaine. Cette configuration garantit le renvoi des enregistrements DNS qui conviennent, car vous ne créez pas de zone DNS locale avec des enregistrements en double dans le domaine managé pour refléter ces ressources.

Pour créer un redirecteur conditionnel dans votre domaine managé, procédez comme suit :

1. Sélectionnez votre zone DNS, par exemple *aaddscontoso.com*.
1. Sélectionnez **Redirecteurs conditionnels**, puis cliquez avec le bouton droit et sélectionnez **Nouveau redirecteur conditionnel...**
1. Entrez votre autre **Domaine DNS**, par exemple *contoso.com*, puis entrez les adresses IP des serveurs DNS pour cet espace de noms, comme illustré dans l’exemple suivant :

    ![Ajouter et configurer un redirecteur conditionnel pour le serveur DNS](./media/manage-dns/create-conditional-forwarder.png)

1. Cochez la case pour **Stocker ce redirecteur conditionnel dans Active Directory, et le répliquer comme suit**, puis sélectionnez l’option *Tous les serveurs DNS de ce domaine*, comme illustré dans l’exemple suivant :

    ![Console DNS - sélectionner tous les serveurs DNS dans ce domaine](./media/manage-dns/store-in-domain.png)

    > [!IMPORTANT]
    > Si le redirecteur conditionnel est stocké dans la *forêt* et non dans le *domaine*, il échoue.

1. Pour créer le redirecteur conditionnel, sélectionnez **OK**.

La résolution de noms des ressources dans d’autres espaces de noms à partir de machines virtuelles connectées au domaine managé doit maintenant être correctement résolue. Les requêtes correspondant au domaine DNS configuré dans le redirecteur conditionnel sont transmises aux serveurs DNS qui conviennent.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion DNS, consultez l’article [Outils DNS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753579(v=ws.11))sur Technet.

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[create-join-windows-vm]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[connect-windows-server-vm]: join-windows-vm.md#connect-to-the-windows-server-vm

<!-- EXTERNAL LINKS -->
[install-rsat]: /windows-server/remote/remote-server-administration-tools#BKMK_Thresh