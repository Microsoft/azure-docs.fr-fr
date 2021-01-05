---
title: Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services (script) | Microsoft Docs
description: Découvrez comment créer un modèle de machine virtuelle contenant plusieurs machines virtuelles en utilisant un script.  En d’autres termes, activez la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 5ae50bd11ab9a8adb769920f6d473a2ff2ce9342
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91251493"
---
# <a name="enable-nested-virtualization-on-a-template-virtual-machine-in-azure-lab-services-using-a-script"></a>Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services à l’aide d’un script

La virtualisation imbriquée vous permet de créer un environnement avec plusieurs machines virtuelles au sein de la machine virtuelle d’un modèle de labo. La publication du modèle fournit à chaque utilisateur du labo une machine virtuelle configurée avec plusieurs machines virtuelles en son sein.  Pour plus d’informations sur la virtualisation imbriquée et Azure Lab Services, consultez [Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services](how-to-enable-nested-virtualization-template-vm.md).

Les étapes de cet article traitent de la configuration de la virtualisation imbriquée pour Windows Server 2016, Windows Server 2019 ou Windows 10. Vous allez utiliser un script pour configurer la machine modèle avec Hyper-V.  Les étapes suivantes vous guident tout au long de l’utilisation des [scripts Hyper-V Lab Services](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/HyperV).

>[!IMPORTANT]
>Sélectionnez **Grande (virtualisation imbriquée)** ou **Moyenne (virtualisation imbriquée)** pour la taille de la machine virtuelle lorsque vous créez le labo.  Sinon, la virtualisation imbriquée ne fonctionnera pas.  

## <a name="run-script"></a>Exécuter le script

1. Si vous utilisez Internet Explorer, vous devrez peut-être ajouter `https://github.com` à la liste des sites de confiance.
    1. Ouvrez Internet Explorer.
    1. Sélectionnez l’icône en forme de roue dentée, puis choisissez **Options Internet**.  
    1. Quand la boîte de dialogue **Options Internet** s’affiche, sélectionnez **Sécurité**, sélectionnez **Sites de confiance**, puis cliquez sur le bouton **Sites**.
    1. Quand la boîte de dialogue **Sites de confiance** s’affiche, ajoutez `https://github.com` à la liste des sites web de confiance, puis sélectionnez **Fermer**.

        ![Sites de confiance](./media/how-to-enable-nested-virtualization-template-vm-using-script/trusted-sites-dialog.png)
1. Téléchargez les fichiers du dépôt Git, comme indiqué dans les étapes suivantes.
    1. Accédez à  [https://github.com/Azure/azure-devtestlab/](https://github.com/Azure/azure-devtestlab/).
    1. Cliquez sur le bouton **Clone ou Download** (Cloner ou télécharger).
    1. Cliquez sur **Download ZIP** (Télécharger le fichier zip).
    1. Extrayez le fichier zip.

    >[!TIP]
    >Vous pouvez également cloner le référentiel Git à l’adresse [https://github.com/Azure/azure-devtestlab.git](https://github.com/Azure/azure-devtestlab.git).

1. Lancez **Azure PowerShell** en mode **Administrateur**.
1. Dans la fenêtre PowerShell, accédez au dossier contenant le script téléchargé. Si vous partez du premier dossier des fichiers du dépôt, le script se trouve à l’emplacement `azure-devtestlab\samples\ClassroomLabs\Scripts\HyperV\`.
1. Vous devrez peut-être modifier la stratégie d’exécution pour exécuter le script. Exécutez la commande suivante :

    ```powershell
    Set-ExecutionPolicy bypass -force
    ```

1. Exécutez le script :

    ```powershell
    .\SetupForNestedVirtualization.ps1
    ```

    > [!NOTE]
    > Le script peut nécessiter le redémarrage de la machine. Suivez les instructions du script, puis réexécutez-le jusqu’à ce que **Script completed** (Exécution du script terminée) s’affiche dans la sortie.
1. N’oubliez pas de réinitialiser la stratégie d’exécution. Exécutez la commande suivante :

    ```powershell
    Set-ExecutionPolicy default -force
    ```

## <a name="conclusion"></a>Conclusion

Votre machine modèle est maintenant prête pour la création de machines virtuelles Hyper-V. Pour obtenir des instructions sur la création de machines virtuelles Hyper-V, consultez [Créer une machine virtuelle dans Hyper-V](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v). Consultez également le [Centre d’évaluation Microsoft](https://www.microsoft.com/evalcenter/) pour voir les systèmes d’exploitation et les logiciels qui sont disponibles.  

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)