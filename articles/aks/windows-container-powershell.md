---
title: Création d’un conteneur Windows Server sur un cluster AKS à l’aide de PowerShell
description: Découvrez comment créer rapidement un cluster Kubernetes et déployer une application dans un conteneur Windows Server dans Azure Kubernetes Service (AKS) à l’aide de PowerShell.
services: container-service
ms.topic: article
ms.date: 05/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bf446c858e40014a4085721d646f819e08542064
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87497883"
---
# <a name="create-a-windows-server-container-on-an-azure-kubernetes-service-aks-cluster-using-powershell"></a>Création d’un conteneur Windows Server sur un cluster Azure Kubernetes Service (AKS) à l’aide de PowerShell

AKS (Azure Kubernetes Service) est un service Kubernetes managé qui vous permet de déployer et de gérer rapidement des clusters. Dans cet article, vous allez déployer un cluster AKS à l’aide de PowerShell, ainsi qu’un exemple d’application `ASP.NET` dans un conteneur Windows Server sur le cluster.

![Image de la navigation vers l’exemple d’application ASP.NET](media/windows-container-powershell/asp-net-sample-app.png)

Cet article suppose une compréhension élémentaire des concepts liés à Kubernetes. Pour plus d’informations, consultez [Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)][kubernetes-concepts].

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell][install-azure-powershell].

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un ID d’abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez les clusters AKS prenant en charge plusieurs pools de nœuds :

* Vous ne pouvez pas supprimer le premier pool de nœuds.

Les limitations supplémentaires suivantes s’appliquent aux pools de nœuds Windows Server :

* Le cluster AKS peut comprendre au maximum 10 pools de nœuds.
* Le cluster AKS peut comprendre au maximum 100 nœuds dans chaque pool de nœuds.
* Le nom de pool de nœud Windows Server a une limite de 6 caractères.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Lorsque vous créez un groupe de ressources, vous devez spécifier un emplacement. Il s’agit de l’emplacement de stockage des métadonnées de groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création de ressources. Créez un groupe de ressources avec la cmdlet [New-AzResourceGroup][new-azresourcegroup].

L’exemple suivant crée un groupe de ressources nommé **myResourceGroup** à l’emplacement **eastus**.

> [!NOTE]
> Cet article utilise la syntaxe PowerShell pour les commandes de ce tutoriel. Si vous utilisez Azure Cloud Shell, vérifiez que la liste déroulante en haut à gauche de la fenêtre Cloud Shell est définie sur **PowerShell**.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location eastus
```

L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

```plaintext
ResourceGroupName : myResourceGroup
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup
```

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

Utilisez l’utilitaire de ligne de commande `ssh-keygen` pour générer une paire de clés SSH. Pour plus d’informations, consultez [Étapes rapides : Création et utilisation d’une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

Pour pouvoir exécuter un cluster AKS qui prend en charge des pools de nœuds pour les conteneurs Windows Server, votre cluster doit appliquer une stratégie de réseau qui utilise un plug-in réseau [Azure CNI][azure-cni-about] (avancé). Pour plus d’informations sur la planification des plages de sous-réseau nécessaires et les considérations réseau, consultez [Configurer le réseau Azure CNI][use-advanced-networking]. Utilisez la commande [New-AzAks][new-azaks] ci-dessous pour créer un cluster AKS nommé **myAKSCluster**. L’exemple suivant crée les ressources réseau nécessaires si elles n’existent pas.

> [!NOTE]
> Pour garantir un fonctionnement fiable de votre cluster, vous devez exécuter au moins 2 (deux) nœuds dans le pool de nœuds par défaut.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzAKS -ResourceGroupName myResourceGroup -Name myAKSCluster -NodeCount 2 -KubernetesVersion 1.16.7 -NetworkPlugin azure -NodeVmSetType VirtualMachineScaleSets -WindowsProfileAdminUserName akswinuser -WindowsProfileAdminUserPassword $Password
```

> [!Note]
> Si vous ne parvenez pas à créer le cluster AKS parce que la version n’est pas prise en charge dans cette région, vous pouvez utiliser la commande `Get-AzAksVersion -Location eastus` pour trouver la liste des versions prises en charge pour cette région.

Au bout de quelques minutes, la commande se termine et retourne des informations sur le cluster. Parfois, le provisionnement du cluster peut prendre plus de quelques minutes. Autorisez jusqu’à 10 minutes dans ces cas de figure.

## <a name="add-a-windows-server-node-pool"></a>Ajouter un pool de nœuds Windows Server

Par défaut, un cluster AKS est créé avec un pool de nœuds qui peut exécuter des conteneurs Linux. Utilisez la cmdlet `New-AzAksNodePool` pour ajouter un pool de nœuds capable d’exécuter des conteneurs Windows Server en même temps que le pool de nœuds Linux.

```azurepowershell-interactive
New-AzAksNodePool -ResourceGroupName myResourceGroup -ClusterName myAKSCluster -OsType Windows -Name npwin -KubernetesVersion 1.16.7
```

La commande ci-dessus crée un pool de nœuds nommé **npwin** et l’ajoute à **myAKSCluster**. Lorsque vous créez un pool de nœuds pour exécuter des conteneurs Windows Server, la valeur par défaut de **VmSize** est **Standard_D2s_v3**. Si vous choisissez de définir le paramètre **VmSize**, consultez la liste des [tailles de machines virtuelles restreintes][restricted-vm-sizes]. La taille minimale recommandée est **Standard_D2s_v3**. La commande précédente utilise également le sous-réseau par défaut dans le réseau virtuel par défaut créé lors de l’exécution de `New-AzAks`.

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Pour gérer un cluster Kubernetes, vous utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes. Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Pour installer `kubectl` localement, utilisez la cmdlet `Install-AzAksKubectl` :

```azurepowershell-interactive
Install-AzAksKubectl
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la cmdlet [Import-AzAksCredential][import-azakscredential]. Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser.

```azurepowershell-interactive
Import-AzAksCredential -ResourceGroupName myResourceGroup -Name myAKSCluster
```

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get][kubectl-get] pour retourner une liste des nœuds du cluster.

```azurepowershell-interactive
kubectl get nodes
```

L’exemple de sortie suivant montre tous les nœuds du cluster. Vérifiez que l’état de tous les nœuds est **Prêt** :

```plaintext
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.16.7
aksnpwin987654                      Ready    agent   108s   v1.16.7
```

## <a name="run-the-application"></a>Exécution de l'application

Un fichier manifeste Kubernetes définit un état souhaité pour le cluster, notamment les images conteneur à exécuter. Dans cet article, un manifeste est utilisé pour créer tous les objets nécessaires pour exécuter l’exemple d’application ASP.NET dans un conteneur Windows Server. Ce manifeste comprend un [déploiement Kubernetes][kubernetes-deployment] pour l’exemple d’application ASP.NET et un [service Kubernetes][kubernetes-service] externe pour accéder à l’application à partir d’Internet.

L’exemple d’application ASP.NET est fourni dans le cadre des [Exemples .NET Framework][dotnet-samples] et s’exécute dans un conteneur Windows Server. AKS exige que les conteneurs Windows Server soient basés sur des images de **Windows Server 2019** ou versions supérieures. Le fichier manifeste Kubernetes doit également définir un [sélecteur de nœud][node-selector] pour indiquer à votre cluster AKS d’exécuter le pod de votre exemple d’application ASP.NET sur un nœud qui peut exécuter des conteneurs Windows Server.

Créez un fichier nommé `sample.yaml`, et copiez-y la définition YAML suivante. Si vous utilisez Azure Cloud Shell, vous pouvez créer ce fichier à l’aide de `vi` ou de `nano` comme si vous travailliez sur un système virtuel ou physique :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
  labels:
    app: sample
spec:
  replicas: 1
  template:
    metadata:
      name: sample
      labels:
        app: sample
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: sample
        image: mcr.microsoft.com/dotnet/framework/samples:aspnetapp
        resources:
          limits:
            cpu: 1
            memory: 800M
          requests:
            cpu: .1
            memory: 300M
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sample
---
apiVersion: v1
kind: Service
metadata:
  name: sample
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sample
```

Déployez l’application à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```azurepowershell-interactive
kubectl apply -f sample.yaml
```

L’exemple de sortie suivant montre que le déploiement et le service ont été créés correctement :

```plaintext
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>Test de l’application

Quand l’application s’exécute, un service Kubernetes expose le front-end de l’application sur Internet.
L’exécution de ce processus peut prendre plusieurs minutes. Parfois, le provisionnement du service peut prendre plus de quelques minutes. Autorisez jusqu’à 10 minutes dans ces cas de figure.

Pour surveiller la progression, utilisez la commande [kubectl get service][kubectl-get] avec l’argument `--watch`.

```azurepowershell-interactive
kubectl get service sample --watch
```

Dans un premier temps, la valeur **EXTERNAL-IP** pour **l’exemple** de service apparaît comme étant **en attente**.

```plaintext
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample             LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Quand l’adresse **EXTERNAL-IP** passe de l’état **pending** à une adresse IP publique réelle, utilisez `CTRL-C` pour arrêter le processus de surveillance `kubectl`. L’exemple de sortie suivant montre une adresse IP publique valide affectée au service :

```plaintext
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Pour voir l’exemple d’application en action, ouvrez un navigateur web en utilisant l’adresse IP externe de votre service.

![Image de la navigation vers l’exemple d’application ASP.NET](media/windows-container-powershell/asp-net-sample-app.png)

> [!Note]
> Si vous dépassez le délai d'attente de la connexion en tentant de charger la page, vérifiez que l’exemple d’application est prêt avec la commande `kubectl get pods --watch` suivante. Parfois, le conteneur Windows n’est pas démarré au moment où votre adresse IP externe est disponible.

## <a name="delete-cluster"></a>Supprimer un cluster

Lorsque vous n’avez plus besoin du cluster, utilisez la cmdlet [Remove-AzResourceGroup][remove-azresourcegroup] pour supprimer le groupe de ressources, le service conteneur et toutes les ressources associées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

> [!NOTE]
> Lorsque vous supprimez le cluster, le principal de service Azure Active Directory utilisé par le cluster AKS n’est pas supprimé. Pour obtenir des instructions sur la façon de supprimer le principal de service, consultez [Considérations et suppression du principal de service AKS][sp-delete]. Si vous avez utilisé une identité managée, l’identité est managée par la plateforme et n’a pas besoin d’être supprimée.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez déployé un cluster Kubernetes et un exemple d’application `ASP.NET` dans un conteneur Windows Server sur ce cluster. [Accédez au tableau de bord web de Kubernetes][kubernetes-dashboard] pour le cluster que vous avez créé.

Pour en savoir plus sur AKS et parcourir le code complet de l’exemple de déploiement, passez au tutoriel sur le cluster Kubernetes.

> [!div class="nextstepaction"]
> [Didacticiel AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[dotnet-samples]: https://hub.docker.com/_/microsoft-dotnet-framework-samples/
[node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[install-azure-powershell]: /powershell/azure/install-az-ps
[new-azresourcegroup]: /powershell/module/az.resources/new-azresourcegroup
[azure-cni-about]: concepts-network.md#azure-cni-advanced-networking
[use-advanced-networking]: configure-azure-cni.md
[new-azaks]: /powershell/module/az.aks/new-azaks
[restricted-vm-sizes]: quotas-skus-regions.md#restricted-vm-sizes
[import-azakscredential]: /powershell/module/az.aks/import-azakscredential
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[kubernetes-dashboard]: kubernetes-dashboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
