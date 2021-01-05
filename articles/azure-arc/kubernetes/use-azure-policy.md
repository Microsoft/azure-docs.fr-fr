---
title: Utiliser Azure Policy pour appliquer des configurations de cluster à grande échelle (préversion)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Utiliser Azure Policy pour appliquer des configurations de cluster à grande échelle
keywords: Kubernetes, Arc, Azure, K8s, conteneurs
ms.openlocfilehash: e4279f3d89376320116067bf191e3196271918ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87050041"
---
# <a name="use-azure-policy-to-apply-cluster-configurations-at-scale-preview"></a>Utiliser Azure Policy pour appliquer des configurations de cluster à grande échelle (préversion)

## <a name="overview"></a>Vue d’ensemble

Utiliser Azure Policy pour faire en sorte que chaque ressource `Microsoft.Kubernetes/connectedclusters` ou ressource `Microsoft.ContainerService/managedClusters` activée par Git-Ops fasse l'objet d'une application spécifique de `Microsoft.KubernetesConfiguration/sourceControlConfigurations`. Pour utiliser Azure Policy, vous sélectionnez une définition de stratégie et créez une attribution de stratégie. Lors de la création de l’attribution de stratégie, vous définissez l’étendue de l’attribution, à savoir un groupe de ressources ou un abonnement Azure. Vous définissez également les paramètres pour la configuration `sourceControlConfiguration` qui sera créée. Une fois l’attribution créée, le moteur de stratégie identifie toutes les ressources `connectedCluster` ou `managedCluster` qui se trouvent dans l’étendue, et applique la `sourceControlConfiguration` à chacune d’elles.

Si vous utilisez plusieurs dépôts Git comme sources fidèles pour chaque cluster (par exemple, un dépôt pour l’opérateur IT/de cluster central et d’autres pour les équipes d’applications), vous pouvez activer cela en utilisant plusieurs attributions de stratégies, chacune étant configurée pour utiliser dépôt Git distinct.

## <a name="prerequisite"></a>Configuration requise

Vérifiez que vous disposez des autorisations `Microsoft.Authorization/policyAssignments/write` sur l’étendue (abonnement ou groupe de ressources) dans laquelle vous souhaitez créer cette attribution de stratégie.

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

1. Dans le portail Azure, accédez à Stratégie, puis, dans la section **Création** de la barre latérale, sélectionnez **Définitions**.
2. Choisissez la stratégie intégrée « Déployer GitOps sur le cluster Kubernetes » dans la catégorie « Kubernetes », puis cliquez sur **Affecter**.
3. Définissez l’**Étendue** sur le groupe d’administration, l’abonnement ou le groupe de ressources auxquels s’appliquera l’attribution de stratégie.
4. Si vous souhaitez exclure des ressources de l’étendue de la stratégie, définissez **Exclusions**.
5. Donnez un **Nom** et une **Description** à l’affectation de stratégie, qui permettront de l’identifier facilement.
6. Assurez-vous que l’option **Application de la stratégie** est définie sur *Activée*.
7. Sélectionnez **Suivant**.
8. Définissez les valeurs de paramètre qui seront utilisées lors de la création de la `sourceControlConfiguration`.
9. Sélectionnez **Suivant**.
10. Activez l’option **Créer une tâche de correction**.
11. Assurez-vous que l’option **Créer une identité managée** est activée et que l’identité aura des autorisations de **Contributeur**. Pour plus d’informations sur les autorisations dont vous avez besoin, consultez [ce document](../../governance/policy/assign-policy-portal.md) et [le commentaire dans ce document](../../governance/policy/how-to/remediate-resources.md).
12. Sélectionnez **Revoir + créer**.

Une fois l’attribution de stratégie créée, pour toute nouvelle ressource `connectedCluster` (ou ressource `managedCluster` avec les agents GitOps installés) se trouvant dans l’étendue de l’affectation, la `sourceControlConfiguration` sera appliquée. Pour des clusters existants, vous devrez exécuter manuellement une tâche de correction. La prise d’effet de l’attribution de stratégie prend généralement de 10 à 20 minutes.

## <a name="verify-a-policy-assignment"></a>Vérifier une attribution de stratégie

1. Dans le portail Azure, accédez à l’une de vos ressources `connectedCluster`, puis, dans la section **Paramètres** de la barre latérale, sélectionnez **Stratégies** (l’expérience utilisateur pour le cluster AKS n’est pas encore implémentée, mais sera bientôt disponible).
2. Dans la liste, vous devriez voir l’attribution de stratégie que vous avez créée ci-dessus, et l’**État de conformité** devrait être *Conforme*.
3. Dans la section **Paramètres** de la barre latérale, sélectionnez **Configurations**.
4. Dans la liste, vous devriez voir la `sourceControlConfiguration` que l’attribution de stratégie a créée.
5. Utilisez **kubectl** pour interroger le cluster. Vous devriez voir l’espace de noms et les artefacts créés par la `sourceControlConfiguration`.
6. Après au maximum 5 minutes, vous devriez voir dans le cluster les artefacts décrits dans les manifestes du dépôt Cit configuré.

## <a name="next-steps"></a>Étapes suivantes

* [Configurer Azure Monitor pour des conteneurs avec des clusters Kubernetes compatibles avec Azure Arc](../../azure-monitor/insights/container-insights-enable-arc-enabled-clusters.md)
