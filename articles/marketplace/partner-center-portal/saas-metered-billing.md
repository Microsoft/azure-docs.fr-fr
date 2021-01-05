---
title: Facturation à l’usage pour les offres SaaS à l’aide du service de mesure de la consommation de la Place de marché commerciale Microsoft
description: Découvrez les modèles de facturation flexibles pour les offres SaaS à l’aide du service de mesure de la consommation de la place de marché commerciale.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/08/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 9f72d54fda8f66c2fce35f0520b51406aa276bb0
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92892751"
---
# <a name="metered-billing-for-saas-using-the-commercial-marketplace-metering-service"></a>Facturation à l’usage pour SaaS à l’aide du service de mesure de la consommation de la Place de marché commerciale

Avec le service de mesure de la consommation de la Place de marché commerciale, vous pouvez créer des offres de logiciel en tant que service (SaaS) qui sont facturées en fonction d’unités non standard. Avant de publier une offre SaaS sur la Place de marché commerciale, vous définissez les dimensions de la facturation, comme la bande passante, ou le nombre de tickets ou d’e-mails traités.  Les clients paient ensuite en fonction de leur consommation de ces dimensions, votre système informant Microsoft des événements facturables quand ils se produisent, via l’API du service de mesure de la consommation de la Place de marché commerciale.  

## <a name="prerequisites-for-metered-billing"></a>Prérequis pour la facturation à la consommation

Pour qu’une offre SaaS utilise la facturation à la consommation, elle doit :

- Respecter toutes les exigences d’une offre pour une [vente via l’offre de Microsoft](../plan-saas-offer.md#listing-options), comme indiqué dans [Créer une offre SaaS](../create-new-saas-offer.md) dans la Place de marché commerciale.
- S’intégrer aux [API de traitement SaaS](./pc-saas-fulfillment-api-v2.md) pour que les clients provisionnent votre offre et s’y connectent.  
- Être configuré pour le modèle de tarification **forfaitaire** lors de la facturation de votre service aux clients.  Les dimensions sont une extension facultative du modèle de tarification forfaitaire. 

L’offre SaaS peut alors s’intégrer avec les [API du service de mesure de la consommation de la Place de marché commerciale](./marketplace-metering-service-apis.md) pour informer Microsoft des événements facturables.

>[!Note]
>Le service de mesure de la consommation de la Place de marché est disponible seulement pour le modèle de facturation forfaitaire et ne s’applique pas au modèle de facturation par utilisateur.

## <a name="how-metered-billing-fits-in-with-pricing"></a>Comment la facturation à la consommation s’adapte à la tarification

Quand il s’agit de définir l’offre avec ses modèles de tarification, il est important de comprendre la hiérarchie des offres.

- Chaque offre SaaS est configurée pour être vendue via Microsoft ou non.  Une fois l’offre publiée, il n’est plus possible de modifier cette option.
- Chaque offre SaaS, configurée pour être vendue via Microsoft, peut avoir un ou plusieurs plans.  Un utilisateur s’abonne à l’offre SaaS, mais elle est achetée auprès de Microsoft dans le contexte d’un plan.
- Chaque plan a un modèle de tarification associé : **forfaitaire** ou **par utilisateur**. Tous les plans d’une offre doivent être associés au même modèle de tarification. Par exemple, il ne peut pas y avoir une offre incluant des plans pour un modèle de tarification forfaitaire, et une autre suivant un modèle de tarification par utilisateur.
- Dans chaque plan configuré pour un modèle de facturation forfaitaire, au moins un coût récurrent (qui peut être de 0 dollar) est inclus :
    - Coût récurrent **mensuel**  : coût récurrent mensuel forfaitaire qui est prépayé tous les mois quand l’utilisateur achète le plan.
    - Coût récurrent **annuel**  : coût récurrent annuel forfaitaire qui est prépayé tous les ans quand l’utilisateur achète le plan.
- Outre les coûts récurrents, un plan forfaitaire peut également inclure des dimensions personnalisées facultatives utilisées pour facturer les clients pour le dépassement d’utilisation non compris dans le forfait.  Chaque dimension représente une unité facturable que votre service communiquera à Microsoft en utilisant l’[API du service de mesure de la consommation de la Place de marché commerciale](./marketplace-metering-service-apis.md).

## <a name="sample-offer"></a>Exemple d’offre

Par exemple, Contoso est un éditeur avec un service SaaS appelé Contoso Notification Services (CNS). CNS permet à ses clients d’envoyer des notifications par e-mail ou par SMS. Contoso est inscrit en tant qu’éditeur dans l’Espace partenaires pour le programme commercial de la Place de marché afin de publier des offres SaaS destinées aux clients Azure.  Il existe deux plans associés à CNS, présentés ci-dessous :

- Plan De base
    - Envoyer 10 000 e-mails et 1 000 SMS pour 0 USD/mois (tarif mensuel fixe)
    - Au-delà des 10 000 e-mails, payer 1 $ tous les 100 e-mails
    - Au-delà des 1 000 SMS, payer 0,02 $ pour chaque SMS

    [![Tarification du plan De base](./media/saas-basic-pricing.png "Cliquer pour agrandir l’affichage")](./media/saas-basic-pricing.png)

- Plan Premium
    - Envoyer 50 000 e-mails et 10 000 SMS pour 350 USD/mois ou 5 millions d’e-mails et 1 million de SMS pour 3 500 USD par an
    - Au-delà des 50 000 e-mails, payer 0,5 $ tous les 100 e-mails
    - Au-delà des 10 000 SMS, payer 0,01 $ pour chaque SMS

    [![Tarification du plan Premium](./media/saas-premium-pricing.png "Cliquer pour agrandir l’affichage")](./media/saas-premium-pricing.png)

- Plan Entreprise
    - Envoyer un nombre illimité d’e-mails et 50 000 SMS pour 400 USD/mois
    - Au-delà de 50 000 SMS, payer 0,005 USD par SMS

    [![Tarification du plan Entreprise](./media/saas-enterprise-pricing.png "Cliquer pour agrandir l’affichage")](./media/saas-enterprise-pricing.png)

En fonction du plan sélectionné, un client Azure achetant un abonnement à une offre SaaS de CNS sera en mesure d’envoyer la quantité de SMS et d’e-mails incluse par durée d’abonnement (mois ou année spécifiés dans les détails de l’abonnement par DateDébut et DateFin).  Contoso comptabilise l’utilisation jusqu’à la quantité incluse sans envoyer d’événements d’utilisation à Microsoft. Quand les clients consomment plus que la quantité incluse, ils n’ont pas à changer de plan ou à procéder différemment.  Contoso mesure le dépassement au-delà de la quantité incluse et commence à émettre des événements d’utilisation à destination de Microsoft pour facturer le dépassement d’utilisation avec l’[API du service de mesure de la consommation de la Place de marché commerciale](./marketplace-metering-service-apis.md).  Microsoft, à son tour, facture le client pour le dépassement d’utilisation spécifié par l’éditeur dans les dimensions personnalisées. La facturation du dépassement est effectuée lors du prochain cycle de facturation (mensuel, mais peut être trimestriel ou précoce pour certains clients).  Dans le cas d’un forfait mensuel, la facturation du dépassement est effectuée pour chaque mois où un dépassement s’est produit.  Dans le cas d’un forfait annuel, une fois que la quantité de base incluse par année est consommée, toute utilisation supplémentaire émise par le compteur du client est facturée comme dépassement au cours de chaque cycle de facturation (mensuel) jusqu’à la fin de la durée annuelle de l’abonnement.

## <a name="billing-dimensions"></a>Dimensions de facturation

Chaque dimension de facturation définit une unité personnalisée par laquelle l’ISV peut émettre des événements d’utilisation.  Les dimensions de facturation sont également utilisées pour indiquer au client comment il sera facturé pour l’utilisation du logiciel.  Elles sont définies comme suit :

- **ID**  : identificateur immuable référencé lors de l’émission d’événements d’utilisation.
- **Nom d’affichage**  : nom d’affichage associé à la dimension, par exemple, « messages SMS envoyés ».
- **Unité de mesure**  : description de l’unité de facturation, par exemple, « par SMS » ou « par 100 e-mails ».
- **Prix unitaire en USD**  : prix pour une unité de la dimension.  Il peut être égal à 0. 
- **Quantité mensuelle incluse de base**  : quantité de la dimension incluse par mois pour les clients qui paient la redevance mensuelle récurrente ; il doit s’agir d’un entier. Il peut être égal à 0 ou illimité.
- **Quantité annuelle incluse de base**  : quantité de la dimension incluse par année pour les clients qui paient la redevance annuelle récurrente ; il doit s’agir d’un entier. Il peut être égal à 0 ou illimité.

Les dimensions de facturation sont partagées entre tous les plans d’une offre.  Certains attributs s’appliquent à la dimension dans tous les plans, et les autres attributs sont spécifiques à un plan.

Les attributs qui définissent la dimension proprement dite sont partagés entre tous les plans d’une offre.  Avant la publication de l’offre, une modification apportée à ces attributs pour le contexte d’un plan affectera la définition de la dimension dans tous les plans.  Une fois que vous avez publié l’offre, ces attributs ne sont plus modifiables.  Ces attributs sont :

- id
- Nom d’affichage
- Unité de mesure

Les autres attributs d’une dimension sont spécifiques à chaque plan et peuvent avoir des valeurs différentes d’un plan à l’autre.  Avant de publier le plan, vous pouvez modifier ces valeurs. Seul ce plan sera affecté.  Une fois que vous avez publié le plan, ces attributs ne sont plus modifiables.  Ces attributs sont :

- Prix unitaire en USD
- Quantité mensuelle incluse de base  
- Quantité annuelle incluse de base  

Les dimensions ont également deux concepts spéciaux, « Activé » et « Infini » :

- **Activé** indique que ce plan participe à cette dimension.  Si vous créez un plan qui n’envoie pas d’événements d’utilisation basés sur cette dimension, vous pouvez ne pas activer cette option .  En outre, toutes les nouvelles dimensions ajoutées après la publication initiale d’un plan apparaissent comme « Non activées » sur le plan déjà publié.  Une dimension désactivée ne s’affiche dans aucune des listes de dimensions pour un plan visualisé par des clients.
- **Infini** , représenté par le symbole de l’infini « ∞ », indique que ce plan participe à cette dimension, mais n’émet pas l’utilisation pour cette dimension.  Si vous voulez indiquer à vos clients que les fonctionnalités représentées par cette dimension sont incluses dans le plan, mais sans limite d’utilisation.  Une dimension dont l’utilisation est infinie apparaît dans les listes de dimensions d’un plan montrées aux clients, avec une indication établissant qu’elle n’engendrera jamais de coûts pour ce plan.

>[!Note] 
>Les scénarios suivants sont explicitement pris en charge : <br> - Vous pouvez ajouter une nouvelle dimension à un nouveau plan.  La nouvelle dimension ne sera pas activée pour les plans déjà publiés. <br> - Vous pouvez publier un plan **forfaitaire** sans aucune dimension, puis ajouter un nouveau plan et configurer une nouvelle dimension pour ce plan. La nouvelle dimension ne sera pas activée pour les plans déjà publiés.

### <a name="setting-dimension-price-per-unit-per-supported-market"></a>Définition du prix de la dimension par unité par marché pris en charge

À l’instar du tarif forfaitaire, les prix de la dimension de facturation peuvent être définis par pays ou région pris en charge. Vous devez utiliser la fonctionnalité de tarification d’importation et d’exportation de données dans l’Espace partenaires conformément à la description ci-dessous.

1. Définissez les dimensions souhaitées et marquez les marchés pris en charge. 
1. Exportez ces données dans un fichier.
1. Ajoutez les prix appropriés par pays/région et importez le fichier dans l’Espace partenaires.

L’interface utilisateur du compteur change pour indiquer que les prix de la dimension ne sont visibles que dans le fichier.

[![dimensions du service de mesure de la Place de marché commerciale](media/metering-service-dimensions.png "Cliquer pour agrandir l’affichage")](media/metering-service-dimensions.png)

### <a name="private-plan"></a>Plan privé

À l’instar des plans forfaitaires, un plan assorti de dimensions peut être défini en tant que plan privé, accessible uniquement au public défini par le plan.

## <a name="constraints"></a>Contraintes

### <a name="trial-behavior"></a>Comportement des évaluations

La facturation à la consommation avec le service de mesure de la consommation de la Place de marché commerciale n’est pas compatible avec une offre d’évaluation gratuite.  Il n’est pas possible de configurer un plan pour utiliser à la fois la facturation à la consommation et une évaluation gratuite.

### <a name="locking-behavior"></a>Comportement du verrouillage

Une dimension utilisée avec le service de mesure de la consommation de la Place de marché commerciale représentant une indication de la manière dont un client paye pour le service, tous les détails d’une dimension ne sont plus modifiables après que vous l’avez publiée.  Il est important que vos dimensions soient entièrement définies pour un plan avant la publication.

Une fois qu’une offre est publiée avec une dimension, les détails au niveau de l’offre pour cette dimension ne peuvent plus être modifiés :

- id
- Nom d’affichage
- Unité de mesure

Une fois qu’un plan est publié, les détails au niveau du plan ne peuvent plus être modifiés :

- Prix unitaire en USD
- Quantité mensuelle incluse de base
- Quantité annuelle incluse de base
- Indique si la dimension est activée pour le plan

### <a name="upper-limits"></a>Limites supérieures

Le nombre maximal de dimensions pouvant être configurées pour une même offre est de 30 dimensions uniques.

## <a name="get-support"></a>Obtenir de l’aide

Si vous rencontrez l’un des problèmes suivants, vous pouvez ouvrir un ticket de support.

- Problèmes techniques liés à l’API du service de mesure de la consommation de la Place de marché.
- Un problème qui doit être remonté en raison d’une erreur ou d’un bogue de votre côté (par exemple un événement d’utilisation incorrect).
- Tout autre problème lié à la facturation à la consommation.

Pour comprendre les options du serveur de publication et ouvrir un ticket de support auprès de Microsoft, suivez les instructions de la section [Support technique pour le programme Place de marché commerciale dans l’Espace partenaires](../support.md).

## <a name="next-steps"></a>Étapes suivantes

- [API de service de mesure de la Place de marché](./marketplace-metering-service-apis.md)
