---
title: Arrêter d’utiliser l’extension de machine virtuelle d’une identité managée - Azure AD
description: Instructions pas à pas pour cesser d’utiliser l’extension de machine virtuelle et commencer à utiliser Azure Instance Metadata Service (IMDS) pour l’authentification.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/25/2018
ms.author: barclayn
ms.openlocfilehash: 84a262cae17a4e26724ab06da397e699e09468db
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90969198"
---
# <a name="how-to-stop-using-the-virtual-machine-managed-identities-extension-and-start-using-the-azure-instance-metadata-service"></a>Comment cesser d’utiliser l’extension de machine virtuelle pour les identités managées et commencer à utiliser Azure Instance Metadata Service

## <a name="virtual-machine-extension-for-managed-identities"></a>Extension de machine virtuelle pour les identités managées

L’extension de machine virtuelle pour les entités managées est utilisée pour demander des jetons pour une identité managée au sein de la machine virtuelle. Le flux de travail se déroule comme suit :

1. Tout d’abord, la charge de travail dans la ressource appelle le point de terminaison local `http://localhost/oauth2/token` pour demander un jeton d’accès.
2. L’extension de machine virtuelle utilise ensuite les informations d’identification pour l’identité managée, pour demander un jeton d’accès auprès d’Azure AD. 
3. Le jeton d’accès est retourné à l’appelant et peut être utilisé pour l’authentification aux services qui prennent en charge l’authentification Azure AD, comme Azure Key Vault ou Stockage Azure.

De par certaines limitations décrites dans la section suivante, l’extension de machine virtuelle pour les identités managées est désormais déconseillée. Il est plutôt recommandé d’utiliser le point de terminaison équivalent dans Azure Instance Metadata Service (IMDS).

### <a name="provision-the-extension"></a>Approvisionner l’extension 

Lorsque vous configurez une machine virtuelle ou un groupe de machines virtuelles identiques pour disposer d’une identité managée, vous pouvez éventuellement choisir d’approvisionner les identités managées pour l’extension de machine virtuelle pour ressources Azure en utilisant le paramètre `-Type` dans la cmdlet [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension). Vous pouvez passer `ManagedIdentityExtensionForWindows` ou `ManagedIdentityExtensionForLinux` selon le type de machine virtuelle et lui affecter un nom en utilisant le paramètre `-Name`. Le paramètre `-Settings` spécifie le port utilisé par le point de terminaison de jeton OAuth pour l’acquisition de jeton :

```azurepowershell-interactive
$settings = @{ "port" = 50342 }
   Set-AzVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
```

Vous pouvez également utiliser le modèle de déploiement Azure Resource Manager pour approvisionner l’extension de machine virtuelle, en ajoutant le code JSON suivant à la section `resources` du modèle (utilisez `ManagedIdentityExtensionForLinux` pour les éléments de nom et de type pour la version Linux).

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
    "apiVersion": "2018-06-01",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.ManagedIdentity",
        "type": "ManagedIdentityExtensionForWindows",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "port": 50342
        }
    }
}
```
    
    
Si vous utilisez des groupes de machines virtuelles identiques, vous pouvez également approvisionner les identités managées pour l’extension de groupe de machines virtuelles identiques pour ressources Azure en utilisant la cmdlet [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension). Vous pouvez passer `ManagedIdentityExtensionForWindows` ou `ManagedIdentityExtensionForLinux` selon le type de groupe de machines virtuelles identiques et lui affecter un nom en utilisant le paramètre `-Name`. Le paramètre `-Settings` spécifie le port utilisé par le point de terminaison de jeton OAuth pour l’acquisition de jeton :

   ```azurepowershell-interactive
   $setting = @{ "port" = 50342 }
   $vmss = Get-AzVmss
   Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Setting $settings 
   ```
Pour approvisionner l’extension de groupe de machines virtuelles identiques avec le modèle de déploiement Azure Resource Manager, ajoutez le code JSON suivant à la section `extensionpProfile` du modèle (utilisez `ManagedIdentityExtensionForLinux` pour les éléments de nom et de type pour la version Linux).

```json
"extensionProfile": {
    "extensions": [
        {
            "name": "ManagedIdentityWindowsExtension",
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
                },
                "protectedSettings": {}
            }
        }
```

L’approvisionnement de l’extension de machine virtuelle peut échouer en raison d’échecs de recherche DNS. Si cela se produit, redémarrez la machine virtuelle, puis réessayez. 

### <a name="remove-the-extension"></a>Supprimer l’extension 
Pour supprimer l’extension, utilisez le commutateur `-n ManagedIdentityExtensionForWindows` ou `-n ManagedIdentityExtensionForLinux` (selon le type de machine virtuelle) avec [az vm extension delete](/cli/azure/vm/) ou [az vmss extension delete](/cli/azure/vmss) pour les groupes de machines virtuelles identiques à l’aide d’Azure CLI ou `Remove-AzVMExtension` pour PowerShell :

```azurecli-interactive
az vm identity --resource-group myResourceGroup --vm-name myVm -n ManagedIdentityExtensionForWindows
```

```azurecli-interactive
az vmss extension delete -n ManagedIdentityExtensionForWindows -g myResourceGroup -vmss-name myVMSS
```

```azurepowershell-interactive
Remove-AzVMExtension -ResourceGroupName myResourceGroup -Name "ManagedIdentityExtensionForWindows" -VMName myVM
```

### <a name="acquire-a-token-using-the-virtual-machine-extension"></a>Acquérir un jeton à l’aide de l’extension de machine virtuelle

Voici un exemple de requête utilisant les identités managées pour le point de terminaison d’extension de machine virtuelle pour ressources Azure :

```
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Élément | Description |
| ------- | ----------- |
| `GET` | Le verbe HTTP, indiquant votre souhait de récupérer des données du point de terminaison. Dans ce cas, un jeton d’accès OAuth. | 
| `http://localhost:50342/oauth2/token` | Le point de terminaison d’identités managées pour ressources Azure, où 50342 est le port par défaut configurable. |
| `resource` | Un paramètre de chaîne de requête, indiquant l’URI ID d’application de la ressource cible. Il apparaît également dans la revendication `aud` (audience) du jeton émis. Cet exemple demande un jeton pour accéder à Azure Resource Manager, qui possède un URI ID d’application, `https://management.azure.com/`. |
| `Metadata` | Un champ d’en-tête de requête HTTP, requis par les identités managées pour ressources Azure afin d’atténuer une attaque par falsification de requête côté serveur (SSRF). Cette valeur doit être définie sur « true », en minuscules.|
| `object_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément object_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|
| `client_id` | (Facultatif) Un paramètre de chaîne de requête, indiquant l’élément client_id de l’identité managée pour laquelle vous souhaitez obtenir le jeton. Obligatoire, si votre machine virtuelle possède plusieurs identités managées affectées par l’utilisateur.|


Exemple de réponse :

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Élément | Description |
| ------- | ----------- |
| `access_token` | Le jeton d’accès demandé. Lors de l’appel d’une API REST sécurisée, le jeton est incorporé dans le champ d’en-tête de requête `Authorization` comme un jeton « du porteur », autorisant l’API à authentifier l’appelant. | 
| `refresh_token` | Non utilisé par les identités managées pour ressources Azure. |
| `expires_in` | Le nombre de secondes pendant lesquelles le jeton d’accès est toujours valide, avant d’expirer, à partir de son émission. L’heure d’émission est accessible dans la revendication `iat` du jeton. |
| `expires_on` | L’intervalle de temps lorsque le jeton d’accès expire. La date est exprimée en nombre de secondes à partir de « 1970-01-01T0:0:0Z UTC » (correspond à la revendication `exp` du jeton). |
| `not_before` | L’intervalle de temps pendant lequel le jeton d’accès prend effet, et peut être accepté. La date est exprimée en nombre de secondes à partir de « 1970-01-01T0:0:0Z UTC » (correspond à la revendication `nbf` du jeton). |
| `resource` | La ressource pour laquelle le jeton d’accès a été demandé, correspondant au paramètre de chaîne de requête `resource` de la requête. |
| `token_type` | Le type de jeton, qui est un jeton d’accès « du porteur », ce qui signifie que la ressource peut donner l’accès au porteur de ce jeton. |


### <a name="troubleshoot-the-virtual-machine-extension"></a>Détecter un problème lié à l’extension de machine virtuelle 

#### <a name="restart-the-virtual-machine-extension-after-a-failure"></a>Redémarrez l’extension de machine virtuelle après un échec

Sur Windows et certaines versions de Linux, si l’extension s’arrête, la cmdlet suivante peut être utilisée pour redémarrer manuellement l’extension :

```azurepowershell-interactive
Set-AzVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Où : 
- Type et nom de l’extension pour Windows : `ManagedIdentityExtensionForWindows`
- Type et nom de l’extension pour Linux : `ManagedIdentityExtensionForLinux`

#### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Le « Script d’automatisation » échoue lors de la tentative d’exportation de schéma pour l’extension des entités managées pour ressources Azure

Lorsque des identités managées pour ressources Azure sont activées sur une machine virtuelle, le message d'erreur suivant s'affiche en cas de tentative d'utilisation de la fonctionnalité « Script d'automatisation » pour la machine virtuelle ou son groupe de ressources :

![Erreur d’exportation de script d’automatisation d’identités managées pour ressources Azure](./media/howto-migrate-vm-extension/automation-script-export-error.png)

Actuellement, les identités managées pour l’extension de machine virtuelle pour ressources Azure ne prennent pas en charge la possibilité d’exporter leur schéma vers un modèle de groupe de ressources. Par conséquent, le modèle généré n’affiche pas les paramètres de configuration permettant d’activer des identités managées pour ressources Azure sur la ressource. Ces sections peuvent être ajoutées manuellement en suivant les exemples fournis dans [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant un modèle](qs-configure-template-windows-vm.md).

Lorsque la fonctionnalité d’exportation de schéma sera disponible pour les identités managées pour l’extension de machine virtuelle pour ressources Azure (dont l’abandon est prévu en janvier 2019), elle sera répertoriée dans [Exportation de groupes de ressources contenant des extensions de machine virtuelle](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

## <a name="limitations-of-the-virtual-machine-extension"></a>Limitations de l’extension de machine virtuelle 

L’utilisation de l’extension de machine virtuelle est soumise à plusieurs limitations majeures. 

 * La limitation la plus importante est le fait que les informations d’identification utilisées pour demander des jetons sont stockées sur la machine virtuelle. Un attaquant qui parvient à accéder à la machine virtuelle peut exfiltrer les informations d’identification. 
 * En outre, l’extension de machine virtuelle n’est pas encore prise en charge par plusieurs distributions Linux, et le coût de développement lié à la modification, à la génération et au test de l’extension sur chacune de ces distributions est colossal. Actuellement, seules les distributions Linux suivantes sont prises en charge : 
    * CoreOS Stable
    * CentOS 7.1 
    * Red Hat 7.2 
    * Ubuntu 15.04 
    * Ubuntu 16.04
 * Le déploiement de machines virtuelles avec des identités managées a un impact sur les performances dans la mesure où l’extension de machine virtuelle doit également être approvisionnée. 
 * Enfin, l’extension de machine virtuelle peut prendre en charge uniquement 32 identités managées affectées par l’utilisateur par machine virtuelle. 

## <a name="azure-instance-metadata-service"></a>Service de métadonnées d’instance Azure

[Azure Instance Metadata Service (IMDS)](../../virtual-machines/windows/instance-metadata-service.md) est un point de terminaison REST qui fournit des informations sur les instances de machine virtuelle en cours d’exécution qui peuvent être utilisées pour gérer et configurer vos machines virtuelles. Le point de terminaison est disponible à une adresse IP non routable bien connue (`169.254.169.254`) accessible uniquement à partir de la machine virtuelle.

L’utilisation d’Azure IMDS pour demander des jetons offre plusieurs avantages. 

1. Le service est externe à la machine virtuelle. Par conséquent, les informations d’identification utilisées par les identités managées ne sont plus présentes sur la machine virtuelle. Au lieu de cela, elles sont hébergées et sécurisées sur l’ordinateur hôte de la machine virtuelle Azure.   
2. Tous les systèmes d’exploitation Windows et Linux pris en charge sur Azure IaaS peuvent utiliser des identités managées.
3. Le déploiement est plus rapide et plus facile, étant donné qu’il n’est plus nécessaire d’approvisionner l’extension de machine virtuelle.
4. Avec le point de terminaison IMDS, il est possible d’attribuer jusqu’à 1 000 identités managées affectées par l’utilisateur à une seule machine virtuelle.
5. Il n’existe pas de différence significative entre les requêtes utilisant IMDS et celles utilisant l’extension de machine virtuelle. Par conséquent il est relativement simple de procéder au portage des déploiements existants qui utilisent l’extension de machine virtuelle.

Pour ces raisons, le service Azure IMDS représentera sera la méthode de facto pour demander des jetons lorsque l’extension de machine virtuelle sera déconseillée. 


## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique de l’utilisation d’identités managées pour ressources Azure sur une machine virtuelle Azure afin d’acquérir un jeton d’accès](how-to-use-vm-token.md)
* [Azure Instance Metadata Service](../../virtual-machines/windows/instance-metadata-service.md)
