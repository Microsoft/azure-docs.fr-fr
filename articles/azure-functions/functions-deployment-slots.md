---
title: Emplacements de déploiement Azure Functions
description: Apprendre à créer et à utiliser des emplacements de déploiement avec Azure Functions
author: craigshoemaker
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: cshoe
ms.openlocfilehash: 87d7d4676c604ca7219b7580eb3ce585282a7f11
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327238"
---
# <a name="azure-functions-deployment-slots"></a>Emplacements de déploiement Azure Functions

Les emplacements de déploiement Azure Functions permettent à votre application de fonction d’exécuter différentes instances appelées « emplacements ». Les emplacements sont différents environnements exposés via un point de terminaison disponible publiquement. Une instance d’application est toujours mappée à l’emplacement de production, et vous pouvez permuter des instances affectées à un emplacement à la demande. Les applications de fonction qui s’exécutent dans le cadre du plan App Service peuvent avoir plusieurs emplacements tandis que, dans le cadre du plan Consommation, un seul emplacement est autorisé.

Les éléments suivants reflètent la manière dont les fonctions sont affectées par la permutation d’emplacements :

- La redirection du trafic est transparente. La permutation n’entraîne l’abandon d’aucune demande.
- Si une fonction est en cours d’exécution pendant un permutation, l’exécution se poursuit et les déclencheurs suivants sont routés vers l’instance d’application permutée.

> [!NOTE]
> Les emplacements ne sont actuellement pas disponibles pour le plan Consommation Linux.

## <a name="why-use-slots"></a>Pourquoi utiliser des emplacements ?

L’utilisation d’emplacements de déploiement présente un certain nombre d’avantages. Les scénarios suivants décrivent des utilisations courantes des emplacements :

- **Différents environnements à des fins différentes** : L’utilisation de différents emplacements vous donne la possibilité de différencier des instances d’application avant de basculer vers un emplacement de production ou de préproduction.
- **Préchauffage** : Le déploiement de l’application vers un emplacement plutôt que directement en production permet de « préchauffer » celle-ci avant sa mise en ligne. En outre, l’utilisation d’emplacements réduit la latence pour les charges de travail déclenchées par HTTP. Les instances sont préchauffées avant déploiement, ce qui réduit le démarrage à froid de fonctions nouvellement déployées.
- **Facilité de secours** : Après permutation avec l’emplacement de production, la précédente application en production se retrouve dans l’emplacement de l’application précédemment en préproduction. Si les modifications résultant de la permutation dans l’emplacement de production ne vous conviennent pas, vous pouvez inverser immédiatement la permutation pour revenir à votre « dernière bonne instance connue ».

## <a name="swap-operations"></a>Permuter des opérations

Pendant une permutation, un emplacement est considéré source et l’autre comme cible. L’emplacement source contient l’instance de l’application qui est appliquée à l’emplacement cible. Les étapes suivantes garantissent que l’emplacement cible ne rencontre pas de temps d’arrêt lors d’une permutation :

1. **Appliquer les paramètres :** Les paramètres de l’emplacement cible sont appliqués à toutes les instances de l’emplacement source. Par exemple, les paramètres de l’emplacement production sont appliqués à l’instance en préproduction. Les paramètres appliqués incluent les catégories suivantes :
    - Les paramètres d’application et les chaînes de connexion [spécifiques de l’emplacement](#manage-settings) (le cas échéant)
    - Les paramètres de [déploiement continu](../app-service/deploy-continuous-deployment.md) (si activés)
    - Les paramètres relatifs à l’[authentification App Service](../app-service/overview-authentication-authorization.md) (si activés)

1. **Attendre les redémarrages et la disponibilité :** L’opération de permutation attend que chaque instance dans l’emplacement source ait redémarré et soit disponible pour recevoir des demandes. Si l’une des instances ne parvient pas à redémarrer, l’opération d’échange rétablit toutes les modifications apportées à l’emplacement source, puis cesse d’être exécutée.

1. **Mettre à jour le routage :** Si toutes les instances dans l’emplacement source sont correctement préchauffées, les deux emplacements opèrent la permutation en échangeant leurs règles d’acheminement. Après cette étape, l’emplacement cible (par exemple, l’emplacement de production) dispose de l’application déjà initialisée dans l’emplacement source.

1. **Répéter l’opération :** Maintenant que l’emplacement source contient l’application qui se trouvait dans l’emplacement cible avant la permutation, menez à bien la même opération en appliquant tous les paramètres, puis en redémarrant les instances pour l’emplacement source.

Gardez à l’esprit les points suivants :

- À tout moment pendant une opération de permutation, l’initialisation des applications permutées se produit sur l’emplacement source. L’emplacement cible reste en ligne pendant la préparation de l’emplacement source, que la permutation réussisse ou échoue.

- Pour permuter un emplacement de préproduction avec l’emplacement de production, assurez-vous que l’emplacement de production est *toujours* l’emplacement cible. De cette façon, l’opération d’échange n’affectera pas votre application de production.

- Les paramètres liés aux sources et aux liaisons d’événements doivent être configurés en tant que [paramètres d’emplacement de déploiement](#manage-settings) *avant le lancement d’une permutation*. Le marquage à l’avance de ces paramètres comme « rémanents », garantit que les événements et les sorties sont dirigés vers l’instance appropriée.

## <a name="manage-settings"></a>Gérer les paramètres

Certains paramètres de configuration sont spécifiques à l’emplacement. Les listes suivantes décrivent en détail les paramètres qui évoluent quand vous effectuez un changement d’emplacement, et ceux qui restent identiques.

**Paramètres spécifiques à l’emplacement** :

* Points de terminaison de publication
* Noms de domaine personnalisés
* Certificats non publics et paramètres TLS/SSL
* Paramètres de mise à l’échelle
* Planificateurs WebJobs
* Restrictions d’adresse IP
* Always On
* Paramètres de diagnostic
* Partage des ressources cross-origin (CORS)

**Paramètres non spécifiques à l’emplacement** :

* Paramètres généraux, par exemple versions du framework, 32/64 bits, sockets web
* Paramètres d’application (peuvent être configurés pour respecter un emplacement)
* Chaînes de connexion (peuvent être configurées pour respecter un emplacement)
* Mappages de gestionnaires
* Certificats publics
* Contenu WebJobs
* Connexions hybrides*
* Intégration du réseau virtuel*
* Points de terminaison de service*
* Azure Content Delivery Network*

Il est prévu que les fonctionnalités marquées d’un astérisque (*) ne soient plus échangées. 

> [!NOTE]
> Certains paramètres d’application qui s’appliquent à des paramètres non échangés ne sont pas non plus échangés. Par exemple, étant donné que les paramètres de diagnostic ne sont pas échangés, les paramètres d’application associés comme `WEBSITE_HTTPLOGGING_RETENTION_DAYS` et `DIAGNOSTICS_AZUREBLOBRETENTIONDAYS` ne sont pas non plus échangés, même s’ils n’apparaissent pas comme des paramètres d’emplacement.
>

### <a name="create-a-deployment-setting"></a>Créer un paramètre de déploiement

Vous pouvez marquer des paramètres en tant que paramètres de déploiement, ce qui a pour effet de les rendre « rémanents ». Un paramètre rémanent ne suit pas la permutation de l’instance d’application.

Si vous créez un paramètre de déploiement dans un emplacement, veillez à créer le même paramètre avec une valeur unique dans tout autre emplacement qui est impliqué dans une permutation. De cette façon, si la valeur d’un paramètre ne change pas, son nom reste cohérent dans les emplacements. Cette cohérence de nom garantit que votre code ne tente pas d’accéder à un paramètre défini dans un emplacement, mais pas dans un autre.

Pour créer un paramètre de déploiement, procédez comme suit :

1. Dans l’application de fonction, accédez à **Deployment slots** (Emplacements de déploiement), puis sélectionnez le nom de l’emplacement.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Recherchez des emplacements dans le portail Azure." border="true":::

1. Sélectionnez **Configuration**, puis le nom du paramètre que vous souhaitez utiliser avec l’emplacement actuel.

    :::image type="content" source="./media/functions-deployment-slots/functions-configure-deployment-slot.png" alt-text="Configurez le paramètre d’application d’un emplacement dans le portail Azure." border="true":::

1. Sélectionnez **Deployment slot setting** (Paramètre d’emplacement de déploiement), puis **OK**.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slot-setting.png" alt-text="Configurez le paramètre d’emplacement de déploiement." border="true":::

1. Lorsque la section du paramètre disparaît, sélectionnez **Save** (Enregistrer) pour conserver les modifications

    :::image type="content" source="./media/functions-deployment-slots/functions-save-deployment-slot-setting.png" alt-text="Enregistrez le paramètre d’emplacement de déploiement." border="true":::

## <a name="deployment"></a>Déploiement

Les emplacements sont vides lorsque vous créez un emplacement. Pour déployer votre application dans un emplacement, vous pouvez utiliser toute [technologie de déploiement prise en charge](./functions-deployment-technologies.md).

## <a name="scaling"></a>Mise à l'échelle

Tous les emplacements sont mis à l’échelle du nombre de rôles de travail dans l’emplacement de production.

- Pour les plans Consommation, l’emplacement est mis à l’échelle à mesure que l’échelle de l’application de fonction évolue.
- Pour les plans App Service, l’application s’adapte à un nombre fixe de rôles de travail. Les emplacements s’exécutent sur le même nombre de rôles de travail que le plan de l’application.

## <a name="add-a-slot"></a>Ajouter un emplacement

Vous pouvez ajouter un emplacement via l’[interface de ligne de commande](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-create) ou le portail. Les étapes suivantes montrent comment créer un emplacement sur le portail :

1. Accédez à votre application de fonction.

1. Sélectionnez **Deployment slots** (Emplacements de déploiement), puis **+ Add Slot** (+ Ajouter un emplacement).

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slots-add.png" alt-text="Ajout d’un emplacement de déploiement Azure Functions." border="true":::

1. Tapez le nom de l’emplacement, puis sélectionnez **Add** (Ajouter).

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slots-add-name.png" alt-text="Nommez l’emplacement de déploiement Azure Functions." border="true":::

## <a name="swap-slots"></a>Permuter des emplacements

Vous pouvez permuter des emplacements via l’[interface de ligne de commande](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-swap) ou le portail. Les étapes suivantes expliquent comment permuter des emplacements dans le portail :

1. Accédez à l’application de fonction.
1. Sélectionnez **Deployment slots** (Emplacements de déploiement), puis **Swap** (Échanger).

    :::image type="content" source="./media/functions-deployment-slots/functions-swap-deployment-slot.png" alt-text="Capture d’écran montrant la page « Emplacement de déploiement » avec l’action « Ajouter un emplacement » sélectionnée." border="true":::

1. Vérifiez les paramètres de configuration de votre échange, puis sélectionnez **Swap** (Échanger)
    
    :::image type="content" source="./media/functions-deployment-slots/azure-functions-deployment-slots-swap-config.png" alt-text="Échangez l’emplacement de déploiement." border="true":::

L’exécution de l’opération peut prendre un certain temps.

## <a name="roll-back-a-swap"></a>Restaurer un échange

Si une permutation provoque une erreur ou si vous souhaitez simplement « annuler » une permutation, vous pouvez restaurer l’état initial. Pour revenir à l’état avant permutation, effectuez un autre permutation pour inverser la permutation précédente.

## <a name="remove-a-slot"></a>Supprimer un emplacement

Vous pouvez supprimer un emplacement via l’[interface de ligne de commande](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-delete) ou le portail. Les étapes suivantes montrent comment supprimer un emplacement sur le portail :

1. Dans l’application de fonction, accédez à **Deployment slots** (Emplacements de déploiement), puis sélectionnez le nom de l’emplacement.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Recherchez des emplacements dans le portail Azure." border="true":::

1. Sélectionnez **Supprimer**.

    :::image type="content" source="./media/functions-deployment-slots/functions-delete-deployment-slot.png" alt-text="Capture d’écran montrant la page « Vue d’ensemble » avec l’action « Supprimer » sélectionnée." border="true":::

1. Tapez le nom de l’emplacement de déploiement que vous souhaitez supprimer, puis sélectionnez **Delete** (Supprimer).

    :::image type="content" source="./media/functions-deployment-slots/functions-delete-deployment-slot-details.png" alt-text="Suppression de l’emplacement de déploiement dans le portail Azure." border="true":::

1. Fermez le volet confirmation de la suppression.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slot-deleted.png" alt-text="Confirmation de suppression de l’emplacement de déploiement." border="true":::

## <a name="automate-slot-management"></a>Automatiser la gestion des emplacements

[Azure CLI](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest) vous permet d’automatiser les actions suivantes pour un emplacement :

- [create](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-create)
- [delete](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-delete)
- [list](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-list)
- [swap](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-swap)
- [auto-swap](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-auto-swap)

## <a name="change-app-service-plan"></a>Changer le plan App Service

Une application de fonction qui s’exécute dans le cadre d’un plan App Service vous permet de modifier le plan App service sous-jacent pour un emplacement.

> [!NOTE]
> Vous ne pouvez pas modifier le plan App Service d’un emplacement dans le cadre du plan Consommation.

Pour modifier le plan App service d’un emplacement, procédez comme suit :

1. Dans l’application de fonction, accédez à **Deployment slots** (Emplacements de déploiement), puis sélectionnez le nom de l’emplacement.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Recherchez des emplacements dans le portail Azure." border="true":::

1. Sous **App Service plan** (Plan App Service), sélectionnez **Change App Service plan** (Modifier le plan App Service).

1. Sélectionnez le plan vers lequel vous souhaitez effectuer la mise à niveau ou créez un nouveau plan.

    :::image type="content" source="./media/functions-deployment-slots/azure-functions-deployment-slots-change-app-service-apply.png" alt-text="Modification du plan App Service dans le portail Azure." border="true":::

1. Sélectionnez **OK**.

## <a name="limitations"></a>Limites

Les emplacements de déploiement Azure Functions présentent les limitations suivantes :

- Le nombre d’emplacements disponibles pour une application dépend du plan. Le plan Consommation ne peut disposer que d’un seul emplacement de déploiement. Des emplacements supplémentaires sont disponibles pour des applications s’exécutant dans le cadre du plan App Service.
- La permutation d’un emplacement a pour effet de réinitialiser les clés pour les applications dont le paramètre d’application `AzureWebJobsSecretStorageType` a la valeur `files`.
- Les emplacements ne sont pas disponibles pour le plan Consommation Linux.

## <a name="support-levels"></a>Niveaux de prise en charge

Il existe deux niveaux de prise en charge des emplacements de déploiement :

- **Disponibilité générale** : entièrement pris en charge et approuvé pour la production.
- **Préversion** : pas encore pris en charge, mais le statut de disponibilité générale est prévu à l’avenir.

| Système d’exploitation/Plan d’hébergement           | Niveau de prise en charge     |
| ------------------------- | -------------------- |
| Consommation Windows       | Disponibilité générale |
| Windows Premium           | Disponibilité générale  |
| Dédié (Windows)         | Disponibilité générale |
| Consommation Linux         | Non pris en charge          |
| Linux Premium             | Disponibilité générale  |
| Dédié (Linux)           | Disponibilité générale |

## <a name="next-steps"></a>Étapes suivantes

- [Technologies de déploiement dans Azure Functions](./functions-deployment-technologies.md)
