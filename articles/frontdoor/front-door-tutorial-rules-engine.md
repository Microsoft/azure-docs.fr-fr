---
title: 'Tutoriel : Configurer le moteur de règles - Azure Front Door'
description: Cet article propose un tutoriel sur la façon de configurer le moteur de règles dans le portail Azure et dans Azure CLI.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/09/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: c15b6115c2b07503320cd7b5f3c790be03aeea94
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92278120"
---
# <a name="tutorial-configure-your-rules-engine"></a>Tutoriel : Configurer votre moteur de règles

Ce tutoriel vous montre comment créer une configuration de moteur de règles, et votre première règle dans le portail Azure et dans Azure CLI. 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Configurer le moteur de règles en utilisant le portail Azure.
> - Configurer le moteur de règles en utilisant Azure CLI.

## <a name="prerequisites"></a>Prérequis

* Avant de pouvoir effectuer les étapes de ce tutoriel, vous devez d’abord créer un Front Door. Pour plus d’informations, consultez [Démarrage rapide : Créer une porte d’entrée](quickstart-create-front-door.md)

## <a name="configure-rules-engine-in-azure-portal"></a>Configurer le moteur de règles dans le portail Azure
1. Dans votre ressource Front Door, accédez à **Paramètres** et sélectionnez **Configuration du moteur de règles**. Cliquez sur **Ajouter**, donnez un nom à votre configuration, puis commencez à créer votre première configuration de moteur de règles.

    ![Menu des paramètres de Front Door](./media/front-door-rules-engine/rules-engine-tutorial-1.png)

1. Cliquez sur **Ajouter une règle** pour créer votre première règle. Ensuite, en cliquant sur **Ajouter une condition** ou **Ajouter une action**, vous pouvez définir votre règle.
    
    > [!NOTE]
    >- Pour supprimer une condition ou une action de la règle, utilisez la corbeille sur le côté droit de la condition ou de l’action spécifique.
    > - Pour créer une règle qui s’applique à tout le trafic entrant, ne spécifiez aucune condition.
    > - Pour arrêter l’évaluation des règles une fois que la première condition de correspondance est remplie, cochez **Arrêter l’évaluation de la règle restante**. Si cette case est cochée alors que toutes les conditions de correspondance d’une règle particulière sont satisfaites, les règles restantes de la configuration ne sont pas exécutées.  

    ![Configuration du moteur de règles](./media/front-door-rules-engine/rules-engine-tutorial-4.png) 

1. Déterminez la priorité des règles dans votre configuration en utilisant les boutons Monter, Descendre et Déplacer en haut. La priorité est dans l’ordre croissant, ce qui signifie que la règle listée en premier est la règle la plus importante.

1. Une fois que vous avez créé une ou plusieurs règles, appuyez sur **Enregistrer**. Cette action crée la configuration de votre moteur de règles.

1. Une fois que vous avez créé une ou plusieurs configurations, associez une configuration de moteur de règles à une règle de routage. Une configuration peut être appliquée à de nombreuses règles de routage, tandis qu’une règle de routage ne peut contenir qu’une seule configuration de moteur de règles. Pour établir l’association, accédez à votre **concepteur Front Door** > **Règles de routage**. Sélectionnez la règle de routage à laquelle vous souhaitez ajouter la configuration de moteur de règles, accédez à **Détails de la route** > **Configuration du moteur de règles**, puis sélectionnez la configuration que vous voulez associer.

    ![Configurer une règle de routage](./media/front-door-rules-engine/rules-engine-tutorial-5.png)


## <a name="configure-rules-engine-in-azure-cli"></a>Configurer le moteur de règles dans Azure CLI

1. Si ce n’est déjà fait, installez [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest). Ajoutez l’extension « Front-Door » :- az extension add --name front-door. Ensuite, connectez-vous et basculez vers votre abonnement az account set --subscription <name_or_Id>.

1. Commencez par créer un moteur de règles : cet exemple montre une règle avec une action basée sur l’en-tête et une condition de correspondance. 

    ```azurecli-interactive
    az network front-door rules-engine rule create -f {front_door} -g {resource_group} --rules-engine-name {rules_engine} --name {rule1} --priority 1 --action-type RequestHeader --header-action Overwrite --header-name Rewrite --header-value True --match-variable RequestFilenameExtension --operator Contains --match-values jpg png --transforms Lowercase
    ```

1. Listez toutes les règles. 

    ```azurecli-interactive
    az network front-door rules-engine rule list -f {front_door} -g {rg} --name {rules_engine}
    ```

1. Ajoutez une action de remplacement de la route de transfert. 

    ```azurecli-interactive
    az network front-door rules-engine rule action add -f {front_door} -g {rg} --rules-engine-name {rules_engine} --name {rule1} --action-type ForwardRouteOverride --backend-pool {backend_pool_name} --caching Disabled
    ```

1. Listez toutes les actions d’une règle. 

    ```azurecli-interactive
    az network front-door rules-engine rule action list -f {front_door} -g {rg} -r {rules_engine} --name {rule1}
    ```

1. Associez une configuration de moteur de règles à une règle de routage.  

    ```azurecli-interactive
    az network front-door routing-rule update -g {rg} -f {front_door} -n {routing_rule_name} --rules-engine {rules_engine}
    ```

1. Dissociez le moteur de règles. 

    ```azurecli-interactive
    az network front-door routing-rule update -g {rg} -f {front_door} -n {routing_rule_name} --remove rulesEngine # case sensitive word ‘rulesEngine’
    ```

Pour plus d’informations, vous trouverez la liste complète des commandes du moteur de règles AFD [ici](/cli/azure/ext/front-door/network/front-door/rules-engine?preserve-view=true&view=azure-cli-latest).   

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez configuré et associé la configuration du moteur de règles à vos règles de routage. Si vous ne voulez plus que la configuration du moteur de règles soit associée à votre instance Front Door, vous pouvez supprimer la configuration en effectuant les étapes suivantes :

1. Dissociez les règles de routage de la configuration du moteur de règles en cliquant sur les trois points en regard de nom du moteur de règles.

    :::image type="content" source="./media/front-door-rules-engine/front-door-rule-engine-routing-association.png" alt-text="Associer des règles de routage":::

1. Décochez toutes les règles de routage auxquelles cette configuration de moteur de règles est associée, puis cliquez sur Enregistrer.

    :::image type="content" source="./media/front-door-rules-engine/front-door-routing-rule-association.png" alt-text="Associer des règles de routage":::

1. Vous pouvez maintenant supprimer la configuration du moteur de règles de votre instance Front Door.

    :::image type="content" source="./media/front-door-rules-engine/front-door-delete-rule-engine-configuration.png" alt-text="Associer des règles de routage":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

* Créer une configuration de moteur de règles
* Associez la configuration à vos règles de routage de Front Door.

Pour découvrir comment ajouter des en-têtes de sécurité avec le moteur de règles, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [En-têtes de sécurité avec le moteur de règles](front-door-security-headers.md)