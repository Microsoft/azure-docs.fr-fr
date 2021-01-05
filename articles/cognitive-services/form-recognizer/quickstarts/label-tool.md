---
title: 'Démarrage rapide : Étiqueter des formulaires, effectuer l’apprentissage d’un modèle et analyser des formulaires à l’aide de l’exemple d’outil d’étiquetage – Form Recognizer'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous allez utiliser l’outil d’étiquetage des exemples Form Recognizer pour étiqueter manuellement des documents de formulaire. Ensuite, vous allez effectuer l’apprentissage d’un modèle de traitement de documents personnalisé, et utiliser le modèle pour extraire des paires clé/valeur.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 09/30/2020
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: traitement de documents
ms.openlocfilehash: 7671d8d58ffbd0fca444eefe53c46c99a4e76d37
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96009328"
---
# <a name="train-a-form-recognizer-model-with-labels-using-the-sample-labeling-tool"></a>Entraîner un modèle Form Recognizer avec des étiquettes à l’aide de l’outil d’étiquetage des exemples

Dans ce démarrage rapide, vous allez utiliser l’API REST Form Recognizer avec l’exemple d’outil d’étiquetage pour effectuer l’apprentissage d’un modèle de traitement de documents personnalisé avec des données étiquetées manuellement. Pour en savoir plus sur l’apprentissage supervisé avec Form Recognizer, consultez la section [Effectuer l’apprentissage avec des étiquettes](../overview.md#train-with-labels) de la vue d’ensemble.

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Azure-Form-Recognizer/player]

## <a name="prerequisites"></a>Prérequis

Pour suivre cette procédure de démarrage rapide, vous avez besoin des éléments suivants :

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="créez une ressource Form Recognizer"  target="_blank">créer une ressource Form Recognizer<span class="docon docon-navigate-external x-hidden-focus"></span></a> sur le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Form Recognizer. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.
* Au minimum un ensemble de six formulaires du même type. Vous allez utiliser ces données pour entraîner le modèle et tester un formulaire. Pour ce guide de démarrage rapide, vous pouvez utiliser un [exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2090451) (téléchargez et extrayez *sample_data.zip*). Chargez les fichiers d’apprentissage à la racine d’un conteneur de stockage d’objets blob dans un compte Stockage Azure offrant un niveau de performance standard.

## <a name="create-a-form-recognizer-resource"></a>Créer une ressource Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="try-it-out"></a>Faire un essai

Pour essayer en ligne l’outil d’étiquetage des exemples Form Recognizer, accédez au [site web FOTT](https://fott-preview.azurewebsites.net/).

# <a name="v20"></a>[v2.0](#tab/v2-0)
> [!div class="nextstepaction"]
> [Essayer des modèles prédéfinis](https://fott.azurewebsites.net/)

# <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1)
> [!div class="nextstepaction"]
> [Essayer des modèles prédéfinis](https://fott-preview.azurewebsites.net/)

---

Vous aurez besoin d’un abonnement Azure ([créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services)) et d’un point de terminaison et d’une clé [de ressource Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer) pour tester le service Form Recognizer. 


## <a name="set-up-the-sample-labeling-tool"></a>Configurer l’outil d’étiquetage des exemples

Vous allez utiliser le moteur Docker pour exécuter l’outil d’étiquetage des exemples. Procédez comme suit pour configurer le conteneur Docker. Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).

> [!TIP]
> OCR Form Labeling Tool est également disponible en tant que projet open source sur GitHub. L’outil est une application web TypeScript créée à l’aide de React + Redux. Pour en savoir plus ou apporter votre contribution, consultez le dépôt [OCR Form Labeling Tool](https://github.com/microsoft/OCR-Form-Tools/blob/master/README.md#run-as-web-application). Pour tester l’outil en ligne, accédez au [site web FOTT](https://fott.azurewebsites.net/).   

1. Tout d’abord, installez Docker sur un ordinateur hôte. Ce guide va vous montrer comment utiliser l’ordinateur local en tant qu’hôte. Si vous voulez utiliser un service d’hébergement Docker dans Azure, consultez le guide pratique [Déployer l’outil d’étiquetage des exemples](../deploy-label-tool.md). 

   L’ordinateur hôte doit satisfaire à la configuration matérielle suivante :

    | Conteneur | Minimum | Recommandé|
    |:--|:--|:--|
    |Outil d’étiquetage des exemples|2 cœurs, 4 Go de mémoire|4 cœurs, 8 Go de mémoire|

   Installez Docker sur votre ordinateur en suivant les instructions appropriées pour votre système d’exploitation : 
   * [Windows](https://docs.docker.com/docker-for-windows/)
   * [macOS](https://docs.docker.com/docker-for-mac/)
   * [Linux](https://docs.docker.com/install/)



1. Récupérez le conteneur de l’outil d’étiquetage des exemples avec la commande `docker pull`.

    # <a name="v20"></a>[v2.0](#tab/v2-0)    
    ```
    docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool
    ```
    # <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1)    
    ```
    docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-preview
    ```

    ---

1. Vous êtes maintenant prêt à exécuter le conteneur avec `docker run`.

    # <a name="v20"></a>[v2.0](#tab/v2-0)    
    ```
    docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool eula=accept
    ```
    # <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1)    
    ```
    docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-preview eula=accept    
    ```

    --- 

   Cette commande rend l’outil d’étiquetage des exemples disponible par le biais d’un navigateur web. Atteindre `http://localhost:3000`.

> [!NOTE]
> Vous pouvez également étiqueter des documents et entraîner des modèles à l’aide de l’API REST Form Recognizer. Pour effectuer un entraînement et une analyse avec l’API REST, consultez [Entraîner avec des étiquettes en utilisant l’API REST et Python](./python-labeled-data.md).

## <a name="set-up-input-data"></a>Configurer les données d’entrée

Tout d’abord, vérifiez que tous les documents d’entraînement ont le même format. Si vous avez des formulaires dans plusieurs formats, organisez-les en sous-dossiers en fonction du format. Lors de l’entraînement, vous devez diriger l’API vers un sous-dossier.

### <a name="configure-cross-domain-resource-sharing-cors"></a>Configurer le partage des ressources inter-domaines (CORS)

Activez CORS sur votre compte de stockage. Sélectionnez votre compte de stockage dans le portail Azure, puis cliquez sur l’onglet **CORS** dans le volet gauche. Sur la ligne inférieure, renseignez les valeurs suivantes. Cliquez ensuite sur **Enregistrer** en haut.

* Origines autorisées = * 
* Méthodes autorisées = \[tout sélectionner\]
* En-têtes autorisés = *
* En-têtes exposés = * 
* Âge maximal = 200

> [!div class="mx-imgBorder"]
> ![Configuration de CORS dans le portail Azure](../media/label-tool/cors-setup.png)

## <a name="connect-to-the-sample-labeling-tool"></a>Se connecter à l’outil d’étiquetage des exemples

L’outil d’étiquetage des exemples se connecte à une source (où se trouvent vos formulaires d’origine) et à une cible (vers laquelle il exporte les étiquettes et les données de sortie créées).

Les connexions peuvent être configurées et partagées entre les projets. Elles utilisent un modèle de fournisseur extensible, ce qui vous permet d’ajouter facilement de nouveaux fournisseurs sources/cibles.

Pour créer une connexion, cliquez sur l’icône **Nouvelles connexions** (fiche électrique) dans la barre de navigation de gauche.

Renseignez les champs avec les valeurs suivantes :

* **Display Name** (nom d’affichage) : nom d’affichage de la connexion.
* **Description** : description de votre projet.
* **SAS URL** (URL SAS) : URL de signature d’accès partagé (SAS) de votre conteneur Stockage Blob Azure. Pour récupérer l’URL SAS, ouvrez l’Explorateur Stockage Microsoft Azure, cliquez avec le bouton droit sur votre conteneur, puis sélectionnez **Obtenir une signature d’accès partagé**. Définissez le délai d’expiration sur une heure à laquelle vous aurez fini d’utiliser le service. Assurez-vous que les autorisations **Read** (Lecture), **Write** (Écriture), **Delete** (Suppression) et **List** (Listage) sont cochées, puis cliquez sur **Create** (Créer). Copiez alors la valeur dans la section **URL**. Il doit avoir le format : `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.

:::image type="content" source="../media/label-tool/connections.png" alt-text="Paramètres de connexion de l’outil d’étiquetage des exemples.":::


## <a name="create-a-new-project"></a>Création d'un projet

Dans l’outil d’étiquetage des exemples, les projets stockent vos configurations et paramètres. Créez un projet et renseignez les champs avec les valeurs suivantes :

* **Display Name** (Nom d’affichage) : nom d’affichage du projet
* **Security Token** (Jeton de sécurité) : certains paramètres de projet peuvent inclure des valeurs sensibles, telles que des clés API ou d’autres secrets partagés. Chaque projet génère un jeton de sécurité qui peut être utilisé pour chiffrer/déchiffrer les paramètres de projet sensibles. Vous trouvez les jetons de sécurité dans les paramètres de l’application, en cliquant sur l’icône d’engrenage en bas de la barre de navigation gauche.
* **Source Connection** (Connexion source) : connexion au Stockage Blob Azure que vous avez créée à l’étape précédente et que vous souhaitez utiliser pour ce projet.
* **Folder Path** (Chemin du dossier) : (facultatif) Si vos formulaires sources se trouvent dans un dossier sur le conteneur d’objets blob, spécifiez le nom du dossier ici.
* **Form Recognizer Service Uri** (URI du service Form Recognizer) : votre URL de point de terminaison Form Recognizer.
* **API Key** (Clé API) : votre clé d’abonnement Form Recognizer.
* **Description** : (facultatif) Description du projet

:::image type="content" source="../media/label-tool/new-project.png" alt-text="Page de nouveau projet dans l’outil d’étiquetage des exemples.":::

## <a name="label-your-forms"></a>Étiqueter vos formulaires

Quand vous créez ou ouvrez un projet, la fenêtre principale de l’éditeur d’étiquettes s’ouvre. L’éditeur d’étiquettes est composé de trois parties :

* Un volet de visualisation redimensionnable, qui contient une liste déroulante de formulaires à partir de la connexion source.
* Le volet principal de l’éditeur, qui vous permet d’appliquer des étiquettes.
* Le volet de l’éditeur d’étiquettes, qui permet aux utilisateurs de modifier, de verrouiller, de réorganiser et de supprimer des étiquettes. 

### <a name="identify-text-elements"></a>Identifier les éléments de texte

Cliquez sur **Run OCR on all files** (Exécuter l’OCR sur tous les fichiers) dans le volet gauche pour obtenir des informations de disposition du texte pour chaque document. L’outil d’étiquetage dessine des rectangles englobants autour de chaque élément de texte.

Il indique aussi les tables qui ont été automatiquement extraites. Cliquez sur l’icône de table/grille à gauche du document pour voir la table extraite. Dans ce démarrage rapide, le contenu des tables étant automatiquement extrait, nous n’étiquèterons pas le contenu des tables et ferons confiance à l’extraction automatisée.

:::image type="content" source="../media/label-tool/table-extraction.png" alt-text="Visualisation d’une table dans l’outil d'étiquetage des exemples.":::

### <a name="apply-labels-to-text"></a>Appliquer des étiquettes à du texte

Vous allez ensuite créer des balises (étiquettes) et les appliquer aux éléments de texte que vous souhaitez que le modèle reconnaisse.

# <a name="v20"></a>[v2.0](#tab/v2-0)  
1. Tout d’abord, utilisez le volet de l’éditeur d’étiquettes pour créer les étiquettes que vous souhaitez identifier.
   1. Cliquez sur **+** pour créer une étiquette.
   1. Entrez le nom de l’étiquette.
   1. Appuyez sur Entrée pour enregistrer l’étiquette.
1. Dans l’éditeur principal, cliquez pour sélectionner des mots parmi les éléments de texte en surbrillance.
1. Cliquez sur l’étiquette que vous souhaitez appliquer ou appuyez sur la touche du clavier correspondante. Les touches numériques sont affectées comme touches d’accès rapide pour les 10 premières étiquettes. Vous pouvez réorganiser vos étiquettes à l’aide des icônes de flèches haut et bas dans le volet de l’éditeur d’étiquettes.
    > [!Tip]
    > Gardez à l’esprit les conseils suivants quand vous étiquetez vos formulaires.
    > * Vous ne pouvez appliquer qu’une seule étiquette à chaque élément de texte sélectionné.
    > * Chaque étiquette ne peut être appliquée qu’une seule fois par page. Si une valeur apparaît plusieurs fois sur le même formulaire, créez des étiquettes différentes pour chaque instance, par exemple « facture n° 1 », « facture n° 2 », etc.
    > * Les étiquettes ne peuvent pas s’étendre sur plusieurs pages.
    > * Étiquetez les valeurs telles qu’elles apparaissent sur le formulaire ; n’essayez pas de fractionner une valeur en deux parties avec deux étiquettes différentes. Par exemple, un champ d’adresse doit être étiqueté avec une étiquette unique, même s’il s’étend sur plusieurs lignes.
    > * N’incluez pas de clés dans vos champs étiquetés&mdash;uniquement les valeurs.
    > * Les données de la table doivent être détectées automatiquement et seront disponibles dans le fichier JSON de sortie final. Toutefois, si le modèle ne parvient pas à détecter toutes les données de votre table, vous pouvez aussi étiqueter manuellement ces champs. Étiquetez chaque cellule de la table avec une étiquette différente. Si vos formulaires comportent des tables avec un nombre variable de lignes, veillez à étiqueter au moins un formulaire avec la table la plus grande possible.
    > * Servez-vous des boutons situés à droite de **+** pour rechercher, renommer, réorganiser et supprimer vos étiquettes.
    > * Pour supprimer une étiquette appliquée sans supprimer l’étiquette proprement dite, sélectionne lez rectangle étiqueté dans la vue du document et appuyez sur la touche de suppression.


# <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1) 
1. Tout d’abord, utilisez le volet de l’éditeur d’étiquettes pour créer les étiquettes que vous souhaitez identifier.
   1. Cliquez sur **+** pour créer une étiquette.
   1. Entrez le nom de l’étiquette.
   1. Appuyez sur Entrée pour enregistrer l’étiquette.
1. Dans l’éditeur principal, cliquez pour sélectionner des mots parmi les éléments de texte en surbrillance. Dans _v2.1 preview.2_, vous pouvez aussi cliquer pour sélectionner des _marques de sélection_ comme les cases d’option et les cases à cocher en tant que paires clé-valeur. Form Recognizer détermine si la marque de sélection est « sélectionnée » ou « non sélectionnée » en tant que valeur.
1. Cliquez sur l’étiquette que vous souhaitez appliquer ou appuyez sur la touche du clavier correspondante. Les touches numériques sont affectées comme touches d’accès rapide pour les 10 premières étiquettes. Vous pouvez réorganiser vos étiquettes à l’aide des icônes de flèches haut et bas dans le volet de l’éditeur d’étiquettes.
    > [!Tip]
    > Gardez à l’esprit les conseils suivants quand vous étiquetez vos formulaires.
    > * Vous ne pouvez appliquer qu’une seule étiquette à chaque élément de texte sélectionné.
    > * Chaque étiquette ne peut être appliquée qu’une seule fois par page. Si une valeur apparaît plusieurs fois sur le même formulaire, créez des étiquettes différentes pour chaque instance, par exemple « facture n° 1 », « facture n° 2 », etc.
    > * Les étiquettes ne peuvent pas s’étendre sur plusieurs pages.
    > * Étiquetez les valeurs telles qu’elles apparaissent sur le formulaire ; n’essayez pas de fractionner une valeur en deux parties avec deux étiquettes différentes. Par exemple, un champ d’adresse doit être étiqueté avec une étiquette unique, même s’il s’étend sur plusieurs lignes.
    > * N’incluez pas de clés dans vos champs étiquetés&mdash;uniquement les valeurs.
    > * Les données de la table doivent être détectées automatiquement et seront disponibles dans le fichier JSON de sortie final. Toutefois, si le modèle ne parvient pas à détecter toutes les données de votre table, vous pouvez aussi étiqueter manuellement ces champs. Étiquetez chaque cellule de la table avec une étiquette différente. Si vos formulaires comportent des tables avec un nombre variable de lignes, veillez à étiqueter au moins un formulaire avec la table la plus grande possible.
    > * Servez-vous des boutons situés à droite de **+** pour rechercher, renommer, réorganiser et supprimer vos étiquettes.
    > * Pour supprimer une étiquette appliquée sans supprimer l’étiquette proprement dite, sélectionne lez rectangle étiqueté dans la vue du document et appuyez sur la touche de suppression.


---

:::image type="content" source="../media/label-tool/main-editor-2-1.png" alt-text="Fenêtre principale de l’éditeur de l’outil d’étiquetage des exemples.":::


Effectuez les étapes ci-dessus pour étiqueter au moins cinq de vos formulaires.

### <a name="specify-tag-value-types"></a>Spécifier des types de valeurs d’étiquettes

Si vous le souhaitez, vous pouvez définir le type de données attendu pour chaque étiquette. Ouvrez le menu contextuel à droite d’une étiquette et sélectionnez un type dans le menu. Cette fonctionnalité permet à l’algorithme de détection d’effectuer certaines hypothèses qui amélioreront la justesse de la détection de texte. Elle garantit également que les valeurs détectées sont retournées dans un format normalisé dans la sortie JSON finale. 

> [!div class="mx-imgBorder"]
> ![Sélection d'un type valeur avec l'outil d'étiquetage des exemples](../media/whats-new/formre-value-type.png)

Les types et variantes de valeurs suivants sont actuellement pris en charge :
* `string`
    * default, `no-whitespaces`, `alphanumeric`
* `number`
    * default, `currency`
* `date` 
    * default, `dmy`, `mdy`, `ymd`
* `time`
* `integer`
* `selectionMark` – _Nouveautés de v2.1-preview.1_

> [!NOTE]
> Consultez les règles suivantes pour la mise en forme des dates :
> 
> Vous devez spécifier un format (`dmy`, `mdy`, `ymd`) pour le fonctionnement de la mise en forme de la date.
>
> Les caractères suivants peuvent être utilisés comme délimiteurs de date : `, - / . \`. L’espace blanc ne peut pas être utilisé comme délimiteur. Par exemple :
> * 01,01,2020
> * 01-01-2020
> * 01/01/2020
>
> Le jour et le mois peuvent chacun être écrits sous la forme d’un ou de deux chiffres, et l’année sous la forme de deux ou quatre chiffres :
> * 1-1-2020
> * 1-01-20
>
> Si une chaîne de date contient huit chiffres, le délimiteur est facultatif :
> * 01012020
> * 01 01 2020
>
> Le mois peut également être écrit sous sa forme complète ou abrégée. Si le nom est utilisé, les caractères de délimitation sont facultatifs. Toutefois, ce format peut être reconnu moins précisément que d’autres.
> * 01/Jan/2020
> * 01Jan2020
> * 01 Jan 2020

## <a name="train-a-custom-model"></a>Entraîner un modèle personnalisé

Dans le volet gauche, cliquez sur l’icône d’entraînement (wagon) pour ouvrir la page Training (Entraînement). Cliquez ensuite sur le bouton **Train** pour commencer l’entraînement du modèle. Une fois le processus d’entraînement terminé, les informations suivantes s’affichent :

* **Model ID** : ID du modèle qui a été créé et entraîné. Chaque appel d’entraînement crée un modèle avec son propre ID. Copiez cette chaîne dans un emplacement sûr. Vous en aurez besoin si vous souhaitez effectuer des appels de prédiction via l’[API REST](./curl-train-extract.md) pi une [bibliothèque de client](./client-library.md).
* **Average Accuracy** : justesse moyenne du modèle. Vous pouvez améliorer la justesse du modèle en étiquetant des formulaires supplémentaires et en effectuant un nouvel entraînement pour créer un modèle. Nous vous recommandons de commencer par étiqueter cinq formulaires et d’ajouter des formulaires en fonction des besoins.
* Liste des étiquettes et justesse estimée par étiquette.


:::image type="content" source="../media/label-tool/train-screen.png" alt-text="Vue de l’entraînement.":::

Une fois l’entraînement terminé, examinez la valeur **Average Accuracy**. Si cette valeur est petite, vous devez ajouter d’autres documents d’entrée et répéter les étapes ci-dessus. Les documents que vous avez déjà étiquetés sont conservés dans l’index du projet.

> [!TIP]
> Vous pouvez également exécuter le processus d’entraînement avec un appel d’API REST. Pour savoir comment procéder, consultez [Effectuer un entraînement avec des étiquettes à l’aide de Python](./python-labeled-data.md).

## <a name="compose-trained-models"></a>Composer des modèles entraînés

# <a name="v20"></a>[v2.0](#tab/v2-0)  

Cette fonctionnalité est actuellement disponible dans v2.1. preview. 

# <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1) 

Avec Model Compose (Composition de modèles), vous pouvez composer jusqu’à 100 modèles pour un même ID de modèle. Quand vous appelez la fonction d’analyse avec cet ID de modèle composé, Form Recognizer classifie d’abord le formulaire soumis, en le rapprochant du modèle qui correspond le mieux, puis retourne les résultats pour ce modèle. Cela est utile quand les formulaires entrants sont susceptibles d’appartenir à l’un des différents modèles.

Pour composer des modèles dans l’outil d’étiquetage des exemples, cliquez sur l’icône Model Compose (flèche de fusion) à gauche. À gauche, sélectionnez les modèles que vous souhaitez composer ensemble. Les modèles présentant une icône de flèche sont déjà des modèles composés. Cliquez sur le bouton « Compose » (Composer). Dans la fenêtre contextuelle, nommez votre nouveau modèle composé, puis cliquez sur « Compose » (Composer). Une fois l’opération terminée, le nouveau modèle composé apparaît dans la liste. 

:::image type="content" source="../media/label-tool/model-compose.png" alt-text="Vue UX de composition de modèles.":::

---

## <a name="analyze-a-form"></a>Analyser un formulaire 

Cliquez sur l’icône de prédiction (ampoule) sur la gauche pour tester votre modèle. Chargez un document de formulaire que vous n’avez pas utilisé dans le processus d’entraînement. Cliquez ensuite sur le bouton **Predict** à droite pour obtenir les prédictions de clé/valeur pour le formulaire. L’outil applique des étiquettes dans les cadres englobants et signale la confiance de chaque étiquette.

> [!TIP]
> Vous pouvez également exécuter l’API Analyze (Analyser) avec un appel REST. Pour savoir comment procéder, consultez [Effectuer un entraînement avec des étiquettes à l’aide de Python](./python-labeled-data.md).

## <a name="improve-results"></a>Améliorer les résultats

Selon la justesse signalée, vous souhaiterez peut-être effectuer d’autres entraînements pour améliorer le modèle. Une fois que vous avez effectué une prédiction, examinez les valeurs de confiance de chacune des étiquettes appliquées. Si la valeur d’entraînement de la justesse moyenne était élevée, mais que les scores de confiance sont faibles (ou que les résultats sont imprécis), vous devez ajouter le fichier utilisé pour la prédiction dans le jeu d’entraînement, l’étiqueter et renouveler l’entraînement.

La justesse moyenne signalée, les scores de confiance et la justesse réelle peuvent être incohérents quand les documents analysés diffèrent de ceux utilisés dans l’entraînement. N’oubliez pas que certains documents peuvent sembler similaires aux yeux de l’utilisateur, mais distincts du point de vue du modèle IA. Par exemple, vous pouvez effectuer l’entraînement avec un type de formulaire qui a deux variantes, où le jeu d’entraînement est constitué de 20 % de variante A et de 80 % de variante B. Au cours de la prédiction, les scores de confiance des documents de variante A sont susceptibles d’être inférieurs.

## <a name="save-a-project-and-resume-later"></a>Enregistrer un projet et le reprendre plus tard

Pour reprendre votre projet à un autre moment ou dans un autre navigateur, vous devez enregistrer son jeton de sécurité et le retaper ultérieurement. 

### <a name="get-project-credentials"></a>Récupérer les informations d’identification du projet
Accédez à la page des paramètres du projet (icône en forme de curseurs) et prenez note du nom du jeton de sécurité. Accédez ensuite aux paramètres d’application (icône en forme d’engrenage), où apparaissent tous les jetons de sécurité dans votre instance de navigateur actuelle. Recherchez le jeton de sécurité de votre projet et copiez son nom et sa valeur de clé en lieu sûr.

### <a name="restore-project-credentials"></a>Restaurer les informations d’identification du projet
Quand vous souhaitez reprendre votre projet, vous devez d’abord créer une connexion au même conteneur de stockage d’objets blob. Pour ce faire, répétez les étapes ci-dessus. Ensuite, accédez à la page des paramètres de l’application (icône en forme d’engrenage) et vérifiez si le jeton de sécurité de votre projet y figure. Si ce n’est pas le cas, ajoutez un nouveau jeton de sécurité et copiez le nom et la clé de votre jeton obtenus à l’étape précédente. Cliquez ensuite sur Save Settings (Enregistrer les paramètres). 

### <a name="resume-a-project"></a>Reprendre un projet

Enfin, accédez à la page principale (icône en forme de maison), puis cliquez sur Open Cloud Project (Ouvrir le projet cloud). Sélectionnez ensuite la connexion au stockage d’objets blob, puis le fichier *.fott* de votre projet. L’application chargera tous les paramètres du projet, car elle contient le jeton de sécurité.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser l’outil d’étiquetage des exemples Form Recognizer pour entraîner un modèle avec des données étiquetées manuellement. Si vous souhaitez créer votre propre utilitaire pour étiqueter des données d’apprentissage, utilisez les API REST dédiées à l’apprentissage de données étiquetées.

> [!div class="nextstepaction"]
> [Effectuer un entraînement avec des étiquettes à l’aide de Python](./python-labeled-data.md)

* [Qu’est-ce que Form Recognizer ?](../overview.md)
* [Démarrages rapides de bibliothèque de client Form Recognizer](client-library.md)
