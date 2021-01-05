---
title: Créer une offre d’application Power BI dans Microsoft AppSource
description: Découvrez comment créer une offre d’application Power BI et la publier sur Microsoft AppSource.
author: navits09
ms.author: navits
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 07/22/2020
ms.openlocfilehash: bff20468e8185073f5c192c1e115bc405dd089eb
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693615"
---
# <a name="create-a-power-bi-app-offer"></a>Créer une offre d’application Power BI

Cet article explique comment créer une offre d’application Power BI et la publier sur [Microsoft AppSource](https://appsource.microsoft.com/).

Avant de commencer, si ce n’est déjà fait, [créez un compte de la Place de marché commerciale dans l’Espace partenaires](create-account.md). Vérifiez que votre compte est inscrit dans le programme du marketplace commercial.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le menu de navigation de gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.
3. Dans la page de présentation, sélectionnez **+ Nouvelle offre** > **Application de service Power BI**.

   ![Illustre le menu de navigation de gauche.](./media/new-offer-power-bi-app.png)

> [!NOTE]
> Après la publication d’une offre, les modifications qui lui ont été apportées dans l’Espace partenaires n’apparaissent dans les magasins en ligne qu’après republication de l’offre. Veillez à toujours effectuer une nouvelle publication après avoir apporté des modifications.

> [!IMPORTANT]
> Si l’**Application de service Power BI** n’est pas affichée ou activée, votre compte n’est pas autorisé à créer ce type d’offre. Vérifiez que vous remplissez toutes les [exigences](create-power-bi-app-overview.md) pour ce type d’offre, notamment l’inscription d’un compte de développeur.

## <a name="new-offer"></a>Nouvelle offre

Entrez un **ID d’offre**. Il s’agit d’un identificateur unique par offre dans votre compte.

- Cet ID est visible par les clients dans l’adresse web de l’offre de la Place de marché et des modèles Resource Manager, le cas échéant.
- Utilisez uniquement des lettres minuscules et des chiffres. Il peut inclure des traits d’union et des traits de soulignement, mais pas d’espaces, et est limité à 50 caractères. Par exemple, si vous entrez **test-offer-1** ici, l’adresse web de l’offre sera `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- L’ID d’offre ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

- Ce nom n’est pas utilisé dans la Place de marché et est différent du nom de l’offre et des autres informations présentées aux clients.
- L’alias d’offre ne peut pas être modifié une fois que vous avez sélectionné **Créer**.

Sélectionnez **Créer** pour générer l’offre et continuer.

## <a name="offer-overview"></a>Vue d’ensemble de l’offre

La page montre une représentation visuelle des étapes nécessaires à la publication de cette offre (à la fois terminées et à venir), et indique la durée d’exécution de chaque étape.

Elle contient des liens permettant d’effectuer des opérations sur cette offre en fonction de la sélection que vous effectuez. Par exemple :

- Si l’offre est un brouillon : Supprimer le brouillon de l’offre
- Si l’offre est publiée : [Arrêter de vendre l’offre](update-existing-offer.md#stop-selling-an-offer-or-plan)
- Si l’offre est disponible en préversion : [Mise en service](../review-publish-offer.md#previewing-and-approving-your-offer)
- Si vous n’avez pas complété Déconnexion de l’éditeur : [Annuler la publication](../review-publish-offer.md#cancel-publishing)

## <a name="offer-setup"></a>Configuration de l’offre

### <a name="customer-leads"></a>Prospects

Lors de la publication de votre offre sur la Place de marché avec l’Espace partenaires, vous devez la connecter à votre système de gestion des relations avec la clientèle (CRM). Cela vous permet de recevoir les coordonnées des clients dès qu’une personne exprime son intérêt ou utilise votre produit.

1. Sélectionnez une destination de prospect afin de diriger les prospects vers le système de votre choix. L’Espace partenaires prend en charge les systèmes CRM suivants :

    - [Dynamics 365](commercial-marketplace-lead-management-instructions-dynamics.md) for Customer Engagement
    - [Marketo](commercial-marketplace-lead-management-instructions-marketo.md)
    - [Salesforce](commercial-marketplace-lead-management-instructions-salesforce.md)

    > [!NOTE]
    > Si votre système CRM ne figure pas dans la liste, utilisez une [table Azure](commercial-marketplace-lead-management-instructions-azure-table.md) ou un [point de terminaison HTTPS](commercial-marketplace-lead-management-instructions-https.md) pour stocker les données des prospects. Ensuite, exportez les données vers votre système CRM.

2. Connectez votre offre à la destination de prospect tout en la publiant dans l’Espace partenaires.
3. Vérifiez que la connexion à la destination de prospect est correctement configurée. Une fois que vous avez effectué la publication dans l’Espace partenaires, nous validons la connexion et nous vous envoyons un prospect de test. Quand vous visualisez l’offre avant son lancement, vous pouvez également tester votre connexion de prospect en essayant d’acheter vous-même l’offre dans l’environnement en version préliminaire.
4. Assurez-vous que la connexion à la destination de prospect reste à jour afin de ne perdre aucun prospect.

Voici des ressources de gestion des prospects supplémentaires :

- [Prospects de votre offre de marketplace commercial](commercial-marketplace-get-customer-leads.md)
- [Questions courantes sur la gestion des prospects](../lead-management-faq.md#common-questions-about-lead-management)
- [Résolution des erreurs de configuration des prospects](../lead-management-faq.md#publishing-config-errors)
- PDF [Vue d’ensemble de la gestion des prospects](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf) (assurez-vous que votre bloqueur de fenêtres publicitaires est désactivé)

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="properties"></a>Propriétés

C’est dans cette page que vous définissez les catégories et secteurs utilisés pour grouper votre offre selon les Places de marché, la version de votre application et les contrats légaux associés.

### <a name="category"></a>Category

Sélectionnez les catégories et les sous-catégories pour placer votre offre dans les zones de recherche appropriées sur la Place de marché. Veillez à décrire l’adéquation de votre offre à ces catégories dans la description de l’offre. Sélectionnez :

- Au moins une et jusqu’à deux catégories, y compris une catégorie principale et une catégorie secondaire (facultatif).
- Jusqu’à deux sous-catégories pour chaque catégorie principale et/ou secondaire. Si aucune sous-catégorie n’est applicable à votre offre, sélectionnez **Non applicable**.

Consultez la liste complète des catégories et sous-catégories dans [Bonnes pratiques pour le référencement des offres](../gtm-offer-listing-best-practices.md).

### <a name="industry"></a>Secteur d’activité

[!INCLUDE [Industry Taxonomy](./includes/industry-taxonomy.md)]

### <a name="legal"></a>Informations juridiques

#### <a name="terms-and-conditions"></a>Conditions générales

Pour fournir vos propres conditions générales, entrez jusqu’à 10 000 caractères dans la zone **Conditions générales**. Les clients doivent accepter ces conditions avant de pouvoir essayer votre application.

Sélectionnez **Enregistrer le brouillon** avant de passer à la section suivante, Annonce de l’offre.

## <a name="offer-listing"></a>Annonce de l’offre

Ici, vous allez définir les détails de l’offre qui s’affichent dans la Place de marché. Cela comprend le nom, la description, les images, etc. de l’offre.

### <a name="language"></a>Langage

Sélectionnez la langue dans laquelle votre offre sera listée. Actuellement, **Anglais (États-Unis)** est la seule option disponible.

Définissez les informations de la Place de marché (telles que le nom de l’offre, la description et les images) pour chaque langue/marché. Sélectionnez le nom de la langue/du marché pour fournir ces informations.

> [!NOTE]
> Les détails de l’offre n’ont pas à être en anglais si la description de l’offre commence par l’expression « This application is available only in [langue non anglaise] ». Il est également possible de fournir un lien utile pour offrir du contenu dans une langue autre que celle utilisée dans la description de l’offre.

Voici un exemple de la façon dont les informations de l’offre s’affichent dans Microsoft AppSource (les prix listés sont fournis à titre d’exemple uniquement et ne reflètent pas les coûts réels) :

:::image type="content" source="media/example-power-bi-app.png" alt-text="Illustre la façon dont cette offre s’affiche dans Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Logo
2. Products
3. Catégories
4. Secteurs d’activité
5. Adresse du support technique (lien)
6. Conditions d’utilisation
7. Politique de confidentialité
8. Nom de l’offre
9. Résumé
10. Description
11. Captures d’écran/vidéos

### <a name="name"></a>Nom

Le nom que vous entrez ici s’affiche comme titre de votre offre. Ce champ est prérempli avec le texte que vous avez entré dans la zone **Alias d’offre** lors de la création de l’offre. Vous pouvez changer ce nom ultérieurement.

Le nom :

- Peut être une marque déposée (et vous pouvez inclure des symboles de marque ou de copyright).
- Ne peut pas compter plus de 50 caractères.
- Ne peut pas inclure d’emojis.

### <a name="search-results-summary"></a>Résumé des résultats de recherche

Fournissez une brève description de votre offre. Ce champ peut comporter jusqu’à 100 caractères, et est utilisé dans les résultats de recherche de la Place de marché.

### <a name="description"></a>Description

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

### <a name="search-keywords"></a>Mots clés de recherche

Entrez jusqu’à trois mots clés de recherche facultatifs pour aider les clients à trouver votre offre sur la Place de marché. Pour obtenir des résultats optimaux, utilisez également ces mots clés dans votre description.

### <a name="helpprivacy-web-addresses"></a>Adresses web d’aide/confidentialité

Fournissez des liens pour aider les clients à mieux comprendre votre offre.

#### <a name="help-link"></a>Lien d'aide

Entrez l’adresse web à partir de laquelle les clients peuvent en savoir plus sur votre offre.

#### <a name="privacy-policy-url"></a>URL de la politique de confidentialité

Entrez l’adresse web de la politique de confidentialité de votre organisation. Vous devez veiller à ce que votre offre respecte les lois et les réglementations en matière de confidentialité. Vous devez également veiller à publier une politique de confidentialité conforme sur votre site web.

### <a name="contact-information"></a>Informations de contact

Entrez le nom, l’adresse e-mail et le numéro de téléphone d’un **contact du support** et d’un **contact en ingénierie**. Ces informations ne sont pas présentées aux clients. Elles sont disponibles pour Microsoft et peuvent être fournies aux partenaires fournisseurs de solutions Cloud (CSP).

- Contact du support (obligatoire) : Pour toute question de support générale.
- Contact en ingénierie (obligatoire) : Pour les questions techniques et les problèmes de certification.
- Contact du programme du fournisseur de solutions cloud (facultatif) : Pour toute question du revendeur relative au programme CSP.

Dans la section **Contact du support**, indiquez l’adresse du **site web du support** où les partenaires peuvent trouver l’assistance pour votre offre.

### <a name="supporting-documents"></a>Documents de soutien

Fournissez au moins un et au plus trois documents marketing associés au format PDF. Par exemple des livres blancs, des brochures, des check-lists ou des présentations.

### <a name="marketplace-images"></a>Images de la Place de marché

Fournissez des logos et images à utiliser avec votre offre. Toutes les images doivent être au format .png. Les images floues seront rejetées.

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si vous rencontrez un problème de téléchargement de fichiers, assurez-vous que votre réseau local ne bloque pas le service `https://upload.xboxlive.com` utilisé par l’espace partenaires.

#### <a name="store-logos"></a>Stocker des logos

Fournissez un fichier PNG pour le logo de **grande taille**. L'Espace partenaires l'utilisera pour créer un **Petit** logo. Plus tard, vous pourrez éventuellement le remplacer par une autre image.

- **Grande taille** (de 216 x 216 à 350 x 350 px, obligatoire)
- **Petite taille** (48 x 48 px, facultatif)

Ces logos sont utilisés à différents emplacements dans la liste :

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Captures d’écran.

Ajoutez au moins une et au plus cinq captures d’écran qui illustrent le fonctionnement de votre offre. Chacune doit avoir une taille de 1280 x 720 pixels et être au format .png.

#### <a name="videos-optional"></a>Vidéos (facultatif)

Ajoutez jusqu’à cinq vidéos qui illustrent votre offre. Entrez le nom de la vidéo, son adresse web et une image miniature PNG de 1280 x 720 pixels de la vidéo.

#### <a name="additional-marketplace-listing-resources"></a>Ressources d’annonce supplémentaires de place de marché

Pour en savoir plus sur la création d’annonces d’offres, consultez [Bonnes pratiques pour le référencement des offres](../gtm-offer-listing-best-practices.md).

## <a name="technical-configuration"></a>Configuration technique

Promouvez votre application dans le service Power BI en production et fournissez le lien du programme d’installation Power BI qui permet aux clients d’installer votre application. Pour plus d’informations, consultez [Publish apps with dashboards and reports in Power BI](/power-bi/service-create-distribute-apps) (Publier des applications avec tableaux de bord et rapports dans Power BI).

## <a name="supplemental-content"></a>Contenu supplémentaire

Fournissez des informations supplémentaires sur votre offre pour que nous puissions la valider. Ces informations ne sont ni montrées aux clients ni publiées sur la Place de marché.

### <a name="validation-assets"></a>Ressources de validation

Éventuellement, ajoutez des instructions (jusqu’à 3 000 caractères) pour aider l’équipe de validation de Microsoft à configurer votre application, s’y connecter et la tester. Insérez des paramètres de configuration standard, des comptes, des paramètres ou d’autres informations qui peuvent être utilisés pour tester l’option Connecter des données. Ces informations sont uniquement visibles par l’équipe de validation et sont dédiées à la validation.

## <a name="review-and-publish"></a>Vérifier et publier

Une fois que vous avez rempli toutes les sections requises de l’offre, vous pouvez envoyer votre pour révision et publication.

Dans le coin supérieur droit du portail, sélectionnez **Vérifier et publier**.

Dans la page de révision, vous pouvez :

- Consultez l’état d’achèvement de chaque section de l’offre. Vous ne pouvez pas publier tant que toutes les sections de l’offre ne sont pas marquées comme étant complètes.
  - **Non démarrée** : la section n’a pas été démarrée et doit être complétée.
  - **Incomplète** : la section comporte des erreurs qui doivent être corrigées ou nécessite plus d’informations de votre part. Pour obtenir de l’aide, consultez les sections plus haut dans ce document.
  - **Complète** : la section contient toutes les données nécessaires et ne comporte aucune erreur. Toutes les sections de l’offre doivent être complètes pour que vous puissiez envoyer l’offre.
- Fournissez des instructions de test à l’équipe de certification pour vous assurer que votre application est correctement testée. Vous pouvez également fournir des notes supplémentaires utiles pour comprendre votre offre.

Pour envoyer l’offre à des fins de publication, sélectionnez **Publier**.

Nous vous enverrons un e-mail pour vous faire savoir si une préversion de l’offre est disponible pour vérification et approbation. Pour publier votre offre au public, accédez à l’Espace partenaires, puis sélectionnez **Mise en service**.
