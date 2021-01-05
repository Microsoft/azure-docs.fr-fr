---
title: Facturation basée sur des mesures pour les applications managées à l’aide du service de mesure de la Place de marché | Place de marché Azure
description: Cette documentation est un guide pour les éditeurs de logiciels indépendants publiant des applications Azure avec des modèles de facturation flexibles.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 04/22/2020
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: d015cec30e516541b50c2acfac38fad898965e1b
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436349"
---
# <a name="managed-application-metered-billing"></a>Facturation des applications managées basée sur des mesures 

Le service de mesure de la Place de marché vous permet de créer des plans d’applications managées pour les offres Azure Application facturées sur la base d’unités non standard. Avant de publier cette offre, vous définissez les dimensions de la facturation, comme la bande passante, ou le nombre de tickets ou d’e-mails traités. Les clients paient ensuite en fonction de leur utilisation de ces dimensions.  Votre système utilise l’API du service de mesure de la Place de marché pour informer Microsoft des événements facturables à mesure qu’ils se produisent.

## <a name="prerequisites-for-metered-billing"></a>Prérequis pour la facturation à la consommation

Pour qu’un plan d’application managée utilise la facturation basée sur des mesures, il doit remplir les critères suivants :

* Respecter toutes les exigences d’offre décrites dans [Créer une offre d’application Azure](../create-new-azure-apps-offer.md).
* Fixer une **Tarification** pour facturer aux clients le coût mensuel de votre service. Le prix peut être égal à zéro si vous ne souhaitez pas facturer des frais fixes mais plutôt vous appuyer entièrement sur la facturation basée sur des mesures.
* Définir des **dimensions de facturation** pour les événements mesurés pour lesquels le client paiera en plus du tarif forfaitaire.
* S’intégrer aux [API du service de mesure de la consommation de la Place de marché](./marketplace-metering-service-apis.md) pour informer Microsoft des événements facturables.

## <a name="how-metered-billing-fits-in-with-pricing"></a>Comment la facturation à la consommation s’adapte à la tarification

Quand il s’agit de définir l’offre avec ses modèles de tarification, il est important de comprendre la hiérarchie des offres.

* Chaque offre Azure Application peut avoir un modèle de solution ou des plans d’application managée.
* La facturation basée sur des mesures est implémentée uniquement avec des plans d’application managées.
* Un modèle de tarification est associé à chaque plan d’application managée. 
* Le modèle de tarification entraîne des frais mensuels récurrents qui peuvent être fixés à 0.
* En plus de frais récurrents, le plan peut inclure des dimensions facultatives utilisées pour facturer les clients pour l’utilisation non comprise dans le taux forfaitaire. Chaque dimension représente une unité facturable que votre service communiquera à Microsoft en utilisant l’[API du service de mesure de la consommation de la Place de marché](marketplace-metering-service-apis.md).

## <a name="sample-offer"></a>Exemple d’offre

Par exemple, contoso est un serveur de publication disposant d’un service d’applications managées nommé Contoso Analytics (CoA). Le service CoA permet aux clients d’analyser une grande quantité de données à des fins de création de rapports et d’entreposage de données. Contoso est inscrit en tant qu’éditeur dans l’Espace partenaires pour le programme commercial de la Place de marché afin de publier des offres destinées aux clients Azure. Deux plans sont associés au service CoA, à savoir :

* Plan de base
    * Analyser 100 Go et générer 100 rapports pour 0 USD/mois
    * Au-delà de 100 Go, payer 10 USD par Go
    * Au-delà de 100 rapports, payez 1 USD par rapport
* Plan Premium
    * Analyser 1000 Go et générer 1000 rapports pour 350 USD/mois
    * Au-delà de 1000 Go, payer 100 USD par To
    * Au-delà de 1000 rapports, payer 0,5 USD par rapport

Un client Azure s’abonnant au service CoA peut analyser et générer des rapports par mois en fonction du plan sélectionné. Contoso mesure l’utilisation jusqu’à la quantité incluse sans envoyer d’événements d’utilisation à Microsoft. Quand les clients consomment plus que la quantité incluse, ils n’ont pas à changer de plan ou à procéder différemment. Contoso mesure le dépassement au-delà de la quantité incluse et commence à émettre des événements d’utilisation à destination de Microsoft pour l’utilisation supplémentaire avec l’[API du service de mesure de la consommation de la Place de marché](./marketplace-metering-service-apis.md). Microsoft facture à son tour le client pour l’utilisation supplémentaire comme spécifié par l’éditeur.

## <a name="billing-dimensions"></a>Dimensions de facturation

Les dimensions de facturation sont utilisées pour indiquer au client comment il sera facturé pour l’utilisation du logiciel.  Ces dimensions sont également utilisées pour signaler des événements d’utilisation à Microsoft. Elles sont définies comme suit :

* **Identificateur de dimension** : identificateur immuable référencé lors de l’émission d’événements d’utilisation.
* **Nom de la dimension** : nom d’affichage associé à la dimension, par exemple, « messages SMS envoyés ».
* **Unité de mesure** : description de l’unité de facturation, par exemple, « par SMS » ou « par 100 e-mails ».
* **Prix unitaire** : prix pour une unité de la dimension.
* **Quantité incluse pour l’échéance mensuelle** : quantité de la dimension incluse par mois pour les clients qui paient les coûts mensuels récurrents ; il doit s’agir d’un entier.

Les dimensions de facturation sont partagées entre tous les plans d’une offre. Certains attributs s’appliquent à la dimension dans tous les plans, et les autres attributs sont spécifiques à un plan.

Les attributs qui définissent la dimension proprement dite sont partagés entre tous les plans d’une offre. Avant la publication de l’offre, une modification apportée à ces attributs pour le contexte d’un plan affectera la définition de la dimension dans tous les plans. Une fois que vous avez publié l’offre, ces attributs ne sont plus modifiables. Les attributs sont les suivants :

* Identificateur
* Nom
* Unité de mesure

Les autres attributs d’une dimension sont spécifiques à chaque plan et peuvent avoir des valeurs différentes d’un plan à l’autre.  Avant de publier le plan, vous pouvez modifier ces valeurs. Seul ce plan sera affecté. Une fois que vous avez publié le plan, ces attributs ne sont plus modifiables. Les attributs sont les suivants :

* Prix unitaire
* Quantité incluse pour les clients avec facture mensuelle 
* Quantité incluse pour les clients avec facture annuelle 

Les dimensions ont également deux concepts spéciaux, « Activé » et « Infini » :

* **Activé** indique que ce plan participe à cette dimension.  Vous pouvez ne pas activer cette option si vous créez un plan qui n’envoie pas d’événements d’utilisation basés sur cette dimension. En outre, toutes les nouvelles dimensions ajoutées après la publication initiale d’un plan apparaissent comme « Non activées » sur le plan déjà publié.  Une dimension désactivée ne s’affiche dans aucune des listes de dimensions pour un plan visualisé par des clients.
* La dimension **Infini** représentée par le symbole de l’infini « ∞ », indique que ce plan participe de cette dimension, mais ne mesure pas d’utilisation par rapport à celle-ci. Si vous voulez indiquer à vos clients que les fonctionnalités représentées par cette dimension sont incluses dans le plan, mais sans limite d’utilisation.  Une dimension associée à une utilisation infinie s’affiche dans des listes de dimensions pour un plan visible par les clients.  Ce plan ne sera jamais facturé.

>[!Note] 
>Les scénarios suivants sont explicitement pris en charge :  <br> - Vous pouvez ajouter une nouvelle dimension à un nouveau plan.  La nouvelle dimension ne sera pas activée pour les plans déjà publiés. <br> - Vous pouvez publier un plan assorti d’un forfait mensuel sans aucune dimension, puis ajouter un nouveau plan et configurer une nouvelle dimension pour celui-ci. La nouvelle dimension ne sera pas activée pour les plans déjà publiés.

## <a name="constraints"></a>Contraintes

### <a name="locking-behavior"></a>Comportement du verrouillage

Une dimension utilisée avec le service de mesure de la Place de marché apporte une compréhension de la manière dont un client paiera pour le service.  Les détails d’une dimension ne sont plus modifiables une fois l’offre publiée.  Il est important que vos dimensions soient entièrement définies avant la publication de votre offre.

Une fois qu’une offre est publiée avec une dimension, les détails au niveau de l’offre pour cette dimension ne peuvent plus être modifiés :

* Identificateur
* Nom
* Unité de mesure

Une fois qu’un plan est publié, les détails au niveau du plan ne peuvent plus être modifiés :

* Prix unitaire
* Quantité incluse pour l’échéance mensuelle
* Indique si la dimension est activée pour le plan

>[!Note]
>La facturation basée sur des mesures du service de mesure de la Place de marché n’est pas encore prise en charge dans le Cloud Azure Government.

### <a name="upper-limits"></a>Limites supérieures

Le nombre maximal de dimensions pouvant être configurées pour une même offre est de 30 dimensions uniques.

## <a name="get-support"></a>Obtenir de l’aide

Si vous rencontrez l’un des problèmes suivants, vous pouvez ouvrir un ticket de support.

* Problèmes techniques liés à l’API du service de mesure de la consommation de la Place de marché.
* Un problème qui doit être remonté en raison d’une erreur ou d’un bogue de votre côté (par exemple un événement d’utilisation incorrect).
* Tout autre problème lié à la facturation à la consommation.

Suivez les instructions de la section [Support technique pour le programme Place de marché commerciale dans l’Espace partenaires](../support.md) pour comprendre les options du éditeur et pour ouvrir un ticket de support auprès de Microsoft.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations, consultez [API du service de mesure de la consommation de la Place de marché](./marketplace-metering-service-apis.md).
