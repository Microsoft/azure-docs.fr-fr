---
title: Emplacement d’enregistrement et d’écriture des fichiers d’expérimentation
titleSuffix: Azure Machine Learning
description: Découvrez où enregistrer vos fichiers d’entrée et de sortie pour empêcher les erreurs de limites de stockage et la latence des expériences.
services: machine-learning
author: rastala
ms.author: roastala
manager: danielsc
ms.reviewer: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 03/10/2020
ms.openlocfilehash: 49e1e9efbd6f59bd037a8033f83836bf7fc71c43
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630326"
---
# <a name="where-to-save-and-write-files-for-azure-machine-learning-experiments"></a>Emplacement où enregistrer et écrire des fichiers pour les expériences de Azure Machine Learning


Dans cet article, vous découvrez où enregistrer les fichiers d’entrée et où écrire les fichiers de sortie de vos expériences pour empêcher les erreurs de limites de stockage et la latence des expériences.

Quand des exécutions d’entraînement sont lancées sur une [cible de calcul](concept-compute-target.md), elles sont isolées des environnements extérieurs. L’objectif de cette conception est de garantir la reproductibilité et la portabilité des expériences. Si vous exécutez deux fois le même script sur la même cible de calcul ou sur une autre cible, vous obtenez les mêmes résultats. Grâce à cette conception, vous pouvez traiter les cibles de calcul comme des ressources de calcul sans état, chacune d’entre elles n’ayant aucune affinité avec les travaux exécutés une fois qu’ils sont terminés.

## <a name="where-to-save-input-files"></a>Emplacement où enregistrer les fichiers d’entrée

Avant de pouvoir démarrer une expérience sur une cible de calcul ou sur votre ordinateur local, vous devez vérifier que les fichiers nécessaires sont disponibles pour cette cible de calcul, comme les fichiers de dépendance et les fichiers de données dont votre code a besoin pour s’exécuter.

Azure Machine Learning exécute des scripts d’apprentissage en copiant l’intégralité du répertoire source. Si vous avez des données sensibles que vous ne souhaitez pas charger, utilisez un [fichier .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) ou ne l’incluez pas dans le répertoire source. À la place, accédez à vos données à l’aide d’un [magasin de données](/python/api/azureml-core/azureml.data?preserve-view=true&view=azure-ml-py).

La limite de stockage pour les instantanés d’expérience est de 300 Mo et/ou de 2 000 fichiers.

C’est pourquoi nous vous recommandons d’effectuer les opérations suivantes :

* **Stockage de vos fichiers dans un [magasin de données](/python/api/azureml-core/azureml.data?preserve-view=true&view=azure-ml-py) Azure Machine Learning.** Cela évite les problèmes de latence des expériences et présente l’avantage d’un accès aux données à partir d’une cible de calcul distante, ce qui signifie que l’authentification et le montage sont gérés par Azure Machine Learning. Découvrez-en plus sur la spécification d’un magasin de données comme votre répertoire source et sur le chargement de fichiers dans votre magasin de données dans l’article [Accéder aux données à partir de vos magasins de données](how-to-access-data.md).

* **Si vous avez besoin de seulement quelques fichiers de données et scripts de dépendance, et que vous ne pouvez pas utiliser un magasin de données,** placez les fichiers dans le même répertoire de dossier que votre script d’entraînement. Spécifiez ce dossier comme votre `source_directory` directement dans votre script d’entraînement ou dans le code qui appelle votre script d’entraînement.

<a name="limits"></a>

### <a name="storage-limits-of-experiment-snapshots"></a>Limites de stockage des instantanés d’expérience

Pour les expériences, Azure Machine Learning crée automatiquement un instantané d’expérience de votre code en fonction du répertoire que vous suggérez quand vous configurez l’exécution. Cela a une limite totale de 300 Mo et/ou de 2000 fichiers. Si vous dépassez cette limite, l’erreur suivante s’affiche :

```Python
While attempting to take snapshot of .
Your total snapshot size exceeds the limit of 300.0 MB
```

Pour résoudre cette erreur, stockez vos fichiers d’expérience sur un magasin de données. Si vous ne pouvez pas utiliser un magasin de données, le tableau ci-dessous propose des solutions alternatives possibles.

Description de&nbsp;l’expérience|Solution à la limite de stockage
---|---
Moins de 2 000 fichiers et impossibilité d’utiliser un magasin de données| Remplacez la limite de taille des captures instantanées par <br> `azureml._restclient.snapshots_client.SNAPSHOT_MAX_SIZE_BYTES = 'insert_desired_size'`<br> Cette opération peut durer plusieurs minutes en fonction de la taille des fichiers et de leur nombre.
Obligation d’utiliser un répertoire de scripts spécifique| [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
Pipeline|Utilisez un autre sous-répertoire pour chaque étape.
Notebooks Jupyter| Créez un fichier `.amlignore` ou déplacez votre notebook vers un nouveau sous-répertoire vide, puis réexécutez votre code.

## <a name="where-to-write-files"></a>Emplacement où écrire les fichiers

En raison de l’isolation des expériences d’entraînement, les changements apportés aux fichiers qui se produisent lors des exécutions ne sont pas nécessairement conservés en dehors de votre environnement. Si votre script modifie le chemin local des fichiers pour le calcul, les changements ne sont pas conservés pour votre prochaine exécution de l’expérience, et ils ne sont pas automatiquement propagés vers l’ordinateur client. Par conséquent, les changements apportés pendant la première exécution de l’expérience n’affectent pas, et ne doivent pas affecter, ceux de la deuxième.

Lors de l’écriture de changements, nous vous recommandons d’écrire les fichiers dans un magasin de données Azure Machine Learning. Consultez [Accéder aux données à partir de vos magasins de données](how-to-access-data.md).

Si vous n’avez pas besoin d’un magasin de données, écrivez les fichiers dans le dossier `./outputs` et/ou `./logs`.

>[!Important]
> Deux dossiers, *outputs* et *logs*, reçoivent un traitement spécial de la part d’Azure Machine Learning. Pendant l’entraînement, quand vous écrivez des fichiers dans les dossiers `./outputs` and`./logs`, les fichiers sont automatiquement chargés dans votre historique des exécutions. Vous pouvez ainsi y accéder une fois l’exécution terminée.

* **Pour la sortie, comme des messages d’état ou des résultats de notation,** écrivez les fichiers dans le dossier `./outputs` afin qu’ils soient conservés en tant qu’artefacts dans l’historique des exécutions. Soyez attentif au nombre de fichiers écrits dans ce dossier ainsi qu’à leur taille, car il peut se produire une latence quand le contenu est chargé dans l’historique des exécutions. Si la latence pose problème, il est recommandé d’écrire les fichiers dans un magasin de données.

* **Pour enregistrer les fichiers écrits sous forme de journaux dans l’historique des exécutions,** écrivez-les dans le dossier `./logs`. Les journaux sont chargés en temps réel. Cette méthode convient donc pour le streaming des mises à jour automatiques à partir d’une exécution à distance.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur l’[accès aux données à partir de vos magasins de données](how-to-access-data.md).

* En savoir plus sur la [création de cibles de calcul pour l’entraînement et le déploiement de modèle](how-to-create-attach-compute-studio.md)