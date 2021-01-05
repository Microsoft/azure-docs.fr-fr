---
title: Restaurer un serveur Azure Database pour PostgreSQL supprimé
description: Cet article explique comment, à l’aide du portail Azure, restaurer un serveur Azure Database pour PostgreSQL supprimé.
author: Bashar-MSFT
ms.author: bahusse
ms.service: postgresql
ms.topic: how-to
ms.date: 11/03/2020
ms.openlocfilehash: 591f01004cfba247112f702625ab05ddc0aaede3
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652923"
---
# <a name="restore-a-dropped-azure-database-for-postgresql-server"></a>Restaurer un serveur Azure Database pour PostgreSQL supprimé

Lorsqu’un serveur est supprimé, la sauvegarde du serveur de base de données peut être conservée jusqu’à cinq jours dans le service. La sauvegarde de base de données est accessible et peut être restaurée uniquement à partir de l’abonnement Azure sur lequel le serveur a été initialement installé. Vous pouvez suivre les étapes recommandées ci-dessous pour récupérer une ressource de serveur PostgreSQL supprimée dans les cinq jours suivant la suppression du serveur. Les étapes recommandées ne fonctionnent que si la sauvegarde du serveur est toujours disponible et n’a pas été supprimée du système. 

## <a name="pre-requisites"></a>Conditions préalables
Pour restaurer un serveur Azure Database pour PostgreSQL supprimé, vous avez besoin des éléments suivants :
- Nom de l’abonnement Azure hébergeant le serveur d’origine
- Emplacement où le serveur a été créé

## <a name="steps-to-restore"></a>Étapes de restauration

1. Accédez au [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_ActivityLog/ActivityLogBlade). Sélectionnez le service **Azure Monitor**, puis sélectionnez **Journal d’activité**.

2. Dans Journal d’activité, cliquez sur **Ajouter un filtre** comme indiqué, puis définissez ces filtres comme suit

    - **Abonnement** = votre abonnement hébergeant le serveur supprimé
    - **Type de ressource** = Serveurs Azure Database pour PostgreSQL (Microsoft.DBforPostgreSQL/servers).
    - **Opération** = Supprimer le serveur PostgreSQL (Microsoft.DBforPostgreSQL/servers/delete).
 
    ![Journal d’activité filtré pour l’opération de suppression du serveur PostgreSQL](./media/howto-restore-dropped-server/activity-log-azure.png)

3. Sélectionnez l’événement **Supprimer le serveur PostgreSQL**, puis sélectionnez l’**onglet JSON**. Copiez les attributs `resourceId` et `submissionTimestamp` dans la sortie JSON. Le format de resourceId est le suivant : `/subscriptions/ffffffff-ffff-ffff-ffff-ffffffffffff/resourceGroups/TargetResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/deletedserver`.


 4. Accédez à la page PostgreSQL [API REST Créer un serveur](/rest/api/PostgreSQL/servers/create) et sélectionnez l’onglet **Essayer** mis en surbrillance en vert. Connectez-vous à votre compte Azure.

 5. Indiquez les propriétés **resourceGroupName**, **serverName** (nom du serveur supprimé), **subscriptionId**, basées sur la valeur JSON de l’attribut resourceId capturée à l’étape 3 précédente. La propriété api-version est préremplie et peut être laissée telle quelle, comme illustré dans l’image suivante.

    ![Créer un serveur à l’aide de l’API REST](./media/howto-restore-dropped-server/create-server-from-rest-api-azure.png)
  
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

    Par exemple, si la date et l’heure actuelles sont 2020-11-02T23:59:59.0000000Z, nous recommandons un point de restauration dans le temps antérieur de 15 minutes 2020-11-02T23:44:59.0000000Z.

    > [!Important]
    > Il existe un délai limite de cinq jours après la date de suppression du serveur. Au bout de cinq jours, une erreur est attendue, car le fichier de sauvegarde est désormais introuvable.
    
7. Si vous voyez le code de réponse 201 ou 202, cela signifie que la requête de restauration a été correctement envoyée. 

    La création du serveur peut prendre du temps en fonction de la taille de la base de données et des ressources de calcul approvisionnées sur le serveur d’origine. L’état de la restauration peut être surveillé à partir du journal d’activité en filtrant sur 
   - **Abonnement** = votre abonnement
   - **Type de ressource** = Serveurs Azure Database pour PostgreSQL (Microsoft.DBforPostgreSQL/servers). 
   - **Opération** = mettre à jour la création de serveur PostgreSQL

## <a name="next-steps"></a>Étapes suivantes
- Si vous essayez de restaurer un serveur dans un délai de cinq jours et que vous recevez toujours une erreur après avoir correctement effectué les étapes décrites précédemment, ouvrez un incident de support pour obtenir de l’aide. Si vous essayez de restaurer un serveur supprimé après cinq jours, une erreur est attendue, car le fichier de sauvegarde est introuvable. N’ouvrez pas de ticket de support dans ce cas. L’équipe de support ne peut pas fournir d’assistance si la sauvegarde est supprimée du système. 
- Pour éviter la suppression accidentelle de serveurs, nous vous recommandons vivement d’utiliser des [verrous de ressources](https://techcommunity.microsoft.com/t5/azure-database-for-PostgreSQL/preventing-the-disaster-of-accidental-deletion-for-your-PostgreSQL/ba-p/825222).
