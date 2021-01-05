---
title: Tutoriel sur les connecteurs Video Indexer avec Logic Apps et Power Automate.
description: Ce tutoriel montre comment bénéficier de nouvelles expériences et opportunités de monétisation grâce aux connecteurs Video Indexer avec Logic Apps et Power Automate.
author: anzaman
manager: johndeu
ms.author: alzam
ms.service: media-services
ms.subservice: video-indexer
ms.topic: tutorial
ms.date: 09/21/2020
ms.openlocfilehash: 5c545fa7d58d925dfcb9fa98b301c3bb0a20833e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358569"
---
# <a name="tutorial-use-video-indexer-with-logic-app-and-power-automate"></a>Tutoriel : utiliser Video Indexer avec Logic Apps et Power Automate

L’[API REST Video Indexer v2](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Delete-Video?) Azure Media Services prend en charge la communication de serveur à serveur et de client à serveur, et permet aux utilisateurs de Video Indexer d’intégrer facilement des insights vidéo et audio dans leur logique d’application pour profiter de nouvelles expériences et opportunités de monétisation.

Pour faciliter encore davantage l’intégration, nous prenons en charge les connecteurs [Logic Apps](https://azure.microsoft.com/services/logic-apps/) et [Power Automate](https://preview.flow.microsoft.com/connectors/shared_videoindexer-v2/video-indexer-v2/) qui sont compatibles avec notre API. Vous pouvez utiliser les connecteurs pour configurer des workflows personnalisés afin d’indexer et d’extraire efficacement des insights à partir d’une grande quantité de fichiers vidéo et audio, sans écrire une seule ligne de code. De plus, l’utilisation des connecteurs pour l’intégration offre une meilleure visibilité de l’intégrité de votre workflow et un moyen simple de le déboguer.  

Pour vous aider à bien démarrer avec les connecteurs Video Indexer, nous allons examiner pas à pas un exemple de solution Logic Apps et Power Automate que vous pouvez configurer. Ce tutoriel montre comment configurer des flux à l’aide de Logic Apps. Cependant, les éditeurs et les fonctionnalités sont presque identiques dans les deux solutions : les diagrammes et les explications s’appliquent donc à la fois à Logic Apps et à Power Automate.

Le scénario « Charger et indexer votre vidéo automatiquement » abordé dans ce tutoriel est constitué de deux flux différents qui fonctionnent ensemble. 
* Le premier flux est déclenché quand un objet blob est ajouté ou modifié dans un compte Stockage Azure. Il charge le nouveau fichier vers Video Indexer avec une URL de rappel pour envoyer une notification une fois l’opération d’indexation terminée. 
* Le deuxième flux est déclenché en fonction de l’URL de rappel ; il enregistre les insights extraites dans un fichier JSON dans Stockage Azure. Cette approche à deux flux est utilisée pour prendre en charge le chargement asynchrone et l’indexation des fichiers volumineux de manière efficace. 

Ce tutoriel utilise une application logique pour montrer comment :

> [!div class="checklist"]
> * Configurer le flux de chargement de fichier.
> * Configurer le flux d’extraction JSON.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Pour commencer, vous avez besoin d’un compte Video Indexer ainsi que d’un [accès aux API via la clé API](video-indexer-use-apis.md). 
* Vous aurez aussi besoin d’un compte Stockage Azure. Prenez note de la clé d’accès de votre compte de stockage. Créez deux conteneurs : un pour stocker les vidéos et un pour stocker les insights générées par Video Indexer.  
* Ensuite, vous devrez ouvrir deux flux distincts sur Logic Apps ou Power Automate (selon ce que vous utilisez). 

## <a name="set-up-the-first-flow---file-upload"></a>Configurer le premier flux – chargement de fichier   

Le premier flux est déclenché chaque fois qu’un objet blob est ajouté à votre conteneur Stockage Azure. Une fois déclenché, ce flux crée un URI SAS que vous pouvez utiliser pour charger et indexer la vidéo dans Video Indexer. Dans cette section, vous allez créer le flux suivant. 

![Flux de chargement de fichier](./media/logic-apps-connector-tutorial/file-upload-flow.png)

Pour configurer le premier flux, vous devez fournir votre clé API Video Indexer et les informations d’identification de Stockage Azure. 

![Stockage Blob Azure](./media/logic-apps-connector-tutorial/azure-blob-storage.png)

![Nom de la connexion et clé API](./media/logic-apps-connector-tutorial/connection-name-api-key.png)

> [!TIP]
> Si vous avez connecté un compte de stockage Azure ou un compte Video Indexer à une application logique, les détails de votre connexion sont stockés et vous êtes connecté automatiquement. <br/>Vous pouvez modifier la connexion en cliquant sur **Modifier la connexion** dans le bas d’une action Stockage Azure (la fenêtre du stockage) ou Video Indexer (la fenêtre du lecteur).

Une fois que vous pouvez vous connecter à vos comptes de stockage Azure et Video Indexer, recherchez et choisissez le déclencheur « Quand un blob est ajouté ou modifié » dans le **Concepteur Logic Apps**.

Sélectionnez le conteneur dans lequel vous allez placer vos fichiers vidéo. 

![La capture d’écran montre la boîte de dialogue Quand un blob est ajouté ou modifié, dans laquelle vous pouvez sélectionner un conteneur.](./media/logic-apps-connector-tutorial/container.png)

Ensuite, recherchez et sélectionnez l’action « Créer l’URI SAP par chemin ». Dans la boîte de dialogue de l’action, sélectionnez Liste de chemins de fichiers parmi les options de contenu dynamique.  

Ajoutez également un nouveau paramètre « Protocole d’accès partagé ». Choisissez HttpsOnly comme valeur du paramètre.

![URI SAS par chemin](./media/logic-apps-connector-tutorial/sas-uri-by-path.jpg)

Renseignez [l’emplacement](regions.md) et [l’ID de votre compte](./video-indexer-use-apis.md#account-id) pour obtenir le jeton du compte Video Indexer.

![Obtenir le jeton d’accès au compte](./media/logic-apps-connector-tutorial/account-access-token.png)

Pour « Charger une vidéo et indexer », renseignez l’URL de la vidéo et les paramètres requis. Sélectionnez « Ajouter un nouveau paramètre », puis URL de rappel. 

![Charger et indexer](./media/logic-apps-connector-tutorial/upload-and-index.png)

Laissez l’URL de rappel vide pour le moment. Vous l’ajouterez uniquement après avoir terminé le deuxième flux où l’URL de rappel est créée. 

Vous pouvez utiliser la valeur par défaut pour les autres paramètres ou les définir en fonction de vos besoins. 

Cliquez sur **Enregistrer**. Nous allons maintenant passer à la configuration du deuxième flux, afin d’extraire les insights une fois le chargement et l’indexation terminés. 

## <a name="set-up-the-second-flow---json-extraction"></a>Configurer le second flux – extraction JSON  

L’achèvement du chargement et de l’indexation à partir du premier flux enverra une requête HTTP avec l’URL de rappel correcte pour déclencher le second flux. Ensuite, ce flux récupère les insights générées par Video Indexer. Dans cet exemple, il stocke la sortie de votre travail d’indexation dans votre Stockage Azure.  Toutefois, c’est vous qui décidez de ce que vous pouvez faire avec la sortie.  

Créez le second flux distinct du premier. 

![Flux d’extraction JSON](./media/logic-apps-connector-tutorial/json-extraction-flow.png)

Pour configurer ce flux, vous devez fournir à nouveau votre clé API Video Indexer et les informations d’identification de Stockage Azure. Vous devez mettre à jour les mêmes paramètres que ceux du premier flux. 

Pour votre déclencheur, vous verrez un champ d’URL HTTP POST. L’URL ne sera générée qu’une fois que vous aurez enregistré votre flux ; toutefois, vous en aurez besoin à un moment donné. Nous y reviendrons. 

Renseignez [l’emplacement](regions.md) et [l’ID de votre compte](./video-indexer-use-apis.md#account-id) pour obtenir le jeton du compte Video Indexer.  

Accédez à l’action « Obtenir un index vidéo » et renseignez les paramètres requis. Pour ID vidéo, spécifiez l’expression suivante : triggerOutputs()['queries']['id'] 

![informations d’action Video Indexer](./media/logic-apps-connector-tutorial/video-indexer-action-info.jpg)

Cette expression indique au connecteur qu’il doit extraire l’ID vidéo à partir de la sortie de votre déclencheur. En l’occurrence, la sortie de votre déclencheur sera la sortie de « Charger une vidéo et indexer » dans votre premier déclencheur. 

Accédez à l’action « Créer un objet blob » et sélectionnez le chemin du dossier dans lequel vous enregistrerez les insights. Définissez le nom de l’objet blob que vous créez. Pour Contenu de l’objet blob, spécifiez l’expression suivante : body(‘Get_Video_Index’) 

![Action Créer un objet blob](./media/logic-apps-connector-tutorial/create-blob-action.jpg)

Cette expression prend la sortie de l’action « Obtenir un index vidéo » à partir de ce flux. 

Cliquez sur **Enregistrer le flux**. 

Une fois le flux enregistré, une URL HTTP POST est créée dans le déclencheur. Copiez l’URL à partir du déclencheur. 

![Enregistrer le déclencheur d’URL](./media/logic-apps-connector-tutorial/save-url-trigger.png)

À présent, revenez au premier flux et collez l’URL dans l’action « Charger une vidéo et indexer » pour le paramètre d’URL de rappel. 

Vérifiez que les deux flux sont enregistrés, et voilà, tout est prêt ! 

Essayez votre nouvelle solution Logic Apps ou Power Automate en ajoutant une vidéo à votre conteneur d’objets blob Azure, puis revenez après quelques minutes pour constater que les insights apparaissent dans le dossier de destination. 

## <a name="generate-captions"></a>Générer des légendes

Consultez le blog suivant pour connaître les étapes qui montrent [comment générer des légendes avec Video Indexer et Logic Apps](https://techcommunity.microsoft.com/t5/azure-media-services/generating-captions-with-video-indexer-and-logic-apps/ba-p/1672198). 

L’article montre également comment indexer une vidéo automatiquement en la copiant sur OneDrive et comment stocker les légendes générées par Video Indexer dans OneDrive.
 
## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé ce tutoriel, n’hésitez pas à conserver cette solution Logic Apps ou Power Automate si nécessaire. Toutefois, si vous ne voulez pas que cette solution continue à s’exécuter et ne souhaitez pas être facturé, désactivez vos deux flux si vous utilisez Power Automate. Désactivez les deux flux si vous utilisez Logic Apps. 

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel n’a présenté qu’un seul exemple de connecteur Video Indexer. Vous pouvez utiliser les connecteurs Video Indexer pour tout appel d’API fourni par Video Indexer par exemple pour charger et récupérer des insights, traduire les résultats, obtenir des widgets intégrables et même personnaliser vos modèles. Vous pouvez aussi choisir de déclencher ces actions en fonction de différentes sources, telles que des mises à jour de référentiels de fichiers ou des e-mails envoyés. Vous pouvez ensuite choisir de mettre à jour les résultats dans notre infrastructure ou application appropriée, ou de générer un nombre quelconque d’éléments d’action.  

> [!div class="nextstepaction"]
> [Utiliser l’API Video Indexer](video-indexer-use-apis.md)

Pour obtenir des ressources supplémentaires, reportez-vous à ce document sur [Video Indexer](https://docs.microsoft.com/connectors/videoindexer-v2/).
