---
title: Partager des tableaux de bord du portail Azure à l’aide du contrôle d’accès en fonction du rôle Azure
description: Cet article vous explique comment partager un tableau de bord dans le portail Azure en utilisant le contrôle d’accès en fonction du rôle Azure.
ms.assetid: 8908a6ce-ae0c-4f60-a0c9-b3acfe823365
ms.topic: how-to
ms.date: 03/23/2020
ms.openlocfilehash: de2efd75d07c3dc7df771aad1bd9c73453dad212
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745976"
---
# <a name="share-azure-dashboards-by-using-azure-role-based-access-control"></a>Partager des tableaux de bord Azure à l’aide du contrôle d’accès en fonction du rôle Azure

Après avoir configuré un tableau de bord, vous pouvez le publier et le partager avec d’autres utilisateurs de votre organisation. Vous permettez à d'autres utilisateurs de voir votre tableau de bord au moyen du [contrôle d'accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md). Attribuez un rôle à un utilisateur ou à un groupe d'utilisateurs. Ce rôle détermine si ces utilisateurs peuvent consulter ou modifier le tableau de bord publié.

Tous les tableaux de bord publiés sont implémentés en tant que ressources Azure. Ceux-ci constituent des éléments gérables de votre abonnement et sont contenus dans un groupe de ressources. En termes de contrôle d’accès, les tableaux de bord sont traités de la même manière que les autres ressources, telles qu’une machine virtuelle ou un compte de stockage.

> [!TIP]
> Les vignettes individuelles du tableau de bord appliquent leurs propres exigences en matière de contrôle d’accès, compte tenu des ressources qu’elles affichent. Vous pouvez partager un tableau de bord à grande échelle tout en protégeant les données sur des vignettes individuelles.
> 
> 

## <a name="understanding-access-control-for-dashboards"></a>Présentation du contrôle d’accès relatif aux tableaux de bord

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet d’attribuer des utilisateurs à des rôles avec trois niveaux d’étendue distincts :

* subscription
* resource group
* resource

Les ressources héritent des autorisations que vous attribuez à l’abonnement. Le tableau de bord publié est une ressource. Certains de vos utilisateurs disposent peut-être déjà pour l'abonnement de rôles qui s'appliquent au tableau de bord publié.

Supposons que vous disposiez d’un abonnement Azure et que les différents membres de votre équipe aient reçu les rôles de *propriétaire*, de *contributeur* ou de *lecteur* pour l’abonnement. Les utilisateurs disposant du rôle de propriétaire ou de contributeur peuvent répertorier, afficher, créer, modifier ou supprimer des tableaux de bord associés à l'abonnement. Les utilisateurs disposant du rôle de lecteur peuvent répertorier et afficher des tableaux de bord, mais ils ne peuvent pas les modifier ou les supprimer. Les utilisateurs dotés d'un accès en lecture peuvent apporter des modifications locales à un tableau de bord publié (par exemple, lors de la résolution d'un problème), mais ils ne peuvent pas republier ces modifications sur le serveur. Ils peuvent effectuer une copie du tableau de bord pour leur usage personnel.

Vous pouvez également attribuer des autorisations au groupe de ressources qui contient plusieurs tableaux de bord ou à un tableau de bord spécifique. Par exemple, vous pouvez décider qu’un groupe d’utilisateurs doit disposer d’autorisations limitées dans l’ensemble de l’abonnement, mais un accès plus étendu à un tableau de bord donné. Attribuez un rôle à ces utilisateurs pour ce tableau de bord.

## <a name="publish-dashboard"></a>Publier un tableau de bord

Supposons que vous configuriez un tableau de bord que vous souhaitez partager avec un groupe d'utilisateurs dans le cadre de votre abonnement. Les étapes suivantes expliquent comment partager un tableau de bord avec un groupe appelé Gestionnaires de stockage. Vous pouvez donner le nom de votre choix à votre groupe. Pour plus d'informations, consultez [Gérer des groupes dans Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Avant d’affecter l’accès, vous devez publier le tableau de bord.

1. Dans le tableau de bord, sélectionnez **Partager**.

    ![sélectionner un partage pour votre tableau de bord](./media/azure-portal-dashboard-share-access/share-dashboard-for-access-control.png)

1. Dans **Partage + contrôle d'accès**, sélectionnez **Publier**.

    ![publier votre tableau de bord](./media/azure-portal-dashboard-share-access/publish-dashboard-for-access-control.png)

     Par défaut, le partage publie votre tableau de bord dans un groupe de ressources nommé **tableaux de bord**. Pour sélectionner un autre groupe de ressources, désactivez la case à cocher.

Votre tableau de bord est à présent publié. Si les autorisations héritées de l'abonnement vous conviennent, vous n'avez rien d'autre à faire. Les autres utilisateurs de votre organisation ont accès au tableau de bord et peuvent le modifier en fonction de leur rôle au niveau de l'abonnement.

## <a name="assign-access-to-a-dashboard"></a>Affecter l’accès à un tableau de bord

Vous pouvez attribuer un rôle à un groupe d'utilisateurs pour ce tableau de bord.

1. Après avoir publié le tableau de bord, sélectionnez l’option **Partager** ou **Annuler le partage** pour accéder à l’option **Partage + contrôle d’accès**.

1. Dans **Partage + contrôle d’accès**, sélectionnez **Gérer les utilisateurs**.

    ![gérer les utilisateurs pour un tableau de bord](./media/azure-portal-dashboard-share-access/manage-users-for-access-control.png)

1. Sélectionnez **Attributions de rôle** pour afficher la liste des utilisateurs existants déjà dotés d'un rôle pour ce tableau de bord.

1. Pour ajouter un nouvel utilisateur ou un nouveau groupe, sélectionnez **Ajouter**, puis **Ajouter une attribution de rôle**.

    ![ajouter un utilisateur pour l'accès au tableau de bord](./media/azure-portal-dashboard-share-access/manage-users-existing-users.png)

1. Sélectionnez le rôle correspondant aux autorisations à accorder. Dans cet exemple, sélectionnez **Contributeur**.

1. Sélectionnez l'utilisateur ou le groupe à attribuer auquel vous souhaitez attribuer le rôle. Si l'utilisateur ou le groupe que vous recherchez n'apparaît pas dans la liste, utilisez la zone de recherche. Votre liste de groupes disponibles dépend des groupes que vous avez créés dans Active Directory.

1. Lorsque vous avez terminé d’ajouter des utilisateurs ou des groupes, sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir la liste des rôles, voir [Rôles Azure intégrés](../role-based-access-control/built-in-roles.md).
* Pour en savoir plus sur la gestion des ressources, consultez [Gérer des ressources Azure à l'aide du portail Azure](../azure-resource-manager/management/manage-resources-portal.md).