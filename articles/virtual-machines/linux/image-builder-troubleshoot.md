---
title: Résoudre les problèmes liés au service Azure VM Image Builder
description: Résoudre les erreurs et problèmes courants lors de l’utilisation du service Azure VM Image Builder
author: cynthn
ms.author: danis
ms.date: 10/02/2020
ms.topic: troubleshooting
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: 7c937353c645ee5d977a52ec0f8e935eba19a940
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91969974"
---
# <a name="troubleshoot-azure-image-builder-service"></a>Résoudre les problèmes liés au service Azure VM Image Builder

Cet article vous aide à dépanner et à résoudre les problèmes courants que vous pouvez rencontrer lors de l’utilisation du service Azure VM Image Builder.

Les échecs d’AIB peuvent se produire à deux endroits :
- Envoi de modèle d’image
- Génération d’image

## <a name="troubleshoot-image-template-submission-errors"></a>Résoudre les erreurs d’envoi du modèle d’image

Les erreurs d’envoi du modèle d’image sont renvoyées uniquement au moment de l’envoi. Il n’existe pas de journal des erreurs pour ce type d’erreur. En cas d’erreur lors de l’envoi, vous pouvez retourner l’erreur en vérifiant l’état du modèle, en particulier en examinant `ProvisioningState` et `ProvisioningErrorMessage`/`provisioningError`.

```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object ProvisioningState, ProvisioningErrorMessage
```
> [!NOTE]
> Pour PowerShell, vous devez installer les [modules PowerShell Azure VM Image Builder](../windows/image-builder-powershell.md#prerequisites).

Les sections suivantes incluent des conseils de résolution des problèmes pour les erreurs courantes d’envoi du modèle d’image.

### <a name="updateupgrade-of-image-templates-is-currently-not-supported"></a>La mise à jour/mise à niveau des modèles d’images n’est pas prise en charge actuellement.

#### <a name="error"></a>Error

```text
'Conflict'. Details: Update/Upgrade of image templates is currently not supported
```

#### <a name="cause"></a>Cause

Le modèle existe déjà.

#### <a name="solution"></a>Solution

Si vous soumettez un modèle de configuration d’image et que l’envoi échoue, il existe toujours un artefact de l’échec du modèle. Supprimez le modèle qui a échoué.

### <a name="the-resource-operation-completed-with-terminal-provisioning-state-failed"></a>L’opération de ressource s’est achevée avec l’état d’approvisionnement du terminal « Échec ».

#### <a name="error"></a>Error

```text
Microsoft.VirtualMachineImages/imageTemplates 'helloImageTemplateforSIG01' failed with message '{
  "status": "Failed",
  "error": {
    "code": "ResourceDeploymentFailure",
    "message": "The resource operation completed with terminal provisioning state 'Failed'.",
    "details": [
      {
        "code": "InternalOperationError",
        "message": "Internal error occurred."
```
#### <a name="cause"></a>Cause

Dans la plupart des cas, l’échec du déploiement de la ressource est due à des autorisations manquantes.

#### <a name="solution"></a>Solution

Selon votre scénario, Azure VM Image Builder peut avoir besoin d’autorisations pour :
- Image source ou groupe de ressources Shared Image Gallery
- Image de distribution ou ressource Shared Image Gallery
- Compte de stockage, conteneur ou blob auquel le personnalisateur de fichiers accède 

Pour plus d’informations sur la configuration des autorisations, consultez [Configurer les autorisations de service Azure Image Builder à l’aide d’Azure CLI](image-builder-permissions-cli.md) ou [Configurer les autorisations du service Azure Image Builder à l’aide de PowerShell](image-builder-permissions-powershell.md).

### <a name="error-getting-managed-image"></a>Erreur lors de l’obtention de l’image managée

#### <a name="error"></a>Error

```text
Build (Managed Image) step failed: Error getting Managed Image '/subscriptions/.../providers/Microsoft.Compute/images/mymanagedmg1': Error getting managed image (...): compute.
ImagesClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error.
Status=403 Code="AuthorizationFailed" Message="The client '......' with object id '......' does not have authorization to perform action 'Microsoft.Compute/images/read' over scope 
```
#### <a name="cause"></a>Cause

Autorisations manquantes.

#### <a name="solution"></a>Solution

Selon votre scénario, Azure VM Image Builder peut avoir besoin d’autorisations pour :
* Image source ou groupe de ressources Shared Image Gallery
* Image de distribution ou ressource Shared Image Gallery
* Compte de stockage, conteneur ou blob auquel le personnalisateur de fichiers accède 

Pour plus d’informations sur la configuration des autorisations, consultez [Configurer les autorisations de service Azure Image Builder à l’aide d’Azure CLI](image-builder-permissions-cli.md) ou [Configurer les autorisations du service Azure Image Builder à l’aide de PowerShell](image-builder-permissions-powershell.md).

### <a name="build--step-failed-for-image-version"></a>Échec de l’étape de génération pour la version de l’image

#### <a name="error"></a>Error
```text
Build (Shared Image Version) step failed for Image Version '/subscriptions/.../providers/Microsoft.Compute/galleries/.../images/... /versions/0.23768.4001': Error getting Image Version '/subscriptions/.../resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001': Error getting image version '... :0.23768.4001': compute.GalleryImageVersionsClient#Get: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. 
Status=404 Code="ResourceNotFound" Message="The Resource 'Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001' under resource group '<rgName>' was not found."
```
#### <a name="cause"></a>Cause

Azure VM Image Builder ne peut pas localiser l’image source.

#### <a name="solution"></a>Solution

Assurez-vous que l’image source est correcte et qu’elle existe à l’emplacement du service Azure VM Image Builder.

### <a name="downloading-external-file-to-local-file"></a>Téléchargement d’un fichier externe vers un fichier local

#### <a name="error"></a>Error

```text
Downloading external file (<myFile>) to local file (xxxxx.0.customizer.fp) [attempt 1 of 10] failed: Error downloading '<myFile>' to 'xxxxx.0.customizer.fp'..
```

#### <a name="cause"></a>Cause

Le nom ou l’emplacement du fichier est incorrect, ou l’emplacement n’est pas accessible.

#### <a name="solution"></a>Solution

Assurez-vous que le fichier est accessible. Vérifiez que le nom et l’emplacement sont corrects.

## <a name="troubleshoot-build-failures"></a>Résoudre les échecs de build

Pour les échecs de génération d’images, vous pouvez obtenir l’erreur à partir du `lastrunstatus`, puis passer en revue les détails dans le fichier customization.log.


```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object LastRunStatus, LastRunStatusMessage
```

### <a name="customization-log"></a>Journal de personnalisation

Lorsque la build de l’image est en cours d’exécution, des journaux sont créés et stockés dans un compte de stockage. Azure VM Image Builder crée le compte de stockage dans le groupe de ressources temporaire lorsque vous créez un artefact de modèle d’image.

Le nom du compte de stockage utilise le modèle suivant : **IT_\<ImageResourceGroupName\> _\<TemplateName\>_ \<GUID\>**

Par exemple, *IT_aibmdi_helloImageTemplateLinux01*.

Vous pouvez afficher le customization.log dans le compte de stockage du groupe de ressources en sélectionnant **Compte de stockage** > **Blobs** > `packerlogs`.  Sélectionnez ensuite **répertoire > customization.log**.


### <a name="understanding-the-customization-log"></a>Fonctionnement du journal de personnalisation

Le journal est détaillé. Il couvre la build d’image, notamment les problèmes liés à la distribution d’image tels que la réplication de Shared Image Gallery. Ces erreurs sont signalées dans le message d’erreur relatif à l’état du modèle d’image.

Le fichier customization.log comprend les étapes suivantes :

1. Étape de déploiement de la machine virtuelle de build et des dépendances à l’aide de modèles Resource Manager dans le groupe de ressources intermédiaire IT_. Cette étape comprend plusieurs revendications POST adressées au fournisseur de ressources Azure VM Image Builder :
    ```text
    Azure request method="POST" request="https://management.azure.com/subscriptions/<subID>/resourceGroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-cccc-cccc-ccccccc/providers/Microsoft.Storage/storageAccounts
    ..
    PACKER OUT ==> azure-arm: Deploying deployment template ...
    ..
    ```

2. Étape d’état des déploiements. Cette étape comprend l’état de chaque déploiement de ressource :
    ```text
    PACKER ERR 2020/04/30 23:28:50 packer: 2020/04/30 23:28:50 Azure request method="GET" request="https://management.azure.com/subscriptions/<subID>/resourcegroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-4505-ae28-6661e43fac48/providers/Microsoft.Resources/deployments/pkrdp51lc0339jg/operationStatuses/08586133176207523519?[REDACTED]" body=""
    ```

3. Étape de connexion à la machine virtuelle de build.

    Sous Windows, le service Azure VM Image Builder se connecte à l’aide de WinRM :
    ```text
    PACKER ERR 2020/04/30 23:30:50 packer: 2020/04/30 23:30:50 Waiting for WinRM, up to timeout: 10m0s
    ..
    PACKER OUT     azure-arm: WinRM connected.
    ```

    Sous Linux, le service Azure VM Image Builder se connecte à l’aide de SSH :
    ```text
    PACKER OUT ==> azure-arm: Waiting for SSH to become available...
    PACKER ERR 2019/12/10 17:20:51 packer: 2020/04/10 17:20:51 [INFO] Waiting for SSH, up to timeout: 20m0s
    PACKER OUT ==> azure-arm: Connected to SSH!
    ```

4. Étape d’exécution des personnalisations. Lorsque les personnalisations s’exécutent, vous pouvez les identifier en consultant le fichier customization.log. Recherchez *(telemetry)* .
    ```text
    (telemetry) Starting provisioner windows-update
    (telemetry) ending windows-update
    (telemetry) Starting provisioner powershell
    (telemetry) ending powershell
    (telemetry) Starting provisioner file
    (telemetry) ending file
    (telemetry) Starting provisioner windows-restart
    (telemetry) ending windows-restart
    
    (telemetry) Finalizing. - This means the build hasfinished
    ```
5. Étape de désapprovisionnement. Azure VM Image Builder ajoute un personnalisateur masqué. Cette étape de désapprovisionnement prépare la machine virtuelle pour le désapprovisionnement. Il exécute Sysprep sous Windows (à l’aide de c:\DeprovisioningScript.ps1) ou waagent deprovision sous Linux (à l’aide de /tmp/DeprovisioningScript.sh). 

    Par exemple :
    ```text
    PACKER ERR 2020/03/04 23:05:04 [INFO] (telemetry) Starting provisioner powershell
    PACKER ERR 2020/03/04 23:05:04 packer: 2020/03/04 23:05:04 Found command: if( TEST-PATH c:\DeprovisioningScript.ps1 ){cat c:\DeprovisioningScript.ps1} else {echo "Deprovisioning script [c:\DeprovisioningScript.ps1] could not be found. Image build may fail or the VM created from the Image may not boot. Please make sure the deprovisioning script is not accidentally deleted by a Customizer in the Template."}
    ```

6. Étape de nettoyage. Une fois la build terminée, les ressources Azure VM Image Builder sont supprimées.
    ```text
    PACKER ERR 2020/02/04 02:04:23 packer: 2020/02/04 02:04:23 Azure request method="DELETE" request="https://management.azure.com/subscriptions/<subId>/resourceGroups/IT_aibDevOpsImg_t_vvvvvvv_yyyyyy-de5f-4f7c-92f2-xxxxxxxx/providers/Microsoft.Network/networkInterfaces/pkrnijamvpo08eo?[REDACTED]" body=""
    ```
## <a name="tips-for-troubleshooting-scriptinline-customization"></a>Conseils pour la résolution des problèmes de personnalisation de script/inlined
- Testez le code avant de le fournir à Image Builder.
- Assurez-vous que les pare-feu et les stratégies Azure autorisent la connectivité aux ressources distantes.
- Affichez les commentaires dans la console, par exemple en utilisant `Write-Host` ou `echo`, ce qui vous permettra d’effectuer des recherches dans le customization.log.

## <a name="troubleshoot-common-build-errors"></a>Résolution des erreurs de build courantes

### <a name="packer-build-command-failure"></a>Échec de la commande de build Packer

#### <a name="error"></a>Error

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-04-30T23:24:06.756985789Z",
   "endTime": "2020-04-30T23:39:14.268729811Z",
   "runState": "Failed",
   "message": "Failed while waiting for packerizer: Microservice has failed: Failed while processing request: Error when executing packerizer: Packer build command has failed: exit status 1. During the image build, a failure has occurred, please review the build log to identify which build/customization step failed. For more troubleshooting steps go to https://aka.ms/azvmimagebuilderts. Image Build log location: https://xxxxxxxxxx.blob.core.windows.net/packerlogs/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/customization.log. OperationId: xxxxxx-5a8c-4379-xxxx-8d85493bc791. Use this operationId to search packer logs."
```

#### <a name="cause"></a>Cause

Échec de la personnalisation.

#### <a name="solution"></a>Solution

Consultez le journal pour localiser les échecs des personnalisateurs. Recherchez *(telemetry)* . 

Par exemple :
```text
(telemetry) Starting provisioner windows-update
(telemetry) ending windows-update
(telemetry) Starting provisioner powershell
(telemetry) ending powershell
(telemetry) Starting provisioner file
(telemetry) ending file
(telemetry) Starting provisioner windows-restart
(telemetry) ending windows-restart

(telemetry) Finalizing. - This means the build has finished
```

### <a name="timeout-exceeded"></a>Délai expiré

#### <a name="error"></a>Error

```text
Deployment failed. Correlation ID: xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx. Failed in building/customizing image: Failed while waiting for packerizer: Timeout waiting for microservice to complete: 'context deadline exceeded'
```

#### <a name="cause"></a>Cause

Le délai d’attente de la build a expiré. Cette erreur s’affiche dans « lastrunstatus ».

#### <a name="solution"></a>Solution

1. Passez en revue le fichier customization.log. Identifiez le dernier personnalisateur à s’exécuter. Recherchez `(telemetry)` en partant du bas du journal. 

2. Vérifiez les personnalisations du script. Les personnalisations ne doivent pas supprimer l’interaction de l’utilisateur avec les commandes, telles que les options `quiet`. Par exemple, `apt-get install -y` entraîne l’exécution du script en attente de l’intervention de l’utilisateur.

3. Si vous utilisez le personnalisateur `File` pour télécharger des artefacts supérieurs à 20 Mo, consultez la section relative aux solutions de contournement.

4. Examinez les erreurs et les dépendances dans le script qui peuvent entraîner l’attente du script.

5. Si vous vous attendez à ce que les personnalisations nécessitent davantage de temps, augmentez la valeur du paramètre [buildTimeoutInMinutes](image-builder-json.md). La valeur par défaut est de quatre heures.

6. Si vous avez des actions gourmandes en ressources, telles que le téléchargement de gigaoctets de fichiers, tenez compte de la taille de machine virtuelle de build sous-jacente. Le service utilise une machine virtuelle Standard_D1_v2. La machine virtuelle possède 1 processeur virtuel et 3,5 Go de mémoire. Si vous téléchargez 50 Go, cela épuisera probablement les ressources de machine virtuelle et provoquera des échecs de communication entre le service Azure VM Image Builder et la machine virtuelle de build. Réessayez la build avec une machine virtuelle dont la mémoire est plus grande, en définissant [VM_Size](image-builder-json.md#vmprofile).

### <a name="long-file-download-time"></a>Longue durée de téléchargement des fichiers

#### <a name="error"></a>Error
```text
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 826 B / 826000 B  1.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 1652 B / 826000 B  2.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
..
hours later...
..
myBigFile.zip 826000 B / 826000 B  100.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
```
#### <a name="cause"></a>Cause 

Le personnalisateur de fichiers télécharge un fichier volumineux.

#### <a name="solution"></a>Solution

Le personnalisateur de fichiers est uniquement adapté au téléchargement de fichiers de petite taille (moins de 20 Mo). Pour les téléchargements de fichiers plus volumineux, utilisez un script ou une commande inlined. Par exemple, vous pouvez utiliser `wget` ou `curl`sous Linux. Sous Windows, vous pouvez utiliser `Invoke-WebRequest`.

### <a name="error-waiting-on-shared-image-gallery"></a>Erreur lors de l’attente de Shared Image Gallery

#### <a name="error"></a>Error

```text
Deployment failed. Correlation ID: XXXXXX-XXXX-XXXXXX-XXXX-XXXXXX. Failed in distributing 1 images out of total 1: {[Error 0] [Distribute 0] Error publishing MDI to shared image gallery:/subscriptions/<subId>/resourceGroups/xxxxxx/providers/Microsoft.Compute/galleries/xxxxx/images/xxxxxx, Location:eastus. Error: Error returned from SIG client while publishing MDI to shared image gallery for dstImageLocation: eastus, dstSubscription: <subId>, dstResourceGroupName: XXXXXX, dstGalleryName: XXXXXX, dstGalleryImageName: XXXXXX. Error: Error waiting on shared image gallery future for resource group: XXXXXX, gallery name: XXXXXX, gallery image name: XXXXXX.Error: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded}
```

#### <a name="cause"></a>Cause

Image Builder a expiré lors de l’ajout et de la réplication de l’image dans Shared Image Gallery (SIG). Si l’image est injectée dans SIG, il peut être supposé que la génération de l’image a réussi. Toutefois, le processus global a échoué, car le générateur d’images attendait Shared Image Gallery pour terminer la réplication. Même si la génération a échoué, la réplication se poursuit. Vous pouvez récupérer les propriétés de la version de l’image en vérifiant la distribution *runOutput*.

```bash
$runOutputName=<distributionRunOutput>
az resource show \
    --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/$imageTemplateName/runOutputs/$runOutputName"  \
    --api-version=2019-05-01-preview
```

#### <a name="solution"></a>Solution

Augmentez la valeur de **buildTimeoutInMinutes**.
 
### <a name="low-windows-resource-information-events"></a>Événements d’informations de ressources Windows faibles

#### <a name="error"></a>Error

```text
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 1% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 51% cpu; 35% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 36% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 90% cpu; 32% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for the Windows Modules Installer to exit...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'PowerShell -ExecutionPolicy Bypass -OutputFormat Text -File C:/Windows/Temp/packer-windows-update-elevated.ps1' exited with code: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Restarting the machine...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Waiting for machine to become available...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] starting remote command: shutdown.exe -f -r -t 0 -c "packer restart"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'shutdown.exe -f -r -t 0 -c "packer restart"' exited with code: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: A system shutdown is in progress.(1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] starting remote command: shutdown.exe -f -r -t 60 -c "packer restart test"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] command 'shutdown.exe -f -r -t 60 -c "packer restart test"' exited with code: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 Retryable error: Machine not yet available (exit status 1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT Build 'azure-arm' errored: unexpected EOF
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT 
```
#### <a name="cause"></a>Cause

Épuisement des ressources. Ce problème est communément constaté avec Windows Update fonctionnant avec la taille de machine virtuelle de build par défaut D1_V2.

#### <a name="solution"></a>Solution

Augmentez la taille de machine virtuelle de la build.

### <a name="builds-finished-but-no-artifacts-were-created"></a>Builds terminées, mais aucun artefact n’a été créé

#### <a name="error"></a>Error

```text
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT Build 'azure-arm' errored: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 machine readable: azure-arm,error []string{"Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded"}
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT --> azure-arm: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 Cancelling builder after context cancellation context canceled
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 [INFO] (telemetry) Finalizing.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:24 waiting for all plugin processes to complete...
Done exporting Packer logs to Azure for Packer prefix: [a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT
```
#### <a name="cause"></a>Cause

Expiration du délai d’attente provoquée par l’attente de la création des ressources Azure requises.

#### <a name="solution"></a>Solution

Réexécutez la build pour réessayer.

### <a name="resource-not-found"></a>Ressource introuvable

#### <a name="error"></a>Error

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-05-01T00:13:52.599326198Z",
   "endTime": "2020-05-01T00:15:13.62366898Z",
   "runState": "Failed",
   "message": "network.InterfacesClient#UpdateTags: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. Status=404 Code=\"ResourceNotFound\" Message=\"The Resource 'Microsoft.Network/networkInterfaces/aibpls7lz2e.nic.4609d697-be0a-4cb0-86af-49b6fe877fe1' under resource group 'IT_aibImageRG200_window2019VnetTemplate01_9988723b-af56-413a-9006-84130af0e9df' was not found.\""
  },
```
#### <a name="cause"></a>Cause

Autorisations manquantes.

#### <a name="solution"></a>Solution

Revérifiez qu’Azure VM Image Builder dispose de toutes les autorisations nécessaires. 

Pour plus d’informations sur la configuration des autorisations, consultez [Configurer les autorisations de service Azure Image Builder à l’aide d’Azure CLI](image-builder-permissions-cli.md) ou [Configurer les autorisations du service Azure Image Builder à l’aide de PowerShell](image-builder-permissions-powershell.md).

### <a name="sysprep-timing"></a>Minutage Sysprep

#### <a name="error"></a>Error

```text
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service RdAgent) -and ((Get-Service RdAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprepping VM ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: & $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while($true) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Write-Output $imageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Start-Sleep -s 5
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprep complete ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (RdAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Sysprepping VM ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_COMPLETE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: Get-Service : Cannot find any service with service name 'WindowsAzureGuestAgent'.
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: At C:\DeprovisioningScript.ps1:6 char:9
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: + while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service Window ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: +         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + CategoryInfo          : ObjectNotFound: (WindowsAzureGuestAgent:String) [Get-Service], ServiceCommandException
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 Cancelling builder after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT Cancelling build after receiving terminated
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling provisioning due to context cancellation: context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: 
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling hook after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
```
#### <a name="cause"></a>Cause

La cause peut être un problème de minutage en raison de la taille de la machine virtuelle D1_V2. Si les personnalisations sont limitées et s’exécutent en moins de trois secondes, les commandes Sysprep sont exécutées par Azure VM Image Builder pour désapprovisionner. Lorsqu’Azure VM Image Builder est désapprovisionné, la commande Sysprep vérifie si *WindowsAzureGuestAgent* n’est pas entièrement installé, ce qui entraîne un problème de synchronisation. 

#### <a name="solution"></a>Solution

Augmentez la taille de la machine virtuelle. Vous pouvez également ajouter une personnalisation de veille PowerShell de 60 secondes pour éviter le problème de minutage.

### <a name="cancelling-builder-after-context-cancellation-context-canceled"></a>Annulation du générateur après l’annulation du contexte

#### <a name="error"></a>Error
```text
PACKER ERR 2020/03/26 22:11:23 Cancelling builder after context cancellation context canceled
PACKER OUT Cancelling build after receiving terminated
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling hook after context cancellation context canceled
..
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling provisioning due to context cancellation: context canceled
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [ERROR] Remote command exited without exit status or exit signal.
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [INFO] RPC endpoint: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] 148974 bytes written for 'stdout'
PACKER ERR 2020/03/26 22:11:25 [INFO] 0 bytes written for 'stderr'
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC client: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC endpoint: Communicator ended with: 2300218
```
#### <a name="cause"></a>Cause
Le service Image Builder utilise le port 22 (Linux) ou 5986 (Windows) pour se connecter à la machine virtuelle de build, ce qui se produit lorsque le service est déconnecté de la machine virtuelle de build pendant une génération d’image. Les raisons de la déconnexion peuvent varier, mais l’activation ou la configuration de pare-feu dans le script peut bloquer les ports ci-dessus.

#### <a name="solution"></a>Solution
Passez en revue vos scripts pour connaître les modifications/activations du pare-feu ou les modifications de SSH ou WinRM, et assurez-vous que les modifications autorisent une connectivité constante entre le service et la machine virtuelle de build sur les ports ci-dessus. Pour plus d’informations sur la mise en réseau d’Image Builder, consultez les [conditions requises](./image-builder-networking.md).

## <a name="devops-task"></a>Tâche DevOps 

### <a name="troubleshooting-the-task"></a>Résolution des problèmes liés à la tâche
La tâche échoue uniquement si une erreur se produit pendant la personnalisation. Lorsque cela se produit, la tâche signale l’échec et quitte le groupe de ressources intermédiaire avec les journaux afin que vous puissiez identifier le problème. 

Pour localiser le journal, vous devez connaître le nom du modèle, accéder au pipeline > échec de la build > accéder à la tâche AIB DevOps. Vous verrez alors le journal et le nom du modèle qui s’affichent :
```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
``` 

Accédez au portail, recherchez le nom du modèle dans le groupe de ressources, puis recherchez le groupe de ressources avec IT_*.
Accédez au compte de stockage > blobs > conteneurs > journaux.

### <a name="troubleshooting-successful-builds"></a>Résolution des problèmes de builds réussies
Il peut arriver que vous deviez examiner les builds réussies et que vous souhaitiez consulter le journal. Comme mentionné, si la génération de l’image réussit, le groupe de ressources intermédiaire qui contient les journaux sera supprimé dans le cadre du nettoyage. Cependant, ce que vous pouvez faire, c’est introduire une mise en veille après la commande inlined, puis récupérer les journaux lorsque la build est suspendue. Pour ce faire, procédez comme suit :
 
1. Mettez à jour la commande inlined et ajoutez : Write-Host / Echo “Sleep” : cela vous permet d’effectuer une recherche dans le journal.
2. Ajoutez une mise en veille d’au moins 10 minutes : vous pouvez utiliser [Start-Sleep](/powershell/module/microsoft.powershell.utility/start-sleep) ou la commande Linux `Sleep`.
3. Utilisez la méthode ci-dessus pour identifier l’emplacement du journal, puis continuez à télécharger/vérifier le journal jusqu’à ce qu’il soit en veille.


### <a name="operation-was-canceled"></a>Opération annulée

#### <a name="error"></a>Error

```text
2020-05-05T18:28:24.9280196Z ##[section]Starting: Azure VM Image Builder Task
2020-05-05T18:28:24.9609966Z ==============================================================================
2020-05-05T18:28:24.9610739Z Task         : Azure VM Image Builder Test(Preview)
2020-05-05T18:28:24.9611277Z Description  : Build images using Azure Image Builder resource provider.
2020-05-05T18:28:24.9611608Z Version      : 1.0.18
2020-05-05T18:28:24.9612003Z Author       : Microsoft Corporation
2020-05-05T18:28:24.9612718Z Help         : For documentation, and end to end example, please visit: http://aka.ms/azvmimagebuilderdevops
2020-05-05T18:28:24.9613390Z ==============================================================================
2020-05-05T18:28:26.0651512Z start reading task parameters...
2020-05-05T18:28:26.0673377Z found build at:  d:\a\r1\a\_AppsAndImageBuilder\webApp
2020-05-05T18:28:26.0708785Z end reading parameters
2020-05-05T18:28:26.0745447Z getting storage account details for aibstagstor1565047758
2020-05-05T18:28:29.8812270Z created archive d:\a\_temp\temp_web_package_09737279437949953.zip
2020-05-05T18:28:33.1568013Z Source for image:  { type: 'PlatformImage',
2020-05-05T18:28:33.1584131Z   publisher: 'MicrosoftWindowsServer',
2020-05-05T18:28:33.1585965Z   offer: 'WindowsServer',
2020-05-05T18:28:33.1592768Z   sku: '2016-Datacenter',
2020-05-05T18:28:33.1594191Z   version: '14393.3630.2004101604' }
2020-05-05T18:28:33.1595387Z template name:  t_1588703313152
2020-05-05T18:28:33.1597453Z starting put template...
2020-05-05T18:28:52.9278603Z put template:  Succeeded
2020-05-05T18:28:52.9281282Z starting run template...
2020-05-05T19:33:14.3923479Z ##[error]The operation was canceled.
2020-05-05T19:33:14.3939721Z ##[section]Finishing: Azure VM Image Builder Task
```
#### <a name="cause"></a>Cause

Si la build n’a pas été annulée par un utilisateur, elle a été annulée par l’agent utilisateur Azure DevOps. L’expiration du délai d’une heure s’est probablement produit en raison des capacités d’Azure DevOps. Si vous utilisez un projet et un agent privés, vous bénéficiez de 60 minutes de temps de build. Si le délai d’attente de la build expire, DevOps annule la tâche en cours d’exécution.

Pour plus d’informations sur les capacités et limitations d’Azure DevOps, consultez [Agents hébergés par Microsoft](/azure/devops/pipelines/agents/hosted?view=azure-devops#capabilities-and-limitations).
 
#### <a name="solution"></a>Solution

Vous pouvez héberger vos propres agents DevOps ou chercher à réduire la durée de votre build. Par exemple, si vous distribuez vers Shared Image Gallery, effectuez une réplication vers une région si vous souhaitez répliquer de façon asynchrone. 

### <a name="slow-windows-logon-please-wait-for-the-windows-modules-installer"></a>Ouverture de session Windows lente : « Veuillez patienter pendant l’installation des modules Windows »

#### <a name="error"></a>Erreur
Après avoir créé une image Windows 10 avec Image Builder, créez une machine virtuelle à partir de l’image, votre RDP, puis attendez quelques minutes lors la première ouverture de session qu’un écran bleu s’affiche avec le message suivant :
```text
Please wait for the Windows Modules Installer
```

#### <a name="solution"></a>Solution
Tout d’abord, dans la génération d’image, vérifiez qu’aucun redémarrage en attente n’est requis en ajoutant un personnalisateur de redémarrage Windows comme dernière personnalisation, et que l’installation du logiciel est terminée. Enfin, ajoutez l’option [/mode:vm](/windows-hardware/manufacture/desktop/sysprep-command-line-options) au Sysprep par défaut utilisé par AIB. Consultez « Les machines virtuelles créées à partir d’images AIB ne sont pas créées correctement » > « Remplacement de commandes » ci-dessous  

 
## <a name="vms-created-from-aib-images-do-not-create-successfully"></a>Les machines virtuelles créées à partir d’images AIB ne sont pas créées correctement

Par défaut, Azure VM Image Builder exécute du code de *désapprovisionnement* à la fin de chaque phase de personnalisation d’image pour *généraliser* l’image. Généraliser est un processus dans lequel l’image est configurée pour être réutilisée afin de créer plusieurs machines virtuelles. Vous pouvez ainsi transmettre les paramètres de la machine virtuelle, tels que le nom d’hôte, le nom d’utilisateur, etc. Azure VM Image Builder exécute *Sysprep* pour Windows et `waagent -deprovision` pour Linux. 

Pour Windows, Azure VM Image Builder utilise une commande Sysprep générique. Toutefois, cela peut ne pas convenir à chaque généralisation Windows réussie. Azure VM Image Builder vous permet de personnaliser la commande Sysprep. Notez qu’Azure VM Image Builder est un outil d’automatisation d’image. Il est responsable de l’exécution réussie de la commande Sysprep. Toutefois, vous pouvez avoir besoin de différentes commandes Sysprep pour rendre votre image réutilisable. Pour Linux, Azure VM Image Builder utilise une commande `waagent -deprovision+user` générique. Pour plus d’informations, consultez [Documentation de l’agent Linux Microsoft Azure](https://github.com/Azure/WALinuxAgent#command-line-options).

Si vous migrez une personnalisation existante et que vous utilisez différentes commandes Sysprep/waagent, vous pouvez essayer les commandes génériques du générateur. Si la création de la machine virtuelle échoue, utilisez vos commandes Sysprep/waagent précédentes.

> [!NOTE]
> Si AIB crée une image personnalisée Windows avec succès et que vous créez une machine virtuelle à partir de celle-ci, puis que vous constatez que la machine virtuelle ne sera pas créée correctement (par exemple, la commande de création de machine virtuelle ne se termine pas/expire), vous devez consulter la documentation Sysprep de Windows Server. Vous pouvez également adresser une demande de support au support technique du service clientèle de Windows Server Sysprep, qui peut dépanner et conseiller sur la commande Sysprep appropriée.

### <a name="command-locations-and-filenames"></a>Emplacements de commande et noms de fichiers

Windows :

```
c:\DeprovisioningScript.ps1
```

Linux :
```bash
/tmp/DeprovisioningScript.sh
```

### <a name="sysprep-command-windows"></a>Commande Sysprep : Windows

```PowerShell
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```

### <a name="deprovision-command-linux"></a>Commande de désapprovisionnement : Linux

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

### <a name="overriding-the-commands"></a>Remplacement de commandes

Pour remplacer les commandes, utilisez les fournisseurs de script PowerShell ou de l’interpréteur de commandes pour créer les fichiers de commandes avec le nom de fichier exact, puis placez-les dans les répertoires indiqués précédemment. Azure VM Image Builder lit ces commandes et la sortie est écrite dans le fichier *customization.log*.

## <a name="getting-support"></a>Obtenir de l'aide
Si vous vous êtes référé aux instructions et que vous ne parvenez toujours pas à résoudre votre problème, vous pouvez créer une demande de support. Lorsque vous effectuez cette opération, sélectionnez le produit et la rubrique de support appropriés, ce qui permettra de solliciter le support technique d’Azure VM Image Builder.

Sélection du produit de la demande :
```bash
Product Family: Azure
Product: Virtual Machine Running (Window\Linux)
Support Topic: Azure Features
Support Subtopic: Azure Image Builder
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Vue d’ensemble d’Azure VM Image Builder](image-builder-overview.md).