---
title: Superviser votre appareil Azure Stack Edge Pro via le tableau de bord Kubernetes | Microsoft Docs
description: Décrit comment accéder au tableau de bord Kubernetes et l’utiliser pour superviser votre appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: 137cff47d49be1405f60bc47cd16f7f027ab63a9
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91320827"
---
# <a name="use-kubernetes-dashboard-to-monitor-your-azure-stack-edge-pro-gpu-device"></a>Utiliser le tableau de bord Kubernetes pour superviser votre appareil Azure Stack Edge Pro avec GPU

Cet article décrit comment accéder au tableau de bord Kubernetes et l’utiliser pour superviser votre appareil Azure Stack Edge Pro avec GPU. Pour surveiller votre appareil, vous pouvez utiliser des graphiques dans le portail Azure, afficher le tableau de bord Kubernetes ou exécuter des commandes `kubectl` par le biais de l’interface PowerShell de l’appareil. 

Cet article se concentre uniquement sur les tâches de surveillance qui peuvent être effectuées sur le tableau de bord Kubernetes.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Accéder au tableau de bord Kubernetes sur votre appareil
> * Afficher les modules déployés sur votre appareil
> * Obtenir l’adresse IP des applications déployées sur votre appareil
> * Afficher les journaux de conteneur pour les modules déployés sur votre appareil


## <a name="about-kubernetes-dashboard"></a>À propos du tableau de bord Kubernetes

Le tableau de bord Kubernetes est une interface utilisateur web que vous pouvez utiliser pour dépanner vos applications en conteneur. Le tableau de bord Kubernetes est une alternative basée sur une interface utilisateur à la ligne de commande Kubernetes `kubectl`. Pour plus d’informations, consultez la section [Tableau de bord Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). 

Sur votre appareil Azure Stack Edge Pro, vous pouvez utiliser le tableau de bord Kubernetes en mode *lecture seule* pour obtenir une vue d’ensemble des applications en cours d’exécution sur votre appareil Azure Stack Edge Pro, afficher l’état des ressources de cluster Kubernetes et afficher les erreurs qui se sont produites sur l’appareil.

## <a name="access-dashboard"></a>Accès au tableau de bord

Le tableau de bord Kubernetes est en *lecture seule* et s’exécute sur le nœud principal Kubernetes sur le port 31000. Pour accéder au tableau de bord, procédez comme suit : 

1. Dans l’interface utilisateur locale de votre appareil, accédez à **Appareil**, puis accédez à **Points de terminaison d’appareil**. 
1. Sélectionnez **Télécharger la configuration** pour télécharger un fichier `kubeconfig` qui vous permettra d’accéder au tableau de bord. Enregistrez le fichier `config.json` sur votre système local.
1. Sélectionnez l’URL du tableau de bord Kubernetes pour ouvrir le tableau de bord dans un navigateur.

    ![URL du tableau de bord Kubernetes dans la page Appareil dans l’interface utilisateur locale](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-dashboard-url-local-ui-1.png)

1. Dans la page **Connexion au tableau de bord Kubernetes** :
    
    1. Sélectionnez **kubeconfig**. 
        ![Sélectionner l’option kubeconfig](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-dashboard-sign-in-1.png) 
    1. Sélectionnez les points de suspension ( **...** ). Recherchez et pointez sur le fichier `kubeconfig` que vous avez téléchargée précédemment sur votre système local. Sélectionnez **Se connecter**.
        ![Accéder au fichier kubeconfig](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-dashboard-sign-in-2.png)    

6. Vous pouvez maintenant afficher le tableau de bord Kubernetes pour votre appareil Azure Stack Edge Pro en mode lecture seule.

    ![Page principale du tableau de bord Kubernetes](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-dashboard-main-page-1.png)

## <a name="view-module-status"></a>Afficher l’état du module

Les modules de calcul sont des conteneurs avec une logique métier implémentée. Vous pouvez utiliser le tableau de bord pour vérifier si un module de calcul a été déployé correctement sur votre appareil Azure Stack Edge Pro.

Pour afficher l’état du module, procédez comme suit dans le tableau de bord :

1. Dans le volet gauche du tableau de bord, accédez à **Espace de noms**. Filtrez par l’espace de noms dans lequel les modules IoT Edge sont affichés, dans notre cas **iotedge**.
1. Dans le volet gauche, accédez à **Charges de travail > Déploiements**.
1. Dans le volet droit, vous verrez tous les modules déployés sur votre appareil. Dans notre cas, un module GettingStartedWithGPU a été déployé sur l’appareil Azure Stack Edge Pro. Vous pouvez voir que le module a été déployé.

    ![Afficher le déploiement de module](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-view-module-deployment-1.png)

 
## <a name="get-ip-address-for-services-or-modules"></a>Obtenir l’adresse IP des services ou des modules

Vous pouvez utiliser le tableau de bord pour obtenir les adresses IP des services ou modules que vous souhaitez exposer en dehors du cluster Kubernetes. 

Vous attribuez la plage d’adresses IP de ces services externes via l’interface utilisateur web locale de l’appareil dans la page **Paramètres du réseau de calcul**. Une fois que vous avez déployé les modules IoT Edge, vous souhaiterez peut-être obtenir l’adresse IP affectée à un module ou service spécifique. 

Pour récupérer l’adresse IP, procédez comme suit sur le tableau de bord :

1. Dans le volet gauche du tableau de bord, accédez à **Espace de noms**. Filtrez par l’espace de noms dans lequel un service externe est déployé, dans notre cas, **iotedge**.
1. Dans le volet gauche, accédez à **Découverte et équilibrage de charge > Services**.
1. Dans le volet droit, vous verrez tous les services qui s’exécutent dans l’espace de noms `iotedge` sur votre appareil Azure Stack Edge Pro.

    ![Obtenir des adresses IP pour les services externes](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-get-ip-external-service-1.png)

## <a name="view-container-logs"></a>Afficher les journaux d’activité du conteneur

Dans certains cas, vous devez afficher les journaux de conteneur. Vous pouvez utiliser le tableau de bord pour obtenir les journaux d’un conteneur spécifique que vous avez déployé sur votre cluster Kubernetes.

Pour afficher les journaux de conteneur, procédez comme suit dans le tableau de bord :

1. Dans le volet gauche du tableau de bord, accédez à **Espace de noms**. Filtrez par l’espace de noms dans lequel les modules IoT Edge sont déployés, dans notre cas **iotedge**.
1. Dans le volet gauche, accédez à **Charges de travail > Pods**.
1. Dans le volet droit, vous verrez tous les pods exécutés sur votre appareil. Identifiez le pod qui exécute le module pour lequel vous souhaitez afficher les journaux. Sélectionnez les points de suspension verticaux pour le pod que vous avez identifié et, dans le menu contextuel, sélectionnez **Journaux**.

    ![Afficher les journaux du conteneur 1](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-view-container-logs-1.png)

1. Les journaux sont affichés dans la visionneuse de journaux qui est intégrée au tableau de bord. Vous pouvez également télécharger les journaux.

    ![Afficher les journaux du conteneur 2](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/kubernetes-view-container-logs-1.png)
    

## <a name="view-cpu-memory-usage"></a>Afficher l’utilisation du processeur et de la mémoire

Le tableau de bord Kubernetes pour l’appareil Azure Stack Edge Pro possède également un [module complémentaire de serveur de métriques](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/) qui agrège l’utilisation du processeur et de la mémoire sur les ressources Kubernetes.
 
Par exemple, vous pouvez afficher le processeur et la mémoire consommés entre les déploiements dans tous les espaces de noms. 

![Afficher l’utilisation de l’UC et de la mémoire dans tous les déploiements](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/view-cpu-memory-all-1.png)

Vous pouvez également filtrer par un espace de noms spécifique. Dans l’exemple suivant, vous pouvez afficher la consommation de l’UC et de la mémoire uniquement pour les déploiements Azure Arc.  

![Afficher l’utilisation de l’UC et de la mémoire pour les déploiements Azure Arc](./media/azure-stack-edge-gpu-monitor-kubernetes-dashboard/view-cpu-memory-azure-arc-1.png)

Le serveur de métriques Kubernetes fournit des pipelines de mise à l’échelle automatique tels que [Autoscaler de pods horizontaux](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Résoudre les problèmes d’appareil](azure-stack-edge-gpu-troubleshoot.md).
