---
title: Planifier une offre Azure Application pour la Place de marché commerciale
description: Découvrez comment planifier une offre Azure Application pour référencer ou vendre votre application sur la Place de marché Azure ou via le programme Fournisseur de solutions cloud (CSP), en utilisant le portail de la Place de marché commerciale dans l’Espace partenaires Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 45bc3aaf5217c626e2593b4eda861eaace2a8be2
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620407"
---
# <a name="plan-an-azure-application-offer-for-the-commercial-marketplace"></a>Planifier une offre Azure Application pour la Place de marché commerciale

Cet article explique les différentes options et exigences relatives à la publication d’une offre Azure Application sur la place de marché commerciale Microsoft.

## <a name="before-you-begin"></a>Avant de commencer

Concevoir, créer et tester une offre d’application Azure nécessite de disposer de connaissances techniques sur la plateforme Azure et les technologies utilisées pour créer l’offre. Votre équipe d’ingénierie doit connaître les technologies Microsoft suivantes :

- Avoir une connaissance générale des [services Azure](https://azure.microsoft.com/services/).
- Savoir comment [concevoir et créer des applications Azure](https://azure.microsoft.com/solutions/architecture/).
- Avoir une expérience de travail avec les [Machines virtuelles Azure](https://azure.microsoft.com/services/virtual-machines/), le [Stockage Azure](https://azure.microsoft.com/services/?filter=storage#storage) et la [Mise en réseau Azure](https://azure.microsoft.com/services/?filter=networking#networking).
- Avoir une expérience de travail avec [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
- Avoir une expérience de travail avec [JSON](https://www.json.org/).

### <a name="technical-documentation-and-resources"></a>Documentation et ressources techniques

Lors de la planification de votre offre Azure Application pour la Place de marché commerciale, consultez les ressources suivantes.

- [Comprendre les modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
- Guides de démarrage rapide :
    - [Modèles de démarrage rapide Azure](https://azure.microsoft.com/documentation/templates/)
    - [Guide des meilleures pratiques pour les modèles Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md)
    - [Publier une définition d’application](../azure-resource-manager/managed-applications/publish-service-catalog-app.md)
    - [Déployer une application de catalogue de services](../azure-resource-manager/managed-applications/deploy-service-catalog-quickstart.md)
- Tutoriels :
    - [Créer les fichiers de définition](../azure-resource-manager/managed-applications/publish-service-catalog-app.md)
- Exemples :
    - [Azure CLI](../azure-resource-manager/managed-applications/cli-samples.md)
    - [Azure PowerShell](../azure-resource-manager/managed-applications/powershell-samples.md)
    - [Solutions d’applications managées](../azure-resource-manager/managed-applications/sample-projects.md)

La vidéo sur la [Création de modèles de solution et d’applications managées pour la Place de marché Azure](https://channel9.msdn.com/Events/Build/2018/BRK3603) présente de façon exhaustive le type d’offre d’application Azure :

- Quels sont les types d’offres disponibles ?
- Quelles sont les ressources techniques requises ?
- Comment créer un modèle Azure Resource Manager ?
- Comment développer et tester l’interface utilisateur d’une application ?
- Comment publier l’offre d’application ?
- Quel est le processus de revue d’une application ?

### <a name="suggested-tools"></a>Outils suggérés

Choisissez un ou plusieurs des environnements de scripts suivants pour faciliter la gestion de votre application Azure :

- [Azure PowerShell](/powershell/azure/)
- [Azure CLI](/cli/azure)

Nous vous recommandons d’ajouter les outils suivants à votre environnement de développement :

- [Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md)
- [Visual Studio Code](https://code.visualstudio.com/) avec les extensions suivantes :
    - Extension : [Outils Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
    - Extension : [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
    - Extension : [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json)

Vous pouvez examiner les outils disponibles dans la page [Outils de développement Azure](https://azure.microsoft.com/tools/) et, Si vous utilisez Visual Studio, consultez [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="listing-options"></a>Options de référencement

Une fois votre offre publiée, les options de référencement pour votre offre s’affichent sous la forme d’un bouton dans l’angle supérieur gauche de la page de référencement de l’offre. Par exemple, la capture d’écran suivante montre une page de référencement d’offre dans la Place de marché Azure avec les boutons _Obtenir maintenant_. Si vous avez choisi d’offrir un test drive, le bouton _Test Drive_ s’affiche également.

:::image type="content" source="media/create-new-azure-app-offer/azure-app-listing-page.png" alt-text="Illustre une page de la liste dans la Place de marché Azure.":::

## <a name="test-drive"></a>Test drive

Vous pouvez choisir d’activer un test drive pour votre offre Azure Application qui permet aux clients d’essayer votre offre avant de l’acheter. Pour en savoir plus sur les versions d’évaluation, consultez [Qu’est-ce qu’une version d’évaluation ?](what-is-test-drive.md). Pour plus d’informations sur la configuration de différents types de versions d’évaluation, consultez [Configuration technique de la version d’évaluation](test-drive-technical-configuration.md).

Vous pouvez également découvrir les [meilleures pratiques en matière de version d’évaluation](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices) et télécharger le [PDF de présentation des versions d’évaluation](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (assurez-vous que le bloqueur de fenêtres publicitaires est désactivé).

> [!NOTE]
> Informations que l’utilisateur devrait remarquer même en parcourant rapidement le documentDu fait que toutes les applications Azure sont implémentées à l’aide d’un modèle Azure Resource Manager, le seul type de version d’évaluation disponible pour une application Azure est une [version d’évaluation d’Azure Resource Manager](azure-resource-manager-test-drive.md).

## <a name="customer-leads"></a>Prospects

Vous devez connecter votre offre à votre système de gestion des relations avec la clientèle (Customer Relationship Management, CRM) pour collecter des informations sur les clients. Le client devra autoriser le partage de ses informations. Ces détails du client, ainsi que le nom de l’offre, son ID et le magasin en ligne dans lequel il a trouvé votre offre, seront envoyés au système CRM que vous avez configuré. La place de marché commerciale prend en charge un vaste éventail de systèmes CRM, ainsi que la possibilité d’utiliser une table Azure ou de configurer un point de terminaison HTTPS à l’aide de Power Automate.

Vous pouvez ajouter ou modifier une connexion CRM à tout moment pendant ou après la création de l’offre. Pour obtenir des instructions détaillées, consultez [Prospects de votre offre de marketplace commercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="categories-and-subcategories"></a>Catégories et sous-catégories

Vous pouvez choisir une ou deux catégories pour grouper votre offre dans les zones de recherche commerciales appropriées de la place de marché. Vous pouvez choisir jusqu’à deux sous-catégories pour chaque catégorie principale et secondaire. Pour obtenir une liste complète des catégories et sous-catégories, consultez [Bonnes pratiques pour le référencement des offres](gtm-offer-listing-best-practices.md#categories).

## <a name="legal-contracts"></a>Contrats juridiques

Pour simplifier le processus d’approvisionnement pour les clients et réduire la complexité juridique pour les éditeurs de logiciels, Microsoft propose un contrat Standard que vous pouvez utiliser pour vos offres dans la place de marché commerciale. Lorsque vous proposez votre logiciel dans le cadre du contrat standard, les clients ne doivent lire et accepter ce contrat qu’une seule fois, et vous n’avez pas besoin d’élaborer des conditions générales personnalisées.

Si vous choisissez d’utiliser le contrat standard, vous avez la possibilité d’ajouter à celui-ci des conditions d’avenant universel et jusqu’à 10 avenants personnalisés. Vous pouvez également utiliser vos propres conditions générales à la place du contrat standard. Vous devez gérer ces détails dans la page **Propriétés**. Pour plus d’informations, consultez [Contrat Standard pour la place de marché commerciale de Microsoft](standard-contract.md).

> [!NOTE]
> Après avoir publié une offre à l’aide du contrat standard pour la place de marché commerciale, vous ne pouvez pas utiliser vos propres conditions générales personnalisées. Vous devez faire un choix entre les deux. Vous proposez votre solution avec le contrat standard ou vos propres conditions générales. Si vous souhaitez modifier les conditions du contrat standard, vous le pouvez par le biais d’avenants au contrat standard.

## <a name="offer-listing-details"></a>Détails du référencement de l’offre

Lorsque vous créez une nouvelle application Azure dans l’Espace partenaires, vous devez entrer du texte, des images, des vidéos facultatives et d’autres détails dans la page de référencement de l’offre. Il s’agit des informations que les clients verront quand ils découvriront le référencement de votre offre dans la Place de marché Azure, comme illustré dans l’exemple suivant.

:::image type="content" source="media/create-new-azure-app-offer/example-azure-marketplace-app.png" alt-text="Illustre la façon dont cette offre apparaît dans la Place de marché Azure.":::

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Logo
2. Catégories
3. Adresse du support technique (lien)
4. Conditions d’utilisation
5. Adresse de la politique de confidentialité (lien)
6. Nom de l’offre
7. Résumé
8. Description
9. Captures d’écran/vidéos

La capture d’écran suivante montre la façon dont les informations de l’offre s’affichent dans le portail Azure :

[![Illustre la façon dont cette offre s’affiche dans le portail Azure.](media/create-new-azure-app-offer/example-virtual-machine-container-iot-edge-saas.png)](media/create-new-azure-app-offer/example-virtual-machine-container-iot-edge-saas.png#lightbox)

#### <a name="call-out-descriptions"></a>Descriptions de légende

1. Intitulé
2. Description
3. Liens utiles
4. Captures d’écran.

> [!NOTE]
> Le contenu du référencement de l’offre ne doit pas nécessairement être en anglais si la description de l’offre commence par la phrase « This application is available only in [langue autre que l’anglais] ».

Pour faciliter la création de votre offre, préparez certains de ces éléments à l’avance. Sauf indication contraire, les éléments suivants sont requis.

- **Name** : ce nom apparaîtra en tant que titre du référencement de votre annonce dans la place de marché commerciale. Le nom peut être une marque. Il ne peut pas contenir d’emojis (sauf s’il s’agit du symbole de marque ou de copyright) et ne doit pas dépasser 50 caractères.
- **Résultats de recherche récapitulatifs** : décrivez l’objectif ou la fonction de votre offre sous la forme d’une phrase en texte brut sans saut de ligne de maximum 100 caractères. Ce récapitulatif est utilisé dans les résultats de recherche dans les référencements de la place de marché commerciale.
- **Description courte** : ajoutez jusqu’à 256 caractères de texte brut. Ce résumé s’affichera sur la page des détails de votre offre.
- **Description** : cette description s’affichera dans la vue d’ensemble des référencements sur la Place de marché Azure. Vous pouvez inclure une proposition de valeur, des avantages clés, une base utilisateur visée, des associations de catégories ou de secteurs, des opportunités d’achats dans l’application, les besoins des clients ou le problème que cette offre concerne, des informations requises et un lien pour en savoir plus.

    Cette zone de texte contient des contrôles d’éditeur de texte riches, que vous pouvez utiliser pour rendre votre description plus attrayante. Vous pouvez également utiliser des balises HTML pour mettre en forme votre description. Vous pouvez entrer dans cette zone de texte jusqu’à 3 000 caractères, balisage HTML et espaces compris. Pour obtenir d’autres conseils, consultez [Rédiger une bonne description d’application](/windows/uwp/publish/write-a-great-app-description) et [Balises HTML prises en charge dans les descriptions d’offre de la place de marché commerciale](supported-html-tags.md).

- **Mots clés de recherche** (facultatif) : Entrez jusqu’à trois mots clés que les clients pourront utiliser pour rechercher votre offre dans le magasin en ligne. Pour obtenir des résultats optimaux, utilisez également ces mots clés dans votre description. Vous n’avez pas besoin d’inclure le **nom** et la **description** de l’offre. Ce texte est inclus automatiquement dans la recherche.
- **Lien vers la politique de confidentialité** : URL de la politique de confidentialité de votre société. Vous devez renseigner une politique de confidentialité conforme et veiller à ce que votre application respecte les lois et réglementations en matière de confidentialité.
- **Liens utiles** (facultatif) : vous pouvez fournir des liens vers diverses ressources destinées aux utilisateurs de votre offre. Par exemple, forums, FAQ et notes de publication.
- **Informations de contact** : vous devez désigner les contacts suivants de votre organisation :
  - **Contact de support** : indiquez le nom, le numéro de téléphone et l’adresse e-mail des partenaires Microsoft à utiliser quand vos clients ouvrent des tickets. Vous devez également inclure l’URL de votre site web de support.
  - **Contact d’ingénierie** : indiquez le nom, le numéro de téléphone et l’adresse e-mail de Microsoft à utiliser directement en cas de problème avec votre offre. Ces informations de contact ne sont pas répertoriées dans la place de marché commerciale.
  - **Contact du programme du fournisseur de solutions cloud** (facultatif) : indiquez le nom, le numéro de téléphone et l’adresse e-mail si vous optez pour le programme du fournisseur de solutions cloud (CSP), afin que ces partenaires puissent vous contacter pour toute question. Vous pouvez également inclure une URL pour vos documents marketing.
- **Média – Logos** : Fournissez un fichier PNG pour le logo de **grande taille**. L'Espace partenaires l'utilisera pour créer un logo **Petit** et **Moyen**. Plus tard, vous pourrez éventuellement les remplacer par d'autres images.
  - Grande taille (de 216 x 216 à 350 x 350 px, obligatoire)
  - Moyen (90 x 90 px, facultatif)
  - Petite taille (48 x 48 px, facultatif)

  Ces logos sont utilisés à différents emplacements dans les magasins en ligne :
  - Le petit logo s’affiche dans les résultats de la recherche de la Place de marché Azure.
  - Le logo moyen s’affiche lorsque vous créez une nouvelle ressource dans Microsoft Azure.
  - Le grand logo apparaît sur la page de votre liste d’offres dans la Place de marché Azure.

  Suivez ces instructions pour vos logos:

  - Le design Azure a une palette de couleurs simple. Limitez le nombre de couleurs primaires et secondaires sur votre logo.
  - Les couleurs de thème du portail sont le blanc et le noir. Évitez d’utiliser ces couleurs comme couleur d’arrière-plan pour votre logo. Utilisez une couleur qui donne à votre logo plus de visibilité dans le portail. Nous vous recommandons d’utiliser des couleurs primaires simples.
  - Si vous utilisez un arrière-plan transparent, assurez-vous que le logo et le texte ne sont pas blancs, noirs ou bleus.
  - L’apparence de votre logo doit être « plate » et les dégradés doivent être évités sur le logo ou l’arrière-plan. Ne placez pas de texte sur le logo, pas même le nom de votre marque ou de votre entreprise. Votre demande sera rejetée si les images sont floues.
  - Assurez-vous que le logo n’est pas étiré.

- **Média - Captures d’écran** (facultatif) : Nous vous recommandons d’ajouter des captures d’écran qui illustrent le fonctionnement de votre offre. Vous pouvez ajouter jusqu’à cinq captures d’écran avec les exigences suivantes, qui illustrent le fonctionnement de votre offre :
  - 1280 x 720 pixels
  - Fichier .png
  - Doit inclure une légende
- **Média – Vidéos** (facultatif) : vous pouvez ajouter jusqu’à cinq vidéos avec les exigences suivantes, qui présentent votre offre :
  - Nom
  - URL : doit être hébergée sur YouTube ou Vimeo uniquement.
  - Miniature : fichier .png 1280 x 720

> [!NOTE]
> Pour être publiées sur la place de marché commerciale, votre offre doit respecter les [stratégies de certification de la place de marché commerciale](/legal/marketplace/certification-policies#100-general.md).

## <a name="preview-audience"></a>Public de la préversion

Le public de la préversion peut accéder à votre offre avant sa publication dans les magasins en ligne, afin de tester sa fonctionnalité de bout en bout avant la publication en ligne.

> [!NOTE]
> Le public d’une préversion diffère de celui d’un plan privé. Un plan privé est un plan que vous mettez à la disposition d’un public spécifique que vous choisissez. Cela vous permet de négocier un plan personnalisé avec des clients spécifiques.

Vous définissez le public de la préversion à l’aide d’ID d’abonnement Azure et d’une description facultative pour chacun. Aucun de ces champs ne peut être vu par les clients.

## <a name="technical-configuration"></a>Configuration technique

Pour les applications managées qui émettent des événements de contrôle à l’aide des [API du service de contrôle de la Place de marché](partner-center-portal/marketplace-metering-service-apis.md), vous devez fournir l’identité que votre service utilisera lors de l’émission d’événements de contrôle.

Cette configuration est requise si vous souhaitez utiliser l’option [Événement d’utilisation par lot](partner-center-portal/marketplace-metering-service-apis.md#metered-billing-batch-usage-event). Si vous souhaitez soumettre un [événement d’utilisation](partner-center-portal/marketplace-metering-service-apis.md#metered-billing-single-usage-event), vous pouvez également utiliser le [service de métadonnées d’instance](../active-directory/managed-identities-azure-resources/overview.md) pour obtenir le [Jeton du porteur du JSON Web Token (JWT)](partner-center-portal/pc-saas-registration.md#how-to-get-the-publishers-authorization-token)).

- **ID locataire Azure Active Directory** (requis) : Dans le portail Azure, vous devez [créer une application Azure Active Directory (AD)](../active-directory/develop/howto-create-service-principal-portal.md) afin que nous puissions confirmer que la connexion entre nos deux services se fait bien dans le cadre d’une communication authentifiée. Pour trouver l’[ID de locataire](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de votre application Azure Active Directory (AD), accédez au panneau [Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) dans Azure Active Directory. Dans la colonne **Nom d’affichage**, sélectionnez l’application. Recherchez ensuite **Propriétés** puis le **numéro d’ID de répertoire (locataire)** référencé (par exemple, `50c464d3-4930-494c-963c-1e951d15360e`).
- **ID d'application Azure Active Directory** (obligatoire) : Vous avez également besoin de l’[ID de votre application](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) et d’une clé d’authentification. Pour rechercher votre ID d’application, accédez au panneau [Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) dans votre Azure Active Directory. Dans la colonne **Nom d’affichage**, sélectionnez l’application, puis recherchez l’**ID de l’application de (client)** (par exemple `50c464d3-4930-494c-963c-1e951d15360e`). Pour trouver la clé d’authentification, accédez à **Paramètres** et sélectionnez **Clés**. Vous devez fournir une description et une durée, et vous obtiendrez ensuite une valeur numérique.

> [!NOTE]
> L’ID d’application Azure est associé à votre ID d’éditeur et ne peut être réutilisé que dans ce compte d’éditeur.

## <a name="additional-sales-opportunities"></a>Opportunités de ventes supplémentaires

Vous pouvez opter pour des canaux marketing et de vente pris en charge par Microsoft. Lorsque vous créez votre offre dans l’Espace partenaires, deux onglets s’affichent vers la fin du processus :

- **Revendre via des fournisseurs de solutions cloud** : utilisez cette option pour permettre aux partenaires fournisseurs de solutions cloud (CSP) Microsoft de revendre votre solution dans le cadre d’une offre groupée. Pour plus d’informations, consultez [Programme des fournisseurs de solution cloud](./cloud-solution-providers.md).
- **Co-vendre avec Microsoft** : cette option permet aux équipes de vente de Microsoft de tenir compte de votre solution éligible de co-vente IP lors de l’évaluation des besoins de leurs clients. Pour plus d’informations sur la façon de préparer votre offre à des fins d’évaluation, consultez [Option de co-vente sur le marketplace commercial](commercial-marketplace-co-sell.md). Pour plus d’informations sur les exigences en matière de co-vente IP, consultez [Prérequis relatifs à l’état de co-vente](https://aka.ms/CertificationPolicies#3000-requirements-for-co-sell-status). Pour plus d’informations sur le marketing de votre offre via les canaux partenaires fournisseurs de solutions cloud Microsoft, consultez [Fournisseurs de solutions cloud](cloud-solution-providers.md).

Pour en savoir plus, consultez [Développer votre activité avec la Place de marché Azure](https://azuremarketplace.microsoft.com/sell).

## <a name="plans"></a>Plans

Les offres Azure Application nécessitent au moins un plan. Un plan définit l’étendue et les limites de la solution, ainsi que la tarification associée, le cas échéant. Vous pouvez créer plusieurs plans pour votre offre afin d’offrir à vos clients différentes options techniques et de tarification.

Pour obtenir des conseils généraux sur les plans, notamment des modèles de tarification, et des plans privés, consultez [Plans et tarification des offres de la Place de marché commerciale](plans-pricing.md). Les sections suivantes présentent des informations supplémentaires spécifiques aux plans Azure Application.

### <a name="types-of-plans"></a>Types de plans

Il existe deux types de plans d’application Azure : les _modèles de solution_ et les _applications managées_. Les deux types de plan prennent en charge l’automatisation du déploiement et de la configuration d’une solution au-delà d’une seule machine virtuelle. Vous pouvez automatiser la fourniture de plusieurs ressources, notamment les machines virtuelles, les ressources réseau et les ressources de stockage nécessaires pour fournir des solutions complexes, telles que des solutions IaaS. Les deux types de plans peuvent utiliser de nombreux types de ressources Azure, notamment, mais sans s’y limiter, les machines virtuelles.

- Les plans de **modèles de solution** constituent l’un des principaux moyens de publier une solution sur la place de marché commerciale. Les plans de modèle de solution ne peuvent pas faire l’objet d’une transaction sur la place de marché commerciale, mais peuvent être utilisés pour déployer des offres de machines virtuelles payantes facturées via la place de marché commerciale. Utilisez le type de plan du modèle de solution lorsque le client gère la solution et que les transactions sont facturées par le biais d’un autre plan. Pour plus d’informations sur la création de modèles de solution, consultez [Qu’est ce qu’Azure Resource Manager ?](../azure-resource-manager/management/overview.md)
- Les plans d’**applications managées** vous permettent de créer et fournir facilement des applications clés en main et entièrement managées à vos clients. Ils ont les mêmes fonctionnalités que les plans de modèles de solution, malgré quelques différences essentielles :
    - Les ressources sont déployées vers un groupe de ressources et sont gérées par l’éditeur de l’application. Le groupe de ressources est présent dans l’abonnement du consommateur, mais une identité du locataire de l’éditeur a accès au groupe de ressources. 
    - En tant qu’éditeur, vous spécifiez le coût de la prise en charge de la solution en continu, et les transactions sont prises en charge via la place de marché commerciale.
 
    Utilisez le type de plan d’application managée lorsque vous ou votre client exigez que la solution soit gérée par un partenaire ou que vous déployiez une solution basée sur un abonnement. Pour plus d’informations sur les avantages et les différents types d’applications managées, consultez [Vue d’ensemble des applications managées Azure](../azure-resource-manager/managed-applications/overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour planifier un modèle de solution, consultez [Planifier un modèle de solution pour une offre d’application Azure](plan-azure-app-solution-template.md).
- Pour planifier une application managée Azure, consultez [Planifier une application managée Azure pour une offre Azure Application](plan-azure-app-managed-app.md).