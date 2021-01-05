---
title: Configurer le domaine par défaut NFSv4.1 pour Azure NetApp Files | Microsoft Docs
description: Explique comment configurer le client NFS afin d’utiliser NFSv4.1 avec Azure NetApp Files.
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/14/2020
ms.author: b-juche
ms.openlocfilehash: c3c853190d5f63bbe9012727d8b7b7ac91da135f
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072150"
---
# <a name="configure-nfsv41-default-domain-for-azure-netapp-files"></a>Configurer le domaine par défaut NFSv4.1 pour Azure NetApp Files

NFSv4 introduit le concept de domaine d’authentification. Azure NetApp Files prend en charge le mappage utilisateur racine uniquement depuis le service vers le client NFS. Pour utiliser la fonctionnalité NFSv4.1 avec Azure NetApp Files, vous devez mettre à jour le client NFS.

## <a name="default-behavior-of-usergroup-mapping"></a>Comportement par défaut du mappage de l’utilisateur/groupe

Par défaut, le mappage racine est réglé sur l’utilisateur `nobody`, car le domaine NFSv4 est défini sur `localdomain` par défaut. Quand vous montez un volume Azure NetApp Files NFSv4.1 en tant que racine, vous voyez des autorisations de fichiers comme suit :  

![Comportement par défaut du mappage de l’utilisateur/groupe pour NFSv4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-default-behavior-user-group-mapping.png)

Comme le montre l’exemple ci-dessus, l’utilisateur de `file1` doit être `root`, mais il est mappé à `nobody` par défaut.  Cet article explique comment définir l’utilisateur `file1` sur `root` en modifiant le paramètre `idmap Domain` sur `defaultv4iddomain.com`.  

## <a name="steps"></a>Étapes 

1. Modifiez le fichier `/etc/idmapd.conf` sur le client NFS.   
    Supprimez les marques de commentaire de la ligne `#Domain` (c’est-à-dire, supprimez le caractère `#` de la ligne), puis remplacez la valeur `localdomain` par `defaultv4iddomain.com`. 

    Configuration initiale : 
    
    ![Configuration initiale pour NFSv4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-initial-config.png)

    Configuration mise à jour :
    
    ![Configuration mise à jour pour NFSv4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-updated-config.png)

2. Démontez tout volume NFS monté.
3. Mettre à jour le fichier `/etc/idmapd.conf`.
4. Redémarrez le service `rpcbind` sur votre hôte (`service rpcbind restart`) ou redémarrez simplement l’hôte.
5. Montez les volumes NFS selon les besoins.   

    Voir [Monter ou démonter un volume pour des machines virtuelles Windows ou Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md). 

L’exemple suivant illustre la modification de l’utilisateur/groupe obtenue : 

![Capture d’écran montrant un exemple de la modification de groupe/d’utilisateur obtenue.](../media/azure-netapp-files/azure-netapp-files-nfsv41-resulting-config.png)

Comme le montre l’exemple, l’utilisateur/groupe est désormais passé de `nobody` à `root`.

## <a name="behavior-of-other-non-root-users-and-groups"></a>Comportement d’autres utilisateurs et groupes (non racine)

Azure NetApp Files prend en charge les utilisateurs locaux (utilisateurs créés localement sur un hôte) qui ont des autorisations associées à des fichiers ou dossiers dans les volumes NFSv4.1. Toutefois, le service ne prend pas en charge le mappage des utilisateurs/groupes sur plusieurs nœuds. Ainsi, les utilisateurs créés sur un hôte ne sont pas mappés par défaut aux utilisateurs créés sur un autre hôte. 

Dans l’exemple suivant, `Host1` a trois comptes d’utilisateur de test existants (`testuser01`, `testuser02`, `testuser03`) : 

![Capture d’écran montrant que Host1 a trois comptes d’utilisateur de test.](../media/azure-netapp-files/azure-netapp-files-nfsv41-host1-users.png)

Sur `Host2`, notez que les comptes d’utilisateur de test n’ont pas été créés, mais que le même volume est monté sur les deux hôtes :

![Configuration résultante pour NFSv4.1](../media/azure-netapp-files/azure-netapp-files-nfsv41-host2-users.png)

## <a name="next-step"></a>Étape suivante 

[Monter ou démonter un volume pour des machines virtuelles Windows ou Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)

