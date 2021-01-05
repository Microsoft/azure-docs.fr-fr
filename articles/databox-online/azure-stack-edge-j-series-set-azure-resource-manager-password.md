---
title: Définir un mot de passe Azure Resource Manager sur votre appareil Azure Stack Edge Pro avec GPU
description: Décrit la procédure de définition du mot de passe Azure Resource Manager sur votre processeur Azure Stack Edge Pro à l’aide d’Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 915146cd17b90272daea4ce57f5243baf1d49cb3
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94578788"
---
# <a name="set-azure-resource-manager-password-on-azure-stack-edge-pro-gpu-device"></a>Définir un mot de passe Azure Resource Manager sur un appareil Azure Stack Edge Pro avec GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Cet article explique comment définir votre mot de passe Azure Resource Manager. Vous devez définir ce mot de passe lorsque vous vous connectez aux API locales de l’appareil via Azure Resource Manager.

La procédure permettant de définir le mot de passe peut être différente selon que vous utilisez le portail Azure ou les cmdlets PowerShell. Chacune de ces procédures est décrite dans les sections suivantes.


## <a name="reset-password-via-the-azure-portal"></a>Réinitialiser le mot de passe via le portail Azure

1. Dans le portail Azure, accédez à la ressource Azure Stack Edge que vous avez créée pour gérer votre appareil. Accédez à **Computing en périphérie > Bien démarrer**.

2. Dans le volet droit, à partir de la barre de commandes, sélectionnez **Réinitialiser le mot de passe ARM Edge**. 

    ![Réinitialiser le mot de passe utilisateur EdgeARM 1](media/azure-stack-edge-j-series-set-azure-resource-manager-password/set-edgearm-password-1.png)

3. Dans le panneau **Réinitialiser le mot de passe utilisateur EdgeARM**, fournissez un mot de passe pour vous connecter aux API locales de votre appareil via Azure Resource Manager. Confirmez le mot de passe, puis sélectionnez **Réinitialiser**.

    ![Réinitialiser le mot de passe utilisateur EdgeARM 2](media/azure-stack-edge-j-series-set-azure-resource-manager-password/set-edgearm-password-2.png)



## <a name="reset-password-via-powershell"></a>Réinitialiser le mot de passe via PowerShell

1. Dans le portail Azure, accédez à la ressource Azure Stack Edge que vous avez créée pour gérer votre appareil. Prenez note des paramètres suivants dans la page **Vue d’ensemble**.

    - Nom de la ressource Azure Stack Edge
    - Identifiant d’abonnement

2. Accédez à **Paramètres > Propriétés**. Prenez note des paramètres suivants dans la page **Propriétés**.

    - Resource group
    - Clé de chiffrement CIK : sélectionnez « Afficher », puis copiez la **clé de chiffrement**.

    ![Récupérer la clé de chiffrement CIK](media/azure-stack-edge-j-series-set-azure-resource-manager-password/get-cik-portal.png)
 
3. Identifiez un mot de passe que vous utiliserez pour vous connecter à Azure Resource Manager.

4. Démarrez l’interpréteur de commandes cloud. Sélectionnez l’icône dans le coin supérieur droit :

    ![Démarrer Cloud Shell](media/azure-stack-edge-j-series-set-azure-resource-manager-password/start-cloud-shell.png) 

    Une fois que l’interpréteur de commandes cloud a démarré, vous devrez peut-être basculer vers PowerShell.

    ![Cloud Shell](media/azure-stack-edge-j-series-set-azure-resource-manager-password/cloud-shell.png)   


5. Définissez le contexte. Tapez :

    `Set-AzContext -SubscriptionId <Subscription ID>`

    Voici un exemple de sortie :

    
    ```azurepowershell
    PS Azure:\> Set-AzContext -SubscriptionId 8eb87630-972c-4c36-a270-f330e6c063df
    
        Name        Account   SubscriptionName   Environment  TenantId
        ----       -------    ----------------   -----------  --------
        DataBox_Edge_Test (8eb87630-972c-4c36-a… MSI@50342 DataBox_Edge_Tes AzureCloud           72f988bf-86f1-41af-91ab-2d7…
    
        PS Azure:/
    ```
    
5.  Si vous avez d’anciens modules PS, vous devez les installer.

    `Remove-Module  Az.DataBoxEdge -force`

    Voici un exemple de sortie. Dans cet exemple, il n’existait aucun ancien module à installer.

    
    ```azurepowershell
        PS Azure:\> Remove-Module  Az.DataBoxEdge -force
        Remove-Module : No modules were removed. Verify that the specification of modules to remove is correct and those modules exist in the runspace.
        At line:1 char:1
        + Remove-Module  Az.DataBoxEdge -force
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : ResourceUnavailable: (:) [Remove-Module], InvalidOperationException
        + FullyQualifiedErrorId : Modules_NoModulesRemoved,Microsoft.PowerShell.Commands.RemoveModuleCommand
    
        PS Azure:\
    ```

6. Le jeu de commandes suivant permet de télécharger et d’exécuter un script pour installer les modules PowerShell.
    
    ```azurepowershell
        cd ~/clouddrive
        wget https://aka.ms/dbe-cmdlet-beta -O Az.DataBoxEdge.zip
        unzip ./Az.DataBoxEdge.zip
        Import-Module ~/clouddrive/Az.DataBoxEdge/Az.DataBoxEdge.psd1 -Force
    ```

7. Dans le jeu de commandes suivant, vous devez fournir le nom de la ressource, le nom du groupe de ressources, la clé de chiffrement et le mot de passe que vous avez identifié à l’étape précédente.

    ```azurepowershell
    $devicename = "<Azure Stack Edge resource name>"
    $resourceGroup = "<Resource group name>"
    $cik = "<Encryption key>"
    $password = "<Password>"
    ```
    Les paramètres de clé de chiffrement et de mot de passe doivent être transmis sous formes de chaînes sécurisées. Utilisez les cmdlets suivantes pour convertir le mot de passe et la clé de chiffrement en chaînes sécurisées.

    ```azurepowershell
    $pass = ConvertTo-SecureString $password -AsPlainText -Force
    $key = ConvertTo-SecureString $cik -AsPlainText -Force
    ```
    Utilisez les chaînes sécurisées générées ci-dessus comme paramètres dans la cmdlet Set-AzDataBoxEdgeUser pour réinitialiser le mot de passe. Utilisez le même groupe de ressources que celui utilisé lors de la création de la ressource Azure Stack Edge Pro/Data Box Gateway.

    ```azurepowershell
    Set-AzDataBoxEdgeUser -ResourceGroupName $resourceGroup -DeviceName $devicename -Name EdgeARMUser  -Password $pass -EncryptionKey $key
    ```
    Voici l’exemple de sortie.
    
    ```azurepowershell
    PS /home/aseuser/clouddrive> $devicename = "myaseresource"
    PS /home/aseuser/clouddrive> $resourceGroup = "myaserg"
    PS /home/aseuser/clouddrive> $cik = "54a7450fd7b3c506e10efea4e0c88a9390f37e299fbf43e01fb5dfe483ac036b6d0f85a6246e1926e297f98c0ff84c20a57348c689eff179ce31571fc787ac0a"
    PS /home/aseuser/clouddrive> $password = "Password2"
    PS /home/aseuser/clouddrive> $pass = ConvertTo-SecureString $password -AsPlainText -Force
    PS /home/aseuser/clouddrive> $key = ConvertTo-SecureString $cik -AsPlainText -Force
    PS /home/aseuser/clouddrive> Set-AzDataBoxEdgeUser -ResourceGroupName $resourceGroup -DeviceName $devicename -Name EdgeARMUser  -Password $pass -EncryptionKey $key
    
        User name   Type ResourceGroupName DeviceName
    ---------   ---- ----------------- ----------
        EdgeARMUser ARM  myaserg        myaseresource
    
        PS /home/aseuser/clouddrive>
    ```
Utilisez le nouveau mot de passe pour vous connecter à Azure Resource Manager.

## <a name="next-steps"></a>Étapes suivantes

[Se connecter à Azure Resource Manager](azure-stack-edge-j-series-connect-resource-manager.md)
