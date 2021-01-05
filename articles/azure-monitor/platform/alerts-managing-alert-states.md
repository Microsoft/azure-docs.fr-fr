---
title: Gérer les états des alertes et des groupes intelligents
description: Gestion des états des instances d’alerte et de groupe intelligent
ms.subservice: alerts
ms.topic: conceptual
author: anantr
ms.date: 09/24/2018
ms.openlocfilehash: fa1701c59f099f5222ab6454cc67b9dc20d1e7e7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186675"
---
# <a name="manage-alert-and-smart-group-states"></a>Gérer les états des alertes et des groupes intelligents

Les alertes dans Azure Monitor disposent maintenant d’un [état d’alerte et d’une condition de surveillance](./alerts-overview.md), même chose pour les groupes intelligents qui ont un [état de groupe intelligent](./alerts-smartgroups-overview.md?toc=%2fazure%2fazure-monitor%2ftoc.json). Les changements d’état sont désormais inscrits dans l’historique associé à l’alerte ou au groupe intelligent correspondant. Cet article va vous expliquer comment changer l’état d’une alerte ou d’un groupe intelligent.

## <a name="change-the-state-of-an-alert"></a>Changer l’état d’une alerte

1. Vous pouvez modifier l’état d’une alerte de plusieurs manières différentes : 
    * Dans la page Toutes les alertes, cochez la case en regard des alertes dont vous souhaitez modifier l’état, puis cliquez sur Modifier l'état.   
    ![Capture d'écran représentant la page Toutes les alertes, sur laquelle l'option Modifier l'état est sélectionnée.](./media/alerts-managing-alert-states/state-all-alerts.jpg)
    * Dans la page Détails de l'alerte d’une instance d’alerte, vous pouvez cliquer sur Modifier l’état   
    ![Capture d'écran représentant la page Détails de l'alerte, sur laquelle l'option Modifier l'état de l'alerte est sélectionnée.](./media/alerts-managing-alert-states/state-alert-details.jpg)
    * Dans la page Détails de l'alerte d’une instance d’alerte, dans le volet Groupe intelligent, vous pouvez cocher la case en regard des alertes qui vous intéressent.    
    ![Capture d'écran représentant la page des détails de l'alerte de pulsation, sur laquelle certaines instances sont cochées.](./media/alerts-managing-alert-states/state-alert-details-sg.jpg)

    * Dans la page Détails du groupe intelligent, dans la liste des alertes de membre, vous pouvez cocher la case en regard des alertes dont vous souhaitez modifier l’état, puis cliquer sur Modifier l'état.   
    ![Capture d'écran représentant la page Détails du groupe intelligent sur laquelle vous pouvez sélectionner les alertes dont vous souhaitez modifier l'état.](./media/alerts-managing-alert-states/state-sg-details-alerts.jpg)
1. Lorsque vous cliquez sur Modifier l’état, une fenêtre contextuelle s’ouvre et vous permet de sélectionner l’état (Nouveau/Reconnu/Fermé) ainsi que de saisir un commentaire, si nécessaire.   
![Capture d'écran représentant la boîte de dialogue d'alerte de modification des détails.](./media/alerts-managing-alert-states/state-alert-change.jpg)
1. Une fois l’opération terminée, le changement d’état est enregistré dans l’historique de l’alerte correspondante. Vous pouvez accéder à cette information en ouvrant la page Détails correspondante et en recherchant dans l’historique.    
![Capture d'écran présentant l'historique des modifications d'état.](./media/alerts-managing-alert-states/state-alert-history.jpg)

## <a name="change-the-state-of-a-smart-group"></a>Modifier l’état d’un groupe intelligent
1. Vous pouvez modifier l’état d’un groupe intelligent de plusieurs manières différentes :
    1. Dans la page de la liste Groupe intelligent, cochez la case en regard des groupes intelligents dont vous souhaitez modifier l’état, puis cliquez sur Modifier l'état.  
    ![Capture d'écran représentant la page Modifier l'état d'un groupe intelligent.](./media/alerts-managing-alert-states/state-sg-list.jpg)
    1. Dans la page Détails du groupe intelligent, vous pouvez cliquer sur Modifier l’état.        
    ![Capture d'écran représentant la page Détails du groupe intelligent, sur laquelle l'option Modifier l'état du groupe intelligent est sélectionnée.](./media/alerts-managing-alert-states/state-sg-details.jpg)
1. Lorsque vous cliquez sur Modifier l’état, une fenêtre contextuelle s’ouvre et vous permet de sélectionner l’état (Nouveau/Reconnu/Fermé) ainsi que de saisir un commentaire, si nécessaire. 
![Capture d'écran représentant la boîte de dialogue Modifier l'état du groupe intelligent.](./media/alerts-managing-alert-states/state-sg-change.jpg)
   > [!NOTE]
   >  La modification de l'état d'un groupe intelligent ne modifie pas l'état des alertes de membre individuelles.

1. Une fois l’opération terminée, le changement d’état est enregistré dans l’historique du groupe intelligent correspondant. Vous pouvez accéder à cette information en ouvrant la page Détails correspondante et en recherchant dans l’historique.     
![Capture d'écran présentant l'historique des modifications apportées au groupe intelligent.](./media/alerts-managing-alert-states/state-sg-history.jpg)