---
title: Connectez-vous à vos machines virtuelles Windows dans Azure DevTest Labs
description: Découvrez comment vous connecter à votre machine virtuelle Windows dans le lab (Azure DevTest Labs)
ms.topic: how-to
ms.date: 07/17/2020
ms.openlocfilehash: e1e786daa396548030976159d1b150caa4b24396
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86540571"
---
# <a name="connect-to-a-windows-vm-in-your-lab-azure-devtest-labs"></a>Se connecter à une machine virtuelle Windows dans votre lab (Azure DevTest Labs)
Cet article vous explique comment vous connecter à une machine virtuelle Windows dans votre lab. 

## <a name="connect-to-a-windows-vm"></a>Se connecter à une machine virtuelle Windows
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre de recherche, recherchez et sélectionnez **DevTest Lab** . 

    :::image type="content" source="./media/connect-windows-virtual-machine/search-select.png" alt-text="Recherchez et sélectionnez DevTest Labs":::    
1. Dans la liste de labos, sélectionnez votre **labo** .

    :::image type="content" source="./media/connect-windows-virtual-machine/select-lab.png" alt-text="Recherchez et sélectionnez DevTest Labs":::            
1. Dans la page d’accueil de votre lab, sélectionnez votre machine virtuelle Windows dans la liste **Mes machines virtuelles** . 

    :::image type="content" source="./media/connect-windows-virtual-machine/select-windows-vm.png" alt-text="Recherchez et sélectionnez DevTest Labs":::                
1. Sur la page **Machine virtuelle** de votre machine virtuelle, sélectionnez **Se connecter** dans la barre d’outils. Si la machine virtuelle est arrêtée, sélectionnez **Démarrer** en premier pour démarrer la machine virtuelle.

    :::image type="content" source="./media/connect-windows-virtual-machine/select-connect.png" alt-text="Recherchez et sélectionnez DevTest Labs":::                    
1. Si vous utilisez le navigateur Edge, le lien vers le **fichier RDP téléchargé s’affiche** en bas du navigateur. 

    :::image type="content" source="./media/connect-windows-virtual-machine/rdp-download.png" alt-text="Recherchez et sélectionnez DevTest Labs":::                        
1. Ouvrez le fichier RDP, puis entrez les informations d’identification de votre machine virtuelle que vous avez tapées lors de la création de la machine virtuelle. Vous devriez maintenant être connecté à la machine virtuelle Windows. 

## <a name="next-steps"></a>Étapes suivantes
[Comment se connecter à une machine virtuelle Linux](connect-linux-virtual-machine.md)
