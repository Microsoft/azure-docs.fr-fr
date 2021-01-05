---
title: Notes de publication d’Azure Stack Edge Pro (préversion) | Microsoft Docs
description: Décrit les problèmes majeurs existants et les résolutions pour Azure Stack Edge Pro exécutant la version préliminaire disponible.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 09/07/2020
ms.author: alkohli
ms.openlocfilehash: 25db4e7f3e4e1f7056979c4c40c6ffc61f340439
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96345369"
---
# <a name="azure-stack-edge-pro-with-gpu-preview-release-notes"></a>Notes de publication d’Azure Stack Edge Pro avec GPU (préversion)

Les notes de publication suivantes identifient des problèmes critiques, résolus ou non, de la préversion 2008 pour vos appareils Azure Stack Edge Pro avec GPU.

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Avant de déployer votre appareil Azure Stack Edge Pro, lisez attentivement les informations contenues dans les notes de version.

Cet article s’applique à la version logicielle suivante, **Azure Stack Edge Pro 2008**. 

<!--- **2.1.1328.1904**-->

## <a name="whats-new"></a>Nouveautés

Les nouvelles fonctionnalités suivantes ont été ajoutées à la version d’Azure Stack Edge 2008. Selon la version spécifique du logiciel en préversion que vous exécutez, vous pouvez voir une partie de ces fonctionnalités. 

- **Classes de stockage** – Dans la version précédente, vous pouviez provisionner le stockage uniquement de manière statique via des partages SMB ou NFS pour des applications avec état, déployées sur le cluster Kubernetes s’exécutant sur votre appareil Azure Stack Edge Pro. Dans cette mise en production, des classes de stockage permettant d’approvisionner le stockage de manière dynamique ont été ajoutées. Pour plus d’informations, consultez [Gestion du stockage Kubernetes sur votre appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-kubernetes-storage.md#dynamicprovisioning). 
- **Tableau de bord Kubernetes avec serveur de métriques** – Dans cette mise en production, un tableau de bord Kubernetes est ajouté avec un module complémentaire serveur de métriques. Vous pouvez utiliser le tableau de bord pour obtenir une vue d’ensemble des applications en cours d’exécution sur votre appareil Azure Stack Edge Pro, afficher l’état des ressources de cluster Kubernetes et voir les erreurs qui se sont produites sur l’appareil. Le serveur de métriques agrège l’utilisation de l’UC et de la mémoire par les ressources Kubernetes sur l’appareil. Pour plus d’informations, consultez [Utiliser le tableau de bord Kubernetes pour superviser votre appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).
- **Azure Arc pour Azure Stack Edge Pro** – À partir de cette version, vous pouvez déployer des charges de travail d’application sur votre appareil Azure Stack Edge Pro via Azure Arc. Azure Arc est un outil de gestion hybride qui vous permet de déployer des applications sur vos clusters Kubernetes. Pour plus d’informations, consultez [Déployer des charges de travail via Azure Arc sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).  

## <a name="known-issues"></a>Problèmes connus 

Le tableau suivant fournit un résumé de problèmes connus de l’appareil Azure Stack Edge Pro.

| Non. | Fonctionnalité | Problème | Solution de contournement/commentaires |
| --- | --- | --- | --- |
| **1.** |Azure Stack Edge Pro + SQL Azure | La création d’une base de données SQL nécessite un accès administrateur.   |Suivez les étapes suivantes au lieu des étapes 1-2 dans [https://docs.microsoft.com/azure/iot-edge/tutorial-store-data-sql-server#create-the-sql-database](../iot-edge/tutorial-store-data-sql-server.md#create-the-sql-database). <ul><li>Dans l’interface utilisateur locale de votre appareil, activez l’interface de calcul. Sélectionnez **Compute > N° de port > Activer pour le calcul > Appliquer.**</li><li>Téléchargez `sqlcmd` sur votre ordinateur client à partir de https://docs.microsoft.com/sql/tools/sqlcmd-utility. </li><li>Connectez-vous à l’adresse IP de votre interface de calcul (port activé), en ajoutant un « 1401 » à la fin de l’adresse.</li><li>La commande finale ressemble à ceci : sqlcmd -S {Interface IP},1401 -U SA -P "Strong!Passw0rd".</li>Ensuite, les étapes 3-4 de la documentation actuelle devraient être identiques. </li></ul> |
| **2.** |Actualiser| Les modifications incrémentielles d’objets blob restaurés via la fonctionnalité **Actualiser** ne sont pas prises en charge. |Pour les points de terminaison d’objet blob, des mises à jour partielles d’objets blob après une actualisation peuvent empêcher le chargement des mises à jour dans le cloud. Prenons l’exemple de la séquence d’actions suivante :<ul><li>Créer un objet blob dans le cloud. Ou supprimer un objet blob précédemment chargé à partir de l’appareil.</li><li>Actualiser l’objet blob à partir du cloud dans l’appliance à l’aide de la fonctionnalité d’actualisation.</li><li>Mettre à jour uniquement une partie de l’objet blob à l’aide des API REST du Kit de développement logiciel (SDK) Azure.</li></ul>Ces actions peuvent avoir pour effet que des sections mises à jour de l’objet blob ne sont pas mises à jour dans le cloud. <br>**Solution de contournement** : servez-vous d’outils tels que Robocopy ou d’une copie de fichiers normale via l’Explorateur ou la ligne de commande pour remplacer des objets blob entiers.|
|**3.**|Limitation|Lors d’une limitation, si de nouvelles écritures dans l’appareil ne sont pas autorisées, les écritures effectuées par le client NFS échouent avec l’erreur « Autorisation refusée ».| L’erreur s’affiche comme ci-dessous :<br>`hcsuser@ubuntu-vm:~/nfstest$ mkdir test`<br>mkdir : impossible de créer le répertoire ’test’ : Autorisation refusée|
|**4.**|Ingestion du Stockage Blob|Lors de l’utilisation d’AzCopy version 10 pour l’ingestion du stockage d’objets blob, exécutez AzCopy avec l’argument suivant : `Azcopy <other arguments> --cap-mbps 2000`.| Si ces limites ne sont pas fournies pour AzCopy, cela risque d’entraîner l’envoi d’un grand nombre de demandes à l’appareil et d’occasionner des problèmes avec le service.|
|**5.**|Comptes de stockage hiérarchisés|Voici ce qu’il se produit lors de l’utilisation de comptes de stockage hiérarchisés :<ul><li> Seuls les objets blob de blocs sont pris en charge. Les objets blob de pages ne sont pas pris en charge.</li><li>Il n’existe aucune prise en charge de l’API de copie ou de capture instantanée.</li><li> L’ingestion de charge de travail Hadoop via `distcp` n’est pas prise en charge, car elle sollicite abondamment l’opération de copie.</li></ul>||
|**6.**|Connexion de partage NFS|Si plusieurs processus copient vers le même partage et que l’attribut `nolock` n’est pas utilisé, des erreurs peuvent apparaître en cours de copie.|L’attribut `nolock` doit être passé à la commande mount pour copier des fichiers vers le partage NFS. Par exemple : `C:\Users\aseuser mount -o anon \\10.1.1.211\mnt\vms Z:`.|
|**7.**|Cluster Kubernetes|Lorsque vous appliquez une mise à jour sur votre appareil exécutant un cluster Kubernetes, les machines virtuelles Kubernetes redémarrent. Dans ce cas, seuls les pods déployés avec des réplicas spécifiés sont automatiquement restaurés après une mise à jour.  |Si vous avez créé des pods individuels en dehors d’un contrôleur de réplication sans spécifier de jeu de réplicas, ces pods ne seront pas restaurés automatiquement après la mise à jour de l’appareil. Vous devrez les restaurer.<br>Un jeu de réplicas remplace des pods supprimés ou arrêtés pour une raison quelconque, par exemple un échec de nœud ou une mise à niveau de nœud perturbatrice. Pour cette raison, nous vous recommandons d’utiliser un jeu de réplicas, même si votre application ne requiert qu’un seul pod.|
|**8.**|Cluster Kubernetes|Kubernetes sur Azure Stack Edge Pro est pris en charge uniquement avec Helm v3 ou ultérieur. Pour plus d’informations, consultez le [Forum Aux Questions : Suppression de Tiller](https://v3.helm.sh/docs/faq/).|
|**9.**|Azure Arc + Azure Stack Edge Pro|Les déploiements d’Azure Arc ne sont pas pris en charge si un proxy web est configuré sur votre appareil Azure Stack Edge Pro.||
|**10.**|Kubernetes |Le port 31000 est réservé au tableau de bord Kubernetes. De même, dans la configuration par défaut, les adresses IP 172.28.0.1 et 172.28.0.10 sont réservées respectivement pour le service Kubernetes et le service DNS de base.|N’utilisez pas d’adresses IP réservées.|
|**11.**|Kubernetes |Kubernetes n’autorise pas actuellement les services LoadBalancer multiprotocoles. Par exemple, un service DNS devant écouter à la fois les protocoles TCP et UDP. |Pour contourner cette limitation de Kubernetes avec MetalLB, deux services (un pour TCP, un pour UDP) peuvent être créés sur le même sélecteur de pod. Ces services utilisent la même clé de partage et spec.loadBalancerIP pour partager la même adresse IP. Des adresses IP peuvent également être partagées si vous avez plus de services qu’il n’y d’adresses IP disponibles. <br> Pour plus d’informations, voir [Partage d’adresse IP](https://metallb.universe.tf/usage/#ip-address-sharing).|
|**12.**|Cluster Kubernetes|Les modules existants de la place de marché Azure IoT Edge ne sont pas exécutés sur le cluster Kubernetes en tant que plateforme d’hébergement pour IoT Edge sur un appareil Azure Stack Edge.|Les modules doivent être modifiés avant leur déploiement sur l’appareil Azure Stack Edge. Pour plus d’informations, consultez Modifier des modules Azure IoT Edge de la place de marché pour les exécuter sur des appareils Azure Stack Edge.<!-- insert link-->|
|**13.**|Kubernetes |Les montages de liaison basés sur des fichiers ne sont pas pris en charge avec Azure IoT Edge sur Kubernetes sur un appareil Azure Stack Edge.|IoT Edge utilise une couche de traduction pour convertir des options de `ContainerCreate` en constructions Kubernetes. La création de `Binds` mappe au répertoire hostpath. Par conséquent, des montages de liaison basés sur des fichiers ne peuvent pas être liés à des chemins d’accès dans des conteneurs IoT Edge.|
|**14.**|Kubernetes |Si vous apportez vos propres certificats pour IoT Edge et les ajoutez sur votre appareil Azure Stack Edge, les nouveaux certificats ne sont pas sélectionnés dans le cadre de la mise à jour des graphiques Helm.|Pour contourner ce problème, [connectez-vous à l’interface PowerShell de l’appareil](azure-stack-edge-gpu-connect-powershell-interface.md). Redémarrez les pods `iotedged` et `edgehub`.|
|**15.**|Certificats |Dans certains cas, la mise à jour de l’état des certificats dans l’interface utilisateur locale peut prendre plusieurs secondes. |Les scénarios suivants dans l’interface utilisateur locale peuvent être affectés.<ul><li>Colonne **État** dans la page **Certificats**.</li><li>Vignette **Sécurité** dans la page **Démarrer**.</li><li>Vignette **Configuration** dans la page **Vue d’ensemble**.</li></ul>  |

## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’un appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-deploy-prep.md)