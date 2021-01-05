---
title: Comprendre la gestion des charges de travail Kubernetes sur un appareil Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment les charges de travail Kubernetes peuvent être gérées sur votre appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: ef840b3d9db4e82eeecea37079a08ccb0858a77b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448531"
---
# <a name="kubernetes-workload-management-on-your-azure-stack-edge-pro-device"></a>Gestion des charges de travail Kubernetes sur votre appareil Azure Stack Edge Pro

Sur votre appareil Azure Stack Edge Pro, un cluster Kubernetes est créé lorsque vous configurez le rôle de calcul. Une fois le cluster Kubernetes créé, les applications en conteneur peuvent être déployées sur le cluster Kubernetes dans des pods. Il existe différentes façons de déployer des charges de travail dans votre cluster Kubernetes. 

Cet article décrit les différentes méthodes qui peuvent être utilisées pour déployer des charges de travail sur votre appareil Azure Stack Edge Pro.

## <a name="workload-types"></a>Types de charge de travail

Les deux types courants de charges de travail que vous pouvez déployer sur votre appareil Azure Stack Edge Pro sont des applications sans état ou des applications avec état.

- Les **applications sans état** ne conservent pas leur état et n’enregistrent aucune donnée dans le stockage persistant. Toutes les données de l’utilisateur et de la session sont conservées côté client. Voici quelques exemples d’applications sans état : serveurs web front-end, comme Nginx, et autres applications web.

    Vous pouvez créer un déploiement Kubernetes pour déployer une application sans état sur votre cluster. 

- Les **applications avec état** requièrent l’enregistrement de leur état. Les applications avec état utilisent un stockage persistant, comme des volumes persistants, pour enregistrer des données à des fins d’utilisation par le serveur ou par d’autres utilisateurs. Les applications avec état sont, par exemple, des bases de données telles qu’[Azure SQL Edge](../azure-sql-edge/overview.md) et MongoDB.

    Vous pouvez créer un déploiement Kubernetes pour déployer une application avec état. 

## <a name="deployment-flow"></a>Flux de déploiement

Pour déployer des applications sur un appareil Azure Stack Edge Pro, procédez comme suit : 
 
1. **Configurer l’accès** : Tout d’abord, vous allez utiliser l’instance d’exécution de PowerShell pour créer un utilisateur, créer un espace de noms et accorder à l’utilisateur l’accès à cet espace de noms.
2. **Configurer le stockage** : Ensuite, vous allez utiliser la ressource Azure Stack Edge dans le portail Azure pour créer des volumes persistants à l’aide du provisionnement statique ou dynamique pour les applications avec état que vous allez déployer.
3. **Configurer la mise en réseau** : Enfin, vous allez utiliser les services pour exposer des applications en externe et au sein du cluster Kubernetes.
 
## <a name="deployment-types"></a>Types de déploiement

Il existe trois façons principales de déployer vos charges de travail. Chacune de ces méthodologies de déploiement vous permet de vous connecter à un espace de noms distinct sur l’appareil, puis de déployer des applications sans état ou avec état.

![Déploiement de charges de travail Kubernetes](./media/azure-stack-edge-gpu-kubernetes-workload-management/kubernetes-workload-management-1.png)

- **Déploiement local** : Ce déploiement s’effectue par le biais de l’outil d’accès en ligne de commande, comme `kubectl`, qui vous permet de déployer Kubernetes `yamls`. Vous accédez au cluster Kubernetes sur votre appareil Azure Stack Edge Pro via un fichier `kubeconfig`. Pour plus d’informations, consultez [Accéder à un cluster Kubernetes via kubectl](azure-stack-edge-gpu-create-kubernetes-cluster.md).

- **Déploiement IoT Edge** : Celui-ci s’effectue via IoT Edge, qui se connecte à Azure IoT Hub. Vous vous connectez au cluster Kubernetes sur votre appareil Azure Stack Edge Pro via l’espace de noms `iotedge`. Les agents IoT Edge déployés dans cet espace de noms sont responsables de la connectivité à Azure. Vous appliquez la configuration `IoT Edge deployment.json` à l’aide d’Azure DevOps CI/CD. La gestion de l’espace de noms et d’IoT Edge s’effectue par le biais de l’opérateur cloud.

- **Déploiement de Kubernetes activé par Azure Arc** : Kubernetes activé par Azure Arc est un outil de gestion hybride qui vous permet de déployer des applications sur vos clusters Kubernetes. Vous vous connectez au cluster Kubernetes sur votre appareil Azure Stack Edge Pro via `azure-arc namespace`. Les agents déployés dans cet espace de noms sont responsables de la connectivité à Azure. Vous appliquez la configuration du déploiement à l’aide de la gestion de la configuration basée sur GitOps. 
    
    Un Kubernetes activé par Azure Arc vous permet également d’utiliser Azure Monitor pour conteneurs afin d’afficher et de surveiller votre cluster. Pour plus d’informations, consultez [Qu’est-ce que Kubernetes activé par Azure Arc ?](../azure-arc/kubernetes/overview.md).

## <a name="choose-the-deployment-type"></a>Choisir le type de déploiement

Lors du déploiement d’applications, prenez en compte les informations suivantes :

- **Un ou plusieurs types** : Vous pouvez choisir une seule option de déploiement ou une combinaison d’options de déploiement différentes.
- **Cloud ou local** : Selon vos applications, vous pouvez choisir un déploiement local via kubectl ou un déploiement cloud via IoT Edge et Azure Arc. 
    - Lorsque vous choisissez un déploiement local, vous êtes limité au réseau dans lequel votre appareil Azure Stack Edge Pro est déployé.
    - Si vous avez un agent cloud que vous pouvez déployer, nous vous conseillons de déployer votre opérateur cloud et d’utiliser la gestion cloud.
- **IoT ou Azure Arc** : Le choix du déploiement dépend également de l’objectif de votre scénario de produit. Si vous déployez des applications ou des conteneurs qui ont une intégration plus poussée avec IoT ou un écosystème IoT, sélectionnez IoT Edge pour déployer vos applications. Si vous avez des déploiements Kubernetes existants, Azure Arc constitue le meilleur choix.


## <a name="next-steps"></a>Étapes suivantes

Pour déployer localement une application via kubectl, consultez :

- [Déployer une application sans état sur votre Azure Stack Edge Pro via kubectl](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).

Pour déployer une application via IoT Edge, consultez :

- [Déployer un exemple de module sur votre Azure Stack Edge Pro via IoT Edge](azure-stack-edge-gpu-deploy-sample-module.md).

Pour déployer une application via Azure Arc, consultez :

- [Déployer une application via Azure Arc](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).