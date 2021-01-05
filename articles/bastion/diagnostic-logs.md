---
title: Activer et utiliser les journaux de ressources Azure Bastion
description: Dans cet article, découvrez comment activer et utiliser les journaux de diagnostic Azure Bastion.
services: bastion
author: charwen
ms.service: bastion
ms.topic: how-to
ms.date: 02/03/2020
ms.author: charwen
ms.openlocfilehash: 1e76fffd17ee565d4103ca8a7bf1523bbd16209d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91445381"
---
# <a name="enable-and-work-with-bastion-resource-logs"></a>Activer et utiliser les journaux de ressources Bastion

Quand les utilisateurs se connectent à des charges de travail à l’aide d’Azure Bastion, Bastion peut enregistrer les diagnostics des sessions à distance. Vous pouvez ensuite utiliser les diagnostics pour voir quels utilisateurs se connectent à quelles charges de travail, à quel moment, depuis où et d’autres informations de journalisation pertinentes. Pour pouvoir utiliser les diagnostics, vous devez activer les journaux de diagnostic sur Azure Bastion. Cet article vous aide à activer les journaux de diagnostic, puis à les afficher.

## <a name="enable-the-resource-log"></a><a name="enable"></a>Activer le journal des ressources

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre ressource Azure Bastion et sélectionnez **Paramètres de diagnostic** sur la page Azure Bastion.

   ![Capture d'écran représentant la page « Paramètres de diagnostic ».](./media/diagnostic-logs/1diagnostics-settings.png)
2. Sélectionnez **Paramètres de diagnostic**, puis **+ Ajouter un paramètre de diagnostic** pour ajouter une destination pour les journaux.

   ![Capture d'écran représentant la page « Paramètres de diagnostic », sur laquelle le bouton « Ajouter un paramètre de diagnostic » est sélectionné.](./media/diagnostic-logs/2add-diagnostic-setting.png)
3. Dans la page **Paramètres de diagnostic**, sélectionnez le type de compte de stockage à utiliser pour stocker les journaux de diagnostic.

   ![Capture d'écran représentant la page « Paramètres de diagnostic », sur laquelle la section permettant de sélectionner un emplacement de stockage en surbrillance.](./media/diagnostic-logs/3add-storage-account.png)
4. Une fois que vous avez configuré les paramètres, le résultat doit ressembler à l’exemple suivant :

   ![exemples de paramètres](./media/diagnostic-logs/4example-settings.png)

## <a name="view-diagnostics-log"></a><a name="view"></a>Afficher le journal de diagnostic

Pour accéder à vos journaux de diagnostic, vous pouvez utiliser directement le compte de stockage que vous avez spécifié lors de l’activation des paramètres de diagnostic.

1. Accédez à votre ressource de compte de stockage, puis à **Conteneurs**. Vous voyez l’objet Blob **insights-logs-bastionauditlogs** créé dans le conteneur d’objets Blob de votre compte de stockage.

   ![paramètres de diagnostic](./media/diagnostic-logs/1-navigate-to-logs.png)
2. Lorsque vous parcourez le conteneur, vous voyez différents dossiers. Ces dossiers indiquent la hiérarchie des ressources pour votre ressource Azure Bastion.

   ![ajouter le paramètre de diagnostic](./media/diagnostic-logs/2-resource-h.png)
3. Accédez à la hiérarchie complète de la ressource Azure Bastion pour laquelle souhaitez récupérer/afficher les journaux de diagnostic. 'y=', 'm=', 'd=', 'h=' et 'm=' indiquent respectivement l’année, le mois, le jour, l’heure et la minute des journaux de ressources.

   ![sélectionner l’emplacement de stockage](./media/diagnostic-logs/3-resource-location.png)
4. Localisez le fichier JSON créé par Azure Bastion contenant les données du journal de diagnostic pour la période à laquelle vous avez accédé.

5. Téléchargez le fichier JSON à partir du conteneur d’objets Blob de votre stockage. Un exemple d’entrée du fichier JSON est présenté ci-dessous pour référence :

   ```json
   { 
   "time":"2019-10-03T16:03:34.776Z",
   "resourceId":"/SUBSCRIPTIONS/<subscripionID>/RESOURCEGROUPS/MYBASTION/PROVIDERS/MICROSOFT.NETWORK/BASTIONHOSTS/MYBASTION-BASTION",
   "operationName":"Microsoft.Network/BastionHost/connect",
   "category":"BastionAuditLogs",
   "level":"Informational",
   "location":"eastus",
   "properties":{ 
      "userName":"<username>",
      "userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36",
      "clientIpAddress":"131.107.159.86",
      "clientPort":24039,
      "protocol":"ssh",
      "targetResourceId":"/SUBSCRIPTIONS/<subscripionID>/RESOURCEGROUPS/MYBASTION/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/LINUX-KEY",
      "subscriptionId":"<subscripionID>",
      "message":"Successfully Connected.",
      "resourceType":"VM",
      "targetVMIPAddress":"172.16.1.5",
      "tunnelId":"<tunnelID>"
   },
   "FluentdIngestTimestamp":"2019-10-03T16:03:34.0000000Z",
   "Region":"eastus",
   "CustomerSubscriptionId":"<subscripionID>"
   }
   ```

## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).
