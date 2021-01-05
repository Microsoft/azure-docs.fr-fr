---
title: À propos des environnements Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Découvrez les environnements d’apprentissage automatique qui permettent des définitions de dépendance d’apprentissage automatique reproductibles, auditables et portables sur diverses cibles de calcul.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: larryfr
author: BlackMist
ms.date: 11/16/2020
ms.openlocfilehash: 78f8d6d216659eaad01d512dd45696dd31035885
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695382"
---
# <a name="what-are-azure-machine-learning-environments"></a>Présentation des environnements Azure Machine Learning

Les environnements Azure Machine Learning sont une encapsulation de l’environnement dans lequel votre formation Machine Learning se produit. Ils spécifient les packages, variables d’environnement et paramètres logiciels Python autour de vos scripts d’apprentissage et de scoring. Ils spécifient également les temps d’exécution (Python, Spark ou Docker). Les environnements sont des entités managées et avec version dans votre espace de travail Machine Learning qui permettent de bénéficier de workflows Machine Learning reproductibles, pouvant être audités et portables parmi une variété de cibles de calcul.

Vous pouvez utiliser un objet `Environment` sur votre calcul local pour :
* Développer votre script d’apprentissage.
* Réutiliser le même environnement sur la capacité de calcul Azure Machine Learning pour l’apprentissage du modèle à l’échelle.
* Déployer votre modèle avec le même environnement.
* Revisitez l’environnement dans lequel un modèle existant a été formé.

Le diagramme suivant montre comment vous pouvez utiliser un seul objet `Environment` à la fois dans votre configuration d’exécution (pour l’apprentissage) et votre configuration d’inférence et de déploiement (pour les déploiements de services web).

![Diagramme d’environnement dans un workflow Machine Learning](./media/concept-environments/ml-environment.png)

L’environnement, la cible de calcul et le script de formation forment la configuration d’exécution : la spécification complète d’une exécution d’apprentissage.

## <a name="types-of-environments"></a>Types d’environnements

Les environnements se répartissent globalement en trois catégories : *organisés*, *gérés par l’utilisateur* et *gérés par le système*.

Les environnements organisés sont fournis par Azure Machine Learning et sont disponibles dans votre espace de travail par défaut. Prévus pour être utilisés tels quels, ils contiennent des collections de packages et paramètres Python destinés à vous aider à prendre en main diverses infrastructures de Machine Learning. Ces environnements précréés offrent également un temps de déploiement plus rapide. Pour obtenir une liste complète, consultez [l’article sur les environnements organisés](resource-curated-environments.md).

Dans un environnement géré par l’utilisateur, vous êtes responsable de la configuration de votre environnement et de l’installation de chaque package dont votre script d’apprentissage a besoin sur la cible de calcul. Conda ne vérifie pas votre environnement et n’installe rien à votre place. Si vous définissez votre propre environnement, vous devez répertorier `azureml-defaults` avec la version `>= 1.0.45` comme dépendance PIP. Ce package contient les fonctionnalités qui sont nécessaires pour héberger le modèle en tant que service web.

Vous utilisez des environnements gérés par le système quand vous souhaitez que [Conda](https://conda.io/docs/) gère automatiquement l’environnement Python et les dépendances de script. Un nouvel environnement conda est généré en fonction de l’objet de dépendances conda. Le service Azure Machine Learning adopte ce type d’environnement par défaut, à cause de son utilité sur les cibles de calcul distantes qui ne sont pas configurables manuellement.

## <a name="create-and-manage-environments"></a>Créer et gérer des environnements

Vous pouvez créer des environnements en :

* Définissant de nouveaux objets `Environment`, soit en utilisant un environnement organisé, soit en définissant vos propres dépendances.
* utilisant des objets `Environment` existants à partir de votre espace de travail. Cette approche permet de garantir la cohérence et la reproductibilité de vos dépendances.
* Importation à partir d’une définition d’environnement Anaconda existante.
* Utilisation de l’interface CLI Azure Machine Learning
* [Utilisation de l’extension VS Code](how-to-manage-resources-vscode.md#create-environment)

Pour des exemples de code spécifiques, consultez la section « Créer un environnement » de [Comment utiliser les environnements](how-to-use-environments.md#create-an-environment). Les environnements sont également facilement gérés via votre espace de travail. Ils doivent inclure la fonctionnalité suivante :

* Les environnements sont automatiquement inscrits à votre espace de travail lorsque vous soumettez une expérience. Ils peuvent également être inscrits manuellement.
* Vous pouvez récupérer des environnements à partir de votre espace de travail à utiliser pour l’apprentissage ou le déploiement, ou bien pour apporter des modifications à la définition de l’environnement.
* Avec le contrôle de version, vous pouvez voir les modifications apportées à vos environnements dans le temps, ce qui garantit la reproductibilité.
* Vous pouvez créer automatiquement des images Docker à partir de vos environnements.

Pour des exemples de code, consultez la section « Gérer les environnements » de [Comment utiliser les environnements](how-to-use-environments.md#manage-environments).

## <a name="environment-building-caching-and-reuse"></a>Création, mise en cache et réutilisation d’environnement

Le service Azure Machine Learning génère des définitions d’environnement dans des images Docker et des environnements Conda. Il met également en cache les environnements afin qu’ils puissent être réutilisés lors des exécutions d’entraînement et des déploiements de points de terminaison de service ultérieurs. L’exécution d’un script d’apprentissage à distance requiert la création d’une image Docker, tandis qu’une exécution locale peut utiliser directement un environnement conda. 

### <a name="submitting-a-run-using-an-environment"></a>Envoi d’une exécution à l’aide d’un environnement

Quand vous envoyez pour la première fois une exécution à distance à l’aide d’un environnement, le service Azure Machine Learning appelle une [tâche de génération ACR](../container-registry/container-registry-tasks-overview.md) sur l’Azure Container Registry (ACR) associé à l’espace de travail. L’image Docker génrée est ensuite mise en cache sur l’ACR Espace de travail. Les environnements organisés sont sauvegardés par des images Docker mises en cache dans l’ACR global. Au début de l’exécution, l’image est récupérée par la cible de calcul de l’ACR pertinent.

Pour les exécutions locales, un environnement Docker ou conda est créé en fonction de la définition de l’environnement. Les scripts sont ensuite exécutés sur le calcul cible (un environnement d’exécution local ou un moteur Docker local).

### <a name="building-environments-as-docker-images"></a>Génération d’environnements en tant qu’images Docker

Si la définition de l’environnement n’existe pas encore dans l’ACR Espace de travail, une nouvelle image est générée. La génération d’image comprend deux étapes :

 1. Téléchargement d’une image de base et exécution des étapes Docker
 2. Génération d’un environnement Conda en fonction des dépendances Conda spécifiées dans la définition d’environnement

La deuxième étape est omise si vous spécifiez des [dépendances gérées par l’utilisateur](/python/api/azureml-core/azureml.core.environment.pythonsection?preserve-view=true&view=azure-ml-py). Dans ce cas, vous êtes responsable de l’installation des packages Python, en les incluant dans votre image de base, ou de la spécification des étapes Docker personnalisées lors de la première étape. Vous devez aussi spécifier l’emplacement correct de l’exécutable Python. Il est également possible d’utiliser une [image de base Docker personnalisée](how-to-deploy-custom-docker-image.md).

### <a name="image-caching-and-reuse"></a>Mise en cache et réutilisation des images

Si vous utilisez la même définition d’environnement pour une autre exécution, le service Azure Machine Learning réutilise l’image mise en cache à partir de l’ACR Espace de travail. 

Pour afficher les détails d’une image mise en cache, utilisez la méthode [Environment.get_image_details](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-image-details-workspace-).

Pour déterminer s’il faut réutiliser une image mise en cache ou en créer une nouvelle, le service calcule [une valeur de hachage](https://en.wikipedia.org/wiki/Hash_table) à partir de la définition d’environnement et la compare aux hachages des environnements existants. Le hachage est basé sur :
 
 * La valeur de la propriété d’image de base.
 * La valeur de la propriété des étapes Docker personnalisées
 * La liste des packages Python dans la définition Conda.
 * La liste des packages dans la définition Spark. 

Le hachage ne dépend pas du nom ou de la version de l’environnement ; si vous renommez simplement votre environnement ou si vous en créez un nouveau avec les propriétés et packages exacts d’un environnement existant, la valeur de hachage reste la même. Cependant, les modifications de définition d’environnement, telles que l’ajout ou la suppression d’un package Python ou la modification de la version du package, entraînent la modification de la valeur de hachage. La modification de l’ordre des dépendances ou des canaux dans un environnement entraîne un nouvel environnement et nécessite donc une nouvelle génération d’image. Il est important de noter que toute modification apportée à un environnement organisé invalidera le hachage et générera un nouvel environnement « non organisé ».

La valeur de hachage calculée est comparée à celles de l’espace de travail et de l’ACR global (ou sur la cible de calcul pour les exécutions locales). En cas de correspondance, l’image mise en cache est extraite, sinon une génération d’image est déclenchée. La durée d’extraction d’une image mise en cache comprend le temps de téléchargement, tandis que la durée d’extraction d’une image nouvellement générée comprend la durée de génération et le temps de téléchargement. 

Le diagramme suivant montre trois définitions d’environnement. Deux d’entre elles ont un nom et une version différents, mais une image de base et des packages Python identiques. Mais elles ont le même hachage et correspondent donc à la même image mise en cache. Le troisième environnement a des packages et des versions Python différents, et correspond donc à une image mise en cache différente.

![Diagramme de la mise en cache d’environnement en tant qu’images Docker](./media/concept-environments/environment-caching.png)

>[!IMPORTANT]
> Si vous créez un environnement avec une dépendance de package désépinglée, par exemple ```numpy```, cet environnement continuera à utiliser la version de package installée _au moment de la création de l’environnement_. En outre, tout environnement ultérieur avec une définition correspondante continuera à utiliser l’ancienne version. 

Pour mettre à jour le package, spécifiez un numéro de version afin de forcer la regénération de l’image, par exemple ```numpy==1.18.1```. Les nouvelles dépendances, notamment celles imbriquées, seront installées et risquent de provoquer le non-fonctionnement d’un scénario précédemment opérationnel. 

> [!WARNING]
>  La méthode [Environment.build](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py#&preserve-view=truebuild-workspace--image-build-compute-none-) regénérera l’image mise en cache, avec comme effet secondaire possible la mise à jour des packages désépinglés et la rupture de la reproductibilité pour toutes les définitions d’environnement correspondant à cette image mise en cache.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer et utiliser des environnements](how-to-use-environments.md) dans Azure Machine Learning.
* Consultez la documentation de référence du kit de développement logiciel (SDK) Python pour la [classe d’environnement](/python/api/azureml-core/azureml.core.environment%28class%29?preserve-view=true&view=azure-ml-py).
* Consultez la documentation de référence du kit de développement logiciel (SDK) R pour les [environnements](https://azure.github.io/azureml-sdk-for-r/reference/index.html#section-environments).