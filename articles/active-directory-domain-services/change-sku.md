---
title: Modifier la référence SKU pour Azure AD Domain Services | Microsoft Docs
description: En savoir plus sur le niveau de référence SKU d’un domaine managé Azure Active Directory Domain Services si les besoins de votre entreprise évoluent
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 320bd87aa78d26cee44c48f27365febd1dd426ff
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620288"
---
# <a name="change-the-sku-for-an-existing-azure-active-directory-domain-services-managed-domain"></a>Modifier la référence SKU d'un domaine managé Azure Active Directory Domain Servicess existant

Dans Azure Active Directory Domain Services (Azure AD DS), les performances et fonctionnalités disponibles sont basées sur le type de référence SKU. Ces différences de fonctionnalités incluent la fréquence de sauvegarde ou le nombre maximal d’approbations de forêts sortantes unidirectionnelles.

Vous sélectionnez une référence SKU lorsque vous créez le domaine managé, et vous pouvez augmenter ou diminuer la référence SKU lorsque les besoins de votre entreprise évoluent, après le déploiement du domaine managé. Les besoins de l'entreprise évoluent notamment lorsqu'il lui faut effectuer des sauvegardes plus fréquentes ou créer des approbations de forêts supplémentaires. Pour plus d’informations sur les limites et la tarification des différentes références SKU, consultez les pages [Concepts relatifs aux références SKU Azure AD][concepts-sku] et [Tarification Azure AD DS][pricing].

Cet article vous explique comment modifier la référence SKU d'un domaine managé Azure AD DS à l’aide du portail Azure.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le didacticiel pour [créer et configurer un domaine managé][create-azure-ad-ds-instance].

## <a name="sku-change-limitations"></a>Limitations relatives aux changements de références SKU

Vous pouvez augmenter ou diminuer les références SKU une fois le domaine managé déployé. Toutefois, si vous utilisez une forêt de ressources et que vous avez créé des approbations de forêts sortantes unidirectionnelles entre Azure AD DS et un environnement AD DS local, il existe des limitations en termes de changements de références SKU. Les références SKU *Premium* et *Entreprise* définissent une limite quant au nombre d’approbations que vous pouvez créer. Vous ne pouvez pas opter pour une référence SKU dont la limite maximale est inférieure à celle que vous avez configurée.

Par exemple :

* Si vous avez créé deux approbations de forêt sur la référence SKU *Premium*, vous ne pouvez pas opter pour la référence SKU inférieure *Standard*. La référence SKU *Standard* ne prend pas en charge les approbations de forêts.
* De même, si vous avez créé sept approbations sur la référence SKU *Premium*, vous ne pouvez pas opter pour la référence SKU *Entreprise*. La référence SKU *Entreprise* prend en charge un maximum de cinq approbations.

Pour plus d’informations sur ces limites, consultez [Fonctionnalités et limites des références SKU Azure AD DS][concepts-sku].

## <a name="select-a-new-sku"></a>Sélectionner une nouvelle référence SKU

Pour modifier la référence SKU d'un domaine managé à l'aide du portail Azure, procédez comme suit :

1. Sur le Portail Microsoft Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé dans la liste, par exemple *aaddscontoso.com*.
1. Dans le menu gauche de la page Azure AD DS, sélectionnez **Paramètres > Référence SKU**.

    ![Sélectionner l'option de menu de la référence SKU pour votre domaine managé Azure AD DS dans le portail Azure](media/change-sku/overview-change-sku.png)

1. Dans le menu déroulant, sélectionnez la référence SKU souhaitée pour votre domaine managé. Si vous disposez d’une forêt de ressources, vous ne pouvez pas sélectionner la référence SKU *Standard* car les approbations de forêts sont uniquement disponibles sur la référence SKU *Entreprise* ou supérieure.

    Sélectionnez la référence SKU de votre choix dans le menu déroulant, puis **Enregistrer**.

    ![Sélectionner la référence SKU requise dans le menu déroulant du portail Azure](media/change-sku/change-sku-selection.png)

Le changement de type de référence SKU peut prendre une ou deux minutes.

## <a name="next-steps"></a>Étapes suivantes

Si vous disposez d’une forêt de ressources et souhaitez créer des approbations supplémentaires une fois la référence SKU modifiée, consultez [Créer une approbation de forêt sortante vers un domaine local dans Azure AD DS][create-trust].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[concepts-sku]: administration-concepts.md#azure-ad-ds-skus
[create-trust]: tutorial-create-forest-trust.md

<!-- EXTERNAL LINKS -->
[pricing]: https://azure.microsoft.com/pricing/details/active-directory-ds/
