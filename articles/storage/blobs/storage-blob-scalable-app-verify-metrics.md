---
title: Vérifier les métriques de débit et de latence d’un compte de stockage dans le portail Azure | Microsoft Docs
description: Découvrez comment vérifier les métriques de débit et de latence d’un compte de stockage dans le portail.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.subservice: blobs
ms.openlocfilehash: 708a2369e5ba21f47f3d3fea509654b0385363f7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912312"
---
# <a name="verify-throughput-and-latency-metrics-for-a-storage-account"></a>Vérifier les métriques de débit et de latence d’un compte de stockage

Ce didacticiel est le quatrième et dernier volet d’une série. Dans les didacticiels précédents, vous avez appris à charger et à télécharger des quantités importantes de données aléatoires dans un compte de stockage Azure. Ce didacticiel vous montre comment utiliser des métriques pour afficher le débit et la latence dans le portail Azure.

Dans ce quatrième volet, vous apprenez à :

> [!div class="checklist"]
> * Configurer des graphiques dans le portail Azure
> * Vérifier les métriques de débit et de latence

Les [métriques de stockage Azure](./monitor-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) utilisent Azure Monitor pour fournir une vue unifiée des performances et de la disponibilité de votre compte de stockage.

## <a name="configure-metrics"></a>Configurer les métriques

Accédez à **Métriques (préversion)** sous **PARAMÈTRES** dans votre compte de stockage.

Choisissez Objet blob dans la liste déroulante **SOUS-SERVICE**.

Sous **MÉTRIQUE**, sélectionnez l’une des métriques présentes dans le tableau suivant :

Les métriques suivantes vous donnent une idée de la latence et du débit de l’application. Les métriques que vous configurez dans le portail sont des moyennes d’une minute. Si une transaction se termine au milieu d’une minute, les données de cette minute sont divisées en deux pour la moyenne. Dans l’application, les opérations de chargement et de téléchargement sont minutées. Vous savez ainsi combien de temps il a fallu pour charger et télécharger les fichiers. Vous pouvez utiliser ces informations conjointement avec les métriques du portail pour bien comprendre le débit.

|Métrique|Définition|
|---|---|
|**Latence E2E (réussite)**|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|
|**Latence serveur (réussite)**|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency. |
|**Transactions**|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Dans l’exemple, la taille de bloc est définie à 100 Mo. Dans ce cas, chaque bloc de 100 Mo est considéré comme une transaction.|
|**Entrée**|Quantité de données d’entrée. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure. |
|**Sortie**|Quantité de données de sortie. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables. |

Sélectionnez **Dernières 24 heures (automatique)** à côté de **Durée**. Choisissez **Dernière heure** et **Minute** pour **Granularité temporelle**, puis cliquez sur **Appliquer**.

![Métriques du compte de stockage](./media/storage-blob-scalable-app-verify-metrics/figure1.png)

Vous pouvez affecter plusieurs métriques à un graphique. Mais dans ce cas, vous ne pouvez plus effectuer de regroupement par dimensions.

## <a name="dimensions"></a>Dimensions

Les [dimensions](./monitor-blob-storage-reference.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#metrics-dimensions) permettent d’examiner plus en détail les graphiques et d’obtenir de plus amples informations. Chaque métrique a des dimensions différentes. **Nom de l’API** est l’une des dimensions disponibles. Cette dimension découpe le graphique par appel d’API. La première image ci-dessous montre un exemple de graphique représentant le nombre total de transactions pour un compte de stockage. La deuxième image montre le même graphique, mais avec la dimension Nom de l’API sélectionnée. Comme vous pouvez le voir, chaque transaction est répertoriée. Vous obtenez ainsi plus de détails sur le nombre d’appels effectués par nom d’API.

![Métriques de compte de stockage - transactions sans dimension](./media/storage-blob-scalable-app-verify-metrics/transactionsnodimensions.png)

![Métriques de compte de stockage - transactions](./media/storage-blob-scalable-app-verify-metrics/transactions.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la machine virtuelle et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources de la machine virtuelle et cliquez sur Supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans cette quatrième partie, vous avez appris à afficher des métriques pour l’exemple de solution. Vous avez notamment vu comment :

> [!div class="checklist"]
> * Configurer des graphiques dans le portail Azure
> * Vérifier les métriques de débit et de latence

Suivez ce lien pour consulter des exemples de stockage préconçus.

> [!div class="nextstepaction"]
> [Exemples de script de stockage Azure](storage-samples-blobs-cli.md)

[previous-tutorial]: storage-blob-scalable-app-download-files.md