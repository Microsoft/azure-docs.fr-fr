---
title: Développement et débogage itératifs dans Azure Data Factory
description: Apprenez à développer et à déboguer des pipelines Data Factory de façon itérative dans l’expérience utilisateur ADF
ms.date: 10/29/2020
ms.topic: conceptual
ms.service: data-factory
services: data-factory
documentationcenter: ''
ms.workload: data-services
author: dcstwh
ms.author: weetok
ms.openlocfilehash: 9b28fb24439354e09e5262281a99cd9dc0153a04
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485244"
---
# <a name="iterative-development-and-debugging-with-azure-data-factory"></a>Développement et débogage itératifs dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Factory vous permet de développer et de déboguer de manière itérative des pipelines Data Factory au fur et à mesure que vous développez vos solutions d’intégration de données. Ces fonctionnalités vous permettent de tester vos modifications avant de créer une demande de tirage (pull request) ou de les publier sur le service de la fabrique de données. 

Pour une présentation de huit minutes et la démonstration de cette fonctionnalité, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Iterative-development-and-debugging-with-Azure-Data-Factory/player]

## <a name="debugging-a-pipeline"></a>Débogage d’un pipeline

Lorsque vous créez à l’aide du canevas du pipeline, vous pouvez tester vos activités à l’aide de la capacité **Déboguer**. Lorsque vous effectuez des séries de tests, vous n’êtes pas obligé de publier vos modifications sur la fabrique de données avant de sélectionner **Déboguer**. Cette fonctionnalité est utile dans les scénarios où vous souhaitez vous assurer que les modifications fonctionnent comme prévu avant de mettre à jour le flux de travail de la fabrique de données.

![Fonctionnalité Déboguer sur le canevas du pipeline](media/iterative-development-debugging/iterative-development-1.png)

Lorsque le pipeline est en cours d’exécution, vous pouvez voir les résultats de chaque activité dans l’onglet **Sortie** du canevas du pipeline.

Affichez les résultats de vos séries de tests dans la fenêtre **Sortie** du canevas du pipeline.

![Fenêtre Sortie du canevas du pipeline](media/iterative-development-debugging/iterative-development-2.png)

Après la réussite d’une série de tests, ajoutez d’autres activités à votre pipeline et continuez le débogage de façon itérative. Vous pouvez également **Annuler** une série de tests alors qu’elle est en cours d’exécution.

> [!IMPORTANT]
> En sélectionnant **Déboguer**, le pipeline est exécuté. Par exemple, si le pipeline contient une activité de copie, la série de tests copie des données de la source vers la destination. Par conséquent, nous vous recommandons d’utiliser des dossiers test pour vos activités de copie et autres lors du débogage. Une fois que vous avez débogué le pipeline, basculez vers les dossiers que vous souhaitez utiliser lors des opérations normales.

### <a name="setting-breakpoints"></a>Définition de points d’arrêt

Azure Data Factory vous permet de déboguer un pipeline jusqu’à ce que vous atteigniez une activité précise dans le canevas du pipeline. Placez un point d’arrêt sur l’activité jusqu’à laquelle vous souhaitez tester, puis sélectionnez **Déboguer**. Data Factory garantit que le test s’exécute uniquement jusqu’à l’activité de point d’arrêt sur le canevas du pipeline. Cette fonctionnalité *Déboguer jusqu’à* est utile lorsque vous ne souhaitez pas tester le pipeline en entier, mais uniquement un sous-ensemble des activités à l’intérieur du pipeline.

![Points d’arrêt sur le canevas du pipeline](media/iterative-development-debugging/iterative-development-3.png)

Pour définir un point d’arrêt, sélectionnez un élément du canevas du pipeline. Une option *Déboguer jusqu’à* apparaît sous la forme d’un cercle rouge vide dans le coin supérieur droit de l’élément.

![Avant de définir un point d’arrêt sur l’élément sélectionné](media/iterative-development-debugging/iterative-development-4.png)

Après que vous avez sélectionné l’option *Déboguer jusqu’à*, elle se transforme en un cercle rouge plein pour indiquer que le point d’arrêt est activé.

![Après avoir défini un point d’arrêt sur l’élément sélectionné](media/iterative-development-debugging/iterative-development-5.png)

## <a name="monitoring-debug-runs"></a>Supervision des exécutions de débogage

Lorsque vous exécutez un débogage de pipeline, les résultats s’affichent dans la fenêtre **Sortie** du canevas du pipeline. L’onglet Sortie ne contiendra que l’exécution la plus récente qui a eu lieu pendant la session de navigation en cours. 

![Fenêtre Sortie du canevas du pipeline](media/iterative-development-debugging/iterative-development-2.png)

Pour afficher l’historique des exécutions de débogage ou afficher une liste de toutes les exécutions de débogage actives, vous pouvez accéder à l’expérience **Superviser**. 

![Sélectionner l’icône View active debug runs (Afficher les exécutions de débogage actives)](media/iterative-development-debugging/view-debug-runs.png)

> [!NOTE]
> Le service Azure Data Factory conserve uniquement l’historique des exécutions de débogage pendant 15 jours. 

## <a name="debugging-mapping-data-flows"></a>Débogage de flux de données de mappage

Les flux de données de mappage vous permettent de créer une logique de transformation des données sans code qui s’exécute à grande échelle. Lors de la génération de votre logique, vous pouvez activer une session de débogage pour travailler de manière interactive avec vos données à l’aide d’un cluster Spark en direct. Pour plus d’informations, consultez [Mode de débogage du flux de données de mappage](concepts-data-flow-debug-mode.md).

Vous pouvez superviser les sessions de débogage du flux de données qui sont actives dans une fabrique dans l’expérience **Superviser**.

![Afficher les sessions de débogage du flux de données](media/iterative-development-debugging/view-dataflow-debug-sessions.png)
 
### <a name="debugging-a-pipeline-with-a-data-flow-activity"></a>Débogage d’un pipeline avec une activité de flux de données

Lorsque vous procédez à l’exécution d’un débogage de pipeline avec un flux de données, vous disposez de deux options de calcul à utiliser. Vous pouvez utiliser un cluster de débogage existant ou créer un nouveau cluster juste-à-temps pour vos flux de données.

L’utilisation d’une session de débogage existante réduira considérablement le temps de démarrage du flux de données puisque le cluster fonctionne déjà, mais cette pratique n’est pas recommandée pour les charges de travail complexes ou parallèles, car elle peut échouer lorsque plusieurs travaux sont exécutés en même temps.

L’utilisation de l’exécution d’activité permettra de créer un nouveau cluster en utilisant les paramètres spécifiés dans le runtime d’intégration de chaque activité de flux de données. Cela permet d’isoler chaque travail ; cette méthode doit être utilisée pour des charges de travail complexes ou des tests de performance. Vous pouvez également contrôler le temps jusqu’au dernier octet dans Azure IR, afin que les ressources de cluster utilisées pour le débogage restent disponibles pendant cette période pour traiter des demandes de travaux supplémentaires.

> [!NOTE]
> Si vous avez un pipeline avec des flux de données s’exécutant en parallèle, choisissez « Use Activity Runtime » (Utiliser le runtime d’activité), afin que Data Factory puisse utiliser le Runtime d’intégration que vous avez sélectionné dans votre activité de flux de données. Les flux de données pourront ainsi s’exécuter sur plusieurs clusters et prendre en charge vos exécutions de flux de données parallèles.

![Exécution d’un pipeline avec un flux de données](media/iterative-development-debugging/iterative-development-dataflow.png)

## <a name="next-steps"></a>Étapes suivantes

Après avoir testé vos modifications, promouvez-les dans des environnements plus élevés à l’aide de [l’intégration et le déploiement continus dans Azure Data Factory](continuous-integration-deployment.md).
