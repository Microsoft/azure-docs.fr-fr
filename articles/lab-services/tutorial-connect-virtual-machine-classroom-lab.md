---
title: Accéder à un laboratoire de classe dans Azure Lab Services | Microsoft Docs
description: Dans ce tutoriel, vous accédez aux machines virtuelles dans un laboratoire de classe qui est configuré par un formateur.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: dacfa34c0d3ab637ef513342bc5ce5fe81038e11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85443466"
---
# <a name="tutorial-access-a-classroom-lab-in-azure-lab-services"></a>Tutoriel : Accéder à un laboratoire de salle de classe dans Azure Lab Services
Dans ce didacticiel, vous vous connectez en tant qu’étudiant à une machine virtuelle dans un laboratoire de classe. 

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Vous inscrire au labo
> * Démarrer la machine virtuelle
> * Connexion à la machine virtuelle

## <a name="register-to-the-lab"></a>Vous inscrire au labo

1. Accédez à **l’URL d’inscription** que vous avez reçue du formateur. Vous n’avez pas besoin d’utiliser l’URL d’inscription après avoir effectué l’enregistrement. À la place, utilisez l’URL : [https://labs.azure.com](https://labs.azure.com). Internet Explorer 11 n’est pas encore pris en charge. 

    ![Vous inscrire au labo](./media/tutorial-connect-vm-in-classroom-lab/register-lab.png)
1. Connectez-vous au service à l’aide de votre compte scolaire pour terminer l’inscription. 

    > [!NOTE]
    > L’utilisation d’Azure Lab Services nécessite un compte Microsoft. Si vous souhaitez vous connecter au portail avec un compte autre que Microsoft, tel qu’un compte Yahoo ou Google, suivez les instructions permettant de créer un compte Microsoft en vue de l’associer à votre compte non Microsoft. Ensuite, suivez les étapes pour terminer le processus d’inscription. 
1. Une fois l’inscription terminée, confirmez que vous visualisez bien les machines virtuelles du laboratoire auquel vous avez accès. 

    ![Machines virtuelles accessibles](./media/tutorial-connect-vm-in-classroom-lab/accessible-vms.png)
1. Attendez que la machine virtuelle soit prête. Vous pouvez voir les champs suivants sur la vignette de la machine virtuelle :
    1. En haut de la vignette, vous voyez le **nom du labo**.
    1. À droite, vous voyez l’icône représentant le **système d’exploitation (SE)** de la machine virtuelle. Dans cet exemple, il s’agit du système d’exploitation Windows. 
    1. La barre de progression sur la vignette indique le nombre d’heures d’utilisation par rapport aux [heures de quota](how-to-configure-student-usage.md#set-quotas-for-users) qui vous sont attribuées. Il s’agit du temps qui vous est alloué en plus du temps planifié pour le labo. 
    1. Les icônes/boutons situés en bas de la vignette vous permettent de démarrer/d’arrêter la machine virtuelle et de vous y connecter. 
    1. À droite des boutons, vous voyez l’état de la machine virtuelle. Vérifiez que l’état de la machine virtuelle est **Arrêtée**. 

        ![Machine virtuelle à l’état Arrêtée](./media/tutorial-connect-vm-in-classroom-lab/vm-in-stopped-state.png)

## <a name="start-the-vm"></a>Démarrer la machine virtuelle
1. **Démarrez** la machine virtuelle en cliquant sur le premier bouton comme illustré dans l’image suivante. Ce processus prend un certain temps.  

    ![Démarrer la machine virtuelle](./media/tutorial-connect-vm-in-classroom-lab/start-vm.png)
4. Vérifiez que l’état de la machine virtuelle est **En cours d’exécution**. 

    ![Machine virtuelle à l’état En cours d’exécution](./media/tutorial-connect-vm-in-classroom-lab/vm-running.png)

    Notez que l’icône du premier bouton a changé et qu’elle représente maintenant une action d’**arrêt**. Vous pouvez cliquer sur ce bouton pour arrêter la machine virtuelle. 

## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

1. Cliquez sur le deuxième bouton comme illustré dans l’image suivante pour vous **connecter** à la machine virtuelle du labo. 

    ![Se connecter à une machine virtuelle](./media/tutorial-connect-vm-in-classroom-lab/connect-vm.png)
2. Effectuez l’une des étapes suivantes : 
    1. Pour les machines virtuelles **Windows**, enregistrez le fichier **RDP** sur le disque dur. Ouvrez le fichier RDP pour vous connecter à la machine virtuelle. Utilisez le **nom d’utilisateur** et le **mot de passe** obtenus auprès de votre formateur pour vous connecter à la machine. 
    3. Pour les machines virtuelles **Linux**, vous pouvez utiliser **SSH** ou **RDP** (s’il est activé) pour vous y connecter. Pour plus d’informations, consultez [Activer la connexion Bureau à distance pour les machines Linux](how-to-enable-remote-desktop-linux.md). 

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez accédé à un laboratoire de classe à l’aide du lien d’inscription obtenu auprès de votre formateur.

En tant que propriétaire de laboratoire, vous souhaitez voir qui y est inscrit et suivre l’utilisation des machines virtuelles. Passez au tutoriel suivant pour savoir comment effectuer le suivi de l’utilisation du laboratoire :

> [!div class="nextstepaction"]
> [Suivre l’utilisation d’un laboratoire](tutorial-track-usage.md) 
