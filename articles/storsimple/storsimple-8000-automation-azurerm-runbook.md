---
title: Utiliser un runbook Azure Automation pour gérer des appareils StorSimple
description: Découvrez comment utiliser le Runbook Azure Automation pour gérer votre appareil StorSimple série 8000 dans le portail Azure.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/23/2017
ms.author: alkohli
ms.openlocfilehash: b5f2a86fb7c776436ea707ef293a66c033014c33
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021076"
---
# <a name="use-azure-automation-runbooks-to-manage-storsimple-devices"></a>Utiliser des runbooks Azure Automation pour gérer des appareils StorSimple

Cet article décrit comment des runbooks Azure Automation vous permettent de gérer votre appareil StorSimple série 8000 dans le portail Azure. Un exemple de runbook est fourni pour vous guider au travers des étapes de configuration de votre environnement pour exécuter ce runbook.


## <a name="configure-add-and-run-azure-runbook"></a>Configurer, ajouter et exécuter un runbook Azure

Cette section prend un exemple de script Windows PowerShell pour StorSimple, et décrit en détail les différentes étapes requises pour importer le script dans un runbook, puis publier et exécuter celui-ci.

### <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, assurez-vous de satisfaire les exigences suivantes :

* un abonnement Azure actif associé à votre service StorSimple Device Manager inscrit avec un appareil StorSimple série 8000 ;

* Windows PowerShell 5.0 installé sur votre ordinateur (ou votre hôte Windows Server pour votre StorSimple si vous en utilisez un).

### <a name="create-automation-runbook-module-in-windows-powershell"></a>Créer un module de runbook Automation dans Windows PowerShell

Pour créer un module Automation pour la gestion d’appareils StorSimple série 8000, procédez comme suit :

1. Démarrez Windows PowerShell. Créez un dossier et basculez vers ce nouveau dossier.

    ```powershell
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```

2. [Télécharger l’interface de ligne de commande NuGet](https://www.nuget.org/downloads) sous le dossier créé à l’étape précédente. Il existe différentes versions de _nuget.exe_. Choisissez la version correspondant à votre kit de développement logiciel (SDK). Chaque lien de téléchargement pointe directement vers un fichier _.exe_. Veillez à cliquer avec le bouton droit et à enregistrer le fichier sur votre ordinateur plutôt que de l’exécuter à partir du navigateur.

    Vous pouvez également exécuter la commande suivante pour télécharger et stocker le script dans le même dossier que vous avez créé précédemment.

    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```

3. Téléchargez le kit de développement logiciel (SDK) dépendant.

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```

4. Téléchargez le script à partir de l’exemple de projet GitHub.

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1 -Out Monitor-Backups.ps1
    ```

5. Créez un module Runbook Azure Automation pour la gestion des appareils StorSimple série 8000. Dans la fenêtre de Windows Powershell, tapez les commandes suivantes :

    ```powershell
        # set path variables
        $downloadDir = "C:\scripts\StorSimpleSDKTools"
        $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"

        #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
        mkdir "$moduleDir"
        Copy-Item "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
        Copy-Item "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
        Copy-Item "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir

        #Don't change the name of the Archive
        compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip
    ```

6. Vérifiez qu’un fichier zip de module Automation est créé dans `C:\scripts\StorSimpleSDKTools`.

    ![Capture d’écran montrant le dossier StorSimpleSDKTools avec le fichier zip du module Automation.](./media/storsimple-8000-automation-azurerm-runbook/verify-automation-module.png)

7. La sortie suivante s’affiche une fois le module Automation créé via Windows PowerShell.

    ```powershell
    mkdir C:\scripts\StorSimpleSDKTools
    ```

    ```Output
        Directory: C:\scripts

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:43 AM                StorSimpleSDKTools
    ```

    ```powershell
    wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.Azure.Management.Storsimple8000series 1.0.0' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 1.77 sec
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.IdentityModel.Clients.ActiveDirectory 2.28.3' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 927.64 ms
    ```

    ```powershell
    C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```

    ```Output
    -------------------------------------------
    CUT              CUT
    -------------------------------------------
    Successfully installed 'Microsoft.Rest.ClientRuntime.Azure.Authentication 2.2.9-preview' to C:\scripts\StorSimpleSDKTools
    Executing nuget actions took 717.48 ms
    ```

    ```powershell
    wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1 -Out Monitor-Backups.ps1
    # set path variables
    $downloadDir = "C:\scripts\StorSimpleSDKTools"
    $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"
    #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
    mkdir "$moduleDir"
    ```

    ```Output
        Directory: C:\scripts\StorSimpleSDKTools\AutomationModule

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/18/2017   8:48 AM                Microsoft.Azure.Management.StorSimple8000Series
    ```

    ```powershell
    Copy-Item "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
    Copy-Item "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
    Copy-Item "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir
    #Don't change the name of the Archive
    compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip
    ```

### <a name="import-publish-and-run-automation-runbook"></a>Importer, publier et exécuter un runbook Automation

1. Créez un compte Automation d’identification Azure dans le portail Azure. Pour ce faire, accédez à **Place de marché Azure > Tout**, puis recherchez **Automation**. Sélectionnez **Comptes Automation**.

    ![rechercher-automation](./media/storsimple-8000-automation-azurerm-runbook/automation1.png)

2. Dans le panneau **Ajouter un compte Automation** :

   1. Entrez le **Nom** de votre compte Automation.
   2. Sélectionnez l’**Abonnement** lié à votre service StorSimple Device Manager.
   3. Créez un groupe de ressources ou sélectionnez un groupe de ressources existant.
   4. Sélectionnez un **Emplacement** (si possible celui où votre service est en cours d’exécution).
   5. Laissez l’option par défaut **Créer un compte d’identification** activée.
   6. Vous pouvez également activer l’option **Épingler au tableau de bord**. Cliquez sur **Créer**.

       ![Capture d’écran montrant le volet Ajouter un compte Automation avec des valeurs pour Abonnement, Créer un compte d’identification Azure et Créer 2](./media/storsimple-8000-automation-azurerm-runbook/create-automation-account.png)

      Une fois le compte Automation créé, vous êtes averti. Pour plus d’informations sur la façon de créer un compte Automation, accédez à [Créer un compte d’identification](../automation/manage-runas-account.md).

3. Pour vous assurer que le compte Automation créé peut accéder au service StorSimple Device Manager, vous devez assigner les autorisations appropriées au compte Automation. Accédez à **Contrôle d’accès** dans votre service StorSimple Device Manager. Cliquez sur **+ Ajouter**, puis entrez le nom de votre compte Azure Automation. **Enregistrez** les paramètres.

    ![ajout-autorisations-compte-automation](./media/storsimple-8000-automation-azurerm-runbook/goto-add-roles.png)

4. Dans le compte que vous venez de créer, accédez à **Ressources partagées > Modules**, puis cliquez sur **+ Ajouter un module**.

5. Dans le panneau **Ajouter un module**, accédez à l’emplacement du module zippé, puis sélectionnez et ouvrez le module. Cliquez sur **OK**.

    ![Capture d’écran montrant le volet Ajouter un module avec une valeur Charger un fichier et OK.](./media/storsimple-8000-automation-azurerm-runbook/add-module.png)

6. Accédez à **Automatisation des processus > Runbooks, puis cliquez sur + Ajouter un runbook**. Dans le panneau **Ajouter un runbook**, cliquez sur **Importer un runbook existant**. Pointez sur le fichier de script Windows PowerShell pour le **Fichier de runbook**. Le type de runbook est sélectionné automatiquement. Fournissez un nom et une description facultative pour le runbook. Cliquez sur **Créer**.

    ![ajouter-module](./media/storsimple-8000-automation-azurerm-runbook/import-runbook.png)

7. Le runbook est ajouté à la liste des runbooks. Sélectionnez et cliquez sur ce runbook.

    ![cliquer-nouveau-runbook](./media/storsimple-8000-automation-azurerm-runbook/verify-runbook-created.png)

8. Modifiez le runbook, puis cliquez sur **Volet de test**. Fournissez les paramètres tels que le nom de votre service StorSimple Device Manager, le nom de l’appareil StorSimple et l’abonnement. **Démarrez** le test. Le rapport est généré une fois l’exécution du test terminée. Pour plus d’informations, accédez à [Comment tester un runbook](../automation/learn/automation-tutorial-runbook-textual-powershell.md#step-3---test-the-runbook).

    ![Capture d’écran montrant le volet Test dans lequel vous pouvez entrer des valeurs de paramètre et démarrer un test.](./media/storsimple-8000-automation-azurerm-runbook/test-runbook.png)

9. Inspectez la sortie du runbook dans le volet de test. Si vous êtes satisfait, fermez le volet. Cliquez sur **Publier**, puis, lorsque vous êtes invité à confirmer l’opération, confirmez-la et publiez le runbook.

    ![Capture d’écran montrant le volet Modifier un runbook PowerShell, qui vous invite à procéder à la publication du runbook.](./media/storsimple-8000-automation-azurerm-runbook/publish-runbook.png)

## <a name="next-steps"></a>Étapes suivantes

[Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).