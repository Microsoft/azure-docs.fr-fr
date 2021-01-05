---
title: Joindre une machine virtuelle Windows Server à un domaine managé par Azure AD Domain Services | Microsoft Docs
description: Dans ce tutoriel, vous découvrez comment joindre une machine virtuelle Windows Server à un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 869c827485d9b7a6baf68d2619af98d4c2ee82b9
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619570"
---
# <a name="tutorial-join-a-windows-server-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain"></a>Tutoriel : Joindre une machine virtuelle Windows Server à un domaine managé par Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) fournit des services de domaine managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP, et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory. Avec un domaine managé Azure AD DS, vous pouvez fournir des fonctionnalités de jonction de domaine et de gestion à des machines virtuelles dans Azure. Ce tutoriel montre comment créer une machine virtuelle Windows Server, puis la joindre à un domaine managé.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une machine virtuelle Windows Server
> * Connecter la machine virtuelle Windows Server à un réseau virtuel Azure
> * Joindre la machine virtuelle au domaine managé

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin des ressources suivantes :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Un compte d’utilisateur membre du domaine managé.
    * Vérifiez que la synchronisation du hachage de mot de passe ou que la réinitialisation du mot de passe en libre-service d’Azure AD Connect a été effectuée pour que le compte puisse se connecter au domaine managé.
* Un hôte Azure Bastion déployé dans votre réseau virtuel Azure AD DS.
    * Si nécessaire, [créez un hôte Azure Bastion][azure-bastion].

Si vous disposez déjà d’une machine virtuelle que vous voulez joindre à un domaine, passez à la section expliquant comment [joindre la machine virtuelle au domaine managé](#join-the-vm-to-the-managed-domain).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans ce tutoriel, vous allez créer une machine virtuelle Windows Server à joindre à votre domaine managé avec le portail Azure. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-windows-server-virtual-machine"></a>Créer une machine virtuelle Windows Server

Pour voir comment joindre un ordinateur à un domaine managé, créons une machine virtuelle Windows Server. Cette machine virtuelle est connectée à un réseau virtuel Azure qui fournit la connectivité au domaine managé. Le processus de jonction à un domaine managé est identique à celui d’un domaine Active Directory Domain Services local normal.

Si vous disposez déjà d’une machine virtuelle que vous voulez joindre à un domaine, passez à la section expliquant comment [joindre la machine virtuelle au domaine managé](#join-the-vm-to-the-managed-domain).

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

1. Dans **Bien démarrer**, choisissez **Windows Server 2016 Datacenter**.

    ![Choisir de créer une machine virtuelle Windows Server 2016 Datacenter dans le portail Azure](./media/join-windows-vm/select-vm-image.png)

1. Dans la fenêtre **De base**, configurez les paramètres principaux de la machine virtuelle. Conservez les paramètres par défaut pour *Options de disponibilité*, *Image* et *Taille*.

    | Paramètre            | Valeur suggérée   |
    |----------------------|-------------------|
    | Resource group       | Sélectionnez ou créez un groupe de ressources, par exemple *myResourceGroup* |
    | Nom de la machine virtuelle | Entrez un nom pour la machine virtuelle, par exemple *myVM* |
    | Région               | Choisissez la région où créer votre machine virtuelle, par exemple *USA Est*. |
    | Nom d’utilisateur             | Entrez un nom d’utilisateur pour le compte d’administrateur local à créer sur la machine virtuelle, par exemple *azureuser* |
    | Mot de passe             | Entrez puis confirmez un mot de passe sécurisé pour l’administrateur local à créer sur la machine virtuelle. Ne spécifiez pas les informations d’identification d’un compte d’utilisateur du domaine. |

1. Par défaut, les machines virtuelles créées dans Azure sont accessibles à partir d’Internet à l’aide de RDP. Quand RDP est activé, des attaques de connexion automatique sont susceptibles de se produire, ce qui peut désactiver les comptes portant des noms courants, comme *admin* ou *administrator*, en raison d’un trop grand nombre de tentatives de connexion.

    RDP doit être activé seulement quand c’est nécessaire et être limité à un ensemble de plages d’adresses IP autorisées. Cette configuration permet d’améliorer la sécurité de la machine virtuelle et de réduire le champ des attaques potentielles. Vous pouvez aussi créer et utiliser un hôte Azure Bastion qui autorise l’accès uniquement via le portail Azure sur TLS. Dans l’étape suivante de ce tutoriel, vous allez utiliser un hôte Azure Bastion pour vous connecter de façon sécurisée à la machine virtuelle.

    Sous **Ports d’entrée publics**, sélectionnez *Aucun*.

1. Quand vous avez terminé, sélectionnez **Suivant : Disques**.
1. Dans le menu déroulant pour le **Type de disque de système d’exploitation**, choisissez *SSD Standard*, puis sélectionnez **Suivant : Mise en réseau**.
1. Votre machine virtuelle doit se connecter à un sous-réseau du réseau virtuel Azure qui peut communiquer avec le sous-réseau où votre domaine managé est déployé. Nous recommandons de déployer un domaine managé sur son propre sous-réseau dédié. Ne déployez pas votre machine virtuelle sur le même sous-réseau que votre domaine managé.

    Il existe deux manières principales de déployer votre machine virtuelle et de se connecter à un sous-réseau de réseau virtuel approprié :
    
    * Créez ou sélectionnez un sous-réseau existant sur le même réseau virtuel que celui où votre domaine managé est déployé.
    * Sélectionnez un sous-réseau dans un réseau virtuel Azure qui lui est connecté avec le [peering de réseau virtuel Azure][vnet-peering].
    
    Si vous sélectionnez un sous-réseau de réseau virtuel qui n’est pas connecté au sous-réseau de votre domaine managé, vous ne pouvez pas joindre la machine virtuelle au domaine managé. Pour ce tutoriel, créons un sous-réseau dans le réseau virtuel Azure.

    Dans le volet **Mise en réseau**, sélectionnez le réseau virtuel où le domaine managé est déployé, par exemple *aaads-vnet*
1. Dans cet exemple, le sous-réseau *aaads-subnet* existant est celui auquel le domaine managé domaine est connecté. Ne connectez pas votre machine virtuelle à ce sous-réseau. Pour créer un sous-réseau pour la machine virtuelle, sélectionnez **Gérer la configuration du sous-réseau**.

    ![Choisir de gérer la configuration du sous-réseau dans le portail Azure](./media/join-windows-vm/manage-subnet.png)

1. Dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Espace d’adressage**. Le réseau virtuel est créé avec l’espace d’adressage unique *10.0.2.0/24*, qui est utilisé par le sous-réseau par défaut. D’autres sous-réseaux, comme pour *workloads* ou Azure Bastion, peuvent aussi déjà exister.

    Ajoutez une plage d’adresses IP supplémentaire au réseau virtuel. La taille de cette plage d’adresses et la plage d’adresses IP réelle à utiliser dépendent des autres ressources réseau qui sont déjà déployées. La plage d’adresses IP ne doit pas chevaucher les plages d’adresses existantes dans votre environnement Azure ou local. Assurez-vous d’utiliser une plage d’adresses IP suffisamment grande pour le nombre de machines virtuelles que vous prévoyez de déployer dans le sous-réseau.

    Dans l’exemple suivant, la plage d’adresses IP supplémentaire *10.0.5.0/24* est ajoutée. Quand vous êtes prêt, sélectionnez **Enregistrer**.

    ![Ajouter une plage d’adresses IP de réseau virtuel supplémentaire dans le portail Azure](./media/join-windows-vm/add-vnet-address-range.png)

1. Ensuite, dans le menu à gauche dans la fenêtre du réseau virtuel, sélectionnez **Sous-réseaux**, puis choisissez **+ Sous-réseau** pour ajouter un sous-réseau.

1. Sélectionnez **+ Sous-réseau**, puis entrez un nom pour le sous-réseau, par exemple *gestion*. Spécifiez une **Plage d’adresses (bloc CIDR)** , par exemple *10.0.5.0/24*. Vérifiez que cette plage d’adresses IP ne chevauche pas d’autres plages d’adresses Azure ou locales existantes. Laissez les autres options avec leur valeur par défaut, puis sélectionnez **OK**.

    ![Créer une configuration de sous-réseau dans le portail Azure](./media/join-windows-vm/create-subnet.png)

1. La création du sous-réseau prend quelques secondes. Une fois qu’il est créé, sélectionnez le *X* pour fermer la fenêtre du sous-réseau.
1. De retour dans le volet **Mise en réseau** pour créer une machine virtuelle, choisissez le sous-réseau que vous avez créé dans le menu déroulant, par exemple *gestion*. Là encore, veillez à choisir le sous-réseau approprié et ne déployez pas votre machine virtuelle sur le même sous-réseau que votre domaine managé.
1. Pour **IP publique**, sélectionnez *Aucune* dans le menu déroulant. Quand vous utilisez Azure Bastion dans ce tutoriel pour vous connecter à la gestion, vous n’avez pas besoin qu’une adresse IP publique soit affectée à la machine virtuelle.
1. Laissez les autres options avec leur valeur par défaut, puis sélectionnez **Gestion**.
1. Définissez **Diagnostics de démarrage** sur *Désactivé*. Laissez les autres options avec leur valeur par défaut, puis sélectionnez **Vérifier + créer**.
1. Passez en revue vos paramètres de la machine virtuelle, puis sélectionnez **Créer**.

La création de la machine virtuelle ne nécessite que quelques minutes. Le portail Azure montre l’état du déploiement. Quand votre machine virtuelle est prête, sélectionnez **Accéder à la ressource**.

![Accéder à la ressource de machine virtuelle dans le portail Azure une fois qu’elle a été créée](./media/join-windows-vm/vm-created.png)

## <a name="connect-to-the-windows-server-vm"></a>Se connecter à la machine virtuelle Windows Server

Pour vous connecter de façon sécurisée à vos machines virtuelles, utilisez un hôte Azure Bastion. Avec Azure Bastion, un hôte managé est déployé dans votre réseau virtuel et fournit aux machines virtuelles des connexions RDP ou SSH basées sur le web. Aucune adresse IP publique n’est requise pour les machines virtuelles et vous n’avez pas besoin d’ouvrir de règles de groupe de sécurité réseau pour le trafic distant externe. Vous vous connectez aux machines virtuelles à l’aide du portail Azure à partir de votre navigateur web. Si nécessaire, [créez un hôte Azure Bastion][azure-bastion].

Pour utiliser un hôte Bastion pour vous connecter à votre machine virtuelle, procédez comme suit :

1. Dans le volet **Vue d’ensemble** de votre machine virtuelle, sélectionnez **Se connecter**, puis **Bastion**.

    ![Se connecter à une machine virtuelle Windows à l’aide de Bastion dans le portail Azure](./media/join-windows-vm/connect-to-vm.png)

1. Entrez les informations d’identification pour votre machine virtuelle que vous avez spécifiées dans la section précédente, puis sélectionnez **Se connecter**.

   ![Se connecter via l’hôte Bastion dans le portail Azure](./media/join-windows-vm/connect-to-bastion.png)

Si nécessaire, autorisez votre navigateur web à ouvrir des fenêtres contextuelles pour afficher la connexion Bastion. Il faut quelques secondes pour établir la connexion à votre machine virtuelle.

## <a name="join-the-vm-to-the-managed-domain"></a>Joindre la machine virtuelle au domaine managé

Maintenant que la machine virtuelle est créée et qu’une connexion RDP basée sur le web a été établie à l’aide d’Azure Bastion, joignons la machine virtuelle Windows Server au domaine managé. Ce processus est identique à celui d’un ordinateur qui se connecte à un domaine Active Directory Domain Services local normal.

1. Si **Gestionnaire de serveur** ne s’ouvre pas par défaut lorsque vous vous connectez à la machine virtuelle, sélectionnez le menu **Démarrer**, puis choisissez **Gestionnaire de serveur**.
1. Dans le volet gauche de la fenêtre **Gestionnaire de serveur**, cliquez sur l’option **Serveur local**. Sous **Propriétés** dans le volet droit, choisissez **Groupe de travail**.

    ![Ouvrez Gestionnaire de serveur sur la machine virtuelle et modifiez la propriété du groupe de travail.](./media/join-windows-vm/server-manager.png)

1. Dans la fenêtre **Propriétés système**, sélectionnez **Modifier** pour joindre le domaine managé.

    ![Choisir de modifier les propriétés de groupe de travail ou du domaine](./media/join-windows-vm/change-domain.png)

1. Dans la zone **Domaine**, spécifiez le nom de votre domaine managé, par exemple *aaddscontoso.com*, puis sélectionnez **OK**.

    ![Spécifier le domaine managé à joindre](./media/join-windows-vm/join-domain.png)

1. Entrez les informations d’identification du domaine pour vous joindre au domaine. Fournissez les informations d’identification d’un utilisateur membre du domaine managé. Le compte doit faire partie du domaine managé ou du locataire Azure AD : les comptes d’annuaires externes associés à votre locataire Azure AD ne peuvent pas s’authentifier correctement au cours du processus de jonction de domaine.

    Les informations d’identification du compte peuvent être spécifiées de l’une des manières suivantes :

    * **Format UPN** (recommandé) : Entrez le suffixe du nom de l’utilisateur principal (UPN) pour le compte d’utilisateur, tel qu’il est configuré dans Azure AD. Par exemple, le suffixe UPN de l’utilisateur *contosoadmin* serait `contosoadmin@aaddscontoso.onmicrosoft.com`. Il existe deux cas d’utilisation courants dans lesquels le format UPN peut être utilisé de façon fiable pour se connecter au domaine, au lieu du format *SAMAccountName* :
        * Si le préfixe UPN d’un utilisateur est long, par exemple *deehasareallylongname*, le *SAMAccountName* peut être généré automatiquement.
        * Si plusieurs utilisateurs ont le même préfixe UPN au sein de votre locataire Azure AD, par exemple, *dee*, leur format *SAMAccountName* peut être généré automatiquement.
    * **Format SAMAccountName** : Entrez le nom du compte au format *SAMAccountName*. Par exemple, le *SAMAccountName* de l’utilisateur *contosoadmin* serait `AADDSCONTOSO\contosoadmin`.

1. Quelques secondes sont nécessaires pour la jonction au domaine managé. Quand c’est terminé, le message suivant vous accueille dans le domaine :

    ![Bienvenue dans le domaine](./media/join-windows-vm/join-domain-successful.png)

    Sélectionnez **OK** pour continuer.

1. Pour terminer le processus de jonction au domaine managé, redémarrez la machine virtuelle.

> [!TIP]
> Vous pouvez joindre une machine virtuelle à un domaine en utilisant PowerShell avec l’applet de commande [Add-Computer][add-computer]. L’exemple suivant joint le domaine *AADDSCONTOSO*, puis redémarre la machine virtuelle. À l’invite, entrez les informations d’identification d’un utilisateur membre du domaine managé :
>
> `Add-Computer -DomainName AADDSCONTOSO -Restart`
>
> Pour joindre une machine virtuelle à un domaine sans vous y connecter et configurer manuellement la connexion, vous pouvez utiliser l’applet de commande Azure PowerShell [Set-AzVmAdDomainExtension][set-azvmaddomainextension].

Une fois la machine virtuelle Windows Server redémarrée, toutes les stratégies appliquées dans le domaine managé sont envoyées (push) à la machine virtuelle. Vous pouvez également vous connecter à la machine virtuelle Windows Server en utilisant des informations d’identification du domaine appropriées.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans le tutoriel suivant, vous utiliserez cette machine virtuelle Windows Server pour installer les outils de gestion qui vous permettent d’administrer le domaine managé. Si vous ne voulez pas continuer cette série de tutoriels, passez en revue les étapes de nettoyage suivantes pour [supprimer la machine virtuelle](#delete-the-vm). Sinon, [passez au tutoriel suivant](#next-steps).

### <a name="unjoin-the-vm-from-the-managed-domain"></a>Annuler la jonction de la machine virtuelle au domaine managé

Pour supprimer la machine virtuelle du domaine managé, réeffectuez les étapes permettant de [joindre la machine virtuelle à un domaine](#join-the-vm-to-the-managed-domain). Au lieu de joindre le domaine managé, choisissez de joindre un groupe de travail, comme le *WORKGROUP* (groupe de travail) par défaut. Une fois la machine virtuelle redémarrée, l’objet ordinateur est supprimé du domaine managé.

Si vous [supprimez la machine virtuelle](#delete-the-vm) sans annuler la jonction au domaine, un objet ordinateur orphelin est laissé dans Azure AD DS.

### <a name="delete-the-vm"></a>Supprimer la machine virtuelle

Si vous ne voulez pas utiliser cette machine virtuelle Windows Server, supprimez-la en effectuant les étapes suivantes :

1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.
1. Choisissez votre groupe de ressources, par exemple *myResourceGroup*.
1. Sélectionnez votre machine virtuelle, par exemple *myVM*, puis sélectionnez **Supprimer**. Sélectionnez **Oui** pour confirmer la suppression de la ressource. La suppression de la machine virtuelle prend quelques minutes.
1. Une fois la machine virtuelle supprimée, sélectionnez le disque du système d’exploitation, la carte d’interface réseau et les autres ressources avec le préfixe *myVM-* , et supprimez-les.

## <a name="troubleshoot-domain-join-issues"></a>Résoudre les problèmes de jonction à un domaine

La machine virtuelle Windows Server doit être jointe avec succès au domaine managé, de la même façon qu’un ordinateur local normal est joint à un domaine Active Directory Domain Services. Si la machine virtuelle Windows Server ne peut pas se joindre au domaine managé, cela indique qu’il existe un problème lié à la connectivité ou aux informations d’identification. Consultez les sections de dépannage suivantes pour effectuer avec succès la jonction au domaine managé.

### <a name="connectivity-issues"></a>Problèmes de connectivité

Si vous ne recevez pas d’invite demandant des informations d’identification pour la jonction au domaine, c’est qu’il y a un problème de connectivité. La machine virtuelle ne peut pas atteindre le domaine managé sur le réseau virtuel.

Après avoir essayé chacune de ces étapes de dépannage, réessayez de joindre la machine virtuelle Windows Server au domaine managé.

* Vérifiez que la machine virtuelle est connectée au même réseau virtuel que celui où Azure AD DS est activé, ou qu’elle a une connexion réseau appairée.
* Effectuez un test Ping du nom de domaine DNS du domaine managé, par exemple `ping aaddscontoso.com`.
    * Si le test Ping échoue, essayez d’effectuer un test Ping des adresses IP du domaine managé, par exemple `ping 10.0.0.4`. L’adresse IP de votre environnement est affiché dans la page *Propriétés* quand vous sélectionnez le domaine managé dans votre liste de ressources Azure.
    * Si le test ping de l’adresse IP aboutit contrairement à celui du domaine, il se peut que la fonction DNS ne soit pas correctement configurée. Vérifiez que les adresses IP du domaine managé sont configurées en tant que serveurs DNS pour le réseau virtuel.
* Essayez de vider le cache du résolveur DNS sur la machine virtuelle en utilisant la commande `ipconfig /flushdns`.

### <a name="credentials-related-issues"></a>Problèmes liés aux informations d’identification

Si vous recevez une invite demandant des informations d’identification pour joindre le domaine, puis une erreur après avoir entré ces informations d’identification, cela signifie que la machine virtuelle est en mesure de se connecter au domaine managé. Les informations d’identification que vous avez fournies n’autorisent pas la machine virtuelle à se joindre au domaine managé.

Après avoir essayé chacune de ces étapes de dépannage, réessayez de joindre la machine virtuelle Windows Server au domaine managé.

* Vérifiez que le compte d’utilisateur spécifié appartient au domaine managé.
* Vérifiez que le compte fait partie du domaine managé ou du locataire Azure AD. Les comptes de répertoires externes associés à votre Client Azure AD ne peuvent pas s’authentifier correctement au cours du processus de jonction de domaine.
* Essayez en utilisant le format UPN pour spécifier les informations d’identification, par exemple `contosoadmin@aaddscontoso.onmicrosoft.com`. S’il existe de nombreux utilisateurs avec le même préfixe UPN sur votre locataire, ou si votre préfixe UPN est très long, le *SAMAccountName* de votre compte peut être généré automatiquement. Dans ces cas-là, le format *SAMAccountName* de votre compte peut être différent de ce que vous attendez ou de ce que vous utilisez dans votre domaine local.
* Vérifiez que vous avez [activé la synchronisation de mot de passe][password-sync] avec votre domaine managé. Si vous n’effectuez pas cette étape de configuration, les hachages de mot de passe nécessaires ne seront pas présents dans le domaine managé et votre tentative de connexion ne pourra pas être authentifiée correctement.
* Attendez la fin de la synchronisation du mot de passe. Quand le mot de passe d’un compte d’utilisateur est changé, une synchronisation automatique en arrière-plan à partir d’Azure AD met à jour le mot de passe dans Azure AD DS. Il faut un certain temps pour que le mot de passe soit disponible pour être utilisé dans une jonction de domaine.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer une machine virtuelle Windows Server
> * Se connecter à la machine virtuelle Windows Server sur un réseau virtuel Azure
> * Joindre la machine virtuelle au domaine managé

Pour administrer votre domaine managé, configurez une machine virtuelle de gestion en utilisant le Centre d’administration Active Directory.

> [!div class="nextstepaction"]
> [Installer des outils d’administration sur une machine virtuelle de gestion](tutorial-create-management-vm.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[vnet-peering]: ../virtual-network/virtual-network-peering-overview.md
[password-sync]: ./tutorial-create-instance.md
[add-computer]: /powershell/module/microsoft.powershell.management/add-computer
[azure-bastion]: ../bastion/tutorial-create-host-portal.md
[set-azvmaddomainextension]: /powershell/module/az.compute/set-azvmaddomainextension