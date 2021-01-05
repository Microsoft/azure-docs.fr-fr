---
title: 'Tutoriel : Détection d’anomalie avec Cognitive Services'
description: Tutoriel permettant de découvrir comment exploiter Cognitive Services pour la détection d’anomalie dans Synapse
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 4052d6a0773aa27e0a378ee04975c7946f1ffbfe
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463919"
---
# <a name="tutorial-anomaly-detection-with-cognitive-services-preview"></a>Tutoriel : Détection d’anomalie avec Cognitive Services (Préversion)

Dans ce tutoriel, vous allez apprendre à enrichir facilement vos données dans Azure Synapse avec [Cognitive Services](https://go.microsoft.com/fwlink/?linkid=2147492). Nous allons utiliser le [Détecteur d’anomalies](https://go.microsoft.com/fwlink/?linkid=2147493) pour détecter des anomalies. Dans Azure Synapse, un utilisateur peut simplement sélectionner une table à enrichir pour la détection d’anomalie.

Ce didacticiel contient les sections suivantes :

> [!div class="checklist"]
> - Étapes à effectuer pour obtenir un jeu de données de table Spark contenant des données de série chronologique.
> - Utilisation d’une expérience d’assistant dans Azure Synapse pour enrichir les données à l’aide du Détecteur d’anomalies de Cognitive Services.

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit avant de commencer](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

- [Espace de travail Azure Synapse Analytics](../get-started-create-workspace.md) avec un compte de stockage ADLS Gen2 configuré comme stockage par défaut. Vous devez être le **contributeur de données Blob du stockage** du système de fichiers ADLS Gen2 que vous utilisez.
- Pool Spark dans votre espace de travail Azure Synapse Analytics. Pour plus d’informations, consultez [Créer un pool Spark dans Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Avant de pouvoir utiliser ce tutoriel, vous devez également effectuer les étapes de préconfiguration qu’il décrit. [Configurez Cognitive Services dans Azure Synapse](tutorial-configure-cognitive-services-synapse.md).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/)

## <a name="create-a-spark-table"></a>Créer une table Spark

Vous aurez besoin d’une table Spark pour ce tutoriel.

1. Téléchargez le fichier de notebook suivant contenant du code pour générer une table Spark : [prepare_anomaly_detector_data.ipynb](https://go.microsoft.com/fwlink/?linkid=2149577)

1. Chargez le fichier dans votre espace de travail Azure Synapse.
![Charger le notebook](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00a.png)

1. Ouvrez le fichier de notebook, puis choisissez d’exécuter toutes les cellules (**Exécuter tout**).
![Créer une table Spark](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00b.png)

1. Une table Spark nommée **anomaly_detector_testing_data** doit maintenant apparaître dans la base de données Spark par défaut.

## <a name="launch-cognitive-services-wizard"></a>Lancer l’Assistant Cognitive Services

1. Cliquez avec le bouton droit sur la table Spark créée à l’étape précédente. Sélectionnez « Machine Learning -> Enrichir avec un modèle existant » pour ouvrir l’Assistant.
![Lancer l’Assistant de scoring](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00g.png)

2. Un panneau de configuration s’affiche et vous êtes invité à sélectionner un modèle Cognitive Services. Sélectionnez le Détecteur d’anomalies.

![Lancer l’Assistant de scoring - Étape 1](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00c.png)

## <a name="provide-authentication-details"></a>Fournir des informations d’authentification

Pour vous authentifier auprès de Cognitive Services, vous devez référencer le secret à utiliser dans votre coffre de clés. Les entrées ci-dessous dépendent des [étapes préalables](tutorial-configure-cognitive-services-synapse.md) que vous devez avoir effectuées avant cette étape.

- **Abonnement Azure** : Sélectionnez l’abonnement auquel appartient votre coffre de clés.
- **Compte Cognitive Services** : il s’agit de la ressource Analyse de texte à laquelle vous allez vous connecter.
- **Service lié Azure Key Vault** : dans le cadre des étapes préalables, vous avez créé un service lié à votre ressource Analyse de texte. Sélectionnez-le ici.
- **Nom du secret** : il s’agit du nom du secret dans votre coffre de clés contenant la clé qui permet de vous authentifier auprès de votre ressource Cognitive Services.

![Fournir les détails Azure Key Vault](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00d.png)

## <a name="configure-anomaly-detection"></a>Configurer la détection d’anomalie

Vous devez ensuite configurer la détection d’anomalie. Fournissez les détails suivants :

- Granularité : fréquence d’échantillonnage de vos données. Par exemple, si vos données ont une valeur pour chaque minute, votre granularité est Toutes les minutes. Choisissez **mensuelle** 

- Horodatage : colonne représentant l’heure de la série. Choisissez la colonne **horodatage**

- Valeur de série chronologique : colonne représentant la valeur de la série au moment spécifié par la colonne Horodatage. Choisissez la colonne **valeur**

- Colonne de regroupement : colonne qui regroupe les séries. Autrement dit, toutes les lignes qui ont la même valeur dans cette colonne doivent former une série chronologique. Choisissez la colonne **groupe**

Une fois que vous avez terminé, sélectionnez **Ouvrir le notebook**. Cette opération génère automatiquement un notebook avec le code PySpark qui effectue la détection d’anomalie à l’aide d’Azure Cognitive Services.

![Configurer le détecteur d’ anomalies](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00e.png)

## <a name="open-notebook-and-run"></a>Ouvrir un notebook et effectuer l’exécution

Le notebook que vous venez d’ouvrir utilise la [bibliothèque mmlspark](https://github.com/Azure/mmlspark) pour se connecter à Cognitive Services.

Les détails Azure Key Vault que vous avez fournis vous permettent de référencer vos secrets de manière sécurisée à partir de cette expérience sans les divulguer.

Vous pouvez maintenant exécuter toutes les cellules (**Exécuter tout**) pour effectuer une détection d’anomalie. Découvrez-en plus sur [Cognitive Services - Détecteur d’anomalies](https://go.microsoft.com/fwlink/?linkid=2147493).

![Exécuter la détection d’anomalie](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00f.png)

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Analyse des sentiments avec Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutoriel : Scoring de modèle Machine Learning dans des pools SQL dédiés Azure Synapse](tutorial-sql-pool-model-scoring-wizard.md)
- [Fonctionnalités de Machine Learning dans Azure Synapse Analytics](what-is-machine-learning.md)