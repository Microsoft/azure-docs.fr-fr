---
title: Offres privées dans la place de marché commerciale de Microsoft
description: Offres privées dans la Place de marché commerciale de Microsoft pour les éditeurs d’applications et de services.
ms.subservice: partnercenter-marketplace-publisher
ms.service: marketplace
ms.topic: article
author: vikrambmsft
ms.author: vikramb
ms.date: 07/01/2020
ms.openlocfilehash: fdd8c752e8cf610ef3c32e1af91b3ad6454c0813
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95971710"
---
# <a name="private-offers-in-the-microsoft-commercial-marketplace"></a>Offres privées dans la Place de marché commerciale de Microsoft

Les offres privées dans la [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/) permettent aux éditeurs de créer des plans que seuls les clients ciblés peuvent voir. Cet article présente les options et les avantages des offres privées.

## <a name="unlock-enterprise-deals-with-private-offers"></a>Élargissez les possibilités de transactions B2B grâce aux offres privées

Les entreprises clientes sont de plus en plus nombreuses à utiliser des places de marché en ligne pour trouver, tester et acheter des solutions cloud. Grâce aux offres privées, les éditeurs peuvent à présent partager des solutions personnalisées avec les clients ciblés sur la place de marché, en proposant les fonctionnalités exigées par les entreprises :

- Les *tarifs négociés* permettent aux éditeurs d’étendre les remises et les prix hors catalogue par rapport aux offres publiques.
- Grâce aux *conditions générales privées*, ils peuvent personnaliser les conditions générales pour chaque client.
- Les *configurations spécialisées* aident les éditeurs à adapter leur offre Machines virtuelles, Azure Application ou SaaS aux besoins de chaque client. Cette option leur permet également de proposer un accès en préversion aux nouvelles fonctionnalités des produits, avant le lancement général auprès de l’ensemble des clients.

Les offres privées permettent aux éditeurs de tirer parti de l’envergure et de la disponibilité mondiale d’une place de marché publique, avec la flexibilité et le contrôle nécessaires pour négocier et proposer des configurations et des offres personnalisées. Toutes ces fonctionnalités ouvrent la voie à une adoption massive des places de marché cloud dans les entreprises. Ces dernières ont ainsi la liberté de gérer leurs achats/ventes comme elles le souhaitent.

Les offres privées sont maintenant disponibles pour les offres Machines virtuelles, Azure Application (implémentées sous forme de modèles de solution ou d’applications managées) et SaaS.

<!--- Like public offers, private offers can be created and managed via the [Cloud Partner Portal](). Customers can be granted or revoked access to private offers in minutes.
--->

## <a name="creating-private-offers-using-plans"></a>Créer des offres privées avec des plans

À partir d’*offres nouvelles ou existantes avec plans*, les éditeurs peuvent ajouter sans peine des variantes privées en créant de nouveaux plans (anciennement les « références SKU ») et en les marquant comme privés. Chaque offre peut avoir jusqu’à 45 plans privés.

<!--- [Private SKUs]() --->

Les plans privés sont des composants d’une offre que seuls les clients ciblés peuvent voir et acquérir. Ils peuvent réutiliser les images de base et offrir des métadonnées déjà publiées pour un plan public. Cette option permet aux éditeurs de créer plusieurs variantes privées d’une offre publique sans avoir à publier plusieurs versions de la même image de base ou à proposer des métadonnées. Dans le cas des offres Machines virtuelles et Azure Application uniquement, quand un plan privé partage une image de base avec un plan public, les modifications apportées à l’image de base de l’offre se propagent sur tous les plans, publics et privés, qui utilisent cette image de base.

Pour les *nouvelles offres qui comportent uniquement des plans privés*, les éditeurs peuvent créer des offres classiques, puis marquer les plans comme privés. Ces offres ne sont accessibles ni sur la Place de marché commerciale de Microsoft ni sur le [portail Azure](https://azure.microsoft.com/features/azure-portal/) par les clients qui n’y sont pas associés.

Des plans privés sont disponibles pour Azure Global et Azure Government.

>[!NOTE]
>Une offre qui contient uniquement des plans privés ne sera pas visible dans la Place de marché Azure publique, ni dans AppSource.

## <a name="targeting-customers-with-private-offers"></a>Cibler des clients avec des offres privées

Avec les offres privées, qu’elles soient anciennes ou nouvelles, les éditeurs ont la possibilité de cibler des clients à l’aide de leurs identificateurs d’abonnement. Les éditeurs qui utilisent une offre Machine virtuelle ou Azure Application peuvent limiter la disponibilité d’un plan privé à un seul ID d’abonnement Azure, ou charger un fichier CSV pouvant comporter jusqu’à 20 000 ID d’abonnement Azure. Lorsqu’ils utilisent une offre privée d’application SaaS, ils peuvent associer un ID de client pour limiter la disponibilité d’un plan privé, suivant l’approche manuelle ou par chargement CSV.

Une fois qu’une offre a été certifiée et publiée, il suffit de quelques minutes pour mettre à jour ou supprimer des clients du plan avec la fonctionnalité de synchronisation des abonnements privés. Cette capacité permet aux éditeurs de mettre à jour rapidement et facilement la liste des clients auxquels le plan privé est présenté, sans avoir à certifier ou publier de nouveau l’offre.

>[!NOTE]
>Applicabilité d’Azure Gov : Les offres privées (ou, plus précisément, les plans privés) sont disponibles pour le cloud Gov comme toute autre offre. Un plan peut également passer de l’état privé à l’état public, et inversement. Le contrôle du cloud sur lequel l’offre sera disponible est entre les mains du partenaire et peut être configuré via Espace partenaires. 

## <a name="deploying-private-offers"></a>Déployer des offres privées

Les offres privées ne sont accessibles que sur le [Portail Azure](https://azure.microsoft.com/features/azure-portal/) et non sur [Microsoft AppSource](https://appsource.microsoft.com/) ou la [Place de marché Azure](https://azuremarketplace.microsoft.com). Pour en savoir plus sur la publication sur les différentes magasins en ligne des places de marché commerciales, consultez [Déterminer votre option de publication](./determine-your-listing-type.md).

Une fois connectés au portail Azure, les clients sélectionnent l’élément de navigation Place de marché pour accéder à leurs offres privées. Les offres privées peuvent être déployées en ligne de commande et avec des modèles Azure Resource Manager, comme toutes les autres offres.

![[Offres privées]](./media/marketplace-publishers-guide/private-offer.png)

Les offres privées apparaissent également dans les résultats de recherche. Recherchez simplement le badge **Privé**.

>[!Note]
>Les offres privées ne sont pas prises en charge avec les abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).


<!---
## Next steps

To start using private offers, follow the steps in the [Private SKUs and Plans]() guide.
--->
