---
title: Reprise d’activité après sinistre pour Hyper-V à l’aide d’Azure Site Recovery et PowerShell
description: Automatisez la récupération après sinistre des machines virtuelles Hyper-V sur Azure avec le service Azure Site Recovery à l’aide de PowerShell et d’Azure Resource Manager.
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 01/10/2020
ms.author: sutalasi
ms.openlocfilehash: a4140a0b22f7ca8164d50cf60fe57c861f826eb4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86132507"
---
# <a name="set-up-disaster-recovery-to-azure-for-hyper-v-vms-using-powershell-and-azure-resource-manager"></a>Configurer la récupération d’urgence dans Azure pour les machines virtuelles Hyper-V à l’aide de PowerShell et de Azure Resource Manager

[Azure Site Recovery](site-recovery-overview.md) contribue à mettre en œuvre la stratégie de continuité des activités et de récupération d'urgence de votre entreprise en coordonnant la réplication, le basculement et la récupération de machines virtuelles Azure, de machines virtuelles locales et de serveurs physiques.

Cet article décrit comment utiliser Windows PowerShell, ainsi que Azure Resource Manager pour répliquer des ordinateurs virtuels Hyper-V vers Azure. L’exemple utilisé dans cet article vous explique comment répliquer une machine virtuelle unique exécutée sur un hôte Hyper-V sur Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell fournit des applets de commande pour gérer Azure à l’aide de Windows PowerShell. Les applets de commande PowerShell de Site Recovery, disponibles avec Azure PowerShell pour Azure Resource Manager, vous permettent de protéger et récupérer vos serveurs dans Azure.

Vous n’avez pas besoin d’être un expert de PowerShell pour utiliser cet article, mais vous devez tout de même connaître des concepts de base, tels que les modules, les applets de commande et les sessions. Pour plus d’informations, consultez la [documentation PowerShell](/powershell) et [Utilisation d’Azure PowerShell avec Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

> [!NOTE]
> Les partenaires Microsoft membres du programme Cloud Solution Provider (CSP) peuvent configurer et gérer la protection des serveurs clients sur leurs abonnements CSP respectifs (abonnements des locataires).

## <a name="before-you-start"></a>Avant de commencer

Assurez-vous que les conditions préalables sont remplies :

- Un compte [Microsoft Azure](https://azure.microsoft.com/) . Vous pouvez commencer par une version d’ [essai gratuit](https://azure.microsoft.com/pricing/free-trial/). Vous pouvez aussi consulter la [Tarification Azure Site Recovery Manager](https://azure.microsoft.com/pricing/details/site-recovery/).
- Azure PowerShell. Pour plus d’informations sur cette version et la méthode d’installation, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

De plus, l’exemple décrit dans cet article présente les conditions préalables suivantes :

- Un hôte Hyper-V exécuté sous Windows Server 2012 R2 ou Microsoft Hyper-V Server 2012 R2 et hébergeant une ou plusieurs machines virtuelles. Les serveurs Hyper-V doivent être connectés à Internet, directement ou à travers un proxy.
- Les machines virtuelles que vous souhaitez répliquer doivent respecter ces [conditions préalables](hyper-v-azure-support-matrix.md#replicated-vms).

## <a name="step-1-sign-in-to-your-azure-account"></a>Étape 1 : Connexion à votre compte Azure

1. Ouvrez une console PowerShell et exécutez la commande suivante pour vous connecter à votre compte Azure. L’applet de commande permet d’afficher une page web qui vous demande les informations d’identification de votre compte : `Connect-AzAccount`.
   - Vous pouvez également inclure les informations d’identification de votre compte en tant que paramètre dans l’applet de commande `Connect-AzAccount` à l’aide du paramètre **Credential**.
   - Si vous êtes partenaire CSP travaillant pour le compte d’un locataire, spécifiez le client en tant que locataire, à l’aide de son ID locataire ou de son nom de domaine principal. Par exemple : `Connect-AzAccount -Tenant "fabrikam.com"`
1. Associez l’abonnement que vous souhaitez utiliser avec le compte, car un compte peut compter plusieurs abonnements :

   ```azurepowershell
   Set-AzContext -Subscription $SubscriptionName
   ```

1. Utilisez les commandes suivantes pour vérifier que votre abonnement vous autorise à utiliser les fournisseurs Azure pour Recovery Services et Site Recovery :

   ```azurepowershell
   Get-AzResourceProvider -ProviderNamespace  Microsoft.RecoveryServices
   ```

1. Vérifiez que dans la sortie de commande **RegistrationState** est défini sur **Inscrit**, vous pouvez ensuite passer à l’étape 2. Dans le cas contraire, vous devez inscrire le fournisseur manquant dans votre abonnement en exécutant ces commandes :

   ```azurepowershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.RecoveryServices
   ```

1. Vérifiez que les fournisseurs ont été correctement inscrits à l’aide des commandes suivantes :

   ```azurepowershell
   Get-AzResourceProvider -ProviderNamespace  Microsoft.RecoveryServices
   ```

## <a name="step-2-set-up-the-vault"></a>Étape 2 : Configuration du coffre

1. Créez un groupe de ressources Azure Resource Manager dans lequel vous allez créer le coffre, ou utilisez un groupe de ressources existant. Créez un nouveau groupe de ressources de la manière suivante. La variable `$ResourceGroupName` contient le nom du groupe de ressources que vous souhaitez créer et la variable $Geo contient la région Azure dans laquelle créer le groupe de ressources (par exemple, « Brésil Sud »).

   ```azurepowershell
   New-AzResourceGroup -Name $ResourceGroupName -Location $Geo
   ```

1. Pour obtenir une liste des groupes de ressources de votre abonnement, exécutez l’applet de commande `Get-AzResourceGroup`.
1. Créez un coffre Azure Recovery Services de la manière suivante :

   ```azurepowershell
   $vault = New-AzRecoveryServicesVault -Name <string> -ResourceGroupName <string> -Location <string>
   ```

Vous pouvez récupérer la liste des coffres existants avec l’applet de commande `Get-AzRecoveryServicesVault`.

## <a name="step-3-set-the-recovery-services-vault-context"></a>Étape 3 : Définition du contexte du coffre Recovery Services

Définissez le contexte d’archivage comme suit :

```azurepowershell
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

## <a name="step-4-create-a-hyper-v-site"></a>Étape 4 : Créer un site Hyper-V

1. Créez un nouveau site Hyper-V comme suit :

   ```azurepowershell
   $sitename = "MySite"                #Specify site friendly name
   New-AzRecoveryServicesAsrFabric -Type HyperVSite -Name $sitename
   ```

1. Cette applet de commande démarre une tâche Site Recovery pour créer le site et renvoie un objet de tâche Site Recovery. Attendez que la tâche soit terminée, puis vérifiez son exécution.
1. Utilisez l’applet de commande `Get-AzRecoveryServicesAsrJob` pour récupérer l’objet de traitement, puis vérifiez l’état actuel du travail.
1. Générez et téléchargez une clé d’inscription pour le site comme suit :

   ```azurepowershell
   $SiteIdentifier = Get-AzRecoveryServicesAsrFabric -Name $sitename | Select-Object -ExpandProperty SiteIdentifier
   $path = Get-AzRecoveryServicesVaultSettingsFile -Vault $vault -SiteIdentifier $SiteIdentifier -SiteFriendlyName $sitename
   ```

1. Copiez la clé téléchargée sur l’hôte Hyper-V. Vous avez besoin de la clé pour inscrire l’hôte Hyper-V sur le site.

## <a name="step-5-install-the-provider-and-agent"></a>Étape 5 : Installez le fournisseur et l’agent

1. Téléchargez le programme d’installation de la dernière version du fournisseur sur le site de [Microsoft](https://aka.ms/downloaddra).
1. Exécutez le programme d’installation sur l’hôte Hyper-V.
1. À la fin de l’installation, passez à l’étape d’inscription.
1. Quand vous y êtes invité, renseignez la clé que vous avez téléchargée et terminez l’inscription de l’hôte Hyper-V.
1. Vérifiez que l’hôte Hyper-V a bien été inscrit sur le site comme suit :

   ```azurepowershell
   $server = Get-AzRecoveryServicesAsrFabric -Name $siteName | Get-AzRecoveryServicesAsrServicesProvider -FriendlyName $server-friendlyname
   ```

Si vous exécutez un serveur Hyper-V (installation minimale), téléchargez le fichier d’installation et effectuez les étapes suivantes :

1. Extrayez les fichiers d’_AzureSiteRecoveryProvider.exe_ dans un répertoire local en exécutant cette commande :

   ```console
   AzureSiteRecoveryProvider.exe /x:. /q
   ```

1. Exécutez la commande suivante :

   ```console
   .\setupdr.exe /i
   ```

   Les résultats sont journalisés dans _%ProgramData%\ASRLogs\DRASetupWizard.log_.

1. Inscrivez le serveur en exécutant cette commande :

   ```console
   cd  C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe" /r /Friendlyname "FriendlyName of the Server" /Credentials "path to where the credential file is saved"
   ```

## <a name="step-6-create-a-replication-policy"></a>Étape 6 : Créer une stratégie de réplication

Avant de commencer, le compte de stockage spécifié doit se trouver dans la même région Azure que le coffre et doit utiliser la fonction de géoréplication.

1. Créez une stratégie de réplication comme suit :

   ```azurepowershell
   $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
   $PolicyName = “replicapolicy”
   $Recoverypoints = 6                    #specify the number of recovery points
   $storageaccountID = Get-AzStorageAccount -Name "mystorea" -ResourceGroupName "MyRG" | Select-Object -ExpandProperty Id

   $PolicyResult = New-AzRecoveryServicesAsrPolicy -Name $PolicyName -ReplicationProvider “HyperVReplicaAzure” -ReplicationFrequencyInSeconds $ReplicationFrequencyInSeconds -NumberOfRecoveryPointsToRetain $Recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId $storageaccountID
   ```

1. Vérifiez la tâche renvoyée pour vous assurer que la stratégie de réplication a bien été créée.

1. Récupérez le conteneur de protection correspondant au site comme suit :

   ```azurepowershell
   $protectionContainer = Get-AzRecoveryServicesAsrProtectionContainer
   ```

1. Associez le conteneur de protection avec la stratégie de réplication comme suit :

   ```azurepowershell
   $Policy = Get-AzRecoveryServicesAsrPolicy -FriendlyName $PolicyName
   $associationJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name $mappingName -Policy $Policy -PrimaryProtectionContainer $protectionContainer[0]
   ```

1. Attendez que la tâche d’association se termine.

1. Récupérez le mappage de conteneurs de protection.

   ```azurepowershell
   $ProtectionContainerMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $protectionContainer
   ```

## <a name="step-7-enable-vm-protection"></a>Étape 7 : Activation de la protection de machine virtuelle

1. Récupérez l’élément protégeable correspondant à la machine virtuelle que vous souhaitez protéger comme suit :

   ```azurepowershell
   $VMFriendlyName = "Fabrikam-app"          #Name of the VM
   $ProtectableItem = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName
   ```

1. Protéger la machine virtuelle. Si la machine virtuelle que vous protégez compte plusieurs disques, spécifiez le disque de système d’exploitation avec le paramètre **OSDiskName**.

   ```azurepowershell
   $OSType = "Windows"          # "Windows" or "Linux"
   $DRjob = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $VM -Name $VM.Name -ProtectionContainerMapping $ProtectionContainerMapping -RecoveryAzureStorageAccountId $StorageAccountID -OSDiskName $OSDiskNameList[$i] -OS $OSType -RecoveryResourceGroupId $ResourceGroupID
   ```

1. Attendez que les machines virtuelles basculent à l’état protégé après la réplication initiale. Cette opération prendra un certain temps compte tenu de certains facteurs, notamment la quantité de données à répliquer et la bande passante en amont disponible pour Azure. Lorsque l’état protégé est en place, les paramètres State et StateDescription de la tâche s’actualisent comme suit :

   ```console
   PS C:\> $DRjob = Get-AzRecoveryServicesAsrJob -Job $DRjob

   PS C:\> $DRjob | Select-Object -ExpandProperty State
   Succeeded

   PS C:\> $DRjob | Select-Object -ExpandProperty StateDescription
   Completed
   ```

1. Mettez à jour les propriétés de récupération (telles que la taille du rôle de machine virtuelle) et le réseau Azure auquel attacher la carte réseau de la machine virtuelle après le basculement.

   ```console
   PS C:\> $nw1 = Get-AzVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

   PS C:\> $VMFriendlyName = "Fabrikam-App"

   PS C:\> $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

   PS C:\> $UpdateJob = Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $rpi -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

   PS C:\> $UpdateJob = Get-AzRecoveryServicesAsrJob -Job $UpdateJob

   PS C:\> $UpdateJob | Select-Object -ExpandProperty state

   PS C:\> Get-AzRecoveryServicesAsrJob -Job $job | Select-Object -ExpandProperty state

   Succeeded
   ```

> [!NOTE]
> Si vous voulez répliquer sur des disques managés activés par une CMK dans Azure, effectuez les étapes suivantes avec la commande Az de PowerShell 3.3.0 et versions ultérieures :
>
> 1. Activez le basculement vers des disques managés en mettant à jour les propriétés des machines virtuelles.
> 1. Utilisez l’applet de commande `Get-AzRecoveryServicesAsrReplicationProtectedItem` pour extraire l’ID de chaque disque de l’élément protégé.
> 1. Créez un objet de dictionnaire à l’aide de l’applet de commande `New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"` destiné à contenir le mappage de l’ID de disque au jeu de chiffrement de disque. Vous devez créer ces jeux de chiffrement de disque au préalable dans la région cible.
> 1. Mettez à jour les propriétés des machines virtuelles à l’aide de l’applet de commande `Set-AzRecoveryServicesAsrReplicationProtectedItem` en passant l’objet du dictionnaire dans le paramètre **DiskIdToDiskEncryptionSetMap**.

## <a name="step-8-run-a-test-failover"></a>Étape 8 : Exécution d’un test de basculement

1. Exécutez un test de basculement, en procédant comme suit :

   ```azurepowershell
   $nw = Get-AzVirtualNetwork -Name "TestFailoverNw" -ResourceGroupName "MyRG" #Specify Azure vnet name and resource group

   $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

   $TFjob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $VM -Direction PrimaryToRecovery -AzureVMNetworkId $nw.Id
   ```

1. Vérifiez que la machine virtuelle de test est bien créée dans Azure. La tâche de test de basculement est suspendue après la création de la machine virtuelle de test dans Azure.
1. Pour nettoyer et terminer le test de basculement, exécutez :

   ```azurepowershell
   $TFjob = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $rpi -Comment "TFO done"
   ```

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](/powershell/module/az.recoveryservices) sur Azure Site Recovery avec les applets de commande PowerShell Azure Resource Manager.
