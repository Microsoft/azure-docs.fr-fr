---
title: Supprimer les serveurs et désactiver la protection | Microsoft Docs
description: Cet article montre comment annuler l’inscription des serveurs à partir d’un coffre Site Recovery et comment désactiver la protection des machines virtuelles et serveurs physiques.
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 06/18/2019
ms.author: sharrai
ms.openlocfilehash: a4f6c318a7521e1fbc03ff3a47e34e992cce44df
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89424784"
---
# <a name="remove-servers-and-disable-protection"></a>Supprimer des serveurs et désactiver la protection

Cet article explique comment annuler l’inscription des serveurs à partir d’un coffre Recovery Services et désactiver la protection des machines virtuelles protégées par Site Recovery.


## <a name="unregister-a--configuration-server"></a>Annuler l’inscription d’un serveur de configuration

Si vous répliquez des machines virtuelles VMware ou des serveurs physiques Windows/Linux sur Azure, vous pouvez annuler l’inscription d’un serveur de configuration non connecté à partir d’un coffre, en procédant comme suit :

1. [Désactivez la protection des machines virtuelles](#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure).
2. [Dissociez ou supprimez](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) les stratégies de réplication.
3. [Supprimez le serveur de configuration](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server).

## <a name="unregister-a-vmm-server"></a>Annuler l’inscription d’un serveur VMM

1. Arrêtez la réplication des machines virtuelles dans les clouds sur le serveur VMM que vous souhaitez supprimer.
2. Supprimez tous les mappages réseau utilisés par les clouds sur le serveur VMM que vous souhaitez supprimer. Dans **Infrastructure Site Recovery** > **For System Center VMM (Pour System Center VMM)**  > **Mappage réseau**, cliquez avec le bouton droit sur le mappage réseau et sélectionnez **Supprimer**.
3. Notez l’ID du serveur VMM.
4. Dissociez les stratégies de réplication des clouds sur le serveur VMM que vous souhaitez supprimer.  Dans **Infrastructure Site Recovery** > **For System Center VMM (Pour System Center VMM)**  >  **Stratégies de réplication**, double-cliquez sur la stratégie associée. Cliquez avec le bouton droit sur le cloud et sélectionnez **Dissocier**.
5. Supprimez le serveur VMM ou le nœud actif. Dans **Infrastructure Site Recovery** > **For System Center VMM (Pour System Center VMM)**  > **Serveurs VMM**, cliquez avec le bouton droit sur le serveur et sélectionnez **Supprimer**.
6. Si votre serveur VMM était dans un état déconnecté, téléchargez et exécutez le [script de nettoyage](https://aka.ms/asr-cleanup-script-vmm) sur le serveur VMM. Ouvrez PowerShell avec l’option **Exécuter en tant qu’administrateur** pour modifier la stratégie d’exécution associée à l’étendue par défaut (LocalMachine). Dans le script, spécifiez l’ID du serveur VMM que vous souhaitez supprimer. Le script supprime du serveur l’inscription et les informations de pairage de cloud.
5. Exécutez le script de nettoyage sur tous les serveurs VMM secondaires.
6. Exécutez le script de nettoyage sur les autres nœuds de cluster VMM passifs sur lesquels le fournisseur est installé.
7. Désinstallez le fournisseur manuellement sur le serveur VMM. Si vous avez un cluster, supprimez-le de tous les nœuds.
8. Si vos machines virtuelles étaient répliquées vers Azure, vous devez désinstaller l’agent Microsoft Recovery Services des hôtes Hyper-V dans les clouds supprimés.

## <a name="unregister-a-hyper-v-host-in-a-hyper-v-site"></a>Annuler l’inscription d’un hôte Hyper-V dans un site Hyper-V

Les hôtes Hyper-V non gérés par VMM sont rassemblés dans un site Hyper-V. Pour supprimer un hôte d’un site Hyper-V, procédez comme suit :

1. Désactivez la réplication des machines virtuelles Hyper-V situées sur l’hôte.
2. Dissociez les stratégies du site Hyper-V. Dans **Infrastructure Site Recovery** > **For Hyper-V Sites (Pour les sites Hyper-V)**  >  **Stratégies de réplication**, double-cliquez sur la stratégie associée. Cliquez avec le bouton droit sur le site et sélectionnez **Dissocier**.
3. Supprimez les hôtes Hyper-V. Dans **Infrastructure Site Recovery** > **For Sites Hyper-V (Pour sites Hyper-V)**  > **Hyper-V Hosts (Hôtes Hyper-V)** , cliquez avec le bouton droit sur le serveur et sélectionnez **Supprimer**.
4. Supprimez le site Hyper-V après avoir supprimé tous les hôtes de celui-ci. Dans **Infrastructure Site Recovery** > **For Sites Hyper-V (Pour sites Hyper-V)**  > **Hyper-V Sites (Sites Hyper-V)** , cliquez avec le bouton droit sur le site et sélectionnez **Supprimer**.
5. Si votre hôte Hyper-V se trouvait dans un état **Déconnecté**, exécutez le script suivant sur chaque hôte Hyper-V que vous avez supprimé. Ce script nettoie les paramètres sur le serveur et annule son inscription dans le coffre.


```powershell
        pushd .
        try
        {
            $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
            $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
            $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
            $isAdmin=$principal.IsInRole($administrators)
            if (!$isAdmin)
            {
                "Please run the script as an administrator in elevated mode."
                $choice = Read-Host
                return;
            }

            $error.Clear()
            "This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
            $choice =  Read-Host

            if (!($choice -eq 'Y' -or $choice -eq 'y'))
            {
            "Stopping cleanup."
            return;
            }

            $serviceName = "dra"
            $service = Get-Service -Name $serviceName
            if ($service.Status -eq "Running")
            {
                "Stopping the Azure Site Recovery service..."
                net stop $serviceName
            }

            $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
            $registrationPath = $asrHivePath + '\Registration'
            $proxySettingsPath = $asrHivePath + '\ProxySettings'
            $draIdvalue = 'DraID'
            $idMgmtCloudContainerId='IdMgmtCloudContainerId'


            if (Test-Path $asrHivePath)
            {
                if (Test-Path $registrationPath)
                {
                    "Removing registration related registry keys."
                    Remove-Item -Recurse -Path $registrationPath
                }

                if (Test-Path $proxySettingsPath)
                {
                    "Removing proxy settings"
                    Remove-Item -Recurse -Path $proxySettingsPath
                }

                $regNode = Get-ItemProperty -Path $asrHivePath
                if($regNode.DraID -ne $null)
                {
                    "Removing DraId"
                    Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
                }
                if($regNode.IdMgmtCloudContainerId -ne $null)
                {
                    "Removing IdMgmtCloudContainerId"
                    Remove-ItemProperty -Path $asrHivePath -Name $idMgmtCloudContainerId
                }
                "Registry keys removed."
            }

            # First retrieve all the certificates to be deleted
            $ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
            # Open a cert store object
            $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
            $store.Open('ReadWrite')
            # Delete the certs
            "Removing all related certificates"
            foreach ($cert in $ASRcerts)
            {
                $store.Remove($cert)
            }
        }catch
        {
            [system.exception]
            Write-Host "Error occurred" -ForegroundColor "Red"
            $error[0]
            Write-Host "FAILED" -ForegroundColor "Red"
        }
        popd
```


## <a name="disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure"></a>Désactiver la protection d’une machine virtuelle VMware ou d’un serveur physique (VMware vers Azure)

1. Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
2. Dans la page **Désactiver la réplication**, sélectionnez une des options suivantes :
    - **Désactiver la réplication et supprimer (recommandé)** : cette option supprime l’élément répliqué d’Azure Site Recovery, puis la réplication de la machine est arrêtée. La configuration de la réplication sur le serveur de configuration est nettoyée et la facturation de Site Recovery pour ce serveur protégé est arrêtée. Notez que cette option peut uniquement être utilisée lorsque le serveur de configuration est à l’état Connecté.
    - **Supprimer** : cette option n’est censée être utilisée que si l’environnement source est supprimé ou est inaccessible (non connecté). Elle supprime l’élément répliqué d’Azure Site Recovery (la facturation est arrêtée). La configuration de la réplication sur le serveur de configuration **n’est pas** nettoyée. 

> [!NOTE]
> Aucune des deux options n’entraînant la désinstallation du service Mobilité des serveurs protégés, vous devez le désinstaller manuellement. Si vous envisagez de reprotéger le serveur en utilisant le même serveur de configuration, vous pouvez ignorer la désinstallation du service Mobilité.

> [!NOTE]
> Si vous avez déjà fait basculer une machine virtuelle et qu’elle s’exécute dans Azure, notez que la désactivation de la protection ne supprime/affecte pas la machine virtuelle ayant basculé.
## <a name="disable-protection-for-a-azure-vm-azure-to-azure"></a>Désactiver la protection d’une machine virtuelle Azure (Azure vers Azure)

-  Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
> [!NOTE]
> Le service Mobilité des serveurs protégés ne sera pas désinstallé, vous devez le faire manuellement. Si vous envisagez de reprotéger le serveur, vous pouvez ignorer la désinstallation du service Mobilité.

## <a name="disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure"></a>Désactiver la protection d’une machine virtuelle Hyper-V (Hyper-V vers Azure)

> [!NOTE]
> Utilisez cette procédure si vous répliquez des machines virtuelles Hyper-V sur Azure sans serveur VMM. Si vous répliquez vos machines virtuelles à l'aide du scénario **System Center VMM vers Azure**, suivez les instructions de la section Désactiver la protection d'une machine virtuelle Hyper-V répliquée à l'aide du scénario System Center VMM vers Azure.

1. Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
2. Dans **Désactiver la réplication**, vous pouvez sélectionner les options suivantes :
   - **Désactiver la réplication et supprimer (recommandé)** : cette option supprime l’élément répliqué d’Azure Site Recovery, puis la réplication de la machine est arrêtée. La configuration de la réplication sur la machine virtuelle locale est nettoyée et la facturation de Site Recovery pour ce serveur protégé est arrêtée.
   - **Supprimer** : cette option n’est censée être utilisée que si l’environnement source est supprimé ou est inaccessible (non connecté). Elle supprime l’élément répliqué d’Azure Site Recovery (la facturation est arrêtée). La configuration de la réplication sur la machine virtuelle locale **n’est pas** nettoyée. 

    > [!NOTE]
    > Si vous avez choisi l’option **Supprimer**, exécutez le jeu de scripts suivant pour nettoyer les paramètres de réplication du serveur Hyper-V local.

    > [!NOTE]
    > Si vous avez déjà fait basculer une machine virtuelle et qu’elle s’exécute dans Azure, notez que la désactivation de la protection ne supprime/affecte pas la machine virtuelle ayant basculé.

1. Sur le serveur hôte Hyper-V source, pour supprimer la réplication de la machine virtuelle, remplacez SQLVM1 par le nom de votre machine virtuelle, puis exécutez le script à partir d’une console PowerShell d’administration.

```powershell
    $vmName = "SQLVM1"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario"></a>Désactiver la protection d’une machine virtuelle Hyper-V répliquée vers Azure à l’aide du scénario System Center VMM vers Azure

1. Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
2. Dans **Désactiver la réplication**, sélectionnez une des options suivantes :

   - **Désactiver la réplication et supprimer (recommandé)** : cette option supprime l’élément répliqué d’Azure Site Recovery, puis la réplication de la machine est arrêtée. La configuration de la réplication sur la machine virtuelle locale est nettoyée et la facturation de Site Recovery pour ce serveur protégé est arrêtée.
   - **Supprimer** : cette option n’est censée être utilisée que si l’environnement source est supprimé ou est inaccessible (non connecté). Elle supprime l’élément répliqué d’Azure Site Recovery (la facturation est arrêtée). La configuration de la réplication sur la machine virtuelle locale **n’est pas** nettoyée. 

     > [!NOTE]
     > Si vous avez choisi l’option **Supprimer**, exécutez les scripts suivants pour nettoyer les paramètres de réplication du serveur VMM local.
3. Exécutez ce script sur le serveur VMM source, en utilisant PowerShell (autorisations d’administrateur requises) à partir de la console VMM. Remplacez l’espace réservé **SQLVM1** par le nom de votre machine virtuelle.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. Les étapes ci-dessus effacent les paramètres de réplication sur le serveur VMM. Pour arrêter la réplication de la machine virtuelle sur le serveur hôte Hyper-V, exécutez ce script. Remplacez SQLVM1 par le nom de votre machine virtuelle et host01.contoso.com par le nom du serveur hôte Hyper-V.

```powershell
    $vmName = "SQLVM1"
    $hostName  = "host01.contoso.com"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario"></a>Désactiver la protection d’une machine virtuelle Hyper-V répliquée vers un serveur VMM secondaire à l’aide du scénario System Center VMM vers VMM

1. Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
2. Dans **Désactiver la réplication**, sélectionnez une des options suivantes :

   - **Désactiver la réplication et supprimer (recommandé)** : cette option supprime l’élément répliqué d’Azure Site Recovery, puis la réplication de la machine est arrêtée. La configuration de la réplication sur la machine virtuelle locale est nettoyée et la facturation de Site Recovery pour ce serveur protégé est arrêtée.
   - **Supprimer** : cette option n’est censée être utilisée que si l’environnement source est supprimé ou est inaccessible (non connecté). Elle supprime l’élément répliqué d’Azure Site Recovery (la facturation est arrêtée). La configuration de la réplication sur la machine virtuelle locale **n’est pas** nettoyée. Exécutez le jeu de scripts suivant pour nettoyer les paramètres de réplication des machines virtuelles locales.
     > [!NOTE]
     > Si vous avez choisi l’option **Supprimer**, exécutez les scripts suivants pour nettoyer les paramètres de réplication du serveur VMM local.

3. Exécutez ce script sur le serveur VMM source, en utilisant PowerShell (autorisations d’administrateur requises) à partir de la console VMM. Remplacez l’espace réservé **SQLVM1** par le nom de votre machine virtuelle.

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Set-SCVirtualMachine -VM $vm -ClearDRProtection
    ```

4. Sur le serveur VMM secondaire, exécutez ce script pour nettoyer les paramètres de la machine virtuelle secondaire :

    ```powershell
    $vm = get-scvirtualmachine -Name "SQLVM1"
    Remove-SCVirtualMachine -VM $vm -Force
    ```

5. Sur le serveur VMM secondaire, actualisez les machines virtuelles sur le serveur hôte Hyper-V pour que la machine virtuelle secondaire soit détectée à nouveau dans la console VMM.
6. Les étapes ci-dessus effacent les paramètres de réplication sur le serveur VMM. Si vous souhaitez arrêter la réplication de la machine virtuelle, exécutez le script suivant sur les machines virtuelles principales et secondaires. Remplacez l’élément SQLVM1 par le nom de votre machine virtuelle.

    ```powershell
    Remove-VMReplication –VMName "SQLVM1"
    ```
