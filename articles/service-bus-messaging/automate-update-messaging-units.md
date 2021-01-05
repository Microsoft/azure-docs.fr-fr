---
title: Azure Service Bus – Mettre à jour automatiquement les unités de messagerie
description: Cet article vous explique de quelle façon vous pouvez mettre à jour automatiquement les unités de messagerie d’un espace de noms Service Bus.
ms.topic: how-to
ms.date: 09/15/2020
ms.openlocfilehash: 0a72cc991e768a7bed01762d984cc56238ae0ad0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90984647"
---
# <a name="automatically-update-messaging-units-of-an-azure-service-bus-namespace"></a>Mettre à jour automatiquement les unités de messagerie d’un espace de noms Azure Service Bus 
La mise à l’échelle automatique vous permet de disposer de la bonne quantité de ressources en cours d’exécution pour gérer la charge sur votre application. Elle vous permet d’ajouter des ressources pour gérer les augmentations de charge et d’économiser de l’argent en supprimant les ressources qui sont inactives. Pour en savoir plus sur la fonctionnalité de mise à l’échelle automatique d’Azure Monitor, consultez [Vue d’ensemble de la mise à l’échelle automatique dans Microsoft Azure](../azure-monitor/platform/autoscale-overview.md). 

La messagerie Service Bus Premium isole les ressources au niveau processeur et mémoire, ce qui permet d’exécuter chaque charge de travail client de manière isolée. Ce conteneur de ressources est appelé **unité de messagerie**. Pour en savoir plus sur les unités de messagerie, consultez [Messagerie Service Bus Premium](service-bus-premium-messaging.md). 

En utilisant la fonctionnalité de mise à l’échelle automatique pour les espaces de noms Service Bus Premium, vous pouvez spécifier un nombre minimal et maximal d’[unités de messagerie](service-bus-premium-messaging.md) et ajouter ou supprimer automatiquement des unités de messagerie en fonction d’un ensemble de règles. 

Par exemple, vous pouvez implémenter les scénarios de mise à l’échelle suivants pour les espaces de noms Service Bus à l’aide de la fonctionnalité de mise à l’échelle automatique. 

- Augmenter les unités de messagerie pour un espace de noms Service Bus quand l’utilisation du processeur de l’espace de noms est supérieure à 75 %. 
- Diminuer les unités de messagerie pour un espace de noms Service Bus quand l’utilisation du processeur de l’espace de noms est inférieure à 25 %. 
- Utiliser davantage d’unités de messagerie pendant les heures de bureau et moins pendant les heures creuses. 

Cet article vous explique de quelle façon vous pouvez mettre automatiquement à l’échelle un espace de noms Service Bus (mettre à jour les [unités de messagerie](service-bus-premium-messaging.md)) dans le portail Azure. 

> [!IMPORTANT]
> Cet article s’applique uniquement au niveau de service **Premium** d’Azure Service Bus. 

## <a name="autoscale-setting-page"></a>Page des paramètres de mise à l’échelle automatique
Tout d’abord, procédez comme suit pour accéder à la page **Paramètres de mise à l’échelle automatique** pour votre espace de noms Service Bus.

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Dans la barre de recherche, tapez **Service Bus**, sélectionnez **Service Bus** dans la liste déroulante et appuyez sur **Entrée**. 
1. Sélectionnez votre **espace de noms Premium** dans la liste des espaces de noms. 
1. Accédez à la page **Mettre à l’échelle**. 

    :::image type="content" source="./media/automate-update-messaging-units/scale-page.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::

## <a name="manual-scale"></a>Mise à l’échelle manuelle 
Ce paramètre vous permet de définir un nombre fixe d’unités de messagerie pour l’espace de noms. 

1. Dans la page **Paramètre de mise à l’échelle automatique**, sélectionnez **Mise à l’échelle manuelle** si cette option ne l’est pas déjà. 
1. Pour le paramètre **Unités de messagerie**, sélectionnez le nombre d’unités de messagerie dans la liste déroulante.
1. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer le paramètre. 

    :::image type="content" source="./media/automate-update-messaging-units/manual-scale.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::       


## <a name="custom-autoscale---default-condition"></a>Mise à l’échelle automatique personnalisée - Condition par défaut
Vous pouvez configurer la mise à l’échelle automatique des unités de messagerie à l’aide de conditions. Cette condition de mise à l’échelle est exécutée quand aucune des autres conditions de mise à l’échelle ne correspond. Vous pouvez définir la condition par défaut de l’une des manières suivantes :

- Mettre à l’échelle selon une métrique (telle que l’utilisation du processeur ou de la mémoire)
- Mettre à l’échelle selon un nombre spécifique d’unités de messagerie

Vous ne pouvez pas définir une planification de mise à l’échelle automatique selon des jours ou une plage de dates spécifiques pour une condition par défaut. Cette condition de mise à l’échelle est exécutée quand aucune des autres conditions de mise à l’échelle avec planifications ne correspond. 

### <a name="scale-based-on-a-metric"></a>Mettre à l’échelle selon une métrique
La procédure suivante vous montre comment ajouter une condition pour augmenter automatiquement les unités de messagerie (scale-out) quand l’utilisation du processeur est supérieure à 75 % et diminuer les unités de messagerie (scale-in) quand elle est inférieure à 25 %. Les incréments sont effectués de 1 à 2, de 2 à 4 et de 4 à 8. De même, les décrémentations sont effectuées de 8 à 4, de 4 à 2 et de 2 à 1. 

1. Dans la page **Paramètre de mise à l’échelle automatique**, sélectionnez **Mise à l’échelle automatique personnalisée** pour l’option **Choisir comment mettre à l’échelle vos ressources**. 
1. Dans la section **Par défaut** de la page, spécifiez un **nom** pour la condition par défaut. Sélectionnez l’icône en forme de **crayon** pour modifier le texte. 
1. Pour **Mode de mise à l’échelle**, sélectionnez **Mettre à l’échelle selon une métrique**. 
1. Sélectionnez **+ Ajouter une règle**. 

    :::image type="content" source="./media/automate-update-messaging-units/default-scale-metric-add-rule-link.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::    
1. Dans la page **Règle de mise à l’échelle**, procédez comme suit :
    1. Sélectionnez une métrique dans la liste déroulante **Nom de la métrique**. Dans cet exemple, il s’agit de **Processeur**. 
    1. Sélectionnez un opérateur et des valeurs de seuil. Dans cet exemple, il s’agit de **Supérieur à** et **75** pour **Seuil de métrique pour déclencher l’action de mise à l’échelle**. 
    1. Sélectionnez une **opération** dans la section **Action**. Dans cet exemple, la valeur est **Augmenter**. 
    1. Sélectionnez ensuite **Ajouter**.
    
        :::image type="content" source="./media/automate-update-messaging-units/scale-rule-cpu-75.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::       

        > [!NOTE]
        > La fonctionnalité de mise à l’échelle automatique augmente les unités de messagerie de l’espace de noms si l’utilisation globale du processeur dépasse 75 % dans cet exemple. Les incréments sont effectués de 1 à 2, de 2 à 4 et de 4 à 8. 
1. Sélectionnez une nouvelle fois **+ Ajouter une règle**, puis procédez comme suit dans la page **Règle de mise à l’échelle** :
    1. Sélectionnez une métrique dans la liste déroulante **Nom de la métrique**. Dans cet exemple, il s’agit de **Processeur**. 
    1. Sélectionnez un opérateur et des valeurs de seuil. Dans cet exemple, il s’agit des valeurs **Inférieur à** et **25** pour **Seuil de métrique pour déclencher l’action de mise à l’échelle**. 
    1. Sélectionnez une **opération** dans la section **Action**. Dans cet exemple, la valeur est **Diminuer**. 
    1. Sélectionnez ensuite **Ajouter**. 

        :::image type="content" source="./media/automate-update-messaging-units/scale-rule-cpu-25.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::       

        > [!NOTE]
        > La fonctionnalité de mise à l’échelle automatique diminue les unités de messagerie de l’espace de noms si l’utilisation globale du processeur est inférieure à 25 % dans cet exemple. Les décrémentations sont effectuées de 8 à 4, de 4 à 2 et de 2 à 1. 
1. Définissez le nombre **minimal**, **maximal** et **par défaut** d’unités de messagerie.

    :::image type="content" source="./media/automate-update-messaging-units/default-scale-metric-based.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::
1. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer le paramètre de mise à l’échelle automatique. 
        
### <a name="scale-to-specific-number-of-messaging-units"></a>Mettre à l’échelle selon un nombre spécifique d’unités de messagerie
Procédez comme suit pour configurer la règle afin de mettre à l’échelle l’espace de noms pour utiliser un nombre spécifique d’unités de messagerie. Là encore, la condition par défaut est appliquée quand aucune des autres conditions de mise à l’échelle ne correspond. 

1. Dans la page **Paramètre de mise à l’échelle automatique**, sélectionnez **Mise à l’échelle automatique personnalisée** pour l’option **Choisir comment mettre à l’échelle vos ressources**. 
1. Dans la section **Par défaut** de la page, spécifiez un **nom** pour la condition par défaut. 
1. Pour **Mode de mise à l’échelle**, sélectionnez **Mettre à l’échelle sur des unités de messagerie spécifiques**. 
1. Pour **Unités de messagerie**, sélectionnez le nombre d’unités de messagerie par défaut. 

    :::image type="content" source="./media/automate-update-messaging-units/default-scale-messaging-units.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::       

## <a name="custom-autoscale---additional-conditions"></a>Mise à l’échelle automatique personnalisée - Conditions supplémentaires
La section précédente vous montre comment ajouter une condition par défaut pour le paramètre de mise à l’échelle automatique. Cette section vous montre comment ajouter d’autres conditions au paramètre de mise à l’échelle automatique. Pour ces autres conditions non définies par défaut, vous pouvez configurer une planification basée sur des jours spécifiques d’une semaine ou une plage de dates. 

### <a name="scale-based-on-a-metric"></a>Mettre à l’échelle selon une métrique
1. Dans la page **Paramètre de mise à l’échelle automatique**, sélectionnez **Mise à l’échelle automatique personnalisée** pour l’option **Choisir comment mettre à l’échelle vos ressources**. 
1. Sélectionnez **Ajouter une condition de mise à l’échelle** sous le bloc **Par défaut**. 

    :::image type="content" source="./media/automate-update-messaging-units/add-scale-condition-link.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::    
1. Spécifiez un **nom** pour la condition. 
1. Confirmez la sélection de l’option **Mettre à l’échelle selon une métrique**. 
1. Sélectionnez **+ Ajouter une règle** pour ajouter une règle afin d’augmenter les unités de messagerie quand l’utilisation globale du processeur dépasse 75 %. Suivez les étapes de la section [Condition par défaut](#custom-autoscale---default-condition). 
5. Définissez le nombre **minimal**, **maximal** et **par défaut** d’unités de messagerie.
6. Vous pouvez également définir une **planification** sur une condition personnalisée (mais pas sur la condition par défaut). Vous pouvez spécifier des dates de début et de fin pour la condition (ou) sélectionner des jours spécifiques (lundi, mardi, etc.) d’une semaine. 
    1. Si vous sélectionnez **Spécifier des dates de début/fin**, sélectionnez **Fuseau horaire**, **Date et heure de début** et **Date et heure de fin** (comme indiqué dans l’image suivante) pour que la condition soit appliquée. 

       :::image type="content" source="./media/automate-update-messaging-units/custom-min-max-default.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::
    1. Si vous sélectionnez **Répéter des jours spécifiques**, sélectionnez les jours de la semaine, le fuseau horaire, l’heure de début et l’heure de fin auxquels la condition doit s’appliquer. 

        :::image type="content" source="./media/automate-update-messaging-units/repeat-specific-days.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::
  
### <a name="scale-to-specific-number-of-messaging-units"></a>Mettre à l’échelle selon un nombre spécifique d’unités de messagerie
1. Dans la page **Paramètre de mise à l’échelle automatique**, sélectionnez **Mise à l’échelle automatique personnalisée** pour l’option **Choisir comment mettre à l’échelle vos ressources**. 
1. Sélectionnez **Ajouter une condition de mise à l’échelle** sous le bloc **Par défaut**. 

    :::image type="content" source="./media/automate-update-messaging-units/add-scale-condition-link.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::    
1. Spécifiez un **nom** pour la condition. 
2. Pour **Mode de mise à l’échelle**, sélectionnez l’option **Mettre à l’échelle sur des unités de messagerie spécifiques**. 
1. Sélectionnez le nombre d’**unités de messagerie** dans la liste déroulante. 
6. Pour la **planification**, spécifiez des dates de début et de fin pour la condition (ou) sélectionnez des jours spécifiques (lundi, mardi, etc.) d’une semaine ainsi que des heures. 
    1. Si vous sélectionnez **Spécifier des dates de début/fin**, sélectionnez **Fuseau horaire**, **Date et heure de début** et **Date et heure de fin** pour que la condition soit appliquée. 
    
    :::image type="content" source="./media/automate-update-messaging-units/scale-specific-messaging-units-start-end-dates.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::        
    1. Si vous sélectionnez **Répéter des jours spécifiques**, sélectionnez les jours de la semaine, le fuseau horaire, l’heure de début et l’heure de fin auxquels la condition doit s’appliquer.
    
    :::image type="content" source="./media/automate-update-messaging-units/repeat-specific-days-2.png" alt-text="Espace de noms Service Bus - Page Mettre à l’échelle":::

> [!IMPORTANT]
> Pour en savoir plus sur le fonctionnement des paramètres de mise à l’échelle automatique, en particulier sur le choix d’un profil ou d’une condition et l’évaluation de plusieurs règles, consultez [Comprendre les paramètres de mise à l’échelle automatique](../azure-monitor/platform/autoscale-understanding-settings.md).          

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les unités de messagerie, consultez l’article relatif à la [messagerie Premium](service-bus-premium-messaging.md).

