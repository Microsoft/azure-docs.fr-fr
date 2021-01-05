---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/15/2019
ms.author: alkohli
ms.openlocfilehash: 9ac7966538102273b91d6b7f15b90e18ceedd421
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83779936"
---
Vous pouvez également voir les métriques pour superviser les performances de l’appareil et, dans certains cas, pour résoudre les problèmes liés à l’appareil.

Effectuez les étapes suivantes dans le portail Azure pour créer un graphique des métriques d’appareil de votre choix.

1. Pour votre ressource dans le portail Azure, accédez à **Supervision > Métriques** et sélectionnez **Ajouter une métrique**.

    ![Ajouter une métrique](media/data-box-edge-gateway-view-metrics/view-metrics-1.png)

2. La ressource est automatiquement renseignée.  

    ![Ressource actuelle](media/data-box-edge-gateway-view-metrics/view-metrics-2.png)

    Pour spécifier une autre ressource, sélectionnez-la. Dans le panneau **Sélectionner une ressource**, sélectionnez l’abonnement, le groupe de ressources, le type de ressource et la ressource pour laquelle vous souhaitez afficher les métriques et sélectionnez **Appliquer**.

    ![Choisir une autre ressource](media/data-box-edge-gateway-view-metrics/view-metrics-3.png)

3. Dans la liste déroulante, sélectionnez une métrique pour superviser votre appareil. Pour obtenir la liste complète de ces métriques, consultez [Métriques sur votre appareil](#metrics-on-your-device).

4. Si une métrique est sélectionnée dans la liste déroulante, l’agrégation peut également être définie. L’agrégation fait référence à la valeur réelle agrégée sur un intervalle de temps spécifié. Les valeurs d’agrégation peuvent être les valeurs moyenne, minimale ou maximale. Sélectionnez Moyenne, Maximum ou Minimum comme agrégation.

    ![Afficher le graphique](media/data-box-edge-gateway-view-metrics/view-metrics-4.png)

5. Si la métrique que vous avez sélectionnée comporte plusieurs instances, l’option de division est disponible. Sélectionnez **Appliquer la division**, puis sélectionnez la valeur de référence à utiliser pour la répartition.

    ![Appliquer la division](media/data-box-edge-gateway-view-metrics/view-metrics-5.png)

6. Si vous voulez à présent voir la répartition uniquement pour quelques instances, vous pouvez filtrer les données. Par exemple, dans ce cas, si vous souhaitez voir le débit réseau uniquement pour les deux interfaces réseau connectées sur votre appareil, vous pouvez filtrer ces interfaces. Sélectionnez **Ajouter un filtre** et spécifiez le nom de l’interface réseau pour le filtrage.

    ![Ajouter un filtre](media/data-box-edge-gateway-view-metrics/view-metrics-6.png)

7. Vous pouvez également épingler le graphique au tableau de bord pour y accéder facilement.

    ![Épingler au tableau de bord](media/data-box-edge-gateway-view-metrics/view-metrics-7.png)

8. Pour exporter les données du graphique dans une feuille de calcul Excel ou obtenir un lien vers le graphique en vue de le partager, sélectionnez l’option de partage dans la barre de commandes.

    ![Exporter des données](media/data-box-edge-gateway-view-metrics/view-metrics-8.png)
