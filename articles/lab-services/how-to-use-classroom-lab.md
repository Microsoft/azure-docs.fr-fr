---
title: Accéder à un laboratoire de classe dans Azure Lab Services | Microsoft Docs
description: Découvrez comment s’inscrire à un labo de salle de classe, voir tous les labos auxquels vous pouvez accéder, démarrer/arrêter une machine virtuelle dans le labo et vous connecter à la machine virtuelle.
ms.topic: how-to
ms.date: 06/26/2020
ms.openlocfilehash: 6d5585082473a363df5f25f331475536544ae828
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433136"
---
# <a name="how-to-access-a-classroom-lab-in-azure-lab-services"></a>Accéder à un laboratoire de classe dans Azure Lab Services
Cet article explique comment vous inscrire à un labo de salle de classe, voir tous les labos auxquels vous pouvez accéder, démarrer/arrêter une machine virtuelle dans le labo et vous connecter à la machine virtuelle. 

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
    1. Les icônes/boutons situés en bas de la vignette vous permettent de démarrer/d’arrêter la machine virtuelle et de vous y connecter. 
    1. À droite des boutons, vous voyez l’état de la machine virtuelle. Vérifiez que l’état de la machine virtuelle est **Arrêtée**.

        ![Machine virtuelle à l’état Arrêtée](./media/tutorial-connect-vm-in-classroom-lab/vm-in-stopped-state.png)

## <a name="start-or-stop-the-vm"></a>Démarrer ou arrêter la machine virtuelle
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
    1. Si vous utilisez un **Mac** pour vous connecter à la machine virtuelle de laboratoire, suivez les instructions de la section suivante. 

## <a name="progress-bar"></a>Barre de progression 
La barre de progression sur la vignette indique le nombre d’heures d’utilisation par rapport aux [heures de quota](how-to-configure-student-usage.md#set-quotas-for-users) qui vous sont attribuées. Il s’agit du temps qui vous est alloué en plus du temps planifié pour le labo. La couleur de la barre de progression et le texte apparaissant en dessous varient selon les scénarios suivants :

- Si une classe (planifiée) est en cours, la barre de progression est grisée. Ceci indique que les heures de quota ne sont pas utilisées. 

    ![Barre de progression grisée](./media/tutorial-connect-vm-in-classroom-lab/progress-bar-class-in-progress.png)
- Si aucun quota n’est attribué (zéro heure), le texte **Disponible dans le cadre des classes uniquement** apparaît à la place de la barre de progression. 
    
    ![État quand aucun quota n’est défini](./media/tutorial-connect-vm-in-classroom-lab/available-during-class.png)
- Si **votre quota est épuisé**, la barre de progression s’affiche en **rouge**. 

    ![Barre de progression en rouge](./media/tutorial-connect-vm-in-classroom-lab/progress-bar-red-color.png)
- La barre de progression s’affiche en **bleu** quand le labo ne se déroule pas pendant les heures planifiées et qu’une partie des heures de quota a été utilisée. 

    ![Barre de progression en bleu](./media/tutorial-connect-vm-in-classroom-lab/progress-bar-blue-color.png)


## <a name="view-all-the-labs"></a>Afficher tous les labos
Une fois votre inscription effectuée, procédez comme suit pour voir tous les labos : 

1. Accédez à [https://labs.azure.com](https://labs.azure.com). Internet Explorer 11 n’est pas encore pris en charge. 
2. Connectez-vous au service à l’aide du compte d’utilisateur que vous avez utilisé pour vous inscrire au labo. 
3. Confirmez que vous voyez tous les labos auxquels vous avez accès. 

    ![Afficher tous les laboratoires](./media/how-to-manage-classroom-labs/all-labs.png)


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [En tant qu’administrateur, créer et gérer des comptes de labo](how-to-manage-lab-accounts.md)
- [En tant que propriétaire de labo, créer et gérer des labos](how-to-manage-classroom-labs.md)
- [En tant que propriétaire de labo, configurer et publier des modèles](how-to-create-manage-template.md)
- [En tant que propriétaire de labo, configurer et contrôler l’utilisation d’un labo](how-to-configure-student-usage.md)
 