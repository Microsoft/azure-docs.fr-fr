---
title: Gérer un cluster Azure Kubernetes Service avec le tableau de bord web
description: Découvrez comment utiliser le tableau de bord intégré de l’interface utilisateur web de Kubernetes pour gérer un cluster Azure Kubernetes Service (AKS).
services: container-service
author: mlearned
ms.topic: article
ms.date: 06/03/2020
ms.author: mlearned
ms.openlocfilehash: 85f0a42cdfcbea2223d202a9dc35f58746580e85
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350124"
---
# <a name="access-the-kubernetes-web-dashboard-in-azure-kubernetes-service-aks"></a>Accéder au tableau de bord web Kubernetes dans Azure Kubernetes Service (AKS)

Kubernetes comprend un tableau de bord web qui peut être utilisé pour les opérations de gestion de base. Ce tableau de bord vous permet d’afficher l’état de santé de base et les métriques concernant vos applications, ainsi que de créer et déployer des services et de modifier les applications existantes. Cet article vous montre comment accéder au tableau de bord Kubernetes à l’aide de l’interface CLI Azure, et fournit des instructions pour certaines opérations de base.

Pour plus d’informations sur le tableau de bord Kubernetes, consultez la section [Tableau de bord de l’interface utilisateur web de Kubernetes][kubernetes-dashboard]. AKS utilise la version 2.0 ou une version ultérieure du tableau de bord open source.

> [!WARNING]
> **Le module complémentaire du tableau de bord AKS sera bientôt déconseillé. Utilisez plutôt l’[affichage des ressources Kubernetes dans le portail Azure][kubernetes-portal] (préversion).** 
> * Le tableau de bord Kubernetes est activé par défaut pour les clusters exécutant une version de Kubernetes antérieure à 1.18.
> * Le module complémentaire du dashboard est désactivé par défaut pour tous les nouveaux clusters créés sur Kubernetes 1.18 ou une version ultérieure. 
 > * À partir de la préversion de Kubernetes 1.19, AKS ne prend plus en charge l’installation du module complémentaire kube-dashboard géré. 
 > * Les clusters existants avec le module complémentaire activé ne seront pas impactés. Les utilisateurs pourront continuer à installer manuellement le tableau de bord open source en tant que logiciel installé par l’utilisateur.

## <a name="before-you-begin"></a>Avant de commencer

Les étapes détaillées dans ce document supposent que vous avez créé un cluster AKS et que vous avez établi une connexion `kubectl` avec le cluster. Si vous devez créer un cluster AKS, consultez [Démarrage rapide : Déployer un cluster Azure Kubernetes Service à l’aide d’Azure CLI][aks-quickstart].

Azure CLI version 2.6.0 ou ultérieure doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="disable-the-kubernetes-dashboard"></a>Désactiver le tableau de bord Kubernetes

Le module complémentaire kube-dashboard est **activé par défaut sur les clusters antérieurs à K8s 1.18**. Le module complémentaire peut être désactivé en exécutant la commande suivante.

``` azurecli
az aks disable-addons -g myRG -n myAKScluster -a kube-dashboard
```

## <a name="start-the-kubernetes-dashboard"></a>Ouvrir le tableau de bord Kubernetes

> [!WARNING]
> Le module complémentaire du tableau de bord AKS sera bientôt déconseillé à partir de la version 1.19. Utilisez plutôt l’[affichage des ressources Kubernetes dans le portail Azure][kubernetes-portal] (préversion). 
> * La commande suivante ouvre l’affichage des ressources du portail Azure à la place du tableau de bord Kubernetes pour la version 1.19 et ultérieures.

Pour ouvrir le tableau de bord Kubernetes sur un cluster, utilisez la commande [az aks browse][az-aks-browse]. Cette commande requiert l’installation sur le cluster du module complémentaire kube-dashboard, qui est inclus par défaut sur les clusters exécutant une version antérieure à Kubernetes 1.18.

L’exemple suivant ouvre le tableau de bord du cluster nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli
az aks browse --resource-group myResourceGroup --name myAKSCluster
```

Cette commande crée un proxy entre votre système de développement et l’API Kubernetes, et ouvre un navigateur web sur le tableau de bord Kubernetes. Si le tableau de bord Kubernetes ne s’ouvre pas dans le navigateur web, copiez et collez l’adresse URL indiquée dans Azure CLI (habituellement `http://127.0.0.1:8001`).

> [!NOTE]
> Si vous ne voyez pas le tableau de bord sous `http://127.0.0.1:8001`, vous pouvez accéder manuellement aux adresses suivantes. Les clusters de la version 1.16 ou des versions ultérieures utilisent https et requièrent un point de terminaison séparé.
> * K8s 1.16 ou version supérieure : `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy`
> * K8s 1.15 et versions inférieures : `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy`

<!--
![The login page of the Kubernetes web dashboard](./media/kubernetes-dashboard/dashboard-login.png)

You have the following options to sign in to your cluster's dashboard:

* A [kubeconfig file][kubeconfig-file]. You can generate a kubeconfig file using [az aks get-credentials][az-aks-get-credentials].
* A token, such as a [service account token][aks-service-accounts] or user token. On [AAD-enabled clusters][aad-cluster], this token would be an AAD token. You can use `kubectl config view` to list the tokens in your kubeconfig file. For more details on creating an AAD token for use with an AKS cluster see [Integrate Azure Active Directory with Azure Kubernetes Service using the Azure CLI][aad-cluster].
* The default dashboard service account, which is used if you click *Skip*.

> [!WARNING]
> Never expose the Kubernetes dashboard publicly, regardless of the authentication method used.
> 
> When setting up authentication for the Kubernetes dashboard, it is recommended that you use a token over the default dashboard service account. A token allows each user to use their own permissions. Using the default dashboard service account may allow a user to bypass their own permissions and use the service account instead.
> 
> If you do choose to use the default dashboard service account and your AKS cluster uses Kubernetes RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays Kubernetes RBAC access errors. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
>
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command as shown in the following example. **This sample binding does not apply any additional authentication components and may lead to insecure use.**
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> You can now access the Kubernetes dashboard in your Kubernetes RBAC-enabled cluster. To start the Kubernetes dashboard, use the [az aks browse][az-aks-browse] command as detailed in the previous step.
>
> If your cluster does not use Kubernetes RBAC, it is not recommended to create a *ClusterRoleBinding*.
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].

After you choose a method to sign in, the Kubernetes dashboard is displayed. If you chose to use *token* or *skip*, the Kubernetes dashboard will use the permissions of the currently logged in user to access the cluster.

> [!IMPORTANT]
> If your AKS cluster uses Kubernetes RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays Kubernetes RBAC access errors. The Kubernetes dashboard does not currently support user-provided credentials to determine the level of access, rather it uses the roles granted to the service account. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
> 
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command. The following example shows how to create a sample binding, however, this sample binding does not apply any additional authentication components and may lead to insecure use. The Kubernetes dashboard is open to anyone with access to the URL. Do not expose the Kubernetes dashboard publicly.
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].
-->

## <a name="sign-in-to-the-dashboard-kubernetes-116"></a>Connectez-vous au tableau de bord (kubernetes 1.16+)

> [!IMPORTANT]
> À partir de la version [v1.10.1 du tableau de bord Kubernetes](https://github.com/kubernetes/dashboard/releases/tag/v1.10.1) ou de Kubernetes v1.16+, le compte de service « kubernetes-dashboard » ne peut plus être utilisé pour récupérer des ressources en raison d’un [correctif de sécurité dans cette version](https://github.com/kubernetes/dashboard/pull/3400). Par conséquent, les requêtes sans informations d’authentification retournent une [erreur 401 Non autorisé](https://github.com/Azure/AKS/issues/1573#issuecomment-703040998). Un jeton de porteur récupéré à partir d’un compte de service peut toujours être utilisé comme dans cet [exemple de tableau de bord Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui), mais cela a un impact sur le déroulement de la connexion du module additionnel du tableau de bord par rapport aux versions antérieures.
>
>Si vous continuez à exécuter une version antérieure à 1.16, vous pouvez toujours donner des autorisations au compte de service « kubernetes-dashboard », mais cela n’est **pas recommandé** :
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```

L’écran initial présenté requiert un fichier kubeconfig ou un jeton. Les deux options requièrent des autorisations d’accès aux ressources pour afficher ces ressources dans le tableau de bord.

![écran de connexion](./media/kubernetes-dashboard/login.png)

**Utiliser un fichier kubeconfig**

Pour les clusters activés par Azure AD et non activés par Azure AD, un fichier kubeconfig peut être transmis. Vérifiez que les jetons d’accès sont valides. Si vos jetons ont expiré, vous pouvez les actualiser via kubectl.

1. Définir le fichier kubeconfig d’administration avec `az aks get-credentials -a --resource-group <RG_NAME> --name <CLUSTER_NAME>`
1. Sélectionnez `Kubeconfig`, puis cliquez sur `Choose kubeconfig file` pour ouvrir le sélecteur de fichiers
1. Sélectionnez votre fichier kubeconfig ($HOME/.Kube/config par défaut)
1. Cliquez sur `Sign In`

**Utilisation d’un jeton**

1. Pour un **cluster non Azure AD activé**, exécutez `kubectl config view` et copiez le jeton associé au compte d’utilisateur de votre cluster.
1. Collez dans l’option du jeton lors de la connexion.    
1. Cliquez sur `Sign In`

Pour les clusters Azure AD activés, récupérez votre jeton AAD avec la commande suivante. Vérifiez que vous avez remplacé le groupe de ressources et le nom du cluster dans la commande.

```
## Update <RESOURCE_GROUP and <AKS_NAME> with your input.

kubectl config view -o jsonpath='{.users[?(@.name == "clusterUser_<RESOURCE GROUP>_<AKS_NAME>")].user.auth-provider.config.access-token}'
```

Une fois l’opération réussie, une page similaire à celle ci-dessous s’affiche.

![Page de présentation du tableau de bord web Kubernetes](./media/kubernetes-dashboard/dashboard-overview.png)

## <a name="create-an-application"></a>Créer une application

Les étapes suivantes requièrent que l’utilisateur dispose d’autorisations d’accès aux ressources respectives. 

Pour voir comment le tableau de bord Kubernetes peut réduire la complexité des tâches de gestion, nous allons créer une application. Vous pouvez créer une application à partir du tableau de bord Kubernetes en fournissant le texte d’entrée (un fichier YAML) ou en vous aidant d’un assistant graphique.

Pour créer une application, procédez comme suit :

1. Sélectionnez le bouton **Créer** dans l’angle supérieur droit.
1. Pour utiliser l’assistant graphique, choisissez **Create an app** (Créer une application).
1. Donnez un nom au déploiement, par exemple *nginx*.
1. Entrez le nom de l’image de conteneur à utiliser, par exemple *nginx:1.15.5*.
1. Créez un service Kubernetes pour exposer le port 80 réservé au trafic web. Sous **Service**, sélectionnez **Externe**, puis entrez **80** pour le port et le port cible.
1. Lorsque vous êtes prêt, sélectionnez **Déployer** pour créer l’application.

![Déployer une application dans le tableau de bord web Kubernetes](./media/kubernetes-dashboard/create-app.png)

Une à deux minutes sont nécessaires pour assigner une adresse IP externe publique au service Kubernetes. Sur la gauche, sous **Discovery and Load Balancing** (Découverte et équilibrage de charge), sélectionnez **Services**. Le service de votre application est répertorié, y compris les *points de terminaison externes*, comme illustré dans l’exemple suivant :

![Afficher la liste des services et des points de terminaison](./media/kubernetes-dashboard/view-services.png)

Sélectionnez l’adresse du point de terminaison pour ouvrir une fenêtre de navigateur web vers la page NGINX par défaut :

![Afficher la page NGINX par défaut de l’application déployée](./media/kubernetes-dashboard/default-nginx.png)

## <a name="view-pod-information"></a>Afficher les informations de pod

Le tableau de bord Kubernetes peut fournir des métriques de supervision de base et des informations de dépannage telles que des journaux d’activité.

Pour plus d’informations sur vos pods d’application, sélectionnez **Pods** dans le menu de gauche. Une liste des pods disponibles s’affiche. Choisissez votre pod *nginx* pour afficher des informations, par exemple la consommation des ressources :

![Afficher les informations de pod](./media/kubernetes-dashboard/view-pod-info.png)

## <a name="edit-the-application"></a>Modifier l’application

En plus de la création et de l’affichage des applications, le tableau de bord Kubernetes permet de modifier et de mettre à jour des déploiements d’applications. Pour assurer une redondance supplémentaire à l’application, nous allons augmenter le nombre de réplicas NGINX.

Pour modifier un déploiement :

1. Sélectionnez **Déploiements** dans le menu de gauche, puis choisissez votre déploiement *nginx*.
1. Sélectionnez **Modifier** dans la barre de navigation en haut à droite.
1. Recherchez la valeur `spec.replica` vers la ligne 20. Pour augmenter le nombre de réplicas de l’application, modifiez cette valeur. Elle doit être comprise entre *1* et *3*.
1. Lorsque vous êtes prêt, sélectionnez **Mettre à jour**.

![Modifier le déploiement pour mettre à jour le nombre de réplicas](./media/kubernetes-dashboard/edit-deployment.png)

Il faut un certain temps pour créer les nouveaux pods à l’intérieur d’un jeu de réplicas. Dans le menu de gauche, choisissez **Replica Sets** (Jeux de réplicas), puis votre jeu de réplicas *nginx*. La liste des pods reflète maintenant le nombre de réplicas mis à jour, comme indiqué dans la sortie fournie à titre d’exemple :

![Afficher les informations sur le jeu de réplicas](./media/kubernetes-dashboard/view-replica-set.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le tableau de bord Kubernetes, consultez le [tableau de bord de l’interface utilisateur web de Kubernetes][kubernetes-dashboard].

<!-- LINKS - external -->
[dashboard-authentication]: https://github.com/kubernetes/dashboard/wiki/Access-control
[kubeconfig-file]: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
[kubectl-create-clusterrolebinding]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-clusterrolebinding-em-
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubernetes-dashboard]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

<!-- LINKS - internal -->
[aad-cluster]: ./azure-ad-integration-cli.md
[aks-quickstart]: ./kubernetes-walkthrough.md
[aks-service-accounts]: ./concepts-identity.md#kubernetes-service-accounts
[az-account-get-access-token]: /cli/azure/account?view=azure-cli-latest#az-account-get-access-token
[az-aks-browse]: /cli/azure/aks#az-aks-browse
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[kubernetes-portal]: ./kubernetes-portal.md
