---
title: Filtres de connexion IP d’instance IoT DPS Azure | Microsoft Docs
description: Utilisation de filtres IP pour bloquer les connexions à partir d’adresses IP spécifiques pour votre instance IoT DPS Azure. Vous pouvez bloquer les connexions à partir d’adresses IP individuelles ou de plages d’adresses IP.
author: wesmc7777
ms.author: wesmc
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: 6ff5731e7d346c954253ec4186357595461b7678
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400252"
---
# <a name="use-azure-iot-dps-ip-connection-filters"></a>Utiliser les filtres de connexion IP d’instance IoT DPS Azure

La sécurité est un aspect important de toute solution IoT. Parfois, dans le cadre de votre configuration de sécurité, vous devez spécifier explicitement les adresses IP à partir desquelles les appareils peuvent se connecter. La fonctionnalité *Filtre IP* d’un service Azure IoT Hub Device Provisioning Service (DPS) vous permet de configurer des règles de rejet ou d’acceptation du trafic provenant de certaines adresses IPv4.

## <a name="when-to-use"></a>Quand l’utiliser

Il existe deux cas d’utilisation spécifiques dans lesquels il est utile de bloquer les connexions à un point de terminaison d’instance DPS à partir de certaines adresses IP :

* Votre instance DPS doit recevoir le trafic uniquement d’une plage d’adresses IP spécifiée et refuser tout le reste. Par exemple, vous utilisez votre instance DPS avec [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services) pour créer des connexions privées entre une instance DPS et vos appareils.

* Vous devez refuser le trafic provenant d’adresses IP qui ont été identifiées comme suspectes par l’administrateur de l’instance DPS.

## <a name="how-filter-rules-are-applied"></a>Application des règles de filtre

Les règles de filtre IP sont appliquées au niveau de l’instance DPS. Par conséquent, les règles de filtre IP s’appliquent à toutes les connexions issues des appareils et des applications principales utilisant n’importe quel protocole pris en charge.

Toute tentative de connexion à partir d’une adresse IP qui correspond à une règle IP de rejet dans votre instance DPS reçoit un code d’état 401 non autorisé et une description. Le message de réponse ne mentionne pas la règle IP.

## <a name="default-setting"></a>Paramètre par défaut

Par défaut, la grille **Filtre IP** du portail pour l’instance DPS est vide. Ce paramètre par défaut signifie que votre instance DPS accepte les connexions de n’importe quelle adresse IP. Ce paramètre par défaut est équivalent à une règle qui accepte la plage d’adresses IP 0.0.0.0/0.

![Paramètres de filtre IP par défaut de l’instance IoT DPS](./media/iot-dps-ip-filtering/ip-filter-default.png)

## <a name="add-or-edit-an-ip-filter-rule"></a>Ajouter ou modifier une règle de filtre IP

Pour ajouter une règle de filtre IP, sélectionnez **+ Ajouter une règle de filtre IP**.

![Ajouter une règle de filtre IP à une instance IoT DPS](./media/iot-dps-ip-filtering/ip-filter-add-rule.png)

Après avoir sélectionné **Ajouter une règle de filtre IP**, renseignez les champs.

![Après avoir sélectionné Ajouter une règle de filtre IP](./media/iot-dps-ip-filtering/ip-filter-after-selecting-add.png)

* Donnez un **nom** à la règle de filtre IP. Ce nom doit être une chaîne alphanumérique unique qui ne prend pas en compte la casse et qui ne dépasse pas 128 caractères. Seuls les caractères alphanumériques ASCII 7 bits et `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}` sont acceptés.

* Fournissez une adresse IPv4 unique ou un bloc d’adresses IP en notation CIDR. Par exemple, dans la notation CIDR, 192.168.100.0/22 représente les 1024 adresses IPv4 allant de 192.168.100.0 à 192.168.103.255.

* Sélectionnez **Autoriser** ou **Bloquer** comme **action** pour la règle de filtre IP.

Après avoir rempli les champs, sélectionnez **Enregistrer** pour enregistrer la règle. Une alerte s’affiche vous informant que la mise à jour est en cours.

![Notification sur l’enregistrement d’une règle de filtre IP](./media/iot-dps-ip-filtering/ip-filter-save-new-rule.png)

L’option **Ajouter** est désactivée lorsque vous atteignez le nombre maximal de 100 règles de filtre IP.

Pour modifier une règle existante, sélectionnez les données que vous souhaitez modifier, effectuez la modification, puis sélectionnez **Enregistrer** pour enregistrer votre modification.

> [!NOTE]
> Le rejet d’adresses IP peut empêcher d’autres services Azure d’interagir avec l’instance DPS.

## <a name="delete-an-ip-filter-rule"></a>Suppression d’une règle de filtre IP

Pour supprimer une règle de filtre IP, sélectionnez l’icône de la corbeille sur cette ligne, puis sélectionnez **Enregistrer**. La règle est supprimée et la modification est enregistrée.

![Supprimer une règle de filtre IP de l’instance IoT DPS](./media/iot-dps-ip-filtering/ip-filter-delete-rule.png)


## <a name="update-ip-filter-rules-in-code"></a>Modifier des règles de filtre IP dans le code

Vous pouvez récupérer et modifier le filtre IP de votre instance DPS via le point de terminaison REST du fournisseur de ressources Azure. Reportez-vous à `properties.ipFilterRules` dans la [méthode createorupdate](/rest/api/iot-dps/iotdpsresource/createorupdate).

La mise à jour des règles de filtre IP de l’instance DPS n’est pas prise en charge avec Azure CLI ou Azure PowerShell pour le moment mais peut être effectuée avec des modèles Azure Resource Manager. Consultez [Modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md) pour obtenir des instructions sur l’utilisation des modèles Resource Manager. Les exemples de modèles ci-dessous montrent comment créer, modifier et supprimer des règles de filtre IP d’instance DPS.

### <a name="add-an-ip-filter-rule"></a>Ajouter une règle de filtre IP

L’exemple de modèle suivant crée une règle de filtre IP appelée « AllowAll » qui accepte tout le trafic.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#", 
    "contentVersion": "1.0.0.0", 
    "parameters": {
        "iotDpsName": {
            "type": "string",
            "defaultValue": "[resourceGroup().name]",
            "minLength": 3,
            "metadata": {
                "description": "Specifies the name of the IoT DPS service."
            }
        }, 
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for Iot DPS resource."
            }
        }        
    }, 
    "variables": {
        "iotDpsApiVersion": "2020-01-01"
    }, 
    "resources": [
        {
            "type": "Microsoft.Devices/provisioningServices",
            "apiVersion": "[variables('iotDpsApiVersion')]",
            "name": "[parameters('iotDpsName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "IpFilterRules": [
                    {
                        "FilterName": "AllowAll",
                        "Action": "Accept",
                        "ipMask": "0.0.0.0/0"
                    }
                ]
            }            
        }
    ]
}
```

Modifiez les attributs de la règle de filtre IP du modèle selon vos besoins.

| Attribut                | Description |
| ------------------------ | ----------- |
| **FilterName**           | Donnez un nom à la règle de filtre IP. Ce nom doit être une chaîne alphanumérique unique qui ne prend pas en compte la casse et qui ne dépasse pas 128 caractères. Seuls les caractères alphanumériques ASCII 7 bits et `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}` sont acceptés. |
| **Action**               | Les valeurs autorisées sont **Accepter** ou  **Refuser** pour l’action de la règle de filtre IP. |
| **ipMask**               | Fournissez une adresse IPv4 unique ou un bloc d’adresses IP en notation CIDR. Par exemple, dans la notation CIDR, 192.168.100.0/22 représente les 1024 adresses IPv4 allant de 192.168.100.0 à 192.168.103.255. |


### <a name="update-an-ip-filter-rule"></a>Modifier une règle de filtre IP

L’exemple de modèle suivant met à jour la règle de filtre IP nommée « AllowAll », mentionnée précédemment, pour refuser tout le trafic.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
                "IpFilterRules": [ 
                    { 
                        "FilterName": "AllowAll", 
                        "Action": "Reject", 
                        "ipMask": "0.0.0.0/0" 
                    } 
                ] 
            }             
        } 
    ] 
}
```

### <a name="delete-an-ip-filter-rule"></a>Suppression d’une règle de filtre IP

L’exemple de modèle suivant supprime toutes les règles de filtre IP pour l’instance DPS.

```json
{ 
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",  
    "contentVersion": "1.0.0.0",  
    "parameters": { 
        "iotDpsName": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().name]", 
            "minLength": 3, 
            "metadata": { 
                "description": "Specifies the name of the IoT DPS service." 
            } 
        },  
        "location": { 
            "type": "string", 
            "defaultValue": "[resourceGroup().location]", 
            "metadata": { 
                "description": "Location for Iot DPS resource." 
            } 
        }        
    },  
    "variables": { 
        "iotDpsApiVersion": "2020-01-01" 
    },  
    "resources": [ 
        { 
            "type": "Microsoft.Devices/provisioningServices", 
            "apiVersion": "[variables('iotDpsApiVersion')]", 
            "name": "[parameters('iotDpsName')]", 
            "location": "[parameters('location')]", 
            "sku": { 
                "name": "S1", 
                "tier": "Standard", 
                "capacity": 1 
            }, 
            "properties": { 
            }             
        } 
    ] 
}
```



## <a name="ip-filter-rule-evaluation"></a>Évaluation de règle de filtre IP

Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

Par exemple, si vous souhaitez accepter les adresses dans la plage 192.168.100.0/22 et rejeter tout le reste, la première règle de la grille doit accepter la plage d’adresses 192.168.100.0/22. La règle suivante doit rejeter toutes les adresses à l’aide de la plage 0.0.0.0/0.

Vous pouvez modifier l’ordre de vos règles de filtre IP dans la grille en cliquant sur les trois points verticaux au début d’une ligne et en effectuant un glisser-déplacer.

Pour enregistrer le nouvel ordre de vos règles de filtre IP, cliquez sur **Enregistrer**.

![Modifier l’ordre des règles de filtre IP de votre instance DPS](./media/iot-dps-ip-filtering/ip-filter-rule-order.png)

## <a name="next-steps"></a>Étapes suivantes

Pour aller plus loin dans la gestion d’une instance DPS, consultez :

* [Compréhension des adresses IP d’une instance IoT DPS](iot-dps-understand-ip-address.md)
* [Configurer une instance DPS avec l’interface de ligne de commande Azure](how-to-manage-dps-with-cli.md)
* [Contrôler l’accès à une instance DPS](how-to-control-access.md)
