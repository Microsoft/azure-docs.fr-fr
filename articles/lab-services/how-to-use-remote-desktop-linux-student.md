---
title: Se connecter à une machine virtuelle dans Azure Lab Services | Microsoft Docs
description: Découvrez comment utiliser le Bureau à distance pour les machines virtuelles Linux dans un labo dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: d13868477ff2e3378d87d7785789a7498ed17e59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85443415"
---
# <a name="connect-to-linux-virtual-machines-in-a-classroom-lab-of-azure-lab-services"></a>Se connecter aux machines virtuelles Linux dans un labo de classe d’Azure Lab Services
Cet article explique comment les étudiants peuvent se connecter à une machine virtuelle Linux d’un labo en utilisant :
- le terminal SSH (Secure Shell Protocol)
- le Bureau à distance Gui (interface graphique utilisateur)

> [!IMPORTANT] 
> SSH est configuré automatiquement afin que les étudiants et l’instructeur puissent se faire au format SSH sur des machines virtuelles Linux sans aucune configuration supplémentaire. Toutefois, si les étudiants doivent se connecter à l’aide d’un bureau à distance GUI, l’instructeur devra peut-être effectuer une installation supplémentaire.  Pour plus d’informations, consultez [Activer le Bureau à distance pour les machines virtuelles Linux](how-to-enable-remote-desktop-linux.md).

## <a name="connect-to-the-student-vm-using-ssh"></a>Se connecter à la machine virtuelle de l’étudiant à l’aide de SSH

1. Lorsqu’un étudiant se connecte directement au portail Labs (`https://labs.azure.com`) ou à l'aide d'un lien d’inscription (`https://labs.azure.com/register/<registrationCode>`), une vignette correspondant à chaque lab auquel l'étudiant a accès s'affiche. 
   
1. Sur la vignette, basculez le bouton pour démarrer la machine virtuelle si elle est en état arrêté. 

2. Sélectionnez **Connecter**. Deux options de connexion à la machine virtuelle vous sont proposées : **SSH** et **RDP**.

    ![Machine virtuelle d'étudiant - options de connexion](./media/how-to-enable-remote-desktop-linux/student-vm-connect-options.png)

3. Sélectionnez l'option **SSH** pour afficher la boîte de dialogue **Se connecter à votre machine virtuelle** :  

    ![Chaîne de connexion SSH](./media/how-to-enable-remote-desktop-linux/ssh-connection-string.png)

4. Cliquez sur le bouton **Copier** en regard de la zone de texte pour copier les informations de connexion SSH dans le presse-papiers. 

5. Enregistrez les informations de connexion SSH, par exemple dans le panneau Texte, afin de pouvoir utiliser ces informations de connexion à l’étape suivante.

6. À partir d’un terminal SSH (comme [Putty](https://www.putty.org/)), connectez-vous à votre machine virtuelle.

## <a name="connect-to-the-student-vm-using-gui-remote-desktop"></a>Se connecter à la machine virtuelle de l’étudiant à l’aide du bureau à distance GUI
L’instructeur peut choisir de configurer des machines virtuelles afin que les étudiants puissent également se connecter à l’aide d’un bureau à distance GUI.  Dans ce cas, les étudiants doivent comprendre auprès de leur instructeur s’il faut se connecter à leurs machines virtuelles à l’aide du **Bureau à distance Microsoft (RDP)** ou de l’application cliente **X2Go**.  Ces deux applications permettent à un étudiant de se connecter à distance à sa machine virtuelle et d’afficher le bureau graphique Linux sur son ordinateur local.

### <a name="connect-to-the-student-vm-using-microsoft-remote-desktop-rdp"></a>Se connecter à la machine virtuelle de l’étudiant à l’aide du bureau à distance Microsoft (RDP)
Les étudiants peuvent utiliser le Bureau à distance Microsoft (RDP) pour se connecter à leurs machines virtuelles Linux une fois que leur instructeur a configuré leur labo avec RDP et des packages d’interface graphique utilisateur pour un environnement de bureau graphique Linux (par exemple, MATE, XFCE, etc.). Voici la procédure à suivre pour se connecter : 

1. Sur la mosaïque de votre machine virtuelle, assurez-vous que la machine virtuelle est en cours d’exécution, puis cliquez sur **Se connecter**. Deux options de connexion à la machine virtuelle vous sont proposées : **SSH** et **RDP**.

    ![Machine virtuelle d'étudiant - options de connexion](./media/how-to-enable-remote-desktop-linux/student-vm-connect-options.png)
2. Sélectionnez l’option **RDP**.  Une fois le fichier RDP téléchargé sur votre ordinateur, enregistrez-le sur votre machine virtuelle.

3. Si vous vous connectez à partir d’un ordinateur Windows, en général, le client Bureau à distance Microsoft (RDP) est déjà installé et configuré.  Par conséquent, il vous suffit de cliquer sur le fichier RDP pour l’ouvrir et démarrer la session à distance.

    Par contre, si vous vous connectez à partit d’un Mac ou Chromebook, procédez comme suit :
   - [Se connecter à une machine virtuelle à l’aide de RDP sur un Mac](connect-virtual-machine-mac-remote-desktop.md).
   - [Se connecter à une machine virtuelle à l’aide de RDP sur un Chromebook](connect-virtual-machine-chromebook-remote-desktop.md).  

### <a name="connect-to-the-student-vm-using-x2go"></a>Se connecter à la machine virtuelle de l’étudiant à l’aide de X2Go
Les étudiants peuvent utiliser X2Go pour se connecter à leurs machines virtuelles Linux une fois que leur instructeur a configuré leur labo avec X2Go et des packages d’interface graphique utilisateur pour un environnement de bureau graphique Linux (par exemple, MATE, XFCE, etc.).

L’instructeur doit indiquer aux étudiants l’environnement de bureau graphique Linux qu’il a installé.  Ces informations sont nécessaires dans les étapes suivantes pour la connexion à l’aide du client X2Go.

1. Installez le [client X2Go](https://wiki.x2go.org/doku.php/doc:installation:x2goclient) sur votre ordinateur local.

1. Suivez les instructions de la [première section](how-to-use-remote-desktop-linux-student.md#connect-to-the-student-vm-using-ssh) pour copier les informations de connexion SSH pour votre machine virtuelle.  Ces informations sont utiles pour se connecter à l’aide du client X2Go.

1. Une fois que vous disposez des informations de connexion SSH, ouvrez le client X2Go et sélectionnez **Session** > **Nouvelle session**.
   ![Créer une nouvelle session X2Go](./media/how-to-use-classroom-lab/x2go-new-session.png)

1. Entrez les valeurs dans le volet **Préférences de session** en fonction des informations de connexion SSH.  Par exemple, vos informations de connexion se présentent comme suit :

    ```bash
    ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
    ```

    En suivant cet exemple, les valeurs ci-dessous sont entrées :

   - **Nom de session** : spécifiez un nom, tel que le nom de votre machine virtuelle.
   - **Hôte** : ID de votre machine virtuelle ; par exemple, **`ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com`** .
   - **Session** : nom d’utilisateur de votre machine virtuelle ; par exemple, **Étudiant**.
   - **Port SSH** : port unique affecté à votre machine virtuelle ; par exemple, **12345**.
   - **Type de session** : sélectionnez l’environnement de bureau graphique Linux sur lequel votre instructeur a configuré votre machine virtuelle.  Vous devez vous procurer ces informations auprès de votre instructeur.

    Enfin, cliquez sur **OK** pour créer la session.

    ![Préférences de session X2Go](./media/how-to-use-classroom-lab/x2go-session-preferences.png)

1.  Dans le volet de droite, cliquez sur votre session.

    ![Démarrer une nouvelle session X2Go](./media/how-to-use-classroom-lab/x2go-start-session.png)

    > [!NOTE] 
    > Si vous y êtes invité par un message similaire à celui-ci, sélectionnez **Oui** pour continuer à entrer votre mot de passe : Impossible d’établir **l’authenticité de l’hôte « [`00000000-0000-0000-0000-000000000000.eastus2.cloudapp.eastus.cloudapp.azure.com`] : 12345 ».  L’empreinte digitale de la clé ECDSA est SHA256:00000000000000000000000000000000000000000000. voulez-vous vraiment continuer à vous connecter (oui/non) ?**

2. Entrez votre mot de passe et cliquez sur **OK** lorsque vous y êtes invité.  Vous êtes maintenant connecté à distance à l’environnement de bureau de l’interface utilisateur graphique de votre machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur l’activation de la fonctionnalité de connexion du Bureau à distance pour les machines virtuelles Linux d’un labo de classe, consultez [Activer le Bureau à distance pour les machines virtuelles Linux](how-to-enable-remote-desktop-linux.md). 

