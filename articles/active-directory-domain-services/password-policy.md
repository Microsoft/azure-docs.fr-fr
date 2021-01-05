---
title: Créer et utiliser des stratégies de mot de passe dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment et pourquoi utiliser des stratégies de mot de passe affinées pour sécuriser et contrôler les mots de passe de comptes dans un domaine managé Azure AD DS.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 1a14637e-b3d0-4fd9-ba7a-576b8df62ff2
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: df132af1675b3f373fe1eab5685c5d2f07813445
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619230"
---
# <a name="password-and-account-lockout-policies-on-azure-active-directory-domain-services-managed-domains"></a>Stratégies de mot de passe et de verrouillage de compte sur des domaines managés Azure Active Directory Domain Services

Pour gérer la sécurité des utilisateurs dans Azure AD DS (Azure Active Directory Domain Services), vous pouvez définir des stratégies de mot de passe affinées qui contrôlent les paramètres de verrouillage des comptes ou la longueur minimale et la complexité des mots de passe. Une stratégie de mot de passe affinée par défaut est créée et appliquée à tous les utilisateurs membres d’un domaine managé Azure AD DS. Pour assurer un contrôle précis et répondre à des besoins métier ou de conformité spécifiques, il est possible de créer et d’appliquer des stratégies supplémentaires à des groupes d’utilisateurs spécifiques.

Cet article montre comment créer et configurer une stratégie de mot de passe affinée dans Azure AD DS à partir du Centre d’administration Active Directory.

> [!NOTE]
> Les stratégies de mot de passe ne sont disponibles que pour les domaines managés créés à l’aide du modèle de déploiement Resource Manager. Dans le cas des anciens domaines managés créés suivant le modèle Classic, [migrez du modèle de réseau virtuel Classic vers le modèle Resource Manager][migrate-from-classic].

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
  * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
  * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
  * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
  * Le domaine managé doit avoir été créé à l’aide d’un modèle de déploiement Resource Manager. Si nécessaire, [Migrez du modèle de réseau virtuel Classic vers le modèle Resource Manager][migrate-from-classic].
* Machine virtuelle de gestion Windows Server jointe au domaine managé.
  * Si nécessaire, suivez le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.

## <a name="default-password-policy-settings"></a>Paramètres de la stratégie de mot de passe par défaut

Les stratégies de mot de passe affinées (SMPA) vous permettent d’appliquer des restrictions spécifiques pour les stratégies de verrouillage de compte et de mot de passe à différents utilisateurs d’un domaine. Par exemple, vous pouvez sécuriser les comptes privilégiés en leur appliquant des paramètres de verrouillage de compte plus stricts que pour les comptes non privilégiés standard. Vous pouvez créer plusieurs stratégies de mot de passe affinées dans un domaine managé et spécifier l’ordre de priorité selon lequel elles doivent être appliquées aux utilisateurs.

Pour plus d’informations sur les stratégies de mot de passe et l’utilisation du Centre d’administration Active Directory, consultez les articles suivants :

* [En savoir plus sur les stratégies de mot de passe affinées](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770394(v=ws.10))
* [Configurer des stratégies de mot de passe affinées à partir du Centre d’administration Active Directory](/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#fine_grained_pswd_policy_mgmt)

Les stratégies sont distribuées par le biais de l’association de groupes dans un domaine managé et les modifications que vous apportez sont appliquées à la connexion utilisateur suivante. La modification de la stratégie ne déverrouille pas un compte d’utilisateur qui est déjà verrouillé.

Les stratégies de mot de passe se comportent un peu différemment en fonction de la manière dont le compte d’utilisateur auquel elles sont appliquées a été créé. Deux méthodes sont possibles pour créer un compte d’utilisateur dans Azure AD DS :

* Le compte d’utilisateur peut être synchronisé à partir d’Azure AD. Cette méthode concerne les comptes d’utilisateur cloud uniquement qui sont créés directement dans Azure, et les comptes d’utilisateur hybrides qui sont synchronisés à partir d’un environnement AD DS local à l’aide d’Azure AD Connect.
    * La majorité des comptes d’utilisateur dans Azure AD DS sont créés par le biais du processus de synchronisation à partir d’Azure AD.
* Le compte d’utilisateur peut être créé manuellement dans un domaine managé et n’existe pas dans Azure AD.

Quelle que soit la méthode de création de compte d’utilisateur choisie, les stratégies de verrouillage de compte suivantes sont appliquées à tous les utilisateurs par la stratégie de mot de passe par défaut dans Azure AD DS :

* **Durée de verrouillage de compte :** 30
* **Nombre d’échecs d’ouverture de session autorisés :** 5
* **Réinitialisation du nombre d’échecs d’ouverture de session après :** 30 minutes
* **Âge maximal du mot de passe (durée de vie) :** 90 jours

Avec ces paramètres par défaut, les comptes d’utilisateurs sont verrouillés pendant 30 minutes si cinq mots de passe non valides sont utilisés en l’espace de 2 minutes. Les comptes sont déverrouillés automatiquement après 30 minutes.

Les verrouillages de comptes se produisent uniquement dans le domaine managé. Les comptes d’utilisateur sont verrouillés uniquement dans Azure AD DS et seulement en cas d’échec des tentatives de connexion au domaine managé. Les comptes d’utilisateur qui ont été synchronisés à partir d’Azure AD ou localement ne sont pas verrouillés dans leurs répertoires sources, mais uniquement dans Azure AD DS.

Si vous utilisez une stratégie de mot de passe Azure AD qui spécifie un âge maximal du mot de passe supérieur à 90 jours, ce paramètre d’âge est appliqué à la stratégie par défaut dans Azure AD DS. Vous pouvez configurer une stratégie de mot de passe personnalisée pour définir un âge maximal du mot de passe différent dans Azure AD DS. Soyez vigilant si l’âge maximal du mot de passe configuré dans une stratégie de mot de passe Azure AD DS est inférieur à celui défini dans Azure AD ou dans un environnement AD DS local. Dans ce scénario, le mot de passe d’un utilisateur peut expirer dans Azure AD DS avant que l’utilisateur ne soit invité à le changer dans Azure AD ou dans un environnement AD DS local.

Pour les comptes d’utilisateur créés manuellement dans un domaine managé, les paramètres de mot de passe supplémentaires suivants sont également appliqués à partir de la stratégie par défaut. Ces paramètres ne s’appliquent pas aux comptes d’utilisateur synchronisés à partir d’Azure AD, car un utilisateur ne peut pas mettre à jour son mot de passe directement dans Azure AD DS.

* **Longueur minimale du mot de passe (caractères) :** 7
* **Les mots de passe doivent respecter les conventions de complexité**

Vous ne pouvez pas modifier les paramètres de verrouillage de compte ou de mot de passe dans la stratégie de mot de passe par défaut. En revanche, les membres du groupe *Administrateurs AAD DC* peuvent créer des stratégies de mot de passe personnalisées et les configurer pour qu’elles remplacent (prioritairement) la stratégie intégrée par défaut, comme cela est expliqué dans la section suivante.

## <a name="create-a-custom-password-policy"></a>Créer une stratégie de mot de passe personnalisée

Après avoir créé et exécuté plusieurs applications dans Azure, vous souhaiterez peut-être configurer une stratégie de mot de passe personnalisée. Par exemple, vous pouvez créer une stratégie pour définir différents paramètres de stratégie de verrouillage de compte.

Les stratégies de mot de passe personnalisées sont appliquées aux groupes dans un domaine managé. Cette configuration remplace de fait la stratégie par défaut.

Pour créer une stratégie de mot de passe personnalisée, utilisez les outils d’administration Active Directory à partir d’une machine virtuelle jointe à un domaine. Le Centre d’administration Active Directory vous permet d’afficher, de modifier et de créer des ressources dans un domaine managé, notamment des unités d’organisation.

> [!NOTE]
> Pour créer une stratégie de mot de passe personnalisée dans un domaine managé, vous devez être connecté à un compte d’utilisateur membre du groupe *Administrateurs AAD DC*.

1. Dans l’écran d’accueil, sélectionnez **Outils d’administration**. La liste des outils de gestion disponibles qui ont été installés dans le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm] s’affiche à l’écran.
1. Pour créer et gérer des unités d’organisation, sélectionnez **Centre d’administration Active Directory** dans la liste des outils d’administration.
1. Dans le volet gauche, choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Ouvrez le conteneur **Système**, puis la classe d’objets PSC (**Password Settings Container**).

    Une stratégie de mot de passe intégrée pour le domaine managé s’affiche. Vous ne pouvez pas modifier cette stratégie intégrée. Vous devez créer une stratégie de mot de passe personnalisée pour remplacer la stratégie par défaut.

    ![Créer une stratégie de mot de passe à partir du Centre d’administration Active Directory](./media/password-policy/create-password-policy-adac.png)

1. Dans le volet **Tâches** à droite, sélectionnez **Nouveau > Paramètres de mot de passe**.
1. Dans la boîte de dialogue **Créer des paramètres de mot de passe**, entrez un nom pour la stratégie, par exemple *MyCustomFGPP*.
1. Quand plusieurs stratégies de mot de passe ont été définies, la stratégie appliquée à un utilisateur est celle qui a la précédence, ou priorité, la plus élevée. Plus le numéro est faible, plus la priorité est élevée. La stratégie de mot de passe par défaut a une priorité de *200*.

    Définissez la précédence de votre stratégie de mot de passe personnalisée pour remplacer la valeur par défaut, par exemple *1*.

1. Modifiez les autres paramètres de la stratégie de mot de passe comme vous le souhaitez. Souvenez-vous des points clés suivants :

    * Les paramètres, tels que la complexité, l’âge ou le délai d’expiration du mot de passe, s’appliquent uniquement aux utilisateurs ayant été créés manuellement dans un domaine managé.
    * Les paramètres de verrouillage de compte s’appliquent à tous les utilisateurs, mais prennent effet uniquement dans le domaine managé et non dans Azure AD lui-même.

    ![Créer une stratégie de mot de passe affinée personnalisée](./media/password-policy/custom-fgpp.png)

1. Décochez **Protéger contre la suppression accidentelle**. Si cette option est sélectionnée, vous ne pouvez pas enregistrer la SMPA.
1. Dans la section **S’applique directement à**, cliquez sur le bouton **Ajouter**. Dans la boîte de dialogue **Sélectionner Utilisateurs ou Groupes**, sélectionnez le bouton **Emplacements**.

    ![Sélectionner les utilisateurs et les groupes auxquels appliquer la stratégie de mot de passe](./media/password-policy/fgpp-applies-to.png)

1. Les stratégies de mot de passe peuvent uniquement être appliquées à des groupes. Dans la boîte de dialogue **Emplacements**, développez le nom de domaine, par exemple *aaddscontoso.com*, puis sélectionnez une unité d’organisation, par exemple **Utilisateurs AADDC**. Si vous avez à disposition une unité d’organisation personnalisée qui contient un groupe d’utilisateurs auquel vous souhaitez appliquer une stratégie, sélectionnez cette unité d’organisation.

    ![Sélectionner l’unité d’organisation à laquelle le groupe appartient](./media/password-policy/fgpp-container.png)

1. Tapez le nom du groupe auquel vous souhaitez appliquer la stratégie, puis sélectionnez **Vérifier les noms** pour vérifier que le groupe existe.

    ![Rechercher et sélectionner le groupe auquel appliquer la SMPA](./media/password-policy/fgpp-apply-group.png)

1. Comme le nom du groupe que vous avez sélectionné figure dans la section **S’applique directement à**, sélectionnez **OK** pour enregistrer votre stratégie de mot de passe personnalisée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les stratégies de mot de passe et l’utilisation du Centre d’administration Active Directory, consultez les articles suivants :

* [En savoir plus sur les stratégies de mot de passe affinées](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770394(v=ws.10))
* [Configurer des stratégies de mot de passe affinées à partir du Centre d’administration Active Directory](/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#fine_grained_pswd_policy_mgmt)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[migrate-from-classic]: migrate-from-classic-vnet.md
