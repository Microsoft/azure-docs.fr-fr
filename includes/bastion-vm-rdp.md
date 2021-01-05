---
title: Fichier include
description: Fichier include
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 10/21/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 083ab61d5a20bfb8e38747ae0694b1176c0a0fd1
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521529"
---
1. Ouvrez le [portail Azure](https://portal.azure.com). Accédez à la machine virtuelle à laquelle vous souhaitez vous connecter, puis sélectionnez **Se connecter** . Dans la liste déroulante, sélectionnez **Bastion** .

   :::image type="content" source="./media/bastion-vm-rdp/connect-vm.png" alt-text="Sélectionnez Bastion":::

1. Une fois que vous avez sélectionné Bastion dans la liste déroulante, une barre latérale affiche trois onglets : RDP, SSH et Bastion. Si le service Bastion a été approvisionné pour le réseau virtuel, l’onglet Bastion est actif par défaut. Sélectionnez **Utiliser Bastion** .

   :::image type="content" source="./media/bastion-vm-rdp/select-use-bastion.png" alt-text="Sélectionnez Bastion":::

1. Dans la page **Se connecter à l’aide d’Azure Bastion** , entrez le nom d’utilisateur et le mot de passe de votre machine virtuelle, puis sélectionnez **Se connecter** .

   :::image type="content" source="./media/bastion-vm-rdp/connect-vm-host.png" alt-text="Sélectionnez Bastion":::

1. La connexion RDP à cette machine virtuelle avec Bastion s’ouvrira directement dans le portail Azure (en HTML5) via le port 443 et le service Bastion.

   :::image type="content" source="./media/bastion-vm-rdp/connection.png" alt-text="Sélectionnez Bastion":::
