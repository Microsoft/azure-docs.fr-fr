---
title: Plans et tarification pour les offres de la place de marché commerciale
description: Découvrez les plans pour les offres de la place de marché commerciale de Microsoft dans l’Espace partenaires.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: mingshen
ms.author: mingshen
ms.date: 09/02/2020
ms.openlocfilehash: e97171e61814b832e78f7bd815d8d6f341a90840
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91858170"
---
# <a name="plans-and-pricing-for-commercial-marketplace-offers"></a>Plans et tarification pour les offres de la place de marché commerciale

Un plan définit l’étendue et les limites d’une offre, ainsi que la tarification associée, le cas échéant. Par exemple, selon le type d’offre, vous pouvez sélectionner des marchés régionaux et choisir si un plan est visible du grand public ou seulement d’un public privé. Certains types d’offres prennent en charge les abonnements récurrents ou la tarification basée sur l’utilisation, et certains permettent aux clients d’acheter l’offre avec une licence acquise directement auprès de l’éditeur. Vous avez ainsi la possibilité de fournir à vos clients des options techniques et de tarification différentes, le cas échéant.

Vous pouvez créer jusqu’à 100 plans pour chaque offre prenant en charge les plans, et jusqu’à 45 de ces plans peuvent être [privés](#plan-visibility). Vous pouvez créer des plans payants uniquement pour les offres de machines virtuelles Azure, les offres d’applications Azure (applications managées) et les offres SaaS (Software as a Service). Lorsque vous choisissez de vendre une ces offres par le biais de Microsoft (offres pouvant faire l’objet d’une transaction), vous devez créer au moins un plan. Vous pouvez créer des plans pour certains autres types d’offres, mais ces plans n’incluent pas d’options de tarification.

> [!TIP]
> Une offre pouvant faire l’objet d’une transaction est une offre en lien avec laquelle Microsoft facilite l’échange d’argent contre une licence logicielle au nom de l’éditeur.

## <a name="plans-by-offer-type"></a>Plans par type d’offre

Le tableau suivant présente les options de plan pour chaque type d’offre. Les sections suivantes décrivent ces options de façon plus détaillée. 

| Type d’offre | Plans avec options de tarification | Plans sans option de tarification | Option pour public visé |
| ------------ | ------------- | ------------- | ------------- |
| Application managée Azure | &#10004; | | &#10004; |
| Modèle de nouvelle solution | | &#10004; | &#10004; |
| Conteneur Azure | | &#10004; (BYOL) | |
| Module IoT Edge |  | &#10004; |  |
| Service géré |  | &#10004; (BYOL) | &#10004; |
| Software as a Service | &#10004; |  | &#10004; |
| Machine virtuelle Azure | &#10004; |  | &#10004; |
|||||

Les plans ne sont pas pris en charge pour les types d’offres suivants :

- Service de conseil
- Dynamics 365 Business Central
- Dynamics 365 Customer Engagement et PowerApps
- Dynamics 365 for Operations
- Application Power BI

## <a name="plan-information"></a>Informations sur le plan

Chaque type d’offre nécessite des informations distinctes lorsque vous créez un plan. Vous trouverez des liens vers des articles spécifiques de l’offre dans [Guide de publication par type d’offre](publisher-guide-by-offer-type.md). Lorsque vous créez un plan dans la page de **Vue d’ensemble du plan**, vous voyez différents onglets, tels que **Référencement du plan** ou **Tarification et disponibilité**, permettant de configurer divers détails pour votre plan. Lorsque vous utilisez ces champs, chaque onglet affiche un état incomplet ou complet.

![Illustre la page de référencement de plan dans l’Espace partenaires. Les champs Nom du plan, Résumé du plan et Description du plan sont en surbrillance.](./media/commercial-marketplace-plans/plan-listing-tab.png)

Voici quelques informations courantes à fournir pour un nouveau plan :

- **ID du plan** : créez un ID unique pour chaque plan dans cette offre. Utilisez au maximum 50 caractères : caractères minuscules, caractères alphanumériques, tirets et traits de soulignement. Cet ID sera visible par les clients dans l’adresse URL du produit et des modèles Resource Manager (si applicable). Une fois l’offre publiée, vous ne pouvez plus modifier cet ID.
- **Nom du plan** : (renvoi 1 dans l’image ci-dessus) créez un nom unique pour chaque plan dans cette offre. Utilisez au maximum 50 caractères. Le nom du plan sert à différencier les plans logiciels pouvant faire partie de la même offre (par exemple, nom de l’offre : plan Standard et plan Entreprise). Les clients verront ce nom lorsqu’ils décideront du plan à sélectionner dans votre offre.
- **Résumé du plan** : (renvoi 2 dans l’image ci-dessus) ce résumé s’affiche dans les résultats de la recherche de la Place de marché Azure et peut contenir jusqu’à 100 caractères.
   > [!NOTE]
   > Ce champ ne s’applique pas aux offres SaaS.
- **Description du plan** : (renvoi 3 dans l’image ci-dessus) ajoutez une description du plan, expliquant en quoi celui-ci se distingue des autres plans de votre offre. Utilisez au maximum 500 caractères. Vos clients verront ce contenu s’afficher sur les pages de référencement de l’offre quand ils les parcourront et sélectionneront un plan pour votre offre.

Le nom et la description du plan s’affichent sur la page de référencement de l’offre dans les magasins en ligne de la place de marché commerciale. La capture d’écran suivante montre trois plans pour un référencement d’offre SaaS dans la Place de marché Azure.

![Illustre une page de référencement d’offre dans l’Espace partenaires. Trois plans sont présentés.](./media/commercial-marketplace-plans/offer-listing-page.png)

Une fois vos plans créés, la page **Vue d’ensemble du plan** répertorie, pour chacun de vos plans, son nom, son ID, d’autres détails, son état de publication actuel et des actions disponibles. Selon l’état de publication du plan, les actions disponibles sont les suivantes :

- Si le plan est dans l’état **Brouillon** : « Supprimer le brouillon ».
- Si le plan est dans l’état **Publié** : « Arrêter la vente du plan » ou « Synchroniser le public privé ».

La capture d’écran suivante montre deux offres à l’état de brouillon.

![Illustre la page Vue d’ensemble du plan dans l’Espace partenaires. Deux plans sont présentés.](./media/commercial-marketplace-plans/plan-overview-tab.png)

## <a name="pricing-and-availability"></a>Tarification et disponibilité

La place de marché commerciale fonctionne selon un modèle d’agence. Les prix sont définis par l’éditeur, les factures sont établies par Microsoft, et l’éditeur est payé par Microsoft après déduction des frais d’agence. Vous définissez les marchés, la visibilité et la tarification de votre offre (le cas échéant) sous l’un des onglets **Tarification et disponibilité** ou **Disponibilité**.

- **Marchés** : Chaque plan doit être disponible sur au moins un marché.  Vous avez la possibilité de sélectionner uniquement des pays avec « Impôt versé », dans lesquels Microsoft acquitte les taxes de vente et d’utilisation en votre nom.
- **Prix** : les modèles de tarification s’appliquent uniquement aux plans pour des offres d’applications managées, de SaaS et de machines virtuelles Azure. Tous les plans d’une même offre doivent utiliser le même modèle de tarification.  
- **Visibilité du plan** : selon le type d’offre, vous pouvez définir un public privé ou masquer l’offre ou le plan dans la Place de marché Azure. Cela est décrit plus en détail dans la section [Visibilité du plan](#plan-visibility) plus loin dans cet article.

> [!TIP]
> Nous vous recommandons de créer des plans aussi adaptés que possible aux modèles d’utilisation de votre base de clients cible. Cela contribue à réduire la fréquence à laquelle les utilisateurs changent de plan en fonction de leurs changements d’utilisation. Pour obtenir un exemple d’offre SaaS avec trois plans de facturation contrôlés, consultez [Exemple d’offre](./partner-center-portal/saas-metered-billing.md#sample-offer).

### <a name="pricing-models"></a>Modèles tarifaires

Vous devez associer un modèle de tarification à chaque plan pour les types d’offres suivants. Chacun de ces types d’offres présente différents modèles de tarification disponibles :

- **Application managée Azure** : taux forfaitaire (mensuel) et tarification basée sur l’utilisation (dimensions de service de contrôle).
- **Logiciel en tant que service** : taux forfaitaire (mensuel ou annuel), par utilisateur, et tarification basée sur l’utilisation (dimensions du service de contrôle). 
- **Machine virtuelle Azure** : BYOL (apportez votre propre licence) et tarification basée sur l’utilisation. Pour un modèle de tarification basé sur l’utilisation, vous pouvez facturer par cœur, par taille de cœur ou par marché et taille de cœur. Un modèle de licence BYOL n’autorise pas de frais supplémentaires basés sur l’utilisation   (les offres de machine virtuelle BYOL ne nécessitent pas de modèle de tarification).

Tous les plans d’une même offre doivent utiliser le même modèle de tarification. Par exemple, une offre ne peut pas avoir à la fois un plan forfaitaire et un plan par utilisateur. Pour plus de détails, consultez la documentation de l’offre spécifique.

Si vous avez déjà fixé des prix pour votre plan en dollar américain (USD) et ajouté un autre emplacement de marché, le prix de ce nouveau marché sera calculé selon le taux de change en vigueur. Une fois vos modifications enregistrées, vous voyez s’afficher un lien **Exporter les prix (xlsx)** que vous pouvez suivre pour réviser et modifier les prix pour chaque marché avant la publication.

> [!IMPORTANT]
> Une fois votre offre publiée, vous ne pouvez plus modifier le choix du modèle de tarification.

Les offres SaaS à taux forfaitaire et les offres d’applications managées prennent en charge la facturation contrôlée à l’aide du service de contrôle de la place de marché. Il s’agit d’un modèle de facturation basé sur l’utilisation, qui vous permet de définir des unités non standard, telles que la bande passante ou les e-mails, que vos clients paieront en fonction de leur consommation. Consultez la documentation connexe pour en savoir plus sur la facturation contrôlée pour les [applications managées](./partner-center-portal/azure-app-metered-billing.md) et les [applications SaaS](./partner-center-portal/saas-metered-billing.md).

## <a name="custom-prices"></a>Prix personnalisés

Pour fixer des prix personnalisés dans un marché individuel, exportez, modifiez et importez la feuille de calcul de tarification. Il vous appartient de valider cette tarification et de définir ces paramètres.

Vérifiez vos prix avec soin avant la publication, car il y a des restrictions sur ce qui peut être modifié après la publication d’un plan :

- Une fois qu’un plan est publié, vous ne pouvez plus modifier le modèle de tarification.
- Une fois qu’une fréquence de facturation est publiée pour un plan, vous ne pouvez plus la supprimer.
- Une fois qu’un prix pour un marché dans votre plan est publié, il ne peut pas être modifié ultérieurement.

Les prix fixés en USD sont convertis dans la devise locale de tous les marchés sélectionnés selon les taux de change en vigueur disponibles lors de l’enregistrement. Validez ces prix avant la publication en exportant la feuille de calcul de tarification et en vérifiant le prix pour chaque marché sélectionné.

1. Vous devez d’abord enregistrer les modifications de vos tarifs pour autoriser l’exportation des données de tarification. Près du bas de l’onglet **Tarification et disponibilité**, sélectionnez **Enregistrer le brouillon**.
1. Sous **Tarification**, sélectionnez le lien **Exporter les données de tarification**.
1. Ouvrez le fichier exportedPrice.xlsx dans Microsoft Excel.
1. Dans la feuille de calcul, effectuez les mises à jour souhaitées pour vos informations de tarification, puis enregistrez le fichier .CSV.
    > [!NOTE]
    > Vous devrez peut-être activer la modification dans Excel avant de pouvoir mettre à jour le fichier.
1. Sous l’onglet de **Tarification et de disponibilité**, sous **Tarification**, sélectionnez **Importer des données de tarification**.
1. Dans la boîte de dialogue qui s’affiche, sélectionnez **Oui**.
1. Sélectionnez le fichier exportedPrice.xlsx que vous avez mis à jour, puis cliquez sur **Ouvrir**.

## <a name="plan-visibility"></a>Visibilité du plan

Pour certains types d’offres, vous pouvez rendre des plans publiquement disponibles ou disponibles uniquement pour un public spécifique (privé). Les offres assorties de plans privés sont publiées sur le portail Azure. En savoir plus sur les offres privées dans les [offres du marketplace commercial de Microsoft](private-offers.md).

Selon le type d’offre, vous définissez votre public privé à l’aide d’ID de locataire Azure ou d’ID d’abonnement Azure. Vous pouvez entrer jusqu’à 10 ID manuellement, ou importer jusqu’à 10 000 ID d’abonnement ou 20 000 ID de locataire (selon le cas) à l’aide d’un fichier. csv. Vous pouvez également définir des publics privés pour des services de conseil et des offres Dynamics 365 qui n’ont pas de plans associés.

Une fois votre offre publiée avec un plan privé, vous pouvez mettre à jour le public ou choisir de rendre le plan disponible pour tous. Une fois qu’un plan est publié comme visible pour tous, il doit le rester. Vous ne pouvez pas le reconfigurer en plan privé.

> [!NOTE]
> Un public privé diffère d’un public de préversion. Sur la page **Disponibilité** pour certains types d’offres, vous pouvez définir un public pouvant accéder à la préversion de votre offre avant la publication de l’offre en ligne sur la place de marché commerciale. Si la désignation de public privé ne s’applique qu’à un plan spécifique, le public de la préversion peut voir tous les plans (privés ou non), mais seulement pendant la période limitée de la préversion, pendant la phase de test et de validation du plan.

Vous pouvez également choisir de masquer entièrement le plan dans la Place de marché Azure si vous comptez déployer votre offre en lien avec un modèle de solution ou d’application managée.

## <a name="free-trials"></a>Essais gratuits

Vous pouvez activer un essai gratuit sur des plans pour des offres SaaS et de machines virtuelles Azure pouvant faire l’objet d’une transaction.

> [!NOTE]
> Cette section décrit des essais gratuits de plans payants pour les offres vendues via Microsoft. Cela diffère des référencements d’essais gratuits de partenaires qui choisissent de traiter leurs transactions de façon indépendante. Vous pouvez créer un référencement d’essai gratuit (non associé à un plan) pour les types d’offres suivants :
> - Machine virtuelle Azure 
> - SaaS 
> - Dynamics 365 Business Central
> - Dynamics 365 for Customer Engagement et PowerApps
> - Dynamics 365 for Operations
>
> Pour plus d’informations sur les options de référencement, consultez [Déterminer votre option de publication](determine-your-listing-type.md).

Les essais gratuits sont pris en charge pour tous les modèles de facturation, à l’exception des plans contrôlés. Les plans SaaS permettent des essais gratuits de 1 mois. Les plans de machines virtuelles Azure permettent des essais gratuits de 1, 3 ou 6 mois.

Quand un client sélectionne un essai gratuit, nous recueillons ses informations de facturation, mais nous ne commençons à le facturer qu’une fois la version d’évaluation convertie en abonnement payant. Les essais gratuits sont automatiquement convertis en abonnements payants à l’issue de la période d’essai, sauf si le client annule l’abonnement avant la fin de cette période.

Pendant la période d’essai, les clients peuvent évaluer les plans pris en charge à l’intérieur d’une même offre pour laquelle un essai gratuit est disponible. S’ils basculent vers un autre essai au sein de la même offre, la période d’essai ne recommence pas. Par exemple, si un client utilise un essai gratuit pendant 15 jours, puis passe à un autre essai gratuit pour la même offre, la nouvelle période d’essai prend en compte les 15 jours déjà écoulés. L’essai gratuit que le client est en train d’évaluer lorsque la période d’essai prend fin est celui qui est automatiquement converti en abonnement payant.

Après qu’un client a sélectionné un essai gratuit pour un plan, il ne peut pas passer à un abonnement payant pour ce plan tant que la période d’essai n’est pas terminée. Si un client choisit de passer à un autre plan de l’offre qui ne propose pas d’essai gratuit, la conversion se produit mais l’essai gratuit prend fin immédiatement et toutes les données sont perdues.

> [!NOTE]
> Une fois qu’un client a commencé à payer pour un plan, il ne peut plus obtenir d’essai gratuit pour la même offre, même s’il passe à un autre plan proposant des essais gratuits.

Pour obtenir des informations sur les abonnements clients qui participent actuellement à un essai gratuit `isFreeTrial`, utilisez la nouvelle propriété API, qui sera marquée comme true ou false. Pour plus d’informations, consultez [SaaS Obtenir l’API d’abonnement](./partner-center-portal/pc-saas-fulfillment-api-v2.md#get-subscription).

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment ajouter ou mettre à jour des plans pour une offre existante, consultez [Mettre à jour une offre existante dans la place de marché commerciale](./partner-center-portal/update-existing-offer.md).
- Pour en savoir plus sur les options de transaction et les modèles de tarification associés, consultez [Fonctionnalités de transaction de la place de marché commerciale](./marketplace-commercial-transaction-capabilities-and-considerations.md).
