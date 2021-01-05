---
title: Résoudre les problèmes liés aux recommandations de réservation
description: Cet article vous aide à comprendre et à résoudre les problèmes des recommandations de réservation Azure montrées dans le portail Azure.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: reservations
ms.author: banders
ms.reviewer: yashar
ms.topic: troubleshooting
ms.date: 10/19/2020
ms.openlocfilehash: a3137d779908bf2791ca396068a8c9edf5d56739
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92492235"
---
# <a name="troubleshoot-azure-reservation-recommendations"></a>Résoudre les problèmes liés aux recommandations de réservation

Cet article vous aide à comprendre et à résoudre les problèmes des recommandations de réservation Azure montrées dans le portail Azure. Vous pouvez ne voir aucune recommandation ou voir des recommandations qui ne correspondent pas à vos attentes. Par exemple, vous disposez peut-être déjà d’une instance réservée pour une configuration de machine virtuelle spécifique. Vous pouvez vous attendre à voir une recommandation pour une configuration de machine virtuelle similaire.

## <a name="symptoms"></a>Symptômes

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez à **Réservations**.
2. Sélectionnez **+ Ajouter** , puis sélectionnez un produit.
3. Sous l’onglet **Recommandé** , vous pouvez ne voir aucune recommandation ou voir des recommandations qui ne correspondent pas à vos attentes.

## <a name="cause"></a>Cause :

La liste des produits recommandés est générée sur la base de la quantité d’utilisation que vous avez pour votre étendue (partagée ou unique) par rapport au coût d’une réservation pour le produit. Si le moteur de recommandation détecte des économies, il recommande l’achat d’un produit. Cependant, quand le moteur n’identifie aucune économie, il ne recommande pas de produit.

Quand vous utilisez une ressource avec une configuration donnée, il n’est pas garanti que vous allez économiser de l’argent en achetant une réservation pour cette configuration. Il se peut par exemple que vous en ayez une utilisation sporadique. Dans ce cas, le coût total de la réservation risque de ne pas être plus économique pendant la durée de la période de réservation.

Il est également important de comprendre comment la sélection de l’étendue affecte les recommandations. Quand l’étendue est définie sur **Partagée** , les recommandations de la liste montrent les instances réservées où Azure trouve des économies pour toute l’inscription associée à l’abonnement de facturation. Quand l’étendue est définie sur **Unique** , les recommandations de la liste s’appliquent seulement aux ressources qui s’exécutent dans l’abonnement. Il est possible que certaines tailles de machine virtuelle soient recommandées pour une étendue mais pas pour une autre. Par exemple, vous pouvez avoir une utilisation agrégée de **Standard_B1ls** pour votre inscription qui est suffisamment élevée pour justifier le coût de l’achat d’une réservation dans l’étendue de l’inscription. Cependant, un abonnement donné dans l’inscription peut ne pas être assez utilisé pour justifier le coût de l’achat d’une réservation dans l’étendue. Le changement d’étendue entre **Partagée** et **Unique** peut produire des recommandations différentes.

Azure peut recommander l’achat d’une réservation pour certaines périodes, mais pas pour d’autres, en fonction des économies de coûts identifiées. En particulier, les périodes de trois années ont des remises supérieures à celles d’une année. Il est plus probable qu’Azure trouve des économies sur une période de trois années plutôt que sur une période d’une année.

Si vous voulez comprendre pourquoi Azure recommande une taille et une quantité de ressource spécifiques, sélectionnez **&lt;Quantité&gt; Afficher les détails** pour obtenir une vue détaillée montrant les économies potentielles au fil du temps.

:::image type="content" source="./media/troubleshoot-reservation-recommendation/see-details-link.png" alt-text="Exemple montrant le lien de la recommandation de réservation Afficher les détails" lightbox="./media/troubleshoot-reservation-recommendation/see-details-link.png" :::

## <a name="solution"></a>Solution

Vous pouvez acheter n’importe quelle quantité de réservation pour une période de votre choix. L’achat d’une réservation avec une quantité et une période qui diffèrent de la recommandation entraînera probablement des économies et une utilisation inférieures aux chiffres optimaux de la recommandation.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les recommandations de réservation, consultez [Recommandations relatives à l’achat de réservation](determine-reservation-purchase.md).
