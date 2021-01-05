---
title: 'Démarrage rapide : Déployer un cluster AKS (Azure Kubernetes Service) avec des nœuds d’informatique confidentielle à l’aide d’Azure CLI'
description: Apprenez à créer un cluster AKS avec des nœuds d’informatique confidentielle et à déployer une application Hello World en utilisant Azure CLI.
author: agowdamsft
ms.service: container-service
ms.topic: quickstart
ms.date: 9/22/2020
ms.author: amgowda
ms.openlocfilehash: 95626836afb09ada286cf7e171f97db450167999
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94564342"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-with-confidential-computing-nodes-using-azure-cli-preview"></a>Démarrage rapide : Déployer un cluster AKS (Azure Kubernetes Service) avec des nœuds d’informatique confidentielle à l’aide d’Azure CLI (préversion)

Ce guide de démarrage rapide est destiné aux développeurs ou aux opérateurs de cluster qui souhaitent créer rapidement un cluster AKS et déployer une application de surveillance des applications en utilisant le service Kubernetes managé dans Azure.

## <a name="overview"></a>Vue d’ensemble

Dans ce guide de démarrage rapide, vous allez apprendre à déployer un cluster AKS (Azure Kubernetes Service) avec des nœuds d’informatique confidentielle à l’aide d’Azure CLI, et à exécuter une application Hello World dans une enclave. AKS est un service Kubernetes géré qui vous permet de déployer et de gérer rapidement des clusters. Vous trouverez [ici](../aks/intro-kubernetes.md) plus d’informations sur AKS.

> [!NOTE]
> Les machines virtuelles DCsv2 pour l’informatique confidentielle utilisent du matériel spécialisé plus cher et dépendant de la disponibilité régionale. Pour plus d’informations sur les [références SKU disponibles et les régions prises en charge](virtual-machine-solutions.md), consultez la page consacrée aux machines virtuelles.

### <a name="deployment-pre-requisites"></a>Conditions préalables au déploiement

1. Avoir un abonnement Azure actif. Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer
1. Avoir la version 2.0.64 ou ultérieure d’Azure CLI installée et configurée sur votre machine de déploiement (exécutez`az --version` pour connaître la version). Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](../container-registry/container-registry-get-started-azure-cli.md).
1. Avoir l’[extension aks-preview](https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview) version 0.4.62 ou ultérieure 
1. Avoir au moins six cœurs **DC<x>s-v2** disponibles dans votre abonnement. Par défaut, le quota de cœurs de machine virtuelle pour l’informatique confidentielle est de 8 cœurs par abonnement Azure. Si vous envisagez de provisionner un cluster qui nécessite plus de 8 cœurs, suivez [ces instructions](../azure-portal/supportability/per-vm-quota-requests.md) pour créer un ticket de demande d’augmentation du quota

### <a name="confidential-computing-node-features-dcxs-v2"></a>Fonctionnalités des nœuds d’informatique confidentielle (DC<x>s-v2)

1. Nœuds Worker Linux prenant en charge les conteneurs Linux uniquement
1. Machines virtuelles Ubuntu 18.04 de deuxième génération
1. Processeur Intel SGX avec mémoire EPC (Encrypted Page Cache). En savoir plus [ici](./faq.md)
1. Kubernetes version 1.16+
1. Pilote Intel SGX DCAP préinstallé. En savoir plus [ici](./faq.md)
1. Déploiement basé sur la CLI pendant la préversion


## <a name="installing-the-cli-pre-requisites"></a>Installation des prérequis de la CLI

Pour installer l’extension aks-preview 0.4.62 ou version ultérieure, utilisez les commandes Azure CLI suivantes :

```azurecli-interactive
az extension add --name aks-preview
az extension list
```
Pour mettre à jour l’extension CLI aks-preview, utilisez les commandes Azure CLI suivantes :

```azurecli-interactive
az extension update --name aks-preview
```

Inscrire Gen2VMPreview :

```azurecli-interactive
az feature register --name Gen2VMPreview --namespace Microsoft.ContainerService
```
Quelques minutes peuvent être nécessaires pour que l’état Inscrit s’affiche. Vous pouvez vérifier l’état de l’inscription en utilisant la commande « az feature list » :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/Gen2VMPreview')].{Name:name,State:properties.state}"
```
Une fois que l’état affiche « Inscrit », actualisez l’inscription du fournisseur de ressources Microsoft.ContainerService à l’aide de la commande « az provider register » :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="creating-an-aks-cluster"></a>Création d’un cluster AKS

Si vous disposez déjà d’un cluster AKS qui remplit les exigences ci-dessus, [passez à la section avec cluster existant](#existing-cluster) pour ajouter un nouveau pool de nœuds d’informatique confidentielle.

Tout d’abord, créez un groupe de ressources pour le cluster avec la commande az group create. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* dans la région *westus2* :

```azurecli-interactive
az group create --name myResourceGroup --location westus2
```

Créez maintenant un cluster AKS avec la commande az aks create. L’exemple suivant crée un cluster contenant un nœud unique de taille `Standard_DC2s_v2`. Vous pouvez choisir une autre liste de références SKU DCsv2 prises en charge [ici](../virtual-machines/dcv2-series.md) :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-vm-size Standard_DC2s_v2 \
    --node-count 3 \
    --enable-addon confcom \
    --network-plugin azure \
    --vm-set-type VirtualMachineScaleSets \
    --aks-custom-headers usegen2vm=true
```
La commande ci-dessus doit provisionner un nouveau cluster AKS avec des pools de nœuds **DC<x>s-v2** et installer automatiquement deux ensembles de démons ([SGX Device Plugin](confidential-nodes-aks-overview.md#sgx-plugin) & [SGX Quote Helper](confidential-nodes-aks-overview.md#sgx-quote)).

Récupérez les informations d’identification de votre cluster AKS avec la commande az aks get-credentials :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```
Vérifiez que les nœuds sont correctement créés et que les ensembles de démons SGX s’exécutent sur des pools de nœuds **DC<x>s-v2**, en utilisant la commande kubectl get pods et nodes comme indiqué ci-dessous :

```console
$ kubectl get pods --all-namespaces

output
kube-system     sgx-device-plugin-xxxx     1/1     Running
```
Si vous obtenez la sortie ci-dessus, votre cluster AKS est maintenant prêt à exécuter des applications confidentielles.

Passez à la section de déploiement de l’application [Hello World à partir de l’enclave](#hello-world) pour tester une application dans une enclave. Ou alors, suivez les instructions ci-dessous pour ajouter des pools de nœuds supplémentaires sur AKS (AKS prend en charge la combinaison de pools de nœuds SGX et de pools de nœuds non-SGX)

>Si les ensembles de démons SGX ne sont pas installés sur vos pools de nœuds DCSv2, exécutez la commande ci-dessous.

```azurecli-interactive
az aks update --enable-addons confcom --resource-group myResourceGroup --name myAKSCluster
```

![Création d’un cluster AKS DCSv2](./media/confidential-nodes-aks-overview/CLIAKSProvisioning.gif)

## <a name="adding-confidential-computing-node-to-existing-aks-cluster"></a>Ajout d’un nœud d’informatique confidentielle à un cluster AKS existant<a id="existing-cluster"></a>

Cette section suppose que vous disposez déjà d’un cluster AKS qui répond aux critères indiqués dans la section des conditions préalables.

Tout d’abord, activez les modules complémentaires AKS pour l’informatique confidentielle sur le cluster existant :

```azurecli-interactive
az aks enable-addons --addons confcom --name MyManagedCluster --resource-group MyResourceGroup 
```
Ajoutez maintenant un pool de nœuds **DC<x>s-v2** au cluster.
    
> [!NOTE]
> Pour utiliser la fonctionnalité d’informatique confidentielle, il faut que votre cluster AKS dispose au minimum d’un pool de nœuds basés sur la référence SKU de machine virtuelle **DC<x>s-v2**. Pour en savoir plus sur les référence SKU de machine virtuelle DCsv2 d’informatique confidentielle, consultez [Références SKU disponibles et régions prises en charge](virtual-machine-solutions.md).
    
  ```azurecli-interactive
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-count 1 --node-vm-size Standard_DC4s_v2 --aks-custom-headers usegen2vm=true

output node pool added

Verify

az aks nodepool list --cluster-name myAKSCluster --resource-group myResourceGroup
```

```console
kubectl get nodes
```
La sortie affiche normalement le pool confcompool1 qui vient d’être ajouté au cluster AKS.

```console
$ kubectl get pods --all-namespaces

output (you may also see other daemonsets along SGX daemonsets as below)
kube-system     sgx-device-plugin-xxxx     1/1     Running
kube-system     sgx-quote-helper-xxxx      1/1     Running
```
Si vous obtenez la sortie ci-dessus, votre cluster AKS est maintenant prêt à exécuter des applications confidentielles.

## <a name="hello-world-from-isolated-enclave-application"></a>Hello World de l’application isolée en enclave <a id="hello-world"></a>
Créez un fichier nommé *hello-world-enclave.yaml* et collez le manifeste YAML suivant. Cet exemple de code d’application Open Enclave est disponible dans le [projet Open Enclave](https://github.com/openenclave/openenclave/tree/master/samples/helloworld).

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sgx-test
  labels:
    app: sgx-test
spec:
  template:
    metadata:
      labels:
        app: sgx-test
    spec:
      containers:
      - name: sgxtest
        image: oeciteam/sgx-test:1.0
        resources:
          limits:
            kubernetes.azure.com/sgx_epc_mem_in_MiB: 5 # This limit will automatically place the job into confidential computing node. Alternatively you can target deployment to nodepools
      restartPolicy: Never
  backoffLimit: 0
  ```

À présent, utilisez la commande kubectl apply pour créer un exemple de travail qui s’exécutera dans une enclave sécurisée, comme dans l’exemple de sortie suivant :

```console
$ kubectl apply -f hello-world-enclave.yaml

job "sgx-test" created
```

Vous pouvez vérifier que la charge de travail a correctement créé un environnement TEE (l’enclave) en exécutant les commandes suivantes :

```console
$ kubectl get jobs -l app=sgx-test
```

```console
$ kubectl get jobs -l app=sgx-test
NAME       COMPLETIONS   DURATION   AGE
sgx-test   1/1           1s         23s
```

```console
$ kubectl get pods -l app=sgx-test
```

```console
$ kubectl get pods -l app=sgx-test
NAME             READY   STATUS      RESTARTS   AGE
sgx-test-rchvg   0/1     Completed   0          25s
```

```console
$ kubectl logs -l app=sgx-test
```

```console
$ kubectl logs -l app=sgx-test
Hello world from the enclave
Enclave called into host to print: Hello World!
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les pools de nœuds associés ou supprimer le cluster AKS, utilisez les commandes ci-dessous :

Suppression du cluster AKS
``````azurecli-interactive
az aks delete --resource-group myResourceGroup --name myAKSCluster
```
Removing the confidential computing node pool

``````azurecli-interactive
az aks nodepool delete --cluster-name myAKSCluster --name myNodePoolName --resource-group myResourceGroup
``````

## <a name="next-steps"></a>Étapes suivantes

Exécutez des applications Python, Node, etc. de manière confidentielle dans des conteneurs confidentiels en utilisant ces [exemples de conteneurs confidentiels](https://github.com/Azure-Samples/confidential-container-samples).

Exécutez des applications d’enclave en utilisant ces [exemples de conteneur Azure reconnaissant l’enclave](https://github.com/Azure-Samples/confidential-computing/blob/main/containersamples/).