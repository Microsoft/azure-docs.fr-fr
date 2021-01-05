---
title: Dépannage d’Azure Monitor for containers | Microsoft Docs
description: Cet article vous permet de dépanner Azure Monitor for containers et de résoudre certains problèmes.
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: 5727702ff973523ce7ab6400c1c7748e0584acbf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010586"
---
# <a name="troubleshooting-azure-monitor-for-containers"></a>Dépannage d’Azure Monitor for containers

Lorsque vous configurez la surveillance de votre cluster Azure Kubernetes Service (AKS) avec Azure Monitor for containers, il se peut que vous rencontriez un problème empêchant la collecte de données ou la récupération de l’état. Cet article décrit en détail certains problèmes courants et indique la façon de les résoudre.

## <a name="authorization-error-during-onboarding-or-update-operation"></a>Erreur d’autorisation durant une opération d’intégration ou de mise à jour

Lors de l’activation d’Azure Monitor pour les conteneurs ou de la mise à jour d’un cluster pour la prise en charge de la collecte des métriques, vous pouvez recevoir une erreur semblable à la suivante : *le client <identité de l’utilisateur> avec l’ID de l’objet <ID d’objet de l’utilisateur> n’est pas autorisé à effectuer l’action « Microsoft.Authorization/roleAssignments/write » sur l’étendue*

Lors du processus de mise à jour ou d’intégration, une tentative d’attribution de rôle **Publication des métriques de surveillance** sur la ressource de cluster est effectuée. L’utilisateur qui initialise le processus d’activation d’Azure Monitor pour les conteneurs ou de mise à jour pour la prise en charge de la collecte des métriques doit disposer de l’autorisation **Microsoft.Authorization/roleAssignments/write** sur l’étendue de la ressource de cluster AKS. Seuls les membres des rôles intégrés **Propriétaire** et **Administrateur des accès utilisateur** se voient accorder l’accès. Si vos stratégies de sécurité nécessitent l’attribution d’autorisations au niveau granulaire, nous vous recommandons d’afficher [des rôles personnalisés](../../role-based-access-control/custom-roles.md) et de les attribuer aux utilisateurs qui en ont besoin.

Vous pouvez également accorder manuellement ce rôle à partir du portail Azure en effectuant les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le portail Azure, cliquez sur **Tous les services** en haut à gauche. Dans la liste des ressources, tapez **Kubernetes**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Azure Kubernetes**.
3. Dans la liste des clusters Kubernetes, sélectionnez-en un.
2. Cliquez sur **Contrôle d’accès (IAM)** dans le menu de gauche.
3. Sélectionnez **+ Ajouter** pour ajouter une attribution de rôle et sélectionnez le rôle **Publication des métriques de surveillance**. Sous le champ **Sélectionner**, tapez **AKS** afin de filtrer les résultats sur les principaux du service des clusters définis dans l’abonnement. Sélectionnez dans la liste celui qui est spécifique à ce cluster.
4. Sélectionnez **Enregistrer** pour finaliser l’attribution du rôle.

## <a name="azure-monitor-for-containers-is-enabled-but-not-reporting-any-information"></a>Azure Monitor for containers est activé, mais aucune information n’est renvoyée

Si Azure Monitor for containers est correctement configuré et activé, mais que les informations d’état ne s’affichent pas ou qu’aucun résultat n’est retourné suite à une requête de journal, vous pouvez diagnostiquer le problème de la façon suivante :

1. Vérifiez l’état de l’agent en exécutant la commande :

    `kubectl get ds omsagent --namespace=kube-system`

    La sortie doit ressembler à l’exemple suivant, qui indique que l’agent a été correctement déployé :

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```
2. Si vous avez des nœuds Windows Server, vérifiez l’état de l’agent en exécutant la commande suivante :

    `kubectl get ds omsagent-win --namespace=kube-system`

    La sortie doit ressembler à l’exemple suivant, qui indique que l’agent a été correctement déployé :

    ```
    User@aksuser:~$ kubectl get ds omsagent-win --namespace=kube-system
    NAME                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
    omsagent-win           2         2         2         2            2           beta.kubernetes.io/os=windows   1d
    ```
3. Vérifiez l’état du déploiement avec la version de l’agent *06072018* (ou ultérieure) à l’aide de cette commande :

    `kubectl get deployment omsagent-rs -n=kube-system`

    La sortie doit ressembler à l’exemple suivant, qui indique que l’agent a été correctement déployé :

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

4. Vérifiez l’état du pod pour confirmer qu’il est bien en cours d’exécution à l’aide de la commande : `kubectl get pods --namespace=kube-system`

    La sortie doit ressembler à l’exemple suivant, avec l’état *En cours d’exécution* pour l’agent OMS :

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system
    NAME                                READY     STATUS    RESTARTS   AGE
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d
    omsagent-484hw                      1/1       Running   0          1d
    omsagent-fkq7g                      1/1       Running   0          1d
    omsagent-win-6drwq                  1/1       Running   0          1d
    ```

## <a name="error-messages"></a>Messages d’erreur

Le tableau ci-dessous récapitule les erreurs connues que vous pouvez rencontrer lors de l’utilisation d’Azure Monitor pour conteneurs.

| Messages d’erreur  | Action |
| ---- | --- |
| Message d’erreur `No data for selected filters`  | L’établissement de la supervision du flux de données pour les nouveaux clusters peut prendre un certain temps. Patientez au moins 10 à 15 minutes avant l’affichage des données de votre cluster. |
| Message d’erreur `Error retrieving data` | Bien que le cluster Azure Kubernetes Service soit configuré pour la supervision de l’intégrité et des performances, une connexion est établie entre le cluster et l’espace de travail Azure Log Analytics. Un espace de travail Log Analytics est utilisé pour stocker toutes les données de supervision de votre cluster. Cette erreur peut se produire lorsque votre espace de travail Log Analytics a été supprimé. Vérifiez si l’espace de travail a été supprimé et, si c’est le cas, vous devrez réactiver l’analyse de votre cluster avec Azure Monitor pour les conteneurs et spécifier un espace de travail existant ou créer un nouvel espace de travail. Pour réactiver, vous devez [désactiver](container-insights-optout.md) la supervision du cluster et [réactiver](container-insights-enable-new-cluster.md) Azure Monitor pour conteneurs. |
| `Error retrieving data` après l’ajout d’Azure Monitor pour conteneurs à l’aide de la commande az aks cli | Lorsque vous activez la surveillance avec `az aks cli`, Azure Monitor pour conteneurs peut ne pas être correctement déployé. Vérifiez si la solution est déployée. Pour vérifier cela, accédez à votre espace de travail Log Analytics et voyez si la solution est disponible en sélectionnant **Solutions** dans le volet de gauche. Pour résoudre ce problème, vous devez redéployer la solution en suivant les instructions de [déploiement d’Azure Monitor pour conteneurs](container-insights-onboard.md) |

Pour vous aider à diagnostiquer le problème, nous vous fournissons un [script de résolution des problèmes](https://aka.ms/troubleshooting-script).

## <a name="azure-monitor-for-containers-agent-replicaset-pods-are-not-scheduled-on-non-azure-kubernetes-cluster"></a>Les pods ReplicaSet d’agent Azure Monitor pour conteneurs ne sont pas planifiés sur un cluster Kubernetes non Azure

Les pods ReplicaSet d’agent Azure Monitor pour conteneurs dépendent des sélecteurs de nœuds suivants sur les nœuds Worker (ou agent) pour la planification :

```
nodeSelector:
  beta.kubernetes.io/os: Linux
  kubernetes.io/role: agent
```

Si aucune étiquette de nœud n’est associée à vos nœuds Worker, les pods ReplicaSet d’agent ne sont pas planifiés. Pour obtenir des instructions sur l’association de l’étiquette, reportez-vous à [Kubernetes assign label selectors](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) (Kubernetes - Attribuer des sélecteurs d’étiquette).

## <a name="performance-charts-dont-show-cpu-or-memory-of-nodes-and-containers-on-a-non-azure-cluster"></a>Les graphiques de performances n’affichent pas l’UC ou la mémoire des nœuds et des conteneurs sur un cluster non Azure

Les pods d’agent Azure Monitor pour conteneurs utilisent le point de terminaison cAdvisor sur l’agent de nœud pour collecter les métriques de performances. Vérifiez que l’agent en conteneur sur le nœud est configuré pour autoriser l’ouverture de `cAdvisor port: 10255` sur tous les nœuds du cluster pour collecter les métriques de performance.

## <a name="non-azure-kubernetes-cluster-are-not-showing-in-azure-monitor-for-containers"></a>Le cluster Kubernetes non Azure n’apparaît pas dans Azure Monitor pour conteneurs

Pour visualiser le cluster Kubernetes non Azure dans Azure Monitor pour conteneurs, un accès en lecture est requis sur l’espace de travail Log Analytics qui prend en charge cet aperçu, ainsi que sur la ressource de solution Container Insights **ContainerInsights (*espace de travail*)** .

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez activer la fonctionnalité de supervision pour collecter des métriques d’intégrité pour les nœuds et pods du cluster AKS et les consulter dans le Portail Azure. Pour savoir comment utiliser Azure Monitor pour les conteneurs, consultez l’article [Connaître l’état d’Azure Kubernetes Service](container-insights-analyze.md).
