---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/16/2020
ms.author: alkohli
ms.openlocfilehash: 27503d1d60d961bac6dd41ebfe4fb083948cb251
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580859"
---
Vous pouvez également voir les métriques pour superviser les performances de l’appareil et, dans certains cas, pour résoudre les problèmes liés à l’appareil.

Effectuez les étapes suivantes dans le portail Azure pour créer un graphique des métriques d’appareil de votre choix.

1. Pour votre ressource dans le portail Azure, accédez à **Supervision > Métriques** et sélectionnez **Ajouter une métrique**.

    ![Ajouter une métrique](media/data-box-gateway-view-metrics/view-metrics-add-metric.png)

2. La ressource est automatiquement renseignée.  

    ![Ressource actuelle](media/data-box-gateway-view-metrics/view-metrics-current-resource.png)

    Pour spécifier une autre ressource, sélectionnez-la. Dans le panneau **Sélectionner une ressource**, sélectionnez l’abonnement, le groupe de ressources, le type de ressource et la ressource pour laquelle vous souhaitez afficher les métriques et sélectionnez **Appliquer**.

    ![Choisir une autre ressource](media/data-box-gateway-view-metrics/view-metrics-choose-another-resource.png)

3. Dans la liste déroulante, sélectionnez une métrique pour superviser votre appareil. Pour obtenir la liste complète de ces métriques, consultez [Métriques sur votre appareil](#metrics-on-your-device).

4. Si une métrique est sélectionnée dans la liste déroulante, l’agrégation peut également être définie. L’agrégation fait référence à la valeur réelle agrégée sur un intervalle de temps spécifié. Les valeurs d’agrégation peuvent être les valeurs moyenne, minimale ou maximale. Sélectionnez Moyenne, Maximum ou Minimum comme agrégation.

    ![Afficher le graphique](media/data-box-gateway-view-metrics/view-metrics-view-chart.png)

5. Si la métrique que vous avez sélectionnée comporte plusieurs instances, l’option de division est disponible. Sélectionnez **Appliquer la division**, puis sélectionnez la valeur de référence à utiliser pour la répartition.

    ![Appliquer la division](media/data-box-gateway-view-metrics/view-metrics-apply-splitting.png)

6. Si vous voulez à présent voir la répartition uniquement pour quelques instances, vous pouvez filtrer les données. Par exemple, dans ce cas, si vous souhaitez voir le débit réseau uniquement pour les deux interfaces réseau connectées sur votre appareil, vous pouvez filtrer ces interfaces. Sélectionnez **Ajouter un filtre** et spécifiez le nom de l’interface réseau pour le filtrage.

    ![Ajouter un filtre](media/data-box-gateway-view-metrics/view-metrics-add-filter.png)

7. Vous pouvez également épingler le graphique au tableau de bord pour y accéder facilement.

    ![Épingler au tableau de bord](media/data-box-gateway-view-metrics/view-metrics-pin-to-dashboard.png)

8. Pour exporter les données du graphique dans une feuille de calcul Excel ou obtenir un lien vers le graphique en vue de le partager, sélectionnez l’option de partage dans la barre de commandes.

    ![Exporter des données](media/data-box-gateway-view-metrics/view-metrics-export-data.png)
