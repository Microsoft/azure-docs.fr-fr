---
title: Migrer un profil Azure CDN depuis Verizon standard vers Verizon Premium
description: Découvrez en détail la migration d’un profil depuis Verizon standard vers Verizon Premium.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/21/2018
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: e58c1177fb3c4d241a3efe1759b3900abbd04ca1
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778384"
---
# <a name="migrate-an-azure-cdn-profile-from-standard-verizon-to-premium-verizon"></a>Migrer un profil Azure CDN depuis Verizon standard vers Verizon Premium

Quand vous créez un profil Azure Content Delivery Network (CDN) pour gérer vos points de terminaison, Azure CDN met à votre disposition quatre produits différents. Pour plus d’informations sur les différents produits et leurs fonctionnalités, consultez [Comparer les caractéristiques du produit CDN Azure](cdn-features.md).

Si vous avez créé un profil **Azure CDN standard de Verizon** et que vous l’utilisez pour gérer vos points de terminaison CDN, vous pouvez le mettre à niveau vers un profil **Azure CDN Premium de Verizon** . Quand vous effectuez la mise à niveau, vos points de terminaison CDN et toutes vos données sont conservés. 

> [!IMPORTANT]
> Une fois que la mise à niveau est effectuée avec un profil **Azure CDN Premium de Verizon** , vous ne pouvez plus reconvertir ce dernier en profil **Azure CDN standard de Verizon** .
> 

Pour mettre à niveau un profil **Azure CDN standard de Verizon** , contactez le [Support Microsoft](https://azure.microsoft.com/support/options/).

## <a name="profile-comparison"></a>Comparaison des profils
Les profils **Azure CDN Premium de Verizon** présentent les caractéristiques suivantes par rapport aux profils **Azure CDN standard de Verizon** :
- Pour certaines fonctionnalités Azure CDN comme la [compression](cdn-improve-performance.md), la [mise en cache des règles](cdn-caching-rules.md) et le [géofiltrage](cdn-restrict-access-by-country.md), vous ne pouvez pas utiliser l’interface d’Azure CDN ; vous devez utiliser le portail de Verizon par le biais du bouton **Gérer** .
- API : à la différence de Verizon standard, vous ne pouvez pas utiliser l’API pour contrôler les fonctionnalités qui sont accessibles à partir du portail Verizon Premium. Toutefois, vous pouvez utiliser l’API pour contrôler d’autres fonctionnalités courantes, telles que la création/suppression d’un point de terminaison, le vidage/le chargement de ressources mises en cache et l’activation/la désactivation d’un domaine personnalisé.
- Prix : Verizon Premium a une structure tarifaire pour les transferts de données différente de celle de Verizon standard. Pour plus d’informations, consultez [Tarifs Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

Les profils **Azure CDN Premium de Verizon** ont les fonctionnalités supplémentaires suivantes :
- [Authentification par jeton](cdn-token-auth.md) : permet aux utilisateurs d’obtenir et d’utiliser un jeton pour extraire des ressources sécurisées.
- [Moteur de règles](./cdn-verizon-premium-rules-engine.md) : vous permet de personnaliser la gestion des requêtes HTTP.
- Outils d’analytique avancée :
   - [Analytique HTTP Détaillée](cdn-advanced-http-reports.md)
   - [Analytique des performances de périmètre](cdn-edge-performance.md)
   - [Analytique en temps réel](cdn-real-time-alerts.md)


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur le moteur de règles, consultez [Référence du moteur de règles Azure CDN](./cdn-verizon-premium-rules-engine-reference.md).