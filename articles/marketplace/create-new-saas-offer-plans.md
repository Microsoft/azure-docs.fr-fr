---
title: Comment créer des plans pour votre offre SaaS dans le Centre pour partenaires Microsoft
description: Comment créer des plans pour une nouvelle offre de logiciel en tant que service (SaaS) à l’aide du portail de la Place de marché commerciale Microsoft dans l’Espace partenaires.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: 238ef9ec80b01470e28535d0eb42dbb3a377b005
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746435"
---
# <a name="how-to-create-plans-for-your-saas-offer"></a>Comment créer des plans pour votre offre SaaS

Les offres vendues par le biais de la Place de marché commerciale Microsoft doivent avoir au moins un plan. Vous pouvez créer divers plans avec différentes options dans la même offre. Ces plans (parfois appelés références SKU) peuvent différer en termes de version, de monétisation ou de niveaux de service. Pour obtenir des conseils détaillés sur les plans, consultez [Plans et tarification des offres de la Place de marché commerciale](plans-pricing.md).

> [!NOTE]
> Si vous choisissez de traiter les transactions indépendamment, vous ne verrez pas cette option. Au lieu de cela, passez à [Comment vendre votre offre SaaS](create-new-saas-offer-marketing.md).

## <a name="create-a-plan"></a>Créer un plan

1. Vers le haut de l’onglet **Vue d’ensemble du plan** , sélectionnez **+ Créer un plan**.

1. Dans la boîte de dialogue qui s’affiche, dans la zone **ID du plan**, entrez un ID de plan unique. Utilisez jusqu’à 50 caractères alphanumériques en minuscules, tirets ou traits de soulignement. Vous ne pouvez pas modifier l’ID de plan une fois que vous avez sélectionné **Créer**.

1. Dans la zone **Nom du plan**, entrez un nom unique pour ce plan. Utilisez un maximum de 50 caractères.

1. Sélectionnez **Create** (Créer).

## <a name="define-the-plan-listing"></a>Définir la liste des plans

Dans l’onglet **Liste des plans**, vous pouvez définir le nom et la description du plan comme vous souhaitez qu’ils apparaissent dans la Place de marché commerciale.

1. Dans la zone **Nom du plan**, le nom que vous avez fourni précédemment pour ce plan s’affiche. Vous pouvez le modifier à tout moment. Ce nom apparaîtra dans la Place de marché commerciale comme titre du plan logiciel de votre offre.

1. La zone **Description du plan** permet d’expliquer ce qui rend ce plan logiciel unique et de pointer les différences par rapport aux autres plans dans votre offre. Cette description peut contenir jusqu’à 500 caractères.
1. Sélectionnez **Enregistrer le brouillon** avant de passer à l’onglet suivant : **Tarification et disponibilité**.

## <a name="define-markets-pricing-and-availability"></a>Définir les marchés, la tarification et la disponibilité

Chaque plan doit être disponible sur au moins un marché. Sur l’onglet **Tarification et disponibilité**, vous pouvez configurer les marchés dans lesquels sera disponible ce plan, ainsi que le modèle de monétisation souhaité, le prix et la fréquence de facturation. De plus, vous pouvez indiquer si vous voulez que le plan soit visible à tous ou uniquement à des utilisateurs spécifiques (également appelé plan privé).

1. Sous **Marchés**, sélectionnez le lien **Modifier les marchés**.
1. Dans la boîte de dialogue qui s’affiche, sélectionnez les emplacements de marché où vous souhaitez que votre plan soit disponible. Vous devez sélectionner au minimum 1 et au maximum 141 marchés.

   > [!NOTE]
   > Cette boîte de dialogue inclut une zone de recherche et une option pour filtrer sur les pays « reversant les taxes », pour lesquels Microsoft reverse les ventes et utilise les taxes en votre nom.

1. puis cliquez sur **Enregistrer** pour fermer la boîte de dialogue.

## <a name="define-a-pricing-model"></a>Définir un modèle de tarification

Vous devez associer un modèle de tarification à chaque plan : _forfaitaire_ ou _par utilisateur_. Tous les plans de la même offre doivent utiliser le même modèle de tarification. Par exemple, une offre ne peut pas avoir un plan forfaitaire et un autre plan par utilisateur. Pour plus d’informations, consultez [Modèles de tarification SaaS](plan-saas-offer.md#saas-pricing-models).

> [!IMPORTANT]
> Une fois votre offre publiée, vous ne pouvez plus modifier le modèle de tarification. En outre, tous les plans d’une même offre doivent partager le même modèle de tarification.

### <a name="configure-flat-rate-pricing"></a>Configurer la tarification forfaitaire

1. Sous l’onglet de **Tarification et de disponibilité**, sous **Tarification**, sélectionnez **Taux fixe**.
1. Cochez la case **Mensuel** ou **Annuel**, ou les deux, puis entrez le prix.

### <a name="add-a-custom-meter-dimension"></a>Ajouter une dimension de compteur personnalisée

Cette option est disponible uniquement si vous avez sélectionné un tarif forfaitaire. Pour plus d’informations, consultez [Facturation à l’usage pour SaaS à l’aide du service de mesure de la Place de marché commerciale](./partner-center-portal/saas-metered-billing.md).

1. Sous **Dimensions du service de mesure de la Place de marché**, sélectionnez le lien **Ajouter une dimension de compteur personnalisée (30 max)** .
1. Dans la zone **ID**, entrez l’identificateur immuable référencé lors de l’émission d’événements d’utilisation.
1. Dans la zone **Nom d’affichage**, entrez le nom d’affichage associé à la dimension. Par exemple, « messages texte envoyés ».
1. Dans la zone **Unité de mesure**, entrez la description de l’unité de facturation. Par exemple, « par message texte » ou « pour 100 e-mails ».
1. Dans la zone **Prix unitaire en USD**, entrez un prix pour une unité de la dimension.
1. Dans la zone **Quantité mensuelle incluse de base**, entrez la quantité (sous la forme d’un entier) de la dimension incluse chaque mois pour les clients qui paient des frais mensuels récurrents. Pour définir une quantité illimitée, cochez la case à la place.
1. Dans la zone **Quantité annuelle incluse de base**, entrez la quantité de la dimension incluse chaque année pour les clients qui paient des frais annuels récurrents. Pour définir une quantité illimitée, cochez la case à la place.
1. Pour ajouter une autre dimension de mesure personnalisée, sélectionnez le lien **Ajouter une autre dimension**, puis répétez les étapes 1 à 7.

### <a name="configure-per-user-pricing"></a>Configurer la tarification par utilisateur

1. Sous l’onglet de **Tarification et de disponibilité**, sous **Tarification**, sélectionnez **Par utilisateur**.
2. Le cas échéant, sous **Limites d’utilisateurs**, spécifiez le nombre minimal et maximal d’utilisateurs pour ce plan.
3. Sous **Fréquence de facturation**, spécifiez un prix mensuel, un prix annuel ou les deux.

### <a name="validate-custom-prices"></a>Valider les prix personnalisés

Pour fixer des prix personnalisés dans un marché individuel, exportez, modifiez et importez la feuille de calcul de tarification. Vous êtes responsable de valider cette tarification et possédez ces paramètres. Pour plus d’informations, consultez [Prix personnalisés](plans-pricing.md#custom-prices).

1. Vous devez d’abord enregistrer les modifications de vos tarifs pour autoriser l’exportation des données de tarification. Près du bas de l’onglet **Tarification et disponibilité**, sélectionnez **Enregistrer le brouillon**.
1. Sous **Tarification**, sélectionnez le lien **Exporter les données de tarification**.
1. Ouvrez le fichier exportedPrice.xlsx dans Microsoft Excel.
1. Dans la feuille de calcul, effectuez les mises à jour souhaitées pour vos informations de tarification, puis enregistrez le fichier .CSV.<br> Vous devrez peut-être activer la modification dans Excel avant de pouvoir mettre à jour le fichier.
2. Sous l’onglet de **Tarification et de disponibilité**, sous **Tarification**, sélectionnez **Importer des données de tarification**.
3. Dans la boîte de dialogue qui s’affiche, cliquez sur **Oui**.
4. Sélectionnez le fichier exportedPrice.xlsx que vous avez mis à jour, puis cliquez sur **Ouvrir**.

### <a name="enable-a-free-trial"></a>Activer une version d’évaluation gratuite

Vous pouvez configurer un essai gratuit pour chaque plan de votre offre. Activez la case à cocher pour autoriser un essai gratuit d’un mois. Cette case n’est pas disponible pour les plans qui utilisent le service de mesure de la Place de marché. Pour plus d’informations, consultez la page [Essais gratuis](plans-pricing.md#free-trials).

> [!IMPORTANT]
> Une fois que votre offre négociable a été publiée avec un essai gratuit, elle ne peut plus être désactivée pour ce plan. Vérifiez que ce paramètre est correct avant de publier l’offre afin d’éviter d’avoir à recréer le plan.

- Sous **Essai gratuit**, activez la case à cocher **Autoriser une évaluation gratuite d’un mois**.

## <a name="choose-who-can-see-your-plan"></a>Choisir qui peut voir votre plan

Vous pouvez configurer chaque plan de sorte qu’il soit visible de tous ou d’un public spécifique. Vous accordez l’accès à un plan privé à l’aide d’ID locataire avec l’option d’inclure une description de chaque ID locataire que vous avez affecté. Vous pouvez ajouter un maximum de 10 ID de locataire manuellement ou jusqu’à 20 000 ID de locataire à l’aide d’un fichier .CSV.

> [!NOTE]
> Si vous publiez un plan privé, vous pouvez modifier sa visibilité sur public ultérieurement. Toutefois, une fois que vous avez publié un plan public, vous ne pouvez pas changer sa visibilité sur privé.

### <a name="make-your-plan-public"></a>Rendre votre plan public

1. Sous **Visibilité du plan**, cochez la case **Public**.
1. Sélectionnez **Enregistrer le brouillon**, puis dans le coin supérieur gauche de l’onglet, sélectionnez **Vue d’ensemble du plan** pour revenir à l’onglet **Vue d’ensemble du plan**.
1. Pour créer un autre plan pour cette offre, en haut de l’onglet **Vue d’ensemble du plan**, sélectionnez **+ Créer un plan**. Répétez ensuite les étapes de la section [Créer un plan](#create-a-plan). Sinon, accédez à [Afficher vos plans](#view-your-plans).

### <a name="manually-add-tenant-ids-for-a-private-plan"></a>Ajouter manuellement des ID de locataire pour un plan privé 

1. Sous **Visibilité du plan**, cochez la case **Il s’agit d’un plan privé**.
1. Dans la zone **ID de locataire** qui s’affiche, entrez l’ID de locataire Azure AD du public auquel vous souhaitez accorder l’accès à ce plan privé. Au moins un ID de locataire est requis.
1. (Facultatif) Entrez une description de ce public dans la zone **Description**.
1. Pour ajouter un autre ID de locataire, répétez les étapes 2 et 3.
1. Lorsque vous avez terminé d’ajouter des ID de locataire, sélectionnez **Enregistrer le brouillon**, puis dans le coin supérieur gauche de l’onglet, sélectionnez **Vue d’ensemble du plan** pour revenir à l’onglet **Vue d’ensemble du plan**.
1. Pour créer un autre plan pour cette offre, en haut de l’onglet **Vue d’ensemble du plan**, sélectionnez **+ Créer un plan**. Répétez ensuite les étapes de la section [Créer un plan](#create-a-plan). Sinon, accédez à [Afficher vos plans](#view-your-plans).

### <a name="use-a-csv-file-for-a-private-plan"></a>Utiliser un fichier .CSV pour un plan privé

1. Sous **Visibilité du plan**, cochez la case **Il s’agit d’un plan privé**.
2. Sélectionnez le lien **Exporter le public (CSV)** .
3. Ouvrez le fichier .CSV et ajoutez les identifiants Azure auxquels vous souhaitez accorder l’accès à l’offre privée à la colonne **ID**.
4. Si vous le souhaitez, entrez une description pour chaque public dans la colonne **Description**.
5. Ajoutez « TenantID » dans la colonne **Type** pour chaque ligne avec un ID Azure.
6. Enregistrez le fichier .CSV.
7. Sous l’onglet **Tarification et disponibilité**, sous **Visibilité du plan**, sélectionnez le lien **Importer le public (CSV)** .
8. Dans la boîte de dialogue qui s’affiche, sélectionnez **Oui**.
9. Sélectionnez le fichier .CSV, puis **Ouvrir**.
10. Sélectionnez **Enregistrer le brouillon**, puis dans le coin supérieur gauche de l’onglet, sélectionnez **Vue d’ensemble du plan** pour revenir à l’onglet **Vue d’ensemble du plan**.
11. Pour créer un autre plan pour cette offre, en haut de l’onglet **Vue d’ensemble du plan**, sélectionnez **+ Créer un plan**. Répétez ensuite les étapes de la section [Créer un plan](#create-a-plan). Sinon, si vous avez terminé la création de plans, passez à la section suivante : **Afficher vos plans**.

## <a name="view-your-plans"></a>Afficher vos plans

Une fois que vous avez créé un ou plusieurs plans, vous pouvez voir le nom de votre plan, son ID, son modèle de tarification, sa disponibilité (public ou privé), son état de publication actuel et toutes les actions disponibles dans l’onglet **Vue d’ensemble du plan**.

Les actions disponibles dans la colonne **Action** de l’onglet **Vue d’ensemble du plan** varient en fonction de l’état de votre plan et peuvent inclure les éléments suivants :

- Si l’état du plan est **Brouillon**, le lien figurant dans la colonne **Action** indique **Supprimer le brouillon**.
- Si l’état du plan est **Disponible**, le lien de la colonne **Action** est soit **Arrêter la vente du plan**, soit **Synchroniser le public privé**. Le lien **Synchroniser l’audience privé** publiera uniquement les modifications apportées à votre audience privée, sans publier d’autres mises à jour éventuellement apportées à l’offre.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez le [guide pratique pour vendre votre offre SaaS](create-new-saas-offer-marketing.md) par le biais des programmes **Co-vente avec Microsoft** et **Revente via des fournisseurs de solutions Cloud**.
- [Comment tester et publier une offre SaaS sur la Place de marché commerciale](test-publish-saas-offer.md).
