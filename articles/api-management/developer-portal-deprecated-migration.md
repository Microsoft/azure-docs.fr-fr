---
title: Migrer du portail des développeurs hérité vers le nouveau portail des développeurs
titleSuffix: Azure API Management
description: Découvrez comment migrer du portail des développeurs hérité vers le nouveau portail des développeurs dans Gestion des API.
services: api-management
documentationcenter: API Management
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/15/2020
ms.author: apimpm
ms.openlocfilehash: f5105c685de4b3ccdffe69eec8ee8eeb32976c1f
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92325917"
---
# <a name="migrate-to-the-new-developer-portal"></a>Migrer vers le nouveau portail des développeurs

Cet article décrit les étapes à suivre pour migrer du portail hérité déconseillé vers le nouveau portail des développeurs dans Gestion des API.

> [!IMPORTANT]
> Le portail des développeurs hérité est désormais déconseillé et recevra uniquement des correctifs de sécurité. Vous pouvez continuer à l’utiliser normalement jusqu’à sa mise hors service en octobre 2023, lorsqu’il sera supprimé de tous les services de Gestion des API.

![Portail des développeurs Gestion des API](media/api-management-howto-developer-portal/cover.png)

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="improvements-in-new-developer-portal"></a>Améliorations apportées au nouveau portail des développeurs

Le nouveau portail des développeurs résout de nombreuses limitations du portail déconseillé. Il comporte un [éditeur visuel par glisser-déposer pour modifier le contenu](api-management-howto-developer-portal-customize.md) et un panneau dédié aux concepteurs pour définir le style du site web. Les pages, les personnalisations et la configuration sont enregistrées en tant que ressources Azure Resource Manager dans votre service Gestion des API, ce qui vous permet d’[automatiser les déploiements du portail](api-management-howto-developer-portal.md#automate). Enfin, le codebase du portail est open source, [vous pouvez donc l’enrichir avec des fonctionnalités personnalisées](api-management-howto-developer-portal.md#managed-vs-self-hosted).

## <a name="how-to-migrate-to-new-developer-portal"></a>Comment migrer vers le nouveau portail des développeurs

Le nouveau portail des développeurs est incompatible avec le portail déconseillé ; la migration automatisée n’est pas possible. Vous devez recréer manuellement le contenu (pages, texte, fichiers multimédias) et personnaliser l’apparence du nouveau portail. Les étapes précises varient en fonction des personnalisations et de la complexité de votre portail. Pour obtenir de l’aide, consultez le [tutoriel sur le portail des développeurs](api-management-howto-developer-portal-customize.md). La configuration restante, par exemple la liste des API, des produits, des utilisateurs et des fournisseurs d’identité, est automatiquement partagée entre les deux portails.

> [!IMPORTANT]
> Si vous avez déjà lancé le nouveau portail des développeurs, mais que vous n’avez effectué aucune modification, [réinitialisez le contenu par défaut](api-management-howto-developer-portal.md#preview-to-ga) pour le mettre à jour à la version la plus récente.

Lorsque vous migrez depuis le portail déconseillé, gardez à l’esprit les modifications suivantes :

- Si vous exposez votre portail des développeurs à l’aide d’un domaine personnalisé, [attribuez un domaine](configure-custom-domain.md) au nouveau portail des développeurs. Utilisez l’option **Portail des développeurs** dans la liste déroulante du portail Azure.
- [Appliquez une stratégie CORS](api-management-howto-developer-portal.md#cors) sur vos API pour activer la console de test interactive.
- Si vous injectez du code CSS personnalisé pour appliquer un style au portail, vous devez [reproduire le style à l’aide du panneau de conception intégré](api-management-howto-developer-portal-customize.md). L’injection de code CSS n’est pas autorisée dans le nouveau portail.
- Vous pouvez injecter du code JavaScript personnalisé uniquement dans la [version auto-hébergée du nouveau portail](api-management-howto-developer-portal.md#managed-vs-self-hosted).
- Si votre instance Gestion des API se trouve dans un réseau virtuel et est exposée à Internet via Application Gateway, [consultez cet article de la documentation](api-management-howto-integrate-internal-vnet-appgateway.md) pour connaître les étapes précises de configuration. Vous devez :

    - Activer la connectivité au point de terminaison de gestion de Gestion des API.
    - Activer la connectivité au point de terminaison du nouveau portail.
    - Désactiver certaines règles de Web Application Firewall.

- Si vous avez modifié les modèles de notification par e-mail par défaut pour inclure une URL du portail déconseillé qui est explicitement définie, modifiez-les pour qu’ils utilisent le paramètre « URL du portail » ou pointent vers l’URL du nouveau portail. Si les modèles utilisent le paramètre « URL du portail » intégré à la place, aucune modification n’est requise.
- *Problèmes* et *Applications* ne sont pas pris en charge dans le nouveau portail des développeurs.
- L’intégration directe à Facebook, Microsoft, Twitter et Google comme fournisseurs d’identité n’est pas prise en charge dans le nouveau portail des développeurs. Vous pouvez intégrer ces fournisseurs à l’aide d’Azure AD B2C.
- Si vous utilisez la délégation, modifiez l’URL de retour dans vos applications et utilisez le [point de terminaison d’API *Recevoir un jeton d’accès partagé*](/rest/api/apimanagement/2019-12-01/user/getsharedaccesstoken) au lieu du point de terminaison *Générer l’URL SSO* .
- Si vous utilisez Azure AD en tant que fournisseur d’identité :

    - Modifiez l’URL de retour dans votre application pour qu’elle pointe vers le nouveau domaine du portail des développeurs.
    - Modifiez le suffixe `/signin-aad` de l’URL de retour dans votre application en `/signin`.

- Si vous utilisez Azure AD B2C en tant que fournisseur d’identité :

    - Modifiez l’URL de retour dans votre application pour qu’elle pointe vers le nouveau domaine du portail des développeurs.
    - Modifiez le suffixe `/signin-aad` de l’URL de retour dans votre application en `/signin`.
    - Incluez *Prénom* , *Nom de famille* et *ID d’objet de l’utilisateur* dans les revendications d’application.

- Si vous utilisez OAuth 2.0 dans la console de test interactive, modifiez l’URL de retour dans votre application pour qu’elle pointe vers le domaine du nouveau portail des développeurs et modifiez le suffixe :

    - `/docs/services/[serverName]/console/oauth2/authorizationcode/callback` en `/signin-oauth/code/callback/[serverName]` pour le flux d’octroi du code d’autorisation.
    - `/docs/services/[serverName]/console/oauth2/implicit/callback` en `/signin-oauth/implicit/callback` pour le flux d’octroi implicite.
- Si vous utilisez OpenID Connect dans la console de test interactive, modifiez l’URL de retour dans votre application pour qu’elle pointe vers le domaine du nouveau portail des développeurs et modifiez le suffixe :

    - `/docs/services/[serverName]/console/openidconnect/authorizationcode/callback` en `/signin-oauth/code/callback/[serverName]` pour le flux d’octroi du code d’autorisation.
    - `/docs/services/[serverName]/console/openidconnect/implicit/callback` en `/signin-oauth/implicit/callback` pour le flux d’octroi implicite.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le portail des développeurs :

- [Vue d’ensemble du portail des développeurs Gestion des API Azure](api-management-howto-developer-portal.md)
- [Accéder et personnaliser le portail des développeurs](api-management-howto-developer-portal-customize.md)