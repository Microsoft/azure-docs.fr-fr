---
title: 'Copier et coller vers et depuis une machine virtuelle : Azure Bastion'
description: Dans cet article, découvrez comment copier et coller vers et depuis une machine virtuelle Azure à l’aide de Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cherylmc
ms.openlocfilehash: 50f7906992aa19daa205a30f71ce21456bafe558
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92079154"
---
# <a name="copy-and-paste-to-a-virtual-machine-azure-bastion"></a>Copier et coller vers une machine virtuelle : Azure Bastion

Cet article vous aide à copier et coller du texte vers et depuis des machines virtuelles à l’aide d’Azure Bastion. Avant d’utiliser une machine virtuelle, assurez-vous d’avoir suivi les étapes de [Créer un hôte Bastion](./tutorial-create-host-portal.md). Connectez-vous ensuite à la machine virtuelle que vous souhaitez utiliser à l’aide de [RDP](bastion-connect-vm-rdp.md) ou de [SSH](bastion-connect-vm-ssh.md).

Pour les navigateurs qui prennent en charge l’accès à l’API de Presse-papiers avancé, vous pouvez copier et coller du texte entre votre appareil local et la session à distance comme si vous le faisiez entre des applications sur votre appareil local. Pour les autres navigateurs, vous pouvez utiliser la palette d’outils d’accès au Presse-papiers Bastion.

>[!NOTE]
>Seul le copier/coller de texte est pris en charge.
>

   ![Autoriser le Presse-papiers](./media/bastion-vm-manage/allow.png)

Seul le copier/coller de texte est pris en charge. Pour les opérations de copie et collage directs, votre navigateur peut vous demander l’accès au Presse-papiers lors de l’initialisation de la session Bastion. **Autorisez** la page web à accéder au Presse-papiers.

## <a name="copy-to-a-remote-session"></a><a name="to"></a>Copier vers une session à distance

Une fois connecté à la machine virtuelle à l’aide du [portail Azure](https://portal.azure.com), procédez comme suit :

1. Copiez le texte/contenu de l’appareil local vers le Presse-papiers local.
1. Au cours de la session à distance, lancez la palette d’outils d’accès au Presse-papiers Bastion en sélectionnant les deux flèches. Les flèches sont situées à gauche au centre de la session.

   ![Capture d’écran montrant les flèches de lancement de la palette d’outils en évidence dans la partie gauche de la fenêtre.](./media/bastion-vm-manage/left.png)

   ![Capture d’écran montrant un presse-papiers pour le texte copié dans bastion.](./media/bastion-vm-manage/clipboard.png)
1. En règle générale, le texte copié s’affiche automatiquement dans la palette copier/coller Bastion. Si votre texte n’y apparaît pas, collez le texte dans la zone de texte de la palette.
1. Lorsque le texte est présent dans la zone de texte, vous pouvez le coller dans la session à distance.

   ![Capture d’écran montrant le bouton copier/coller mis en évidence et un exemple de chaîne de texte copié dans la session à distance.](./media/bastion-vm-manage/local.png)

## <a name="copy-from-a-remote-session"></a><a name="from"></a>Copier depuis une session à distance

Une fois connecté à la machine virtuelle à l’aide du [portail Azure](https://portal.azure.com), procédez comme suit :

1. Copiez le texte/contenu depuis la session à distance dans le Presse-papiers à distance (à l’aide de Ctrl-C).

   ![palette d’outils](./media/bastion-vm-manage/remote.png)
1. Au cours de la session à distance, lancez la palette d’outils d’accès au Presse-papiers Bastion en sélectionnant les deux flèches. Les flèches sont situées à gauche au centre de la session.

   ![presse-papiers](./media/bastion-vm-manage/clipboard2.png)
1. En règle générale, le texte copié s’affiche automatiquement dans la palette copier/coller Bastion. Si votre texte n’y apparaît pas, collez le texte dans la zone de texte de la palette.
1. Lorsque le texte est présent dans la zone de texte, vous pouvez le coller dans l’appareil local.

   ![coller](./media/bastion-vm-manage/local2.png)
 
## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).
