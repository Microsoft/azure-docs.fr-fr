---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 1c3996c3f40da496af0cd795d0873864667a1f19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177068"
---
## <a name="use-the-azure-portal"></a>Utilisation du portail Azure
1. Sélectionnez la machine virtuelle que vous souhaitez redéployer, puis sélectionnez le bouton *Redéployer* dans le panneau *Paramètres*. Si besoin, faites défiler la page vers le bas pour voir la section **Support et dépannage** qui contient le bouton « Redéployer », comme dans l’exemple suivant :
   
    ![Panneau Machines virtuelles Azure](./media/virtual-machines-common-redeploy-to-new-node/vmoverview.png)
2. Cliquez sur le bouton *Redéployer* pour confirmer l’opération :
   
    ![Panneau Redéployer une machine virtuelle](./media/virtual-machines-common-redeploy-to-new-node/redeployvm.png)
3. L’**État** de la machine virtuelle passe à *Mise à jour* lors de sa préparation au redéploiement, comme illustré dans l’exemple suivant :
   
    ![Mise à jour de la machine virtuelle](./media/virtual-machines-common-redeploy-to-new-node/vmupdating.png)
4. L’**État** passe ensuite à *Démarrage* lors du démarrage de la machine virtuelle sur un nouvel hôte Azure, comme illustré dans l’exemple suivant :
   
    ![Démarrage de la machine virtuelle](./media/virtual-machines-common-redeploy-to-new-node/vmstarting.png)
5. Une fois que la machine virtuelle a terminé le processus de démarrage, son **État** redevient *Exécution*, indiquant qu’elle a été correctement redéployée :
   
    ![Exécution de la machine virtuelle](./media/virtual-machines-common-redeploy-to-new-node/vmrunning.png)

