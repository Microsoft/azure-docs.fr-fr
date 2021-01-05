---
title: Créer une offre Dynamics 365 Business Central dans Microsoft AppSource
description: Procédure de création d’une offre Dynamics 365 pour Business Central dans Microsoft AppSource. Répertoriez ou vendez votre offre dans AppSource ou via le programme Fournisseur de solutions Cloud (CSP).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: navits09
ms.author: navits
ms.date: 12/02/2020
ms.openlocfilehash: 99c36354334701dcd4c7c30c5fd5039c13885525
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358705"
---
# <a name="create-a-dynamics-365-for-business-central-offer"></a>Créer une offre Dynamics 365 pour Business Central

Cet article explique la procédure de création d’une offre Dynamics 365 Business Central. [Microsoft Dynamics 365 Business Central](https://dynamics.microsoft.com/business-central) est un service de planification des ressources d’entreprise (ERP) qui gère un vaste éventail de processus d’entreprise, notamment en lien avec les finances, les opérations, la chaîne logistique, la gestion de la relation client, la gestion de projets et le commerce électronique. Les packages Premium prennent également en charge le modèle de déploiement classique et la fabrication. Toutes les offres pour Dynamics 365 Business Central doivent passer par notre processus de certification.

Avant de commencer, et si vous ne l’avez pas déjà fait, [créez un compte Marketplace commercial dans l’Espace partenaires](create-account.md). Vérifiez que votre compte est inscrit dans le programme du marketplace commercial.

>[!NOTE]
> Une fois l’offre publiée, les modifications apportées à l’offre sont uniquement mises à jour dans l’Espace partenaires et le magasin en ligne une fois que vous avez renvoyé l’offre en vue de sa publication.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le menu de navigation de gauche, sélectionnez **Marketplace commercial** > **Vue d’ensemble**.
3. Dans la page Vue d’ensemble, sélectionnez **+ Nouvelle offre** > **Dynamics 365 Business Central**.

    ![Illustre le menu de navigation de gauche.](./media/new-offer-dynamics-365-business-central.png)

## <a name="new-offer"></a>Nouvelle offre

Entrez un **ID d’offre**. Il s’agit d’un identificateur unique par offre dans votre compte.

- Cet ID est visible par les clients dans l’adresse web de l’offre de la Place de marché et des modèles Resource Manager, le cas échéant.
- L’ID d’offre associé à l’ID d’éditeur doit contenir moins de 40 caractères.
- Utilisez uniquement des lettres minuscules et des chiffres. Il peut contenir des traits d’union et des traits de soulignement, mais pas d’espaces. Par exemple, si votre ID d’éditeur est `testpublisherid` et que vous entrez **test-offer-1**, l’adresse web de l’offre sera `https://appsource.microsoft.com/product/dynamics-365/testpublisherid.test-offer-1`.
- Cet ID ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

- Ce nom n’est pas utilisé dans la Place de marché et est différent du nom de l’offre et des autres informations présentées aux clients.
- Vous ne pouvez pas modifier ce nom après avoir sélectionné **Créer**.

Sélectionnez **Créer** pour générer l’offre et continuer.

## <a name="offer-setup"></a>Configuration de l’offre

### <a name="alias"></a>Alias

Entrez un nom descriptif que nous utiliserons pour faire référence à cette offre uniquement dans l’Espace partenaires. Ce nom (pré-rempli avec les informations que vous avez entrées lors de la création de l’offre) n’est pas utilisé dans la marketplace et est différent du nom de l’offre présenté aux clients. Si vous souhaitez mettre à jour le nom de l’offre par la suite, accédez à la page [Annonce de l’offre](#offer-listing).

### <a name="setup-details"></a>Détails de la configuration

Sélectionnez le **Type de package** qui s’applique à votre offre :

- Une application **Module complémentaire** étend l’expérience et les fonctionnalités existantes de Dynamics 365 Business Central. Pour plus d’informations, consultez l’article [Applications de module complémentaire](/dynamics365/business-central/dev-itpro/developer/readiness/readiness-add-on-apps).
- Une application de **connexion** peut être utilisée dans le scénario où une connexion point à point doit être établie entre Dynamics 365 Business Central et une solution ou un service tiers. Pour plus d’informations, consultez l’article [Applications de connexion](/dynamics365/business-central/dev-itpro/developer/readiness/readiness-connect-apps).

Pour **Comment voulez-vous qu’interagissent les clients avec l’offre de liste ?** , sélectionnez l’option que vous souhaitez utiliser pour cette offre.

- **Obtenir maintenant (gratuit)**  : référencez votre offre gratuitement pour les clients.
- **Essai gratuit (annonce)**  : référencez votre offre pour les clients avec un lien vers un essai gratuit. Les versions d’évaluation gratuites sont créées, gérées et configurées par votre service et n’ont pas d’abonnements gérés par Microsoft.

    > [!NOTE]
    > Les jetons que votre application recevra via votre lien d’évaluation peuvent uniquement être utilisés pour obtenir des informations utilisateur par le biais de Azure Active Directory (Azure AD) pour automatiser la création de comptes dans votre application. Les comptes Microsoft ne sont pas pris en charge pour l’authentification à l’aide de ce jeton.

- **Me contacter** : collectez des informations de contact client en connectant votre système de gestion de la relation client (CRM). Le client devra autoriser le partage de ses informations. Ces informations client, ainsi que le nom de l’offre, son ID et la place de marché sur laquelle il a trouvé votre offre, seront envoyées au système CRM que vous avez configuré. Pour plus d’informations sur la configuration de votre CRM, consultez [Prospects](#customer-leads).

### <a name="test-drive"></a>Test drive

Une version d’évaluation est un bon moyen de présenter votre offre à vos clients potentiels en leur donnant la possibilité de l’« essayer avant de l’acheter », ce qui se traduit par une hausse du taux de conversion et par la génération de prospects de qualité. Pour plus d’informations, commencez par [Qu’est-ce qu’un test drive](../what-is-test-drive.md).

Pour activer une version d’évaluation pour une durée déterminée, cochez la case **Activer une version d’évaluation**. Pour supprimer une version d'évaluation de votre offre, désactivez cette case à cocher.

### <a name="customer-leads"></a>Prospects

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Pour plus d’informations, consultez [Vue d’ensemble de la gestion des prospects](./commercial-marketplace-get-customer-leads.md).

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="properties"></a>Propriétés

Cette page vous permet de définir les catégories et les secteurs d’activité auxquels votre offre est destinée sur la Place de marché, la version de votre application et les contrats juridiques accompagnant votre offre.

### <a name="categories"></a>Catégories

Sélectionnez les catégories et les sous-catégories pour placer votre offre dans les zones de recherche appropriées sur la Place de marché. Veillez à décrire l’adéquation de votre offre à ces catégories dans la description de l’offre. Sélectionnez :

- Au moins une et jusqu’à deux catégories, y compris une catégorie principale et une catégorie secondaire (facultatif).
- Jusqu’à deux sous-catégories pour chaque catégorie principale et/ou secondaire. Si aucune sous-catégorie n’est applicable à votre offre, sélectionnez **Non applicable**.

Consultez la liste complète des catégories et sous-catégories dans [Bonnes pratiques pour le référencement des offres](../gtm-offer-listing-best-practices.md).

### <a name="industries"></a>Secteurs d’activité

[!INCLUDE [Industry Taxonomy](includes/industry-taxonomy.md)]

### <a name="app-version"></a>Version de l’application

Entrez le numéro de version de votre offre. Cette version est indiquée aux clients sur la page de détails de l’offre.

### <a name="terms-and-conditions"></a>Conditions générales

Indiquez ici vos propres conditions générales d’ordre juridique. Vous pouvez aussi indiquer l’adresse où se trouvent vos conditions générales. Les clients doivent accepter ces conditions avant de pouvoir essayer votre offre.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="offer-listing"></a>Annonce de l’offre

Cette page vous permet de définir les détails de l’offre, notamment le nom de l’offre, la description, les liens et les contacts.

> [!NOTE]
> Fournissez les détails de l’offre dans une seule langue uniquement. Les détails de l’offre n’ont pas à être en anglais si la description de l’offre commence par l’expression « This application is available only in [langue non anglaise] ». Vous pouvez également de fournir une *URL de lien utile* pour offrir du contenu dans une langue autre que celle utilisée dans le contenu du référencement de l’offre.

Voici un exemple de la façon dont les informations de l’offre s’affichent dans Microsoft AppSource (les prix répertoriés sont fournis à titre d’exemple uniquement et ne reflètent pas les coûts réels) :
<!-- update screen? -->
:::image type="content" source="media/example-d365-business-central.png" alt-text="Illustre la façon dont cette offre s’affiche dans Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Logo
2. Products
3. Catégories
4. Adresse du support technique (lien)
5. Conditions d’utilisation
6. Politique de confidentialité
7. Nom de l’offre
8. Résumé
9. Description
10. Captures d’écran/vidéos

### <a name="marketplace-details"></a>Détails de la Place de marché

Le **nom** que vous entrez ici s’affichera auprès des clients comme titre de votre référencement d’offre. Ce champ est prérempli avec le texte que vous avez entré pour **Alias d’offre** lorsque vous avez créé l’offre, mais vous pouvez modifier cette valeur. Ce nom peut être une marque déposée (et vous pouvez inclure des symboles de marque ou de copyright). Le nom ne peut pas comporter plus de 50 caractères et ne peut pas contenir d’emoji.

Entrez une courte description de votre offre (100 caractères maximum) pour le **résumé des résultats de la recherche**. Cette description pourra être utilisée dans les résultats de recherche de la place de marché.

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Rich text editor](./includes/rich-text-editor.md)]

Vous pouvez éventuellement entrer jusqu’à trois **mots clés de recherche** pour aider les clients à trouver votre offre sur la marketplace. Pour obtenir des résultats optimaux, utilisez également ces mots clés dans votre description.

Si vous souhaitez permettre aux clients de savoir avec **quels produits spécifiques votre application fonctionne**, saisissez jusqu’à trois noms de produit.

### <a name="helpprivacy-urls"></a>URL Aide et URL Confidentialité

Entrez le **lien (URL) vers l’aide de votre application** où les clients peuvent en savoir plus sur votre offre. L’URL d’aide ne peut pas être la même que l’URL de support.

Entrez le **lien (URL) vers la politique de confidentialité** de votre organisation. Vous devez veiller à ce que votre application respecte les lois et réglementations en matière de confidentialité et à renseigner une politique de confidentialité conforme.

### <a name="contact-information"></a>Informations de contact

Entrez le nom, l’adresse e-mail et le numéro de téléphone d’un **contact du support** et d’un **contact en ingénierie**. Ces informations ne sont pas présentées aux clients, mais seront disponibles pour Microsoft et pourront être fournies aux partenaires CSP.

Dans la section **Contact de support**, indiquez **l’URL du support** où les partenaires CSP peuvent trouver l’assistance pour votre offre. L’URL du support ne peut pas être la même que l’URL d’aide.

### <a name="supporting-documents"></a>Documents de soutien

Fournissez ici entrez un et trois documents marketing d’accompagnement, comme des livres blancs, des brochures, des listes de contrôle ou des présentations, au format PDF.

### <a name="marketplace-media"></a>Médias de la Place de marché

Fournissez des logos et des images qui seront utilisés pour montrer votre offre aux clients. Toutes les images doivent être au format .png.

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

>[!Note]
>Si vous rencontrez un problème de chargement de fichiers, vérifiez que votre réseau local ne bloque pas le service https://upload.xboxlive.com utilisé par l’Espace partenaires.

#### <a name="logos"></a>Logos

Fournissez un fichier PNG pour le logo de **Grande taille**. L'Espace partenaires l'utilisera pour créer d’autres tailles requises. Plus tard, vous pourrez éventuellement le remplacer par une autre image.

Ces logos sont utilisés à différents emplacements dans la liste :

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Captures d’écran.

Ajoutez des captures d’écran qui illustrent le fonctionnement de votre offre. Au moins trois captures d’écran sont requises, et vous pouvez en ajouter jusqu’à cinq. Toutes les captures d’écran doivent être au format PNG en 1280 x 720 pixels.

#### <a name="videos"></a>Vidéos

Vous pouvez éventuellement ajouter jusqu’à quatre vidéos qui illustrent votre offre. Les vidéos doivent être hébergées sur un site externe. Pour chaque vidéo, entrez son nom, son adresse et une image miniature correspondante (1280 x 720 pixels).

Pour obtenir des ressources supplémentaires concernant la place de marché, consultez [Meilleures pratiques pour les annonces d’offre d’une place de marché](../gtm-offer-listing-best-practices.md).

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="availability"></a>Disponibilité

Cette page vous permet de définir où et comment mettre votre offre à disposition.

### <a name="markets"></a>Marchés

Pour spécifier les marchés sur lesquels votre offre doit être disponible, sélectionnez **Modifier les marchés** pour afficher la fenêtre contextuelle **Sélection du marché**.

Sélectionnez au moins un marché. Choisissez **Sélectionner tout** pour mettre votre offre à disposition sur tous les marchés possibles ou sélectionnez uniquement des marchés spécifiques. Quand vous avez terminé, Sélectionnez **Enregistrer**.

Vos sélections ici s’appliquent uniquement aux nouvelles acquisitions. Si quelqu’un a déjà votre application sur un marché donné et que vous supprimez par la suite ce marché, les personnes qui disposent déjà de cette offre peuvent continuer à l’utiliser, mais aucun nouveau client sur ce marché ne pourra bénéficier de votre offre.

> [!IMPORTANT]
> Il vous incombe de respecter les exigences légales locales, même si ces exigences ne sont pas répertoriées ici ou dans l’Espace partenaires. Même si vous sélectionnez tous les marchés, les lois et restrictions locales ou d’autres facteurs peuvent empêcher certaines offres d’être répertoriées dans certains pays et régions.

### <a name="preview-audience"></a>Public de la version préliminaire

Avant de publier votre offre sur la place de marché générale, vous devez d’abord la mettre à disposition d’un **public limité en préversion**. Entrez une **Clé de masquage** (n’importe quelle chaîne utilisant uniquement des lettres minuscules et/ou des chiffres) ici. Les membres de votre public en préversion peuvent utiliser cette clé de masquage comme jeton pour afficher un aperçu de votre offre sur la place de marché.

Ensuite, lorsque vous êtes prêt à mettre votre offre à disposition générale et supprimer la restriction de la préversion, vous devez supprimer la **clé de masquage** et publier à nouveau.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="technical-configuration"></a>Configuration technique

Cette page définit les détails techniques utilisés pour se connecter à votre offre. Cette connexion permet d’approvisionner votre offre pour le client final, s’il choisit de l’acquérir.

### <a name="file-upload"></a>Chargement de fichiers

Si vous avez précédemment sélectionné **Module complémentaire**, vous allez charger le fichier de package de votre offre, ainsi que les fichiers de package pour toute extension sur laquelle il a des dépendances.

#### <a name="extension-package-file"></a>Fichier de package d’extension

Téléchargez le fichier de package d’extension (.app) pour votre offre.

#### <a name="library-extension-package-file"></a>Fichier de package d’extension de bibliothèque

Obligatoire si votre offre doit être installée avec une autre extension qui n’est pas publiée sur la place de marché. Si c’est le cas, téléchargez le fichier .app ici.

>[!NOTE]
>Le fichier de package de dépendance n'est plus utilisé. Chargez un fichier de package d’extension de bibliothèque à la place.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

<!-- ## Test drive technical configuration

This page lets you set up a demonstration ("test drive") that allows customers to try your offer before purchasing it. Learn more in [What is test drive](../what-is-test-drive.md).

To enable a test drive, select the **Enable a test drive** check box on the [Offer setup](#test-drive) tab. To remove test drive from your offer, clear this check box.

When you've finished setting up your test drive, select **Save draft** before continuing.
-->
## <a name="supplemental-content"></a>Contenu supplémentaire

Cette page vous permet de fournir des informations supplémentaires pour que nous puissions valider votre offre. Ces informations ne sont pas montrées aux clients ni publiées sur la Place de marché.

### <a name="target-release"></a>Version cible

Indiquez la version de Microsoft Dynamics Business Central ciblée par votre solution : **Actuelle**, **Majeure suivante** ou **Mineure suivante**. Ces informations nous permettent de tester votre solution de manière appropriée.

### <a name="supported-editions"></a>Éditions prises en charge

Si votre offre nécessite l’édition Premium de Microsoft Dynamics 365 Business Central, sélectionnez **Premium** uniquement. Sinon, sélectionnez à la fois **Essentials** et **Premium**.

### <a name="key-usage-scenario"></a>Principaux scénarios d’utilisation

Chargez un fichier PDF qui liste les scénarios d’utilisation clés de votre offre. Tous les scénarios répertoriés ici peuvent être vérifiés par notre équipe de validation avant que nous approuvions votre offre pour la place de marché.

### <a name="test-accounts"></a>Comptes de test

Si un compte de test est nécessaire pour que notre équipe de certification puisse examiner correctement votre offre, vous devrez charger un fichier au format .pdf, .doc ou .docx avec les informations de vos **Comptes de test**.

### <a name="app-tests-automation"></a>Automatisation des tests d’application

Si votre offre est une application de module complémentaire, vous devez charger un fichier d’**automatisation des tests d’application** (.app). Ce fichier ne s’applique pas aux applications de connexion.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="publish"></a>Publish

### <a name="submit-offer-to-preview"></a>Soumettre l’offre en préversion

Après avoir complété toutes les sections obligatoires de l’offre, sélectionnez **Vérifier et publier** en haut à droite du portail.

S’il s’agit de votre première publication de cette offre, vous pouvez :

- Consultez l’état d’achèvement de chaque section de l’offre.
    - **Non démarrée** : la section n’a pas encore été modifiée et doit être complétée.
    - **Incomplète** : la section comporte des erreurs qui doivent être corrigées ou nécessite un complément d’informations. Revenir à la ou les sections et les mettre à jour.
    - **Complète** : la section est complète, toutes les données obligatoires ont été fournies et aucune erreur n’est présente. Toutes les sections de l’offre doivent être dans l’état Complète avant de pouvoir envoyer l’offre.
- Dans la sections **Notes pour la certification**, fournissez des instructions de test à l’équipe de certification pour veiller à ce que votre application soit testée correctement, en plus de toutes autres notes supplémentaires utiles pour comprendre l’application.
- Envoyez l’offre pour publication en sélectionnant **Envoyer**. Nous vous enverrons un e-mail quand une préversion de l’offre sera disponible pour examen et approbation. Retournez dans l’Espace partenaires et sélectionnez **Mise en service** pour publier votre offre au public.

## <a name="next-steps"></a>Étapes suivantes

- [Mettre à jour une offre existante dans la Place de marché commerciale](./update-existing-offer.md)