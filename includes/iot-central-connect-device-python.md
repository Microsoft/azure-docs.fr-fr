---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/24/2020
ms.openlocfilehash: 2eff30333362d461f196972fbaedbeac8f2ae7c9
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033863"
---
## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous avez besoin des éléments suivants :

* Une application Azure IoT Central créée avec le modèle **Application personnalisée**. Pour plus d’informations, consultez [Créer une application](../articles/iot-central/core/quick-deploy-iot-central.md). L’application doit avoir été créée à partir le 14 juillet 2020 ou après.
* Une machine de développement où [Python](https://www.python.org/) version 3.7 ou ultérieure est installé. Vous pouvez exécuter `python --version` au niveau de la ligne de commande pour vérifier la version. Python est disponible pour un large éventail de systèmes d’exploitation. Les instructions de ce tutoriel supposent que vous exécutez la commande **python** à l’invite de commandes Windows.
* Une copie locale du dépôt GitHub [SDK Microsoft Azure IoT pour Python](https://github.com/Azure/azure-iot-sdk-python) qui contient l’exemple de code. Utilisez ce lien pour télécharger une copie du dépôt : [Télécharger le fichier ZIP](https://github.com/Azure/azure-iot-sdk-python/archive/master.zip). Décompressez ensuite le fichier dans un emplacement approprié de votre ordinateur local.

## <a name="review-the-code"></a>Vérifier le code

Dans la copie du SDK Microsoft Azure IoT pour Python que vous avez téléchargée précédemment, ouvrez le fichier *azure-iot-sdk-python/azure-iot-device/samples/pnp/simple_thermostat.py* dans un éditeur de texte.

Quand vous exécutez l’exemple pour vous connecter à IoT Central, il utilise le service Device Provisioning (DPS) pour inscrire l’appareil et générer une chaîne de connexion. L’exemple récupère les informations de connexion DPS dont il a besoin à partir de l’environnement de ligne de commande.

La fonction `main` :

* Utilise le service DPS pour provisionner l’appareil. Les informations de provisionnement incluent l’ID de modèle. IoT Central utilise l’ID de modèle pour identifier ou générer le modèle d’appareil correspondant à cet appareil. Pour en savoir plus, consultez [Associer un appareil à un modèle d’appareil](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).
* Crée un objet `Device_client` et définit l’ID de modèle `dtmi:com:example:Thermostat;1` avant l’ouverture de la connexion.
* Envoie la propriété `maxTempSinceLastReboot` à IoT Central.
* Crée un écouteur pour la commande `getMaxMinReport`.
* Crée un écouteur de propriété afin d’écouter les mises à jour de propriétés accessibles en écriture.
* Démarre une boucle pour envoyer des données de télémétrie de température toutes les 10 secondes.

```python
async def main():
    switch = os.getenv("IOTHUB_DEVICE_SECURITY_TYPE")
    if switch == "DPS":
        provisioning_host = (
            os.getenv("IOTHUB_DEVICE_DPS_ENDPOINT")
            if os.getenv("IOTHUB_DEVICE_DPS_ENDPOINT")
            else "global.azure-devices-provisioning.net"
        )
        id_scope = os.getenv("IOTHUB_DEVICE_DPS_ID_SCOPE")
        registration_id = os.getenv("IOTHUB_DEVICE_DPS_DEVICE_ID")
        symmetric_key = os.getenv("IOTHUB_DEVICE_DPS_DEVICE_KEY")

        registration_result = await provision_device(
            provisioning_host, id_scope, registration_id, symmetric_key, model_id
        )

        if registration_result.status == "assigned":

            device_client = IoTHubDeviceClient.create_from_symmetric_key(
                symmetric_key=symmetric_key,
                hostname=registration_result.registration_state.assigned_hub,
                device_id=registration_result.registration_state.device_id,
                product_info=model_id,
            )
        else:
            raise RuntimeError(
                "Could not provision device. Aborting Plug and Play device connection."
            )

    elif switch == "connectionString":

        # ...

    # Connect the client.
    await device_client.connect()

    max_temp = 10.96  # Initial Max Temp otherwise will not pass certification
    await device_client.patch_twin_reported_properties({"maxTempSinceLastReboot": max_temp})

    listeners = asyncio.gather(
        execute_command_listener(
            device_client,
            method_name="getMaxMinReport",
            user_command_handler=max_min_handler,
            create_user_response_handler=create_max_min_report_response,
        ),
        execute_property_listener(device_client),
    )

    async def send_telemetry():
        global max_temp
        global min_temp
        current_avg_idx = 0

        while True:
            current_temp = random.randrange(10, 50)
            if not max_temp:
                max_temp = current_temp
            elif current_temp > max_temp:
                max_temp = current_temp

            if not min_temp:
                min_temp = current_temp
            elif current_temp < min_temp:
                min_temp = current_temp

            avg_temp_list[current_avg_idx] = current_temp
            current_avg_idx = (current_avg_idx + 1) % moving_window_size

            temperature_msg1 = {"temperature": current_temp}
            await send_telemetry_from_thermostat(device_client, temperature_msg1)
            await asyncio.sleep(8)

    send_telemetry_task = asyncio.create_task(send_telemetry())

    # ...
```

La fonction `provision_device` utilise le service DPS pour provisionner l’appareil et l’inscrire auprès d’IoT Central. La fonction comprend l’ID de modèle d’appareil, qu’IoT Central utilise pour [associer un appareil à un modèle d’appareil](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template), dans la charge utile de provisionnement :

```python
async def provision_device(provisioning_host, id_scope, registration_id, symmetric_key, model_id):
    provisioning_device_client = ProvisioningDeviceClient.create_from_symmetric_key(
        provisioning_host=provisioning_host,
        registration_id=registration_id,
        id_scope=id_scope,
        symmetric_key=symmetric_key,
    )
    provisioning_device_client.provisioning_payload = {"modelId": model_id}
    return await provisioning_device_client.register()
```

La fonction `execute_command_listener` gère les demandes de commande, exécute la fonction `max_min_handler` quand l’appareil reçoit la commande `getMaxMinReport`, puis exécute la fonction `create_max_min_report_response` pour générer la réponse :

```python
async def execute_command_listener(
    device_client, method_name, user_command_handler, create_user_response_handler
):
    while True:
        if method_name:
            command_name = method_name
        else:
            command_name = None

        command_request = await device_client.receive_method_request(command_name)
        print("Command request received with payload")
        print(command_request.payload)

        values = {}
        if not command_request.payload:
            print("Payload was empty.")
        else:
            values = command_request.payload

        await user_command_handler(values)

        response_status = 200
        response_payload = create_user_response_handler(values)

        command_response = MethodResponse.create_from_method_request(
            command_request, response_status, response_payload
        )

        try:
            await device_client.send_method_response(command_response)
        except Exception:
            print("responding to the {command} command failed".format(command=method_name))
```

`async def execute_property_listener` gère les mises à jour de propriétés accessibles en écriture comme `targetTemperature` et génère la réponse JSON :

```python
async def execute_property_listener(device_client):
    ignore_keys = ["__t", "$version"]
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call

        print("the data in the desired properties patch was: {}".format(patch))

        version = patch["$version"]
        prop_dict = {}

        for prop_name, prop_value in patch.items():
            if prop_name in ignore_keys:
                continue
            else:
                prop_dict[prop_name] = {
                    "ac": 200,
                    "ad": "Successfully executed patch",
                    "av": version,
                    "value": prop_value,
                }

        await device_client.patch_twin_reported_properties(prop_dict)
```

La fonction `send_telemetry_from_thermostat` envoie les messages de télémétrie à IoT Central :

```python
async def send_telemetry_from_thermostat(device_client, telemetry_msg):
    msg = Message(json.dumps(telemetry_msg))
    msg.content_encoding = "utf-8"
    msg.content_type = "application/json"
    print("Sent message")
    await device_client.send_message(msg)
```

## <a name="get-connection-information"></a>Obtenir des informations de connexion

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Exécuter le code

Pour exécuter l’exemple d’application, ouvrez un environnement de ligne de commande, puis accédez au dossier *azure-iot-sdk-python/azure-iot-device/samples/pnp* qui contient l’exemple de fichier *simple_thermostat.py*.

[!INCLUDE [iot-central-connection-environment](iot-central-connection-environment.md)]

Installez les packages nécessaires :

```cmd/sh
pip install azure-iot-device
```

Exécutez l’exemple :

```cmd/sh
python simple_thermostat.py
```

La sortie suivante montre l’inscription de l’appareil et la connexion à IoT Central. L’exemple envoie la propriété `maxTempSinceLastReboot` avant de commencer à envoyer des données de télémétrie :

```cmd/sh
Device was assigned
iotc-.......azure-devices.net
sample-device-01
Listening for command requests and property updates
Press Q to quit
Sending telemetry for temperature
Sent message
Sent message
Sent message
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Vous pouvez voir comment l’appareil répond aux commandes et aux mises à jour de propriétés :

```cmd/sh
Sent message
the data in the desired properties patch was: {'targetTemperature': {'value': 86.3}, '$version': 2}
Sent message

...

Sent message
Command request received with payload
2020-10-14T08:00:00.000Z
Will return the max, min and average temperature from the specified time 2020-10-14T08:00:00.000Z to the current time
Done generating
{"avgTemp": 31.5, "endTime": "2020-10-16T10:07:41.580722", "maxTemp": 49, "minTemp": 12, "startTime": "2020-10-16T10:06:21.580632"}
```
