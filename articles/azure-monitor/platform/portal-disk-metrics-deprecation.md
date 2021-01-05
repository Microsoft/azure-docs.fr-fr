---
title: Dépréciation des métriques de disque dans le portail Azure | Microsoft Docs
description: Identifiez les métriques de disque qui ont été dépréciés et comment mettre à jour vos alertes de métriques pour utiliser de nouvelles métriques.
services: azure-monitor
ms.subservice: metrics
ms.topic: conceptual
author: albecker1
ms.author: albecker
ms.date: 03/12/2020
ms.openlocfilehash: 623d9385b9ae6b13c8964f655fb973fe67a0918a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91743707"
---
# <a name="disk-metrics-deprecation-in-the-azure-portal"></a>Dépréciation des métriques de disque dans le portail Azure

Les métriques de disque dépréciées seront bientôt supprimées du portail Azure. Vous pouvez utiliser une nouvelle version de chaque métrique dépréciée. Cet article vous présente les nouvelles métriques et comment mettre à jour vos alertes de métriques afin de les utiliser.

## <a name="list-of-new-metrics"></a>Liste des nouvelles métriques

Ce tableau mappe chaque métrique dépréciée à sa nouvelle métrique correspondante. 

|Métrique dépréciée|Nouvelle métrique (de remplacement)|
|----|----|
|QD par disque de données (déconseillé)|Longueur de file d’attente du disque de données (préversion)|
|Octets lus/s sur disque de données (déconseillé)|Octets lus/s sur disque de données (préversion)|
|Opérations de lecture/s sur disque de données (déconseillé)|Opérations de lecture/s sur disque de données (préversion)|
|Octets écrits/s sur disque de données (déconseillé)|Octets/s écrits sur disque de données (préversion)|
|Opérations d’écriture/s sur disque de données (déconseillé)|Opérations d’écriture/s sur disque de données (préversion)|
|QD du système d'exploitation (déconseillé)|Longueur de file d’attente du système d’exploitation (préversion)|
|Octets lus/s sur système d'exploitation (déconseillé)|Octets lus/s sur système d'exploitation (préversion)|
|Opérations de lecture/s sur système d'exploitation (déconseillé)|Opérations de lecture/s sur système d'exploitation (préversion)|
|Octets écrits/s sur système d'exploitation (déconseillé)|Octets écrits/s sur système d'exploitation (préversion)|
|Opérations d’écriture/s sur système d'exploitation (déconseillé)|Opérations d’écriture/s sur système d'exploitation (préversion)|

<a id="update-metrics" />

## <a name="migrate-metrics-in-your-metric-alerts"></a>Migrer des métriques dans vos alertes de métriques

Mettez à jour vos alertes de métriques pour utiliser de nouvelles métriques.

1. Dans le portail Azure, recherchez **Alertes**. Puis, dans la section **Services**, choisissez **Alertes**.

   > [!div class="mx-imgBorder"]
   > ![Service d’alerte](./media/portal-disk-metrics-deprecation/alert-service-azure-portal.png)

2. Sur la page **Alertes**, choisissez le bouton **Gérer les règles d'alerte**. 

   > [!div class="mx-imgBorder"]
   > ![Gérer les règles d’alerte](./media/portal-disk-metrics-deprecation/manage-alert-rules-button.png)

3. Dans la liste déroulante **Groupe de ressources**, activez la case à cocher **Machines virtuelles** puis, dans la liste déroulante **Type de signal**, cochez la case **Métriques**. 

   > [!div class="mx-imgBorder"]
   > ![Filtrer les alertes](./media/portal-disk-metrics-deprecation/filter-alerts.png)

4. Dans la liste des métriques, identifiez les conditions liées aux disques. Cliquez sur le nom de la règle. 

   Le nom s’affiche sous forme de lien hypertexte dans la colonne **Nom** du tableau.

   > [!div class="mx-imgBorder"]
   > ![Rechercher des conditions de disque](./media/portal-disk-metrics-deprecation/find-disk-conditions.png)

5. Dans la section **Conditions** de la page **Gestion des règles**, cliquez sur la condition de l’alerte. 

   La condition s’affiche sous forme de lien hypertexte.  

   > [!div class="mx-imgBorder"]
   > ![Ajuster les conditions](./media/portal-disk-metrics-deprecation/adjust-condition.png)

   La page **Configurer la logique du signal** s’affiche, et les paramètres de la condition apparaissent dans la section **Logique d’alerte** sur cette page.

6. Créez un enregistrement de ces paramètres à mesure qu’ils disparaissent lorsque vous supprimez la métrique dépréciée.

   > [!div class="mx-imgBorder"]
   > ![Règles de condition](./media/portal-disk-metrics-deprecation/condition-rules.png)

   > [!TIP] 
   > Vous pouvez capturer ces paramètres dans une capture d’écran ou un fichier texte. 

7. Cliquez sur le lien **Retour à la sélection du signal**.

   > [!div class="mx-imgBorder"]
   > ![Retour à la sélection du signal](./media/portal-disk-metrics-deprecation/back-to-signal-selection.png)

8. Sur la page **Configurer la logique du signal**, choisissez la métrique de remplacement appropriée (nouvelle métrique). Utilisez le [tableau ](#update-metrics) qui apparaît plus haut dans cet article pour identifier le nom de la nouvelle métrique.

   > [!TIP] 
   > Commencez à taper dans la barre de recherche pour affiner la liste des noms de métriques. 

   > [!div class="mx-imgBorder"]
   > ![Choisir une nouvelle métrique](./media/portal-disk-metrics-deprecation/choose-new-metric.png)

9. Choisissez le bouton **Terminé**. 

   > [!div class="mx-imgBorder"]
   > ![Définir une nouvelle métrique](./media/portal-disk-metrics-deprecation/set-new-metric.png)

10. Validez vos modifications en choisissant le bouton **Enregistrer**. 

    > [!div class="mx-imgBorder"]
    > ![Enregistrer une nouvelle métrique](./media/portal-disk-metrics-deprecation/save-new-metric.png)






