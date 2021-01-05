---
title: Se connecter à un groupe de machines virtuelles Windows identiques à l’aide d’Azure Bastion | Microsoft Docs
description: Dans cet article, découvrez comment se connecter à un groupe de machines virtuelles Azure identiques à l’aide d’Azure Bastion.
services: bastion
author: charwen
ms.service: bastion
ms.topic: how-to
ms.date: 10/14/2020
ms.author: charwen
ms.openlocfilehash: 001d2ff6789ec4cfcc391171f0859b67ab1ee0a5
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077777"
---
# <a name="connect-to-a-virtual-machine-scale-set-using-azure-bastion"></a>Se connecter à un groupe de machines virtuelles identiques à l’aide d’Azure Bastion

Cet article explique comment se connecter de manière fluide et sécurisée via RDP à l’instance du groupe de machines virtuelles Windows identiques, dans un réseau virtuel Azure à l’aide d’Azure Bastion. Vous pouvez vous connecter à une instance de groupe de machines virtuelles identiques, directement à partir du portail Azure. Lorsque vous utilisez Azure Bastion, les machines virtuelles n’ont pas besoin de client, d’agent ni de logiciel supplémentaire. Pour plus d’informations sur Azure Bastion, consultez la [vue d’ensemble](bastion-overview.md).

## <a name="prerequisites"></a>Prérequis

Assurez-vous que vous avez configuré un hôte Azure Bastion pour le réseau virtuel dans lequel réside le groupe de machines virtuelles identiques. Pour plus d’informations, consultez [Créer un hôte Azure Bastion](./tutorial-create-host-portal.md). Une fois que le service Bastion est provisionné et déployé dans votre réseau virtuel, vous pouvez l’utiliser pour vous connecter à une instance de groupe de machines virtuelles identiques dans ce réseau virtuel. Bastion part du principe que vous utilisez le protocole RDP pour vous connecter à un groupe de machines virtuelles Windows identiques, et le protocole SSH pour vous connecter à votre groupe de machines virtuelles Linux identiques. Pour plus d’informations sur la connexion à une machine virtuelle Linux, consultez [Connexion à une machine virtuelle - Linux](bastion-connect-vm-ssh.md).

## <a name="connect-using-rdp"></a><a name="rdp"></a>Connexion RDP

1. Ouvrez le [portail Azure](https://portal.azure.com). Accédez au groupe de machines virtuelles identiques auquel vous souhaitez vous connecter.

   ![naviguer](./media/bastion-connect-vm-scale-set/1.png)
2. Accédez à l’instance du groupe de machines virtuelles identiques à laquelle vous souhaitez vous connecter, puis sélectionnez **Se connecter**. Lors de l’utilisation d’une connexion RDP, le groupe de machines virtuelles identiques doit être constitué de machines Windows.

   ![groupe de machines virtuelles identiques](./media/bastion-connect-vm-scale-set/2.png)
3. Lorsque vous sélectionnez **Se connecter**, une barre latérale dotée de trois onglets s’affiche : RDP, SSH et Bastion. Sélectionnez l’onglet **Bastion** dans la barre latérale. Si vous n’avez pas provisionné Bastion pour le réseau virtuel, vous pouvez sélectionner le lien permettant de configurer Bastion. Pour en savoir plus sur la procédure de configuration, consultez [Configurer Bastion](./tutorial-create-host-portal.md).

   ![Onglet Bastion](./media/bastion-connect-vm-scale-set/3.png)
4. Dans l’onglet Bastion, entrez le nom d’utilisateur et le mot de passe de votre groupe de machines virtuelles identiques, puis sélectionnez **Se connecter**.

   ![se connecter](./media/bastion-connect-vm-scale-set/4.png)
5. La connexion RDP à cette machine virtuelle avec Bastion s’ouvrira directement dans le portail Azure (en HTML5) via le port 443 et le service Bastion.

## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).