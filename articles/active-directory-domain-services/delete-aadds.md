---
title: Supprimer Azure Active Directory Domain Services | Microsoft Docs
description: Découvrez comment désactiver ou supprimer un domaine managé Azure Active Directory Domain Services à partir du portail Azure
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 89e407e1-e1e0-49d1-8b89-de11484eee46
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: a5126abd6643eba7f63b2bf4ca984bb9892b2d7a
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619808"
---
# <a name="delete-an-azure-active-directory-domain-services-managed-domain-using-the-azure-portal"></a>Désactiver ou supprimer un domaine managé Azure Active Directory Domain Services à partir du portail Azure

Si vous n’avez plus besoin d’un domaine managé Azure Active Directory Domain Services (Azure AD DS), vous pouvez le supprimer. Il n’existe aucune option permettant de désactiver de façon provisoire ou définitive un domaine managé Azure AD DS. La suppression du domaine managé ne supprime pas et n’impacte pas défavorablement le locataire Azure AD.

Cet article explique comment utiliser le portail Azure domaine pour supprimer un domaine managé.

> [!WARNING]
> **La suppression est définitive et ne peut pas être annulée.**
> 
> Quand vous supprimez un domaine managé, voici ce qu’il se passe :
>   * Les contrôleurs de domaine pour le domaine managé sont déprovisionnés et supprimés du réseau virtuel.
>   * Les données sur le domaine managé sont supprimées définitivement. Ces données incluent les unités d’organisation personnalisées, les stratégies de groupe, les enregistrements DNS personnalisés, les principaux de service, les GMSA, etc. que vous avez créés.
>   * Les machines jointes au domaine managé perdent leur relation d’approbation avec ce domaine et doivent être disjointes de celui-ci.
>       * Vous ne pouvez pas vous connecter à ces machines en utilisant des informations d’identification AD. Pour cela, vous devez utiliser les informations d’identification de l’administrateur local de la machine.

## <a name="delete-the-managed-domain"></a>Supprimer le domaine managé

Pour supprimer un domaine managé, procédez comme suit :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**.
1. Sélectionnez le nom de votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans la page **Vue d’ensemble**, sélectionnez **Supprimer**. Pour confirmer la suppression, tapez à nouveau le nom du domaine managé, puis sélectionnez **Supprimer**.

La suppression du domaine managé peut prendre entre 15 et 20 minutes, ou plus.

## <a name="next-steps"></a>Étapes suivantes

N’hésitez pas à [faire part de vos commentaires][feedback] concernant les fonctionnalités que vous aimeriez voir dans Azure AD DS.

Si vous voulez redémarrer avec Azure AD DS, consultez [Créer et configurer un domaine managé Azure Active Directory Domain Services][create-instance].

<!-- INTERNAL LINKS -->
[feedback]: https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160593%3fcategory_id%3d160593
[create-instance]: tutorial-create-instance.md