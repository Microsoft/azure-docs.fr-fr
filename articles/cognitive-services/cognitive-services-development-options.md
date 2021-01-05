---
title: Options de développement Azure Cognitive Services
description: Découvrez comment utiliser Azure Cognitive Services avec différentes options de développement et de déploiement, telles que les bibliothèques de client, les API REST, Logic Apps, Power Automate, Azure Functions, Azure App Service, Azure Databricks et bien plus encore.
services: cognitive-services
manager: nitinme
author: erhopf
ms.author: erhopf
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: a952bfa45515b9d35549d03d18fd94103679344d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349467"
---
# <a name="cognitive-services-development-options"></a>Options de développement Cognitive Services

Ce document fournit une vue d’ensemble des options de développement et de déploiement pour vous aider à prendre en main Azure Cognitive Services.

Azure Cognitive Services est une famille de services informatiques d’intelligence artificielle qui permettent aux développeurs d’intégrer une intelligence dans leurs applications et produits sans avoir une connaissance approfondie de l’apprentissage automatique. Grâce à Cognitive Services, vous avez accès à des capacités ou à des modèles d’IA qui sont générés, formés et mis à jour par Microsoft, prêts à être utilisés dans vos applications. Dans de nombreux cas, vous avez également la possibilité de personnaliser les modèles en vue de répondre aux besoins de votre entreprise. 

Cognitive Services est organisée en quatre catégories : Décision, Langage, Parole et Vision. En général, vous accédez à ces services par l’intermédiaire des API REST, des bibliothèques de client et des outils personnalisés (tels que les interfaces de ligne de commande) fournis par Microsoft. Toutefois, ce n’est qu’une voie vers la réussite. Grâce à Azure, vous avez également accès à plusieurs options de développement, notamment :

* des outils d’automatisation et d’intégration tels que Logic Apps et Power Automate ;
* des options de déploiement telles qu’Azure Functions et App Service ; 
* des conteneurs Docker Cognitive Services pour un accès sécurisé ;
* des outils comme Apache Spark, Azure Databricks, Azure Synapse Analytics et Azure Kubernetes Service pour les scénarios de Big Data. 

Avant de vous lancer, il est important de savoir que Cognitive Services est principalement utilisé pour deux tâches distinctes. En fonction de la tâche que vous souhaitez effectuer, vous avez le choix entre différentes options de développement et de déploiement. 

* [Options de développement pour la prédiction et l’analyse](#development-options-for-prediction-and-analysis)
* [Outils pour personnaliser et configurer des modèles](#tools-to-customize-and-configure-models)

## <a name="development-options-for-prediction-and-analysis"></a>Options de développement pour la prédiction et l’analyse 

Les outils que vous utiliserez pour personnaliser et configurer les modèles sont différents de ceux que vous utiliserez pour appeler Cognitive Services. D’emblée, la plupart des services Cognitive Services vous permettent d’envoyer des données et de recevoir des insights sans aucune personnalisation. Par exemple : 

* Vous pouvez envoyer une image au service Vision par ordinateur pour détecter des mots et des expressions ou pour compter le nombre de personnes dans le cadre.
* Vous pouvez envoyer un fichier audio au service Speech et recevoir des transcriptions et traduire la parole en texte en même temps.
* Vous pouvez envoyer un fichier PDF au service Form Recognizer et détecter des tables, des cellules et du texte à l’intérieur de ces cellules, et obtenir une sortie JSON avec des coordonnées et des détails.

Azure propose un large éventail d’outils conçus pour différents types d’utilisateurs, dont beaucoup peuvent être utilisés avec Cognitive Services. Les outils basés sur le concepteur sont les plus faciles à utiliser et sont rapides à configurer et automatiser, mais ils peuvent présenter des limitations en matière de personnalisation. Nos API REST et nos bibliothèques de client offrent aux utilisateurs davantage de contrôle et de flexibilité, mais requièrent plus d’efforts, de temps et d’expertise pour créer une solution. Si vous utilisez des API REST et des bibliothèques de client, vous devez avoir l’habitude d’utiliser des langages de programmation modernes comme C#, Java, Python, JavaScript ou un autre langage de programmation populaire. 

Examinons les différentes façons dont vous pouvez travailler avec Cognitive Services.

### <a name="client-libraries-and-rest-apis"></a>Bibliothèques de client et API REST

Les bibliothèques de client et les API REST Cognitive Services vous fournissent un accès direct à votre service. Ces outils fournissent un accès programmatique à Cognitive Services et à ses modèles de référence et, dans de nombreux cas, vous permettent de personnaliser par programmation vos modèles et solutions. 

* **Utilisateur(s) cible(s)**  : Développeurs et scientifiques des données
* **Avantages** : Offre la plus grande flexibilité pour appeler les services à partir de n’importe quel langage et environnement 
* **Interface utilisateur** : s.o. (code uniquement)
* **Abonnement(s)**  : Compte Azure + ressources Cognitive Services

Si vous souhaitez en savoir plus sur les bibliothèques de client et les API REST disponibles, consultez notre [présentation de Cognitive Services](index.yml) pour choisir un service et prendre en main l’un de nos démarrages rapides pour la vision, la décision, le langage et la parole.

### <a name="cognitive-services-for-big-data"></a>Cognitive Services pour le Big Data

Avec Cognitive Services pour le Big Data, vous pouvez incorporer des modèles toujours plus intelligents et plus performants dans les calculs Apache Spark&trade; et SQL. Ces outils libèrent les développeurs des tâches réseau de bas niveau, qui peuvent ainsi se concentrer sur la création d’applications intelligentes et distribuées. Cognitive Services pour le Big Data prend en charge les plateformes et connecteurs suivants : Azure Databricks, Azure Synapse, Azure Kubernetes Service et les connecteurs de données.

* **Utilisateur(s) cible(s)**  : Scientifiques des données et ingénieurs des données
* **Avantages** : Azure Cognitive Services pour le Big Data permet aux utilisateurs de canaliser des téraoctets de données à travers Cognitive Services à l’aide d’Apache Spark&trade;. Il est facile de créer des applications intelligentes à grande échelle avec n’importe quel magasin de données
* **Interface utilisateur** : s.o. (code uniquement)
* **Abonnement(s)**  : Compte Azure + ressources Cognitive Services

Si vous souhaitez en savoir plus sur le Big Data pour Cognitive Services, il est préférable de commencer par la [vue d’ensemble](./big-data/cognitive-services-for-big-data.md). Si vous êtes prêt à commencer à créer, essayez nos exemples [Python](./big-data/samples-python.md) ou [Scala](./big-data/samples-scala.md).

### <a name="azure-functions-and-azure-service-web-jobs"></a>Azure Functions et tâches web Azure App Service

[Azure Functions](../azure-functions/index.yml) et [les tâches web Azure App Service](../app-service/index.yml) fournissent tous deux des services d’intégration de code conçus pour les développeurs et sont basés sur [Azure App Service](../app-service/index.yml). Ces produits fournissent une infrastructure serverless pour l’écriture de code. Dans ce code, vous pouvez effectuer des appels à nos services à l’aide des bibliothèques de client et des API REST. 

* **Utilisateur(s) cible(s)**  : Développeurs et scientifiques des données
* **Avantages** : Service de calcul serverless qui vous permet d’exécuter du code déclenché par un événement 
* **Interface utilisateur** : Oui
* **Abonnement(s)**  : Compte Azure + ressource Cognitive Services + abonnement Azure Functions

### <a name="azure-logic-apps"></a>Azure Logic Apps 

[Azure Logic Apps](../logic-apps/index.yml) partage le même concepteur de workflow et les mêmes connecteurs que Power Automate, mais propose un contrôle plus avancé, notamment des intégrations à Visual Studio et à DevOps. Power Automate facilite l’intégration à vos ressources Cognitive Services grâce à des connecteurs spécifiques aux services qui fournissent un proxy ou un wrapper autour des API. Ce sont les mêmes connecteurs que ceux disponibles dans Power Automate. 

* **Utilisateur(s) cible(s)**  : Développeurs, intégrateurs, professionnels de l’informatique, DevOps
* **Avantages** : Modèle de développement orienté concepteur (déclaratif) fournissant des options avancées et une intégration dans une solution à faible code
* **Interface utilisateur** : Oui
* **Abonnement(s)**  : Compte Azure + ressource Cognitive Services + déploiement de Logic Apps

### <a name="power-automate"></a>Power Automate 

Power Automate est un service de [Microsoft Power Platform](/power-platform/) qui vous permet de créer des workflows automatisés entre les applications et les services sans écrire de code. Nous proposons plusieurs connecteurs pour faciliter l’interaction avec votre ressource Cognitive Services dans une solution Power Automate. Power Automate repose sur Logic Apps. 

* **Utilisateur(s) cible(s)**  : Utilisateurs professionnels (analystes) et administrateurs SharePoint
* **Avantages** : Automatisez les tâches manuelles répétitives en enregistrant simplement les clics de souris, les frappes de touche et les opérations de copier-coller sur votre bureau
* **Outils d’interface utilisateur** : Oui ; UI uniquement
* **Abonnement(s)**  : Compte Azure + ressource Cognitive Services + abonnement Power Automate + abonnement Office 365

### <a name="ai-builder"></a>AI Builder 

[Le générateur IA](/ai-builder/overview) est une capacité de Microsoft Power Platform que vous pouvez utiliser pour améliorer les performances de l’entreprise en automatisant les processus et en prédisant les résultats. Le générateur IA apporte la puissance de l’intelligence artificielle à vos solutions grâce à une expérience de pointer-cliquer. De nombreux services cognitifs tels que Form Recognizer, Analyse de texte et Vision par ordinateur ont été directement intégrés ici, et vous n’avez pas besoin de créer vos propres services cognitifs. 

* **Utilisateur(s) cible(s)**  : Utilisateurs professionnels (analystes) et administrateurs SharePoint
* **Avantages** : Solution clé en main qui apporte la puissance de l’intelligence artificielle grâce à une expérience de pointer-cliquer. Aucune qualification en codage ou en science des données n’est requise
* **Outils d’interface utilisateur** : Oui ; UI uniquement
* **Abonnement(s)**  : AI Builder

### <a name="continuous-integration-and-deployment"></a>Intégration et déploiement continus

Vous pouvez utiliser Azure DevOps et GitHub Actions pour gérer vos déploiements. Dans la [section ci-dessous](#continuous-integration-and-delivery-with-devops-and-github-actions), nous présentons deux exemples d’intégrations CI/CD pour effectuer l’apprentissage et déployer des modèles personnalisés pour Speech et le service Language Understanding (LUIS). 

* **Utilisateur(s) cible(s)**  : Développeurs, scientifiques des données et ingénieurs des données
* **Avantages** : Vous permet d’ajuster, de mettre à jour et de déployer continuellement des applications et des modèles par programmation. Il existe un avantage significatif quand vous utilisez régulièrement vos données pour améliorer et mettre à jour des modèles pour les catégories Parole, Vision, Langage et Décision 
* **Outils d’interface utilisateur** : s.o. (code uniquement) 
* **Abonnement(s)**  : Compte Azure + ressource Cognitive Services + compte GitHub

## <a name="tools-to-customize-and-configure-models"></a>Outils pour personnaliser et configurer des modèles

Au fur et à mesure que vous progressez dans la création d’une application ou d’un workflow avec Cognitive Services, il se peut que vous deviez personnaliser le modèle pour obtenir les performances souhaitées. La plupart de nos services vous permettent de compléter des modèles prédéfinis pour répondre à vos besoins professionnels spécifiques. Pour tous nos services personnalisables, nous fournissons à la fois une expérience pilotée par l’interface utilisateur pour parcourir le processus et des API pour la formation pilotée par le code. Par exemple :

* Vous souhaitez effectuer l’apprentissage d’un modèle Custom Speech pour qu’il reconnaisse correctement les termes médicaux avec un taux d’erreur sur les mots (WER) inférieur à 3 %.
* Vous souhaitez créer un classifieur d’images avec Custom Vision qui peut faire la différence entre les conifères et les arbres à feuillage caduc.
* Vous souhaitez créer une voix neurale personnalisée avec vos données vocales personnelles pour une meilleure expérience client automatisée.

Les outils que vous utiliserez pour effectuer l’apprentissage des modèles et les configurer sont différents de ceux que vous utiliserez pour appeler Cognitive Services. Dans de nombreux cas, les services cognitifs qui prennent en charge la personnalisation proposent des portails et des outils d’interface utilisateur conçus pour vous aider à effectuer l’apprentissage des modèles, les évaluer et les déployer. Examinons rapidement quelques options :<br><br>

| Pilier | Service | Personnalisation de l’interface utilisateur | Démarrage rapide |
|--------|---------|------------------|------------|
| Vision | Vision personnalisée | https://www.customvision.ai/ | [Démarrage rapide](./custom-vision-service/quickstarts/image-classification.md?pivots=programming-language-csharp) | 
| Vision | Form Recognizer | Outil d’étiquetage des exemples | [Démarrage rapide](./form-recognizer/quickstarts/label-tool.md?tabs=v2-0) |
| Décision | Content Moderator | https://contentmoderator.cognitive.microsoft.com/dashboard | [Démarrage rapide](./content-moderator/review-tool-user-guide/human-in-the-loop.md) |
| Décision | Metrics Advisor | https://metricsadvisor.azurewebsites.net/  | [Démarrage rapide](./metrics-advisor/quickstarts/web-portal.md) |
| Décision | Personalizer | L’interface utilisateur est disponible dans le portail Azure sous votre ressource Personalizer. | [Démarrage rapide](./personalizer/quickstart-personalizer-sdk.md) |
| Langage | Language Understanding (LUIS) | https://www.luis.ai/ | |
| Langage | QnA Maker | https://www.qnamaker.ai/ | [Démarrage rapide](./qnamaker/quickstarts/create-publish-knowledge-base.md) |
| Langage | Translator/Custom Translator | https://portal.customtranslator.azure.ai/ | [Démarrage rapide](./translator/custom-translator/quickstart-build-deploy-custom-model.md) |
| Voix | Commandes personnalisées | https://speech.microsoft.com/ | [Démarrage rapide](./speech-service/custom-commands.md) |
| Voix | Custom Speech | https://speech.microsoft.com/ | [Démarrage rapide](./speech-service/custom-speech-overview.md) |
| Voix | Custom Voice | https://speech.microsoft.com/ | [Démarrage rapide](./speech-service/how-to-custom-voice.md) |  

### <a name="continuous-integration-and-delivery-with-devops-and-github-actions"></a>Intégration et livraison continues avec un DevOps et GitHub Actions

Language Understanding et le service Speech offrent des solutions d’intégration continue et de déploiement continu qui sont alimentées par Azure DevOps et GitHub Actions. Ces outils sont utilisés pour la formation automatisée, les tests et la gestion des mises en production des modèles personnalisés. 

* [CI/CD pour Custom Speech](./speech-service/how-to-custom-speech-continuous-integration-continuous-deployment.md)
* [CI/CD pour LUIS](./luis/luis-concept-devops-automation.md)

## <a name="on-prem-containers"></a>Conteneurs locaux 

La plupart des services cognitifs peuvent être déployés dans des conteneurs pour un accès et un usage locaux. Grâce à ces conteneurs, Cognitive Services se trouve au plus près de vos données du point de vue de la conformité, de la sécurité et de l’exploitation. Pour obtenir la liste complète des conteneurs Cognitive Services, consultez [Conteneurs locaux pour Cognitive Services](./cognitive-services-container-support.md).

## <a name="next-steps"></a>Étapes suivantes
<!--
* Learn more about low code development options for Cognitive Services -->
* [Créer une ressource Cognitive Services et démarrer la génération](./cognitive-services-apis-create-account.md?tabs=multiservice%252clinux)