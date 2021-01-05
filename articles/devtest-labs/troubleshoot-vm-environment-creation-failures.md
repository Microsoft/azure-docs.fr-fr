---
title: Résoudre des défaillances de machine virtuelle et d’environnement Azure DevTest Labs
description: Découvrez comment résoudre les échecs de création de machine virtuelle et d’environnement dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b7d3f3ad34d8a5bb48607816623c67121d21d78c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476476"
---
# <a name="troubleshoot-virtual-machine-vm-and-environment-creation-failures-in-azure-devtest-labs"></a>Résoudre les échecs de création de machine virtuelle et d’environnement dans Azure DevTest Labs
DevTest Labs vous communique des avertissements si un nom d’ordinateur n’est pas valide ou si vous êtes sur le point de violer une stratégie de laboratoire. Parfois, un symbole `X` rouge apparaît en regard de l’état de l’environnement ou de la machine virtuelle de laboratoire, vous indiquant qu’une erreur est survenue.  Cet article fournit quelques astuces que vous pouvez utiliser pour identifier le problème sous-jacent et, nous l’espérons, pour éviter que le problème ne se reproduise.

## <a name="portal-notifications"></a>Notifications du portail
Si vous utilisez le Portail Azure, la première chose à examiner est le **panneau de notifications**.  Vous pouvez accéder au panneau de notifications en cliquant sur **l’icône représentant une cloche**. Il indique si la création de l’environnement ou de la machine virtuelle de laboratoire a réussi ou non.  En cas d’échec, vous voyez un message d’erreur associé à l’échec de la création. Il offre souvent des informations supplémentaires vous aidant à résoudre le problème. Dans l’exemple suivant, la création de la machine virtuelle a échoué en raison du nombre insuffisant de cœurs. Le message détaillé vous indique comment résoudre le problème et demander une augmentation du quota de cœurs.

![Notification du Portail Azure](./media/troubleshoot-vm-environment-creation-failures/portal-notification.png)

### <a name="vm-in-corruption-state"></a>Machine virtuelle endommagée
Si vous voyez que le statut de votre machine virtuelle dans le laboratoire est **Endommagée**, la machine virtuelle sous-jacente a peut-être été supprimée de la page **Machine virtuelle** à laquelle l’utilisateur peut accéder à partir de la page **Machines virtuelles** (pas la page DevTest Labs). Nettoyez votre laboratoire dans DevTest Labs en supprimant la machine virtuelle du laboratoire. Ensuite, recréez votre machine virtuelle dans le laboratoire. 

![Machine virtuelle endommagée](./media/troubleshoot-vm-environment-creation-failures/vm-corrupted-state.png)



## <a name="activity-logs"></a>Journaux d’activité
Consultez les journaux d’activité si vous examinez un échec se produisant après une tentative de création de machine virtuelle ou d’environnement. Cette section vous montre comment rechercher les journaux d’activité pour les environnements et machines virtuelles.

## <a name="activity-logs-for-virtual-machines"></a>Journaux d’activité pour les machines virtuelles

1. Sur la page d’accueil de votre laboratoire, sélectionnez la machine virtuelle pour ouvrir la page **Machine virtuelle**.
2. Sur la page **Machine virtuelle**, dans la section **MONITORING** du menu de gauche, sélectionnez **Journal d’activité** pour voir tous les journaux associés à la machine virtuelle.
3. Dans les éléments de journal d’activité, sélectionnez l’opération qui a échoué. En règle générale, l’opération qui a échoué est appelée `Write Virtualmachines`.
4. Dans le volet de droite, accédez à l’onglet JSON. Des informations détaillées sont présentées dans l’affichage JSON du journal.

    ![Journal d’activité pour une machine virtuelle](./media/troubleshoot-vm-environment-creation-failures/vm-activity-log.png)
5. Examinez le journal JSON jusqu’à ce que vous trouviez la propriété `statusMessage`. Elle fournit le message d’erreur principal et des informations détaillées supplémentaires, le cas échéant. Le code JSON suivant est un exemple relatif à l’erreur de dépassement du quota de cœurs évoqué plus haut dans cet article.

    ```json
    "properties": {
        "statusCode": "Conflict",
        "statusMessage": "{\"status\":\"Failed\",\"error\":{\"code\":\"ResourceDeploymentFailure\",\"message\":\"The resource operation completed with terminal provisioning state 'Failed'.\",\"details\":[{\"code\":\"OperationNotAllowed\",\"message\":\"Operation results in exceeding quota limits of Core. Maximum allowed: 100, Current in use: 100, Additional requested: 8. Please read more about quota increase at https://aka.ms/corequotaincrease.\"}]}}",
    },
    ```

## <a name="activity-log-for-an-environment"></a>Journal d’activité pour un environnement

Pour afficher le journal d’activité pour la création d’un environnement, procédez comme suit :

1. Sur la page d’accueil de votre laboratoire, sélectionnez **Configuration et stratégies** dans le menu de gauche.
2. Sur la page **Configuration et stratégies**, sélectionnez **Journal d’activité** dans le menu.
3. Recherchez l’échec dans la liste d’activités du journal et sélectionnez-le.
4. Dans le volet de droite, accédez à l’onglet JSON, puis recherchez le **statusMessage**.

    ![Journal d’activité d’un environnement](./media/troubleshoot-vm-environment-creation-failures/envirionment-activity-log.png)

## <a name="resource-manager-template-deployment-logs"></a>Journaux de déploiement de modèle Resource Manager
Si l’environnement ou la machine virtuelle a été créé par automatisation, il existe un dernier emplacement où vous trouverez des informations sur l’erreur. Il s’agit du journal de déploiement de modèle Azure Resource Manager. Lorsqu’une ressource de laboratoire est créée par automatisation, cette création est souvent effectuée par le biais d’un déploiement de modèle Azure Resource Manager. Consultez[https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates) pour obtenir des exemples de modèles Azure Resource Manager qui créent des ressources DevTest Labs.

Pour afficher les journaux de déploiement de modèle de laboratoire, procédez comme suit :

1. Ouvrez la page relative au groupe de ressources dans lequel se trouve le laboratoire.
2. Sélectionnez **Déploiements** dans le menu de gauche, sous **Paramètres**.
3. Recherchez le déploiement présentant un état d’échec et sélectionnez-le.
4. Sur la page **Déploiement**, sélectionnez le lien **Détails de l’opération** pour l’opération qui a échoué.
5. La fenêtre **Détails de l’opération** présente des informations détaillées sur l’opération qui a échoué.

## <a name="next-steps"></a>Étapes suivantes
Consultez [Résoudre les échecs d’artefact](devtest-lab-troubleshoot-artifact-failure.md).
