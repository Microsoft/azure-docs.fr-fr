---
title: Synchronisation délimitée pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment utiliser le Portail Azure pour configurer la synchronisation à étendue limitée entre Azure AD et un domaine managé Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 9389cf0f-0036-4b17-95da-80838edd2225
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/24/2020
ms.author: justinha
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4e65b47b2a1fd71c69ecb350f60df1fedff66b74
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618907"
---
# <a name="configure-scoped-synchronization-from-azure-ad-to-azure-active-directory-domain-services-using-the-azure-portal"></a>Configuration de la synchronisation à étendue limitée entre Azure AD et Azure Active Directory Domain Services à l’aide du Portail Azure

Pour assurer des services d’authentification, Azure Active Directory Domain Services (Azure AD DS) synchronise les utilisateurs et les groupes à partir d’Azure AD. Dans un environnement hybride, les utilisateurs et les groupes d’un environnement Active Directory Domain Services (AD DS) local peuvent d’abord être synchronisés avec Azure AD à l’aide d’Azure AD Connect, puis avec un domaine managé Azure AD DS.

Par défaut, tous les utilisateurs et groupes d’un annuaire Azure AD sont synchronisés avec un domaine managé. Si vous avez des besoins spécifiques, vous pouvez opter pour la synchronisation d’un ensemble défini d’utilisateurs.

Cet article explique comment configurer la synchronisation à étendue limitée, puis modifier ou désactiver l’ensemble des utilisateurs inclus à l’aide du Portail Azure. Vous pouvez également [suivre cette procédure avec PowerShell][scoped-sync-powershell].

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][tutorial-create-instance].
* Vous devez disposer de privilèges *Administrateur général* dans votre locataire Azure AD pour modifier l’étendue de la synchronisation Azure AD DS.

## <a name="scoped-synchronization-overview"></a>Vue d’ensemble de la synchronisation délimitée

Par défaut, tous les utilisateurs et groupes d’un annuaire Azure AD sont synchronisés avec un domaine managé. Si seuls quelques utilisateurs ont besoin d’accéder au domaine managé, vous pouvez synchroniser uniquement ces comptes d’utilisateurs. Cette synchronisation délimitée est basée sur les groupes. Quand vous configurez la synchronisation délimitée basée sur les groupes, seuls les comptes d’utilisateurs qui appartiennent aux groupes que vous spécifiez sont synchronisés avec le domaine managé. Les groupes imbriqués ne sont pas synchronisés ; seuls les groupes explicitement sélectionnés le sont.

Vous pouvez modifier l’étendue de synchronisation lors de la création du domaine managé ou une fois qu’il est déployé. À présent, il est également possible de modifier l’étendue de la synchronisation sur un domaine managé existant sans avoir à le recréer.

Pour en savoir plus sur le processus de synchronisation, consultez [Comprendre la synchronisation dans Azure AD Domain Services][concepts-sync].

> [!WARNING]
> La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les considérations suivantes s'appliquent :
>
>  * Lorsque vous modifiez l’étendue de synchronisation d’un domaine managé, une resynchronisation complète est déclenchée.
>  * Les objets qui ne sont plus requis dans le domaine managé sont supprimés. De nouveaux objets sont créés dans le domaine managé.

## <a name="enable-scoped-synchronization"></a>Activation de la synchronisation à étendue limitée

Pour activer la synchronisation à étendue limitée sur le Portail Azure, procédez comme suit :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Sélectionnez **Synchronisation** dans le menu de gauche.
1. Pour *Type de synchronisation*, sélectionnez **Étendue limitée**.
1. Choisissez **Sélectionner des groupes**, puis recherchez et choisissez les groupes à ajouter.
1. Une fois que toutes les modifications ont été apportées, sélectionnez **Enregistrer l’étendue de la synchronisation**.

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés ; la resynchronisation est susceptible de prendre du temps.

## <a name="modify-scoped-synchronization"></a>Modifier la synchronisation délimitée

Pour modifier la liste des groupes dont les utilisateurs doivent être synchronisés avec le domaine managé, effectuez les étapes suivantes :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Sélectionnez **Synchronisation** dans le menu de gauche.
1. Pour ajouter un groupe, choisissez **+ Sélectionner des groupes** dans la partie supérieure, puis choisissez les groupes à ajouter.
1. Pour supprimer un groupe de l’étendue de la synchronisation, sélectionnez-le dans la liste des groupes actuellement synchronisés et choisissez **Supprimer les groupes**.
1. Une fois que toutes les modifications ont été apportées, sélectionnez **Enregistrer l’étendue de la synchronisation**.

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés ; la resynchronisation est susceptible de prendre du temps.

## <a name="disable-scoped-synchronization"></a>Désactiver la synchronisation délimitée

Pour désactiver la synchronisation délimitée basée sur les groupes pour un domaine managé, effectuez les étapes suivantes :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Sélectionnez **Synchronisation** dans le menu de gauche.
1. Faites passer le *Type de synchronisation* de **Étendue limitée** à **Tout**, puis sélectionnez **Enregistrer l’étendue de la synchronisation**.

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés ; la resynchronisation est susceptible de prendre du temps.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le processus de synchronisation, consultez [Comprendre la synchronisation dans Azure AD Domain Services][concepts-sync].

<!-- INTERNAL LINKS -->
[scoped-sync-powershell]: powershell-scoped-synchronization.md
[concepts-sync]: synchronization.md
[tutorial-create-instance]: tutorial-create-instance.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
