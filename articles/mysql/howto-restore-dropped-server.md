---
title: Restaurer un serveur Azure Database pour MySQL supprimé
description: Cet article décrit comment restaurer un serveur Azure Database pour MySQL supprimé à l’aide du portail Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/09/2020
ms.openlocfilehash: 34dddd8e5f3fb418fc7155630bf82a922e418402
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657088"
---
# <a name="restore-a-dropped-azure-database-for-mysql-server"></a>Restaurer un serveur Azure Database pour MySQL supprimé

Lorsqu’un serveur est supprimé, la sauvegarde du serveur de base de données peut être conservée jusqu’à cinq jours dans le service. La sauvegarde de base de données est accessible et peut être restaurée uniquement à partir de l’abonnement Azure sur lequel le serveur a été initialement installé. Vous pouvez suivre les étapes recommandées ci-dessous pour récupérer une ressource de serveur MySQL supprimée dans les cinq jours suivant la suppression du serveur. Les étapes recommandées ne fonctionnent que si la sauvegarde du serveur est toujours disponible et n’a pas été supprimée du système. 

## <a name="pre-requisites"></a>Conditions préalables
Pour restaurer un serveur Azure Database pour MySQL supprimé, vous avez besoin des éléments suivants :
- Nom de l’abonnement Azure hébergeant le serveur d’origine
- Emplacement où le serveur a été créé

## <a name="steps-to-restore"></a>Étapes de restauration

1. Accédez au [Journal d’activité](https://ms.portal.azure.com/#blade/Microsoft_Azure_ActivityLog/ActivityLogBlade) dans le panneau Surveiller du portail Azure. 

2. Dans Journal d’activité, cliquez sur **Ajouter un filtre** comme indiqué, puis définissez les filtres suivants pour 

    - **Abonnement** = votre abonnement hébergeant le serveur supprimé
    - **Type de ressource** = serveurs Azure Database pour MySQL (Microsoft.DBforMySQL/servers) 
    - **Opération** = supprimer le secteur MySQL (Microsoft.DBforMySQL/servers/delete) 
 
     [![Journal d’activité filtré pour l’opération de suppression du serveur MySQL](./media/howto-restore-dropped-server/activity-log.png)](./media/howto-restore-dropped-server/activity-log.png#lightbox)
   
 3. Double-cliquez sur l’événement Supprimer le serveur MySQL, puis cliquez sur l’onglet JSON et notez les attributs « resourceId » et « submissionTimestamp » dans la sortie JSON. Le resourceId est au format suivant : /subscriptions/ffffffff-ffff-ffff-ffff-ffffffffffff/resourceGroups/TargetResourceGroup/providers/Microsoft.DBforMySQL/servers/deletedserver.
 
 4. Accédez à la [page relative à la création d’un serveur de l’API REST](/rest/api/mysql/servers/create), cliquez sur l’onglet « Essayer » mis en surbrillance en vert et connectez-vous avec votre compte Azure.
 
 5. Indiquez resourceGroupName, serverName (nom du serveur supprimé), subscriptionId, dérivé de l’attribut resourceId capturé à l’étape 3, tandis qu’api-version est prérempli comme illustré dans l’image.
 
     [![Créer un serveur à l’aide de l’API REST](./media/howto-restore-dropped-server/create-server-from-rest-api.png)](./media/howto-restore-dropped-server/create-server-from-rest-api.png#lightbox)
  
 6. Faites défiler la section du corps de la requête et collez le code suivant en remplaçant « Dropped Server Location », « submissionTimestamp » et « resourceId ». Pour « restorePointInTime », spécifiez une valeur de « submissionTimestamp » moins **15 minutes** pour vous assurer que la commande ne génère pas d’erreur.
 
    ```json
    {
        "location": "Dropped Server Location",  
        "properties": 
            {
                "restorePointInTime": "submissionTimestamp - 15 minutes",
                "createMode": "PointInTimeRestore",
                "sourceServerId": "resourceId"
            }
    }
    ```

7. Si vous voyez le code de réponse 201 ou 202, cela signifie que la requête de restauration a été correctement envoyée. 

8. La création du serveur peut prendre du temps en fonction de la taille de la base de données et des ressources de calcul approvisionnées sur le serveur d’origine. L’état de la restauration peut être surveillé à partir du journal d’activité en filtrant sur 
   - **Abonnement** = votre abonnement
   - **Type de ressource** = serveurs Azure Database pour MySQL (Microsoft.DBforMySQL/servers) 
   - **Opération** = mettre à jour la création de serveur MySQL

## <a name="next-steps"></a>Étapes suivantes
- Si vous essayez de restaurer un serveur dans un délai de cinq jours et que vous recevez toujours une erreur après avoir correctement effectué les étapes décrites précédemment, ouvrez un incident de support pour obtenir de l’aide. Si vous essayez de restaurer un serveur supprimé après cinq jours, une erreur est attendue, car le fichier de sauvegarde est introuvable. N’ouvrez pas de ticket de support dans ce cas. L’équipe de support ne peut pas fournir d’assistance si la sauvegarde est supprimée du système. 
- Pour éviter la suppression accidentelle de serveurs, nous vous recommandons vivement d’utiliser des [verrous de ressources](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/preventing-the-disaster-of-accidental-deletion-for-your-mysql/ba-p/825222).