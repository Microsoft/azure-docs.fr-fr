---
title: Configurer un plan d’application managée
description: Découvrez comment configurer un plan d’application managée pour votre offre d’application Azure dans l’Espace partenaires.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 11/06/2020
ms.openlocfilehash: a67a721b8d94112347282e1cd9be4d43f4ac85da
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744633"
---
# <a name="configure-a-managed-application-plan"></a>Configurer un plan d’application managée

Cet article s’applique uniquement aux plans d’applications managées pour une offre d’application Azure. Si vous configurez un plan de modèle de solution, accédez à [Configurer un plan de modèle de solution](create-new-azure-apps-offer-solution.md).

## <a name="define-markets-pricing-and-availability"></a>Définir les marchés, la tarification et la disponibilité

Chaque plan doit être disponible sur au moins un marché. Sous l’onglet **Tarification et disponibilité**, vous pouvez configurer les marchés dans lesquels ce plan sera disponible, son prix, et spécifier s’il doit être visible pour tout le monde ou uniquement pour des clients spécifiques (plan privé).

1. Sous **Marchés**, sélectionnez le lien **Modifier les marchés**.
1. Dans la boîte de dialogue qui s’affiche, sélectionnez les emplacements de marché où vous souhaitez que votre plan soit disponible. Vous devez sélectionner au minimum 1 et au maximum 141 marchés.

    > [!NOTE]
    > Cette boîte de dialogue inclut une zone de recherche et une option pour filtrer sur les pays « reversant les taxes », pour lesquels Microsoft reverse les ventes et utilise les taxes en votre nom.

1. puis cliquez sur **Enregistrer** pour fermer la boîte de dialogue.

## <a name="define-pricing"></a>Définir la tarification

Dans la zone **Prix**, indiquez le tarif mensuel de ce plan. Ce tarif s’ajoute aux coûts d’utilisation et d’infrastructure d’Azure qu’engendrent les ressources que cette solution déploie.

En plus du tarif mensuel, vous pouvez fixer des prix pour l’utilisation d’unités non standard assortie d’une [facturation basée sur des mesures](partner-center-portal/azure-app-metered-billing.md). Si vous le souhaitez, vous pouvez définir le tarif mensuel sur zéro et facturer exclusivement via une facturation basée sur des mesures.

Les prix fixés en USD (USD = dollar américain) sont convertis dans la devise locale de tous les marchés sélectionnés, selon les taux de change en vigueur disponibles lors de l’enregistrement. Validez ces prix avant la publication en exportant la feuille de calcul de tarification et en vérifiant le prix de chaque marché. Si vous souhaitez fixer des prix personnalisés dans un marché individuel, modifiez et importez la feuille de calcul de tarification.

### <a name="add-a-custom-meter-dimension-optional"></a>Ajouter une dimension de compteur personnalisée (facultatif)

1. Sous **Dimensions du service de contrôle de la Place de marché**, sélectionnez le lien **Ajouter une dimension de compteur personnalisée (18 max)** .
1. Dans la zone **ID**, entrez l’identificateur immuable référencé lors de l’émission d’événements d’utilisation.
1. Dans la zone **Nom d’affichage**, entrez le nom d’affichage associé à la dimension. Par exemple, « messages texte envoyés ».
1. Dans la zone **Unité de mesure**, entrez la description de l’unité de facturation. Par exemple, « par message texte » ou « pour 100 e-mails ».
1. Dans la zone **Prix unitaire en USD**, entrez un prix pour une unité de la dimension.
1. Dans la zone **Quantité mensuelle incluse de base**, entrez la quantité (sous la forme d’un entier) de la dimension incluse chaque mois pour les clients qui paient des frais mensuels récurrents. Pour définir une quantité illimitée, cochez la case à la place.
1. Pour ajouter une autre dimension de compteur personnalisée, répétez les étapes 1 à 7.

### <a name="set-custom-prices-optional"></a>Définir des prix personnalisés (facultatif)

Les prix fixés en USD (USD = dollar américain) sont convertis dans la devise locale de tous les marchés sélectionnés selon les taux de change en vigueur disponibles lors de l’enregistrement. Validez ces prix avant la publication en exportant la feuille de calcul de tarification et en vérifiant le prix de chaque marché. Si vous souhaitez fixer des prix personnalisés dans un marché individuel, modifiez et importez la feuille de calcul de tarification.

Vérifiez vos tarifs avec soin avant de publier votre plan, car après la publication, certains éléments ne sont plus modifiables.

> [!NOTE]
> Une fois qu’un prix pour un marché dans votre plan est publié, il ne peut pas être modifié ultérieurement.

Pour fixer des prix personnalisés dans un marché individuel, exportez, modifiez et importez la feuille de calcul de tarification. Vous êtes responsable de valider cette tarification et possédez ces paramètres. Pour plus d’informations, consultez [Prix personnalisés](plans-pricing.md#custom-prices).

1. Vous devez d’abord enregistrer les modifications de vos tarifs pour autoriser l’exportation des données de tarification. Près du bas de l’onglet **Tarification et disponibilité**, sélectionnez **Enregistrer le brouillon**.
1. Sous **Tarification**, sélectionnez le lien **Exporter les données de tarification**.
1. Ouvrez le fichier exportedPrice.xlsx dans Microsoft Excel.
1. Dans la feuille de calcul, effectuez les mises à jour souhaitées pour vos informations de tarification, puis enregistrez le fichier.

   Vous devrez peut-être activer la modification dans Excel avant de pouvoir mettre à jour le fichier.

1. Sous l’onglet de **Tarification et de disponibilité**, sous **Tarification**, sélectionnez **Importer des données de tarification**.
1. Dans la boîte de dialogue qui s’affiche, cliquez sur **Oui**.
1. Sélectionnez le fichier exportedPrice.xlsx que vous avez mis à jour, puis cliquez sur **Ouvrir**.

## <a name="choose-who-can-see-your-plan"></a>Choisir qui peut voir votre plan

Vous pouvez configurer chaque plan de sorte qu’il soit visible de tous ou d’un public spécifique. Vous accordez l’accès à un public privé à l’aide d’ID d’abonnement Azure, avec la possibilité d’inclure une description de chaque ID d’abonnement que vous attribuez. Vous pouvez ajouter jusqu’à 10 ID d’abonnement manuellement ou jusqu’à 10 000 ID d’abonnement à l’aide d’un fichier .CSV. Les ID d’abonnements Azure sont représentés par des GUID, où les lettres doivent être en minuscules.

> [!NOTE]
> Si vous publiez un plan privé, vous pouvez modifier sa visibilité sur public ultérieurement. Toutefois, une fois que vous avez publié un plan public, vous ne pouvez pas changer sa visibilité sur privé.

Sous **Visibilité du plan**, effectuez l’une des opérations suivantes :

- Pour rendre le plan public, sélectionnez la case d’option (ou _bouton d’option_) **Public**.
- Pour rendre le plan privé, sélectionnez la case d’option **Privé**, puis ajoutez les ID d’abonnement Azure manuellement ou à l’aide d’un fichier .CSV.

> [!NOTE]
> Un public privé ou restreint est différent du public de préversion que vous avez défini sous l’onglet **Préversion**. Un public de préversion peut accéder à votre offre avant sa publication sur la place de marché. Tandis que la sélection d’un public privé ne s’applique qu’à un plan spécifique, le public de préversion peut voir tous les plans (privés ou non) dans le cadre de la validation.

### <a name="manually-add-azure-subscription-ids-for-a-private-plan"></a>Ajouter manuellement des ID d’abonnement Azure pour un plan privé

1. Sous **Visibilité du plan**, sélectionnez la case d’option **Privé**.
1. Dans la zone **ID d’abonnement Azure** qui s’affiche, entrez l’ID d’abonnement Azure du public auquel vous souhaitez accorder l’accès à ce plan privé. Au moins un ID d’abonnement est requis.
1. (Facultatif) Entrez une description de ce public dans la zone **Description**.
1. Pour ajouter un ID d’abonnement, sélectionnez le lien **Ajouter ID (max 10)** , puis répétez les étapes 2 et 3.

### <a name="use-a-csv-file-to-add-azure-subscription-ids-for-a-private-plan"></a>Utiliser un fichier .CSV pour ajouter des ID d’abonnement Azure pour un plan privé

1. Sous **Visibilité du plan**, sélectionnez la case d’option **Privé**.
1. Sélectionnez le lien **Exporter le public (CSV)** .
1. Ouvrez le fichier .CSV et ajoutez les ID d’abonnement Azure auxquels vous souhaitez accorder l’accès à l’offre privée dans la colonne **ID**.
1. Si vous le souhaitez, entrez une description pour chaque public dans la colonne **Description**.
1. Ajoutez « SubscriptionId » dans la colonne **Type** pour chaque ligne contenant un ID d’abonnement.
1. Enregistrez le fichier .CSV.
1. Sous l’onglet **Disponibilité**, sous **Visibilité du plan**, sélectionnez le lien **Importer le public (CSV)** .
1. Dans la boîte de dialogue qui s’affiche, sélectionnez **Oui**.
1. Sélectionnez le fichier .CSV, puis **Ouvrir**. Un message s’affiche, indiquant que le fichier .CSV a été correctement importé.

## <a name="define-the-technical-configuration"></a>Définir la configuration technique

Sous l’onglet **Configuration technique**, vous allez charger le package de déploiement permettant aux clients de déployer votre plan, et fournir un numéro de version pour le package. Vous allez également fournir d’autres informations techniques.

> [!NOTE]
> Cet onglet n’est pas visible si vous avez choisi de réutiliser les packages d’un autre plan sous l’onglet **Configuration du plan**. Dans ce cas, accédez à [Afficher vos plans](#view-your-plans).

### <a name="assign-a-version-number-for-the-package"></a>Affecter un numéro de version pour le package

Dans la zone **Version**, fournissez la version actuelle de la configuration technique. Incrémentez cette version chaque fois que vous publiez une modification apportée à cette page. Le numéro de version doit être au format : entier.entier.entier. Par exemple : `1.0.2`.

### <a name="upload-a-package-file"></a>Charger un fichier de package

Sous **Fichier de package (.zip)** , faites glisser votre fichier de package vers la zone grise, ou sélectionnez le lien pour **parcourir vos fichiers**.

> [!NOTE]
> Si vous rencontrez un problème de téléchargement de fichiers, assurez-vous que votre réseau local ne bloque pas le service `https://upload.xboxlive.com` utilisé par l’espace partenaires.

#### <a name="previously-published-packages"></a>Packages précédemment publiés

Le sous-onglet **Packages précédemment publiés** affiche toutes les versions publiées de votre configuration technique.

### <a name="enable-just-in-time-jit-access-optional"></a>Activer l’accès juste-à-temps (JIT) (facultatif)

Pour activer l’accès JIT pour ce plan, activez la case à cocher **Activer l’accès juste-à-temps (JIT)** . Pour exiger que les consommateurs de votre application managée accordent à votre compte un accès permanent, laissez cette option non cochée. Pour en savoir plus sur cette option, consultez [Accès juste-à-temps (JIT)](plan-azure-app-managed-app.md#just-in-time-jit-access).

### <a name="select-a-deployment-mode"></a>Sélectionner un mode de déploiement

Sélectionnez le mode de déploiement **Complet** ou **Incrémentiel**.

- En mode **Complet**, un redéploiement de l’application par le client entraîne la suppression, dans le groupe de ressources managées, des ressources non définies dans le fichier [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template).
- En mode **Incrémentiel**, un redéploiement de l’application laisse les ressources existantes inchangées.

Pour en savoir plus sur les modes de déploiement, consultez [Modes de déploiement Azure Resource Manager](../azure-resource-manager/templates/deployment-modes.md).

### <a name="provide-a-notification-endpoint-url"></a>Fournir une URL de point de terminaison de notification

Dans la zone **URL de point de terminaison de notification**, fournissez un point de terminaison webhook HTTPS pour recevoir des notifications concernant toutes les opérations CRUD effectuées sur des instances d’application managée de cette version de plan.

### <a name="customize-allowed-customer-actions-optional"></a>Personnaliser les actions des clients autorisées (facultatif)

1. Pour spécifier les actions que les clients peuvent effectuer sur les ressources managées en plus des actions « `*/read` » autorisées par défaut, activez la case **Personnaliser les actions des clients autorisées**.
1. Dans les zones qui s’affichent, indiquez les actions de contrôle supplémentaires et les actions sur les données autorisées que vous souhaitez autoriser votre client à effectuer, en les séparant par des points-virgules. Par exemple, pour permettre aux consommateurs de redémarrer des machines virtuelles, ajoutez `Microsoft.Compute/virtualMachines/restart/action` dans la zone **Actions de contrôle autorisées**.

### <a name="choose-who-can-manage-the-application"></a>Choisir qui peut gérer l’application

Définissez qui doit avoir un accès administrateur à cette application managée dans chaque région Azure sélectionnée : _Azure global_ et _Azure Government Cloud_. Vous allez utiliser des identités Azure AD pour identifier les utilisateurs, les groupes ou les applications auxquels vous souhaitez accorder l’autorisation d’accès au groupe de ressources managé. Pour plus d’informations, consultez [Planifier une application managée Azure pour une offre Azure Application](plan-azure-application-offer.md).

Procédez comme suit pour Azure global et Azure Government Cloud, le cas échéant.

1. Dans la zone **ID de locataire Azure Active Directory**, entrez ID du locataire Azure AD (également appelé ID d’annuaire) contenant les identités des utilisateurs, des groupes ou des applications auxquels vous souhaitez accorder des autorisations.
1. Dans la zone **ID du principal**, entrez l’ID d’objet Azure AD de l’utilisateur, du groupe ou de l’application auxquels vous souhaitez accorder l’autorisation d’accès au groupe de ressources managé. Identifiez l’utilisateur par son ID de principal, que vous trouverez dans le [panneau Utilisateurs d’Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) sur le portail Azure.
1. Dans la liste **Définition de rôle**, sélectionnez un rôle intégré Azure AD. Le rôle sélectionné détermine les autorisations que le principal aura sur les ressources dans l’abonnement client.
1. Pour ajouter une autorisation, sélectionnez le lien **Ajouter une autorisation (max 100)** , puis répétez les étapes 1 à 3.

### <a name="policy-settings-optional"></a>Paramètres de stratégie (facultatif)

Vous pouvez configurer un maximum de cinq stratégies, et une seule instance de chaque option de stratégie. Certaines stratégies requièrent des paramètres supplémentaires.

1. Sous **Paramètres de stratégie**, sélectionnez le lien **+ Ajouter une stratégie (max 5)** .
1. Dans la zone **Nom**, entrez le nom de l’attribution de stratégie (limité à 50 caractères).
1. Dans la zone de liste **Stratégies**, sélectionnez la stratégie Azure qui sera appliquée aux ressources créées par l’application managée dans l’abonnement client.
1. Dans la zone **Paramètres de stratégie**, entrez le paramètre auquel les stratégies de paramètres d’audit et de diagnostic doivent être appliquées.
1. Dans la zone de liste **Référence (SKU) de stratégie**, sélectionnez le type de référence (SKU) de la stratégie.

    > [!NOTE]
    > La référence (SKU) _Stratégie standard_ est requise pour les stratégies d’audit.

## <a name="view-your-plans"></a>Afficher vos plans

- Sélectionnez **Enregistrer le brouillon**, puis, en haut à gauche de la page, sélectionnez **Vue d’ensemble du plan** pour revenir à l’onglet **Vue d’ensemble du plan**.

Une fois que vous avez créé un ou plusieurs plans, sous l’onglet **Vue d’ensemble du plan**, vous pouvez voir le nom de votre plan, son ID, son type, sa disponibilité (public ou privé), son état de publication actuel et toutes les actions disponibles.

Les actions disponibles dans la colonne **Action** de l’onglet **Vue d’ensemble du plan** varient en fonction de l’état de votre plan et peuvent inclure les éléments suivants :

- Si l’état du plan est **Brouillon**, le lien figurant dans la colonne **Action** indique **Supprimer le brouillon**.
- Si l’état du plan est **Disponible**, le lien dans la colonne **Action** est **Arrêter la vente du plan** ou **Synchroniser le public privé**. Le lien **Synchroniser l’audience privé** publiera uniquement les modifications apportées à votre audience privée, sans publier d’autres mises à jour éventuellement apportées à l’offre.
- Pour créer un autre plan pour cette offre, en haut de l’onglet **Vue d’ensemble du plan**, sélectionnez **+ Créer un plan**. Répétez les étapes décrites dans [Comment créer des plans pour votre offre d’application Azure](create-new-azure-apps-offer-plans.md). Sinon, si vous avez terminé la création de plans, passez à la section suivante : Étapes suivantes.

## <a name="next-steps"></a>Étapes suivantes

- [Comment tester et publier votre offre Azure Application](create-new-azure-apps-offer-test-publish.md).
- Découvrez [comment vendre votre offre Azure Application](create-new-azure-apps-offer-marketing.md) par le biais des programmes de Co-vente avec Microsoft et de Revente via des fournisseurs de solutions Cloud.