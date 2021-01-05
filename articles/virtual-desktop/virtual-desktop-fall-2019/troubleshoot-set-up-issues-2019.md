---
title: Création d’un pool d’hôtes et d’un locataire Windows Virtual Desktop (classique) – Azure
description: Découvrez comment détecter et résoudre les problèmes de locataire et de pool d’hôtes lors de la configuration d’un environnement client Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 8c6d26de62364b6aca671d1e4283a01c1b78c397
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95014830"
---
# <a name="tenant-and-host-pool-creation-in-windows-virtual-desktop-classic"></a>Création d’un locataire et d’un pool d’hôtes dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../troubleshoot-set-up-issues.md).

Cet article décrit les problèmes pouvant survenir lors de la configuration initiale du locataire Windows Virtual Desktop et de l’infrastructure associée du pool d’hôtes de la session.

## <a name="provide-feedback"></a>Fournir des commentaires

Rendez-vous sur le site [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) pour discuter du service Windows Virtual Desktop avec l’équipe de produit et les membres actifs de la communauté.

## <a name="acquiring-the-windows-10-enterprise-multi-session-image"></a>Acquisition de l’image Windows 10 Entreprise multisession

Pour utiliser l’image multisession Windows 10 Entreprise, accédez à la Place de marché Azure, sélectionnez **Prise en main** > **Microsoft Windows 10** > [Windows 10 Enterprise for Virtual Desktops, Version 1809](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsdesktop.windows-10?tab=PlansAndPrice) (Windows 10 Entreprise pour postes de travail virtuels, version 1809).

> [!div class="mx-imgBorder"]
> ![Capture d’écran de la sélection de Windows 10 Entreprise pour postes de travail virtuels, version 1809.](../media/AzureMarketPlace.png)

## <a name="creating-windows-virtual-desktop-tenant"></a>Création d’un locataire Windows Virtual Desktop

Cette section décrit les problèmes potentiels lors de la création du locataire Windows Virtual Desktop.

### <a name="error-aadsts650052-the-app-needs-access-to-a-service"></a>Erreur : AADSTS650052 L’application a besoin d’accéder à un service.

Exemple d’erreur brute :

```Error
AADSTS650052 Message The app needs access to a service(\"{name}\") that your organization
\"{organization}\" has not subscribed to or enabled. Contact your IT Admin to review the
configuration of your service subscriptions.650052 Message The app needs access to a service
(\"{name}\") that your organization \"{organization}\" has not subscribed to or enabled.
Contact your IT Admin to review the configuration of your service subscriptions.
```

**Cause :** Consentement non accordé à Windows Virtual Desktop dans l’instance Azure Active Directory.

**Correctif :** [Suivez ce guide](./tenant-setup-azure-active-directory.md#grant-permissions-to-windows-virtual-desktop) pour donner votre consentement.

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Erreur : L’utilisateur n’est pas autorisé à interroger le service de gestion

> [!div class="mx-imgBorder"]
> ![Capture d’écran PowerShell indiquant qu’un utilisateur n’est pas autorisé à interroger le service de gestion](../media/UserNotAuthorizedNewTenant.png)

Exemple d’erreur brute :

```Error
   New-RdsTenant : User isn't authorized to query the management service.
   ActivityId: ad604c3a-85c6-4b41-9b81-5138162e5559
   Powershell commands to diagnose the failure:
   Get-RdsDiagnosticActivities -ActivityId ad604c3a-85c6-4b41-9b81-5138162e5559
   At line:1 char:1
   + New-RdsTenant -Name "testDesktopTenant" -AadTenantId "01234567-89ab-c ...
   + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       + CategoryInfo          : FromStdErr: (Microsoft.RDInf...nt.NewRdsTenant:NewRdsTenant) [New-RdsTenant], RdsPowerSh
      ellException
       + FullyQualifiedErrorId : UnauthorizedAccess,Microsoft.RDInfra.RDPowershell.Tenant.NewRdsTenant
```

**Cause :** L’utilisateur qui s’est connecté n’est pas doté du rôle TenantCreator dans son instance Azure Active Directory.

**Correctif :** Suivez les instructions de l’article [Attribuer le rôle d’application TenantCreator à un utilisateur dans votre locataire Azure Active Directory](tenant-setup-azure-active-directory.md#assign-the-tenantcreator-application-role). Après avoir suivi la procédure, vous disposerez d’un utilisateur doté du rôle TenantCreator.

> [!div class="mx-imgBorder"]
> ![Capture d’écran de l’attribution du rôle TenantCreator](../media/TenantCreatorRoleAssigned.png)

## <a name="creating-windows-virtual-desktop-session-host-vms"></a>Création de machines virtuelles hôtes de la session Windows Virtual Desktop

Vous pouvez créer des machines virtuelles hôtes de la session de plusieurs manières, mais l’équipe Windows Virtual Desktop aide uniquement à résoudre les problèmes de provisionnement de machines virtuelles liés à l’offre de la [Place de marché Azure](https://azuremarketplace.microsoft.com/). Pour plus d’informations, consultez [Problèmes d’utilisation de Windows Virtual Desktop - Provisionner un pool d’hôtes - Offre de la Place de marché Azure](#issues-using-windows-virtual-desktop--provision-a-host-pool-azure-marketplace-offering).

## <a name="issues-using-windows-virtual-desktop--provision-a-host-pool-azure-marketplace-offering"></a>Problèmes d’utilisation de l’offre Windows Virtual Desktop - Provision a host pool de la Place de marché Azure

Le modèle Windows Virtual Desktop - Provision a host pool est accessible à partir de la Place de marché Azure.

### <a name="error-when-using-the-link-from-github-the-message-create-a-free-account-appears"></a>Erreur : Quand vous cliquez sur le lien GitHub, le message « Créer un compte gratuit » s’affiche

> [!div class="mx-imgBorder"]
> ![Capture d’écran de création d’un compte gratuit](../media/be615904ace9832754f0669de28abd94.png)

**Cause 1 :** Aucun abonnement n’est actif dans le compte utilisé pour la connexion à Azure, ou le compte utilisé n’est pas autorisé à visualiser les abonnements.

**Correctif 1 :** Connectez-vous avec un compte disposant d’un accès Contributeur (au minimum) à l’abonnement dans lequel les machines virtuelles hôtes de la session doivent être déployées.

**Cause 2 :** L’abonnement utilisé fait partie d’un locataire fournisseur de services cloud (CSP) Microsoft.

**Correctif 2 :** Accédez à l’emplacement GitHub de l’étape **Créer et approvisionner un nouveau pool d’hôtes Windows Virtual Desktop**, puis suivez les instructions ci-après :

1. Cliquez avec le bouton droit sur **Déployer sur Azure**, puis sélectionnez **Copier l’adresse du lien**.
2. Ouvrez le **Bloc-notes** et collez-y le lien.
3. Devant le caractère #, insérez le nom de locataire du client final CSP.
4. Ouvrez le nouveau lien dans un navigateur ; le Portail Azure charge alors le modèle.

    ```Example
    Example: https://portal.azure.com/<CSP end customer tenant name>
    #create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%
    2FRDS-Templates%2Fmaster%2Fwvd-templates%2FCreate%20and%20provision%20WVD%20host%20pool%2FmainTemplate.json
    ```

### <a name="error-you-receive-template-deployment-is-not-valid-error"></a>Erreur : Vous recevez une erreur « Le déploiement du modèle n’est pas valide »

> [!div class="mx-imgBorder"]
> ![Capture d’écran de l’erreur « Le déploiement du modèle ... n’est pas valide »](../media/troubleshooting-marketplace-validation-error-generic.png)

Avant d’effectuer une action spécifique, vous devez vérifier le journal d’activité pour voir l’erreur détaillée concernant la validation du déploiement qui a échoué.

Pour voir l’erreur dans le journal d’activité :

1. Quittez l’offre de déploiement actuelle de la Place de marché Azure.
2. Dans la barre de recherche du haut, recherchez et sélectionnez **Journal d’activité**.
3. Recherchez une activité nommée **Valider le déploiement** dont l’état est **Échec** et sélectionnez l’activité.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran de l’activité **Valider le déploiement** avec un état **Échec**](../media/troubleshooting-marketplace-validation-error-activity-summary.png)

4. Sélectionnez JSON, puis faites défiler jusqu’au bas de l’écran jusqu’à ce que le champ « statusMessage » apparaisse.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran de l’activité en échec, avec un encadré rouge autour de la propriété statusMessage du texte JSON.](../media/troubleshooting-marketplace-validation-error-json-boxed.png)

Si votre modèle d’opération dépasse la limite de quota, vous pouvez effectuer une des actions suivantes pour résoudre cela :

 - Exécutez la Place de marché Azure avec les paramètres que vous avez utilisés la première fois, mais utilisez cette fois moins de machines virtuelles et de cœurs de machines virtuelles.
 - Ouvrez dans un navigateur le lien que vous voyez dans le champ **statusMessage** pour envoyer une demande d’augmentation du quota de votre abonnement Azure pour la référence (SKU) de machine virtuelle spécifiée.

## <a name="azure-resource-manager-template-and-powershell-desired-state-configuration-dsc-errors"></a>Erreurs de modèle Azure Resource Manager et Desired State Configuration (DSC) PowerShell

Suivez les instructions ci-après pour détecter les problèmes d’échecs de déploiements de modèles Azure Resource Manager et DSC PowerShell.

1. Passez en revue les erreurs du déploiement à l’aide de l’article [Afficher les opérations de déploiement avec Azure Resource Manager](../../azure-resource-manager/templates/deployment-history.md).
2. Si le déploiement est dépourvu d’erreurs, passez en revue les erreurs dans le journal d’activité à l’aide de l’article [Afficher les journaux d’activité pour auditer les actions sur les ressources](../../azure-resource-manager/management/view-activity-logs.md).
3. Une fois l’erreur identifiée, utilisez le message d’erreur et les ressources figurant dans l’article [Résolution des erreurs courantes dans des déploiements Azure avec Azure Resource Manager](../../azure-resource-manager/templates/common-deployment-errors.md) pour résoudre le problème.
4. Supprimez toutes les ressources créées lors du déploiement précédent, puis essayez de redéployer le modèle.

### <a name="error-your-deployment-failedhostnamejoindomain"></a>Erreur : Votre déploiement a échoué….\<hostname>/joindomain

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement](../media/e72df4d5c05d390620e07f0d7328d50f.png)

Exemple d’erreur brute :

```Error
 {"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details.
 Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\": \"Failed\",\r\n \"error\":
 {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.
 \",\r\n \"details\": [\r\n {\r\n \"code\": \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has reported a failure when processing
 extension 'joindomain'. Error message: \\\"Exception(s) occurred while joining Domain 'diamondsg.onmicrosoft.com'\\\".\"\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Cause 1 :** Les informations d’identification fournies pour la jonction des machines virtuelles au domaine sont incorrectes.

**Correctif 1 :** Consultez la section « Error: Incorrect credentials » (Erreur : Informations d’identification incorrectes) concernant les machines virtuelles qui ne sont pas jointes au domaine dans l’article [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration-2019.md).

**Cause 2 :** Le nom de domaine est impossible à résoudre.

**Correctif 2 :** Consultez [Erreur : Le nom de domaine ne se résout pas](troubleshoot-vm-configuration-2019.md#error-domain-name-doesnt-resolve) dans [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration-2019.md).

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
> ![Capture d’écran du message d’échec du déploiement indiquant l’échec du provisionnement du terminal](../media/7aaf15615309c18a984673be73ac969a.png)

**Cause 1 :** Erreur temporaire au niveau de l’environnement Windows Virtual Desktop.

**Cause 2 :** Erreur temporaire au niveau de la connexion.

**Correctif :** Vérifiez que l’environnement Windows Virtual Desktop est sain en vous connectant à l’aide de PowerShell. Terminez l’inscription de la machine virtuelle manuellement à l’aide de l’article [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell-2019.md).

### <a name="error-the-admin-username-specified-isnt-allowed"></a>Erreur : Le nom d’utilisateur spécifié pour l’administrateur n’est pas autorisé

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement dans lequel un nom d’administrateur spécifié n’est pas autorisé](../media/f2b3d3700e9517463ef88fa41875bac9.png)

Exemple d’erreur brute :

```Error
 { "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostDesktop/providers/Microsoft.
  Resources/deployments/vmCreation-linkedTemplate/operations/EXAMPLE", "operationId": "EXAMPLE", "properties": { "provisioningOperation":
 "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T20:53:18.904917Z", "duration": "PT3.0574505S", "trackingId":
 "1f460af8-34dd-4c03-9359-9ab249a1a005", "statusCode": "BadRequest", "statusMessage": { "error": { "code": "InvalidParameter", "message":
 "The Admin Username specified is not allowed.", "target": "adminUsername" } }, "targetResource": { "id": "/subscriptions/EXAMPLE
 /resourceGroups/demoHostDesktop/providers/Microsoft.Compute/virtualMachines/demo", "resourceType": "Microsoft.Compute/virtualMachines", "resourceName": "demo" } }}
```

**Cause :** Le mot de passe fourni contient des sous-chaînes interdites (admin, administrateur, racine).

**Correctif :** Modifiez le nom d’utilisateur ou utilisez différents utilisateurs.

### <a name="error-vm-has-reported-a-failure-when-processing-extension"></a>Erreur : La machine virtuelle a signalé un échec lors du traitement de l’extension

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement indiquant que l’opération de ressource s’est terminée avec l’échec du provisionnement du terminal](../media/49c4a1836a55d91cd65125cf227f411f.png)

Exemple d’erreur brute :

```Error
{ "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostD/providers/Microsoft.Resources/deployments/
 rds.wvd-provision-host-pool-20190129132410/operations/5A0757AC9E7205D2", "operationId": "5A0757AC9E7205D2", "properties":
 { "provisioningOperation": "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T21:43:05.1416423Z",
 "duration": "PT7M56.8150879S", "trackingId": "43c4f71f-557c-4abd-80c3-01f545375455", "statusCode": "Conflict",
 "statusMessage": { "status": "Failed", "error": { "code": "ResourceDeploymentFailure", "message":
 "The resource operation completed with terminal provisioning state 'Failed'.", "details": [ { "code":
 "VMExtensionProvisioningError", "message": "VM has reported a failure when processing extension 'dscextension'.
 Error message: \"DSC Configuration 'SessionHost' completed with error(s). Following are the first few:
 PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource functionality with error message:
 One or more errors occurred. The SendConfigurationApply function did not succeed.\"." } ] } }, "targetResource":
 { "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostD/providers/Microsoft.
 Compute/virtualMachines/desktop-1/extensions/dscextension",
 "resourceType": "Microsoft.Compute/virtualMachines/extensions", "resourceName": "desktop-1/dscextension" } }}
```

**Cause :** L’extension DSC PowerShell n’a pas pu obtenir un accès administrateur sur la machine virtuelle.

**Correctif :** Vérifiez que le nom d’utilisateur et le mot de passe disposent d’un accès administrateur sur la machine virtuelle, puis réexécutez le modèle Azure Resource Manager.

### <a name="error-deploymentfailed--powershell-dsc-configuration-firstsessionhost-completed-with-errors"></a>Erreur : DeploymentFailed - La configuration DSC PowerShell de « FirstSessionHost » s’est terminée avec des erreurs

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement indiquant que la configuration DSC PowerShell de « FirstSessionHost » s’est terminée avec des erreurs](../media/64870370bcbe1286906f34cf0a8646ab.png)

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

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Erreur : L’utilisateur n’est pas autorisé à interroger le service de gestion

Exemple d’erreur brute :

```Error
"response": { "content": { "startTime": "2019-04-01T17:45:33.3454563+00:00", "endTime": "2019-04-01T17:48:52.4392099+00:00",
"status": "Failed", "error": { "code": "VMExtensionProvisioningError", "message": "VM has reported a failure when processing
extension 'dscextension'. Error message: \"DSC Configuration 'FirstSessionHost' completed with error(s).
Following are the first few: PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource
 functionality with error message: User is not authorized to query the management service.
\nActivityId: 1b4f2b37-59e9-411e-9d95-4f7ccd481233\nPowershell commands to diagnose the failure:
\nGet-RdsDiagnosticActivities -ActivityId 1b4f2b37-59e9-411e-9d95-4f7ccd481233\n
The SendConfigurationApply function did not succeed.\"." }, "name": "2c3272ec-d25b-47e5-8d70-a7493e9dc473" } } }}
```

**Cause :** L’attribution de rôle de l’administrateur du locataire Windows Virtual Desktop spécifié n’est pas valide.

**Correctif :** L’utilisateur ayant créé le locataire Windows Virtual Desktop doit se connecter à Windows Virtual Desktop PowerShell et attribuer à l’utilisateur en question une attribution de rôle. Si vous exécutez les paramètres du modèle Azure Resource Manager GitHub, suivez les instructions ci-après en utilisant des commandes PowerShell :

```PowerShell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
New-RdsRoleAssignment -TenantName <Windows Virtual Desktop tenant name> -RoleDefinitionName "RDS Contributor" -SignInName <UPN>
```

### <a name="error-user-requires-azure-ad-multi-factor-authentication-mfa"></a>Erreur : L’utilisateur nécessite Azure AD Multi-Factor Authentication (MFA)

> [!div class="mx-imgBorder"]
> ![Capture d’écran du message d’échec du déploiement en raison d’une absence d’authentification multifacteur](../media/MFARequiredError.png)

Exemple d’erreur brute :

```Error
"message": "{\r\n  \"status\": \"Failed\",\r\n  \"error\": {\r\n    \"code\": \"ResourceDeploymentFailure\",\r\n    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",\r\n    \"details\": [\r\n      {\r\n        \"code\": \"VMExtensionProvisioningError\",\r\n        \"message\": \"VM has reported a failure when processing extension 'dscextension'. Error message: \\\"DSC Configuration 'FirstSessionHost' completed with error(s). Following are the first few: PowerShell DSC resource MSFT_ScriptResource  failed to execute Set-TargetResource functionality with error message: One or more errors occurred.  The SendConfigurationApply function did not succeed.\\\".\"\r\n      }\r\n    ]\r\n  }\r\n}"
```

**Cause :** L’administrateur du locataire Windows Virtual Desktop spécifié doit se connecter à l’aide d’Azure AD Multi-Factor Authentication (MFA).

**Correctif :** Créez un principal de service et attribuez-lui un rôle pour votre locataire Windows Virtual Desktop en suivant la procédure décrite dans l’article [Didacticiel : Créer des principaux de service et des attributions de rôles avec PowerShell](create-service-principal-role-powershell.md). Après avoir vérifié que vous pouvez vous connecter à Windows Virtual Desktop avec le principal du service, réexécutez l’offre de la Place de marché Azure ou le modèle Azure Resource Manager GitHub, selon la méthode que vous utilisez. Suivez les instructions ci-après pour saisir les paramètres adaptés à votre méthode.

Si vous exécutez l’offre de la Place de marché Azure, renseignez les paramètres ci-après pour vous authentifier correctement auprès de Windows Virtual Desktop :

- Propriétaire des Services Bureau à distance du locataire Windows Virtual Desktop : Principal du service
- ID d’application : identification d’application du principal de service que vous avez créé
- Mot de passe/Confirmer le mot de passe : secret de mot de passe que vous avez généré pour le principal du service
- ID de locataire Azure AD : ID de locataire Azure AD du principal de service que vous avez créé

Si vous exécutez le modèle Resource Manager GitHub, renseignez les paramètres ci-après pour vous authentifier correctement auprès de Windows Virtual Desktop :

- Nom d’utilisateur principal (UPN) de l’administrateur du locataire ou ID d’application : identification d’application du principal de service que vous avez créé
- Mot de passe d’administrateur du locataire : secret de mot de passe que vous avez généré pour le principal du service
- IsServicePrincipal : **true**
- AadTenantId : ID de locataire Azure AD du principal de service que vous avez créé

### <a name="error-vmsubnet-not-available-when-configuring-virtual-networks"></a>Erreur : vmSubnet n’est pas disponible lors de la configuration des réseaux virtuels

**Cause :** Dans le modèle Marketplace WVD, l’interface utilisateur affiche uniquement les sous-réseaux qui ont au moins autant d’adresses IP disponibles que le nombre total de machines virtuelles spécifiées dans le modèle. Le nombre réel d’adresses IP disponibles sur le sous-réseau doit être égal au nombre de nouvelles machines virtuelles déployées, mais il ne peut pas être calculé par l’interface utilisateur actuelle.

**Correctif :** Vous pouvez spécifier un sous-réseau avec au moins autant d’adresses IP disponibles que le nombre de machines virtuelles ajoutées en n’utilisant pas l’interface utilisateur de la Place de marché. Pour cela, vous pouvez spécifier le nom du sous-réseau dans le paramètre « **existingSubnetName** » quand vous [redéployez un déploiement existant](expand-existing-host-pool-2019.md#redeploy-from-azure) ou [déployez à l’aide du modèle ARM sous-jacent à partir de GitHub](create-host-pools-arm-template.md#run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool).

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir une vue d’ensemble de la résolution des problèmes Windows Virtual Desktop et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview-2019.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration-2019.md).
- Pour résoudre les problèmes de connexion au client Windows Virtual Desktop, consultez [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection-2019.md).
- Pour résoudre les problèmes liés aux clients Bureau à distance, consultez [Résoudre des problèmes du client Bureau à distance](../troubleshoot-client.md).
- Pour résoudre les problèmes d’utilisation de PowerShell avec Windows Virtual Desktop, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell-2019.md).
- Pour plus d’informations sur le service, consultez [Environnement Windows Virtual Desktop](environment-setup-2019.md).
- Suivez le [Didacticiel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Pour en savoir plus sur les actions d’audit, consultez [Opérations d’audit avec Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Pour en savoir plus sur les actions visant à déterminer les erreurs au cours du déploiement, consultez [Voir les opérations de déploiement](../../azure-resource-manager/templates/deployment-history.md).