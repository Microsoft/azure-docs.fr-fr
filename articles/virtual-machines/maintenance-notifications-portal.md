---
title: Utiliser le portail pour les notifications de maintenance
description: Affichez les notifications de maintenance pour les machines virtuelles s’exécutant dans Azure et démarrez la maintenance en libre-service à l’aide du portail.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 09693a09e35503a2a221f8de1903b4218b8766f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87082441"
---
# <a name="handling-planned-maintenance-notifications-using-the-portal"></a>Gestion de notifications de maintenance planifiée à l’aide du portail

**Cet article s’applique aux machines virtuelles exécutant Linux et Windows.**

Après la [planification d’une vague de maintenance](maintenance-notifications.md), vous pouvez consulter la liste des machines virtuelles concernées. 

Vous pouvez utiliser le portail Azure et rechercher les machines virtuelles sur lesquelles une maintenance est planifiée.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans la barre de navigation de gauche, cliquez sur **Machines virtuelles**.

3. Dans le volet Machines virtuelles, sélectionnez le bouton **Modifier les colonnes** pour ouvrir la liste des colonnes disponibles.

4. Sélectionnez et ajoutez les colonnes suivantes :

   **État de la maintenance** : Affiche l’état de maintenance de la machine virtuelle. Les valeurs potentielles sont les suivantes :
      
      | Valeur | Description |
      |-------|-------------|
      | Démarrer maintenant | La machine virtuelle étant dans la fenêtre de maintenance libre-service, vous pouvez lancer la maintenance vous-même. Consultez la procédure ci-dessous pour démarrer la maintenance sur votre machine virtuelle. | 
      | Planifiée | Une maintenance est planifiée sur la machine virtuelle, mais aucune option de lancement ne vous est proposée. Pour déterminer la fenêtre de maintenance, sélectionnez la fenêtre Maintenance - Planifiée dans cet affichage ou cliquez sur la machine virtuelle. | 
      | Mise à jour déjà effectuée | Votre machine virtuelle est déjà mise à jour et aucune action supplémentaire n’est nécessaire pour l’instant. | 
      | Réessayer plus tard | Vous avez lancé la maintenance, mais sans succès. Vous pourrez utiliser l’option de maintenance libre-service ultérieurement. | 
      | Réessayer maintenant | Vous pouvez faire une nouvelle tentative de maintenance automatique après un échec. | 
      | - | Votre machine virtuelle ne fait pas partie d’un cycle de maintenance planifiée. |
      

   **Fenêtre Maintenance - Libre-service** : Affiche la fenêtre de temps dans laquelle vous pouvez démarrer vous-même la maintenance de vos machines virtuelles.
   
   **Fenêtre Maintenance - Planifiée** : Affiche la fenêtre de temps dans laquelle Azure assure la maintenance de votre machine virtuelle pour la terminer. 



## <a name="notification-and-alerts-in-the-portal"></a>Notification et alertes dans le portail

Azure communique une planification de maintenance planifiée en envoyant un e-mail au propriétaire et au groupe de copropriétaires de l’abonnement. Vous pouvez ajouter des destinataires et des chaînes à cette communication en créant des alertes de journal d’activité Azure. Pour plus d’informations, consultez [Créer des alertes de journal d’activité sur les notifications de service](../service-health/alerts-activity-log-service-notifications-portal.md).

Veillez à définir le **Type d’événement** comme **Maintenance planifiée** et les **Services** comme **Groupes de machines virtuelles identiques** et/ou **machines virtuelles**.

## <a name="start-maintenance-on-your-vm-from-the-portal"></a>Démarrer la maintenance sur votre machine virtuelle à partir du portail

L’affichage détaillé de la machine virtuelle contient davantage d’informations relatives à la maintenance.  
En haut de l’affichage détaillé, un nouveau ruban de notification est ajouté si votre machine virtuelle fait l’objet d’une vague d’opérations de maintenance planifiées. En outre, une nouvelle option permet de démarrer la maintenance dès que possible. 


Cliquez sur la notification de maintenance pour afficher la page de maintenance avec plus de détails sur la maintenance planifiée. À partir de là, vous pouvez **démarrer la maintenance** sur votre machine virtuelle.

Une fois la maintenance démarrée, votre machine virtuelle est soumise à une maintenance, puis après quelques minutes, l’état de la maintenance est mis à jour pour refléter le résultat.

Si vous avez raté la fenêtre de libre-service, vous pourrez toujours afficher la fenêtre quand votre machine virtuelle sera soumise à une maintenance par Azure. 


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également gérer la maintenance planifiée à l’aide d’[Azure CLI](maintenance-notifications-cli.md) ou de [PowerShell](maintenance-notifications-powershell.md).
