---
title: Création d’un pool d’hôtes pour un environnement Windows Virtual Desktop – Azure
description: Découvrez comment détecter et résoudre les problèmes de locataire et de pool d’hôtes lors de la configuration d’un environnement Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 09/14/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 126a8e48a8db1c41299a7cb7a34f172342110667
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023120"
---
# <a name="host-pool-creation"></a>Création d’un pool d’hôtes

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/troubleshoot-set-up-issues-2019.md).

Cet article décrit les problèmes pouvant survenir lors de la configuration initiale du locataire Windows Virtual Desktop et de l’infrastructure associée du pool d’hôtes de la session.

## <a name="provide-feedback"></a>Fournir des commentaires

Rendez-vous sur le site [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) pour discuter du service Windows Virtual Desktop avec l’équipe de produit et les membres actifs de la communauté.

## <a name="acquiring-the-windows-10-enterprise-multi-session-image"></a>Acquisition de l’image Windows 10 Entreprise multisession

Pour utiliser l’image multisession Windows 10 Entreprise, accédez à la Place de marché Azure, sélectionnez **Prise en main** > **Microsoft Windows 10** > [Windows 10 Enterprise multi-session, Version 1809](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsdesktop.windows-10?tab=PlansAndPrice) (Windows 10 Entreprise multisession, version 1809).

## <a name="issues-with-using-the-azure-portal-to-create-host-pools"></a>Problèmes liés à l’utilisation du portail Azure pour créer des pools d’hôtes

### <a name="error-create-a-free-account-appears-when-accessing-the-service"></a>Erreur : « Créer un compte gratuit » s’affiche lors de l’accès au service

> [!div class="mx-imgBorder"]
> ![Image du portail Azure affichant le message « Créer un compte gratuit »](media/create-new-account.png)

**Cause** : Aucun abonnement n’est actif dans le compte que vous avez utilisé pour vous connecter à Azure, ou le compte n’est pas autorisé à visualiser les abonnements.

**Correctif** : Connectez-vous à l’abonnement dans lequel vous allez déployer les machines virtuelles hôtes de session avec un compte disposant au moins d’un accès de niveau contributeur.

### <a name="error-exceeding-quota-limit"></a>Erreur : « Dépassement de la limite de quota »

Si votre opération dépasse la limite de quota, vous pouvez effectuer une des actions suivantes :

- Créez un pool d’hôtes avec les mêmes paramètres, mais avec moins de machines virtuelles et de cœurs de machines virtuelles.

- Ouvrez dans un navigateur le lien que vous voyez dans le champ statusMessage pour envoyer une demande d’augmentation du quota de votre abonnement Azure pour la référence SKU de machine virtuelle spécifiée.

### <a name="error-cant-see-user-assignments-in-app-groups"></a>Erreur : Impossible de voir les affectations d’utilisateurs dans les groupes d’applications.

Cause : Cette erreur se produit généralement après que vous avez déplacé l’abonnement d’un locataire Azure Active Directory (AD) vers un autre. Si vos anciennes attributions restent liées à l’ancien locataire Azure AD, le portail Azure en perd la trace.

Correctif : Vous devez réaffecter les utilisateurs aux groupes d’applications.

## <a name="azure-resource-manager-template-errors"></a>Erreurs de modèle Azure Resource Manager

Suivez les instructions ci-après pour détecter les problèmes d’échecs de déploiements de modèles Azure Resource Manager et DSC PowerShell.

1. Passez en revue les erreurs du déploiement à l’aide de l’article [Afficher les opérations de déploiement avec Azure Resource Manager](../azure-resource-manager/templates/deployment-history.md).
2. Si le déploiement est dépourvu d’erreurs, passez en revue les erreurs dans le journal d’activité à l’aide de l’article [Afficher les journaux d’activité pour auditer les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md).
3. Une fois l’erreur identifiée, utilisez le message d’erreur et les ressources figurant dans l’article [Résolution des erreurs courantes dans des déploiements Azure avec Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md) pour résoudre le problème.
4. Supprimez toutes les ressources créées lors du déploiement précédent, puis essayez de redéployer le modèle.

### <a name="error-your-deployment-failedhostnamejoindomain"></a>Erreur : Votre déploiement a échoué….\<hostname>/joindomain

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement](media/failure-joindomain.png)

Exemple d’erreur brute :

```Error
 {"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details.
 Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\": \"Failed\",\r\n \"error\":
 {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.
 \",\r\n \"details\": [\r\n {\r\n \"code\": \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has reported a failure when processing
 extension 'joindomain'. Error message: \\\"Exception(s) occurred while joining Domain 'diamondsg.onmicrosoft.com'\\\".\"\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Cause 1 :** Les informations d’identification fournies pour la jonction des machines virtuelles au domaine sont incorrectes.

**Correctif 1 :** Consultez la section « Error: Incorrect credentials » (Erreur : Informations d’identification incorrectes) concernant les machines virtuelles qui ne sont pas jointes au domaine dans l’article [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).

**Cause 2 :** Le nom de domaine est impossible à résoudre.

**Correctif 2 :** Consultez [Erreur : Le nom de domaine ne se résout pas](troubleshoot-vm-configuration.md#error-domain-name-doesnt-resolve) dans [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).

**Cause 3 :** La configuration DNS de votre réseau virtuel (VNET) est définie sur **Par défaut**.

Pour corriger cela, effectuez les actions suivantes :

1. Ouvrez le portail Azure et accédez à l’onglet **Réseaux virtuels**.
2. Recherchez votre réseau virtuel, puis sélectionnez **Serveurs DNS**.
3. Le menu Serveurs DNS doit apparaître sur le côté droit de l’écran. Dans ce menu, sélectionnez **Personnalisé**.
4. Vérifiez que les serveurs DNS listés sous Personnalisé correspondent à votre contrôleur de domaine ou à votre domaine Active Directory. Si vous ne voyez pas votre serveur DNS, vous pouvez l’ajouter en entrant sa valeur dans le champ **Ajouter un serveur DNS**.

### <a name="error-your-deployment-failedunauthorized"></a>Erreur : Votre déploiement a échoué...\Non autorisé

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Unauthorized","message":"{\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Target\": null,\r\n \"Details\": [\r\n {\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n },\r\n {\r\n \"Code\": \"Unauthorized\"\r\n },\r\n {\r\n \"ErrorEntity\": {\r\n \"ExtendedCode\": \"52020\",\r\n \"MessageTemplate\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Parameters\": [\r\n \"default\"\r\n ],\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n }\r\n }\r\n ],\r\n \"Innererror\": null\r\n}"}]}
```

**Cause :** L’abonnement que vous utilisez est d’un type qui ne peut pas accéder aux fonctionnalités requises dans la région où le client tente de déployer. Par exemple, les abonnements MSDN, Gratuit ou Éducation peuvent afficher cette erreur.

**Correctif :** Modifiez votre type d’abonnement ou votre région pour pouvoir accéder aux fonctionnalités requises.

### <a name="error-vmextensionprovisioningerror"></a>Erreur : VMExtensionProvisioningError

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement indiquant l’échec du provisionnement du terminal](media/failure-vmextensionprovisioning.png)

**Cause 1 :** Erreur temporaire au niveau de l’environnement Windows Virtual Desktop.

**Cause 2 :** Erreur temporaire au niveau de la connexion.

**Correctif :** Vérifiez que l’environnement Windows Virtual Desktop est sain en vous connectant à l’aide de PowerShell. Terminez l’inscription de la machine virtuelle manuellement à l’aide de l’article [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell.md).

### <a name="error-the-admin-username-specified-isnt-allowed"></a>Erreur : Le nom d’utilisateur spécifié pour l’administrateur n’est pas autorisé

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement dans lequel un nom d’administrateur spécifié n’est pas autorisé](media/failure-username.png)

Exemple d’erreur brute :

```Error
 { …{ "provisioningOperation":
 "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T20:53:18.904917Z", "duration": "PT3.0574505S", "trackingId":
 "1f460af8-34dd-4c03-9359-9ab249a1a005", "statusCode": "BadRequest", "statusMessage": { "error": { "code": "InvalidParameter", "message":
 "The Admin Username specified is not allowed.", "target": "adminUsername" } … }
```

**Cause :** Le mot de passe fourni contient des sous-chaînes interdites (admin, administrateur, racine).

**Correctif :** Modifiez le nom d’utilisateur ou utilisez différents utilisateurs.

### <a name="error-vm-has-reported-a-failure-when-processing-extension"></a>Erreur : La machine virtuelle a signalé un échec lors du traitement de l’extension

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement indiquant que l’opération de ressource s’est terminée avec l’échec du provisionnement du terminal](media/failure-processing.png)

Exemple d’erreur brute :

```Error
{ … "code": "ResourceDeploymentFailure", "message":
 "The resource operation completed with terminal provisioning state 'Failed'.", "details": [ { "code":
 "VMExtensionProvisioningError", "message": "VM has reported a failure when processing extension 'dscextension'.
 Error message: \"DSC Configuration 'SessionHost' completed with error(s). Following are the first few:
 PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource functionality with error message:
 One or more errors occurred. The SendConfigurationApply function did not succeed.\"." } ] … }
```

**Cause :** L’extension DSC PowerShell n’a pas pu obtenir un accès administrateur sur la machine virtuelle.

**Correctif :** Vérifiez que le nom d’utilisateur et le mot de passe disposent d’un accès administrateur sur la machine virtuelle, puis réexécutez le modèle Azure Resource Manager.

### <a name="error-deploymentfailed--powershell-dsc-configuration-firstsessionhost-completed-with-errors"></a>Erreur : DeploymentFailed - La configuration DSC PowerShell de « FirstSessionHost » s’est terminée avec des erreurs

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement indiquant que la configuration DSC PowerShell de « FirstSessionHost » s’est terminée avec des erreurs](media/failure-dsc.png)

Exemple d’erreur brute :

```Error
{
    "code": "DeploymentFailed",
   "message": "At least one resource deployment operation failed. Please list
 deployment operations for details. 4 Please see https://aka.ms/arm-debug for usage details.",
 "details": [
         { "code": "Conflict",
         "message": "{\r\n \"status\": \"Failed\",\r\n \"error\": {\r\n \"code\":
         \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource
         operation completed with terminal provisioning state 'Failed'.\",\r\n
         \"details\": [\r\n {\r\n \"code\":
        \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has
              reported a failure when processing extension 'dscextension'.
              Error message: \\\"DSC Configuration 'FirstSessionHost'
              completed with error(s). Following are the first few:
              PowerShell DSC resource MSFT ScriptResource failed to
              execute Set-TargetResource functionality with error message:
              One or more errors occurred. The SendConfigurationApply
              function did not succeed.\\\".\"\r\n }\r\n ]\r\n }\r\n}"  }

```

**Cause :** L’extension DSC PowerShell n’a pas pu obtenir un accès administrateur sur la machine virtuelle.

**Correctif :** Vérifiez que le nom d’utilisateur et le mot de passe fournis disposent d’un accès administrateur sur la machine virtuelle, puis réexécutez le modèle Azure Resource Manager.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Erreur : DeploymentFailed - InvalidResourceReference

Exemple d’erreur brute :

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/networkInterfaces/erd. Please make sure that
the referenced resource exists, and that both resources are in the same
region.\\\",\\r\\n\\\"details\\\": []\\r\\n }\\r\\n}\"\r\n }\r\n ]\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Cause :** Une partie du nom du groupe de ressources est utilisée pour certaines des ressources créées par le modèle. Étant donné que ce nom correspond à des ressources existantes, le modèle risque de sélectionner une ressource existante appartenant à un autre groupe.

**Correctif :** Lorsque vous exécutez le modèle Azure Resource Manager pour déployer des machines virtuelles hôtes de la session, faites en sorte que les deux premiers caractères du nom du groupe de ressources d’abonnement soient uniques.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Erreur : DeploymentFailed - InvalidResourceReference

Exemple d’erreur brute :

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/DEMO/providers/Microsoft.Network/networkInterfaces
/EXAMPLE was not found. Please make sure that the referenced resource exists, and that both
resources are in the same region.\\\",\\r\\n \\\"details\\\": []\\r\\n }\\r\\n}\"\r\n
}\r\n ]\r\n }\r\n ]\r\n }\r\n\
```

**Cause :** Cette erreur découle du fait que la carte réseau créée avec le modèle Azure Resource Manager porte le même nom qu’une autre carte réseau figurant déjà dans le réseau virtuel.

**Correctif :** Utilisez un autre préfixe d’hôte.

### <a name="error-deploymentfailed--error-downloading"></a>Erreur : DeploymentFailed - Erreur de téléchargement

Exemple d’erreur brute :

```Error
\\\"The DSC Extension failed to execute: Error downloading
https://catalogartifact.azureedge.net/publicartifacts/rds.wvd-provision-host-pool-
2dec7a4d-006c-4cc0-965a-02bbe438d6ff-prod
/Artifacts/DSC/Configuration.zip after 29 attempts: The remote name could not be
resolved: 'catalogartifact.azureedge.net'.\\nMore information about the failure can
be found in the logs located under
'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.77.0.0' on
the VM.\\\"
```

**Cause :** Cette erreur découle du fait qu’une route statique, une règle de pare-feu ou un Groupe de sécurité réseau (NSG) bloquent le téléchargement du fichier zip lié au modèle Azure Resource Manager.

**Correctif :** Supprimez la route statique, la règle de pare-feu ou le NSG qui bloquent le téléchargement. Si vous le souhaitez, ouvrez le fichier json du modèle Azure Resource Manager dans un éditeur de texte, recherchez le lien d’accès au fichier zip, puis téléchargez la ressource à un emplacement autorisé.

### <a name="error-cant-delete-a-session-host-from-the-host-pool-after-deleting-the-vm"></a>Erreur : Impossible de supprimer un hôte de session du pool hôte après la suppression de la machine virtuelle

**Cause :** Vous devez supprimer l’hôte de session avant de supprimer la machine virtuelle.

**Correctif :** Mettez l’hôte de session en mode maintenance, déconnectez tous les utilisateurs de l’hôte de session, puis supprimez l’hôte.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir une vue d’ensemble de la résolution des problèmes Windows Virtual Desktop et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).
- Pour résoudre les problèmes de connexion au client Windows Virtual Desktop, consultez [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection.md).
- Pour résoudre les problèmes liés aux clients Bureau à distance, consultez [Résoudre des problèmes du client Bureau à distance](troubleshoot-client.md).
- Pour résoudre les problèmes d’utilisation de PowerShell avec Windows Virtual Desktop, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md).
- Pour plus d’informations sur le service, consultez [Environnement Windows Virtual Desktop](environment-setup.md).
- Suivez le [Didacticiel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Pour en savoir plus sur les actions d’audit, consultez [Opérations d’audit avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Pour en savoir plus sur les actions visant à déterminer les erreurs au cours du déploiement, consultez [Voir les opérations de déploiement](../azure-resource-manager/templates/deployment-history.md).