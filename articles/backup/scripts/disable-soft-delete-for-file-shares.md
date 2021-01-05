---
title: Exemple de script - Désactiver la suppression réversible pour un partage de fichiers
description: Découvrez comment utiliser un script afin de désactiver la suppression réversible pour les partages de fichiers dans un compte de stockage.
ms.topic: sample
ms.date: 02/02/2020
ms.openlocfilehash: 172a7bd19ffd07ca7900a457c447362343e5f185
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84121282"
---
# <a name="disable-soft-delete-for-file-shares-in-a-storage-account"></a>Désactiver la suppression réversible pour les partages de fichiers dans un compte de stockage

Ce document explique le processus de désactivation de la suppression réversible pour les partages de fichiers dans un compte de stockage.

Procédez comme suit :

1. Installez armclient. Pour savoir comment l’installer, visitez [ce lien](https://github.com/projectkudu/ARMClient).

2. Enregistrez les deux fichiers de corps de la demande suivants dans un dossier sur votre ordinateur.

    ```json
    rqbody-enableSoftDelete.json

    {
    "properties": {
        "shareDeleteRetentionPolicy": {
        "enabled":true,
        "days": 14
        }
    },
    "cors": {
        "corsRules": []
    }
    }

    rqbody-disableSoftDelete.json

    {
    "properties": {
        "shareDeleteRetentionPolicy": {
        "enabled":false,
        "days": 0
        }
    },
    "cors": {
        "corsRules": []
    }
    }
    ```

3. Gardez l’ID ARM (Azure Resource Manager) de votre compte de stockage à portée de main. Par exemple : `/subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/afsshare/providers/Microsoft.Storage/storageAccounts/inquirytest`

4. Connectez-vous avec vos informations d’identification en exécutant **armclient login**.

5. Obtenez les propriétés de suppression réversible actuelles des partages de fichiers dans le compte de stockage.

    L’opération GET suivante récupère les propriétés de suppression réversible des partages de fichiers dans le compte *inquirytest* :

    ```cmd
    armclient get /subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/afsshare /providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/default?api-version=2019-04-01
    ```

    ```output
    {
    "id": "/subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/Bugbash/providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/de
    fault",
    "name": "default",
    "type": "Microsoft.Storage/storageAccounts/fileServices",
    "properties": {
        "cors": {
        "corsRules": []
        },
        "shareDeleteRetentionPolicy": {
        "enabled": true,
        "days": 14
        }
    }
    }
    ```

6. Désactivez la suppression réversible pour les partages de fichiers dans le compte de stockage.

    L’opération PUT suivante désactive les propriétés de suppression réversible des partages de fichiers dans le compte *inquirytest* :

    ```cmd
    armclient put /subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/afsshare /providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/default?api-version=2019-04-01 .\rqbody-disableSoftDelete.json
    ```

    ```Output
    {
    "id": "/subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/Bugbash/providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/de
    fault",
    "name": "default",
    "type": "Microsoft.Storage/storageAccounts/fileServices",
    "properties": {
        "shareDeleteRetentionPolicy": {
        "enabled": false,
        "days": 0
        }
    }
    }
    ```

7. Si vous souhaitez réactiver la suppression réversible, utilisez l’exemple suivant.

    L’opération PUT suivante active les propriétés de suppression réversible des partages de fichiers dans le compte « inquirytest ».

    ```cmd
    armclient put /subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/afsshare /providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/default?api-version=2019-04-01 .\rqbody-EnableSoftDelete.json
    ```

    ```Output
    {
    "id": "/subscriptions/37aa2d43-d4f5-4322-bae0-6ee11c627f50/resourceGroups/Bugbash/providers/Microsoft.Storage/storageAccounts/inquirytest/fileServices/default",
    "name": "default",
    "type": "Microsoft.Storage/storageAccounts/fileServices",
    "properties": {
        "shareDeleteRetentionPolicy": {
        "enabled": true,
        "days": 14
        }
    }
    }
    ```
