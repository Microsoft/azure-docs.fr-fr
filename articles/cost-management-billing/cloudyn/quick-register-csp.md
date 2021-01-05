---
title: S’inscrire à Cloudyn à l’aide des informations de partenaire CSP dans Azure
description: Découvrez les détails du processus d’inscription utilisé par les partenaires pour intégrer leurs clients au portail Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 10/23/2020
ms.topic: quickstart
ms.custom: seodec18
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: 95d8a425314ad1c968915085b35ca89aa3f62711
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92543319"
---
# <a name="register-with-the-csp-partner-program-and-view-cost-data"></a>S’inscrire au programme pour les partenaires CSP et afficher les données de coût

En tant que partenaire CSP et utilisateur Cloudyn inscrit, vous pouvez visualiser et analyser vos dépenses cloud dans Cloudyn. [Azure Cost Management est disponible de façon native pour les partenaires directs](../costs/get-started-partners.md) qui ont intégré leurs clients à un Contrat client Microsoft et qui ont acheté un plan Azure.

Votre inscription vous donne accès au portail Cloudyn. Ce guide de démarrage rapide décrit le processus d’inscription nécessaire pour créer un abonnement d’évaluation Cloudyn et vous connecter au portail Cloudyn.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="configure-indirect-csp-access-in-cloudyn"></a>Configurer l’accès CSP indirect dans Cloudyn

Par défaut, l’API Espace partenaires est accessible uniquement aux fournisseurs de solutions Cloud (CSP, Cloud Solution Providers). Toutefois, un fournisseur CSP direct peut configurer l’accès pour ses clients ou partenaires CSP indirects à l’aide de groupes d’entités dans Cloudyn.

Pour activer l’accès des clients ou des partenaires CSP indirects, effectuez les étapes suivantes pour segmenter les données CSP indirectes en utilisant des groupes d’entités Cloudyn. Ensuite, assignez les autorisations utilisateur appropriées pour les groupes d’entités.

1. Créez un groupe d’entités avec les informations fournies dans [Créer des entités](tutorial-user-access.md#create-and-manage-entities).
2. Suivez les étapes fournies dans [Assigning subscriptions to Cost Entities (Affectation d’abonnements à des entités de coût)](https://www.youtube.com/watch?v=d9uTWSdoQYo). Associez le compte du client CSP indirect et ses abonnements Azure à l’entité que vous avez créée.
3. Suivez les étapes fournies dans [Créer un utilisateur avec un accès administrateur](tutorial-user-access.md#create-a-user-with-admin-access) pour créer un compte d’utilisateur avec un accès administrateur. Ensuite, vérifiez que le compte d’utilisateur dispose d’un accès administrateur aux entités spécifiques que vous avez créées pour le compte indirect.

Les partenaires CSP indirects se connectent au portail Cloudyn à l’aide des comptes que vous avez créés pour eux.


[!INCLUDE [cost-management-create-account-view-data](../../../includes/cost-management-create-account-view-data.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé vos informations CSP pour vous inscrire à Cloudyn. Vous vous êtes également connecté au portail Cloudyn et avez affiché des données de coût. Pour en savoir plus sur Cloudyn, suivez le tutoriel sur Cloudyn.

> [!div class="nextstepaction"]
> [Réviser l’utilisation et les coûts](tutorial-review-usage.md)