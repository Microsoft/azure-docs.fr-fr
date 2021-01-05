---
title: 'Tutoriel : Analyse des sentiments avec Cognitive Services'
description: Tutoriel permettant de découvrir comment tirer parti de Cognitive Services pour analyser des sentiments dans Synapse
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 1b407cbee5218149f794ab125ac058e32b422558
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464365"
---
# <a name="tutorial-sentiment-analysis-with-cognitive-services-preview"></a>Tutoriel : Analyse des sentiments avec Cognitive Services (Préversion)

Dans ce tutoriel, vous allez apprendre à enrichir facilement vos données dans Azure Synapse avec [Cognitive Services](https://go.microsoft.com/fwlink/?linkid=2147492). Nous allons utiliser les fonctionnalités de l’[Analyse de texte](https://docs.microsoft.com/azure/cognitive-services/text-analytics/) pour effectuer une analyse des sentiments. Dans Azure Synapse, un utilisateur peut simplement sélectionner une table contenant une colonne de texte à enrichir avec des sentiments. Ces sentiments peuvent être positifs, négatifs, mixtes ou neutres, et une probabilité est également retournée.

Ce didacticiel contient les sections suivantes :

> [!div class="checklist"]
> - Étapes à effectuer pour obtenir un jeu de données de table Spark contenant une colonne de texte pour l’analyse des sentiments.
> - Utilisation d’une expérience d’assistant dans Azure Synapse pour enrichir les données à l’aide de l’Analyse de texte de Cognitive Services.

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit avant de commencer](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

- [Espace de travail Azure Synapse Analytics](../get-started-create-workspace.md) avec un compte de stockage ADLS Gen2 configuré comme stockage par défaut. Vous devez être le **contributeur de données Blob du stockage** du système de fichiers ADLS Gen2 que vous utilisez.
- Pool Spark dans votre espace de travail Azure Synapse Analytics. Pour plus d’informations, consultez [Créer un pool Spark dans Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Avant de pouvoir utiliser ce tutoriel, vous devez également effectuer les étapes de préconfiguration qu’il décrit. [Configurez Cognitive Services dans Azure Synapse](tutorial-configure-cognitive-services-synapse.md).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/)

## <a name="create-a-spark-table"></a>Créer une table Spark

Vous aurez besoin d’une table Spark pour ce tutoriel.

1. Téléchargez le fichier CSV suivant contenant un jeu de données pour l’analyse de texte : [FabrikamComments.csv](https://github.com/Kaiqb/KaiqbRepo0731190208/blob/master/CognitiveServices/TextAnalytics/FabrikamComments.csv)

1. Chargez le fichier dans votre compte de stockage ADLSGen2 Azure Synapse.
![Charger des données](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00a.png)

1. Créez une table Spark à partir du fichier .csv en cliquant avec le bouton droit sur le fichier, puis en sélectionnant **Nouveau notebook -> Créer une table Spark**.
![Créer une table Spark](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00b.png)

1. Nommez la table dans la cellule de code, puis exécutez le notebook sur un pool Spark. N’oubliez pas de définir « Header = true ».
![Exécuter un notebook](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00c.png)

```python
%%pyspark
df = spark.read.load('abfss://default@azuresynapsesa.dfs.core.windows.net/data/FabrikamComments.csv', format='csv'
## If header exists uncomment line below
, header=True
)
df.write.mode("overwrite").saveAsTable("default.YourTableName")
```

## <a name="launch-cognitive-services-wizard"></a>Lancer l’Assistant Cognitive Services

1. Cliquez avec le bouton droit sur la table Spark créée à l’étape précédente. Sélectionnez « Machine Learning -> Enrichir avec un modèle existant » pour ouvrir l’Assistant.
![Lancer l’Assistant de scoring](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00d.png)

2. Un panneau de configuration s’affiche et vous êtes invité à sélectionner un modèle Cognitive Services. Sélectionnez Analyse de texte - Analyse des sentiments.

![Lancer l’Assistant de scoring - Étape 1](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00e.png)

## <a name="provide-authentication-details"></a>Fournir des informations d’authentification

Pour vous authentifier auprès de Cognitive Services, vous devez référencer le secret à utiliser dans votre coffre de clés. Les entrées ci-dessous dépendent des [étapes préalables](tutorial-configure-cognitive-services-synapse.md) que vous devez avoir effectuées avant cette étape.

- **Abonnement Azure** : Sélectionnez l’abonnement auquel appartient votre coffre de clés.
- **Compte Cognitive Services** : il s’agit de la ressource Analyse de texte à laquelle vous allez vous connecter.
- **Service lié Azure Key Vault** : dans le cadre des étapes préalables, vous avez créé un service lié à votre ressource Analyse de texte. Sélectionnez-le ici.
- **Nom du secret** : il s’agit du nom du secret dans votre coffre de clés contenant la clé qui permet de vous authentifier auprès de votre ressource Cognitive Services.

![Fournir les détails Azure Key Vault](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00f.png)

## <a name="configure-sentiment-analysis"></a>Configurer l’analyse des sentiments

Vous devez ensuite configurer l’analyse des sentiments. Sélectionnez les informations suivantes :
- **Langue** : Sélectionnez la langue du texte sur lequel vous souhaitez effectuer une analyse des sentiments. Sélectionnez **EN**.
- **Colonne de texte** : il s’agit de la colonne de texte de votre jeu de données que vous voulez analyser pour déterminer le sentiment. Sélectionner la colonne de texte **comment**.

Quand vous avez terminé, cliquez sur **Ouvrir le notebook**. Cette opération génère automatiquement un notebook avec le code PySpark qui effectue l’analyse des sentiments avec Azure Cognitive Services.

![Configurer l’analyse des sentiments](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00g.png)

## <a name="open-notebook-and-run"></a>Ouvrir un notebook et effectuer l’exécution

Le notebook que vous venez d’ouvrir utilise la [bibliothèque mmlspark](https://github.com/Azure/mmlspark) pour se connecter à Cognitive Services.

Les détails Azure Key Vault que vous avez fournis vous permettent de référencer vos secrets de manière sécurisée à partir de cette expérience sans les divulguer.

Vous pouvez maintenant exécuter toutes les cellules (**Exécuter tout**) pour enrichir vos données avec des sentiments. Les sentiments sont retournés sous la forme Positif/Négatif/Neutre/Mixte, et vous obtenez également des probabilités par sentiment. Découvrez-en plus sur [Cognitive Services - Analyse des sentiments](https://go.microsoft.com/fwlink/?linkid=2147792).

![Exécuter une analyse des sentiments](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00h.png)

## <a name="next-steps"></a>Étapes suivantes
- [Tutoriel : Détection d’anomalie avec Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutoriel : Scoring de modèle Machine Learning dans des pools SQL dédiés Azure Synapse](tutorial-sql-pool-model-scoring-wizard.md)
- [Fonctionnalités de Machine Learning dans Azure Synapse Analytics](what-is-machine-learning.md)