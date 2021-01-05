---
title: Déployer une application sans état Kubernetes sur un appareil Azure Stack Edge Pro avec GPU à l’aide de kubectl | Microsoft Docs
description: Décrit comment créer et gérer un déploiement d’application sans état Kubernetes à l’aide de kubectl sur un appareil Microsoft Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 6356089daed02270a14903639afee8001153b195
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447381"
---
# <a name="deploy-a-kubernetes-stateless-application-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Déployer une application sans état Kubernetes via kubectl sur votre appareil Azure Stack Edge Pro avec GPU

Cet article explique comment déployer une application sans état à l’aide des commandes kubectl sur un cluster Kubernetes existant. Cet article vous guide également tout au long du processus de création et de configuration de pods dans votre application sans état.

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir créer un cluster Kubernetes et d’utiliser l’outil de ligne de commande `kubectl`, vous devez vous assurer que :

- Vous avez les informations d’identification de connexion à un appareil Azure Stack Edge Pro à 1 nœud.

- Windows PowerShell 5.0 ou version ultérieure est installé sur un système client Windows pour accéder à l’appareil Azure Stack Edge Pro. Vous pouvez également utiliser un autre client avec un système d’exploitation pris en charge. Cet article décrit la procédure à utiliser avec un client Windows. Pour télécharger la dernière version de Windows PowerShell, accédez à [Installation de Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-7).

- Le calcul est activé sur l’appareil Azure Stack Edge Pro. Pour activer le calcul, accédez à la page **Calcul** de l’interface utilisateur locale de l’appareil. Puis sélectionnez une interface réseau que vous souhaitez activer pour le calcul. Sélectionnez **Activer**. L’activation du calcul entraîne la création d’un commutateur virtuel sur votre appareil sur cette interface réseau. Pour plus d’informations, consultez [Activer le réseau de calcul sur Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).

- Votre appareil Azure Stack Edge Pro a un serveur de cluster Kubernetes version v1.9 ou ultérieure en cours d’exécution. Pour plus d’informations, consultez [Créer et gérer un cluster Kubernetes sur un appareil Microsoft Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).

- Vous avez installé `kubectl`.

## <a name="deploy-a-stateless-application"></a>Déployer une application sans état

Avant de commencer, vous devez disposer des éléments suivants :

1. Un cluster Kubernetes créé.
2. Un espace de noms configuré.
3. Un utilisateur associé à l’espace de noms.
4. Une configuration utilisateur enregistrée dans `C:\Users\<username>\.kube`.
5. `kubectl` installé.

Vous pouvez maintenant commencer à exécuter et gérer des déploiements d’applications sans état sur un appareil Azure Stack Edge Pro. Avant de commencer à utiliser `kubectl`, vous devez vérifier que vous disposez de la version appropriée de `kubectl`.

### <a name="verify-you-have-the-correct-version-of-kubectl-and-set-up-configuration"></a>Vérifiez que vous disposez de la version appropriée de kubectl et effectuez la configuration

Pour vérifier la version, utilisez `kubectl` :

1. Vérifiez que la version de `kubectl` est supérieure ou égale à 1.9 :

   ```powershell
   kubectl version
   ```
    
   Voici un exemple de sortie :
    
   ```powershell
   PS C:\WINDOWS\system32> C:\windows\system32\kubectl.exe version
   Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
   Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
   ```

   Dans ce cas, la version cliente de kubectl est v 1.15.2 et est compatible pour la suite.

2. Obtenez la liste des pods en cours d’exécution sur votre cluster Kubernetes. Un pod est un conteneur d’application, ou processus, s’exécutant sur votre cluster Kubernetes.

   ```powershell
   kubectl get pods -n <namespace-string>
   ```
    
   Voici un exemple d’utilisation de la commande :
    
   ```powershell
   PS C:\WINDOWS\system32> kubectl get pods -n "test1"
   No resources found.
   PS C:\WINDOWS\system32>
   ```
    
   La sortie doit indiquer qu’aucune ressource (pod) n’a été trouvée, car aucune application n’est en cours d’exécution sur votre cluster.

   La commande remplit la structure de répertoire " C:\Users\\&lt;username&gt;\\.kube\" avec des fichiers de configuration. L’outil de ligne de commande kubectl utilise ces fichiers pour créer et gérer des applications sans État sur votre cluster Kubernetes.

3. Vérifiez manuellement la structure de répertoire "C:\Users\\&lt;username&gt;\\.kube\" pour vérifier que *kubectl* a rempli les sous-dossiers suivants :

   ```powershell
   PS C:\Users\username> ls .kube
    
    
      Directory: C:\Users\user\.kube
    
   Mode                LastWriteTime         Length Name
   ----                -------------         ------ ----
   d-----         2/18/2020 11:05 AM                cache
   d-----         2/18/2020 11:04 AM                http-cache
   -a----         2/18/2020 10:41 AM           5377 config
   ```

> [!NOTE]
> Pour afficher la liste de toutes les commandes kubectl, saisissez `kubectl --help`.

### <a name="create-a-stateless-application-using-a-deployment"></a>Créer une application sans état à l’aide d’un déploiement

Maintenant que vous avez vérifié que la version de la ligne de commande kubectl était correcte et que vous disposez des fichiers de configuration requis, vous pouvez créer un déploiement d’application sans état.

Un Pod est l’unité d’exécution de base d’une application Kubernetes, l’unité la plus petite et la plus simple dans le modèle objet Kubernetes que vous créez ou déployez. Un pod encapsule également les ressources de stockage, une adresse IP réseau unique et des options qui régissent la façon dont le ou les conteneurs doivent s’exécuter.

Le type d’application sans état que vous créez est un déploiement de serveur web nginx.

Toutes les commandes kubectl que vous utilisez pour créer et gérer des déploiements d’application sans état doivent spécifier l’espace de noms associé à la configuration. Vous avez créé l’espace de noms lors de la connexion au cluster sur l’appareil Azure Stack Edge Pro dans le tutoriel [Créer et gérer un cluster Kubernetes sur un appareil Microsoft Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md) avec `New-HcsKubernetesNamespace`.

Pour spécifier l’espace de noms dans une commande kubectl, utilisez `kubectl <command> -n <namespace-string>`.

Pour créer un déploiement nginx, procédez comme suit :

1. Appliquez une application sans état en créant un objet de déploiement Kubernetes :

   ```powershell
   kubectl apply -f <yaml-file> -n <namespace-string>
   ```

   Dans cet exemple, le chemin d’accès au fichier YAML de l’application est une source externe.

   Voici un exemple d’utilisation de la commande et de la sortie :

   ```powershell
   PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment.yaml -n "test1"
    
   deployment.apps/nginx-deployment created
   ```

   Vous pouvez également enregistrer le markdown suivant sur votre ordinateur local et remplacer le chemin d’accès et le nom de fichier dans le paramètre *-f*. Par exemple, « C:\Kubernetes\deployment.yaml ». Voici la configuration pour le déploiement de l’application :

   ```markdown
   apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     selector:
       matchLabels:
         app: nginx
     replicas: 2 # tells deployment to run 2 pods matching the template
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80
   ```

   Cette commande crée un déploiement nginx par défaut qui dispose de deux pods pour exécuter votre application.

2. Obtenez la description du déploiement nginx Kubernetes que vous avez créé :

   ```powershell
   kubectl describe deployment nginx-deployment -n <namespace-string>
   ```

   Voici un exemple d’utilisation de la commande et de la sortie :
    
   ```powershell
   PS C:\Users\user> kubectl describe deployment nginx-deployment -n "test1"
    
   Name:                   nginx-deployment
   Namespace:              test1
   CreationTimestamp:      Tue, 18 Feb 2020 13:35:29 -0800
   Labels:                 <none>
   Annotations:            deployment.kubernetes.io/revision: 1
                           kubectl.kubernetes.io/last-applied-configuration:
                             {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment","namespace":"test1"},"spec":{"repl...
   Selector:               app=nginx
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   StrategyType:           RollingUpdate
   MinReadySeconds:        0
   RollingUpdateStrategy:  25% max unavailable, 25% max surge
   Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.7.9
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
   Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
   OldReplicaSets:  <none>
   NewReplicaSet:   nginx-deployment-5754944d6c (2/2 replicas created)
   Events:
     Type    Reason             Age    From                   Message
     ----    ------             ----   ----                   -------
     Normal  ScalingReplicaSet  2m22s  deployment-controller  Scaled up replica set nginx-deployment-5754944d6c to 2
   ```

   Si vous examinez attentivement le paramètre *replicas*, vous verrez que :
    
   ```powershell
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   ```

   Le paramètre *replicas* indique que votre spécification de déploiement requiert deux pods, que ces pods ont été créés et mis à jour, et qu’ils sont prêts à être utilisés.

   > [!NOTE]
   > Un jeu de réplicas remplace les pods qui sont supprimés ou arrêtés pour une raison quelconque, comme dans le cas d’une défaillance de nœud d’appareil ou d’une mise à niveau d’appareil perturbatrice. Pour cette raison, nous vous recommandons d’utiliser un jeu de réplicas même si votre application ne requiert qu’un seul pod.

3. Pour répertorier les pods dans votre déploiement :

   ```powershell
   kubectl get pods -l app=nginx -n <namespace-string>
   ```
    
   Voici un exemple d’utilisation de la commande et de la sortie :
    
   ```powershell
   PS C:\Users\user> kubectl get pods -l app=nginx -n "test1"
    
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-5754944d6c-7wqjd   1/1     Running   0          3m13s
   nginx-deployment-5754944d6c-nfj2h   1/1     Running   0          3m13s
   ```

   La sortie vérifie que nous avons deux pods avec des noms uniques que nous pouvons référencer à l’aide de kubectl.

4. Pour afficher des informations sur un pod dans votre déploiement :

   ```powershell
   kubectl describe pod <podname-string> -n <namespace-string>
   ```

   Voici un exemple d’utilisation de la commande et de la sortie :

   ```powershell
   PS C:\Users\user> kubectl describe pod "nginx-deployment-5754944d6c-7wqjd" -n "test1"

   Name:           nginx-deployment-5754944d6c-7wqjd
   Namespace:      test1
   Priority:       0
   Node:           k8s-1d9qhq2cl-n1/10.128.46.184
   Start Time:     Tue, 18 Feb 2020 13:35:29 -0800
   Labels:         app=nginx
                   pod-template-hash=5754944d6c
   Annotations:    <none>
   Status:         Running
   IP:             172.17.246.200
   Controlled By:  ReplicaSet/nginx-deployment-5754944d6c
    Containers:
      nginx:
        Container ID:   docker://280b0f76bfdc14cde481dc4f2b8180cf5fbfc90a084042f679d499f863c66979
        Image:          nginx:1.7.9
        Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Tue, 18 Feb 2020 13:35:35 -0800
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-8gksw (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      default-token-8gksw:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-8gksw
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type    Reason     Age    From                       Message
      ----    ------     ----   ----                       -------
      Normal  Scheduled  4m58s  default-scheduler          Successfully assigned test1/nginx-deployment-5754944d6c-7wqjd to k8s-1d9qhq2cl-n1
      Normal  Pulling    4m57s  kubelet, k8s-1d9qhq2cl-n1  Pulling image "nginx:1.7.9"
      Normal  Pulled     4m52s  kubelet, k8s-1d9qhq2cl-n1  Successfully pulled image "nginx:1.7.9"
      Normal  Created    4m52s  kubelet, k8s-1d9qhq2cl-n1  Created container nginx
      Normal  Started    4m52s  kubelet, k8s-1d9qhq2cl-n1  Started container nginx
   ```

### <a name="rescale-the-application-deployment-by-increasing-the-replica-count"></a>Mettre à l’échelle le déploiement de l’application en augmentant le nombre de réplicas

Chaque pod est conçu pour exécuter une seule instance d’une application donnée. Si vous souhaitez mettre à l’échelle votre application horizontalement pour exécuter plusieurs instances, vous pouvez augmenter le nombre de pods, une pour chaque instance. Dans Kubernetes, on parle de réplication.
Vous pouvez augmenter le nombre de pods dans le déploiement de votre application en appliquant un nouveau fichier YAML. Le fichier YAML change le paramètre de réplicas sur 4, ce qui augmente le nombre de pods dans votre déploiement sur quatre pods. Pour augmenter le nombre de pods de 2 à 4 :

```powershell
PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml -n "test1"
```

Vous pouvez également enregistrer le markdown suivant sur votre ordinateur local et remplacer le chemin d’accès et le nom de fichier pour le paramètre *-f* pour `kubectl apply`. Par exemple, « C:\Kubernetes\deployment-scale.yaml ». Voici la configuration pour la mise à l'échelle du déploiement de l’application :

```markdown
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```

Pour vérifier que le déploiement comporte quatre pods :

```powershell
kubectl get pods -l app=nginx
```

Voici un exemple de sortie pour la remise à l’échelle d’un déploiement de deux à quatre pods :

```powershell
PS C:\WINDOWS\system32> kubectl get pods -l app=nginx

NAME                               READY     STATUS    RESTARTS   AGE
nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
nginx-deployment-148880595-fxcez   1/1       Running   0          2m
nginx-deployment-148880595-rwovn   1/1       Running   0          2m
```

Comme vous pouvez le voir dans la sortie, vous avez maintenant quatre pods dans votre déploiement qui peuvent exécuter votre application.

### <a name="delete-a-deployment"></a>Supprimer un déploiement

Pour supprimer le déploiement, y compris tous les pods, vous devez exécuter `kubectl delete deployment` en spécifiant le nom du déploiement *nginx-deployment* et le nom de l’espace de noms. Pour supprimer le déploiement :

   ```powershell
   kubectl delete deployment nginx-deployment -n <namespace-string>
   ```

Voici un exemple d’utilisation et de sortie de la commande :

```powershell
PS C:\Users\user> kubectl delete deployment nginx-deployment -n "test1"
deployment.extensions "nginx-deployment" deleted
```

## <a name="next-steps"></a>Étapes suivantes

[Présentation de Kubernetes](azure-stack-edge-gpu-kubernetes-overview.md)