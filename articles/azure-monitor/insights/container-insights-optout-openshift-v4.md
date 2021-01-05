---
title: Comment arrêter de surveiller votre cluster Azure et Red Hat OpenShift v4 | Microsoft Docs
description: Cet article décrit comment arrêter la surveillance de votre cluster Azure Red Hat OpenShift et Red Hat OpenShift version 4 avec Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: bf61457b9c8cff40eb3fee2c93c7184fbaae6db5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87091145"
---
# <a name="how-to-stop-monitoring-your-azure-and-red-hat-openshift-v4-cluster"></a>Comment arrêter de surveiller votre cluster Azure et Red Hat OpenShift v4

Après avoir activé la surveillance de votre cluster Azure Red Hat OpenShift et Azure Red Hat OpenShift 4.x, vous pouvez l'arrêter si vous décidez de ne plus le surveiller avec Azure Monitor. Cet article explique comment effectuer cette procédure.  

## <a name="how-to-stop-monitoring-using-helm"></a>Arrêt de l’analyse avec Helm

1. Pour identifier d’abord la version de graphique Help pour Azure Monitor pour conteneurs installée sur votre cluster, exécutez la commande Helm suivante.

    ```
    helm list
    ```

    La sortie doit ressembler à ce qui suit :

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *azmon-containers-release-1* représente la version du graphique Helm pour Azure Monitor pour conteneurs.

2. Pour supprimer la version du graphique, exécutez la commande Helm suivante.

    `helm delete <releaseName>`

    Exemple :

    `helm delete azmon-containers-release-1`

    Cette opération supprime la version du cluster. Vous pouvez vérifier cela en exécutant la commande `helm list` :

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

Le changement de configuration peut prendre quelques minutes. Étant donné que Helm suit vos mises en production même après les avoir supprimées, vous pouvez auditer l’historique d’un cluster et même annuler la suppression d'une mise en production avec `helm rollback`.

## <a name="next-steps"></a>Étapes suivantes

Si l’espace de travail Log Analytics a été créé uniquement pour prendre en charge la surveillance du cluster et qu’il n’est plus nécessaire, vous devez le supprimer manuellement. Si vous n’êtes pas familiarisé avec la suppression d’un espace de travail, consultez la rubrique [Supprimer un espace de travail Azure Log Analytics](../platform/delete-workspace.md).
