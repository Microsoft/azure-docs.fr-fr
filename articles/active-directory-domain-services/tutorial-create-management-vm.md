---
title: Créer une machine virtuelle de gestion pour Azure Active Directory Domain Services | Microsoft Docs
description: Dans ce tutoriel, vous allez découvrir comment créer et configurer une machine virtuelle Windows que vous utilisez pour administrer un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: d017fc3fc53c9f89978893670df10ecd731da904
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618176"
---
# <a name="tutorial-create-a-management-vm-to-configure-and-administer-an-azure-active-directory-domain-services-managed-domain"></a>Tutoriel : Créer une machine virtuelle de gestion pour configurer et administrer un domaine managé Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) fournit des services de domaines managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP, et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory. Vous administrez ce domaine managé avec les mêmes outils d’administration de serveur distant (RSAT, Remote Server Administration Tool) qu’avec un domaine Active Directory Domain Services local. Comme Azure AD DS est un service managé, vous ne pouvez pas effectuer certaines tâches d’administration, comme utiliser le protocole RDP (Remote Desktop Protocol) pour vous connecter aux contrôleurs de domaine.

Ce tutoriel explique comment configurer une machine virtuelle Windows Server dans Azure et comment installer les outils nécessaires à l’administration d’un domaine Azure AD DS managé.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Comprendre les tâches d’administration disponibles dans un domaine managé
> * Installer les outils d’administration Active Directory sur une machine virtuelle Windows Server
> * Utiliser le Centre d’administration Active Directory pour effectuer des tâches courantes

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, consultez le premier tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Une machine virtuelle Windows Server jointe au domaine managé.
    * Si nécessaire, consultez le tutoriel précédent pour [créer et joindre une machine virtuelle Windows Server à un domaine managé][create-join-windows-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.
* Un hôte Azure Bastion déployé dans votre réseau virtuel Azure AD DS.
    * Si nécessaire, [créez un hôte Azure Bastion][azure-bastion].

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans ce tutoriel, vous créez et vous configurez une machine virtuelle de gestion avec le portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="available-administrative-tasks-in-azure-ad-ds"></a>Tâches d’administration disponibles dans Azure AD DS

Azure AD DS fournit un domaine managé que vos utilisateurs, vos applications et vos services vont utiliser. Cette approche change certaines des tâches de gestion disponibles que vous pouvez effectuer, ainsi que les privilèges dont vous disposez dans le domaine managé. Ces tâches et autorisations peuvent être différentes de celles que vous rencontrez avec un environnement Active Directory Domain Services local normal. Vous ne pouvez pas vous connecter aux contrôleurs du domaine managé avec Bureau à distance.

### <a name="administrative-tasks-you-can-perform-on-a-managed-domain"></a>Tâches d’administration pouvant être effectuées sur un domaine géré

Les membres du groupe *AAD DC Administrators* bénéficient de privilèges leur permettant d’effectuer les tâches suivantes sur le domaine managé :

* Configurer l’objet de stratégie de groupe intégré pour les conteneurs *Ordinateurs AADDC* et *Utilisateurs AADDC* dans le domaine managé.
* administrer le DNS sur le domaine géré ;
* Créer et administrer des unités d’organisation personnalisées dans le domaine managé.
* obtenir un accès d’administrateur aux ordinateurs joints au domaine géré ;

### <a name="administrative-privileges-you-dont-have-on-a-managed-domain"></a>Privilèges d’administrateur dont vous ne disposez pas sur un domaine managé

Le domaine managé est verrouillé : vous ne disposez donc pas de privilèges permettant d’effectuer certaines tâches d’administration sur le domaine. Voici quelques exemples de tâches que vous ne pouvez pas effectuer :

* Étendre le schéma du domaine managé.
* Vous connecter aux contrôleurs de domaine du domaine managé avec Bureau à distance.
* Ajouter des contrôleurs de domaine au domaine managé.
* Vous ne disposez pas des privilèges *Administrateur de domaine* ou *Administrateur d’entreprise* pour le domaine managé.

## <a name="sign-in-to-the-windows-server-vm"></a>Se connecter à la machine virtuelle Windows Server

Dans le tutoriel précédent, une machine virtuelle Windows Server a été créée et jointe au domaine managé. Utilisez cette machine virtuelle pour installer les outils de gestion. Si nécessaire, [suivez les étapes du tutoriel pour créer et joindre une machine virtuelle Windows Server à un domaine managé][create-join-windows-vm].

> [!NOTE]
> Dans ce tutoriel, vous utilisez une machine virtuelle Windows Server dans Azure qui est jointe au domaine managé. Vous pouvez également utiliser un client Windows, comme Windows 10, qui est joint au domaine managé.
>
> Pour plus d’informations sur l’installation des outils d’administration sur un client Windows, consultez [Installer les outils d’administration de serveur distant](https://social.technet.microsoft.com/wiki/contents/articles/2202.remote-server-administration-tools-rsat-for-windows-client-and-windows-server-dsforum2wiki.aspx).

Pour commencer, connectez-vous à la machine virtuelle Windows Server comme suit :

1. Dans le portail Azure, sélectionnez **Groupes de ressources** sur le côté gauche. Choisissez le groupe de ressources où votre machine virtuelle a été créée, par exemple *myResourceGroup*, puis sélectionnez la machine virtuelle, par exemple *myVM*.
1. Dans le volet **Vue d’ensemble** de votre machine virtuelle, sélectionnez **Se connecter**, puis **Bastion**.

    ![Se connecter à une machine virtuelle Windows avec Bastion dans le portail Azure](./media/join-windows-vm/connect-to-vm.png)

1. Entrez les informations d’identification pour votre machine virtuelle, puis sélectionnez **Se connecter**.

   ![Se connecter via l’hôte Bastion dans le portail Azure](./media/join-windows-vm/connect-to-bastion.png)

Si nécessaire, autorisez votre navigateur web à ouvrir des fenêtres contextuelles pour afficher la connexion Bastion. Il faut quelques secondes pour établir la connexion à votre machine virtuelle.

## <a name="install-active-directory-administrative-tools"></a>Installer les outils d’administration Active Directory

Dans un domaine managé, vous utilisez les mêmes outils d’administration que dans les environnements AD DS locaux, comme le Centre d’administration Active Directory (ADAC) ou AD PowerShell. Ces outils peuvent être installés dans le cadre de la fonctionnalité Outils d’administration de serveur distant sur des ordinateurs Windows Server et des ordinateurs clients. Les membres du groupe *Administrateurs AAD DC* peuvent administrer les domaines managés à distance en utilisant ces outils d’administration AD à partir d’un ordinateur joint au domaine managé.

Pour installer les outils d’administration Active Directory sur une machine virtuelle jointe au domaine, effectuez les étapes suivantes :

1. Si **Gestionnaire de serveur** ne s’ouvre pas par défaut lorsque vous vous connectez à la machine virtuelle, sélectionnez le menu **Démarrer**, puis choisissez **Gestionnaire de serveur**.
1. Dans le volet *Tableau de bord* de la fenêtre **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et des fonctionnalités**.
1. Dans la page **Avant de commencer** de l’*Assistant Ajout de rôles et de fonctionnalités*, sélectionnez **Suivant**.
1. Pour le *Type d’installation*, laissez l’option **Installation basée sur un rôle ou une fonctionnalité** cochée et sélectionnez **Suivant**.
1. Dans la page **Sélection du serveur**, choisissez la machine virtuelle actuelle dans le pool de serveurs, par exemple *myvm.aaddscontoso.com*, puis sélectionnez **Suivant**.
1. Sur la page **Rôles de serveurs**, cliquez sur **Suivant**.
1. Dans la page **Fonctionnalités**, développez le nœud **Outils d’administration de serveur distant**, puis développez le nœud **Outils d’administration de rôles**.

    Choisissez la fonctionnalité **Outils AD DS et AD LDS** dans la liste des outils d’administration de rôles, puis sélectionnez **Suivant**.

    ![Installer les « Outils AD DS et AD LDS » à partir de la page Fonctionnalités](./media/tutorial-create-management-vm/install-features.png)

1. Dans la page **Confirmation**, sélectionnez **Installer**. L’installation des outils d’administration peut prendre une ou deux minutes.
1. Une fois l’installation de la fonctionnalité terminée, sélectionnez **Fermer** pour quitter l’Assistant **Ajout de rôles et de fonctionnalités**.

## <a name="use-active-directory-administrative-tools"></a>Utiliser les outils d’administration Active Directory

Une fois les outils d’administration installés, voyons comment les utiliser pour administrer le domaine managé. Vérifiez que vous êtes connecté à la machine virtuelle avec un compte d’utilisateur membre du groupe *Administrateurs AAD DC*.

1. Dans le menu **Démarrer**, sélectionnez **Outils d’administration Windows**. Les outils d’administration Active Directory installés à l’étape précédente figurent dans la liste.

    ![Liste des outils d’administration installés sur le serveur](./media/tutorial-create-management-vm/list-admin-tools.png)

1. Sélectionnez **Centre d’administration Active Directory**.
1. Pour explorer le domaine managé, choisissez le nom de domaine dans le volet gauche, par exemple *aaddscontoso*. Deux conteneurs nommés *Ordinateurs AADDC* et *Utilisateurs AADDC* se trouvent en haut de la liste.

    ![Lister les conteneurs disponibles faisant partie du domaine managé](./media/tutorial-create-management-vm/active-directory-administrative-center.png)

1. Pour voir les utilisateurs et les groupes appartenant au domaine managé, sélectionnez le conteneur **Utilisateurs AADDC**. Les comptes d’utilisateur et les groupes de votre locataire Azure AD sont listés dans ce conteneur.

    Dans l’exemple de sortie suivant, un compte d’utilisateur nommé *Contoso Admin* et un groupe pour *Administrateurs AAD DC* figurent dans ce conteneur.

    ![Afficher la liste des utilisateurs du domaine Azure AD DS dans le Centre d’administration Active Directory](./media/tutorial-create-management-vm/list-azure-ad-users.png)

1. Pour voir les ordinateurs qui sont joints au domaine managé, sélectionnez le conteneur **Ordinateurs AADDC**. Une entrée pour la machine virtuelle actuelle, par exemple *myVM* figure dans la liste. Les comptes d’ordinateurs de tous les appareils joints au domaine managé sont stockés dans le conteneur *Ordinateurs AADDC*.

Les actions courantes du Centre d’administration Active Directory, comme la réinitialisation du mot de passe d’un compte d’utilisateur ou la gestion de l’appartenance à un groupe, sont disponibles. Ces actions fonctionnent seulement pour les utilisateurs et les groupes créés directement dans le domaine managé. Les informations d’identité ne sont synchronisées que *depuis* Azure AD vers Azure AD DS. Il n’y a pas de mise à jour depuis Azure AD DS vers Azure AD. Vous ne pouvez pas changer les mots de passe ou l’appartenance à un groupe managé pour les utilisateurs synchronisés à partir d’Azure AD et obtenir la synchronisation de ces modifications.

Vous pouvez également utiliser le *Module Active Directory pour Windows PowerShell*, qui est installé dans le cadre des outils d’administration, pour gérer les actions courantes dans votre domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Comprendre les tâches d’administration disponibles dans un domaine managé
> * Installer les outils d’administration Active Directory sur une machine virtuelle Windows Server
> * Utiliser le Centre d’administration Active Directory pour effectuer des tâches courantes

Pour interagir de façon sécurisée avec votre domaine managé à partir d’autres applications, activez le protocole LDAPS.

> [!div class="nextstepaction"]
> [Configurer le protocole LDAP sécurisé pour votre domaine managé](tutorial-configure-ldaps.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[azure-bastion]: ../bastion/tutorial-create-host-portal.md