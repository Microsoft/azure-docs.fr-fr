---
title: 'Tutoriel : Prérequis de Cognitive Services dans Azure Synapse'
description: Tutoriel permettant de découvrir comment configurer les prérequis de l’utilisation de Cognitive Services dans Azure Synapse
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: cf76a3d09da83693791873feb9e6ab03c432e47f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463883"
---
# <a name="tutorial-pre-requisites-for-using-cognitive-services-in-azure-synapse"></a>Tutoriel : Prérequis de l’utilisation de Cognitive Services dans Azure Synapse

Dans ce tutoriel, vous allez apprendre à configurer les prérequis de l’exploitation sécurisée de Cognitive Services dans Azure Synapse.

Ce didacticiel contient les sections suivantes :
> [!div class="checklist"]
> - Créer une ressource Cognitive Services. Par exemple, l’Analyse de texte ou le Détecteur d’anomalies.
> - Stocker la clé d’authentification auprès des ressources Cognitive Services sous la forme de secrets dans Azure Key Vault et configurer l’accès pour l’espace de travail Azure Synapse.
> - Créer le service lié Azure Key Vault dans votre espace de travail Azure Synapse Analytics.

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit avant de commencer](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

- [Espace de travail Azure Synapse Analytics](../get-started-create-workspace.md) avec un compte de stockage ADLS Gen2 configuré comme stockage par défaut. Vous devez être le **contributeur de données Blob du stockage** du système de fichiers ADLS Gen2 que vous utilisez.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/)

## <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

[Azure Cognitive Services](https://go.microsoft.com/fwlink/?linkid=2147492) inclut de nombreux types de services différents. Vous trouverez ci-dessous quelques exemples qui sont utilisés dans les tutoriels Synapse.

### <a name="create-an-anomaly-detector-resource"></a>Créer une ressource Détecteur d’anomalies
Créez un [Détecteur d’anomalies](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) dans le portail Azure.

![Créer un détecteur d’anomalies](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00a.png)

### <a name="create-a-text-analytics-resource"></a>Créer une ressource Analyse de texte
Créez une ressource [Analyse de texte](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) dans le portail Azure.

![Créer l’Analyse de texte](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00b.png)

## <a name="create-key-vault-and-configure-secrets-and-access"></a>Créer un coffre de clés et configurer des secrets et l’accès

1. Créez un [coffre de clés](https://ms.portal.azure.com/#create/Microsoft.KeyVault) dans le portail Azure.
2. Accédez à **Coffre de clés -> Stratégies d’accès**, puis accordez à l’[Identité de service managée (MSI) de l’espace de travail Azure Synapse](https://docs.microsoft.com/azure/synapse-analytics/security/synapse-workspace-managed-identity) des autorisations pour lire des secrets à partir d’Azure Key Vault.

>Vérifiez que les changements de stratégie sont enregistrés. Il est facile d’oublier cette étape.

![Ajouter une stratégie d’accès](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00c.png)

3. Accédez à votre ressource Cognitive Services, par exemple **Détecteur d’anomalies -> Clés et point de terminaison**, puis copiez l’une des deux clés dans le Presse-papiers.

4. Accédez à **Coffre de clés -> Secret** pour créer un secret. Spécifiez le nom du secret, puis collez la clé de l’étape précédente dans le champ « Valeur ». Pour finir, cliquez sur **Créer**.

![Créer un secret](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00d.png)

> Veillez à mémoriser ou à noter le nom de ce secret. Vous l’utiliserez ultérieurement quand vous vous connecterez à Cognitive Services à partir d’Azure Synapse Studio.

## <a name="create-azure-keyvault-linked-service-in-azure-synapse"></a>Créer un service lié Azure Key Vault dans Azure Synapse

1. Ouvrez votre espace de travail dans Azure Synapse Studio. Accédez à **Gérer -> Services liés**. Créez un service lié « Azure Key Vault » pointant vers le coffre de clés que nous venons de créer. Vérifiez ensuite la connexion en cliquant sur le bouton « Tester la connexion », puis en vérifiant si elle s’affiche en vert. Si tout fonctionne bien, cliquez d’abord sur « Créer », puis sur « Publier tout » pour enregistrer le changement apporté.
![Service lié](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00e.png)

Vous êtes maintenant prêt à passer à l’un des tutoriels concernant l’utilisation de l’expérience Azure Cognitive Services dans Azure Synapse Studio.

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Analyse des sentiments avec Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutoriel : Détection d’anomalie avec Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutoriel : Scoring de modèle Machine Learning dans des pools SQL dédiés Azure Synapse](tutorial-sql-pool-model-scoring-wizard.md).
- [Fonctionnalités de Machine Learning dans Azure Synapse Analytics](what-is-machine-learning.md)