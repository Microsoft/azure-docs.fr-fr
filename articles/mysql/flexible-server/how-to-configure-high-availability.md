---
title: Gérer la haute disponibilité redondante interzone - Portail Azure - Serveur flexible Azure Database pour MySQL
description: Cet article explique comment activer ou désactiver la haute disponibilité redondante interzone dans le serveur flexible Azure Database pour MySQL à l’aide du Portail Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/21/2020
ms.custom: references_regions
ms.openlocfilehash: d65b074385311e74444929ef74901e402e29ec03
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241733"
---
# <a name="manage-zone-redundant-high-availability-in-azure-database-for-mysql-flexible-server-preview"></a>Gérer la haute disponibilité redondante interzone dans le serveur flexible (préversion) Azure Database pour MySQL

Cet article décrit comment vous pouvez activer ou désactiver la configuration de haute disponibilité redondante interzone dans votre serveur flexible.

La fonctionnalité de haute disponibilité provisionne physiquement des réplicas principaux et secondaires séparés dans différentes zones. Pour plus d’informations, consultez la [documentation sur les concepts de haute disponibilité](./concepts/../concepts-high-availability.md). 

> [!IMPORTANT]
> Vous pouvez uniquement activer la haute disponibilité redondante interzone lors de la création d’un serveur flexible.

Cette page fournit des instructions sur la façon d’activer ou de désactiver la haute disponibilité. Cette opération ne modifie pas vos autres paramètres, y compris la configuration du réseau virtuel, les paramètres du pare-feu et la rétention de sauvegarde. De même, la désactivation de la haute disponibilité est une opération en ligne qui n’a pas d’impact sur la connectivité et les opérations de votre application.

> [!IMPORTANT]
> La haute disponibilité redondante interzone est disponible dans un ensemble limité de régions : Asie Sud-Est, USA Ouest 2, Europe Ouest et USA Est.  

## <a name="enable-high-availability-during-server-creation"></a>Activer la haute disponibilité lors de la création du serveur

Cette section fournit des détails spécifiques pour les champs à haute disponibilité. Vous pouvez suivre ces étapes pour déployer la haute disponibilité lors de la création de votre serveur flexible.

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez Serveur flexible, puis cliquez sur **Créer**.  Pour plus d’informations sur la façon de remplir les champs tels que **Abonnement** , **Groupe de ressources** , **Nom du serveur** , **Région** , etc., consultez la documentation relative à la création de serveur.

2.  Activez la case à cocher **Haute disponibilité redondante interzone** dans l’option Disponibilité.

3.  Si vous souhaitez modifier le calcul et le stockage par défaut, cliquez sur **Configurer le serveur**.

4.  Si l’option haute disponibilité est cochée, le niveau modulable ne sera pas un choix possible. Vous pouvez choisir les niveaux de calcul **Usage général** ou **Mémoire optimisée**.

    > [!IMPORTANT]
    > Nous prenons uniquement en charge la haute disponibilité redondante interzone pour les niveaux tarifaires * *_Usage général_* _ et _*_Mémoire optimisée_*_.

5.  Dans la liste déroulante, sélectionnez la _ *taille de calcul* * de votre choix.

6.  Sélectionnez **Taille de stockage** dans Gio à l’aide de la barre glissante, puis **Période de rétention des sauvegardes** comprise entre 7 et 35 jours.   

## <a name="disable-high-availability"></a>Désactiver la haute disponibilité

Suivez ces étapes pour désactiver la haute disponibilité de votre serveur flexible qui est déjà configuré avec la redondance de zone.

1.  Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre serveur flexible Azure Database pour MySQL.

2.  Dans la page du serveur flexible, cliquez sur **Haute disponibilité** dans le panneau avant pour ouvrir la page de haute disponibilité.

3.  Cliquez sur la case à cocher **Haute disponibilité redondante interzone** pour désactiver l’option, puis sur **Enregistrer** pour enregistrer la modification.

4.  Une boîte de dialogue de confirmation s’affiche, dans laquelle vous pouvez confirmer la désactivation de la haute disponibilité.

5.  Cliquez sur le bouton **Désactiver la haute disponibilité** pour désactiver la haute disponibilité.

6.  Une notification indique que la désactivation du déploiement à haute disponibilité est en cours.

## <a name="next-steps"></a>Étapes suivantes

-   En savoir plus sur la [continuité de l’activité](./concepts-business-continuity.md)
-   Découvrir la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
