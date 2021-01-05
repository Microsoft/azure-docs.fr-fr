---
title: Tutoriel - Créer une approbation de forêt dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment créer une forêt unidirectionnelle sortante vers un domaine AD DS local dans le portail Azure pour Azure AD Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 0231689acef3345fb2b0f25170522d59552171ba
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618329"
---
# <a name="tutorial-create-an-outbound-forest-trust-to-an-on-premises-domain-in-azure-active-directory-domain-services"></a>Tutoriel : Créer une approbation de forêt sortante pour un domaine local dans Azure Active Directory Domain Services

Dans les environnements ne permettant pas la synchronisation des hachages de mot de passe, ou en présence d’utilisateurs se connectant exclusivement à l’aide de cartes à puce et ne connaissant pas leur mot de passe, vous pouvez utiliser une forêt de ressources dans Azure Active Directory Domain Services (Azure AD DS). Une forêt de ressources utilise une approbation unidirectionnelle sortante entre Azure AD DS et un ou plusieurs environnements AD DS locaux. Cette relation d’approbation permet aux utilisateurs, applications et ordinateurs de s’authentifier auprès d’un domaine local à partir du domaine managé Azure AD DS. Dans une forêt de ressources, les hachages de mots de passe ne sont jamais synchronisés.

![Diagramme d'approbation de forêt entre Azure AD DS et les instances AD DS locales](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer DNS dans un environnement AD DS local pour prendre en charge la connectivité Azure AD DS
> * Créer une approbation de forêt unidirectionnelle entrante dans un environnement AD DS local
> * Créer une approbation de forêt unidirectionnelle sortante dans Azure AD DS
> * Tester et valider la relation d’approbation à des fins d'authentification et d’accès aux ressources

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services créé à l'aide d'une forêt de ressources et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance-advanced].
    
    > [!IMPORTANT]
    > Veillez à créer un domaine managé à l’aide d’une forêt de *ressources*. L’option par défaut crée une forêt d’*utilisateur*. Seules les forêts de ressources peuvent créer des approbations dans des environnements locaux AD DS.
    >
    > Vous devez également utiliser une référence (SKU) minimale d’*Enterprise* pour votre domaine managé. Si nécessaire, [changez de référence SKU pour un domaine managé][howto-change-sku].

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans ce tutoriel, vous créez et vous configurez une approbation de forêt sortante à partir d'Azure AD DS avec le portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="networking-considerations"></a>Mise en réseau - Éléments à prendre en compte

Le réseau virtuel qui héberge la forêt de ressources Azure AD DS requiert une connexion réseau à votre instance Active Directory locale. Les applications et les services requièrent également une connexion au réseau virtuel hébergeant la forêt de ressources Azure AD DS. La connexion réseau à la forêt de ressources Azure AD DS doit être toujours active et stable, à défaut de quoi les utilisateurs risquent de ne pas pouvoir s’authentifier ou accéder aux ressources.

Avant de configurer une approbation de forêt dans Azure AD DS, assurez-vous que votre mise en réseau entre Azure et l’environnement local répond aux conditions requises suivantes :

* Utilisez des adresses IP privées. Ne vous fiez pas à DHCP avec attribution d’adresses IP dynamiques.
* Évitez les espaces d’adressage IP qui se chevauchent pour permettre au routage et à l’appairage de réseaux virtuels de bien communiquer entre Azure et les réseaux locaux.
* Un réseau virtuel Azure a besoin d’un sous-réseau de passerelle pour configurer une connexion [Azure VPN de site à site (S2S)][vpn-gateway] ou [ExpressRoute][expressroute].
* Créez des sous-réseaux avec suffisamment d’adresses IP pour prendre en charge votre scénario.
* Assurez-vous qu'Azure AD DS possède son propre sous-réseau. Ne partagez pas ce sous-réseau de réseau virtuel avec les machines virtuelles et services d’application.
* Les réseaux virtuels appairés ne sont PAS transitifs.
    * Des appairages de réseaux virtuels Azure doivent être créées entre tous les réseaux virtuels pour lesquels vous souhaitez utiliser l’approbation de forêt de ressources Azure AD DS vers l’environnement AD DS local.
* Fournissez une connectivité réseau continue à votre forêt Active Directory locale. N’utilisez pas de connexions à la demande.
* Assurez-vous de la présence d'une résolution de noms (DNS) continue entre votre nom de forêt de ressources Azure AD DS et votre nom de forêt Active Directory locale.

## <a name="configure-dns-in-the-on-premises-domain"></a>Configurer DNS dans le domaine local

Pour résoudre correctement le domaine managé à partir de l’environnement local, vous serez peut-être amené à ajouter des redirecteurs aux serveurs DNS existants. Si vous n’avez pas configuré l’environnement local de manière à ce qu’il communique avec le domaine managé, effectuez les étapes suivantes à partir d’une station de travail de gestion pour le domaine AD DS local :

1. Sélectionnez **Démarrer | Outils d'administration | DNS**.
1. Cliquez avec le bouton droit sur un serveur DNS comme *myAD01*, puis sélectionnez **Propriétés**.
1. Sélectionnez **Redirecteurs**, puis **Modifier** pour ajouter des redirecteurs supplémentaires.
1. Ajoutez les adresses IP du domaine managé, par exemple *10.0.2.4* et *10.0.2.5*.

## <a name="create-inbound-forest-trust-in-the-on-premises-domain"></a>Créer une approbation de forêt entrante dans le domaine local

Le domaine AD DS local nécessite une approbation de forêt entrante pour le domaine managé. Cette approbation doit être créée manuellement dans le domaine AD DS local ; sa création n'est pas possible à partir du portail Azure.

Pour configurer l’approbation entrante sur le domaine AD DS local, procédez comme suit à partir d’une station de travail de gestion pour le domaine AD DS local :

1. Sélectionnez **Démarrer | Outils d'administration | Domaines et approbations Active Directory**
1. Cliquez avec le bouton droit sur un domaine, par exemple *onprem.contoso.com*, puis sélectionnez **Propriétés**.
1. Sélectionnez l'onglet **Approbations**, puis **Nouvelle approbation**.
1. Entrez le nom du domaine Azure AD DS, comme *aaddscontoso.com*, puis sélectionnez **Suivant**.
1. Sélectionnez l’option permettant de créer une **approbation de forêt**, puis une approbation **unidirectionnelle : entrante**.
1. Choisissez de créer l’approbation pour **ce domaine uniquement**. À l’étape suivante, vous allez créer l’approbation dans le portail Azure pour le domaine managé.
1. Choisissez d’utiliser l'**authentification à l'échelle de la forêt**, puis entrez et confirmez un mot de passe d’approbation. Ce même mot de passe est également entré dans le portail Azure à la section suivante.
1. Parcourez les fenêtres suivantes avec les options par défaut, puis sélectionnez l’option pour **Non, ne pas confirmer l'approbation sortante**.
1. Sélectionnez **Terminer**.

## <a name="create-outbound-forest-trust-in-azure-ad-ds"></a>Créer une approbation de forêt sortante dans Azure AD DS

Une fois le domaine AD DS local configuré pour résoudre le domaine managé et une approbation de forêt entrante créée, créez l’approbation de forêt sortante. Cette approbation de forêt sortante établit la relation d’approbation entre le domaine AD DS local et le domaine managé.

Pour créer l’approbation sortante destinée au domaine managé dans le portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**, puis sélectionnez votre domaine managé, par exemple *aaddscontoso.com*
1. Dans le menu de gauche du domaine managé, sélectionnez **Approbations**, puis **+ Ajouter** une approbation.

   > [!NOTE]
   > Si vous ne voyez pas l’option de menu **Approbations**, vérifiez le **Type de forêt** sous *Propriétés*. Seules les forêts de *ressources* peuvent créer des approbations. Si le type de forêt est *Utilisateur*, vous ne pouvez pas créer de relations d’approbation. Il n’existe actuellement aucun moyen de modifier le type de forêt d’un domaine managé. Vous devez supprimer et recréer le domaine géré comme une forêt de ressources.

1. Entrez un nom d’affichage qui identifie votre approbation, puis le nom DNS de la forêt locale approuvée, par exemple *onprem.contoso.com*
1. Indiquez le même mot de passe d’approbation que celui utilisé à la section précédente lors de la configuration de l’approbation de forêt entrante pour le domaine AD DS local.
1. Fournissez au moins deux serveurs DNS pour le domaine AD DS local, par exemple *10.1.1.4* et *10.1.1.5*.
1. Lorsque vous êtes prêt, **enregistrez** l’approbation de forêt sortante.

    ![Créer une approbation de forêt sortante dans le portail Azure](./media/tutorial-create-forest-trust/portal-create-outbound-trust.png)

## <a name="validate-resource-authentication"></a>Valider l’authentification des ressources

Les scénarios courants suivants vous permettent de vérifier que l’approbation de forêt authentifie correctement les utilisateurs et l’accès aux ressources :

* [Authentification des utilisateurs locaux à partir de la forêt de ressources Azure AD DS](#on-premises-user-authentication-from-the-azure-ad-ds-resource-forest)
* [Accéder aux ressources de la forêt de ressources Azure AD DS à l’aide de l’utilisateur local](#access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user)
    * [Activer le partage de fichiers et d'imprimantes](#enable-file-and-printer-sharing)
    * [Créer un groupe de sécurité et ajouter des membres](#create-a-security-group-and-add-members)
    * [Créer un partage de fichiers pour l’accès inter-forêts](#create-a-file-share-for-cross-forest-access)
    * [Valider l'authentification inter-forêts vers une ressource](#validate-cross-forest-authentication-to-a-resource)

### <a name="on-premises-user-authentication-from-the-azure-ad-ds-resource-forest"></a>Authentification des utilisateurs locaux à partir de la forêt de ressources Azure AD DS

Vous devez disposer d’une machine virtuelle Windows Server jointe au domaine managé. Utilisez cette machine virtuelle pour vérifier que votre utilisateur local peut s’authentifier sur une machine virtuelle. Si nécessaire, [créez une machine virtuelle Windows et joignez-la au domaine managé][join-windows-vm].

1. Connectez-vous à la machine virtuelle Windows Server jointe à la forêt de ressources Azure AD DS en utilisant [Azure Bastion](../bastion/bastion-overview.md) et vos informations d’identification d’administrateur Azure AD DS.
1. Ouvrez une invite de commandes et utilisez la commande `whoami` pour afficher le nom unique de l’utilisateur actuellement authentifié :

    ```console
    whoami /fqdn
    ```

1. Utilisez la commande `runas` pour vous authentifier en tant qu’utilisateur à partir du domaine local. Dans la commande suivante, remplacez `userUpn@trusteddomain.com` par l’UPN d’un utilisateur du domaine local approuvé. La commande vous invite à entrer le mot de passe de l’utilisateur :

    ```console
    Runas /u:userUpn@trusteddomain.com cmd.exe
    ```

1. Si l’authentification aboutit, une nouvelle invite de commandes s’ouvre. Le titre de la nouvelle invite de commandes comprend `running as userUpn@trusteddomain.com`.
1. Utilisez `whoami /fqdn` dans la nouvelle invite de commandes pour afficher le nom unique de l’utilisateur authentifié à partir de l'instance Active Directory locale.

### <a name="access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user"></a>Accéder aux ressources de la forêt de ressources Azure AD DS à l’aide de l’utilisateur local

À l’aide de la machine virtuelle Windows Server jointe à la forêt de ressources Azure AD DS, vous pouvez tester le scénario permettant aux utilisateurs d'accéder aux ressources hébergées dans la forêt de ressources lorsqu’ils s’authentifient à partir d’ordinateurs du domaine local auprès d'utilisateurs du domaine local. Les exemples suivants vous montrent comment créer et tester différents scénarios courants.

#### <a name="enable-file-and-printer-sharing"></a>Activer le partage de fichiers et d’imprimantes

1. Connectez-vous à la machine virtuelle Windows Server jointe à la forêt de ressources Azure AD DS en utilisant [Azure Bastion](../bastion/bastion-overview.md) et vos informations d’identification d’administrateur Azure AD DS.

1. Ouvrez **Paramètres Windows**, puis recherchez et sélectionnez **Centre Réseau et partage**.
1. Sélectionnez l’option permettant de **modifier les paramètres de partage avancés**.
1. Sous **Profil du domaine**, sélectionnez **Activer le partage de fichiers et d'imprimantes**, puis **Enregistrer les modifications**.
1. Fermez **Centre Réseau et partage**.

#### <a name="create-a-security-group-and-add-members"></a>Créer un groupe de sécurité et ajouter des membres

1. Ouvrez **Utilisateurs et ordinateurs Active Directory**.
1. Cliquez avec le bouton droit sur le nom de domaine, sélectionnez **Nouveau**, puis **Unité d'organisation**.
1. Dans la zone de nom, entrez *LocalObjects*, puis sélectionnez **OK**.
1. Sélectionnez et cliquez avec le bouton droit sur **LocalObjects** dans le volet de navigation. Sélectionnez **Nouveau**, puis **Groupe**.
1. Entrez *FileServerAccess* dans la zone **Nom du groupe**. Pour **Étendue du groupe**, sélectionnez **Domaine local**, puis **OK**.
1. Dans le volet de contenu, double-cliquez sur **FileServerAccess**. Sélectionnez **Membres**, **Ajouter**, puis **Emplacements**.
1. Sélectionnez votre instance Active Directory locale dans l'affichage **Emplacement**, puis **OK**.
1. Entrez *Utilisateurs du domaine* dans la zone **Entrer les noms des objets à sélectionner**. Sélectionnez **Vérifier les noms**, fournissez les informations d’identification de l'instance Active Directory locale, puis sélectionnez **OK**.

    > [!NOTE]
    > La relation d'approbation étant unidirectionnelle, vous devez fournir les informations d’identification. Cela signifie que les utilisateurs du domaine managé Azure AD DS ne pourront pas accéder aux ressources ni rechercher des utilisateurs ou groupes dans le domaine (local) approuvé.

1. Le groupe **Utilisateurs du domaine** de votre instance Active Directory locale doit être membre du groupe **FileServerAccess**. Sélectionnez **OK** pour enregistrer le groupe et fermer la fenêtre.

#### <a name="create-a-file-share-for-cross-forest-access"></a>Créer un partage de fichiers pour l’accès inter-forêts

1. Sur la machine virtuelle Windows Server jointe à la forêt de ressources Azure AD DS, créez un dossier et attribuez-lui un nom, par exemple *CrossForestShare*.
1. Cliquez avec le bouton droit sur le dossier, puis sélectionnez **Propriétés**.
1. Sélectionnez l'onglet **Sécurité**, puis **Modifier**.
1. Dans la boîte de dialogue *Autorisations pour CrossForestShare*, sélectionnez **Ajouter**.
1. Entrez *FileServerAccess* dans **Entrer les noms des objets à sélectionner**, puis sélectionnez **OK**.
1. Sélectionnez *FileServerAccess* dans la liste **Noms des groupes ou des utilisateurs**. Dans la liste **Autorisations pour FileServerAccess**, sélectionnez *Autoriser* pour les autorisations **Modifier** et **Écrire**, puis **OK**.
1. Sélectionnez l’onglet **Partage**, puis **Partage avancé…**
1. Sélectionnez **Partager ce dossier**, puis entrez un nom de partage de fichiers facile à retenir dans **Nom du partage**, par exemple *CrossForestShare*.
1. Sélectionnez **Autorisations**. Dans la liste **Autorisations pour tous**, sélectionnez **Autoriser** pour l'autorisation **Modifier**.
1. Sélectionnez **OK** deux fois, puis **Fermer**.

#### <a name="validate-cross-forest-authentication-to-a-resource"></a>Valider l’authentification inter-forêts sur une ressource

1. Connectez-vous à un ordinateur Windows joint à votre instance Active Directory locale à l’aide d’un compte d’utilisateur de votre instance Active Directory locale.
1. À l’aide de l'**Explorateur Windows**, connectez-vous au partage que vous avez créé à l’aide du nom d’hôte complet et le partage, par exemple `\\fs1.aaddscontoso.com\CrossforestShare`.
1. Pour valider l’autorisation d’écriture, cliquez avec le bouton droit dans le dossier, sélectionnez **Nouveau**, puis **Document texte**. Utilisez le nom par défaut **Nouveau document texte**.

    Si les autorisations d’écriture sont correctement définies, un nouveau document texte est créé. Les étapes suivantes permettent d'ouvrir, de modifier et de supprimer le fichier, selon vos besoins.
1. Pour valider l’autorisation de lecture, ouvrez **Nouveau document texte**.
1. Pour valider l’autorisation de modification, ajoutez du texte au fichier et fermez le **Bloc-notes**. Lorsque vous êtes invité à enregistrer les modifications, sélectionnez **Enregistrer**.
1. Pour valider l’autorisation de suppression, cliquez avec le bouton droit sur **Nouveau document texte**, puis sélectionnez **Supprimer**. Sélectionnez **Oui** pour confirmer la suppression du fichier.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Configurer DNS dans un environnement AD DS local pour prendre en charge la connectivité Azure AD DS
> * Créer une approbation de forêt unidirectionnelle entrante dans un environnement AD DS local
> * Créer une approbation de forêt unidirectionnelle sortante dans Azure AD DS
> * Tester et valider la relation d’approbation à des fins d'authentification et d’accès aux ressources

Pour plus d’informations conceptuelles sur les types de forêts dans Azure AD DS, consultez [Qu’est-ce que des forêts de ressources ?][concepts-forest] et [Comment fonctionnent les approbations de forêts dans Azure AD DS ?][concepts-trust]

<!-- INTERNAL LINKS -->
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance-advanced]: tutorial-create-instance-advanced.md
[howto-change-sku]: change-sku.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[expressroute]: ../expressroute/expressroute-introduction.md
[join-windows-vm]: join-windows-vm.md