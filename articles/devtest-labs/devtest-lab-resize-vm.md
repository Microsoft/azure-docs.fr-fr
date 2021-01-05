---
title: Redimensionner une machine virtuelle dans un labo dans Azure DevTest Labs
description: Découvrez comment modifier la taille d’une machine virtuelle dans Azure DevTest Labs en fonction de vos besoins en matière de performances de l’UC, du réseau ou du disque.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 592be0862bc33ead86fe43d37753fb72fde120c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85482018"
---
# <a name="resize-a-vm-in-a-lab-in-azure-devtest-labs"></a>Redimensionner une machine virtuelle dans un labo dans Azure DevTest Labs
L’une des fonctionnalités importantes des machines virtuelles Azure est la possibilité de changer la taille d’une machine virtuelle selon vos besoins en termes d’UC, de réseau ou de performances disque. Azure DevTest Labs prend en charge cette fonctionnalité pour les machines virtuelles dans un labo. La fonctionnalité de redimensionnement est conforme à la stratégie de labo pour les tailles de machine virtuelle autorisées dans le labo. Autrement dit, vous ne pouvez définir la taille d’une machine virtuelle que sur les tailles autorisées dans le labo. 


## <a name="steps-to-resize-a-vm-in-a-lab"></a>Étapes du redimensionnement d’une machine virtuelle dans un labo 
Pour redimensionner une machine virtuelle dans un labo dans Azure DevTest Labs, effectuez les étapes suivantes : 

> [!NOTE]
> Si vous êtes connecté à la machine virtuelle via une session Bureau à distance (RDP), enregistrez votre travail et déconnectez-vous de la machine virtuelle avant de la redimensionner.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
3. Dans la liste des labos, sélectionnez celui dans lequel se trouve la machine virtuelle que vous souhaitez redimensionner.  
4. Dans le panneau de gauche, sélectionnez **Mes machines virtuelles**. 
5. Dans la liste des machines virtuelles, sélectionnez une machine virtuelle.
6. Sélectionnez **Arrêter** dans la barre d’outils si la machine virtuelle est en cours d’exécution. Vérifiez l’état de l’opération dans la fenêtre **Notifications**. Attendez que la machine virtuelle soit arrêtée, puis fermez la fenêtre **Notifications**. 

    ![Arrêtez la machine virtuelle.](media/devtest-lab-resize-vm/stop-vm.png)
1. Dans la page Machine virtuelle associée à votre machine virtuelle, sélectionnez **Taille** sous **PARAMÈTRES** dans le menu de gauche.

    ![Menu Taille](media/devtest-lab-resize-vm/size-menu.png)
1. Dans la fenêtre **Choisir une taille**, sélectionnez une taille pour votre machine virtuelle, puis cliquez sur **Sélectionner**.     
1. Vérifiez l’état de l’opération de redimensionnement dans la fenêtre **Notifications**.

    ![État du redimensionnement](media/devtest-lab-resize-vm/resize-status.png)
10. Une fois le redimensionnement effectué, fermez la fenêtre **Notifications**. 
11. Sélectionnez **Vue d’ensemble** dans le menu de gauche, puis sélectionnez **Redémarrer** dans la barre d’outils pour redémarrer la machine virtuelle. 

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la fonctionnalité de redimensionnement prise en charge par les machines virtuelles Azure, consultez [Redimensionner les machines virtuelles](https://azure.microsoft.com/blog/resize-virtual-machines/).


