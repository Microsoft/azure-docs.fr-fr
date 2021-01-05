---
title: 'ML Studio (classique) : Application Lifecycle Management – Azure'
description: Appliquer les meilleures pratiques d’Application Lifecycle Management dans Azure Machine Learning Studio (classique)
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.date: 10/27/2016
ms.openlocfilehash: f5c9e27e894541d71986fe929cbc5d6fde31bc18
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308802"
---
# <a name="application-lifecycle-management-in-azure-machine-learning-studio-classic"></a>Application Lifecycle Management dans Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Azure Machine Learning Studio (classique) est un outil permettant de développer les expériences d'apprentissage automatique qui seront utilisées sur la plateforme cloud Azure. Cet outil est similaire au service cloud évolutif et à l’environnement IDE de Visual Studio, tous deux regroupés au sein d’une même plate-forme. Vous pouvez incorporer dans Azure Machine Learning Studio (classique) des pratiques de gestion du cycle de vie des applications (ALM, Application Lifecycle Management) standard, notamment le contrôle de version de plusieurs ressources, ainsi que l’exécution et le déploiement automatisés. Cet article traite de certaines de ces options et approches.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="versioning-experiment"></a>Contrôle de version d’expérience
Il existe deux méthodes recommandées pour contrôler la version de vos expériences. Vous pouvez soit vous appuyer sur l’historique d’exécution intégrée, soit exporter l’expérience au format JSON et la gérer en externe. Chaque approche présente des avantages et des inconvénients.

### <a name="experiment-snapshots-using-run-history"></a>Instantanés d’expérience à l’aide de l’historique d’exécution
Selon le Modèle d’exécution de l’expérience Azure Machine Learning Studio (classique), un instantané immuable de l’expérience est envoyé au planificateur de travaux à chaque fois que vous cliquez sur le bouton **Exécuter** dans l’éditeur d’expérience. Pour afficher cette liste d’instantanés, cliquez sur le bouton **Historique des exécutions** dans la barre de commandes de la vue de l’éditeur d’expérience.

![Bouton Historique d’exécution](./media/version-control/runhistory.png)

Vous pouvez ensuite ouvrir l’instantané en mode verrouillé en cliquant sur le nom de l’expérience au moment où l’expérience a été envoyée pour exécution et où l’instantané a été créé. Notez que seul le premier élément de la liste, qui représente l’expérience actuelle, est modifiable. Notez également que chaque instantané peut présenter plusieurs états différents, notamment Terminé (Exécution partielle), Échec, Échec (exécution partielle) ou Brouillon.

![Liste Historique d’exécution](./media/version-control/runhistorylist.png)

Une fois ouvert, vous pouvez enregistrer l’instantané de l’expérience en tant que nouvelle expérience et le modifier. Si l’instantané de votre expérience contient des ressources telles que des modèles formés, des transformations, des jeux de données présentant des versions mises à jour, l’instantané conserve les références à la version d’origine lors de la création de l’instantané. Si vous enregistrez l’instantané verrouillé en tant que nouvelle expérience, Azure Machine Learning Studio (classique) détecte l’existence d’une version plus récente de ces ressources et les met automatiquement à jour dans la nouvelle expérience.

Notez également que si vous supprimez l’expérience, tous les instantanés de cette expérience le sont également.

### <a name="exportimport-experiment-in-json-format"></a>Importation/exportation d’expérience au format JSON
Les instantanés de l’historique d’exécution conservent une version immuable de l’expérience dans Azure Machine Learning Studio (classique) chaque fois qu’elle est soumise à des fins d’exécution. Vous pouvez également enregistrer une copie locale de l’expérience et l’archiver dans votre système de contrôle de code source préféré, par exemple Team Foundation Server, puis recréer une expérience à partir de ce fichier local. Pour ce faire, vous pouvez utiliser les applets de commande [*Export-AmlExperimentGraph*](https://github.com/hning86/azuremlps#export-amlexperimentgraph) et [*Import-AmlExperimentGraph*](https://github.com/hning86/azuremlps#import-amlexperimentgraph)[d’Azure Machine Learning Studio](https://aka.ms/amlps).

Le fichier JSON est une représentation textuelle du graphique de l’expérience, qui peut inclure une référence aux ressources dans l’espace de travail, comme le jeu de données ou un modèle formé. Il ne contient aucune version sérialisée de la ressource. Si vous tentez d’importer le document JSON dans l’espace de travail, les ressources référencées doivent déjà exister et présenter les ID de ressource référencés dans l’expérience ; dans le cas contraire, vous ne pouvez pas accéder à l’expérience importée.

## <a name="versioning-trained-model"></a>Contrôle de version du modèle formé
Un Modèle formé dans Azure Machine Learning Studio (classique) est sérialisé sous la forme d’un fichier iLearner (`.iLearner`) et stocké dans le compte de stockage Blob Azure associé à l’espace de travail. Pour obtenir une copie du fichier iLearner, vous pouvez par exemple utiliser l’API de reformation. [Cet article](./retrain-machine-learning-model.md) décrit le fonctionnement de l’API de reformation. Voici la procédure générale :

1. Configurez votre expérience de formation.
2. Ajoutez le port de sortie du service web au module Former le modèle ou au module qui génère le modèle formé, tel que Tune Model Hyperparameter ou Create R Model.
3. Exécutez votre expérience de formation et déployez-la en tant que service web de formation de modèle.
4. Appelez le point de terminaison BES du service web de formation et spécifiez le nom du fichier iLearner souhaité, ainsi que l’emplacement du compte de Stockage Blob Azure dans lequel il sera stocké.
5. Récupérez le fichier iLearner produit une fois l’appel BES terminé.

La cmdlet PowerShell [*Download-AmlExperimentNodeOutput*](https://github.com/hning86/azuremlps#download-amlexperimentnodeoutput) constitue une autre méthode pour récupérer le fichier iLearner. Cela peut être plus facile si vous souhaitez simplement obtenir une copie du fichier iLearner sans avoir besoin de reformer le modèle par programmation.

Une fois que le fichier iLearner contenant le modèle formé est prêt, vous pouvez utiliser votre propre stratégie de contrôle de version. La stratégie à appliquer peut simplement consister à associer un préfixe ou un suffixe dans le cadre d’une convention d’affectation de noms, et à laisser le fichier iLearner dans le Stockage Blob, ou à copier/importer ce fichier dans votre système de contrôle de version.

Vous pouvez ensuite utiliser le fichier iLearner enregistré pour évaluer les services web déployés.

## <a name="versioning-web-service"></a>Contrôle de version du service web
Vous pouvez déployer deux types de services web à partir d'une expérience Azure Machine Learning Studio (classique). Le service web classique est étroitement lié à l’expérience, ainsi qu’à l’espace de travail. Le nouveau service web s’appuie sur l’infrastructure Azure Resource Manager ; il n’est plus associé à l’expérience d’origine, ni à l’espace de travail.

### <a name="classic-web-service"></a>Service web classique
Pour contrôler la version d’un service web classique, vous pouvez tirer parti de la construction du point de terminaison du service web. Voici un flux typique :

1. À partir de votre expérience prédictive, vous déployez un nouveau service web classique qui contient un point de terminaison par défaut.
2. Vous créez un point de terminaison nommé ep2, qui expose la version actuelle de l’expérience/du modèle formé.
3. Vous revenez en arrière et mettez à jour votre expérience prédictive et le modèle formé.
4. Vous redéployez l’expérience prédictive, qui met ensuite à jour le point de terminaison par défaut. Mais cela ne modifie en rien ep2.
5. Vous créez un point de terminaison supplémentaire, ep3, qui expose la nouvelle version du modèle formé et de l’expérience.
6. Revenez à l’étape 3 si nécessaire.

Au fil du temps, vous aurez sans doute plusieurs points de terminaison créés dans le même service web. Chaque point de terminaison représente une copie ponctuelle de l’expérience contenant la version ponctuelle du modèle formé. Vous pouvez alors utiliser une logique externe pour déterminer le point de terminaison à appeler, ce qui signifie sélectionner une version du modèle formé pour l’exécution d’évaluation.

Vous pouvez également créer plusieurs points de terminaison de service web identiques et associer différentes versions du fichier iLearner au point de terminaison pour obtenir des effets similaires. Cet [article](create-models-and-endpoints-with-powershell.md) explique en détail la procédure à suivre.

### <a name="new-web-service"></a>Nouveau service web
Si vous créez un service web basé sur Azure Resource Manager, la construction du point de terminaison n’est plus disponible. Au lieu de cela, vous pouvez générer des fichiers WSD (définition de service web), au format JSON, à partir de votre expérience prédictive à l’aide de l’applet de commande PowerShell [Export-AmlWebServiceDefinitionFromExperiment](https://github.com/hning86/azuremlps#export-amlwebservicedefinitionfromexperiment), ou à l’aide de l’applet de commande PowerShell [*Export-AzMlWebservice*](/powershell/module/az.machinelearning/export-azmlwebservice), à partir d’un service web basé sur Resource Manager et déployé.

Si vous avez déjà exporté, puis soumis le fichier WSD à un contrôle de version, vous pouvez également le déployer en tant que nouveau service web dans un plan de service web différent, dans une région Azure différente. Assurez-vous de fournir la configuration de compte de stockage appropriée, ainsi que l’ID du nouveau plan de service web. Pour associer d’autres fichiers iLearner, vous pouvez modifier le fichier WSD et mettre à jour la référence d’emplacement du modèle formé, puis le déployer en tant que nouveau service web.

## <a name="automate-experiment-execution-and-deployment"></a>Automatiser le déploiement et l’exécution de l’expérience
Un aspect important d’ALM consiste à automatiser l’exécution et le déploiement de l’application. Pour ce faire, vous pouvez utiliser le [module PowerShell](https://aka.ms/amlps) dans Azure Machine Learning Studio (classique). Voici un exemple d'étapes de bout en bout qui correspondent à un processus standard d'exécution/de déploiement ALM automatisé, à l'aide du [module PowerShell Azure Learning Studio (classique)](https://aka.ms/amlps). Chaque étape est liée à une ou plusieurs applets de commande PowerShell, que vous pouvez utiliser pour exécuter l’étape en question.

1. [Téléchargez un jeu de données](https://github.com/hning86/azuremlps#upload-amldataset).
2. Copiez une expérience de formation dans l’espace de travail à partir d’un [espace de travail](https://github.com/hning86/azuremlps#copy-amlexperiment) ou de la [galerie](https://github.com/hning86/azuremlps#copy-amlexperimentfromgallery), ou [importez](https://github.com/hning86/azuremlps#import-amlexperimentgraph) à partir du disque local une expérience [exportée](https://github.com/hning86/azuremlps#export-amlexperimentgraph).
3. [Mettez à jour le jeu de données](https://github.com/hning86/azuremlps#update-amlexperimentuserasset) dans l’expérience de formation.
4. [Exécutez l’expérience de formation](https://github.com/hning86/azuremlps#start-amlexperiment).
5. [Transférez le modèle formé](https://github.com/hning86/azuremlps#promote-amltrainedmodel).
6. [Copiez une expérience prédictive](https://github.com/hning86/azuremlps#copy-amlexperiment) dans l’espace de travail.
7. [Mettez à jour le modèle formé](https://github.com/hning86/azuremlps#update-amlexperimentuserasset) dans l’expérience prédictive.
8. [Exécutez l’expérience prédictive](https://github.com/hning86/azuremlps#start-amlexperiment).
9. [Déployez un service web](https://github.com/hning86/azuremlps#new-amlwebservice) à partir de l’expérience prédictive.
10. Testez le point de terminaison [RR](https://github.com/hning86/azuremlps#invoke-amlwebservicerrsendpoint) ou [BES](https://github.com/hning86/azuremlps#invoke-amlwebservicebesendpoint) du service web.

## <a name="next-steps"></a>Étapes suivantes
* Téléchargez le module [PowerShell Azure Machine Learning Studio (classique)](https://aka.ms/amlps) et commencez à automatiser vos tâches ALM.
* Découvrez comment [créer et gérer un grand nombre de modèles ML à l’aide d’une seule expérience](create-models-and-endpoints-with-powershell.md), via PowerShell et l’API de reformation.
* Découvrez plus d’informations sur le [déploiement de services web Azure Machine Learning](deploy-a-machine-learning-web-service.md).