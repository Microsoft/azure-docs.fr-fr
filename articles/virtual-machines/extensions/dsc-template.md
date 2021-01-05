---
title: Extension de configuration d’état souhaité avec des modèles Azure Resource Manager
description: En savoir plus sur la définition du modèle Resource Manager pour l’extension de configuration d’état souhaité (DSC) dans Azure.
services: virtual-machines-windows
author: bobbytreed
manager: carmonm
tags: azure-resource-manager
keywords: dsc
ms.assetid: b5402e5a-1768-4075-8c19-b7f7402687af
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 10/05/2018
ms.author: robreed
ms.openlocfilehash: e38fcd069fa6a3e8582dcd96b2bd0b4074986de7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955801"
---
# <a name="desired-state-configuration-extension-with-azure-resource-manager-templates"></a>Extension de configuration d’état souhaité avec des modèles Azure Resource Manager

Cet article décrit le modèle Azure Resource Manager destiné au [gestionnaire de l’extension Configuration d’état souhaité (DSC)](dsc-overview.md). La plupart des exemples utilisent **RegistrationURL** (sous forme de chaîne) et **RegistrationKey** (sous forme de [PSCredential](/dotnet/api/system.management.automation.pscredential)) pour l'intégration à Azure Automation. Pour plus d’informations sur l’obtention de ces valeurs, consultez [Intégration des machines pour la gestion avec Azure Automation State Configuration - Enregistrement sécurisé](../../automation/automation-dsc-onboarding.md#enable-machines-securely-using-registration).

> [!NOTE]
> Vous pouvez rencontrer des exemples de schéma légèrement différents. Le schéma a été modifié dans la version d’octobre 2016. Pour plus d’informations, consultez [Mettre à jour à partir d’un format antérieur](#update-from-a-previous-format).

## <a name="template-example-for-a-windows-vm"></a>Exemple de modèle pour une machine virtuelle sous Windows

L’extrait de code suivant va dans la section **Resource** du modèle.
L’extension DSC hérite des propriétés par défaut de l’extension.
Pour plus d’informations, consultez la [classe VirtualMachineExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension?view=azure-dotnet).

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "Microsoft.Powershell.DSC",
  "apiVersion": "2018-06-30",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.77",
    "autoUpgradeMinorVersion": true,
    "protectedSettings": {
      "Items": {
        "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
      }
    },
    "settings": {
      "Properties": [
        {
          "Name": "RegistrationKey",
          "Value": {
            "UserName": "PLACEHOLDER_DONOTUSE",
            "Password": "PrivateSettingsRef:registrationKeyPrivate"
          },
          "TypeName": "System.Management.Automation.PSCredential"
        },
        {
          "Name": "RegistrationUrl",
          "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
          "TypeName": "System.String"
        },
        {
          "Name": "NodeConfigurationName",
          "Value": "[parameters('nodeConfigurationName')]",
          "TypeName": "System.String"
        }
      ]
    }
  }
}
```

## <a name="template-example-for-windows-virtual-machine-scale-sets"></a>Exemple de modèle pour des groupes de machines virtuelles Windows identiques

Un groupe de machines virtuelles identiques comporte une section **properties** avec l’attribut **VirtualMachineProfile, extensionProfile**.
Sous **Extensions**, ajoutez les détails nécessaires pour l’extension DSC.

L’extension DSC hérite des propriétés par défaut de l’extension.
Pour plus d’informations, consultez la [classe VirtualMachineScaleSetExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachinescalesetextension?view=azure-dotnet).

```json
"extensionProfile": {
    "extensions": [
      {
        "name": "Microsoft.Powershell.DSC",
        "properties": {
          "publisher": "Microsoft.Powershell",
          "type": "DSC",
          "typeHandlerVersion": "2.77",
          "autoUpgradeMinorVersion": true,
          "protectedSettings": {
            "Items": {
              "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
            }
          },
          "settings": {
            "Properties": [
              {
                "Name": "RegistrationKey",
                "Value": {
                  "UserName": "PLACEHOLDER_DONOTUSE",
                  "Password": "PrivateSettingsRef:registrationKeyPrivate"
                },
                "TypeName": "System.Management.Automation.PSCredential"
              },
              {
                "Name": "RegistrationUrl",
                "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
                "TypeName": "System.String"
              },
              {
                "Name": "NodeConfigurationName",
                "Value": "[parameters('nodeConfigurationName')]",
                "TypeName": "System.String"
              }
            ]
          }
        }
      }
    ]
  }
```

## <a name="detailed-settings-information"></a>Informations détaillées sur les paramètres

Utilisez le schéma suivant dans la section **settings** de l’extension DSC Azure dans un modèle Resource Manager.

Pour obtenir la liste des arguments disponibles pour le script de configuration par défaut, consultez le [Script de configuration par défaut](#default-configuration-script).

```json
"settings": {
    "wmfVersion": "latest",
    "configuration": {
        "url": "http://validURLToConfigLocation",
        "script": "ConfigurationScript.ps1",
        "function": "ConfigurationFunction"
    },
    "configurationArguments": {
        "argument1": "Value1",
        "argument2": "Value2"
    },
    "configurationData": {
        "url": "https://foo.psd1"
    },
    "privacy": {
        "dataCollection": "enable"
    },
    "advancedOptions": {
        "downloadMappings": {
            "customWmfLocation": "http://myWMFlocation"
        }
    }
},
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
            "userName": "UsernameValue1",
            "password": "PasswordValue1"
        },
        "parameterOfTypePSCredential2": {
            "userName": "UsernameValue2",
            "password": "PasswordValue2"
        }
    },
    "configurationUrlSasToken": "?g!bber1sht0k3n",
    "configurationDataUrlSasToken": "?dataAcC355T0k3N"
}
```

## <a name="details"></a>Détails

| Nom de la propriété | Type | Description |
| --- | --- | --- |
| settings.wmfVersion |string |Spécifie la version de Windows Management Framework (WMF) qui doit être installée sur votre machine virtuelle. Lorsque cette propriété est définie sur **latest**, la version la plus récente de WMF est installée. Actuellement, les seules valeurs possibles pour cette propriété sont **4.0**, **5.0**, **5.1** et **latest**. Les valeurs possibles font l’objet de mises à jour. La valeur par défaut est **latest**. |
| settings.configuration.url |string |Spécifie l’adresse URL de téléchargement de votre fichier .zip de configuration DSC. Si l’accès à l’URL fournie nécessite un jeton SAP, définissez la propriété **protectedSettings.configurationUrlSasToken** sur la valeur de votre jeton SAP. Cette propriété est requise si la propriété **settings.configuration.script** ou **settings.configuration.function** est définie. Si aucune valeur n’est indiquée pour ces propriétés, l’extension appelle le script de configuration par défaut pour définir les métadonnées du gestionnaire de configuration locale (LCM) et les arguments doivent être fournis. |
| settings.configuration.script |string |Spécifie le nom de fichier du script qui contient la définition de votre configuration DSC. Ce script doit se trouver dans le dossier racine du fichier .zip téléchargé depuis l’URL spécifiée par la propriété **settings.configuration.url**. Cette propriété est requise si la propriété **settings.configuration.url** ou **settings.configuration.script** est définie. Si aucune valeur n’est indiquée pour ces propriétés, l’extension appelle le script de configuration par défaut pour définir les métadonnées du gestionnaire de configuration locale et les arguments doivent être fournis. |
| settings.configuration.function |string |Spécifie le nom de votre configuration DSC. La configuration nommée doit se trouver dans le script défini par **settings.configuration.script**. Cette propriété est requise si la propriété **settings.configuration.url** ou **settings.configuration.function** est définie. Si aucune valeur n’est indiquée pour ces propriétés, l’extension appelle le script de configuration par défaut pour définir les métadonnées du gestionnaire de configuration locale et les arguments doivent être fournis. |
| settings.configurationArguments |Collection |Définit les paramètres à transmettre à votre configuration DSC. Cette propriété n’est pas chiffrée. |
| settings.configurationData.url |string |Spécifie l’URL de téléchargement de votre fichier de données de configuration (.psd1) à utiliser comme entrée pour votre configuration DSC. Si l’accès à l’URL fournie nécessite un jeton SAP, définissez la propriété **protectedSettings.configurationDataUrlSasToken** sur la valeur de votre jeton SAP. |
| settings.privacy.dataCollection |string |Active ou désactive la collecte télémétrique. Les seules valeurs possibles pour cette propriété sont **Enable**, **Disable**, **''** ou **$null**. Le fait de laisser cette propriété vide ou de la définir sur $null active la télémétrie. La valeur par défaut est **''** . Pour plus d’informations, consultez la page [Azure DSC Extension Data Collection](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/) (Collection de données d’extension Azure DSC). |
| settings.advancedOptions.downloadMappings |Collection |Définit d’autres emplacements de téléchargement de WMF. Pour plus d’informations, consultez la page [Azure DSC extension 2.8 and how to map downloads of the extension dependencies to your own location](https://devblogs.microsoft.com/powershell/azure-dsc-extension-2-8-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location/) (Extension Azure DSC 2.8 et comment mapper des téléchargements des dépendances de l’extension sur votre propre emplacement). |
| protectedSettings.configurationArguments |Collection |Définit les paramètres à transmettre à votre configuration DSC. Cette propriété est chiffrée. |
| protectedSettings.configurationUrlSasToken |string |Spécifie le jeton SAP à utiliser pour accéder à l’URL définie par **settings.configuration.url**. Cette propriété est chiffrée. |
| protectedSettings.configurationDataUrlSasToken |string |Spécifie le jeton SAP à utiliser pour accéder à l’URL définie par **settings.configurationData.url**. Cette propriété est chiffrée. |

## <a name="default-configuration-script"></a>Script de configuration par défaut

Pour plus d’informations sur les valeurs suivantes, consultez la page de documentation [Paramètres de base du gestionnaire de configuration locale](/powershell/scripting/dsc/managing-nodes/metaConfig#basic-settings).
Vous pouvez utiliser le script de configuration par défaut de l’extension DSC pour configurer uniquement les propriétés du gestionnaire de configuration locale qui sont répertoriées dans le tableau suivant.

| Nom de la propriété | Type | Description |
| --- | --- | --- |
| protectedSettings.configurationArguments.RegistrationKey |PSCredential |Propriété requise. Spécifie la clé utilisée pour un nœud à inscrire avec le service Azure Automation en tant que mot de passe d’un objet d’informations d’identification PowerShell. Cette valeur peut être découverte automatiquement à l’aide de la méthode **listkeys** sur le compte Automation.  Reportez-vous à l’[exemple](#example-using-referenced-azure-automation-registration-values). |
| settings.configurationArguments.RegistrationUrl |string |Propriété requise. Spécifie l’URL du point de terminaison Automation où le nœud tente de s’inscrire. Cette valeur peut être découverte automatiquement à l’aide de la méthode **reference** sur le compte Automation. |
| settings.configurationArguments.NodeConfigurationName |string |Propriété requise. Spécifie la configuration de nœud dans le compte Automation à affecter au nœud. |
| settings.configurationArguments.ConfigurationMode |string |Spécifie le mode pour le gestionnaire de configuration locale. Les options valides sont **ApplyOnly**, **ApplyandMonitor** et **ApplyandAutoCorrect**.  La valeur par défaut est **ApplyandMonitor**. |
| settings.configurationArguments.RefreshFrequencyMins | uint32 | Spécifie la fréquence à laquelle le gestionnaire de configuration locale tente de vérifier les mises à jour du compte Automation.  La valeur par défaut est **30**.  La valeur minimale est **15**. |
| settings.configurationArguments.ConfigurationModeFrequencyMins | uint32 | Spécifie la fréquence à laquelle le gestionnaire de configuration locale valide la configuration actuelle. La valeur par défaut est **15**. La valeur minimale est **15**. |
| settings.configurationArguments.RebootNodeIfNeeded | boolean | Spécifie si un nœud peut être redémarré automatiquement si une opération DSC le demande. La valeur par défaut est **false**. |
| settings.configurationArguments.ActionAfterReboot | string | Spécifie ce qu’il se passe après un redémarrage lors de l’application d’une configuration. Les options valides sont **ContinueConfiguration** et **StopConfiguration**. La valeur par défaut est **ContinueConfiguration**. |
| settings.configurationArguments.AllowModuleOverwrite | boolean | Spécifie si le gestionnaire de configuration locale remplace les modules existants sur le nœud. La valeur par défaut est **false**. |

## <a name="settings-vs-protectedsettings"></a>settings ou protectedSettings

Tous les paramètres sont enregistrés dans un fichier texte de paramètres sur la machine virtuelle.
Les propriétés répertoriées sous **settings** sont des propriétés publiques.
Les propriétés publiques ne sont pas chiffrées dans le fichier texte des paramètres.
Les propriétés définies sous **protectedSettings** sont chiffrées avec un certificat et ne sont pas affichées en texte brut dans le fichier de paramètres sur la machine virtuelle.

Si des informations d’identification sont requises pour la configuration, vous pouvez inclure ces informations dans **protectedSettings** :

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
               "userName": "UsernameValue1",
               "password": "PasswordValue1"
        }
    }
}
```

## <a name="example-configuration-script"></a>Exemple de script de configuration

L’exemple suivant montre le comportement par défaut de l’extension DSC, qui consiste à fournir des paramètres de métadonnées au gestionnaire de configuration locale et de s’inscrire auprès du service DSC Automation.
Des arguments de configuration sont nécessaires.
Les arguments de configuration sont transmis au script de configuration par défaut pour définir les métadonnées du gestionnaire de configuration locale.

```json
"settings": {
    "configurationArguments": {
        "RegistrationUrl" : "[parameters('registrationUrl1')]",
        "NodeConfigurationName" : "nodeConfigurationNameValue1"
    }
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "registrationKey"
        }
    }
}
```

## <a name="example-using-the-configuration-script-in-azure-storage"></a>Exemple à l’aide du script de configuration dans le stockage Azure

L’exemple suivant est extrait de la [vue d’ensemble du gestionnaire d’extensions DSC](dsc-overview.md).
Cet exemple utilise des modèles Resource Manager au lieu d’applets de commande pour déployer l’extension.
Enregistrez la configuration IisInstall.ps1, placez-la dans un fichier .zip (exemple : `iisinstall.zip`), puis chargez le fichier dans une URL accessible.
Cet exemple utilise le stockage Blob Azure, mais vous pouvez télécharger les fichiers .zip depuis n’importe quel emplacement arbitraire.

Dans le modèle Resource Manager, le code suivant demande à la VM de télécharger le fichier correct puis d’exécuter la fonction PowerShell appropriée :

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/iisinstall.zip",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## <a name="example-using-referenced-azure-automation-registration-values"></a>Exemple utilisant des valeurs d'inscription Azure Automation référencées

Dans l’exemple suivant, les paramètres **RegistrationUrl** et **RegistrationKey** sont obtenus en référençant les propriétés du compte Azure Automation et en utilisant la méthode **listkeys** pour récupérer la clé primaire (0).  Dans cet exemple, les paramètres **automationAccountName** et **NodeConfigName** ont été fournis au modèle.

```json
"settings": {
    "RegistrationUrl" : "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
    "NodeConfigurationName" : "[parameters('NodeConfigName')]"
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-01-15').Keys[0].value]"
        }
    }
}
```

## <a name="update-from-a-previous-format"></a>Mettre à jour à partir d’un format antérieur

Tous les paramètres dotés d'un format précédent de l’extension (et contenant les propriétés publiques **ModulesUrl**, **ModuleSource**, **ModuleVersion**, **ConfigurationFunction**, **SasToken** ou **Properties**) s’adaptent automatiquement au format actuel de l’extension.
Ils s’exécuter exactement comme auparavant.

Le schéma suivant montre ce à quoi les paramètres précédents ressemblaient :

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties": {
        "ParameterToConfigurationFunction1": "Value1",
        "ParameterToConfigurationFunction2": "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1"
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": {
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

Voici comment le format précédent s’adapte au format actuel :

| Nom de la propriété actuelle | Équivalent dans le schéma précédent |
| --- | --- |
| settings.wmfVersion |settings.wmfVersion |
| settings.configuration.url |settings.ModulesUrl |
| settings.configuration.script |Première partie de settings.ConfigurationFunction (avant \\\\) |
| settings.configuration.function |Deuxième partie de settings.ConfigurationFunction (après \\\\) |
| settings.configuration.module.name | settings.ModuleSource |
| settings.configuration.module.version | settings.ModuleVersion |
| settings.configurationArguments |settings.Properties |
| settings.configurationData.url |protectedSettings.DataBlobUri (sans jeton SAP) |
| settings.privacy.dataCollection |settings.Privacy.dataCollection |
| settings.advancedOptions.downloadMappings |settings.advancedOptions.downloadMappings |
| protectedSettings.configurationArguments |protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken |settings.SasToken |
| protectedSettings.configurationDataUrlSasToken |Jeton SAP de protectedSettings.DataBlobUri |

## <a name="troubleshooting"></a>Dépannage

Voici certaines des erreurs que vous risquez de rencontrer et la manière dont vous pouvez les résoudre.

### <a name="invalid-values"></a>Valeurs non valides

"Privacy.dataCollection is '{0}'.
The only possible values are '', 'Enable' et 'Disable'.
"WmfVersion is '{0}'.
Les seules valeurs possibles sont les suivantes : and 'latest'".

**Problème** : Une valeur fournie n’est pas autorisée.

**Solution**: Remplacez la valeur non valide par une valeur valide.
Pour plus d’informations, consultez le tableau [Détails](#details).

### <a name="invalid-url"></a>URL non valide

"ConfigurationData.url is '{0}'. This is not a valid URL » (configurationData.url est « {0} ». Il ne s’agit pas d’une URL valide.) « DataBlobUri is '{0}'. This is not a valid URL » (DataBlobUri est « {0} ». Il ne s’agit pas d’une URL valide.) « Configuration.url is '{0}'. This is not a valid URL » (configuration.url est « {0} ». Il ne s’agit pas d’une URL valide.)

**Problème** : Une URL fournie n’est pas valide.

**Solution**: Vérifiez toutes les URL que vous avez fournies.
Assurez-vous que toutes les URL se résolvent en emplacements valides auxquels l’extension peut accéder sur l’ordinateur distant.

### <a name="invalid-registrationkey-type"></a>Type de RegistrationKey non valide

« Type non valide pour le paramètre RegistrationKey de type PSCredential ».

**Problème** : la valeur *RegistrationKey* de la propriété protectedSettings.configurationArguments ne peut être fournie que sous le type PSCredential.

**Solution**: remplacez l'entrée protectedSettings.configurationArguments de la valeur RegistrationKey par un type PSCredential en utilisant le format suivant :

```json
"configurationArguments": {
    "RegistrationKey": {
        "userName": "NOT_USED",
        "Password": "RegistrationKey"
    }
}
```

### <a name="invalid-configurationargument-type"></a>Type configurationArguments non valide

"Invalid configurationArguments type {0}"

**Problème** : la propriété *ConfigurationArguments* ne peut pas se résoudre en objet de **table de hachage**.

**Solution**: faites de votre propriété *ConfigurationArguments* une **table de hachage**.
Suivez le format fourni dans les exemples précédents. Prenez garde aux guillemets, aux virgules et aux accolades.

### <a name="duplicate-configurationarguments"></a>Propriétés configurationArguments en double

« Found duplicate arguments '{0}' in both public and protected configurationArguments » (Arguments en double trouvés dans les paramètres configuration Arguments publics et protégés)

**Problème** : Les arguments *ConfigurationArguments* dans les paramètres publics et les arguments *ConfigurationArguments* dans les paramètres protégés contiennent des propriétés portant le même nom.

**Solution**: Supprimez l’une des propriétés en double.

### <a name="missing-properties"></a>Propriétés manquantes

« settings.Configuration.function requires that settings.configuration.url or settings.configuration.module is specified » (settings.configuration.url ou settings.configuration.module doit être spécifié pour settings.Configuration.function)

« settings.Configuration.url requires that settings.configuration.script is specified » (settings.configuration.script doit être spécifié pour settings.Configuration.url)

« settings.Configuration.script requires that settings.configuration.url is specified » (settings.configuration.url doit être spécifié pour settings.Configuration.script)

« settings.Configuration.url requires that settings.configuration.function is specified » (settings.configuration.function doit être spécifié pour settings.Configuration.url)

« protectedSettings.ConfigurationUrlSasToken requires that settings.configuration.url is specified » (settings.configuration.url doit être spécifié pour protectedSettings.ConfigurationUrlSasToken)

« protectedSettings.ConfigurationDataUrlSasToken requires that settings.configurationData.url is specified » (settings.configurationData.url doit être spécifié pour protectedSettings.ConfigurationDataUrlSasToken)

**Problème** : Une propriété définie a besoin d’une autre propriété, laquelle est manquante.

**Solutions** :

- Fournissez la propriété manquante.
- Supprimez la propriété qui a besoin de la propriété manquante.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [l’utilisation des groupes identiques de machines virtuelles avec l’extension DSC Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-dsc.md).
- En savoir plus sur la [Gestion des informations d’identification sécurisées de DSC](dsc-credentials.md).
- Obtenir une [introduction au gestionnaire d’extensions Azure DSC](dsc-overview.md).
- Pour plus informations sur DSC PowerShell, reportez-vous au [centre de documentation PowerShell](/powershell/scripting/dsc/overview/overview).
