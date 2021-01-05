---
title: Résoudre les problèmes liés à un type de réservation Azure non disponible
description: Cet article vous aide à comprendre et résoudre les problèmes des instances réservées qui apparaissent comme non disponibles dans le portail Azure.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: reservations
ms.author: banders
ms.reviewer: yashar
ms.topic: troubleshooting
ms.date: 10/27/2020
ms.openlocfilehash: 8575d9d86d8e720122a295cf92fa571ef33d5b4c
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92798188"
---
# <a name="troubleshoot-reservation-type-not-available"></a>Résoudre les problèmes de type de réservation non disponible

Cet article vous aide à comprendre et résoudre les problèmes des instances réservées qui apparaissent comme non disponibles dans le portail Azure.

## <a name="symptoms"></a>Symptômes

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez à **Réservations**.
2. Sélectionnez **+ Ajouter** , puis sélectionnez un produit.
3. Sélectionnez l’onglet **Tous les produits**.
4. Dans la liste des produits, sélectionnez-en un. Vous pouvez voir un des messages suivants :
    - `Product unavailable for the selected subscription or region. Contact support.`  
        :::image type="content" source="./media/troubleshoot-product-not-available/product-unavailable-message.png" alt-text="Exemple montrant le message Produit non disponible pour l’abonnement et la région sélectionnés" lightbox="./media/troubleshoot-product-not-available/product-unavailable-message.png" :::
    - `The selected subscription does not have enough core quota remaining to purchase this product. Request quota increase`  
        :::image type="content" source="./media/troubleshoot-product-not-available/not-enough-core-quota-message.png" alt-text="Exemple montrant le message à propos du quota de cœurs insuffisant" lightbox="./media/troubleshoot-product-not-available/not-enough-core-quota-message.png" :::

## <a name="cause"></a>Cause :

Certaines réservations ne sont pas disponibles à l’achat pour les raisons suivantes :

- Les produits d’instance réservée ne sont pas tous disponibles à l’achat dans chaque région
- Votre abonnement a une restriction de quota

### <a name="cause-1"></a>Cause 1

Les produits d’instance réservée ne sont pas tous disponibles à l’achat. Azure décide d’autoriser ou non certains produits en fonction des coûts associés à la remise offerte aux clients. Dans d’autres cas, Azure n’offre pas certains produits en raison des conditions de capacité dans des centres de donnés spécifiques. De plus, certains groupes de développement de produits Azure peuvent maintenant autoriser certains produits, car ils souhaitent mettre hors service un produit plus ancien.

Dans d’autres cas, Azure place des restrictions de capacité sur différents produits en fonction de la demande sur ces ressources dans certaines régions. Par exemple, une telle restriction peut être mise en place quand la demande pour une certaine taille de machine virtuelle est trop forte dans un centre de données. Dans ce cas, Azure ne peut pas garantir la capacité aux clients qui ont acheté une réservation pour cette taille dans cette région. Enfin, certains produits sont uniques pour différentes raisons. Ces produits sont disponibles seulement pour un petit ensemble de clients sélectionnés.

Comme certaines réservations ne sont pas disponibles à l’achat, pourquoi sont-elles affichées dans le portail Azure ? La réponse est que les utilisateurs créent des demandes de support indiquant qu’ils ne peuvent pas trouver les produits qu’ils souhaitent. L’équipe de développement des réservations Azure a déterminé que le fait de montrer tous les produits avec le message `Product unavailable` entraînait moins de demandes de support que de ne pas les montrer.

### <a name="cause-2"></a>Cause 2

Votre abonnement a une restriction de quota. Les abonnements ont des limites quant au nombre de cœurs de processeur qu’ils peuvent consommer. La limite s’applique à certains produits d’instance réservée, pour la plupart aux machines virtuelles. Azure veut garantir que les personnes qui achètent une instance réservée peuvent l’utiliser. Azure effectue un contrôle simple et rapide dans le portail Azure pour vérifier que vous disposez de suffisamment de cœurs libres dans votre abonnement pour déployer la machine virtuelle et tirer parti de l’achat de la réservation.

La vérification pour vous permettre d’ajouter un produit particulier à votre panier et d’acheter une réservation est simple. Azure évalue le nombre total de cœurs de processeur disponibles pour votre abonnement et vérifie si ce nombre est supérieur au nombre de cœurs pour l’élément sélectionné.

Azure ne vérifie pas le quota pour les instances réservées de l’étendue **Partagée**. L’avantage de l’instance réservée pour l’étendue partagée s’applique à tous les abonnements de l’inscription. Azure ne peut pas déterminer si vous avez suffisamment de cœurs pour l’ensemble de vos abonnements pour déployer la ressource. Quel que soit le quota, Azure vous permet toujours de sélectionner une taille de machine virtuelle quand l’étendue sélectionnée est « partagée ».

De plus, Azure n’effectue pas de vérification du quota pour les achats **recommandés**. Les recommandations sont basées sur l’utilisation active. Azure suppose que vous disposez de suffisamment de cœurs pour utiliser une taille de machine virtuelle spécifique, car vous avez déjà généré l’utilisation nécessaire pour créer la recommandation.

## <a name="solution"></a>Solution

En fonction du message d’erreur que vous avez reçu, utilisez une des solutions suivantes pour votre problème.

### <a name="solution-1"></a>Solution 1

Si vous voyez un message _Produit non disponible_ , sélectionnez le lien **Contacter le support technique** dans le message d’erreur pour demander l’ajout d’une exception pour votre abonnement. Les exceptions ne sont pas toujours accordées.

### <a name="solution-2"></a>Solution 2

Si vous voyez un message _Quota de cœurs insuffisant_ , vous pouvez changer l’étendue en **Partagée**. Une fois que vous avez acheté la réservation, vous pouvez changer l’étendue de la réservation de **Partagée** en **Unique**.

Vous pouvez aussi sélectionner le lien **Demander une augmentation de quota** dans le message d’erreur pour demander un quota de cœurs de processeur supplémentaire pour votre abonnement.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les options d’étendue des réservations, consultez [Étendue des réservations](prepare-buy-reservation.md#scope-reservations).