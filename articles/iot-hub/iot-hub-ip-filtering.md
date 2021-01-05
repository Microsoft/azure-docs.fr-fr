---
title: Filtres de connexion IP d’Azure Hub IoT | Microsoft Docs
description: Utilisation de filtres IP pour bloquer les connexions à partir d’adresses IP spécifiques pour votre Azure IoT Hub. Vous pouvez bloquer les connexions à partir d’adresses IP individuelles ou de plages d’adresses IP.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/25/2020
ms.author: robinsh
ms.openlocfilehash: 501b609d745e0a86bc1e00bccae54bb4f6e49376
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545274"
---
# <a name="use-ip-filters"></a>Utiliser des filtres IP

La sécurité est un aspect important de toute solution IoT basée sur Azure IoT Hub. Parfois, dans le cadre de votre configuration de sécurité, vous devez spécifier explicitement les adresses IP à partir desquelles les appareils peuvent se connecter. La fonctionnalité *Filtre IP* vous permet de configurer des règles de rejet ou d’acceptation du trafic provenant de certaines adresses IPv4.

## <a name="when-to-use"></a>Quand l’utiliser

Il existe deux cas d’utilisation spécifiques illustrant lorsqu’il est utile de bloquer les points de terminaison IoT Hub pour certaines adresses IP :

* Votre IoT Hub ne doit recevoir du trafic qu’à partir d’une plage spécifiée d’adresses IP et rejeter tout le reste. Par exemple, vous utilisez votre IoT Hub avec [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services) pour créer des connexions privées entre un IoT Hub et votre infrastructure locale.

* Vous devez refuser le trafic provenant d’adresses IP qui ont été identifiées comme suspectes par l’administrateur de IoT Hub.

## <a name="how-filter-rules-are-applied"></a>Application des règles de filtre

Les règles de filtre IP sont appliquées au niveau du service IoT Hub. Par conséquent, les règles de filtre IP s’appliquent à toutes les connexions issues des appareils et des applications principales utilisant n’importe quel protocole pris en charge. Toutefois, les clients lisant directement à partir du [point de terminaison compatible avec Event Hub intégré](iot-hub-devguide-messages-read-builtin.md) (pas via la chaîne de connexion IoT Hub) ne sont pas liés aux règles de filtre IP. 

Toute tentative de connexion à partir d’une adresse IP qui correspond à une règle IP de rejet dans votre IoT Hub reçoit un code d’état 401 non autorisé et une description. Le message de réponse ne mentionne pas la règle IP. Le rejet d’adresses IP est de nature à empêcher d’autres services Azure (comme Azure Stream Analytics, Azure Virtual Machines ou l’Explorateur d’appareils dans le portail Azure) d’interagir avec IoT Hub.

> [!NOTE]
> Si vous devez utiliser Azure Stream Analytics (ASA) pour lire les messages à partir d’un hub IoT avec le filtre IP activé, utilisez le nom compatible avec Event Hub et le point de terminaison de votre hub IoT pour ajouter manuellement une [entrée de flux Event Hubs](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-event-hubs) dans l’ASA.

## <a name="default-setting"></a>Paramètre par défaut

Par défaut, la grille **Filtre IP** dans le portail pour un IoT Hub est vide. Ce paramètre par défaut signifie que votre hub accepte les connexions de n’importe quelle adresse IP. Ce paramètre par défaut est équivalent à une règle qui accepte la plage d’adresses IP 0.0.0.0/0.

Pour accéder à la page Paramètres de filtre IP, sélectionnez **Réseau** , **Accès public** , puis choisissez **Plages d’adresses IP sélectionnées**  :

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-default.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

## <a name="add-or-edit-an-ip-filter-rule"></a>Ajouter ou modifier une règle de filtre IP

Pour ajouter une règle de filtre IP, sélectionnez **+ Ajouter une règle de filtre IP** .

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-add-rule.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

Après avoir sélectionné **Ajouter une règle de filtre IP** , renseignez les champs.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-after-selecting-add.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

* Donnez un **nom** à la règle de filtre IP. Ce nom doit être une chaîne alphanumérique unique qui ne prend pas en compte la casse et qui ne dépasse pas 128 caractères. Seuls les caractères alphanumériques ASCII 7 bits et `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}` sont acceptés.

* Fournissez une adresse IPv4 unique ou un bloc d’adresses IP en notation CIDR. Par exemple, dans la notation CIDR, 192.168.100.0/22 représente les 1024 adresses IPv4 allant de 192.168.100.0 à 192.168.103.255.

* Sélectionnez **Autoriser** ou **Bloquer** comme **action** pour la règle de filtre IP.

Après avoir rempli les champs, sélectionnez **Enregistrer** pour enregistrer la règle. Une alerte s’affiche vous informant que la mise à jour est en cours.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-save-new-rule.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

L’option **Ajouter** est désactivée lorsque vous atteignez le nombre maximal de dix règles de filtre IP.

Pour modifier une règle existante, sélectionnez les données que vous souhaitez modifier, effectuez la modification, puis sélectionnez **Enregistrer** pour enregistrer votre modification.

## <a name="delete-an-ip-filter-rule"></a>Suppression d’une règle de filtre IP

Pour supprimer une règle de filtre IP, sélectionnez l’icône de la corbeille sur cette ligne, puis sélectionnez **Enregistrer** . La règle est supprimée et la modification est enregistrée.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-delete-rule.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

## <a name="retrieve-and-update-ip-filters-using-azure-cli"></a>Récupérer et mettre à jour des filtres IP à l'aide d'Azure CLI

Les filtres IP de votre hub IoT peuvent être récupérés et mis à jour via [Azure CLI](/cli/azure/).

Pour récupérer les filtres IP actuels de votre hub IoT, exécutez :

```azurecli-interactive
az resource show -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs
```

Cette opération renvoie un objet JSON dans lequel vos filtres IP existants sont répertoriés sous la clé `properties.ipFilterRules` :

```json
{
...
    "properties": {
        "ipFilterRules": [
        {
            "action": "Reject",
            "filterName": "MaliciousIP",
            "ipMask": "6.6.6.6/6"
        },
        {
            "action": "Allow",
            "filterName": "GoodIP",
            "ipMask": "131.107.160.200"
        },
        ...
        ],
    },
...
}
```

Pour ajouter un nouveau filtre IP à votre hub IoT, exécutez :

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules "{\"action\":\"Reject\",\"filterName\":\"MaliciousIP\",\"ipMask\":\"6.6.6.6/6\"}"
```

Pour supprimer un filtre IP existant de votre hub IoT, exécutez :

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules <ipFilterIndexToRemove>
```

Notez que `<ipFilterIndexToRemove>` doit correspondre à l'ordre des filtres IP dans les règles `properties.ipFilterRules` de votre hub IoT.

## <a name="retrieve-and-update-ip-filters-using-azure-powershell"></a>Récupérer et mettre à jour des filtres IP à l'aide d'Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les filtres IP de votre hub IoT peuvent être récupérés et définis via [Azure PowerShell](/powershell/azure/).

```powershell
# Get your IoT Hub resource using its name and its resource group name
$iothubResource = Get-AzResource -ResourceGroupName <resourceGroupNmae> -ResourceName <iotHubName> -ExpandProperties

# Access existing IP filter rules
$iothubResource.Properties.ipFilterRules |% { Write-host $_ }

# Construct a new IP filter
$filter = @{'filterName'='MaliciousIP'; 'action'='Reject'; 'ipMask'='6.6.6.6/6'}

# Add your new IP filter rule
$iothubResource.Properties.ipFilterRules += $filter

# Remove an existing IP filter rule using its name, e.g., 'GoodIP'
$iothubResource.Properties.ipFilterRules = @($iothubResource.Properties.ipFilterRules | Where 'filterName' -ne 'GoodIP')

# Update your IoT Hub resource with your updated IP filters
$iothubResource | Set-AzResource -Force
```

## <a name="update-ip-filter-rules-using-rest"></a>Mettre à jour les règles de filtrage IP à l'aide de REST

Vous pouvez également récupérer et modifier les filtres IP de votre hub IoT via le point de terminaison REST du fournisseur de ressources Azure. Reportez-vous à `properties.ipFilterRules` dans la [méthode createorupdate](/rest/api/iothub/iothubresource/createorupdate).

## <a name="ip-filter-rule-evaluation"></a>Évaluation de règle de filtre IP

Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

Par exemple, si vous souhaitez accepter les adresses dans la plage 192.168.100.0/22 et rejeter tout le reste, la première règle de la grille doit accepter la plage d’adresses 192.168.100.0/22. La règle suivante doit rejeter toutes les adresses à l’aide de la plage 0.0.0.0/0.

Vous pouvez modifier l’ordre de vos règles de filtre IP dans la grille en cliquant sur les trois points verticaux au début d’une ligne et en effectuant un glisser-déplacer.

Pour enregistrer le nouvel ordre de vos règles de filtre IP, cliquez sur **Enregistrer** .

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-rule-order.png" alt-text="Paramètres de filtre IP par défaut d’IoT Hub":::

## <a name="next-steps"></a>Étapes suivantes

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Superviser avec IoT Hub](monitor-iot-hub.md)
