---
title: Utiliser une GPU ou VPU d’accélération du calcul sur des appareils Azure Stack Edge pour des déploiements Kubernetes | Microsoft Docs
description: Décrit comment utiliser une GPU ou VPU d’accélération du calcul sur votre Azure Stack Edge Pro GPU, Azure Stack Edge Pro R ou Azure Stack Mini R pour des déploiements Kubernetes.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/05/2020
ms.author: alkohli
ms.openlocfilehash: cf70b24dae70ad2e64f3443e4c4d959d46fb4ea4
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095030"
---
# <a name="use-compute-acceleration-on-azure-stack-edge-pro-gpu-for-kubernetes-deployment"></a>Utiliser l’accélération du calcul sur Azure Stack Edge Pro GPU pour un déploiement Kubernetes

Cet article explique comment utiliser l’accélération du calcul sur des appareils Azure Stack Edge lors de l’utilisation de déploiements Kubernetes. Cet article s’applique aux appareils Azure Stack Edge Pro GPU, Azure Stack Edge Pro R et Azure Stack Edge Mini R.


## <a name="about-compute-acceleration"></a>À propos de l’accélération du calcul 

L’unité centrale (UC) est le calculateur à usage général par défaut pour la plupart des processus s’exécutant sur un ordinateur. Souvent, du matériel informatique spécialisé est utilisé pour exécuter certaines fonctions plus efficacement que dans le logiciel d’une UC. Par exemple, une unité centrale graphique (GPU) peut être utilisée pour accélérer le traitement de données de pixels.  

L’accélération du calcul est une expression utilisée spécifiquement pour des appareils Azure Stack Edge dans lesquels une unité centrale graphique (Graphical Processing Unit, GPU), une unité de traitement de la vision (Vision Processing Unit, VPU) ou un réseau de portes programmables in situ (Field Programmable Gate Array, FPGA) sont utilisés pour l’accélération matérielle. La plupart des charges de travail déployées sur votre appareil Azure Stack Edge incluent une synchronisation critique, plusieurs flux de caméra et/ou des fréquences d’images élevées, nécessitant une accélération matérielle spécifique.

L’article traite de l’accélération du calcul uniquement à l’aide d’une GPU ou VPU pour les appareils suivants :

- **Azure Stack Edge Pro GPU** : ces appareils peuvent avoir 1 ou 2 GPU Nvidia T4 Tensor Core. Pour plus d’informations, consultez [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).
- **Azure Stack Edge Pro R** : ces appareils ont 1 GPU Nvidia T4 Tensor Core. Pour plus d’informations, consultez [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).
- **Azure Stack Edge Mini R** : ces ont 1 VPU Intel Movidius Myriad X. Pour plus d’informations, consultez [Intel Movidius Myriad X VPU](https://www.movidius.com/MyriadX).


## <a name="use-gpu-for-kubernetes-deployment"></a>Utiliser une GPU pour un déploiement Kubernetes

L’exemple `yaml` suivant peut être utilisé pour un appareil Azure Stack Edge Pro GPU ou Azure Stack Edge Pro R avec une GPU.

<!--In a production scenario, Pods are not used directly and these are wrapped around higher level constructs like Deployment, ReplicaSet which maintain the desired state in case of pod restarts, failures.-->

```yml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: cuda-container
      image: nvidia/cuda:9.0-devel
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
    - name: digits-container
      image: nvidia/digits:6.0
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
```


## <a name="use-vpu-for-kubernetes-deployment"></a>Utiliser une VPU pour un déploiement Kubernetes

L’exemple `yaml` suivant peut être utilisé pour un appareil Azure Stack Edge Mini R équipé d’une VPU.

```yml
apiVersion: batch/v1
kind: Job
metadata:
 name: intelvpu-demo-job
 labels:
   jobgroup: intelvpu-demo
spec:
 template:
   metadata:
     labels:
       jobgroup: intelvpu-demo
   spec:
     restartPolicy: Never
     containers:
       -
         name: intelvpu-demo-job-1
         image: ubuntu-demo-openvino:devel
         imagePullPolicy: IfNotPresent
         command: [ "/do_classification.sh" ]
         resources:
           limits:
             vpu.intel.com/hddl: 1
```


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Utiliser kubectl pour exécuter une application avec état Kubernetes à l’aide d’une ressource PersistentVolume sur votre appareil Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).
