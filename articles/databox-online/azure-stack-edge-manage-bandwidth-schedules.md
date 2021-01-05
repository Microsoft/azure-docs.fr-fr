---
title: Azure Stack Edge Pro – Gérer les planifications de bande passante | Microsoft Docs
description: Explique comment utiliser le portail Azure pour gérer les planifications de bande passante sur votre ressource Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/22/2019
ms.author: alkohli
ms.openlocfilehash: e73a02c93807072e30c8ce2a1a7feb30e9d3c8c6
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978966"
---
# <a name="use-the-azure-portal-to-manage-bandwidth-schedules-on-your-azure-stack-edge-pro"></a>Utiliser le portail Azure pour gérer les planifications de bande passante sur votre ressource Azure Stack Edge Pro  

Cet article explique comment gérer les utilisateurs sur votre ressource Azure Stack Edge Pro. Les planifications de bande passante vous permettent de configurer l’utilisation de la bande passante réseau sur plusieurs planifications selon le moment de la journée. Ces planifications sont applicables aux opérations de chargement et de téléchargement entre votre appareil et le cloud.

Vous pouvez ajouter, modifier ou supprimer des planifications de bande passante pour votre ressource Azure Stack Edge Pro via le portail Azure.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Ajouter une planification
> * Modifier une planification
> * Supprimer une planification


## <a name="add-a-schedule"></a>Ajouter une planification

Pour ajouter une planification, effectuez les étapes suivantes sur le portail Azure.

1. Sur le portail Azure de votre ressource Azure Stack Edge, accédez à **Bande passante**.
2. Dans le volet de droite, sélectionnez **+ Ajouter une planification**.

    ![Sélectionner la bande passante](media/azure-stack-edge-manage-bandwidth-schedules/add-schedule-1.png)

3. Dans le volet **Ajouter une planification** : 

   1. Renseignez les champs **Jour de début**, **Jour de fin**, **Heure de début** et **Heure de fin** de la planification.
   2. Cochez l’option **Toute la journée** si cette planification doit s’exécuter toute la journée.
   3. Le champ **Débit de bande passante** indique la bande passante en mégabits par seconde (Mbits/s) utilisée par votre appareil dans les opérations impliquant le cloud (à la fois pour les chargements et pour les téléchargements). Fournissez un nombre compris entre 20 et 1 000 000 007 pour ce champ.
   4. Si vous ne souhaitez pas limiter le chargement et le téléchargement de données, cochez la case **Bande passante illimitée**.
   5. Sélectionnez **Ajouter**.

      ![Ajouter une planification](media/azure-stack-edge-manage-bandwidth-schedules/add-schedule-2.png)

3. Une planification est créée avec les paramètres spécifiés. Le portail affiche alors cette planification dans la liste des planifications de bande passante.

    ![Liste mise à jour de planifications de bande passante](media/azure-stack-edge-manage-bandwidth-schedules/add-schedule-3.png)

## <a name="edit-schedule"></a>Modifier une planification

Pour modifier une planification de bande passante, procédez comme suit.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Bande passante**. 
2. Dans la liste des planifications de bande passante, sélectionnez la planification que vous voulez modifier.
    ![Sélectionner une planification de bande passante](media/azure-stack-edge-manage-bandwidth-schedules/modify-schedule-1.png)

3. Apportez les modifications souhaitées, puis enregistrez les modifications.

    ![Modification d’une planification](media/azure-stack-edge-manage-bandwidth-schedules/modify-schedule-2.png)

4. Une fois la planification modifiée, la liste des planifications est mise à jour pour refléter cette modification.

    ![Modifier l’utilisateur 2](media/azure-stack-edge-manage-bandwidth-schedules/modify-schedule-3.png)


## <a name="delete-a-schedule"></a>Supprimer une planification

Effectuez les étapes suivantes pour supprimer une planification de bande passante associée à votre appareil Azure Stack Edge Pro.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Bande passante**.  

2. Dans la liste des planifications de bande passante, sélectionnez la planification que vous souhaitez supprimer. Dans **Modifier la planification**, sélectionnez **Supprimer**. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui**.

   ![Supprimer un utilisateur](media/azure-stack-edge-manage-bandwidth-schedules/delete-schedule-2.png)

3. Une fois la planification supprimée, la liste des planifications est mise à jour.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [gérer les partages](azure-stack-edge-manage-shares.md).
