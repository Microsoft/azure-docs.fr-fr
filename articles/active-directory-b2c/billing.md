---
title: Modèle de facturation pour Azure Active Directory B2C
description: Découvrez le modèle de facturation par utilisateurs actifs mensuels (MAU) d’Azure AD B2C, la façon de lier un locataire Azure AD B2C à un abonnement Azure et la façon de sélectionner les tarifs du niveau Premium appropriés.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: reference
ms.workload: identity
ms.date: 09/01/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: edff5354d0565bd32cd0332b4aa0f215c2980d73
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94949783"
---
# <a name="billing-model-for-azure-active-directory-b2c"></a>Modèle de facturation pour Azure Active Directory B2C

La tarification Azure Active Directory B2C (Azure AD B2C) est basée sur les utilisateurs actifs mensuels (MAU), qui représentent le nombre d’utilisateurs uniques avec une activité d’authentification au cours d’un mois civil. Ce modèle de facturation s’applique à la fois aux locataires Azure AD B2C et à la [collaboration d’utilisateur invité (B2B) Azure AD](../active-directory/external-identities/external-identities-pricing.md). La facturation MAU contribue à réduire les coûts en offrant un niveau gratuit et une tarification flexible et prévisible. Dans cet article, découvrez la facturation MAU, la liaison de vos locataires Azure AD B2C à un abonnement et le changement de niveau tarifaire.

> [!IMPORTANT]
> Cet article ne contient pas d’informations sur les tarifs. Pour obtenir les informations les plus récentes sur les prix et la facturation, consultez [Tarification Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

## <a name="what-do-i-need-to-do"></a>Que dois-je faire ?

Pour tirer parti de la facturation MAU, votre locataire Azure AD B2C doit être lié à un abonnement Azure. Vous devrez peut-être également faire basculer votre locataire Azure AD B2C vers un autre niveau tarifaire si vous souhaitez utiliser des fonctionnalités Azure AD B2C Premium P2, telles que l’accès conditionnel basé sur les risques.

|Si votre locataire est :  |Vous devez :  |
|---------|---------|
| Un locataire Azure AD B2C déjà facturé par utilisateur actif mensuel     | Ne rien faire. Lorsque les utilisateurs s’authentifient auprès de votre locataire Azure AD B2C, vous êtes automatiquement facturé à l’aide du modèle de facturation MAU.        |
| Un locataire Azure AD B2C encore non lié à un abonnement     |  [Lier votre locataire Azure AD B2C à un abonnement](#link-an-azure-ad-b2c-tenant-to-a-subscription) pour activer la facturation MAU.     |
| Un locataire Azure AD B2C qui a été lié à un abonnement avant le 1er novembre 2019    | [Basculer vers la facturation MAU (recommandé)](#switch-to-mau-billing-pre-november-2019-azure-ad-b2c-tenants) ou rester sur le modèle de facturation par authentification.     |
| Un locataire Azure AD B2C et vous souhaitez utiliser des fonctionnalités Premium (comme l’accès conditionnel basé sur les risques)    | [Passer à un niveau tarifaire Azure AD](#change-your-azure-ad-pricing-tier) qui prend en charge les fonctionnalités que vous souhaitez utiliser.        |
|  |  |

## <a name="about-the-monthly-active-users-mau-billing-model"></a>À propos du modèle de facturation par utilisateurs actifs mensuels (MAU)

La facturation MAU est entrée en vigueur pour les locataires Azure AD B2C le **1er novembre 2019**. Tout locataire Azure AD B2C que vous avez créé et lié à un abonnement à compter de cette date a été facturé par MAU. Si vous avez un locataire Azure AD B2C qui n’a pas été lié à un abonnement, liez-le maintenant. Si vous avez un locataire Azure AD B2C existant qui a été lié à un abonnement avant le 1er novembre 2019, nous vous recommandons d’effectuer une mise à niveau vers le modèle de facturation par utilisateurs actifs mensuels (MAU), ou vous pouvez conserver le modèle de facturation par authentification.
  
Votre locataire Azure AD B2C doit également être lié au niveau tarifaire Azure approprié en fonction des fonctionnalités que vous souhaitez utiliser. Les fonctionnalités Premium exigent une [tarification Premium P1 ou P2](https://azure.microsoft.com/pricing/details/active-directory-b2c/) d’Azure AD B2C. Vous devrez peut-être mettre à niveau votre niveau tarifaire pour utiliser de nouvelles fonctionnalités. Par exemple, l’accès conditionnel : vous devrez sélectionner le niveau tarifaire Premium P2 d’Azure AD B2C pour votre locataire.

## <a name="link-an-azure-ad-b2c-tenant-to-a-subscription"></a>Lier un locataire Azure AD B2C à un abonnement

Les frais d’utilisation pour Azure Active Directory B2C (Azure AD B2C) sont facturés à un abonnement Azure. Vous devez lier explicitement un locataire Azure AD B2C à un abonnement Azure en créant une *ressource* Azure AD B2C au sein de l’abonnement Azure cible. Plusieurs ressources Azure AD B2C peuvent être créées dans un seul abonnement Azure, ainsi que d’autres ressources Azure, comme des machines virtuelles, des comptes de stockage et des applications logiques. Vous pouvez voir toutes les ressources au sein de l’abonnement en accédant au locataire Azure Active Directory (Azure AD) auquel l’abonnement est associé.

Un abonnement lié à un locataire Azure AD B2C peut être utilisé pour la facturation de l’utilisation d’Azure AD B2C ou d’autres ressources Azure, y compris des ressources Azure AD B2C supplémentaires. Il ne peut pas être utilisé pour ajouter d’autres services basés sur une licence Azure ou des licences Office 365 au sein du locataire Azure AD B2C.

### <a name="prerequisites"></a>Prérequis

* [Abonnement Azure](https://azure.microsoft.com/free/)
* [Locataire Azure AD B2C](tutorial-create-tenant.md) que vous souhaitez lier à un abonnement
  * Vous devez être un administrateur du locataire
  * Le locataire ne doit pas déjà être lié à un abonnement

### <a name="create-the-link"></a>Créer le lien

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient l’abonnement Azure que vous souhaitez utiliser (et *non* votre locataire Azure AD B2C).
3. Sélectionnez **Créer une ressource**, entrez `Active Directory B2C` dans le champ **Rechercher sur la Place de marché**, puis sélectionnez **Azure Active Directory B2C**.
4. Sélectionnez **Create** (Créer).
5. Sélectionnez **Lier un locataire Azure AD B2C existant à mon abonnement Azure**.
6. Sélectionnez un **locataire Azure AD B2C** dans la liste déroulante. Seuls les locataires dont vous êtes administrateur général et qui ne sont pas déjà liés à un abonnement sont affichés. Le **nom de la ressource Azure AD B2C** est rempli avec le nom de domaine du locataire Azure AD B2C que vous sélectionnez.
7. Sélectionnez un **abonnement** Azure actif dont vous êtes administrateur.
8. Sous **Groupe de ressources**, sélectionnez **Créer**, puis spécifiez **l’emplacement du groupe de ressources**. Les paramètres de groupe de ressources n’ont ici aucun impact sur l’emplacement, les performances ou l’état de facturation du locataire Azure AD B2C.
9. Sélectionnez **Create** (Créer).

    ![Page de création de la ressource Azure AD B2C dans le portail Azure](./media/billing/portal-01-create-b2c-resource-page.png)

Une fois ces étapes effectuées pour un locataire Azure AD B2C, votre abonnement Azure est facturé sur la base des détails de votre contrat Azure Direct ou Enterprise, le cas échéant.

## <a name="change-your-azure-ad-pricing-tier"></a>Changer votre niveau tarifaire Azure AD

Un locataire doit être lié au niveau tarifaire Azure approprié en fonction des fonctionnalités que vous souhaitez utiliser avec votre locataire Azure AD B2C. Les fonctionnalités Premium requièrent Azure AD B2C Premium P1 ou P2, comme cela est décrit dans la [tarification Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/). Dans certains cas, vous devrez mettre à niveau votre niveau tarifaire pour utiliser de nouvelles fonctionnalités. Par exemple, si vous souhaitez utiliser la protection d’identité, l’accès conditionnel basé sur les risques et toutes les futures fonctionnalités Premium P2 avec Azure AD B2C, vous devez sélectionner le niveau tarifaire Azure AD B2C Premium P2 pour votre locataire.

Pour changer votre niveau tarifaire, effectuez la procédure suivante.

1. Connectez-vous au portail Azure.

2. Sélectionnez le filtre **Répertoire + abonnement** dans le menu supérieur, puis sélectionnez le répertoire qui contient l’abonnement Azure auquel votre locataire Azure B2C est lié (*ne sélectionnez pas* le locataire Azure AD B2C lui-même).

3. Dans la zone de recherche en haut du portail, entrez le nom de votre locataire Azure AD B2C. Ensuite, sélectionnez le locataire dans les résultats de la recherche, sous **Ressources**.

4. Dans la page **Vue d’ensemble** de la ressource, sous **Niveau tarifaire**, sélectionnez **modifier**.

   ![Changer le niveau tarifaire](media/billing/change-pricing-tier.png)
 
5. Sélectionnez le niveau tarifaire qui comprend les fonctionnalités que vous souhaitez activer.

   ![Sélection du niveau tarifaire](media/billing/select-tier.png)

## <a name="switch-to-mau-billing-pre-november-2019-azure-ad-b2c-tenants"></a>Basculer vers la facturation MAU (locataires Azure AD B2C antérieurs à novembre 2019)

Si vous avez lié votre locataire Azure AD B2C à un abonnement avant le **1er novembre 2019**, le modèle de facturation par authentification précédent est utilisé. Nous vous recommandons d’effectuer la mise à niveau vers le modèle de facturation par utilisateurs actifs mensuels (MAU). Les options de facturation sont configurées dans votre ressource Azure AD B2C.

Le passage à la facturation des utilisateurs actifs mensuels (MAU) **n’est pas réversible**. Une fois que vous avez converti une ressource Azure AD B2C au modèle de facturation MAU, vous ne pouvez pas faire revenir cette ressource au modèle de facturation par authentification.

Voici comment effectuer le basculement vers la facturation MAU pour une ressource Azure AD B2C existante :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant que propriétaire de l’abonnement avec un accès administratif à la ressource Azure AD B2C.

2. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire Azure AD B2C que vous voulez mettre à niveau avec la facturation MAU.<br/>

    ![Filtre de répertoire et d’abonnement dans le portail Azure](./media/billing/portal-mau-01-select-b2c-directory.png)

3. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.

4. Sur la page **Vue d’ensemble** du locataire Azure AD B2C, sélectionnez le lien sous **Nom de la ressource**. Vous êtes dirigé vers la ressource Azure AD B2C dans votre locataire Azure AD.<br/>

    ![Lien de ressource Azure AD B2C mis en surbrillance dans le portail Azure](./media/billing/portal-mau-02-b2c-resource-link.png)

5. Dans la page **Vue d’ensemble** de la ressource Azure AD B2C, sous **Unités facturables**, sélectionnez le lien **Par authentification (changer en MAU)** .<br/>

    ![Lien Changer en MAU mis en surbrillance dans le portail Azure](./media/billing/portal-mau-03-change-to-mau-link.png)

6. Sélectionnez **Confirmer** pour terminer la mise à niveau vers la facturation MAU.<br/>

    ![Boîte de dialogue de confirmation de facturation MAU dans le portail Azure](./media/billing/portal-mau-04-confirm-change-to-mau.png)


### <a name="what-to-expect-when-you-transition-to-mau-billing-from-per-authentication-billing"></a>À quoi s’attendre quand vous passez à la facturation MAU à partir de la facturation par authentification

Le compteur MAU est activé dès que vous, le propriétaire de l’abonnement ou de la ressource, confirmez la modification. Votre facture mensuelle reflète les unités d’authentification facturées jusqu’à la modification et les nouvelles unités MAU à compter de la modification.

Les utilisateurs ne sont pas comptés deux fois pendant le mois de transition. Les utilisateurs actifs uniques qui s’authentifient avant la modification sont facturés par authentification dans un mois civil. Ces mêmes utilisateurs ne sont pas inclus dans le calcul MAU pour le reste du cycle de facturation de l’abonnement. Par exemple :

* Le locataire Contoso B2C a 1 000 utilisateurs. 250 utilisateurs sont actifs dans un mois donné. L’administrateur de l’abonnement passe de la facturation par authentification à celle par utilisateurs actifs mensuels (MAU) le 10 du mois.
* La facturation du 1er au 10 est facturée à l’aide du modèle par authentification.
  * Si 100 utilisateurs se connectent au cours de cette période (du 1er au 10), ces utilisateurs sont marqués comme *payants pour le mois*.
* La facturation à compter du 10 (date effective de la transition) est facturée au tarif MAU.
  * Si 150 utilisateurs supplémentaires se connectent au cours de cette période (du 10 au 30), seuls 150 utilisateurs supplémentaires sont facturés.
  * L’activité continue des 100 premiers utilisateurs n’affecte pas la facturation pour le reste du mois civil.

Pendant la période de facturation de la transition, le propriétaire de l’abonnement verra probablement des entrées pour les deux méthodes (par authentification et par MAU) s’afficher dans le relevé de facturation de l’abonnement Azure :

* Une entrée pour l’utilisation jusqu’à la date/heure de la modification qui reflète la valeur par authentification.
* Une entrée pour l’utilisation après la modification qui reflète les utilisateurs actifs mensuels (MAU).

Pour obtenir les informations les plus récentes sur les prix et la facturation de l’utilisation d’Azure AD B2C, consultez [Tarification Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

## <a name="manage-your-azure-ad-b2c-tenant-resources"></a>Gérer vos ressources de locataire Azure AD B2C

Une fois que vous avez créé une ressource Azure AD B2C dans l’abonnement Azure, une nouvelle ressource du type « Locataire B2C » doit apparaître aux côtés de vos autres ressources Azure.

Vous pouvez utiliser cette ressource pour :

* Accéder à l’abonnement pour afficher les informations de facturation
* Obtient l’ID de locataire du locataire Azure AD B2C au format GUID
* accéder à votre locataire Azure AD B2C ;
* envoyer une demande de support ;
* déplacer votre ressource de locataire Azure AD B2C vers un autre abonnement Azure ou groupe de ressources

### <a name="regional-restrictions"></a>Restrictions régionales

Si vous avez établi des restrictions régionales pour la création de ressources Azure dans votre abonnement, cette restriction peut vous empêcher de créer la ressource Azure AD B2C.

Pour atténuer ce problème, allégez vos restrictions régionales.

## <a name="azure-cloud-solution-providers-csp-subscriptions"></a>Abonnements aux fournisseurs de solutions Cloud Azure

Les abonnements aux fournisseurs de solutions Cloud Azure sont pris en charge dans Azure AD B2C. La fonctionnalité est disponible à l’aide des API ou du portail Azure pour Azure AD B2C et pour toutes les ressources Azure. Les administrateurs des abonnements aux fournisseurs de solutions cloud peuvent lier, déplacer et supprimer des relations avec Azure AD B2C comme pour toutes les ressources Azure.

La gestion d’Azure AD B2C à l’aide du contrôle d’accès en fonction du rôle n’est pas affectée par l’association entre le locataire Azure AD B2C et un abonnement à un fournisseur de solutions cloud Azure. Le contrôle d’accès en fonction du rôle est obtenu à l’aide de rôles basés sur le locataire, et non de rôles basés sur l’abonnement.

## <a name="change-the-azure-ad-b2c-tenant-billing-subscription"></a>Changer l’abonnement de facturation du locataire Azure AD B2C

### <a name="move-using-azure-resource-manager"></a>Déplacer à l’aide d’Azure Resource Manager

Les locataires Azure AD B2C peuvent être déplacés vers un autre abonnement à l’aide d’Azure Resource Manager si les abonnements source et de destination existent au sein du même locataire Azure Active Directory.

Pour savoir comment déplacer des ressources Azure telles que votre locataire Azure AD B2C vers un autre abonnement, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).

Avant de commencer le déplacement, veillez à lire la totalité de l’article pour bien comprendre les limitations et les exigences liées à un déplacement de ce type. En plus des instructions pour déplacer les ressources, il inclut des informations critiques, comme une check-list à consulter avant le déplacement et la validation de l’opération de déplacement.

### <a name="move-by-unlinking-and-relinking"></a>Déplacer par suppression et réédition des liens

Si les abonnements source et de destination sont associés à différents locataires Azure Active Directory, vous ne pouvez pas effectuer le déplacement via Azure Resource Manager comme expliqué ci-dessus. Toutefois, vous pouvez toujours obtenir le même résultat en dissociant le locataire Azure AD B2C de l’abonnement source et en rééditant les liens à l’abonnement de destination. Cette méthode est sécurisée car le seul objet que vous supprimez est le *lien de facturation*, et non le locataire Azure AD B2C lui-même. Aucun des utilisateurs, applications, flux d’utilisateurs, etc., n’est affecté.

1. Dans le répertoire Azure AD B2C lui-même, [invitez un utilisateur invité](user-overview.md#guest-user) à partir du locataire Azure AD de destination (celui auquel l’abonnement Azure de destination est lié) et veillez à ce que cet utilisateur dispose du rôle **Administrateur général** dans Azure AD B2C.
1. Accédez à la *ressource Azure* représentant Azure AD B2C dans votre abonnement Azure source, comme expliqué dans la section [Gérer vos ressources de locataire Azure AD B2C](#manage-your-azure-ad-b2c-tenant-resources) ci-dessus. Ne basculez pas vers le locataire Azure AD B2C réel.
1. Sélectionnez le bouton **Supprimer** dans la page **Vue d’ensemble**. Cela *ne supprime pas* les utilisateurs ou les applications du locataire Azure AD B2C associé. Cela supprime simplement le lien de facturation de l’abonnement source.
1. Connectez-vous au portail Azure avec le compte d’utilisateur qui a été ajouté comme administrateur dans Azure AD B2C, à l’étape 1. Ensuite, accédez à l’abonnement Azure de destination, qui est lié au locataire Azure Active Directory de destination. 
1. Rétablissez le lien de facturation dans l’abonnement de destination en suivant la procédure [Créer le lien](#create-the-link) ci-dessus.
1. Votre ressource Azure AD B2C a maintenant été déplacée vers l’abonnement Azure de destination (lié à Azure Active Directory cible) et sera désormais facturée via cet abonnement.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir les informations les plus récentes sur la tarification, consultez les [Tarification Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).