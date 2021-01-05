---
title: Créer et utiliser des stratégies de groupe dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment modifier les objets de stratégie de groupe (GPO) intégrés et créer vos propres stratégies personnalisées dans un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: f1f2499c49c4adf16b632bc75c246a28330ad27b
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619383"
---
# <a name="administer-group-policy-in-an-azure-active-directory-domain-services-managed-domain"></a>Administrer la stratégie de groupe dans un domaine géré par Azure Active Directory Domain Services

Les paramètres des objets utilisateur et ordinateur dans Azure Active Directory Domain Services (Azure AD DS) sont souvent managés à l’aide d’objets de stratégie de groupe (GPO). Azure AD DS inclut des objets de stratégie de groupe intégrés pour les conteneurs *Utilisateurs AADDC* et *Ordinateurs AADDC*. Vous pouvez personnaliser ces objets de stratégie de groupe intégrés pour configurer la stratégie de groupe selon les besoins de votre environnement. Les membres du groupe *Administrateurs Azure AD DC* disposent de privilèges d’administration de stratégie de groupe pour le domaine managé Azure AD DS et peuvent également créer des objets de stratégie de groupe et des unités d’organisation (OU) personnalisés. Pour plus d’informations sur les stratégie de groupe et son fonctionnement, consultez la page [Vue d’ensemble de la stratégie de groupe][group-policy-overview].

Dans un environnement hybride, les stratégies de groupe configurées dans un environnement AD DS local ne sont pas synchronisées avec Azure AD DS. Pour définir les paramètres de configuration des utilisateurs ou des ordinateurs dans Azure AD DS, modifiez l’un des objets de stratégie de groupe par défaut ou créez un objet de stratégie de groupe personnalisé.

Cet article indique comment installer les outils de gestion de stratégie de groupe, modifier les objets de stratégie de groupe intégrés et créer des objets de stratégie de groupe personnalisés.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Une machine virtuelle de gestion Windows Server jointe au domaine managé Azure AD DS.
    * Si nécessaire, suivez les étapes du tutoriel pour [créer et joindre une machine virtuelle Windows Server à un domaine managé][create-join-windows-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.

> [!NOTE]
> Vous pouvez utiliser des modèles d'administration de stratégies de groupe en copiant les nouveaux modèles sur la station de travail de gestion. Copiez les fichiers *.admx* sous `%SYSTEMROOT%\PolicyDefinitions` et copiez les fichiers *.adml* spécifiques à l'environnement local sous `%SYSTEMROOT%\PolicyDefinitions\[Language-CountryRegion]`, sachant que `Language-CountryRegion` correspond à la langue et à la région des fichiers *.adml*.
>
> Par exemple, copiez la version Anglais (États-Unis) des fichiers *.adml* dans le dossier `\en-us`.
>
> Vous pouvez également centraliser le stockage de votre modèle d'administration des stratégies de groupe sur les contrôleurs de domaine qui appartiennent au domaine managé. Pour plus d'informations, consultez [Créer et gérer le magasin central des modèles d'administration des stratégies de groupe sous Windows](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra).

## <a name="install-group-policy-management-tools"></a>Installez les outils de gestion de stratégie de groupe

Pour créer et configurer les objets de stratégie de groupe (GPO), vous devez installer les outils de gestion de stratégie de groupe. Ces outils peuvent être installés en tant que fonctionnalité de Windows Server. Pour plus d’informations sur l’installation des outils d’administration sur un client Windows, consultez [Installer les outils d’administration de serveur distant][install-rsat].

1. Connectez-vous à votre machine virtuelle de gestion. Pour connaître les différentes étapes vous permettant de vous connecter au portail Azure, consultez la page [Se connecter à une machine virtuelle Windows Server][connect-windows-server-vm].
1. Le **Gestionnaire de serveur** doit s’ouvrir par défaut quand vous vous connectez à la machine virtuelle. Si ce n’est pas le cas, dans le menu **Démarrer**, sélectionnez **Gestionnaire de serveur**.
1. Dans le volet *Tableau de bord* de la fenêtre **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et des fonctionnalités**.
1. Dans la page **Avant de commencer** de l’*Assistant Ajout de rôles et de fonctionnalités*, sélectionnez **Suivant**.
1. Pour le *Type d’installation*, laissez l’option **Installation basée sur un rôle ou une fonctionnalité** cochée et sélectionnez **Suivant**.
1. Dans la page **Sélection du serveur**, choisissez la machine virtuelle actuelle dans le pool de serveurs, par exemple *myvm.aaddscontoso.com*, puis sélectionnez **Suivant**.
1. Sur la page **Rôles de serveurs**, cliquez sur **Suivant**.
1. Sur la page **Fonctionnalités**, sélectionnez la fonctionnalité **Gestion des stratégies de groupe**.

    ![Installez la « gestion des stratégies de groupe » à partir de la page Fonctionnalités](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-gp-management.png)

1. Dans la page **Confirmation**, sélectionnez **Installer**. L’installation des outils de gestion des stratégies de groupe peut prendre une ou deux minutes.
1. Une fois l’installation de la fonctionnalité terminée, sélectionnez **Fermer** pour quitter l’Assistant **Ajout de rôles et de fonctionnalités**.

## <a name="open-the-group-policy-management-console-and-edit-an-object"></a>Ouvrez la Console de gestion des stratégies de groupe et modifiez un objet

Il existe des objets de stratégie de groupe (GPO) par défaut pour les utilisateurs et les ordinateurs dans un domaine managé. La fonctionnalité de gestion des stratégie de groupe étant installée grâce à la section précédente, nous allons pouvoir afficher et modifier un objet de stratégie de groupe existant. Dans la section suivante, vous allez créer un objet de stratégie de groupe personnalisé.

> [!NOTE]
> Pour administrer une stratégie de groupe dans un domaine managé, vous devez être connecté à un compte d’utilisateur membre du groupe *d’administrateurs AAD DDC*.

1. Dans l’écran d’accueil, sélectionnez **Outils d’administration**. Une liste des outils de gestion disponibles s’affiche, dont la **gestion des stratégie de groupe** installée dans la section précédente.
1. Cliquez sur **Gestion des stratégies de groupe** pour ouvrir la console de gestion des stratégies de groupe (GPMC).

    ![La console de gestion des stratégies de groupe s’ouvre et vous permet de modifier les objets de stratégie de groupe](./media/active-directory-domain-services-admin-guide/gp-management-console.png)

Il existe deux objets de stratégie de groupe (GPO) intégrés dans un domaine managé : un pour le conteneur *Ordinateurs AADDC* et un autre pour le conteneur *AADDC utilisateurs*. Vous pouvez personnaliser ces objets de stratégie de groupe pour configurer la stratégie de groupe selon vos besoins dans votre domaine managé.

1. Dans la console **Gestion des stratégies de groupe**, développez le nœud **Forest: aaddscontoso.com**. Ensuite, développez les nœuds **Domaines**.

    Il existe deux conteneurs pour *Ordinateurs AADDC* et *Utilisateurs AADDC*. Un objet de stratégie de groupe par défaut est appliqué à chacun de ces conteneurs.

    ![Objets de stratégie de groupe intégrés appliqués aux conteneurs « Ordinateurs AADDC » et « Utilisateurs AADDC » par défaut](./media/active-directory-domain-services-admin-guide/builtin-gpos.png)

1. Ces objets de stratégie de groupe intégrés peuvent être personnalisés pour configurer des stratégies de groupe particulières sur votre domaine managé. Cliquez avec le bouton droit sur l’un des objets de stratégie de groupe, tel que *Objets de stratégie de groupe ordinateurs AADDC*, puis choisissez **Modifier...** .

    ![Choisissez l’option permettant de « modifier » l’un des objets de stratégie de groupe intégrés](./media/active-directory-domain-services-admin-guide/edit-builtin-gpo.png)

1. L’outil Éditeur de gestion des stratégies de groupe s’ouvre pour vous permettre de personnaliser l’objet de stratégie de groupe, comme *les stratégies de compte* :

    ![Capture d’écran de l’Éditeur de gestion des stratégies de groupe.](./media/active-directory-domain-services-admin-guide/gp-editor.png)

    Lorsque vous avez terminé, sélectionnez **Fichier > Enregistrer** pour enregistrer la stratégie. Les ordinateurs actualisent la stratégie de groupe par défaut toutes les 90 minutes et appliquent les modifications que vous avez apportées.

## <a name="create-a-custom-group-policy-object"></a>Créez un objet de stratégie de groupe personnalisé

Pour regrouper des paramètres de stratégie similaires, vous créez souvent des objets de stratégie de groupe supplémentaires au lieu d’appliquer tous les paramètres requis dans le l’objet de stratégie de groupe unique par défaut. Avec Azure AD DS, vous pouvez créer ou importer vos propres objets de stratégie de groupe personnalisés et les associer à une unité d’organisation personnalisée. Si vous devez d’abord créer une unité d’organisation personnalisée, consultez la page [Créer une unité d’organisation personnalisée dans un domaine managé](create-ou.md).

1. Dans la console **gestion des stratégies de groupe**, sélectionnez votre unité d’organisation personnalisée (OU), telle que *MyCustomOU*. Cliquez avec le bouton droit sur l’unité d’organisation puis sélectionnez **Créer un objet de stratégie de groupe et le lier ici…**  :

    ![Créez un objet de stratégie de groupe personnalisé dans la Console de gestion des stratégies de groupe](./media/active-directory-domain-services-admin-guide/gp-create-gpo.png)

1. Indiquez le nom du nouvel objet de stratégie de groupe, tel que *Mon objet de stratégie de groupe personnalisé*, puis sélectionnez **OK**. Vous pouvez éventuellement baser cet objet de stratégie de groupe personnalisé sur un objet de stratégie de groupe existant et un ensemble d’options de stratégie.

    ![SIndiquez le nom du nouvel objet de stratégie de groupe](./media/active-directory-domain-services-admin-guide/gp-specify-gpo-name.png)

1. Un objet de stratégie de groupe est créé et associé à votre unité d’organisation personnalisée. Pour configurer les paramètres de stratégie, cliquez avec le bouton droit sur l’objet de stratégie de groupe personnalisé et sélectionnez **Modifier…**  :

    ![Choisissez l’option vous permettant de « modifier » votre objet de stratégie de groupe personnalisé](./media/active-directory-domain-services-admin-guide/gp-gpo-created.png)

1. L’outil **Éditeur de gestion des stratégies de groupe** s’ouvre pour vous permettre de personnaliser l’objet de stratégie de groupe :

    ![Personnaliser l’objet de stratégie de groupe pour configurer les paramètres en fonction de vos besoins](./media/active-directory-domain-services-admin-guide/gp-customize-gpo.png)

    Lorsque vous avez terminé, sélectionnez **Fichier > Enregistrer** pour enregistrer la stratégie. Les ordinateurs actualisent la stratégie de groupe par défaut toutes les 90 minutes et appliquent les modifications que vous avez apportées.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les paramètres de stratégie de groupe disponibles et que vous pouvez configurer à l’aide de la console de gestion des stratégies de groupe, consultez la page [Utiliser les éléments de préférence de stratégie de groupe][group-policy-console].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[connect-windows-server-vm]: join-windows-vm.md#connect-to-the-windows-server-vm

<!-- EXTERNAL LINKS -->
[group-policy-overview]: /previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831791(v=ws.11)
[install-rsat]: /windows-server/remote/remote-server-administration-tools#BKMK_Thresh
[group-policy-console]: /previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn789194(v=ws.11)
