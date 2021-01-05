---
title: Configuration locale de l’agent de sécurité (C)
description: Découvrez Defender pour les configurations locales de l’agent pour C.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: elazark
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/08/2020
ms.author: v-ekrieg
ms.openlocfilehash: d96a73c3ba996fecf24a4232e1391f0b814be868
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91851252"
---
# <a name="understanding-the-localconfigurationjson-file---c-agent"></a>Fonctionnement du fichier LocalConfiguration.json - Agent C

L’agent de sécurité Defender pour IoT utilise des configurations d’un fichier de configuration local.
L’agent de sécurité lit la configuration une seule fois, au démarrage de l’agent.
La configuration trouvée dans le fichier de configuration local contient la configuration de l’authentification et d’autres configurations associées à l’agent.
Le fichier contient les configurations dans des paires « clé/valeur » dans la notation JSON et les configurations sont renseignées quand l’agent est installé.

Par défaut, le fichier est situé dans : /var/ASCIoTAgent/LocalConfiguration.json

Les modifications apportées au fichier de configuration ont lieu au redémarrage de l’agent.

## <a name="security-agent-configurations-for-c"></a>Configurations de l’agent de sécurité pour C

| Nom de la configuration | Valeurs possibles | Détails |
|:-----------|:---------------|:--------|
| AgentID | GUID | L’identificateur unique de l’agent |
| TriggerdEventsInterval | Chaîne ISO8601 | Intervalle du planificateur pour la collecte des événements déclenchés |
| ConnectionTimeout | Chaîne ISO8601 | Période avant l’expiration de la connexion à IoThub |
| Authentification | JsonObject | Configuration de l'authentification. Cet objet contient toutes les informations nécessaires pour l’authentification avec IoTHub |
| Identité | « DPS », « SecurityModule », « Device » | Identité d’authentification - DPS si l’authentification est effectuée via DPS, SecurityModule si l’authentification est effectuée via les informations d’identification du module de sécurité ou Device si l’authentification est effectuée avec les informations d’identification de l’appareil |
| AuthenticationMethod | « SasToken », « SelfSignedCertificate » | Secret de l’utilisateur pour l’authentification : choisissez SasToken si le secret de l’utilisateur est une clé symétrique ou SelfSignedCertificate si le secret est un certificat auto-signé  |
| FilePath | Chemin au fichier (chaîne) | Chemin d’accès au fichier qui contient le secret d’authentification |
| HostName | string | Nom d’hôte du hub Azure IoT, en général, <mon-hub>.azure-devices.net |
| deviceId | string | ID de l’appareil (comme inscrit dans Azure IoT Hub) |
| DPS | JsonObject | Configurations relatives à DPS |
| IDScope | string | Étendue de l’ID de DPS. |
| ID d’inscription | string  | ID de l'inscription de l'appareil DPS |
| Journalisation | JsonObject | Configurations liées à l’enregistreur de l’agent |
| SystemLoggerMinimumSeverity | 0 <= nombre <= 4 | les messages de journal égaux ou supérieurs à cette gravité seront enregistrés dans/var/log/syslog (0 est la gravité la plus faible) |
| DiagnosticEventMinimumSeverity | 0 <= nombre <= 4 | les messages de journal égaux ou supérieurs à cette gravité seront envoyés en tant qu’événements de diagnostic (0 est la gravité la plus faible) |

## <a name="security-agent-configurations-code-example"></a>Exemple de configurations de l’agent de sécurité

```json
{
    "Configuration" : {
        "AgentId" : "b97faf0a-0f57-471f-9dab-46a8e1764946",
        "TriggerdEventsInterval" : "PT2M",
        "ConnectionTimeout" : "PT30S",
        "Authentication" : {
            "Identity" : "Device",
            "AuthenticationMethod" : "SasToken",
            "FilePath" : "/path/to/my/SymmetricKey",
            "DeviceId" : "my-device",
            "HostName" : "my-iothub.azure-devices.net",
            "DPS" : {
                "IDScope" : "",
                "RegistrationId" : ""
            }
        },
        "Logging": {
            "SystemLoggerMinimumSeverity": 0,
            "DiagnoticEventMinimumSeverity": 2
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- Lire la [vue d’ensemble](overview.md) du service Defender pour IoT
- Découvrir plus en détail l’[architecture](architecture.md) de Defender pour IoT
- Activer le [service](quickstart-onboard-iot-hub.md) Defender pour IoT
- Lire le [FAQ](resources-frequently-asked-questions.md) sur le service Defender pour IoT
- Découvrir comment accéder aux [données de sécurité brutes](how-to-security-data-access.md)
- Comprendre les [recommandations](concept-recommendations.md)
- Comprendre les [alertes de sécurité](concept-security-alerts.md)
