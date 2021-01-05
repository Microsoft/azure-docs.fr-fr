---
title: Créer une offre Dynamics 365 for Operations sur la Place de marché commerciale
description: Comment créer une nouvelle offre Dynamics 365 for Operations pour répertorier ou vendre dans la Place de marché Azure, AppSource ou via le programme de fournisseur de solutions Cloud (CSP) avec le portail de la Place de marché commerciale sur Microsoft Partner Center.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: navits09
ms.author: navits
ms.date: 06/17/2020
ms.openlocfilehash: 1c28b4edab1e1033bf9ac7df6b1249b549bb4fe1
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93131511"
---
# <a name="create-a-dynamics-365-for-operations-offer"></a>Créer une offre Dynamics 365 for Operations

Cette rubrique explique comment créer une nouvelle offre Dynamics 365 for Operations. [Microsoft Dynamics 365 for Finance and Operations](https://dynamics.microsoft.com/finance-and-operations) est un service de type progiciel de gestion intégré (ERP) qui gère les aspects avancés de la finance, des opérations, de la fabrication et de la gestion de la chaîne logistique. Toutes les offres pour Dynamics 365 for Operations doivent passer par notre processus de certification.

Avant de commencer, [créez un compte Place de marché commerciale dans l’Espace partenaires](create-account.md) si vous ne l’avez pas déjà fait. Vérifiez que votre compte est inscrit dans le programme Place de marché commerciale.

>[!NOTE]
> Une fois l’offre publiée, les modifications apportées à l’offre sont uniquement mises à jour dans l’espace partenaires et le magasin en ligne une fois que vous avez envoyé à nouveau l’offre en vue de sa publication.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le menu de navigation de gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.
3. Dans la page Vue d’ensemble, sélectionnez **+ Nouvelle offre** > **Dynamics 365 for Operations**.

    ![Illustre le menu de navigation de gauche.](./media/new-offer-dynamics-365-operations.png)

> [!NOTE]
> Après la publication d’une offre, les modifications qui lui ont été apportées dans l’Espace partenaires n’apparaissent dans les magasins en ligne qu’après republication de l’offre. Veillez à toujours effectuer une nouvelle publication après avoir apporté des modifications.

## <a name="new-offer"></a>Nouvelle offre

Entrez un **ID d’offre**. Il s’agit d’un identificateur unique par offre dans votre compte.

- Cet ID est visible par les clients dans l’adresse web de l’offre de la Place de marché et des modèles Resource Manager, le cas échéant.
- L’ID d’offre associé à l’ID d’éditeur doit contenir moins de 40 caractères.
- Utilisez uniquement des lettres minuscules et des chiffres. Il peut contenir des traits d’union et des traits de soulignement, mais pas d’espaces. Par exemple, si votre ID d’éditeur est testpublisherid et que vous entrez **test-offer-1** ici, l’adresse web de l’offre sera `https://appsource.microsoft.com/product/dynamics-365/testpublisherid.test-offer-1`.
- L’ID d’offre ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

- Ce nom n’est pas utilisé dans la Place de marché et est différent du nom de l’offre et des autres informations présentées aux clients.

Sélectionnez **Créer** pour générer l’offre et continuer.

## <a name="offer-setup"></a>Configuration de l’offre

### <a name="how-do-you-want-potential-customers-to-interact-with-this-listing-offer"></a>Comment voulez-vous qu’interagissent les clients avec l’offre de liste ?

Sélectionnez l’option que vous souhaitez utiliser pour cette offre.

#### <a name="get-it-now-free"></a>Obtenez-en une maintenant (gratuit)

Référencez votre offre gratuitement pour les clients.

#### <a name="free-trial-listing"></a>Essai gratuit (annonce)

Référencez votre offre pour les clients avec un lien vers un essai gratuit. Les versions d’évaluation gratuites sont créées, gérées et configurées par votre service et n’ont pas d’abonnements gérés par Microsoft.

> [!NOTE]
> Les jetons que votre application recevra via votre lien d’évaluation peuvent uniquement être utilisés pour obtenir des informations utilisateur par le biais de Azure Active Directory (Azure AD) pour automatiser la création de comptes dans votre application. Les comptes Microsoft ne sont pas pris en charge pour l’authentification à l’aide de ce jeton.

#### <a name="contact-me"></a>Me contacter

Collectez des informations de contact client en connectant votre système de gestion de la relation client (CRM). Le client devra autoriser le partage de ses informations. Ces informations client, ainsi que le nom de l’offre, son ID et la place de marché sur laquelle il a trouvé votre offre, seront envoyées au système CRM que vous avez configuré. Pour plus d’informations sur la configuration de votre CRM, consultez [Prospects](#customer-leads).

### <a name="test-drive"></a>Test drive

Une version d’évaluation est un bon moyen de présenter votre offre à vos clients potentiels en leur donnant la possibilité de l’« essayer avant de l’acheter », ce qui se traduit par une hausse du taux de conversion et par la génération de prospects de qualité. Pour plus d’informations, commencez par [Qu’est-ce qu’un test drive](../what-is-test-drive.md).

Pour activer une version d’évaluation pour une durée déterminée, cochez la case **Activer une version d’évaluation**. Pour supprimer une version d'évaluation de votre offre, désactivez cette case à cocher.

### <a name="customer-leads"></a>Prospects

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Pour plus d’informations, consultez [Vue d’ensemble de la gestion des prospects](./commercial-marketplace-get-customer-leads.md).

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="properties"></a>Propriétés

Cette page vous permet de définir les catégories et les secteurs d’activité auxquels votre offre est destinée sur la Place de marché, la version de votre application et les contrats juridiques accompagnant votre offre.

### <a name="category"></a>Category

Sélectionnez les catégories et les sous-catégories pour placer votre offre dans les zones de recherche appropriées sur la Place de marché. Veillez à décrire l’adéquation de votre offre à ces catégories dans la description de l’offre. Sélectionnez :

- Au moins une et jusqu’à deux catégories, y compris une catégorie principale et une catégorie secondaire (facultatif).
- Jusqu’à deux sous-catégories pour chaque catégorie principale et/ou secondaire. Si aucune sous-catégorie n’est applicable à votre offre, sélectionnez **Non applicable**.

Consultez la liste complète des catégories et sous-catégories dans [Bonnes pratiques pour le référencement des offres](../gtm-offer-listing-best-practices.md).

### <a name="industry"></a>Secteur d’activité

[!INCLUDE [Industry Taxonomy](./includes/industry-taxonomy.md)]

### <a name="app-version"></a>Version de l’application

Entrez le numéro de version de votre offre. Cette version est indiquée aux clients sur la page de détails de l’offre.

### <a name="terms-and-conditions"></a>Conditions générales

Fournissez vos propres conditions légales dans le champ **Conditions générales**. Vous pouvez également fournir l’URL permettant d’accéder à vos conditions générales. Les clients doivent accepter ces conditions avant de pouvoir essayer votre offre.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="offer-listing"></a>Annonce de l’offre

Cette page présente les langues dans lesquelles votre offre sera listée. Actuellement, **Anglais (États-Unis)** est la seule option disponible.

Vous devez définir les informations de la Place de marché (nom de l’offre, description, images, etc.) pour chaque langue/marché. Sélectionnez le nom de la langue/du marché pour fournir ces informations.

> [!NOTE]
> Il n’est pas nécessaire que le contenu du référencement de l’offre (par exemple, la description, les documents, les captures d’écran, les conditions d’utilisation, etc.) soit en anglais, tant que sa description commence par l’expression : « This application is available only in [langue autre que l’anglais]. » Il est également possible de fournir une *URL de lien utile* pour offrir du contenu dans une langue autre que celle utilisée dans le contenu de la liste des offres.

Voici un exemple de la façon dont les informations de l’offre s’affichent dans Microsoft AppSource :

:::image type="content" source="media/example-azure-marketplace-d365-operations.png" alt-text="Illustre la façon dont cette offre s’affiche dans Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Logo
2. Products
3. Catégories
4. Secteurs d’activité
5. Adresse du support technique (lien)
6. Conditions d’utilisation
7. Politique de confidentialité
8. Nom de l’offre
9. Captures d’écran/vidéos
10. Description

### <a name="name"></a>Nom

Le nom que vous entrez ici s’affichera auprès des clients comme titre de votre référencement d’offre. Ce champ est prérempli avec le texte que vous avez entré pour **Alias d’offre** lorsque vous avez créé l’offre, mais vous pouvez modifier cette valeur. Ce nom peut être une marque déposée (et vous pouvez inclure des symboles de marque ou de copyright). Le nom ne peut pas comporter plus de 50 caractères et ne peut pas contenir d’emoji.

### <a name="short-description"></a>Description courte

Fournissez une brève description de votre offre, jusqu’à 100 caractères. Cette description pourra être utilisée dans les résultats de recherche de la place de marché.

### <a name="description"></a>Description

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Rich text editor](./includes/rich-text-editor.md)]

### <a name="search-keywords"></a>Mots clés de recherche

Vous pouvez éventuellement entrer jusqu’à trois mots clés de recherche pour aider les clients à trouver votre offre sur la place de marché. Pour obtenir des résultats optimaux, essayez également d’utiliser ces mots clés dans votre description.

### <a name="products-your-app-works-with"></a>Produits utilisés par votre application

Si vous souhaitez permettre aux clients de savoir que votre application fonctionne avec des produits spécifiques, saisissez jusqu’à trois noms de produit ici.

### <a name="support-urls"></a>URL de support technique

Cette section vous permet de fournir des liens permettant aux clients d’en savoir plus sur votre offre.

#### <a name="help-link"></a>Lien d'aide

Entrez l’URL à partir de laquelle les clients peuvent en savoir plus sur votre offre.

#### <a name="privacy-policy-url"></a>URL de la politique de confidentialité

Entrez l’URL du lien de la politique de confidentialité de votre organisation. Vous devez veiller à ce que votre application respecte les lois et réglementations en matière de confidentialité et à renseigner une politique de confidentialité conforme.

### <a name="contacts"></a>Contacts

Dans cette section, indiquez le nom, l’adresse e-mail et le numéro de téléphone d’un **contact de support** et d’un **contact d’ingénierie**. Ces informations ne sont pas présentées aux clients, mais seront disponibles pour Microsoft et pourront être fournies aux partenaires CSP.

Dans la section **Contact de support** , indiquez **l’URL du support** où les partenaires CSP peuvent trouver l’assistance pour votre offre.

### <a name="supporting-documents"></a>Documents de soutien

Fournissez au moins un (et au plus trois) documents marketing associés ici, comme des livres blancs, des brochures, des listes de contrôle ou des présentations. Ces documents doivent être au format .PDF.

### <a name="marketplace-images"></a>Images de la Place de marché

Dans cette section, vous pouvez fournir des logos et des images qui seront utilisés pour montrer votre offre au client. Toutes les images doivent être au format .png.

[!INCLUDE [logo tips](../includes/graphics-suggestions.md)]

>[!Note]
>Si vous rencontrez un problème de chargement de fichiers, vérifiez que votre réseau local ne bloque pas le service https://upload.xboxlive.com utilisé par l’Espace partenaires.

#### <a name="store-logos"></a>Stocker des logos

Fournissez un fichier PNG pour le logo de **grande taille**. L’Espace partenaires s’en servira pour créer un logo de **petite taille**. Si vous le souhaitez, vous pouvez par la suite le remplacer par une autre image.

- **Grand** (de 216 x 216 à 350 x 350 px, obligatoire)
- **Petit** (48 x 48 px, facultatif)

Ces logos sont utilisés à différents emplacements dans la liste :

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Captures d’écran.

Ajoutez des captures d’écran qui illustrent le fonctionnement de votre offre. Au moins une capture d’écran est requise, et vous pouvez en ajouter jusqu’à cinq. Toutes les captures d’écran doivent être au format 1280 x 720 pixels.

#### <a name="videos"></a>Vidéos

Vous pouvez éventuellement ajouter jusqu’à quatre vidéos qui illustrent votre offre. Ces vidéos doivent être hébergées sur YouTube et/ou Vimeo. Pour chacune d’entre elles, entrez le nom de la vidéo, son URL et une image miniature de la vidéo (1280 x 720 pixels)

#### <a name="additional-marketplace-listing-resources"></a>Ressources d’annonce supplémentaires de place de marché

[Meilleures pratiques pour les annonces d’offre d’une place de marché](../gtm-offer-listing-best-practices.md)

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="availability"></a>Disponibilité

Cette page vous offre des options pour déterminer où et comment rendre votre offre disponible.

### <a name="markets"></a>Marchés

Cette section vous permet de spécifier les marchés dans lesquels votre offre doit être disponible. Pour ce faire, sélectionnez **Edit markets** (Modifier les marchés), ce qui affiche la fenêtre contextuelle **Sélection du marché**.

Par défaut, aucun marché n’est sélectionné. Sélectionnez au moins un marché pour publier votre offre. Cliquez sur **Sélectionner tout** pour rendre votre offre disponible sur tous les marchés possibles ou sélectionnez les marchés spécifiques que vous souhaitez ajouter. Quand vous avez terminé, sélectionnez **Enregistrer**.

Vos sélections ici s’appliquent uniquement aux nouvelles acquisitions. Si quelqu’un a déjà votre application sur un marché donné et que vous supprimez par la suite ce marché, les personnes qui disposent déjà de cette offre peuvent continuer à l’utiliser, mais aucun nouveau client sur ce marché ne pourra bénéficier de votre offre.

> [!IMPORTANT]
> Il vous incombe de respecter les exigences légales locales, même si ces exigences ne sont pas répertoriées ici ou dans l’Espace partenaires.

Gardez à l’esprit que même si vous sélectionnez tous les marchés, les lois et restrictions locales ou d’autres facteurs peuvent empêcher certaines offres d’être répertoriées dans certains pays et régions.

### <a name="preview-audience"></a>Public de la version préliminaire

Avant de publier votre offre sur la place de marché générale, vous devez d’abord la mettre à disposition d’un **public limité en préversion**. Entrez une **Clé de masquage** (n’importe quelle chaîne utilisant uniquement des lettres minuscules et/ou des chiffres) ici. Les membres de votre public en préversion peuvent utiliser cette clé de masquage comme jeton pour afficher un aperçu de votre offre sur la place de marché.

Ensuite, lorsque vous êtes prêt à mettre votre offre à disposition générale et supprimer la restriction de la préversion, vous devez supprimer la **clé de masquage** et publier à nouveau.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="technical-configuration"></a>Configuration technique

Cette page définit les détails techniques utilisés pour se connecter à votre offre. Cette connexion permet d’approvisionner votre offre pour le client final, s’il choisit de l’acquérir.

### <a name="solution-identifier"></a>Identificateur de solution

Fournissez l’identificateur de solution (GUID) de votre solution.

Pour trouver l’identificateur de votre solution :

1. Dans les services de cycle de vie Microsoft Dynamics (LCS), sélectionnez **Gestion de la solution**.
2. Sélectionnez votre solution, puis recherchez **l'identificateur de solution** dans la **vue d’ensemble du package**. Si l’identificateur est vide, sélectionnez **Modifier** et republiez votre package, puis réessayez.

### <a name="release-version"></a>Version commerciale

Sélectionnez la version de Dynamics 365 for Finance and Operations avec lesquelles cette solution fonctionne.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="test-drive-technical-configuration"></a>Configuration technique de la version d’évaluation

Cette page vous permet de configurer une démonstration (« version d’évaluation ») qui permet aux clients d’évaluer votre offre avant de l’acheter. Pour en savoir, consultez [Qu’est-ce qu’un test drive ?](../what-is-test-drive.md).

Pour activer une version d’évaluation, cochez la case **Activer une version d’évaluation** sous l’onglet [Configuration de l’offre](#test-drive). Pour supprimer une version d'évaluation de votre offre, désactivez cette case à cocher.

Une fois que vous avez fini de configurer votre test drive, sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="supplemental-content"></a>Contenu supplémentaire

Cette page vous permet de fournir des informations supplémentaires sur votre offre pour que nous puissions la valider. Ces informations ne sont pas montrées aux clients ni publiées sur la Place de marché.

### <a name="validation-assets"></a>Ressources de validation

Chargez un [rapport d’analyse de la personnalisation (CAR)](/dynamics365/unified-operations/dev-itpro/dev-tools/customization-analysis-report) dans cette section. Ce rapport est généré en analysant vos modèles de personnalisation et d’extension, en fonction d’un ensemble prédéfini de règles de meilleures pratiques.

Ce fichier doit être au format .xls ou. xlsx. Si vous avez plus d’un rapport, vous pouvez télécharger un fichier .zip contenant tous les rapports.

### <a name="does-solution-include-localizations"></a>La solution comprend-elle des localisations ?

Sélectionnez **Oui** si la solution permet l’utilisation de stratégies et normes locales (par exemple, si elle prend en charge les différentes règles salariales requises par les différents pays/régions). Sinon, sélectionnez **Non**.

### <a name="does-solution-enable-translations"></a>La solution permet-elle les traductions ?

Répondez **Oui** si le texte de votre solution peut être traduit dans d’autres langues. Sinon, sélectionnez **Non**.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="publish"></a>Publish

### <a name="submit-offer-to-preview"></a>Soumettre l’offre en préversion

Une fois que vous avez complété toutes les sections obligatoires de l’offre, sélectionnez **Vérifier et publier** en haut à droite du portail.

S’il s’agit de votre première publication de cette offre, vous pouvez :

- Consultez l’état d’achèvement de chaque section de l’offre.
    - **Non démarrée**  – La section n’a pas encore été modifiée et doit être complétée.
    - **Incomplète** – La section comporte des erreurs qui doivent être corrigées ou nécessite un complément d’informations. Revenir à la ou les sections et les mettre à jour.
    - **Complète** – La section est complète, toutes les données obligatoires ont été fournies et aucune erreur n’est présente. Toutes les sections de l’offre doivent être dans l’état Complète avant de pouvoir envoyer l’offre.
- Dans la sections **Notes pour la certification** , fournissez des instructions de test à l’équipe de certification pour veiller à ce que votre application soit testée correctement, en plus de toutes autres notes supplémentaires utiles pour comprendre l’application. Pour plus d’informations sur les instructions de test et sur la réalisation d’une première publication, consultez [Validation fonctionnelle d’AppSource Dynamics 365 Finance and Operations](../dynamics-365-finance-operations-functional-validation.md).
- Envoyez l’offre pour publication en sélectionnant **Envoyer**. Nous vous enverrons un e-mail pour vous faire savoir si une version d’essai de l’offre est disponible pour vérification et approbation. Revenez à l’Espace partenaires et sélectionnez **Go-Live** pour l’offre de publication sur le public.

## <a name="next-step"></a>Étape suivante

- [Mettre à jour une offre existante dans la Place de marché commerciale](./update-existing-offer.md)