---
title: Créer des déclencheurs basés sur un événement dans Azure Data Factory
description: Découvrez comment créer un déclencheur dans Azure Data Factory qui exécute un pipeline en réponse à un événement.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
author: chez-charlie
ms.author: chez
manager: jroth
ms.reviewer: maghan
ms.topic: conceptual
ms.date: 10/18/2018
ms.openlocfilehash: de416277de34e1c3717d581697f05c98c48d1959
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146005"
---
# <a name="create-a-trigger-that-runs-a-pipeline-in-response-to-an-event"></a>Créer un déclencheur qui exécute un pipeline en réponse à un événement
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit les déclencheurs basés sur un événement que vous pouvez créer dans vos pipelines Data Factory.

L’architecture basée sur les événements (EDA) est un modèle d’intégration de données courant qui implique la production, la détection, la consommation et la réaction à des événements. Les scénarios d’intégration de données requièrent souvent des clients Data Factory clients qu’ils déclenchent des pipelines basés sur des événements tels que l’arrivée ou la suppression d’un fichier dans votre compte de stockage Azure. Data Factory est désormais intégré à [Azure Event Grid](https://azure.microsoft.com/services/event-grid/), qui vous permet de déclencher des pipelines sur un événement.

Pour afficher une présentation de dix minutes et la démonstration de cette fonctionnalité, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Event-based-data-integration-with-Azure-Data-Factory/player]


> [!NOTE]
> L’intégration décrite dans cet article dépend [d’Azure Event Grid](https://azure.microsoft.com/services/event-grid/). Vérifiez que votre abonnement est inscrit auprès du fournisseur de ressources Event Grid. Pour plus d’informations, consultez [Types et fournisseurs de ressources](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal). Vous devez pouvoir effectuer l’action *Microsoft.EventGrid/eventSubscriptions/* *. Cette action fait partie du rôle intégré Contributeur EventGrid EventSubscription.

## <a name="data-factory-ui"></a>IU de la fabrique de données

Cette section vous montre comment créer un déclencheur d’événements dans l’interface utilisateur Azure Data Factory.

1. Allez à **Canevas de création**

1. Dans le coin inférieur gauche, cliquez sur le bouton **Déclencheurs**

1. Cliquez sur **+ nouveau** pour ouvrir le panneau Créer un déclencheur

1. Sélectionner le type de déclencheur **Événement**

    ![Créer un déclencheur d’événement](media/how-to-create-event-trigger/event-based-trigger-image1.png)

1. Sélectionnez votre compte de stockage dans la liste déroulante abonnement Azure ou manuellement à l’aide de son ID de ressource de compte de stockage. Choisissez le conteneur sur lequel vous souhaitez que les événements se produisent. La sélection du conteneur est facultative, mais gardez à l’esprit que la sélection de tous les conteneurs peut entraîner un grand nombre d’événements.

   > [!NOTE]
   > Le déclencheur d’événements prend actuellement en charge uniquement Azure Data Lake Storage Gen2 et des comptes de stockages de version 2 (usage général). Vous devez disposer au minimum de l’accès *Propriétaire* au compte de stockage.  En raison d’une limitation de Azure Event Grid, Azure Data Factory ne prend en charge qu’un maximum de 500 déclencheurs d’événements par compte de stockage.

1. Avec les propriétés **Blob path begins with** (Chemin d’accès de l’objet blob commence par) et **Blob path ends with** (Chemin d’accès de l’objet blob se termine par), vous pouvez spécifier les conteneurs, les dossiers et les noms d’objets blob pour lesquels vous souhaitez recevoir des événements. Votre déclencheur d’événement requiert la définition d’au moins une de ces propriétés. Vous pouvez utiliser divers modèles pour les deux propriétés **Blob path begins with** (Chemin d’accès de l’objet blob commence par) et **Blob path ends with** (Chemin d’accès de l’objet blob se termine par), comme indiqué dans les exemples plus loin dans cet article.

    * **Blob path begins with:** (Chemin d’accès de l’objet blob commence par) Le chemin d’accès de l’objet blob doit commencer par un chemin d’accès au dossier. Les valeurs autorisées comprennent `2018/` et `2018/april/shoes.csv`. Ce champ ne peut pas être sélectionné si aucun conteneur n’est sélectionné.
    * **Blob path ends with:** (Chemin d’accès de l’objet blob se termine par) Le chemin d’accès de l’objet blob doit se terminer par un nom de fichier ou une extension. Les valeurs autorisées comprennent `shoes.csv` et `.csv`. Le nom du conteneur et du dossier sont facultatifs mais, s’ils sont spécifiés, ils doivent être séparés par un segment `/blobs/`. Par exemple, un conteneur nommé « Orders » peut avoir la valeur `/orders/blobs/2018/april/shoes.csv`. Pour spécifier un dossier dans n’importe quel conteneur, omettez le caractère « / » de début. Par exemple, `april/shoes.csv` déclenche un événement sur tout fichier nommé `shoes.csv` dans le dossier appelé « avril » dans n’importe quel conteneur. 

1. Indiquez si votre déclencheur répondra à un événement **créé par un objet Blob** , **supprimé par un objet Blob** ou les deux. Dans l’emplacement de stockage spécifié, chaque événement déclenchera les pipelines Data Factory associés au déclencheur.

    ![Configurer le déclencheur d’événement](media/how-to-create-event-trigger/event-based-trigger-image2.png)

1. Indiquez si votre déclencheur ignore ou non les objets blob de zéro octet.

1. Une fois que vous avez configuré le déclencheur, cliquez sur **Suivant : Aperçu des données**. Cet écran affiche les objets Blob existants correspondant à la configuration de votre déclencheur d’événements. Assurez-vous d’avoir des filtres spécifiques. La configuration de filtres trop larges peut correspondre à un grand nombre de fichiers créés/supprimés et peut avoir un impact significatif sur vos coûts. Une fois que vos conditions de filtre ont été vérifiées, cliquez sur **Terminer**.

    ![Aperçu des données du déclencheur d’événements](media/how-to-create-event-trigger/event-based-trigger-image3.png)

1. Pour attacher un pipeline à ce déclencheur, accédez à la zone de dessin du pipeline , cliquez sur **Ajouter un déclencheur** , puis sélectionnez **Nouveau/Modifier**. Lorsque la navigation latérale s’affiche, cliquez sur le menu déroulant **Choisir le déclencheur...** et sélectionnez le déclencheur que vous avez créé. Cliquez sur **Suivant : Aperçu des données** pour confirmer que la configuration est correcte, puis cliquez sur **Suivant** pour valider que l’aperçu des données est correct.

1. Si votre pipeline possède des paramètres, vous pouvez les spécifier dans la barre de navigation latérale du paramètre d’exécution du déclencheur. Le déclencheur d’événement capture le chemin de dossier et le nom de fichier de l’objet blob dans les propriétés `@triggerBody().folderPath` et `@triggerBody().fileName`. Pour utiliser les valeurs de ces propriétés dans un pipeline, vous devez mapper les propriétés aux paramètres de pipeline. Après le mappage des propriétés aux paramètres, vous pouvez accéder aux valeurs capturées par le déclencheur à l’aide de l’expression `@pipeline().parameters.parameterName` tout au long du pipeline. Une fois que vous avez terminé, cliquez sur **Terminer**.

    ![Mappage des propriétés aux paramètres de pipeline](media/how-to-create-event-trigger/event-based-trigger-image4.png)

Dans l’exemple précédent, le déclencheur est configuré pour se déclencher lorsqu’un chemin d’accès à un objet Blob se terminant par .csv est créé dans le dossier event-testing du conteneur sample-data. Les propriétés **folderPath** et **filename** capturent l’emplacement du nouvel objet Blob. Par exemple, lorsque MoviesDB.csv est ajouté au chemin d’accès sample-data/event-testing, `@triggerBody().folderPath` a la valeur de `sample-data/event-testing` et `@triggerBody().fileName` a la valeur de `moviesDB.csv`. Ces valeurs sont mappées dans l’exemple aux paramètres de pipeline `sourceFolder` et `sourceFile` peuvent être utilisés dans l’ensemble du pipeline en tant que `@pipeline().parameters.sourceFolder` et `@pipeline().parameters.sourceFile` respectivement.

## <a name="json-schema"></a>Schéma JSON

Le tableau suivant fournit une vue d’ensemble des éléments de schéma associés aux déclencheurs basés sur un événement :

| **Élément JSON** | **Description** | **Type** | **Valeurs autorisées** | **Obligatoire** |
| ---------------- | --------------- | -------- | ------------------ | ------------ |
| **scope** | ID de ressource Azure Resource Manager du compte de stockage. | String | ID d’Azure Resource Manager | Oui |
| **events** | Type des événements qui entraîne l’activation de ce déclencheur. | Array    | Microsoft.Storage.BlobCreated, Microsoft.Storage.BlobDeleted | Oui, n’importe quelle combinaison de ces valeurs. |
| **blobPathBeginsWith** | Le chemin d’accès de l’objet blob doit commencer par le modèle fourni pour activer le déclencheur. Par exemple, `/records/blobs/december/` active uniquement le déclencheur pour les objets blob du dossier `december` sous le conteneur `records`. | String   | | Vous devez indiquer une valeur pour l’une de ces propriétés au moins : `blobPathBeginsWith` ou `blobPathEndsWith`. |
| **blobPathEndsWith** | Le chemin d’accès de l’objet blob doit se terminer par le modèle fourni pour activer le déclencheur. Par exemple, `december/boxes.csv` active uniquement le déclencheur pour les objets blob nommés `boxes` dans un dossier `december`. | String   | | Vous devez indiquer une valeur pour l’une de ces propriétés au moins : `blobPathBeginsWith` ou `blobPathEndsWith`. |
| **ignoreEmptyBlobs** | Indique si des objets blob de zéro octet déclenchent ou non une exécution de pipeline. Valeur par défaut : true. | Boolean | True ou False | Non |

## <a name="examples-of-event-based-triggers"></a>Exemples de déclencheurs basés sur un événement

Cette section fournit des exemples de paramètres de déclencheur basé sur un événement.

> [!IMPORTANT]
> Vous devez inclure le segment `/blobs/` du chemin, comme indiqué dans les exemples suivants, chaque fois que vous spécifiez conteneur et dossier, conteneur et fichier, ou conteneur, dossier et fichier. Pour **blobPathBeginsWith** , l’interface utilisateur de Data Factory ajoute automatiquement `/blobs/` entre le nom du dossier et du conteneur dans le JSON du déclencheur.

| Propriété | Exemple | Description |
|---|---|---|
| **Blob path begins with** | `/containername/` | Reçoit les événements de tout objet blob du conteneur. |
| **Blob path begins with** | `/containername/blobs/foldername/` | Reçoit les événements de tout objet blob du conteneur `containername` et du dossier `foldername`. |
| **Blob path begins with** | `/containername/blobs/foldername/subfoldername/` | Vous pouvez également référencer un sous-dossier. |
| **Blob path begins with** | `/containername/blobs/foldername/file.txt` | Reçoit les événements d’un objet blob nommé `file.txt` dans le dossier `foldername`, sous le conteneur `containername`. |
| **Blob path ends with** | `file.txt` | Reçoit les événements d’un objet blob nommé `file.txt` dans n’importe quel chemin d’accès. |
| **Blob path ends with** | `/containername/blobs/file.txt` | Reçoit les événements d’un objet blob nommé `file.txt` sous le conteneur `containername`. |
| **Blob path ends with** | `foldername/file.txt` | Reçoit les événements d’un objet blob nommé `file.txt`, dans le dossier `foldername`, sous n’importe quel conteneur. |

## <a name="next-steps"></a>Étapes suivantes
Vous trouverez des informations détaillées sur les déclencheurs sur la page [Exécution de pipelines et déclencheurs](concepts-pipeline-execution-triggers.md#trigger-execution).
