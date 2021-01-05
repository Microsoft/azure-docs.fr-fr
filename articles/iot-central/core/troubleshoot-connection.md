---
title: Résoudre les problèmes de connexion des appareils dans Azure IoT Central | Microsoft Docs
description: Déterminer la raison pour laquelle les données de vos appareils ne s’affichent pas dans IoT Central
services: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 08/13/2020
ms.topic: troubleshooting
ms.service: iot-central
ms.custom: device-developer, devx-track-azurecli
ms.openlocfilehash: 2bbf400840c968587de3a0a0951d28c7c35b210f
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94990888"
---
# <a name="troubleshoot-why-data-from-your-devices-isnt-showing-up-in-azure-iot-central"></a>Déterminer la raison pour laquelle les données de vos appareils ne s’affichent pas dans Azure IoT Central

Ce document aide les développeurs d’appareils à déterminer la raison pour laquelle les données que leurs appareils envoient à IoT Central peuvent ne pas s’afficher dans l’application.

Deux points principaux sont à examiner :

- Problèmes de connectivité de l’appareil
  - Problèmes d’authentification tels que l’appareil présente des informations d’identification non valides
  - Problèmes de connectivité réseau
  - L’appareil n’est pas approuvé ou est bloqué
- Problèmes de forme de charge utile de l’appareil

Ce guide de résolution des problèmes se concentre sur les problèmes de connectivité et de forme de charge utile des appareils.

## <a name="device-connectivity-issues"></a>Problèmes de connectivité de l’appareil

Cette section vous aide à déterminer si vos données parviennent à IoT Central.

Si vous ne l'avez pas déjà fait, installez l’outil `az cli` et l’extension `azure-iot`.

Pour savoir comment installer `az cli`, consultez [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

Pour [installer](/cli/azure/azure-cli-reference-for-IoT?view=azure-cli-latest#extension-reference-installation) l’extension `azure-iot`, exécutez la commande suivante :

```cmd/bash
az extension add --name azure-iot
```

> [!NOTE]
> Vous serez peut-être invité à installer la bibliothèque `uamqp` la première fois que vous exécutez une commande d’extension.

Après avoir installé l’extension `azure-iot`, démarrez votre appareil pour voir si les messages qu’il envoie parviennent à IoT Central.

Utilisez les commandes suivantes pour vous connecter à l’abonnement dans lequel se trouve votre application IoT Central :

```cmd/bash
az login
az set account --subscription <your-subscription-id>
```

Pour surveiller les données de télémétrie que votre appareil envoie, utilisez la commande suivante :

```cmd/bash
az iot central diagnostics monitor-events --app-id <app-id> --device-id <device-name>
```

Si l’appareil s’est correctement connecté à IoT Central, une sortie similaire à la suivante s’affiche :

```cmd/bash
Monitoring telemetry.
Filtering on device: device-001
{
    "event": {
        "origin": "device-001",
        "module": "",
        "interface": "",
        "component": "",
        "payload": {
            "temp": 65.57910343679293,
            "humid": 36.16224660107426
        }
    }
}
```

Pour surveiller les mises à jour de propriétés que votre appareil échange avec IoT Central, utilisez la commande d’aperçu suivante :

```cmd/bash
az iot central diagnostics monitor-properties --app-id <app-id> --device-id <device-name>
```

Si l’appareil envoie correctement les mises à jour de propriétés, une sortie similaire à la suivante s’affiche :

```cmd/bash
Changes in reported properties:
version : 32
{'state': 'true', 'name': {'value': {'value': 'Contoso'}, 'status': 'completed', 'desiredVersion': 7, 'ad': 'completed', 'av': 7, 'ac
': 200}, 'brightness': {'value': {'value': 2}, 'status': 'completed', 'desiredVersion': 7, 'ad': 'completed', 'av': 7, 'ac': 200}, 'p
rocessorArchitecture': 'ARM', 'swVersion': '1.0.0'}
```

Si des données s’affichent dans votre terminal, cela signifie que les données parviennent à votre application IoT Central.

Si aucune donnée ne s’affichent après quelques minutes, essayez d’appuyer sur la touche `Enter` ou `return` de votre clavier, si la sortie est bloquée.

Si les données ne s’affichent toujours pas dans votre terminal, il est probable que votre appareil rencontre des problèmes de connectivité réseau ou qu’il n’envoie pas correctement les données à IoT Central.

### <a name="check-the-provisioning-status-of-your-device"></a>Vérifier l’état d’approvisionnement de votre appareil

Si vos données ne s’affichent pas sur le moniteur, vérifiez l’état d’approvisionnement de votre appareil en exécutant la commande suivante :

```cmd/bash
az iot central device registration-info --app-id <app-id> --device-id <device-name>
```

La sortie suivante montre un exemple d’appareil dont la connexion est bloquée :

```json
{
  "@device_id": "v22upeoqx6",
  "device_registration_info": {
    "device_status": "blocked",
    "display_name": "Environmental Sensor - v22upeoqx6",
    "id": "v22upeoqx6",
    "instance_of": "urn:krhsi_k0u:modelDefinition:w53jukkazs",
    "simulated": false
  },
  "dps_state": {
    "error": "Device is blocked from connecting to IoT Central application. Unblock the device in IoT Central and retry. Learn more:
https://aka.ms/iotcentral-docs-dps-SAS",
    "status": null
  }
}
```

| État d’approvisionnement de l’appareil | Description | Atténuation possible |
| - | - | - |
| approvisionné | Aucun problème immédiatement identifiable. | N/A |
| Inscrit | L’appareil n’est pas encore connecté à IoT Central. | Consultez les journaux de votre appareil afin d’y détecter d’éventuels problèmes de connectivité. |
| Bloqué | La connexion de l’appareil à IoT Central est bloquée. | L’appareil ne peut pas se connecter à l’application IoT Central. Débloquez l’appareil dans IoT Central et réessayez. Pour plus d’informations, consultez [Bloquer les appareils](concepts-get-connected.md#device-status-values). |
| Non approuvé | L’appareil n’est pas approuvé. | L’appareil n’est pas approuvé pour se connecter à l’application IoT Central. Approuvez l’appareil dans IoT Central et réessayez. Pour plus d’informations, consultez [Approuver les appareils](concepts-get-connected.md#device-registration). |
| Non associé | L’appareil n’est pas associé à un modèle d’appareil. | Associez l’appareil à un modèle d’appareil pour permettre à IoT Central de savoir comment analyser les données. |

Pour plus d’informations, consultez [Codes d’état des appareils](concepts-get-connected.md#device-status-values).

### <a name="error-codes"></a>Codes d’erreur

Si vous ne parvenez toujours pas à diagnostiquer la raison pour laquelle vos données ne s’affichent pas dans `monitor-events`, l’étape suivante consiste à rechercher les codes d’erreur signalés par votre appareil.

Démarrez une session de débogage sur votre appareil ou collectez les journaux à partir de ce dernier. Recherchez les codes d’erreur signalés par l’appareil.

Les tableaux suivants présentent les codes d’erreur courants, ainsi que les actions possibles pour les atténuer.

Si vous constatez des problèmes liés à votre flux d’authentification :

| Code d'erreur | Description | Atténuation possible |
| - | - | - |
| 400 | Le corps de la requête n’est pas valide. Par exemple, il ne peut pas être analysé ou l’objet ne peut pas être validé. | Assurez-vous que vous envoyez le corps de la requête qui convient en tant que partie intégrante du flux d’attestation ou utilisez un kit de développement logiciel (SDK) d’appareil. |
| 401 | Le jeton d’autorisation ne peut pas être validé. Par exemple, il a expiré ou ne s’applique pas à l’URI de la requête. Ce code d’erreur est également retourné aux appareils dans le cadre du flux d’attestations du module TPM. | Vérifiez que votre appareil dispose des informations d'identification qui conviennent. |
| 404 | L’instance du service Device Provisioning ou une ressource (par exemple, une inscription) n’existe pas. | [Créez un ticket auprès du support technique](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview). |
| 412 | L’`ETag` de la requête ne correspond pas à l’`ETag` de la ressource existante, conformément à la RFC7232. | [Créez un ticket auprès du support technique](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview). |
| 429 | Les opérations sont limitées par le service. Pour connaître les limites de service spécifiques, consultez [Limites du service IoT Hub Device Provisioning](../../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-device-provisioning-service-limits). | Réduisez la fréquence des messages, répartissez les responsabilités entre plusieurs appareils. |
| 500 | Une erreur interne s’est produite. | [Créez un ticket auprès du support technique](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) pour voir s’il est en mesure de vous aider davantage. |

## <a name="payload-shape-issues"></a>Problèmes de forme de charge utile

Après avoir vérifié que votre appareil envoie des données à IoT Central, l’étape suivante consiste à vous assurer que votre appareil envoie des données dans un format valide.

Il existe deux catégories principales de problèmes courants qui empêchent les données de l’appareil de s’afficher dans IoT Central :

- Non-concordance des données du modèle d’appareil avec l’appareil :
  - Non-concordance des noms tels que des fautes de frappe ou des problèmes de correspondance de casse.
  - Propriétés non modélisées où le schéma n’est pas défini dans le modèle d’appareil.
  - Incompatibilité de schéma tel qu’un type défini dans le modèle comme `boolean`, alors que les données sont une chaîne.
  - Le même nom de télémétrie est défini dans plusieurs interfaces, mais l’appareil n’est pas compatible IoT Plug-and-Play.
- La forme des données ne correspond pas à un JSON valide. Pour en savoir plus, consultez [Charges utiles de télémétrie, de propriétés et de commandes](concepts-telemetry-properties-commands.md).

Pour détecter les catégories dans lesquelles se manifeste votre problème, exécutez la commande la plus adaptée à votre scénario :

- Pour valider la télémétrie, utilisez la commande d’aperçu :

    ```cmd/bash
    az iot central diagnostics validate-messages --app-id <app-id> --device-id <device-name>
    ```

- Pour valider les mises à jour de propriétés, utilisez la commande d’aperçu :

    ```cmd/bash
    az iot central diagnostics validate-properties --app-id <app-id> --device-id <device-name>
    ```

Vous pouvez être invité à installer la bibliothèque `uamqp` la première fois que vous exécutez une commande `validate`.

La sortie suivante présente des exemples de messages d’erreur et d’avertissement issus de la commande Valider :

```cmd/bash
Validating telemetry.
Filtering on device: v22upeoqx6.
Exiting after 300 second(s), or 10 message(s) have been parsed (whichever happens first).
[WARNING] [DeviceId: v22upeoqx6] No encoding found. Expected encoding 'utf-8' to be present in message header.

[WARNING] [DeviceId: v22upeoqx6] Content type '' is not supported. Expected Content type is 'application/json'.

[ERROR] [DeviceId: v22upeoqx6] [TemplateId: urn:krhsi_k0u:modelDefinition:w53jukkazs] Datatype of field 'humid' does not match the da
tatype 'double'. Data '56'. All dates/times/datetimes/durations must be ISO 8601 compliant.
```

Si vous préférez utiliser une interface utilisateur graphique, utilisez la vue **Données brutes** IoT Central pour voir si un objet n’est pas modélisé. La vue **Données brutes** ne détecte pas si l’appareil envoie un JSON incorrect.

:::image type="content" source="media/troubleshoot-connection/raw-data-view.png" alt-text="Capture d’écran de la vue Données brutes":::

Après avoir détecté le problème, vous serez peut-être amené à mettre à jour le microprogramme de l’appareil ou à créer un nouveau modèle d’appareil qui modélise les données précédemment non modélisées.

Si vous choisissez de créer un nouveau modèle qui modélise correctement les données, migrez les appareils de votre ancien modèle vers le nouveau modèle. Pour plus d’informations, consultez [Gérer les appareils dans votre application Azure IoT Central](howto-manage-devices.md).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’aide supplémentaire, vous avez la possibilité de contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). Vous pouvez également soumettre un [ticket de support Azure](https://portal.azure.com/#create/Microsoft.Support).

Pour plus d’informations, consultez [Options d’aide et de support Azure IoT](../../iot-fundamentals/iot-support-help.md).
