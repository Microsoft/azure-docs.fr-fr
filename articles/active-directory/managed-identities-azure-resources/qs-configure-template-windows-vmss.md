---
title: Configurer le modèle pour utiliser des identités managées sur les groupes de machines virtuelles identiques - Azure AD
description: Instructions détaillées pour configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant un modèle Azure Resource Manager.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 377bbb9ce111f3cf2daf8426e128186711c30e5f
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587449"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-virtual-machine-scale-using-a-template"></a>Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques Azure en utilisant un modèle

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent automatiquement aux services Azure une identité managée dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code.

Dans cet article, vous allez découvrir comment effectuer les opérations d’identités managées pour ressources Azure suivantes sur un groupe de machines virtuelles identiques Azure, à l’aide du modèle de déploiement Azure Resource Manager :
- Activer et désactiver l’identité managée affectée par le système sur un groupe de machines virtuelles identiques Azure
- Ajouter et supprimer une identité managée affectée par l’utilisateur sur un groupe de machines virtuelles identiques Azure

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour effectuer les opérations de gestion dans cet article, votre compte doit disposer de ces affectations de contrôle d’accès basé sur les rôles Azure :

    > [!NOTE]
    > Aucune attribution de rôle d'annuaire Azure AD supplémentaire n’est requise.

    - [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) pour créer un groupe de machines virtuelles identiques, puis activer ou supprimer l’identité managée affectée par le système et/ou par l’utilisateur à partir d’un groupe de machines virtuelles identiques.
    - [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor) pour créer une identité managée affectée par l’utilisateur.
    - [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator) pour attribuer et supprimer une identité managée affectée par l’utilisateur dans un groupe de machines virtuelles identiques.

## <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

Comme pour le portail Azure et le script, les modèles [Azure Resource Manager](../../azure-resource-manager/management/overview.md) offrent la possibilité de déployer des ressources nouvelles ou modifiées définies par un groupe de ressources Azure. Plusieurs options sont disponibles pour la modification du modèle et le déploiement, à la fois localement et sur le portail, y compris :

   - Utiliser un [modèle personnalisé à partir de Place de marché Azure](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), lequel vous permet de créer un modèle à partir de zéro, ou à partir d’un modèle commun existant ou d’un [modèle de démarrage rapide](https://azure.microsoft.com/documentation/templates/).
   - Dériver à partir d’un groupe de ressources existant, en exportant un modèle à partir du [déploiement d’origine](../../azure-resource-manager/templates/export-template-portal.md), ou à partir de l’[état actuel du déploiement](../../azure-resource-manager/templates/export-template-portal.md).
   - Utilisation d’un [éditeur local JSON (VS Code, par exemple)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md), puis téléchargement/déploiement à l’aide de PowerShell ou Azure CLI.
   - Utilisez le [projet de groupe de ressources Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) de Visual Studio pour créer et déployer un modèle.  

Quelle que soit l’option choisie, la syntaxe de modèle est identique lors du déploiement initial et lors du redéploiement. L’activation d’identités managées pour ressources Azure s’effectue de la même manière sur une machine virtuelle nouvelle ou existante. De plus, par défaut, Azure Resource Manager effectue une [mise à jour incrémentielle](../../azure-resource-manager/templates/deployment-modes.md) au niveau des déploiements.

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, vous allez activer et désactiver l’identité managée affectée par le système en utilisant un modèle Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-the-creation-of-a-virtual-machines-scale-set-or-an-existing-virtual-machine-scale-set"></a>Activer une identité managée affectée par le système pendant la création d’un groupe de machines virtuelles identiques ou dans un groupe de machines virtuelles identiques existant

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques.
2. Pour activer l’identité managée affectée par le système, chargez le modèle dans un éditeur, recherchez la ressource `Microsoft.Compute/virtualMachinesScaleSets` qui vous intéresse dans la section des ressources, puis ajoutez la propriété `identity` au même niveau que la propriété `"type": "Microsoft.Compute/virtualMachinesScaleSets"`. Utilisez la syntaxe suivante :

   ```JSON
   "identity": {
       "type": "SystemAssigned"
   }
   ```

> [!NOTE]
> Vous pouvez éventuellement approvisionner des identités managées pour l’extension de groupe de machines virtuelles identiques de ressources Azure en le spécifiant dans l’élément `extensionProfile` du modèle. Cette étape est facultative, car vous pouvez également utiliser le point de terminaison d’identité IMDS (Instance Metadata Service) Azure pour récupérer des jetons.  Pour plus d’informations, consultez [migrer à partir de l’extension de machine virtuelle vers le point de terminaison IMDS d’Azure pour l’authentification](howto-migrate-vm-extension.md).


4. Lorsque vous avez terminé, les sections suivantes doivent être ajoutées à la section de ressources de votre modèle et doivent ressembler à ceci :

   ```json
    "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "SystemAssigned",
            },
           "properties": {
                //other resource provider properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)
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
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```

### <a name="disable-a-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Désactiver une identité managée affectée par le système d’un groupe de machines virtuelles identiques Azure

Si vous disposez d’un groupe de machines virtuelles identiques qui n’a plus besoin d’une identité managée affectée par le système :

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques.

2. Chargez le modèle dans un [éditeur](#azure-resource-manager-templates) et localisez la ressource `Microsoft.Compute/virtualMachineScaleSets` qui vous intéresse dans la section `resources`. Si votre machine virtuelle dispose uniquement d’une identité managée affectée par le système, vous pouvez la désactiver en remplaçant le type d’identité par `None`.

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/06/2018**

   Si votre version d’API est `2018-06-01` et que votre machine virtuelle a des identités managées affectées par le système et l’utilisateur, supprimez `SystemAssigned` dans le type d’identité et conservez `UserAssigned` avec les valeurs du dictionnaire userAssignedIdentities.

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/06/2018**

   Si votre version d’API est `2017-12-01` et que votre groupe de machines virtuelles identiques dispose d’identités managées affectées par le système et par l’utilisateur, supprimez `SystemAssigned` dans le type d’identité et conservez `UserAssigned` avec le tableau `identityIds` des identités managées affectées par l’utilisateur.



   L’exemple suivant montre comment supprimer une identité managée affectée par le système d’un groupe de machines virtuelles identiques sans identité managée affectée par l’utilisateur :

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "None"
        }

   }
   ```

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, vous allez affecter une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques à l’aide du modèle Azure Resource Manager.

> [!Note]
> Pour créer une identité managée affectée par l’utilisateur en utilisant un modèle Azure Resource Manager, consultez [Créer une identité managée affectée par l’utilisateur](how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity).

### <a name="assign-a-user-assigned-managed-identity-to-a-virtual-machine-scale-set"></a>Attribuer une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques

1. Sous l’élément `resources`, ajoutez l’entrée suivante pour attribuer une identité managée affectée par l’utilisateur à votre groupe de machines virtuelles identiques.  Veillez à remplacer `<USERASSIGNEDIDENTITY>` par le nom de l’identité managée affectée par l’utilisateur que vous avez créée.

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/06/2018**

   Si votre version d’API est `2018-06-01`, vos identités managées affectées par l’utilisateur sont stockées dans le format de dictionnaire `userAssignedIdentities`, et la valeur `<USERASSIGNEDIDENTITYNAME>` doit être stockée dans une variable définie au sein de la section `variables` de votre modèle.

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "userAssigned",
           "userAssignedIdentities": {
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
           }
       }

   }
   ```   

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/12/2017**

   Si votre `apiVersion` est `2017-12-01` ou une version antérieure, vos identités managées affectées par l’utilisateur sont stockées dans le tableau `identityIds` et la valeur `<USERASSIGNEDIDENTITYNAME>` doit être stockée dans une variable définie au sein de la section des variables de votre modèle.

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2017-03-30",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "userAssigned",
           "identityIds": [
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITY>'))]"
           ]
       }

   }
   ```
> [!NOTE]
> Vous pouvez éventuellement approvisionner des identités managées pour l’extension de groupe de machines virtuelles identiques de ressources Azure en le spécifiant dans l’élément `extensionProfile` du modèle. Cette étape est facultative, car vous pouvez également utiliser le point de terminaison d’identité IMDS (Instance Metadata Service) Azure pour récupérer des jetons.  Pour plus d’informations, consultez [migrer à partir de l’extension de machine virtuelle vers le point de terminaison IMDS d’Azure pour l’authentification](howto-migrate-vm-extension.md).

3. Quand vous avez terminé, votre modèle doit ressembler au suivant :

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/06/2018**   

   ```json
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "UserAssigned",
                "userAssignedIdentities": {
                    "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
                }
            },
           "properties": {
                //other virtual machine properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)
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
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```

   **API Microsoft.Compute/virtualMachines version 2017-12-01**

   ```json
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2017-12-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "UserAssigned",
                "identityIds": [
                    "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
                ]
            },
           "properties": {
                //other virtual machine properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)    
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
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```
   ### <a name="remove-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Supprimer une identité managée affectée par l’utilisateur d’un groupe de machines virtuelles identiques Azure

Si vous disposez d’un groupe de machines virtuelles identiques qui n’a plus besoin d’une identité managée affectée par l’utilisateur :

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques.

2. Chargez le modèle dans un [éditeur](#azure-resource-manager-templates) et localisez la ressource `Microsoft.Compute/virtualMachineScaleSets` qui vous intéresse dans la section `resources`. Si votre groupe de machines virtuelles identiques dispose uniquement de l’identité managée assignée par l’utilisateur, vous pouvez la désactiver en remplaçant le type d’identité par `None`.

   L’exemple suivant montre comment supprimer toutes les identités managées affectées par l’utilisateur d’une machine virtuelle sans identité managée affectée par le système :

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "None"
        }
   }
   ```

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/06/2018**

   Pour supprimer une seule identité managée assignée par l’utilisateur d’un groupe de machines virtuelles identiques, supprimez-la du dictionnaire `userAssignedIdentities`.

   Si vous disposez d’une identité affectée par le système, conservez-la dans la valeur `type`, sous la valeur `identity`.

   **API Microsoft.Compute/virtualMachineScaleSets version du 01/12/2017**

   Pour supprimer une seule identité managée affectée par l’utilisateur dans un groupe de machines virtuelles identiques, supprimez-la du tableau `identityIds`.

   Si vous disposez d’une identité managée affectée par le système, conservez-la dans la valeur `type`, sous la valeur `identity`.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des identités managées pour ressources Azure](overview.md).