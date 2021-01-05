---
title: Tutoriel - Désérialiseurs .NET personnalisés pour les travaux cloud Azure Stream Analytics
description: Ce tutoriel montre comment créer un désérialiseur .NET personnalisé pour un travail cloud Azure Stream Analytics à l’aide de Visual Studio.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.date: 05/06/2019
ms.openlocfilehash: 1fffeec1434cb066487bf383589554edec2e6a86
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75443690"
---
# <a name="tutorial-custom-net-deserializers-for-azure-stream-analytics"></a>Tutoriel : Désérialiseurs .NET personnalisés pour Azure Stream Analytics

Azure Stream Analytics dispose d’une [prise en charge intégrée de trois formats de données](stream-analytics-parsing-json.md) : JSON, CSV et Avro. Avec les désérialiseurs .NET personnalisés, vous pouvez lire des données à partir d’autres formats, par exemple le [tampon de protocole](https://developers.google.com/protocol-buffers/), [Bond](https://github.com/Microsoft/bond) et d’autres formats définis par l’utilisateur pour les travaux cloud et de périphérie.

Ce tutoriel montre comment créer un désérialiseur .NET personnalisé pour un travail cloud Azure Stream Analytics à l’aide de Visual Studio. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un désérialiseur personnalisé pour le tampon de protocole.
> * Créer un travail Azure Stream Analytics dans Visual Studio.
> * Configurer votre travail Stream Analytics pour utiliser le désérialiseur personnalisé.
> * Exécuter votre travail Stream Analytics localement pour tester le désérialiseur personnalisé.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Installez [Visual Studio 2017](https://www.visualstudio.com/downloads/) ou [Visual Studio 2015](https://www.visualstudio.com/vs/older-downloads/). Les éditions Enterprise (Ultimate/Premium), Professional et Community sont prises en charge. L’édition Express n’est pas prise en charge.

* [Installez les outils Stream Analytics pour Visual Studio](stream-analytics-tools-for-visual-studio-install.md) ou effectuez une mise à jour vers la dernière version. Les versions suivantes de Visual Studio sont prises en charge :
   * Visual Studio 2015
   * Visual Studio 2017

* Ouvrez **Cloud Explorer** dans Visual Studio, puis connectez-vous à votre abonnement Azure.

* Créez un conteneur dans votre compte de Stockage Azure.
Le conteneur que vous créez sera utilisé pour stocker les ressources associées à votre travail Stream Analytics. Si vous avez déjà un compte de stockage avec des conteneurs existants, vous pouvez les utiliser. Sinon, vous devez [créer un conteneur](../storage/blobs/storage-quickstart-blobs-portal.md).

## <a name="create-a-custom-deserializer"></a>Créer un désérialiseur personnalisé

1. Ouvrez Visual Studio, puis sélectionnez **Fichier > Nouveau > Projet**. Recherchez **Stream Analytics**, puis sélectionnez **Projet de désérialiseur personnalisé (.NET) Azure Stream Analytics**. Donnez un nom au projet, par exemple **Désérialiseur Protobuf**.

   ![Créer un projet de bibliothèque de classes dotnet standard Visual Studio](./media/custom-deserializer/create-dotnet-library-project.png)

2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre projet **Désérialiseur Protobuf**, puis sélectionnez **Gérer les packages NuGet** dans le menu. Installez ensuite les packages NuGet **Microsoft.Azure.StreamAnalytics** et **Google.Protobuf**.

3. Ajoutez la [classe MessageBodyProto](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyProto.cs) et la [classe MessageBodyDeserializer ](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyDeserializer.cs) à votre projet.

4. Générez le projet **Désérialiseur Protobuf**.

## <a name="add-an-azure-stream-analytics-project"></a>Ajouter un projet Azure Stream Analytics

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la solution **Désérialiseur Protobuf**, puis sélectionnez **Ajouter > Nouveau projet**. Sous **Azure Stream Analytics > Stream Analytics**, choisissez **Application Azure Stream Analytics**. Nommez-le **ProtobufCloudDeserializer**, puis sélectionnez **OK**. 

2. Cliquez avec le bouton droit sur **Références** sous le projet Azure Stream Analytics **ProtobufCloudDeserializer**. Sous **Projets**, ajoutez **Désérialiseur Protobuf**. Cette valeur doit être renseignée automatiquement.

## <a name="configure-a-stream-analytics-job"></a>Configurer un travail Stream Analytics

1. Double-cliquez sur **JobConfig.json**. Utilisez les configurations par défaut, sauf pour les paramètres suivants :

   |Paramètre|Valeur suggérée|
   |-------|---------------|
   |Ressource des paramètres de stockage généraux|Choisir une source de données du compte actuel|
   |Abonnement des paramètres de stockage généraux| < votre abonnement >|
   |Compte de stockage des paramètres de stockage généraux| < votre compte de stockage >|
   |Ressource des paramètres de stockage de code personnalisé|Choisir une source de données du compte actuel|
   |Compte de stockage des paramètres de stockage de code personnalisé|< votre compte de stockage >|
   |Conteneur des paramètres de stockage de code personnalisé|< votre conteneur de stockage >|

2. Sous **Entrées**, double-cliquez sur **Input.json**. Utilisez les configurations par défaut, sauf pour les paramètres suivants :

   |Paramètre|Valeur suggérée|
   |-------|---------------|
   |Source|Stockage Blob|
   |Ressource|Choisir une source de données du compte actuel|
   |Abonnement|< votre abonnement >|
   |Compte de stockage|< votre compte de stockage >|
   |Conteneur|< votre conteneur de stockage >|
   |Format de sérialisation de l’événement|Autre (Protobuf, XML, propriétaire...)|
   |Ressource|Charger à partir de la référence de projet ASA ou du code-behind|
   |Nom de l’assembly CSharp|ProtobufDeserializer.dll|
   |Nom de la classe|MessageBodyProto.MessageBodyDeserializer|
   |Type de compression d’événement|None|

3. Ajoutez la requête suivante au fichier **Script.asaql**.

   ```sql
   SELECT * FROM Input
   ```

4. Téléchargez l’[exemple de fichier d’entrée protobuf](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/SimulatedTemperatureEvents.protobuf). Dans le dossier **Entrées**, cliquez avec le bouton droit sur **Input.json**, puis sélectionnez **Ajouter une entrée locale**. Double-cliquez ensuite sur **local_Input.json**, puis configurez les paramètres suivants :

   |Paramètre|Valeur suggérée|
   |-------|---------------|
   |Alias d’entrée|Entrée|
   |Type de source|Flux de données|
   |Format de sérialisation de l’événement|Autre (Protobuf, XML, propriétaire...)|
   |Nom de l’assembly CSharp|ProtobufDeserializer.dll|
   |Nom de la classe|MessageBodyProto.MessageBodyDeserializer|
   |Chemin du fichier d’entrée local|< le chemin de l’exemple de fichier d’entrée protobuf téléchargé >|

## <a name="execute-the-stream-analytics-job"></a>Exécuter le travail Stream Analytics

1. Ouvrez **Script.asaql**, puis sélectionnez **Exécuter localement**.

2. Observez les résultats dans **Résultats de l’exécution locale de Stream Analytics**.

Vous avez réussi à implémenter un désérialiseur personnalisé pour votre travail Stream Analytics. Dans ce tutoriel, vous avez testé le désérialiseur personnalisé localement. Pour vos données réelles, vous devez configurer correctement l’entrée et la sortie. Envoyez ensuite votre travail à Azure à partir de Visual Studio pour l’exécuter dans le cloud à l’aide du désérialiseur personnalisé que vous venez d’implémenter.

## <a name="debug-your-deserializer"></a>Déboguer votre désérialiseur

Vous pouvez déboguer votre désérialiseur .NET localement de la même façon que vous déboguez du code .NET. 

1. Ajoutez des points d’arrêt à votre fonction.

2. Appuyez sur **F5** pour démarrer le débogage. Le programme s’arrête sur vos points d’arrêt comme prévu.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le travail de streaming et toutes les ressources associées. La suppression du travail évite la facturation des unités de streaming consommées par le travail. Si vous envisagez d’utiliser le travail à l’avenir, vous pouvez l’arrêter et le redémarrer plus tard lorsque vous en avez besoin. Si vous ne pensez pas continuer à utiliser ce travail, supprimez toutes les ressources créées au cours de ce didacticiel en procédant comme suit :

1. Dans le menu de gauche du portail Azure, cliquez sur **Groupes de ressources**, puis sur le nom de la ressource que vous avez créée.  

2. Sur la page de votre groupe de ressources, sélectionnez **Supprimer**, saisissez le nom de la ressource à supprimer dans la zone de texte, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à implémenter un désérialiseur .NET personnalisé pour la sérialisation des entrées du tampon de protocole. Pour en savoir plus sur la création de désérialiseurs personnalisés, passez à l’article suivant :

> [!div class="nextstepaction"]
> [Créer différents désérialiseurs .NET pour les travaux Azure Stream Analytics](custom-deserializer-examples.md)
