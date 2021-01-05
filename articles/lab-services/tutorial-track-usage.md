---
title: Suivre l’utilisation d’un laboratoire dans Azure Lab Services | Microsoft Docs
description: Dans ce didacticiel, vous (en tant que créateur et propriétaire d’un laboratoire) allez suivre l’utilisation de votre laboratoire.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: 8a3ca9e011eb89b3db8b202bab11d14f10d74e7e
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434615"
---
# <a name="tutorial-track-usage-of-a-lab-in-azure-lab-service"></a>Didacticiel : suivre l’utilisation d’un labo dans Azure Lab Service
Ce didacticiel vous montre comment un créateur ou propriétaire de laboratoire peut suivre l’utilisation d’un laboratoire.

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Afficher les utilisateurs inscrits à votre laboratoire
> * Afficher l’utilisation des machines virtuelles dans le laboratoire
> * Gérer les machines virtuelles des étudiants 


## <a name="view-registered-users"></a>Afficher les utilisateurs inscrits

1. Accédez au [site web Azure Lab Services](https://labs.azure.com). 
2. Sélectionnez **Se connecter** et entrez vos informations d’identification. Azure Lab Services prend en charge les comptes professionnels et les comptes Microsoft.
3. Sur la page **Mes laboratoires**, sélectionnez le laboratoire pour lequel vous souhaitez suivre l’utilisation. 
4. Sélectionnez **Utilisateurs** dans le menu de gauche ou la vignette **Utilisateurs**. Vous visualisez les étudiants inscrits à votre laboratoire.  

    ![Utilisateurs inscrits](./media/tutorial-track-usage/registered-users.png)

    Pour plus d’informations sur l’ajout et la gestion d’utilisateurs du labo, consultez [Ajouter et gérer des utilisateurs du labo](how-to-configure-student-usage.md).

## <a name="view-the-usage-of-vms"></a>Afficher l’utilisation des machines virtuelles

1. Dans le menu de gauche, sélectionnez **Machines virtuelles**. 
2. Confirmez que vous visualisez bien l’état des machines virtuelles ainsi que le nombre d’heures d’exécution de celles-ci. Le temps qu’un propriétaire de laboratoire passe sur la machine virtuelle d’un étudiant n’est pas comptabilisé dans le temps d’utilisation indiqué dans la dernière colonne. 

    ![Utilisation de machines virtuelles](./media/tutorial-track-usage/vm-usage.png)

## <a name="manage-student-vms"></a>Gérer les machines virtuelles des étudiants 
Dans cette page, vous pouvez démarrer, arrêter ou réinitialiser les machines virtuelles des étudiants en utilisant les contrôles de la colonne **État** ou de la barre d’outils.

![Actions de machine virtuelle](./media/tutorial-track-usage/vm-controls.png)

Pour plus d’informations sur la gestion des pools de machines virtuelles du labo, consultez [Configurer et gérer un pool de machines virtuelles](how-to-set-virtual-machine-passwords.md).

> [!NOTE]
> Lorsqu’un formateur active une machine virtuelle d’un étudiant, le quota de l’étudiant n’est pas affecté. Le quota d’un utilisateur spécifie le nombre d’heures de labo disponibles pour l’utilisateur en dehors des heures de classe planifiées. Pour plus d’informations sur les quotas, consultez [Définir des quotas pour les utilisateurs](how-to-configure-student-usage.md?#set-quotas-for-users).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les labos, consultez les articles de la section [Guides pratiques](how-to-manage-lab-accounts.md).
