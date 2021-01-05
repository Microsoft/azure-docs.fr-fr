---
title: API Module de sécurité pour Azure RTOS
description: API de référence pour le module de sécurité pour Azure RTOS.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/07/2020
ms.author: mlottner
ms.openlocfilehash: 8191d68f701a3be6a37d8623d2d900f6aa6c1851
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930267"
---
# <a name="security-module-for-azure-rtos-api-preview"></a>API Module de sécurité pour Azure RTOS (préversion)

Cette API est conçue pour être utilisée uniquement avec le module de sécurité pour Azure RTOS. Pour obtenir des ressources supplémentaires, consultez le [référentiel GitHub sur le module de sécurité pour Azure RTOS](https://github.com/azure-rtos/azure-iot-preview/releases). 

## <a name="enable-security-module-for-azure-rtos"></a>Activer le module de sécurité pour Azure RTOS

**nx_azure_iot_security_module_enable**

### <a name="prototype"></a>Prototype

```c
UINT nx_azure_iot_security_module_enable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Description

Cette routine active le sous-système du module de sécurité Azure IoT. Une machine à états interne gère la collecte d’événements de sécurité et les transmet à Azure IoT Hub. Une seule instance de NX_AZURE_IOT_SECURITY_MODULE est requise pour gérer la collecte de données.

### <a name="parameters"></a>Paramètres

| Nom | Description |
|---------|---------|
| nx_azure_iot_ptr  [in]    | Pointeur vers un `NX_AZURE_IOT`.  |

### <a name="return-values"></a>Valeurs retournées

|Valeurs retournées  |Description |
|---------|---------|
|NX_AZURE_IOT_SUCCESS|   Le module de sécurité Azure IoT a été correctement activé.     |
|NX_AZURE_IOT_FAILURE   |  L’activation du module de sécurité Azure IoT a échoué, en raison d’une erreur interne.    |
|NX_AZURE_IOT_INVALID_PARAMETER   |  Le module de sécurité requiert une instance de #NX_AZURE_IOT valide.      |

### <a name="allowed-from"></a>Autorisé à partir de

Threads

## <a name="disable-azure-iot-security-module"></a>Désactiver le module de sécurité Azure IoT

**nx_azure_iot_security_module_disable**


### <a name="prototype"></a>Prototype

```c
UINT nx_azure_iot_security_module_disable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Description

Cette routine désactive le sous-système du module de sécurité Azure IoT.

### <a name="parameters"></a>Paramètres

| Nom | Description |
|---------|---------|
| nx_azure_iot_ptr  [in]    | Pointeur vers `NX_AZURE_IOT`. Si la valeur est NULL, l’instance singleton est désactivée. |

### <a name="return-values"></a>Valeurs retournées

|Valeurs retournées  |Description |
|---------|---------|
|NX_AZURE_IOT_SUCCESS     |   Le module de sécurité Azure IoT est correctement désactivé.      |
|NX_AZURE_IOT_INVALID_PARAMETER   |  L’instance de IoT Hub Azure est différente de l’instance composite singleton.       |
|NX_AZURE_IOT_FAILURE    |  La désactivation du module de sécurité Azure IoT a échoué, en raison d’une erreur interne.       |

### <a name="allowed-from"></a>Autorisé à partir de

Threads


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la prise en main du module de sécurité pour Azure RTOS, consultez les articles suivants :

- Consultez la page [Présentation du module de sécurité Defender pour IoT pour Azure RTOS (préversion)](iot-security-azure-rtos.md).