---
title: Utiliser les GPU sur Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser des GPU pour le calcul haute performance ou les charges de travail nécessitant beaucoup de ressources graphiques sur Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/21/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: d7e312f049acc0b74aa0a253864bfce6100044bd
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929138"
---
# <a name="use-gpus-for-compute-intensive-workloads-on-azure-kubernetes-service-aks"></a>Utiliser des GPU pour les charges de travail nécessitant beaucoup de ressources système sur Azure Kubernetes Service (AKS)

Les processeurs graphiques (GPU) sont souvent utilisés pour les charges de travail nécessitant beaucoup de ressources système, comme le traitement graphique et la visualisation. AKS prend en charge la création de pools de nœuds compatibles GPU pour exécuter ce type de charges de travail dans Kubernetes. Pour plus d’informations sur les machines virtuelles compatibles GPU disponibles, voir [Tailles de machines virtuelles optimisées pour les GPU dans Azure][gpu-skus]. Pour les nœuds AKS, nous recommandons la taille minimale *Standard_NC6*.

> [!NOTE]
> Les machines virtuelles compatibles GPU contiennent du matériel spécialisé, plus cher et dépendant de la disponibilité régionale. Pour plus d’informations, voir l’outil de [tarification][azure-pricing] et la [disponibilité régionale][azure-availability].

Actuellement, vous ne pouvez utiliser de pools de nœuds compatibles GPU que sous Linux.

## <a name="before-you-begin"></a>Avant de commencer

Cet article part du principe que vous disposez déjà d’un cluster AKS qui comporte des nœuds prenant en charge les GPU. Votre cluster AKS doit exécuter Kubernetes 1.10 ou une version ultérieure. Si vous avez besoin d’un cluster AKS qui réponde à ces exigences, consultez la première section de cet article, [Créer un cluster AKS](#create-an-aks-cluster).

Le logiciel Azure CLI version 2.0.64 ou ultérieure doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

Si vous avez besoin d’un cluster AKS qui remplisse les exigences minimales (nœud compatible GPU et Kubernetes 1.10 ou version ultérieure), suivez les étapes ci-dessous. Si vous avez déjà un cluster AKS répondant à ces exigences, [passez à la section suivante](#confirm-that-gpus-are-schedulable).

Tout d’abord, créez un groupe de ressources pour le cluster avec la commande [az group create][az-group-create]. L’exemple suivant permet de créer un groupe de ressources nommé *myResourceGroup* dans la région *USA Est* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez maintenant un cluster AKS avec la commande [az aks create][az-aks-create]. L’exemple suivant crée un cluster avec un seul nœud présentant la taille `Standard_NC6` :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-vm-size Standard_NC6 \
    --node-count 1
```

Récupérez les informations d’identification de votre cluster AKS avec la commande [az aks get-credentials][az-aks-get-credentials] :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="install-nvidia-device-plugin"></a>Installer le plug-in d’appareil NVIDIA

Avant de pouvoir utiliser les GPU dans les nœuds, vous devez déployer un DaemonSet pour le plug-in d’appareil NVIDIA. Ce DaemonSet exécute un pod sur chaque nœud de façon à fournir les pilotes requis pour les GPU.

Commencez par créer un espace de noms avec la commande [kubectl create namespace][kubectl-create], par exemple, *gpu-ressources* :

```console
kubectl create namespace gpu-resources
```

Créez un fichier nommé *nvidia-device-plugin-ds.yaml* et collez le manifeste YAML suivant. Ce manifeste est fourni dans le cadre du [plug-in d’appareil NVIDIA pour un projet Kubernetes][nvidia-github].

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: gpu-resources
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: mcr.microsoft.com/oss/nvidia/k8s-device-plugin:1.11
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
```

Ensuite, utilisez la commande [kubectl apply][kubectl-apply] pour créer un DaemonSet et confirmez la création du plug-in d’appareil NVIDIA, comme indiqué dans l’exemple de sortie suivant :

```console
$ kubectl apply -f nvidia-device-plugin-ds.yaml

daemonset "nvidia-device-plugin" created
```

## <a name="use-the-aks-specialized-gpu-image-preview"></a>Utiliser l’image GPU spécialisée AKS (préversion)

En guise d’alternative à ces étapes, AKS fournit une image AKS entièrement configurée qui contient déjà le [plug-in d’appareil NVIDIA pour Kubernetes][nvidia-github].

> [!WARNING]
> Vous ne devez pas installer manuellement le démon du plug-in d’appareil NVIDIA défini pour les clusters à l’aide de la nouvelle image GPU spécialisée AKS.


Inscrivez la fonctionnalité `GPUDedicatedVHDPreview` :

```azurecli
az feature register --name GPUDedicatedVHDPreview --namespace Microsoft.ContainerService
```

Quelques minutes peuvent être nécessaires pour que l’état **Inscrit** s’affiche. Vous pouvez vérifier l’état de l’inscription à l’aide de la commande [az feature list](/cli/azure/feature#az-feature-list) :

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/GPUDedicatedVHDPreview')].{Name:name,State:properties.state}"
```

Quand l’état indique Inscrit, actualisez l’inscription du fournisseur de ressources `Microsoft.ContainerService` à l’aide de la commande [az provider register](/cli/azure/provider#az-provider-register) :

```azurecli
az provider register --namespace Microsoft.ContainerService
```

Pour installer l’extension CLI aks-preview, utilisez les commandes Azure CLI suivantes :

```azurecli
az extension add --name aks-preview
```

Pour mettre à jour l’extension CLI aks-preview, utilisez les commandes Azure CLI suivantes :

```azurecli
az extension update --name aks-preview
```

### <a name="use-the-aks-specialized-gpu-image-on-new-clusters-preview"></a>Utiliser l’image GPU spécialisée AKS sur les nouveaux clusters (préversion)    

Configurez le cluster pour qu’il utilise l’image GPU spécialisée AKS lors de sa création. Utilisez l’indicateur `--aks-custom-headers` pour les nœuds de l’agent GPU sur votre nouveau cluster de manière à utiliser l’image GPU spécialisée AKS.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Si vous souhaitez créer un cluster en utilisant des images AKS classiques, vous pouvez omettre la balise `--aks-custom-headers` personnalisée. Vous pouvez également choisir d’ajouter des pools de nœuds GPU spécialisés comme indiqué ci-dessous.


### <a name="use-the-aks-specialized-gpu-image-on-existing-clusters-preview"></a>Utiliser l’image GPU spécialisée AKS sur les clusters existants (préversion)

Configurez un nouveau pool de nœuds pour utiliser l’image GPU spécialisée AKS. Utilisez l’indicateur `--aks-custom-headers` pour les nœuds de l’agent GPU sur votre nouveau pool de nœuds de manière à utiliser l’image GPU spécialisée AKS.

```azurecli
az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Si vous souhaitez créer un pool de nœuds à l’aide d’images AKS classiques, vous pouvez omettre la balise `--aks-custom-headers` personnalisée. 

> [!NOTE]
> Si votre référence SKU GPU requiert des machines virtuelles de 2e génération, vous pouvez créer les éléments suivants :
> ```azurecli
> az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6s_v2 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true,usegen2vm=true
> ```

## <a name="confirm-that-gpus-are-schedulable"></a>Vérifier que les GPU sont planifiables

Maintenant que votre cluster AKS est créé, vérifiez que les GPU sont planifiables dans Kubernetes. Tout d’abord, listez les nœuds de votre cluster avec la commande [kubectl get nodes][kubectl-get] :

```console
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-28993262-0   Ready    agent   13m   v1.12.7
```

Utilisez maintenant la commande [kubectl describe node][kubectl-describe] pour vérifier que les GPU sont planifiables. Sous la section *Capacité*, le GPU doit apparaître ainsi : `nvidia.com/gpu:  1`.

L’exemple condensé suivant montre qu’un GPU est disponible sur le nœud nommé *aks-nodepool1-18821093-0* :

```console
$ kubectl describe node aks-nodepool1-28993262-0

Name:               aks-nodepool1-28993262-0
Roles:              agent
Labels:             accelerator=nvidia

[...]

Capacity:
 attachable-volumes-azure-disk:  24
 cpu:                            6
 ephemeral-storage:              101584140Ki
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         57713784Ki
 nvidia.com/gpu:                 1
 pods:                           110
Allocatable:
 attachable-volumes-azure-disk:  24
 cpu:                            5916m
 ephemeral-storage:              93619943269
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         51702904Ki
 nvidia.com/gpu:                 1
 pods:                           110
System Info:
 Machine ID:                 b0cd6fb49ffe4900b56ac8df2eaa0376
 System UUID:                486A1C08-C459-6F43-AD6B-E9CD0F8AEC17
 Boot ID:                    f134525f-385d-4b4e-89b8-989f3abb490b
 Kernel Version:             4.15.0-1040-azure
 OS Image:                   Ubuntu 16.04.6 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://1.13.1
 Kubelet Version:            v1.12.7
 Kube-Proxy Version:         v1.12.7
PodCIDR:                     10.244.0.0/24
ProviderID:                  azure:///subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Compute/virtualMachines/aks-nodepool1-28993262-0
Non-terminated Pods:         (9 in total)
  Namespace                  Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                     ------------  ----------  ---------------  -------------  ---
  kube-system                nvidia-device-plugin-daemonset-bbjlq     0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m39s

[...]
```

## <a name="run-a-gpu-enabled-workload"></a>Exécuter une charge de travail compatible GPU

Pour voir fonctionner le GPU, planifiez une charge de travail compatible GPU avec la demande de ressource correspondante. Cet exemple exécute une tâche [Tensorflow](https://www.tensorflow.org/) sur le [jeu de données MNIST](http://yann.lecun.com/exdb/mnist/).

Créez un fichier nommé *samples-tf-mnist-demo.yaml* et collez le manifeste YAML suivant. Le manifeste de travail suivant comporte la limite de ressource `nvidia.com/gpu: 1` :

> [!NOTE]
> Si vous recevez une erreur d’incompatibilité de version lorsque vous appelez des pilotes (par exemple, la version du pilote CUDA est insuffisante pour la version du runtime CUDA), consultez la matrice de compatibilité des pilotes NVIDIA à l’adresse [https://docs.nvidia.com/deploy/cuda-compatibility/index.html](https://docs.nvidia.com/deploy/cuda-compatibility/index.html).

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: samples-tf-mnist-demo
  name: samples-tf-mnist-demo
spec:
  template:
    metadata:
      labels:
        app: samples-tf-mnist-demo
    spec:
      containers:
      - name: samples-tf-mnist-demo
        image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
        args: ["--max_steps", "500"]
        imagePullPolicy: IfNotPresent
        resources:
          limits:
           nvidia.com/gpu: 1
      restartPolicy: OnFailure
```

Utilisez la commande [kubectl apply][kubectl-apply] pour exécuter le travail. Cette commande analyse le fichier manifeste et crée les objets Kubernetes définis :

```console
kubectl apply -f samples-tf-mnist-demo.yaml
```

## <a name="view-the-status-and-output-of-the-gpu-enabled-workload"></a>Afficher l’état et la sortie de la charge de travail compatible GPU

Surveillez la progression de la tâche avec la commande [kubectl get jobs][kubectl-get] et l’argument `--watch`. L’extraction de l’image, puis le traitement du jeu de données peuvent prendre quelques minutes. Lorsque la colonne *COMPLETIONS* affiche *1/1*, la tâche est terminée. Quitter la commande `kubetctl --watch` avec *Ctrl-C* :

```console
$ kubectl get jobs samples-tf-mnist-demo --watch

NAME                    COMPLETIONS   DURATION   AGE

samples-tf-mnist-demo   0/1           3m29s      3m29s
samples-tf-mnist-demo   1/1   3m10s   3m36s
```

Pour voir la sortie de la charge de travail compatible GPU, commencez par récupérer le nom des pods avec la commande [kubectl get pods][kubectl-get] :

```console
$ kubectl get pods --selector app=samples-tf-mnist-demo

NAME                          READY   STATUS      RESTARTS   AGE
samples-tf-mnist-demo-mtd44   0/1     Completed   0          4m39s
```

Utilisez maintenant la commande [kubectl logs][kubectl-logs] pour afficher les journaux d’activité de pods. Les journaux d’activité de pods de l’exemple suivant confirment que l’appareil GPU détecté est le bon (`Tesla K80`). Indiquez le nom de votre propre pod :

```console
$ kubectl logs samples-tf-mnist-demo-smnr6

2019-05-16 16:08:31.258328: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2019-05-16 16:08:31.396846: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties: 
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: 2fd7:00:00.0
totalMemory: 11.17GiB freeMemory: 11.10GiB
2019-05-16 16:08:31.396886: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: 2fd7:00:00.0, compute capability: 3.7)
2019-05-16 16:08:36.076962: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.1081
Accuracy at step 10: 0.7457
Accuracy at step 20: 0.8233
Accuracy at step 30: 0.8644
Accuracy at step 40: 0.8848
Accuracy at step 50: 0.8889
Accuracy at step 60: 0.8898
Accuracy at step 70: 0.8979
Accuracy at step 80: 0.9087
Accuracy at step 90: 0.9099
Adding run metadata for 99
Accuracy at step 100: 0.9125
Accuracy at step 110: 0.9184
Accuracy at step 120: 0.922
Accuracy at step 130: 0.9161
Accuracy at step 140: 0.9219
Accuracy at step 150: 0.9151
Accuracy at step 160: 0.9199
Accuracy at step 170: 0.9305
Accuracy at step 180: 0.9251
Accuracy at step 190: 0.9258
Adding run metadata for 199
Accuracy at step 200: 0.9315
Accuracy at step 210: 0.9361
Accuracy at step 220: 0.9357
Accuracy at step 230: 0.9392
Accuracy at step 240: 0.9387
Accuracy at step 250: 0.9401
Accuracy at step 260: 0.9398
Accuracy at step 270: 0.9407
Accuracy at step 280: 0.9434
Accuracy at step 290: 0.9447
Adding run metadata for 299
Accuracy at step 300: 0.9463
Accuracy at step 310: 0.943
Accuracy at step 320: 0.9439
Accuracy at step 330: 0.943
Accuracy at step 340: 0.9457
Accuracy at step 350: 0.9497
Accuracy at step 360: 0.9481
Accuracy at step 370: 0.9466
Accuracy at step 380: 0.9514
Accuracy at step 390: 0.948
Adding run metadata for 399
Accuracy at step 400: 0.9469
Accuracy at step 410: 0.9489
Accuracy at step 420: 0.9529
Accuracy at step 430: 0.9507
Accuracy at step 440: 0.9504
Accuracy at step 450: 0.951
Accuracy at step 460: 0.9512
Accuracy at step 470: 0.9539
Accuracy at step 480: 0.9533
Accuracy at step 490: 0.9494
Adding run metadata for 499
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les objets Kubernetes associés créés dans cet article, utilisez la commande [delete job][kubectl delete] de la manière suivante :

```console
kubectl delete jobs samples-tf-mnist-demo
```

## <a name="next-steps"></a>Étapes suivantes

Pour exécuter des tâches Apache Spark, voir [Exécuter des tâches Apache Spark sur AKS][aks-spark].

Pour plus d’informations sur les charges de travail Machine Learning (ML) sur Kubernetes, voir [Kubeflow Labs][kubeflow-labs].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeflow-labs]: https://github.com/Azure/kubeflow-labs
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[azure-pricing]: https://azure.microsoft.com/pricing/
[azure-availability]: https://azure.microsoft.com/global-infrastructure/services/
[nvidia-github]: https://github.com/NVIDIA/k8s-device-plugin

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-spark]: spark-job.md
[gpu-skus]: ../virtual-machines/sizes-gpu.md
[install-azure-cli]: /cli/azure/install-azure-cli
