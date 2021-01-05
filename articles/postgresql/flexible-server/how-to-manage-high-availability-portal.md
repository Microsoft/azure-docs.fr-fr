---
title: Gérer la haute disponibilité avec redondance interzone - Portail Azure - Azure Database pour PostgreSQL - Serveur flexible
description: Cet article explique comment activer ou désactiver la haute disponibilité avec redondance interzone dans Azure Database pour PostgreSQL - Serveur flexible à l’aide du portail Azure.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: fc1bca1265139a438fad86bfce770026866d9a2f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929805"
---
# <a name="manage-zone-redundant-high-availability-in-flexible-server"></a>Gérer la haute disponibilité avec redondance interzone dans Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Cet article décrit comment activer ou désactiver la configuration de haute disponibilité avec redondance interzone dans votre serveur flexible.

La fonctionnalité de haute disponibilité provisionne des réplicas principaux et secondaires physiquement séparés dans différentes zones. Pour plus d’informations, consultez la [documentation sur les concepts de haute disponibilité](./concepts-high-availability.md). Vous pouvez choisir d’activer la haute disponibilité pendant ou après la création d’un serveur flexible. Cette page fournit des instructions sur la façon d’activer ou de désactiver la haute disponibilité. Cette opération ne change pas vos autres paramètres, notamment la configuration du réseau virtuel, les paramètres du pare-feu et la conservation de sauvegarde. De même, l’activation et la désactivation de la haute disponibilité sont des opérations en ligne qui n’ont pas d’impact sur la connectivité et les opérations de votre application.

## <a name="pre-requisites"></a>Conditions préalables

La haute disponibilité avec redondance interzone est disponible uniquement dans les régions où plusieurs zones sont prises en charge. 

## <a name="enable-high-availability-during-server-creation"></a>Activer la haute disponibilité lors de la création du serveur

Cette section fournit des détails propres aux champs en rapport avec la haute disponibilité. Vous pouvez suivre ces étapes pour déployer la haute disponibilité lors de la création de votre serveur flexible.

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez Serveur flexible, puis cliquez sur Créer.  Pour plus d’informations sur la façon de renseigner des champs tels que **Abonnement**, **Groupe de ressources**, **Nom du serveur**, **Région** et ainsi de suite, consultez la documentation relative à la création de serveur.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/subscription-region.png" alt-text="Afficher l’abonnement et la région":::

2.  Choisissez votre **zone de disponibilité**. C’est utile si vous souhaitez colocaliser votre application dans la même zone de disponibilité que la base de données afin de réduire la latence. Choisissez **Aucune préférence** si vous souhaitez que le serveur flexible soit déployé sur n’importe quelle zone de disponibilité.
    ![Sélection de la zone de disponibilité]() :::image type="content" source="./media/how-to-manage-high-availability-portal/zone-selection.png" alt-text="Afficher l’abonnement et la région":::  

3.  Cochez la case **Haute disponibilité avec redondance interzone** dans l’option Disponibilité.

    :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-checkbox.png" alt-text="Afficher l’abonnement et la région":::

4.  Si vous souhaitez changer le calcul et le stockage par défaut, cliquez sur **Configurer le serveur**.
 
    :::image type="content" source="./media/how-to-manage-high-availability-portal/configure-server.png" alt-text="Afficher l’abonnement et la région":::  

5.  Si l’option haute disponibilité est cochée, le niveau modulable ne sera pas un choix possible. Vous pouvez choisir le niveau de calcul **Usage général** ou **Mémoire optimisée**. Ensuite, vous pouvez sélectionner la **taille de calcul** de votre choix dans la liste déroulante.

    :::image type="content" source="./media/how-to-manage-high-availability-portal/select-compute.png" alt-text="Afficher l’abonnement et la région":::  


6.  Sélectionnez la **taille de stockage** en Gio à l’aide de la barre glissante, puis une **période de conservation des sauvegardes** comprise entre sept et 35 jours.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/storage-backup.png" alt-text="Afficher l’abonnement et la région"::: 

7. Cliquez sur **Enregistrer**. 

## <a name="enable-high-availability-post-server-creation"></a>Activer la haute disponibilité après la création du serveur

Effectuez les étapes suivantes pour activer la haute disponibilité pour votre serveur flexible existant.

1.  Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre serveur flexible PostgreSQL existant.

2.  Dans la page du serveur flexible, cliquez sur **Haute disponibilité** dans le panneau de gauche pour ouvrir la page de haute disponibilité.
   
     :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-left-panel.png" alt-text="Afficher l’abonnement et la région"::: 

3.  Cochez la case **Haute disponibilité à redondance interzone** pour **activer** l’option, puis cliquez sur **Enregistrer** pour enregistrer la modification.

     :::image type="content" source="./media/how-to-manage-high-availability-portal/enable-high-availability.png" alt-text="Afficher l’abonnement et la région"::: 

4.  Une boîte de dialogue de confirmation indique qu’en cas d’activation de la haute disponibilité, vos frais augmenteront en raison d’un déploiement de serveur et de stockage supplémentaire.

5.  Cliquez sur le bouton **Activer la haute disponibilité** pour activer la haute disponibilité.

6.  Une notification indique que le déploiement de la haute disponibilité est en cours.

## <a name="disable-high-availability"></a>Désactiver la haute disponibilité

Effectuez ces étapes afin de désactiver la haute disponibilité pour votre serveur flexible qui est déjà configuré avec la redondance interzone.

1.  Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre serveur flexible Azure Database pour PostgreSQL existant.

2.  Dans la page du serveur flexible, cliquez sur **Haute disponibilité** dans le panneau avant pour ouvrir la page de haute disponibilité.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-left-panel.png" alt-text="Afficher l’abonnement et la région"::: 

3.  Décochez la case **Haute disponibilité avec redondance interzone** pour **désactiver** l’option. Cliquez ensuite sur **Enregistrer** pour enregistrer la modification.

     :::image type="content" source="./media/how-to-manage-high-availability-portal/disable-high-availability.png" alt-text="Afficher l’abonnement et la région"::: 

4.  Une boîte de dialogue de confirmation s’affiche, dans laquelle vous pouvez confirmer la désactivation de la haute disponibilité.

5.  Cliquez sur le bouton **Désactiver la haute disponibilité** pour désactiver la haute disponibilité.

6.  Une notification indique que la mise hors service du déploiement de haute disponibilité est en cours.

## <a name="next-steps"></a>Étapes suivantes

-   Découvrir la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
