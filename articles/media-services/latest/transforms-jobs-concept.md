---
title: Transformations et travaux dans Media Services
titleSuffix: Azure Media Services
description: Découvrez comment créer des transformations pour décrire les règles de traitement de vos vidéos dans Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/19/2019
ms.author: inhenkel
ms.openlocfilehash: 4cb4bcf5eb205d27cbca764d5ec16890a23be5c6
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426789"
---
# <a name="transforms-and-jobs-in-media-services"></a>Transformations et travaux dans Media Services

Cette rubrique fournit des informations détaillées sur les [transformations](/rest/api/media/transforms) et les [travaux](/rest/api/media/jobs) et explique la relation entre ces entités.

## <a name="overview"></a>Vue d’ensemble

### <a name="transformsjobs-workflow"></a>Flux de travail de transformations/travaux

Le schéma qui suit présente le flux de travail transformations/travaux :

![Flux de travail des transformations et travaux dans Azure Media Services](./media/encoding/transforms-jobs.png)

#### <a name="typical-workflow"></a>Flux de travail classique

1. Créer une transformation.
2. Envoyer des travaux sous cette transformation.
3. Répertorier les transformations.
4. Supprimer une transformation si vous ne prévoyez pas de l’utiliser ultérieurement.

#### <a name="example"></a>Exemple

Supposons que vous souhaitez extraire la première image de toutes vos vidéos sous la forme d’une image miniature. Vous devrez effectuer les étapes suivantes :

1. Définir la recette, ou la règle de traitement de vos vidéos : « utiliser la première image de la vidéo comme miniature ».
2. Pour chaque vidéo, vous devez indiquer au service :
    1. où trouver cette vidéo ;
    2. à quel emplacement écrire l’image miniature de sortie.

Une **transformation** vous permet de créer la recette une seule fois (Étape 1) et d’envoyer des travaux à l’aide de cette recette (Étape 2).

> [!NOTE]
> Les propriétés de **Transform** et **Job** (Transformation et Travail) de type DateHeure sont toujours au format UTC.

## <a name="transforms"></a>Transformations

Utilisez des **transformations** pour configurer des tâches courantes de codage ou d'analyse des vidéos. Chaque **transformation** décrit une recette, ou un flux de travail de tâches pour le traitement de vos fichiers vidéo ou audio. Une transformation unique peut appliquer plusieurs règles. Par exemple, une transformation peut spécifier que chaque vidéo est encodée dans un fichier MP4 à une vitesse de transmission donnée, et qu’une image miniature est générée à partir de la première image de la vidéo. Vous devez ajouter une entrée TransformOutput pour chaque règle que vous souhaitez inclure dans votre transformation. Vous utilisez des présélections pour indiquer à la transformation comment les fichiers multimédias d’entrée doivent être traités.

### <a name="viewing-schema"></a>Consultation du schéma

Dans Media Services v3, les présélections sont des entités fortement typées dans l’API elle-même. Vous trouverez la définition « schema » (schéma) pour ces objets dans [Open API Specification (ou Swagger)](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01). Vous pouvez également consulter les définitions prédéfinies (comme **StandardEncoderPreset** ) dans [l’API REST](/rest/api/media/transforms/createorupdate#standardencoderpreset), [le Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.management.media.models.standardencoderpreset?view=azure-dotnet) (ou d’autres documents de référence sur le Kit de développement logiciel (SDK) Media Services v3).

### <a name="creating-transforms"></a>Création de transformations

Vous pouvez créer des transformations à l’aide de REST ou de l’interface CLI, ou utiliser l’un des Kits de développement logiciel (SDK) publiés. L’API Azure Media Services v3 est pilotée par Azure Resource Manager. Vous pouvez donc utiliser des modèles Resource Manager pour créer et déployer des transformations dans votre compte Media Services. Le Contrôle d'accès en fonction du rôle Azure peut être utilisé pour bloquer l'accès aux transformations.

### <a name="updating-transforms"></a>Mise à jour de transformations

Si vous avez besoin mettre à jour votre [transformation](/rest/api/media/transforms), utilisez l’opération de **mise à jour**. Elle est destinée à modifier la description ou les priorités des TransformOutputs sous-jacents. Il est recommandé que ces mises à jour soient effectuées lorsque tous les travaux en cours sont terminés. Si vous souhaitez réécrire la recette, vous devez créer une nouvelle transformation.

### <a name="transform-object-diagram"></a>Schéma de l’objet de transformation

Le schéma suivant illustre l’objet de **transformation** et les objets qu’il référence avec les relations de dérivation. Les flèches grises montrent un type référencé par le travail, et les flèches vertes indiquent les relations de dérivation de classe.

Sélectionnez l’image pour l’afficher en plein écran.  

[![Schéma illustrant l’objet de transformation et les objets qu’il référence avec les relations de dérivation de classe entre les objets.](./media/api-diagrams/transform-small.png)](./media/api-diagrams/transform-large.png#lightbox)

## <a name="jobs"></a>travaux

Un **travail** est la demande réelle envoyée à Media Services pour appliquer la **transformation** à un contenu vidéo ou audio d’entrée donné. Lorsque la transformation est créée, vous pouvez envoyer des travaux à l’aide des API Media Services ou de l’un des kits de développement logiciel (SDK) publiés. Le **travail** spécifie des informations telles que l’emplacement de la vidéo d’entrée et celui de la sortie. Vous pouvez spécifier l'emplacement de votre vidéo d'entrée en utilisant : des URL HTTPS, des URL SAS ou des [éléments multimédia](/rest/api/media/assets).  

### <a name="job-input-from-https"></a>Entrée de travail à partir de HTTPS

Utilisez une [entrée de travail à partir de HTTPS](job-input-from-http-how-to.md) si votre contenu est déjà accessible via une URL et que vous n’avez pas besoin de stocker le fichier source dans Azure (par exemple, une importation à partir de S3). Cette méthode est également applicable si le contenu est dans le Stockage Blob Azure mais que le fichier ne doit pas nécessairement être dans un élément multimédia. Actuellement, cette méthode ne prend en charge qu’un fichier pour l’entrée.

### <a name="asset-as-job-input"></a>Élément multimédia en tant qu’entrée de travail

Utilisez un [élément multimédia en tant qu’entrée de travail](job-input-from-local-file-how-to.md) si le contenu d’entrée est déjà dans un élément multimédia ou si le contenu est stocké dans le fichier local. Il s’agit également d’une bonne option si vous projetez de publier l’élément multimédia d’entrée pour la diffusion en continu ou le téléchargement (supposons que vous souhaitiez publier le fichier mp4 pour téléchargement, mais que vous souhaitiez également utiliser la reconnaissance vocale à la détection des visages). Cette méthode prend en charge les éléments multimédias multifichiers (par exemple, les ensembles de contenus pour diffusion en continu MBR encodés localement).

### <a name="checking-job-progress"></a>Vérification de la progression des travaux

La progression et l’état des travaux peuvent être obtenus en surveillant les événements avec Event Grid. Pour plus d’informations, consultez la section relative à la [surveillance des événements à l’aide d’Event Grid](job-state-events-cli-how-to.md).

### <a name="updating-jobs"></a>Mise à jour des travaux

L’opération de mise à jour de l’entité [Travail](/rest/api/media/jobs) permet de modifier les propriétés *description* , et *priority* une fois le travail soumis. Une modification de la propriété *priority* ne s’applique que si le travail est toujours dans un état de file d’attente. Si le traitement du travail a commencé, ou est terminé, la modification de la priorité n’a aucun effet.

### <a name="job-object-diagram"></a>Schéma de l’objet de travail

Le schéma suivant illustre l’objet de **travail** et les objets qu’il référence avec les relations de dérivation.

Cliquez sur l’image pour l’afficher en plein écran.  

[![Schéma illustrant l’objet de travail et les objets qu’il référence avec les relations de dérivation de classe entre les objets.](./media/api-diagrams/job-small.png)](./media/api-diagrams/job-large.png#lightbox)

## <a name="configure-media-reserved-units"></a>Configurer des unités réservées Multimédia

Pour les travaux d’analyse audio et vidéo déclenchés par Media Services v3 ou Video Indexer, nous vous recommandons de provisionner votre compte avec des unité réservées Multimédia (MRU) 10 S3. Si vous avez besoin de plus de 10 MRU S3, ouvrez un ticket de support à l’aide du [Portail Azure](https://portal.azure.com/).

Pour plus de détails, voir [Mise à l’échelle du traitement multimédia avec l’interface CLI](media-reserved-units-cli-how-to.md).

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="see-also"></a>Voir aussi

* [Codes d’erreur](/rest/api/media/jobs/get#joberrorcode)
* [Filtrage, classement et pagination d’entités Media Services](entities-overview.md)

## <a name="next-steps"></a>Étapes suivantes

- Avant de commencer le développement, consultez [Développement avec les API Media Services v3](media-services-apis-overview.md) (informations sur l’accès aux API, les conventions d’affectation de noms, etc.)
- Consultez les didacticiels suivants :

    - [Tutoriel : Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu](stream-files-tutorial-with-rest.md)
    - [Tutoriel : Charger, encoder et diffuser des vidéos en continu](stream-files-tutorial-with-api.md)
    - [Tutoriel : Analyser des vidéos avec Media Services v3](analyze-videos-tutorial-with-api.md)
