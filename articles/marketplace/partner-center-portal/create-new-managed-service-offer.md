---
title: Créer une offre de service managé sur la Place de marché commerciale Microsoft
description: Guide pratique pour créer une offre de service managé en vue de l’annoncer sur la Place de marché Azure à l’aide du portail Place de marché commerciale de l’Espace partenaires.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.date: 08/07/2020
ms.openlocfilehash: d253bda69839c76aed0170f9a79a03b7d4bf1a79
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130104"
---
# <a name="create-a-managed-service-offer"></a>Créer une offre de service managé

Les offres de service managé aident à mettre en œuvre les scénarios [Azure Lighthouse](../../lighthouse/overview.md). Quand un client accepte une offre de service managé, il est en mesure d’intégrer des ressources pour la [gestion des ressources déléguées Azure](../../lighthouse/concepts/azure-delegated-resource-management.md). Avant de commencer, si ce n’est déjà fait, [créez un compte de la Place de marché commerciale dans l’Espace partenaires](create-account.md). Vérifiez que votre compte est inscrit dans le programme de Place de marché commerciale.

Vous devez disposer d’un [niveau de compétence de plateforme cloud Silver ou Gold](https://partner.microsoft.com/membership/cloud-platform-competency) ou être [Fournisseur de services managés Azure Expert](https://partner.microsoft.com/membership/azure-expert-msp) pour publier une offre de service managé.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le menu de navigation de gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.
3. Dans la page de présentation, sélectionnez **+ Nouvelle offre** > **Service géré**.

    ![Illustre le menu de navigation de gauche.](./media/new-offer-managed-service.png)

>[!NOTE]
>Après la publication d’une offre, les modifications qui lui ont été apportées dans l’Espace partenaires n’apparaissent dans les magasins en ligne qu’après republication de l’offre. Veillez à toujours effectuer une nouvelle publication après avoir apporté des modifications.

## <a name="new-offer"></a>Nouvelle offre

Entrez un **ID d’offre**. Il s’agit d’un identificateur unique par offre dans votre compte.

* Cet ID est visible par les clients dans l’adresse web de l’offre de la Place de marché et des modèles Resource Manager, le cas échéant.
* Utilisez uniquement des lettres minuscules et des chiffres. Il peut inclure des traits d’union et des traits de soulignement, mais pas d’espaces, et est limité à 50 caractères. Par exemple, si vous entrez **test-offer-1** , l’adresse web de l’offre sera `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
* L’ID d’offre ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

* Ce nom n’est pas utilisé dans la Place de marché et est différent du nom de l’offre et des autres informations présentées aux clients.
* L’alias d’offre ne peut pas être modifié une fois que vous avez sélectionné **Créer**.

Sélectionnez **Créer** pour générer l’offre et continuer.

## <a name="offer-setup"></a>Configuration de l’offre

### <a name="customer-leads"></a>Prospects

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Selon les [stratégies de certification des services gérés](/legal/marketplace/certification-policies#700-managed-services), une **destination de prospect** est requise. Ainsi, un enregistrement est créé dans votre système CRM chaque fois qu’un client déploie votre offre.

Pour plus d’informations, consultez [Vue d’ensemble de la gestion des prospects](./commercial-marketplace-get-customer-leads.md).

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="properties"></a>Propriétés

Cette page vous permet de définir les catégories utilisées pour grouper votre offre sur la Place de marché, ainsi que les contrats juridiques associés.

### <a name="category"></a>Category

Sélectionnez un minimum d’une catégorie et un maximum de cinq catégories qui permettent de placer votre offre dans les zones de recherche appropriées sur la Place de marché. Veillez à expliquer dans la description comment votre offre prend en charge ces catégories dans la description de l’offre.

### <a name="terms-and-conditions"></a>Conditions générales

Fournissez vos propres conditions légales dans le champ **Conditions générales**. Vous pouvez également fournir l’URL permettant d’accéder à vos conditions générales. Les clients doivent accepter ces conditions avant de pouvoir essayer votre offre.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="offer-listing"></a>Annonce de l’offre

Cette page vous permet de définir des détails, tels que le nom, la description et les images de l’offre, affichés sur la Place de marché.

> [!NOTE]
> Il n’est pas nécessaire que le contenu du référencement de l’offre (par exemple, la description, les documents, les captures d’écran et les conditions d’utilisation) soit en anglais, tant que sa description commence par l’expression : « This application is available only in [langue autre que l’anglais]. » Il est également possible de fournir une *URL de lien utile* pour offrir du contenu dans une langue autre que celle utilisée dans le contenu de la liste des offres.

Voici un exemple de la façon dont les informations de l’offre s’affichent dans le portail Azure :

:::image type="content" source="media/example-managed-services.png" alt-text="Illustre la façon dont cette offre s’affiche dans le portail Azure.":::

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Intitulé
2. Description
3. Liens utiles
4. Captures d’écran.

### <a name="name"></a>Nom

Le nom que vous entrez ici s’affichera auprès des clients comme titre de votre référencement d’offre. Ce champ est prérempli avec le texte que vous avez entré pour **Alias d’offre** lorsque vous avez créé l’offre, mais vous pouvez modifier cette valeur. Ce nom peut être une marque déposée (et vous pouvez inclure des symboles de marque ou de copyright). Le nom ne peut pas comporter plus de 50 caractères et ne peut pas contenir d’emoji.

### <a name="search-results-summary"></a>Résumé des résultats de recherche

Renseignez une courte description de votre offre (100 caractères maximum) à utiliser dans les résultats de recherche dans une place de marché.

### <a name="long-summary"></a>Résumé long

Fournissez une description plus longue de votre offre (jusqu’à 256 caractères). Ce résumé long peut également être utilisé dans les résultats de recherche de la Place de marché.

### <a name="description"></a>Description

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

### <a name="privacy-policy-link"></a>Lien de la politique de confidentialité

Entrez l’URL de la politique de confidentialité de votre organisation (hébergée sur votre site). Vous devez veiller à ce que votre application respecte les lois et réglementations en matière de confidentialité et à renseigner une politique de confidentialité conforme.

### <a name="useful-links"></a>Liens utiles

Fournissez facultativement des documents en ligne complémentaires à propos de votre solution. Ajoutez des liens utiles supplémentaires en cliquant sur **+ Ajouter un lien**.

### <a name="contact-information"></a>Informations de contact

Dans cette section, entrez le nom, l’adresse e-mail et le numéro de téléphone d’un **contact d’ingénierie** et d’un **contact du service clientèle**. Ces informations ne sont pas présentées aux clients, mais seront disponibles pour Microsoft et pourront être fournies aux partenaires CSP.

### <a name="support-urls"></a>URL de support technique

Si vous avez des sites web de support pour les **clients Azure international** et/ou les **clients Azure Government** , fournissez ces URL ici.

### <a name="marketplace-images"></a>Images de la Place de marché

Dans cette section, vous pouvez fournir des logos et des images qui seront utilisés pour montrer votre offre au client. Toutes les images doivent être au format .png.

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si vous rencontrez un problème lors du chargement de fichiers, vérifiez que votre réseau local ne bloque pas le service https://upload.xboxlive.com utilisé par l’Espace partenaires.

#### <a name="store-logos"></a>Stocker des logos

Fournissez un fichier PNG pour le logo de **Grande** taille. L'Espace partenaires l'utilisera pour créer un logo **Petit** et **Moyen**. Plus tard, vous pourrez éventuellement les remplacer par d'autres images.

- **Grand** (de 216 x 216 à 350 x 350 px, obligatoire)
- **Moyen** (90 x 90 px, facultatif)
- **Petit** (48 x 48 px, facultatif)

Ces logos sont utilisés à différents emplacements dans la liste :

[!INCLUDE [logos-azure-marketplace-only](../includes/logos-azure-marketplace-only.md)]

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Captures d’écran.

Ajoutez jusqu’à cinq captures d’écran qui illustrent le fonctionnement de votre offre. Toutes les captures d’écran doivent être au format 1280 x 720 pixels.

#### <a name="videos"></a>Vidéos

Vous pouvez éventuellement ajouter jusqu’à cinq vidéos qui illustrent votre offre. Ces vidéos doivent être hébergées sur YouTube et/ou Vimeo. Pour chaque vidéo, entrez son nom, son URL et une image miniature correspondante (1280 x 720 pixels).

#### <a name="additional-marketplace-listing-resources"></a>Ressources d’annonce supplémentaires de place de marché

- [Meilleures pratiques pour les annonces d’offre d’une place de marché](../gtm-offer-listing-best-practices.md)

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="preview"></a>PRÉVERSION

Avant de publier votre offre sur la Place de marché générale, vous devez la mettre à disposition d’un public de préversion limité. Vous pouvez ainsi vérifier l’affichage de votre offre dans la Place de marché Azure avant de la mettre à la disposition des clients. Le support technique Microsoft et les équipes d’ingénierie pourront également voir votre offre pendant cette période de préversion.

Vous pouvez définir le public de préversion en entrant des ID d’abonnement Azure dans la section **Public de préversion**. Vous pouvez entrer jusqu’à 10 ID d’abonnement manuellement ou charger un fichier .csv avec un maximum de 100 ID d’abonnement.

Tous les clients associés à ces abonnements pourront voir l’offre sur la Place de marché Azure avant sa mise en ligne. Veillez à inclure vos propres abonnements ici afin de pouvoir voir un aperçu de votre offre.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="plan-overview"></a>Vue d’ensemble du plan

Chaque offre doit avoir un ou plusieurs plans (anciennement « références SKU »). Vous pouvez ajouter plusieurs plans correspondant à différents ensembles de fonctionnalités proposés à différents prix, ou personnaliser un plan spécifique pour un public limité de clients spécifiques. Les clients peuvent afficher les plans qui sont à leur disposition sous l’offre parente.

Vous pouvez créer jusqu’à 100 plans pour chaque offre : jusqu’à 45 peuvent être privés. En savoir plus sur les offres privées dans les [offres du marketplace commercial de Microsoft](../private-offers.md).

Dans la page **Vue d’ensemble du plan** , sélectionnez **+ Créer un plan**. Entrez ensuite un **ID de plan** et un **Nom de plan**. Ces deux valeurs ne peuvent contenir que des caractères alphanumériques minuscules, des tirets et des traits de soulignement, dont le nombre ne peut pas dépasser 50 signes. Ces valeurs peuvent être visibles par les clients et elles ne peuvent pas être changées après la publication de l’offre.

Sélectionnez **Créer** une fois que vous avez entré ces valeurs pour continuer à travailler sur votre plan. Les sections à effectuer sont au nombre de trois : **Liste des plans** , **Tarification et disponibilité** et **Configuration technique**.

### <a name="plan-listing"></a>Liste des plans

Tout d’abord, fournissez un **Résumé des résultats de recherche** pour le plan. Il s’agit d’une courte description de votre plan (100 caractères maximum) à utiliser dans les résultats de recherche dans une Place de marché.

Ensuite, entrez une **Description** fournissant davantage de détails sur le plan.

### <a name="pricing-and-availability"></a>Tarification et disponibilité

Il n’existe qu’un seul modèle de prix qui peut être utilisé pour une offre de service managé : **BYOL (apportez votre propre licence)** . Cela signifie que vous facturez directement à vos clients les coûts liés à l’offre, et que Microsoft ne vous facture aucuns frais.

La section **Visibilité du plan** vous permet d’indiquer si ce plan doit être [privé](../../marketplace/private-offers.md). Si vous laissez la case **Il s’agit d’un plan privé** décochée, votre plan ne sera pas limité à des clients spécifiques (ou à un certain nombre de clients).

> [!NOTE]
> Les plans privés ne sont pas pris en charge par les abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).

Pour que ce plan soit disponible uniquement pour des clients spécifiques, sélectionnez **Oui**. Dans ce cas, vous devez identifier les clients en fournissant leurs ID d’abonnement. Vous pouvez entrer ceux-ci un par un (pour un maximum de 10 abonnements) ou en chargeant un fichier .csv (pour un maximum de 10 000 abonnements entre tous les plans). Veillez à inclure vos propres abonnements ici afin de pouvoir tester et valider l’offre.

> [!IMPORTANT]
> Une fois que vous avez publié un plan public, vous ne pouvez plus le changer en plan privé. Pour contrôler les clients qui peuvent accepter votre offre et déléguer des ressources, utilisez un plan privé. Avec un plan public, vous ne pouvez pas limiter la disponibilité à des clients spécifiques ou à un certain nombre de clients (en revanche, vous pouvez arrêter complètement la vente du plan si vous le souhaitez). Vous pouvez [supprimer l’accès à une délégation](../../lighthouse/how-to/remove-delegation.md) après qu’un client a accepté une offre uniquement si vous avez inclus une **Autorisation** avec la **Définition de rôle** définie sur [Inscription des services managés, attribution Supprimer le rôle](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) lors de la publication de l’offre. Vous pouvez également contacter le client et lui demander de [supprimer votre accès](../../lighthouse/how-to/view-manage-service-providers.md#add-or-remove-service-provider-offers).

### <a name="technical-configuration"></a>Configuration technique

Cette section de votre plan a pour effet de créer un manifeste contenant des informations d’autorisation pour la gestion des ressources du client. Ces informations sont nécessaires pour activer la [gestion des ressources déléguées Azure](../../lighthouse/concepts/azure-delegated-resource-management.md).

Veillez à consulter [Locataires, rôles et utilisateurs dans les scénarios Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md#best-practices-for-defining-users-and-roles) afin de comprendre quels rôles sont pris en charge et les bonnes pratiques pour la définition de vos autorisations.

> [!NOTE]
> Comme indiqué ci-dessus, les utilisateurs et rôles de vos entrées **Autorisation** s’appliquent à tous les clients qui achètent le plan. Si vous souhaitez limiter l’accès à un client spécifique, vous devez publier un plan privé pour son usage exclusif.

#### <a name="manifest"></a>Manifeste

Tout d’abord, fournissez une **Version** pour le manifeste. Utilisez le format *n.n.n* (par exemple, 1.2.5).

Ensuite, entrez votre **ID de locataire**. Il s’agit d’un GUID associé à l’ID de locataire Azure Active Directory (Azure AD) de votre organisation, autrement dit le locataire gérant à partir duquel vous allez accéder aux ressources de vos clients. Si vous ne l’avez pas sous la main, vous pouvez le trouver en pointant sur le nom de votre compte dans l’angle supérieur droit du portail Azure ou en sélectionnant **Changer de répertoire**.

Si vous publiez une nouvelle version de votre offre et que vous devez créer un manifeste mis à jour, sélectionnez **+ Nouveau manifeste**. Veillez à augmenter le numéro de la version précédente du manifeste.

#### <a name="authorization"></a>Autorisation

Les autorisations définissent les entités dans votre locataire gérant qui peuvent accéder aux ressources et aux abonnements pour les clients qui achètent le plan. Chaque entité se voit attribuer un rôle intégré qui accorde des niveaux d’accès spécifiques.

Vous pouvez créer jusqu’à vingt autorisations pour chaque plan.

> [!TIP]
> Dans la plupart des cas, vous affectez des rôles à un groupe d’utilisateurs ou à un principal de service Azure AD, plutôt qu’à une série de comptes d’utilisateur individuels. Cela vous permet d’ajouter ou de supprimer l’accès d’utilisateurs individuels sans devoir mettre à jour et republier le plan lorsque vos conditions d’accès changent. Quand vous affectez des rôles à des groupes Azure AD, [assurez-vous que le **Type de groupe** est **Sécurité** et non **Office 365**](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Pour obtenir des recommandations supplémentaires, consultez [Locataires, rôles et utilisateurs dans les scénarios Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md).

Pour chaque **autorisation** , vous devez fournir les informations suivantes. Vous pouvez ensuite sélectionner **+ Ajouter une autorisation** autant de fois que nécessaire pour ajouter des utilisateurs et des définitions de rôles.

- **ID d’objet Azure AD**  : identificateur Azure AD d’un utilisateur, d’un groupe d’utilisateurs ou d’une application auxquels certaines autorisations seront accordées (comme défini dans la définition de rôle) sur les ressources de votre client.
- **Nom d’affichage d’objet Azure AD**  : nom convivial destiné à aider le client à comprendre l’objectif de cette autorisation. Le client verra ce nom lors de la délégation de ressources.
- **Définition de rôle**  : sélectionnez l’un des rôles intégrés Azure AD disponibles dans la liste. Ce rôle détermine les autorisations sur les ressources de vos clients dont disposera l’utilisateur spécifié dans le champ **ID d’objet Azure AD**. Pour obtenir une description de ces rôles, consultez [Rôles intégrés](../../role-based-access-control/built-in-roles.md) et [Prise en charge des rôles pour Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md#role-support-for-azure-lighthouse).
  > [!NOTE]
  > Comme les nouveaux rôles intégrés applicables sont ajoutés à Azure, ils sont disponibles ici, même si un certain temps puis s’écouler avant qu’ils n’apparaissent.
- **Rôles attribuables**  : cette option apparaît uniquement si vous avez sélectionné Administrateur de l’accès utilisateur dans la **Définition de rôle** pour cette autorisation. Si tel est le cas, vous devez ajouter un ou plusieurs rôles attribuables ici. L’utilisateur indiqué dans le champ **ID d’objet Azure AD** sera en mesure d’attribuer ces rôles à des [identités managées](../../active-directory/managed-identities-azure-resources/overview.md), ce qui est nécessaire pour [déployer des stratégies qui peuvent être corrigées](../../lighthouse/how-to/deploy-policy-remediation.md). Notez qu’aucune autre autorisation normalement associée au rôle Administrateur de l’accès utilisateur ne s’appliquera à cet utilisateur.

> [!TIP]
> Pour être sûr de pouvoir [supprimer l’accès à une délégation](../../lighthouse/how-to/remove-delegation.md) en cas de nécessité, incluez une **Autorisation** avec la **Définition de rôle** définie sur [Inscription des services managés, attribution Supprimer le rôle](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role). Si ce rôle n’est pas attribué, les ressources déléguées ne peuvent être supprimées que par un utilisateur dans le locataire du client.

Une fois que vous avez renseigné toutes les sections de votre plan, vous pouvez sélectionner **+ Créer un plan** autant de fois que nécessaire pour créer des plans supplémentaires. Quand vous avez terminé, sélectionnez **Enregistrer**.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="publish"></a>Publish

### <a name="submit-offer-to-preview"></a>Soumettre l’offre en préversion

Une fois que vous avez complété toutes les sections obligatoires de l’offre, sélectionnez **Passer en revue et publier** dans l’angle supérieur droit du portail.

S’il s’agit de votre première publication de cette offre, vous pouvez :

- Consultez l’état d’achèvement de chaque section de l’offre.
  - **Non démarrée** : la section n’a pas encore été modifiée et doit être complétée.
  - **Incomplète** : la section comporte des erreurs qui doivent être corrigées ou nécessite un complément d’informations. Revenir à la ou les sections et les mettre à jour.
  - **Complète** : la section est complète, toutes les données obligatoires ont été fournies et aucune erreur n’est présente. Toutes les sections de l’offre doivent être dans l’état Complète avant de pouvoir envoyer l’offre.
- Dans la sections **Notes pour la certification** , fournissez des instructions de test à l’équipe de certification pour veiller à ce que votre application soit testée correctement, en plus de toutes autres notes supplémentaires utiles pour comprendre l’application.
- Envoyez l’offre pour publication en sélectionnant **Envoyer**. Nous vous enverrons un e-mail si une version d’essai de l’offre est disponible pour vérification et approbation. Retourner dans Partner Center et sélectionner **Publier** pour que votre offre soit publiée pour le public (ou au public privé, si l’offre est privée).

### <a name="customer-experience-and-offer-management"></a>Gestion des offres et de l’expérience client

Quand un client déploie votre offre, il est en mesure de déléguer des abonnements ou groupes de ressources pour la [gestion des ressources déléguées Azure](../../lighthouse/concepts/azure-delegated-resource-management.md). Pour plus d’informations sur ce processus, consultez [Processus d’intégration des clients](../../lighthouse/how-to/publish-managed-services-offers.md#the-customer-onboarding-process).

Vous pouvez [publier une version mise à jour de votre offre](update-existing-offer.md) à tout moment. Par exemple, vous pouvez souhaiter ajouter une nouvelle définition de rôle à une offre publiée précédemment. Dans ce cas, les clients qui ont déjà ajouté l’offre voient s’afficher une icône sur la page [**Fournisseurs de services**](../../lighthouse/how-to/view-manage-service-providers.md) du Portail Azure, ce qui leur permet de savoir qu’une mise à jour est disponible. Chaque client pourra passer en revue les modifications et décider s’il souhaite effectuer la mise à jour vers la nouvelle version.

## <a name="next-steps"></a>Étapes suivantes

- [Mettre à jour une offre existante dans la Place de marché commerciale](./update-existing-offer.md)
- [Découvrir Azure Lighthouse](../../lighthouse/overview.md)