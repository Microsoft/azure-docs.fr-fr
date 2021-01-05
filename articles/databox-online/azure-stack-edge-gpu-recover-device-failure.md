---
title: Effectuer une récupération après la défaillance d’un appareil Azure Stack Edge Pro
description: Décrit comment effectuer une récupération après la défaillance d’un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 12/11/2020
ms.author: alkohli
ms.openlocfilehash: e5734591bfc48469eacc1ad39cbb89f3850bfc8c
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97367064"
---
# <a name="recover-from-a-failed-azure-stack-edge-pro-gpu-device"></a>Récupération après la défaillance d’un appareil Azure Stack Edge Pro GPU 

Cet article explique comment effectuer une récupération après une défaillance non tolérable de votre appareil Azure Stack Edge Pro GPU. Une défaillance non tolérable d’un appareil Azure Stack Edge Pro GPU nécessite un remplacement de l’appareil.

## <a name="before-you-begin"></a>Avant de commencer

Vérifiez que vous avez effectué ces actions :

- Vous avez indiqué la défaillance de votre appareil à l’équipe Support Microsoft : la suite de cette procédure implique que l’équipe vous a recommandé de remplacer l’appareil. 
- Vous avez sauvegardé la configuration de votre appareil, comme décrit dans l’article [Se préparer à une défaillance d’appareil](azure-stack-edge-gpu-prepare-device-failure.md).


## <a name="configure-replacement-device"></a>Configurer l’appareil de remplacement

Lorsque votre appareil rencontre une défaillance non tolérable, vous devez commander un appareil de remplacement. Les étapes de configuration de l’appareil de remplacement restent les mêmes. 

Récupérez les informations de configuration de l’appareil que vous avez sauvegardées à partir de l’appareil défaillant. Vous allez utiliser ces informations pour configurer l’appareil de remplacement.  

Pour configurer l’appareil de remplacement, procédez comme suit :

1. Rassemblez les informations requises dans la [liste de contrôle du déploiement](azure-stack-edge-gpu-deploy-checklist.md). Vous devez utiliser les informations que vous avez enregistrées à partir de la configuration d’appareil précédente. 
1. Commandez un nouvel appareil ayant la même configuration que celui qui est défaillant.  Pour passer une commande, [créez une nouvelle ressource Edge Azure Stack](azure-stack-edge-gpu-deploy-prep.md#) dans le Portail Azure.
1. [Déballez l’appareil](azure-stack-edge-gpu-deploy-install.md#unpack-the-device), [montez-le en rack](azure-stack-edge-gpu-deploy-install.md#rack-the-device), puis [câblez-le](azure-stack-edge-gpu-deploy-install.md#cable-the-device). 
1. [Connectez-vous à l’interface utilisateur locale de l’appareil](azure-stack-edge-gpu-deploy-connect.md).
1. Configurez le réseau en utilisant les mêmes adresses IP que celles que vous avez utilisées pour votre ancien appareil. L’utilisation des mêmes adresses IP a pour effet de réduire l’impact sur les ordinateurs clients utilisés dans votre environnement. [Consultez cet article pour savoir comment configurer les paramètres réseau](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).
1. Attribuez le nom et le domaine DNS de votre ancien appareil à votre nouvel appareil. Ainsi, vous serez sûr que vos clients pourront utiliser le même nom d’appareil pour communiquer avec le nouvel appareil. [Consultez cet article pour savoir comment configurer les paramètres de l’appareil](azure-stack-edge-gpu-deploy-set-up-device-update-time.md).
1. Configurez les certificats sur le nouvel appareil de la même façon que pour l’ancien appareil. Gardez à l’esprit que le nouvel appareil a un nouveau numéro de série de nœud. Si vous avez utilisé vos propres certificats sur l’ancien appareil, vous devez obtenir un nouveau certificat de nœud. [Consultez cet article pour savoir comment configurer des certificats](azure-stack-edge-gpu-deploy-configure-certificates.md).
1. Récupérez la clé d’activation à partir du Portail Azure, puis activez le nouvel appareil. [Consultez cet article pour savoir comment activer l’appareil](azure-stack-edge-gpu-deploy-activate.md).

Vous êtes maintenant prêt à déployer les charges de travail que vous exécutiez sur l’ancien appareil.

## <a name="restore-edge-cloud-shares"></a>Restaurer les partages cloud Edge

Procédez comme suit pour restaurer les données sur les partages cloud Edge sur votre appareil :

1. [Ajoutez des partages](azure-stack-edge-j-series-manage-shares.md#add-a-share) portant les mêmes noms de partage que ceux créés précédemment sur l’appareil défaillant. Assurez-vous que, lors de la création des partages, la valeur du paramètre **Sélectionner le conteneur blob** est définie sur **Utilisez l’existant**, puis sélectionnez le conteneur utilisé avec l’appareil précédent.
1. [Ajoutez des utilisateurs](azure-stack-edge-j-series-manage-users.md#add-a-user) qui avaient accès à l’appareil précédent.
1. [Ajoutez les comptes de stockage](azure-stack-edge-j-series-manage-storage-accounts.md#add-an-edge-storage-account) associés aux partages précédemment sur l’appareil. Lors de la création de comptes de stockage Edge, faites votre sélection à partir d’un conteneur existant et pointez sur le conteneur qui a été mappé au compte de stockage Azure mappé sur l’appareil précédent. Toutes les données de l’appareil qui sont écrites dans le compte de stockage Edge de l’appareil précédent sont téléchargées vers le conteneur de stockage sélectionné dans le compte Stockage Azure mappé.
1. [Actualisez le partage](azure-stack-edge-j-series-manage-shares.md#refresh-shares) de données à partir d’Azure. Cela a pour effet d’extraire toutes les données cloud du conteneur existant vers les partages.

## <a name="restore-edge-local-shares"></a>Restaurer les partages locaux Edge

Pour vous préparer à une défaillance potentielle de l’appareil, vous avez peut-être déployé l’une des solutions de sauvegarde suivantes pour protéger les données des partages locaux de vos charges de travail Kubernetes ou IoT :

| Logiciels tiers           | Référence à la solution                               |
|--------------------------------|---------------------------------------------------------|
| Cohesity                       | [https://www.cohesity.com/solution/cloud/azure/](https://www.cohesity.com/solution/cloud/azure/) <br> Pour plus d’informations, contactez Cohesity.          |
| Commvault                      | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br> Pour plus d’informations, contactez Commvault. |
| Veritas                        | [http://veritas.com/azure](http://veritas.com/azure) <br> Pour plus d’informations, contactez Veritas.   |
| Veeam                          | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Pour plus d’informations, contactez Veeam. |

Une fois que l’appareil de remplacement est entièrement configuré, activez l’appareil pour le stockage local. 

Pour récupérer les données à partir de partages locaux, procédez comme suit :

1. [Configurez le calcul sur l’appareil](azure-stack-edge-gpu-deploy-configure-compute.md).
1. [Rajoutez un partage local](azure-stack-edge-j-series-manage-shares.md#add-a-local-share).
1. Exécutez la procédure de récupération fournie par la solution de protection des données de votre choix. Consultez les références du tableau précédent.

## <a name="restore-vm-files-and-folders"></a>Restaurer des fichiers et des dossiers de machines virtuelles

Pour vous préparer à une défaillance potentielle de l’appareil, vous avez peut-être déployé l’une des solutions de sauvegarde suivantes pour protéger les données sur les machines virtuelles :



| Solutions de sauvegarde        | Systèmes d’exploitation pris en charge   | Référence                                                                |
|-------------------------|----------------|--------------------------------------------------------------------------|
| Agent Microsoft Azure Recovery Services (MARS) pour Sauvegarde Azure | Windows        | [À propos de l’agent MARS](../backup/backup-azure-about-mars.md)    |
| Cohesity                | Windows, Linux | [Présentation de la solution d’intégration, de sauvegarde et de récupération de Microsoft Azure](https://www.cohesity.com/solution/cloud/azure) <br>Pour plus d’informations, contactez Cohesity.                          |
| Commvault               | Windows, Linux | [https://www.commvault.com/azure](https://www.commvault.com/azure) <br> Pour plus d’informations, contactez Commvault.
| Veritas                 | Windows, Linux | [https://vox.veritas.com/t5/Protection/Protecting-Azure-Stack-edge-with-NetBackup/ba-p/883370](https://vox.veritas.com/t5/Protection/Protecting-Azure-Stack-edge-with-NetBackup/ba-p/883370) <br> Pour plus d’informations, contactez Veritas.                    |
| Veeam                   | Windows, Linux | [https://www.veeam.com/kb4041](https://www.veeam.com/kb4041) <br> Pour plus d’informations, contactez Veeam. |

Une fois que l’appareil de remplacement est entièrement configuré, vous pouvez redéployer les machines virtuelles avec l’image de machine virtuelle précédemment utilisée. 

Pour récupérer les données dans les machines virtuelles, procédez comme suit :
 
1. [Déployez une machine virtuelle à partir d’une image de machine virtuelle](azure-stack-edge-gpu-deploy-virtual-machine-templates.md) sur l’appareil. 
1. Installez la solution de protection des données de votre choix sur la machine virtuelle.
1. Exécutez la procédure de récupération fournie par la solution de protection des données de votre choix. Consultez les références du tableau précédent.

## <a name="restore-a-kubernetes-deployment"></a>Restaurer un déploiement Kubernetes

Si vous avez effectué votre déploiement Kubernetes via Azure Arc, vous pouvez restaurer le déploiement après une défaillance de l’appareil non tolérable. Vous devez redéployer l’application cliente/les conteneurs à partir du référentiel `git` dans lequel la définition de l’application est stockée. [Informations sur le déploiement de Kubernetes avec Azure Arc](./azure-stack-edge-gpu-deploy-stateless-application-git-ops-guestbook.md)<!--Original text: Kubernetes deployments can be restored from a non-tolerated failure with the device when deployed with Azure Arc. Customer application/containers deployed onto a Kubernetes on Azure Stack Edge via Azure Arc can be redeployed from the git repository where the application definition is. Here is a link to the article to deploy Kubernetes with Arc -->
 
## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [retourner un appareil Azure Stack Edge Pro](azure-stack-edge-return-device.md).
