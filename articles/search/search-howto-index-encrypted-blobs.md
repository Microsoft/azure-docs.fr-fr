---
title: Effectuer une recherche dans des documents chiffrés contenus dans Stockage Blob Azure
titleSuffix: Azure Cognitive Search
description: Apprenez à indexer et extraire du texte à partir de documents chiffrés contenus dans Stockage Blob Azure à l'aide du service Recherche cognitive Azure.
manager: nitinme
author: careyjmac
ms.author: chalton
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/02/2020
ms.openlocfilehash: 4bab8def514df21d948d67f3cfba846c43917be2
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530933"
---
# <a name="how-to-index-encrypted-blobs-using-blob-indexers-and-skillsets-in-azure-cognitive-search"></a>Indexer des objets blob chiffrés à l'aide d'indexeurs d'objets blob et d'ensembles de compétences dans le service Recherche cognitive Azure

Cet article vous explique comment utiliser le service [Recherche cognitive Azure](search-what-is-azure-search.md) pour indexer des documents préalablement chiffrés dans [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md) à l'aide d’[Azure Key Vault](../key-vault/general/overview.md). Normalement, un indexeur ne peut pas extraire le contenu des fichiers chiffrés car il n'a pas accès à la clé de chiffrement. Mais en tirant parti de la compétence personnalisée [DecryptBlobFile](https://github.com/Azure-Samples/azure-search-power-skills/blob/master/Utils/DecryptBlobFile) suivie de [DocumentExtractionSkill](cognitive-search-skill-document-extraction.md), vous pouvez fournir un accès contrôlé à la clé pour déchiffrer les fichiers, puis en extraire le contenu. Cela permet l’indexation de ces documents sans compromettre le statut de chiffrement de vos documents stockés.

En commençant par les documents entiers précédemment chiffrés (texte non structuré) tels que PDF, HTML, DOCX et PPTX dans le stockage d’objets BLOB Azure, ce guide utilise Postman et les API REST de recherche pour effectuer les tâches suivantes :

> [!div class="checklist"]
> * Définissez un pipeline qui déchiffre les documents et en extrait le texte.
> * Définissez un index pour stocker la sortie.
> * Exécutez le pipeline pour créer et charger l’index.
> * Explorez les résultats à l’aide de la recherche en texte intégral et d’une syntaxe de requête enrichie.

Si vous n’avez pas d’abonnement Azure, ouvrez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Cet exemple part du principe que vous avez déjà chargé vos fichiers sur le Stockage Blob Azure et qu'ils sont chiffrés. Si vous avez besoin d'aide pour charger et chiffrer vos fichiers, suivez [ce tutoriel](../storage/blobs/storage-encrypt-decrypt-blobs-key-vault.md).

+ [Stockage Azure](https://azure.microsoft.com/services/storage/)
+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) dans le même abonnement que Recherche cognitive Azure. La **suppression réversible** et la **protection contre le vidage** doivent été activées pour le coffre de clés.
+ [Recherche cognitive Azure](search-create-service-portal.md) sur un [niveau facturable](search-sku-tier.md#tier-descriptions) (de base ou version ultérieure, dans n’importe quelle région)
+ [Fonction Azure](https://azure.microsoft.com/services/functions/)
+ [Application de bureau Postman](https://www.getpostman.com/)

## <a name="1---create-services-and-collect-credentials"></a>1 - Créer des services et collecter les informations d'identification

### <a name="set-up-the-custom-skill"></a>Configurer la compétence personnalisée

Cet exemple utilise l'exemple de projet [DecryptBlobFile](https://github.com/Azure-Samples/azure-search-power-skills/blob/master/Utils/DecryptBlobFile) du référentiel GitHub [Super compétences de la Recherche Azure](https://github.com/Azure-Samples/azure-search-power-skills). Dans cette section, vous allez déployer la compétence sur une instance d'Azure Function afin qu'elle puisse être utilisée dans un ensemble de compétences. Un script de déploiement intégré crée une ressource Azure Function nommée avec le préfixe **psdbf-function-app-** et charge la compétence. Vous êtes ensuite invité à fournir un abonnement et un groupe de ressources. Veillez à choisir le même abonnement que celui auquel votre instance d'Azure Key Vault est associée.

Sur le plan opérationnel, la compétence DecryptBlobFile prend l'URL et le jeton SAS de chaque objet blob en tant qu'entrées, puis extrait le fichier téléchargé et déchiffré à l'aide du contrat de référence de fichier attendu par le service Recherche cognitive Azure. Pour rappel, DecryptBlobFile a besoin de la clé de chiffrement afin de procéder au déchiffrement. Dans le cadre de la configuration, vous allez également créer une stratégie d'accès qui accorde à la fonction DecryptBlobFile l'accès à la clé de chiffrement dans Azure Key Vault.

1. Cliquez sur le bouton **Déployer sur Azure** situé sur la [page de destination de DecryptBlobFile](https://github.com/Azure-Samples/azure-search-power-skills/blob/master/Utils/DecryptBlobFile#deployment) pour ouvrir le modèle Resource Manager fourni sur le portail Azure.

1. Sélectionnez **l'abonnement auquel votre instance Azure Key Vault est associée** (ce guide ne fonctionnera pas si vous sélectionnez un autre abonnement), puis sélectionnez un groupe de ressources existant ou créez-en un (si vous en créez un, vous devrez également sélectionner la région dans laquelle le déploiement sera effectué).

1. Sélectionnez **Vérifier + créer**, acceptez les conditions, puis sélectionnez **Créer** pour déployer l'instance d'Azure Function.

    ![Modèle Resource Manager sur le portail](media/indexing-encrypted-blob-files/arm-template.jpg "Modèle Resource Manager sur le portail")

1. Attendez que le déploiement se termine.

1. Sur le portail, accédez à votre instance d'Azure Key Vault. Dans Azure Key Vault, [créez une stratégie d'accès](../key-vault/general/assign-access-policy-portal.md) qui accorde un accès à la clé de la compétence personnalisée.
 
    1. Sous **Paramètres**, sélectionnez **Stratégies d'accès**, puis **Ajouter une stratégie d'accès**.
     
       ![Keyvault - Ajouter une stratégie d'accès](media/indexing-encrypted-blob-files/keyvault-access-policies.jpg "Keyvault - Stratégies d'accès")

    1. Sous **Configurer à partir du modèle**, sélectionnez **Azure Data Lake Storage ou Stockage Azure**.

    1. Pour le principal, sélectionnez l'instance d'Azure Function que vous avez déployée. Vous pouvez la rechercher à l'aide du préfixe de ressource utilisé pour la créer à l'étape 2. La valeur par défaut du préfixe est **psdbf-function-app**.

    1. Ne sélectionnez rien pour l'option d'application autorisée.
     
        ![Keyvault - Modèle d'ajout de stratégie d'accès](media/indexing-encrypted-blob-files/keyvault-add-access-policy.jpg "Keyvault - Modèle de stratégie d'accès")

    1. Cliquez sur **Enregistrer** avant de quitter la page des stratégies d'accès afin de valider l'ajout de la stratégie.
     
         ![Keyvault - Enregistrer la stratégie d'accès](media/indexing-encrypted-blob-files/keyvault-save-access-policy.jpg "Keyvault - Enregistrer la stratégie d'accès")

1. Sur le portail, accédez à la fonction **psdbf-function-app** et notez les propriétés suivantes, car vous en aurez besoin plus loin dans ce guide :

    1. L'URL de la fonction, qui se trouve sous **Essentials** sur la page principale de la fonction.
    
        ![URL de la fonction](media/indexing-encrypted-blob-files/function-uri.jpg "Où trouver l'URL Azure Function")

    1. Le code de la clé hôte, que vous pourrez vous procurer en accédant à **Clés d'application**, en cliquant pour afficher la clé **par défaut** et en copiant la valeur.
     
        ![Code de la clé hôte de la fonction](media/indexing-encrypted-blob-files/function-host-key.jpg "Où trouver le code de la clé hôte Azure Function")

### <a name="cognitive-services"></a>Cognitive Services

L'enrichissement par IA et l'exécution des ensembles de compétences s'appuient sur Cognitive Services, notamment sur Analyse de texte et Vision par ordinateur pour le traitement des images et du langage naturel. Si votre objectif était de réaliser un prototype ou un projet réel, vous devriez à ce stade provisionner Cognitive Services (dans la même région que Recherche cognitive Azure) afin de pouvoir l’attacher aux opérations d’indexation.

Cependant, dans le cadre de cet exercice, vous pouvez ignorer le provisionnement des ressources, car Recherche cognitive Azure peut se connecter à Cognitive Services en arrière-plan et vous fournir 20 transactions gratuites par exécution de l’indexeur. Après avoir traité 20 documents, l'indexeur échouera, sauf si une clé Cognitive Services est jointe à l'ensemble de compétences. Pour les projets de plus grande envergure, prévoyez de provisionner Cognitive Services au niveau S0 du paiement à l’utilisation. Pour plus d’informations, consultez [Attacher Cognitive Services](cognitive-search-attach-cognitive-services.md). Notez qu'une clé Cognitive Services est nécessaire pour exécuter un ensemble de compétences couvrant plus de 20 documents, même si aucune des compétences cognitives que vous avez sélectionnées n'est liée à Cognitive Services (comme avec l'ensemble de compétences fourni si aucune compétence n'est ajoutée).

### <a name="azure-cognitive-search"></a>Recherche cognitive Azure

Le dernier composant est Recherche cognitive Azure, que vous pouvez [créer sur le portail](search-create-service-portal.md). Vous pouvez utiliser le niveau Gratuit pour suivre ce guide. 

Comme pour l'instance d'Azure Function, prenez un moment pour récupérer la clé d'administration. Par ailleurs, lorsque vous commencez à structurer les demandes, vous devez fournir le point de terminaison et la clé API d’administration utilisés pour authentifier chaque demande.

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Obtenir une clé API d’administration et une URL pour Recherche cognitive Azure

1. [Connectez-vous au Portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez le nom de votre service de recherche. Vous pouvez confirmer le nom de votre service en passant en revue l’URL du point de terminaison. Si votre URL de point de terminaison est `https://mydemo.search.windows.net`, le nom du service doit être `mydemo`.

2. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

   ![Obtenir le nom du service, les clés d’administration et les clés de requête](media/search-get-started-javascript/service-name-and-keys.png)

Une clé API est nécessaire dans l’en-tête de chaque requête envoyée à votre service. Une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="2---set-up-postman"></a>2 - Configurer Postman

Installez et configurez Postman.

### <a name="download-and-install-postman"></a>Télécharger et installer Postman

1. Téléchargez le [code source de la collection Postman](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/index-encrypted-blobs/Index%20encrypted%20Blob%20files.postman_collection.json).
1. Sélectionnez **Fichier** > **Importer** pour importer le code source dans Postman.
1. Sélectionnez l’onglet **Collections**, puis le bouton **...** (points de suspension).
1. Sélectionnez **Modifier**. 
   
   ![Application Postman montrant la navigation](media/indexing-encrypted-blob-files/postman-edit-menu.jpg "Accéder au menu Edition dans Postman")
1. Dans la boîte de dialogue **Edit**, sélectionnez l’onglet **Variables**. 

Sous l’onglet **Variables**, vous pouvez ajouter les valeurs que Postman récupère chaque fois qu’il trouve une variable spécifique entre double accolades. Par exemple, Postman remplace le symbole `{{admin-key}}` par la valeur actuelle que vous avez définie pour `admin-key`. Postman effectue la substitution dans les URL, les en-têtes, le corps de la requête, etc. 

Pour obtenir la valeur de la clé d'administration (`admin-key`), utilisez la clé API d'administration Recherche cognitive Azure notée précédemment. Définissez `search-service-name` sur le nom du service Recherche cognitive Azure que vous utilisez. Définissez `storage-connection-string` en utilisant la valeur disponible dans l'onglet **Clés d'accès** de votre compte de stockage, et définissez `storage-container-name` sur le nom du conteneur d'objets blob où les fichiers chiffrés sont stockés sur ce compte de stockage. Définissez `function-uri` sur l'URL Azure Function que vous avez notée précédemment, et définissez `function-code` sur le code de la clé hôte Azure Function que vous avez également noté précédemment. Partout ailleurs, vous pouvez conserver les valeurs par défaut.

![Onglet Variables de l’application Postman](media/indexing-encrypted-blob-files/postman-variables-window.jpg "Fenêtre des variables de Postman")

| Variable    | Comment les obtenir |
|-------------|-----------------|
| `admin-key` | Dans la page **Clés** du service Recherche cognitive Azure.  |
| `search-service-name` | Nom du service Recherche cognitive Azure. L’URL est `https://{{search-service-name}}.search.windows.net`. |
| `storage-connection-string` | Dans le compte de stockage, sous l’onglet **Clés d’accès**, sélectionnez **key1** > **Chaîne de connexion**. |
| `storage-container-name` | Nom du conteneur d'objets blob qui contient les fichiers chiffrés à indexer. |
| `function-uri` |  Dans Azure Function, sous **Essentials** sur la page principale. |
| `function-code` | Dans Azure Function, en accédant à **Clés d'application**, en cliquant pour afficher la clé **par défaut** et en copiant la valeur. |
| `api-version` | Laisser **2020-06-30**. |
| `datasource-name` | Laisser **encrypted-blobs-ds**. |
| `index-name` | Laisser **encrypted-blobs-idx**. |
| `skillset-name` | Laisser **encrypted-blobs-ss**. |
| `indexer-name` | Laisser **encrypted-blobs-ixr**. |

### <a name="review-the-request-collection-in-postman"></a>Passer en revue la collection de requêtes dans Postman

Dans le cadre de ce guide, vous devez émettre quatre requêtes HTTP :

- **Une requête PUT pour créer l’index** : Cet index contient les données utilisées et retournées par Recherche cognitive Azure.
- **Une requête POST pour créer la source de données** : Cette source de données relie votre service Recherche cognitive Azure à votre compte de stockage et donc aux fichiers blob chiffrés. 
- **Une requête PUT pour créer l’ensemble de compétences** : L'ensemble de compétences spécifie la définition de compétence personnalisée relative à l'instance d'Azure Function qui déchiffrera les données des fichiers blob, et une fonction [DocumentExtractionSkill](cognitive-search-skill-document-extraction.md) pour extraire le texte de chaque document une fois celui-ci déchiffré.
- **Une requête PUT pour créer l’indexeur** : L’exécution de l’indexeur lit les données, applique l’ensemble de compétences et stocke les résultats. Vous devez exécuter cette requête en dernier.

Le [code source](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/index-encrypted-blobs/Index%20encrypted%20Blob%20files.postman_collection.json) contient une collection Postman avec ces quatre requêtes, ainsi que quelques requêtes de suivi utiles. Pour émettre les requêtes, dans Postman, sélectionnez l'onglet correspondant aux requêtes et cliquez sur **Envoyer** pour chacune d'entre elles.

## <a name="3---monitor-indexing"></a>4 - Surveiller l'indexation

L’indexation et l’enrichissement commencent dès que vous envoyez la demande de création d’indexeur. Selon le nombre de documents présents sur votre compte de stockage, l'indexation peut prendre un certain temps. Pour savoir si l'indexeur est toujours en cours d'exécution, utilisez la requête **Get Indexer Status** fournie dans le cadre de la collection Postman, puis examinez la réponse pour savoir si l'indexeur est en cours d'exécution ou pour consulter les erreurs et avertissements.  

Si vous utilisez le niveau Gratuit, le message suivant doit s'afficher : `"Could not extract content or metadata from your document. Truncated extracted text to '32768' characters"`. Ce message s'affiche car l'indexation d'objets blob au niveau Gratuit présente une [limite d'extraction de caractères de 32 000](search-limits-quotas-capacity.md#indexer-limits). Vous ne verrez pas ce message pour ce jeu de données sur des niveaux supérieurs. 

## <a name="4---search"></a>4 - Rechercher

Au terme de l'exécution de l'indexeur, vous pouvez lancer quelques requêtes pour vérifier que les données ont été correctement déchiffrées et indexées. Accédez à votre service Recherche cognitive Azure sur le portail et utilisez l'[explorateur de recherche](search-explorer.md) pour lancer des requêtes sur les données indexées.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez correctement indexé les fichiers chiffrés, vous pouvez [effectuer une itération sur ce pipeline en ajoutant des compétences cognitives](cognitive-search-defining-skillset.md). Cela vous permettra d'enrichir vos données et de bénéficier d'insights supplémentaires sur celles-ci.

Si vous utilisez des données doublement chiffrées, vous avez la possibilité d'étudier les fonctionnalités de chiffrement d'index disponibles dans Recherche cognitive Azure. Bien que l'indexeur ait besoin de données déchiffrées pour l'indexation, une fois que l'index existe, il peut être chiffré à l'aide d'une clé gérée par le client. Ainsi, vos données seront toujours chiffrées lorsqu'elles sont au repos. Pour plus d'informations, consultez [Configurer des clés gérées par le client pour le chiffrement des données dans Recherche cognitive Azure](search-security-manage-encryption-keys.md).