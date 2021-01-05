---
title: Télécharger un disque VHD Linux à partir d’Azure
description: Télécharger un disque VHD Linux à l’aide de l’interface de ligne de commande Azure et du portail Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/03/2020
ms.author: cynthn
ms.openlocfilehash: 897cae53e589f4058e5499c0e6e941d4f1d9bb2f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87761050"
---
# <a name="download-a-linux-vhd-from-azure"></a>Télécharger un disque VHD Linux à partir d’Azure

Dans cet article, vous apprendrez à télécharger un fichier de disque dur virtuel (VHD) Linux à partir d’Azure à l’aide du Portail Azure. 

## <a name="stop-the-vm"></a>Arrêtez la machine virtuelle.

Il n’est pas possible de télécharger un disque VHD associé à une machine virtuelle en cours d’exécution à partir d’Azure. Il vous faut arrêter la machine virtuelle pour télécharger un VHD. 

1.  Connectez-vous au [portail Azure](https://portal.azure.com/).
2.  Dans le menu de gauche, sélectionnez **Machines virtuelles**.
3.  Sélectionnez la machine virtuelle dans la liste.
4.  Sur la page de la machine virtuelle, sélectionnez **Arrêter**.

    :::image type="content" source="./media/download-vhd/export-stop.PNG" alt-text="Affiche le bouton de menu permettant d’arrêter la machine virtuelle.":::

## <a name="generate-sas-url"></a>Générer une URL de SAP

Pour télécharger le fichier VHD, vous devez générer une URL de [signature d’accès partagé (SAP)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json). Un délai d’expiration est affecté à l’URL lors de sa génération.

1. Dans le menu du page pour la machine virtuelle, sélectionnez **Disques**.
2. Sélectionnez le disque de système d’exploitation de la machine virtuelle, puis **Exportation de disque**.
1. Si nécessaire, mettez à jour la valeur de **URL expire dans (secondes)** pour vous fournir suffisamment de temps pour terminer le téléchargement. La valeur par défaut est 3 600 secondes (une heure).
3. Sélectionnez **Générer une URL**.
 
      
## <a name="download-vhd"></a>Télécharger un VHD

1.  Sous l’URL générée, sélectionnez **Télécharger le fichier de disque dur virtuel**.

    :::image type="content" source="./media/download-vhd/export-download.PNG" alt-text="Affiche le bouton de menu permettant d’arrêter la machine virtuelle.":::

2.  Vous devrez peut-être sélectionner **Enregistrer** dans le navigateur pour commencer le téléchargement. Le nom par défaut du fichier VHD est *abcd*.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [charger et créer une machine virtuelle Linux à partir d’un disque personnalisé avec Azure CLI](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 
- [Gestion des disques Azure avec l’interface de ligne de commande Azure](tutorial-manage-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
