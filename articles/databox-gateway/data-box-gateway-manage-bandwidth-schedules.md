---
title: Gérer les planifications de bande passante sur Azure Data Box Gateway | Microsoft Docs
description: Décrit comment utiliser le Portail Azure pour gérer les planifications de bande passante sur votre ressource Azure Data Box Gateway.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: 55676128f482c8fb4dbdeab0523e4556f3957af7
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580882"
---
# <a name="use-the-azure-portal-to-manage-bandwidth-schedules-on-your-azure-data-box-gateway"></a>Utiliser le Portail Azure pour gérer les planifications de bande passante sur votre ressource Azure Data Box Gateway  

Cet article explique comment gérer les planifications de bande passante sur votre ressource Azure Data Box Gateway. Les planifications de bande passante vous permettent de configurer l’utilisation de la bande passante réseau sur plusieurs planifications selon le moment de la journée. Ces planifications sont applicables aux opérations de chargement et de téléchargement entre votre appareil et le cloud.

Vous pouvez ajouter, modifier ou supprimer des planifications de bande passante pour votre ressource Data Box Gateway par le biais du Portail Azure.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Ajouter une planification
> * Modifier une planification
> * Supprimer une planification

## <a name="add-a-schedule"></a>Ajouter une planification

Pour ajouter une planification, procédez comme suit dans le Portail Azure.

1. Dans le Portail Azure, accédez à votre ressource Data Box Gateway, puis accédez à **Bande passante**.
2. Dans le volet droit, cliquez sur **+ Ajouter une planification**.

    ![Ajout d’une planification](media/data-box-gateway-manage-bandwidth-schedules/add-schedule-1.png)

3. Dans le volet **Ajouter une planification** : 

   1. Renseignez les champs **Jour de début**, **Jour de fin**, **Heure de début** et **Heure de fin** de la planification. 
   2. Si cette planification doit s’exécuter sur la journée entière, vous pouvez sélectionner l’option **Toute la journée**. 
   3. Le champ **Débit de bande passante** indique la bande passante en mégabits par seconde (Mbits/s) utilisée par votre appareil dans les opérations impliquant le cloud (à la fois pour les chargements et pour les téléchargements). Fournissez un nombre compris entre 1 et 1 000 pour ce champ. 
   4. Si vous ne souhaitez pas limiter le chargement et le téléchargement de données, cochez la case **Bande passante illimitée**. 
   5. Cliquez sur **Add**.

      ![Cliquez sur Ajouter l’utilisateur 2](media/data-box-gateway-manage-bandwidth-schedules/add-schedule-2.png)

3. Une planification est créée avec les paramètres spécifiés. Le portail affiche alors cette planification dans la liste des planifications de bande passante.


## <a name="edit-schedule"></a>Modifier une planification

Pour modifier une planification de bande passante, procédez comme suit. 

1. Dans le Portail Azure, accédez à votre ressource Data Box Gateway, puis accédez à Bande passante. 
2. Dans la liste des planifications de bande passante, sélectionnez la planification que vous souhaitez modifier.

    ![Modification d’une planification](media/data-box-gateway-manage-bandwidth-schedules/modify-schedule-1.png)

3. Apportez les modifications souhaitées, puis enregistrez les modifications.

    ![Modifier l’utilisateur 2](media/data-box-gateway-manage-bandwidth-schedules/modify-schedule-2.png)

4. Une fois la planification modifiée, la liste des planifications est mise à jour pour refléter cette modification.

    ![Modifier l’utilisateur 3](media/data-box-gateway-manage-bandwidth-schedules/modify-schedule-3.png)


## <a name="delete-a-schedule"></a>Supprimer une planification

Pour supprimer une planification de bande passante associée à votre appareil Data Box Gateway, procédez comme suit.

1. Dans le Portail Azure, accédez à votre ressource Data Box Gateway, puis accédez à **Bande passante**.  

2. Dans la liste des planifications de bande passante, sélectionnez la planification que vous souhaitez supprimer. Cliquez avec le bouton droit pour afficher le menu contextuel, puis cliquez sur **Supprimer**. 

   ![Supprimer un utilisateur](media/data-box-gateway-manage-bandwidth-schedules/delete-schedule-1.png)

3.  Une fois la planification supprimée, la liste des planifications est mise à jour.



## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [gérer la bande passante](data-box-gateway-manage-bandwidth-schedules.md). 
