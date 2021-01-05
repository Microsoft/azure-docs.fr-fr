---
title: Définir des mots de passe pour des machines virtuelles dans Azure Lab Services | Microsoft Docs
description: Découvrez comment définir et réinitialiser le mot de passe de machines virtuelles dans les labos d’Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 6ae577ee4c0c7e31760e0fb12afeaeac1ef8b7e2
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434224"
---
# <a name="set-up-and-manage-virtual-machine-pool"></a>Configurer et gérer un pool de machines virtuelles 
Cet article explique comment effectuer les tâches suivantes :

- Augmenter le nombre de machines virtuelles dans le laboratoire
- Démarrer toutes les machines virtuelles ou les machines virtuelles sélectionnées 
- Réinitialiser les machines virtuelles

## <a name="update-the-lab-capacity"></a>Mettre à jour la capacité du labo
Pour augmenter ou diminuer la capacité du labo (nombre de machines virtuelles dans un laboratoire), procédez comme suit :

1. Dans la page **Pool de machines virtuelles**, sélectionnez **Capacité du labo : &lt;nombre&gt; machines**.
2. Entrez le nouveau **nombre de machines virtuelles** que vous souhaitez dans le laboratoire. Ce nombre doit être supérieur ou égal au nombre d’utilisateurs inscrits dans le labo. 
3. Sélectionnez ensuite **Enregistrer**. 

    ![Capture d’écran montrant la fenêtre « Capacité du labo » avec les boutons « Nombre maximal de machines dans le laboratoire » et « Enregistrer » sélectionnés.](./media/how-to-set-virtual-machine-passwords/number-of-vms-in-lab.png)
4. Si vous avez augmenté la capacité, vous pouvez voir la ou les machines virtuelles en cours de création. Si vous ne voyez pas la nouvelle machine virtuelle dans la liste, actualisez la page. 

    ![Machine virtuelle en cours de création](./media/how-to-set-virtual-machine-passwords/vm-being-created.png)

## <a name="start-vms"></a>Start VMs

### <a name="start-ot-stop-all-vms"></a>Démarrer ou arrêter toutes les machines virtuelles
1. Basculez vers la page **Pool de machines virtuelles**. 
2. Sélectionnez **Démarrer tout** dans la barre d’outils. 

    ![Bouton Démarrer tout](./media/how-to-set-virtual-machine-passwords/start-all-vms-button.png)
3. Une fois que toutes les machines virtuelles ont démarré, vous pouvez toutes les arrêter en sélectionnant le bouton **Arrêter tout** dans la barre d’outils. 

    ![Bouton Arrêter tout](./media/how-to-set-virtual-machine-passwords/stop-all-vms-button.png)

### <a name="start-selected-vms"></a>Démarrer les machines virtuelles sélectionnées
Il existe deux façons de démarrer des machines virtuelles sélectionnées (une ou plusieurs). La première consiste à sélectionner la ou les machines virtuelles dans la liste, puis à sélectionner **Démarrer** dans la barre d’outils. 

La seconde consiste à sélectionner une ou plusieurs machines virtuelles dans la liste et à basculer le bouton dans la colonne **État**. 

![Démarrer les machines virtuelles sélectionnées](./media/how-to-set-virtual-machine-passwords/start-selected-vms.png)

De même, vous pouvez arrêter une ou plusieurs machines virtuelles en activant le bouton dans la colonne **État** ou en sélectionnant **Arrêter** dans la barre d’outils. 

> [!NOTE]
> Lorsqu’un formateur active la machine virtuelle d’un étudiant, le quota de l’étudiant n’est pas affecté. Le quota d’un utilisateur spécifie le nombre d’heures de labo disponibles pour l’utilisateur en dehors des heures de classe planifiées. Pour plus d’informations sur les quotas, consultez [Définir des quotas pour les utilisateurs](how-to-configure-student-usage.md?#set-quotas-for-users).

## <a name="reset-vms"></a>Réinitialiser les machines virtuelles

Pour réinitialiser une ou plusieurs machines virtuelles, sélectionnez-les dans la liste, puis sélectionnez **Réinitialiser** dans la barre d’outils. 

![Réinitialiser les machines virtuelles sélectionnées](./media/how-to-set-virtual-machine-passwords/reset-vm-button.png)

Dans la boîte de dialogue **Réinitialiser la ou les machines virtuelles**, sélectionnez **Réinitialiser**. 

![Boîte de dialogue Réinitialiser la machine virtuelle](./media/how-to-set-virtual-machine-passwords/reset-vms-dialog.png)

## <a name="set-password-for-vms"></a>Définir un mot de passe pour des machines virtuelles
Un propriétaire de laboratoire (formateur) peut définir/réinitialiser le mot de passe pour des machines virtuelles au moment de la création du labo (avec l’Assistant de création de labo) ou après l’avoir créé, sur la page **Modèle**. 

### <a name="set-password-at-the-time-of-lab-creation"></a>Définir le mot de passe au moment de la création du labo
Un propriétaire de laboratoire (formateur) peut définir un mot de passe pour des machines virtuelles dans le labo, sur la page **Informations d’identification de machine virtuelle** de l’Assistant de création de labo.

![Fenêtre Nouveau labo](./media/tutorial-setup-classroom-lab/virtual-machine-credentials.png)

En activant ou désactivant l’option **Use same password for all virtual machines** (Utiliser le même mot de passe pour toutes les machines virtuelles) figurant sur cette page, un formateur peut choisir d’utiliser le même mot de passe pour toutes les machines virtuelles du labo, ou autoriser les étudiants à définir leur propre mot de passe pour l’accès aux machines virtuelles. Par défaut, ce paramètre est activé pour toutes les images de systèmes d’exploitation Windows et Linux, à l’exception d’Ubuntu. Lorsque ce paramètre est désactivé, les étudiants sont invités à définir un mot de passe lorsqu’ils tentent de se connecter à la machine virtuelle pour la première fois. 

### <a name="reset-password-later"></a>Réinitialiser le mot de passe plus tard

1. Dans la page **Modèle** du laboratoire, sélectionnez **Réinitialiser le mot de passe** dans la barre d’outils. 
1. Dans la boîte de dialogue **Réinitialiser le mot de passe**, entrez un mot de passe et sélectionnez **Réinitialiser le mot de passe**.
    
    ![Boîte de dialogue Set Password (Définir un mot de passe)](./media/how-to-set-virtual-machine-passwords/set-password.png)

## <a name="connect-to-student-vms"></a>Se connecter à des machines virtuelles d’étudiant
Le créateur de laboratoire (formateur) peut se connecter à une machine virtuelle d’étudiant, si les conditions suivantes sont réunies : 

- L’option **Use same password for all virtual machines** (Utiliser le même mot de passe pour toutes les machines virtuelles) a été sélectionnée lors de la création du laboratoire
- La machine virtuelle est en cours d’exécution 

 Pour vous connecter à la machine virtuelle d’étudiant, pointez la souris sur la machine virtuelle dans la liste et sélectionnez le bouton ordinateur.  

![Se connecter au bouton de machines virtuelles d’étudiant](./media/how-to-set-virtual-machine-passwords/connect-student-vm.png)

> [!NOTE]
> Lorsque le formateur démarre la machine virtuelle et se connecte à celle-ci, le quota d’étudiants n’est pas modifié. 

## <a name="export-list-of-virtual-machines-to-a-csv-file"></a>Exporter la liste des machines virtuelles vers un fichier CSV

1. Basculez vers l’onglet **Pool de machines virtuelles**.
2. Sélectionnez **...** (ellipse) dans la barre d’outils, puis **Exporter au format CSV**. 

    ![Exporter la liste des machines virtuelles](./media/how-to-export-users-virtual-machines-csv/virtual-machines-export-csv.png)

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les autres options d’utilisation que vous pouvez configurer pour les étudiants (en tant que propriétaire du labo), consultez l’article suivant : [Configure student usage](how-to-configure-student-usage.md) (Configurer l’utilisation pour les étudiants).

Pour savoir comment les étudiants peuvent réinitialiser le mot de passe de leur machine virtuelle, consultez [Définition ou réinitialisation du mot de passe de machines virtuelles dans des labos (étudiants)](how-to-set-virtual-machine-passwords-student.md).