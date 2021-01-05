---
title: Comprendre la prise en charge d’AMQP au niveau d’Azure IoT Hub | Microsoft Docs
description: 'Guide du développeur : prise en charge des appareils se connectant à des points de terminaison IoT Hub côté service et côté appareil en utilisant le protocole AMQP. Inclut des informations sur la prise en charge intégrée d’AMQP dans les kits Azure IoT device SDK.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/30/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7b3dcfc51df7f0fe4291e9c5babccc1444ad32e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81730748"
---
# <a name="communicate-with-your-iot-hub-by-using-the-amqp-protocol"></a>Communication avec votre IoT Hub à l’aide du protocole AMQP

Azure IoT Hub prend en charge le protocole [AMQP (OASIS Advanced Message Queuing Protocol) version 1.0](https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf) afin d’offrir toute une gamme de fonctionnalités par le biais de points de terminaison côté appareil et côté service. Ce document décrit l’utilisation de clients AMQP pour se connecter à un hub IoT en vue d’utiliser la fonctionnalité IoT Hub.

## <a name="service-client"></a>Client du service

### <a name="connect-and-authenticate-to-an-iot-hub-service-client"></a>Se connecter et s’authentifier auprès d’un hub IoT (client du service)

Pour se connecter à un hub IoT à l’aide d’AMQP, un client peut utiliser la [sécurité basée sur les revendications](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) ou l’[authentification SASL (Simple Authentication and Security Layer)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).

Les informations suivantes sont requises pour le client du service :

| Information | Valeur |
|-------------|--------------|
| Nom d’hôte du hub IoT | `<iot-hub-name>.azure-devices.net` |
| Nom de clé | `service` |
| Clé d’accès | Clé primaire ou secondaire associée au service |
| Signature d’accès partagé | Signature d’accès partagé à courte durée de vie au format suivant : `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`. Pour obtenir le code de génération de cette signature, consultez [Contrôler l’accès à IoT Hub](./iot-hub-devguide-security.md#security-token-structure).

L’extrait de code suivant utilise la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) pour se connecter à un hub IoT par le biais d’une liaison d’expéditeur.

```python
import uamqp
import urllib
import time

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '<operation-link-name>'  # example: '/messages/devicebound'

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    iot_hub_name=iot_hub_name, policy_name=policy_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Create a send or receive client
send_client = uamqp.SendClient(uri, debug=True)
receive_client = uamqp.ReceiveClient(uri, debug=True)
```

### <a name="invoke-cloud-to-device-messages-service-client"></a>Appeler des messages cloud-à-appareil (client du service)

Pour en savoir plus sur l’échange de messages cloud-à-appareil entre le service et le hub IoT et entre l’appareil et le hub IoT, consultez [Envoyer des messages cloud-à-appareil à partir de votre hub IoT](iot-hub-devguide-messages-c2d.md). Le service client utilise deux liens afin d’envoyer des messages et de recevoir des commentaires envoyés par les appareils pour les messages précédemment envoyés, comme décrit dans le tableau suivant :

| Créé par | Type de liaison | Chemin de liaison | Description |
|------------|-----------|-----------|-------------|
| Service | Liaison d’expéditeur | `/messages/devicebound` | Les messages cloud-à-appareil destinés aux appareils sont envoyés à cette liaison par le service. Les messages envoyés sur cette liaison ont leur propriété `To` définie sur le chemin de liaison de récepteur de l’appareil cible, `/devices/<deviceID>/messages/devicebound`. |
| Service | Liaison de récepteur | `/messages/serviceBound/feedback` | Messages de commentaires d’achèvement, de rejet et d’abandon provenant d’appareils reçus sur cette liaison par service. Pour plus d’informations sur les messages de commentaires, consultez [Envoyer des messages cloud-à-appareil à partir d’un hub IoT](./iot-hub-devguide-messages-c2d.md#message-feedback). |

L’extrait de code suivant montre comment créer un message cloud-à-appareil et l’envoyer à un appareil à l’aide de la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python).

```python
import uuid
# Create a message and set message property 'To' to the device-bound link on device
msg_id = str(uuid.uuid4())
msg_content = b"Message content goes here!"
device_id = '<device-id>'
to = '/devices/{device_id}/messages/devicebound'.format(device_id=device_id)
ack = 'full'  # Alternative values are 'positive', 'negative', and 'none'
app_props = {'iothub-ack': ack}
msg_props = uamqp.message.MessageProperties(message_id=msg_id, to=to)
msg = uamqp.Message(msg_content, properties=msg_props,
                    application_properties=app_props)

# Send the message by using the send client that you created and connected to the IoT hub earlier
send_client.queue_message(msg)
results = send_client.send_all_messages()

# Close the client if it's not needed
send_client.close()
```

Pour recevoir des commentaires, le client du service crée une liaison de récepteur. L’extrait de code suivant montre comment créer une liaison à l’aide de la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) :

```python
import json

operation = '/messages/serviceBound/feedback'

# ...
# Re-create the URI by using the preceding feedback path and authenticate it
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
batch = receive_client.receive_message_batch(max_batch_size=10)
for msg in batch:
    print('received a message')
    # Check content_type in message property to identify feedback messages coming from device
    if msg.properties.content_type == 'application/vnd.microsoft.iothub.feedback.json':
        msg_body_raw = msg.get_data()
        msg_body_str = ''.join(msg_body_raw)
        msg_body = json.loads(msg_body_str)
        print(json.dumps(msg_body, indent=2))
        print('******************')
        for feedback in msg_body:
            print('feedback received')
            print('\tstatusCode: ' + str(feedback['statusCode']))
            print('\toriginalMessageId: ' + str(feedback['originalMessageId']))
            print('\tdeviceId: ' + str(feedback['deviceId']))
            print
    else:
        print('unknown message:', msg.properties.content_type)
```

Comme le montre le code précédent, un message de commentaire cloud-à-appareil a un type de contenu *application/vnd.microsoft.iothub.feedback.json*. Vous pouvez utiliser les propriétés contenues dans le corps JSON du message pour déduire l’état de remise du message d’origine :

* La clé `statusCode` dans le corps de commentaire a l’une des valeurs suivantes : *Success*, *Expired*, *DeliveryCountExceeded*, *Rejected* ou *Purged*.

* La clé `deviceId` dans le corps de commentaire a l’ID de l’appareil cible.

* La clé `originalMessageId` dans le corps de commentaire a l’ID du message cloud-à-appareil d’origine qui a été envoyé par le service. Vous pouvez utiliser cet état de remise pour mettre en corrélation les messages de commentaires et les messages cloud-à-appareil.

### <a name="receive-telemetry-messages-service-client"></a>Recevoir des messages de télémétrie (client du service)

Par défaut, votre hub IoT stocke les messages de télémétrie d’appareil reçus dans un hub d’événements intégré. Votre client du service peut utiliser le protocole AMQP pour recevoir les événements stockés.

Pour cela, le client du service doit d’abord se connecter au point de terminaison du hub IoT et recevoir une adresse de redirection vers les hubs d’événements intégrés. Le client du service utilise ensuite l’adresse fournie pour se connecter au hub d’événements intégré.

À chaque étape, le client doit présenter les informations suivantes :

* Informations d’identification de service valides (jeton de signature d’accès partagé de service)

* Un chemin correctement mis en forme vers la partition de groupe de consommateurs à partir de laquelle il projette d’extraire les messages. Pour un ID de partition et un groupe de consommateurs donnés, le chemin a le format suivant : `/messages/events/ConsumerGroups/<consumer_group>/Partitions/<partition_id>` (le groupe de consommateurs par défaut est `$Default`)

* Un prédicat de filtrage facultatif pour désigner un point de départ dans la partition. Ce prédicat peut être un numéro de séquence, un décalage ou un horodatage en file d’attente

L’extrait de code suivant utilise la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) pour illustrer les étapes précédentes :

```python
import json
import uamqp
import urllib
import time

# Use the generate_sas_token implementation that's available here: https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
policy_name = 'service'
access_key = '<primary-or-secondary-key>'
operation = '/messages/events/ConsumerGroups/{consumer_group}/Partitions/{p_id}'.format(
    consumer_group='$Default', p_id=0)

username = '{policy_name}@sas.root.{iot_hub_name}'.format(
    policy_name=policy_name, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token(hostname, access_key, policy_name)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

# Optional filtering predicates can be specified by using endpoint_filter
# Valid predicates include:
# - amqp.annotation.x-opt-sequence-number
# - amqp.annotation.x-opt-offset
# - amqp.annotation.x-opt-enqueued-time
# Set endpoint_filter variable to None if no filter is needed
endpoint_filter = b'amqp.annotation.x-opt-sequence-number > 2995'

# Helper function to set the filtering predicate on the source URI


def set_endpoint_filter(uri, endpoint_filter=''):
    source_uri = uamqp.address.Source(uri)
    source_uri.set_filter(endpoint_filter)
    return source_uri


receive_client = uamqp.ReceiveClient(
    set_endpoint_filter(uri, endpoint_filter), debug=True)
try:
    batch = receive_client.receive_message_batch(max_batch_size=5)
except uamqp.errors.LinkRedirect as redirect:
    # Once a redirect error is received, close the original client and recreate a new one to the re-directed address
    receive_client.close()

    sas_auth = uamqp.authentication.SASTokenAuth.from_shared_access_key(
        redirect.address, policy_name, access_key)
    receive_client = uamqp.ReceiveClient(set_endpoint_filter(
        redirect.address, endpoint_filter), auth=sas_auth, debug=True)

# Start receiving messages in batches
batch = receive_client.receive_message_batch(max_batch_size=5)
for msg in batch:
    print('*** received a message ***')
    print(''.join(msg.get_data()))
    print('\t: ' + str(msg.annotations['x-opt-sequence-number']))
    print('\t: ' + str(msg.annotations['x-opt-offset']))
    print('\t: ' + str(msg.annotations['x-opt-enqueued-time']))
```

Pour un ID d’appareil donné, le hub IoT utilise un hachage de l’ID d’appareil pour déterminer dans quelle partition stocker ses messages. L’extrait de code précédent illustre la façon dont les événements sont reçus à partir d’une seule partition de ce type. Toutefois, notez qu’une application classique a souvent besoin de récupérer les événements qui sont stockés dans toutes les partitions de hubs d’événements.

## <a name="device-client"></a>Client d’appareil

### <a name="connect-and-authenticate-to-an-iot-hub-device-client"></a>Se connecter et s’authentifier auprès d’un hub IoT (client d’appareil)

Pour se connecter à un hub IoT à l’aide d’AMQP, un appareil peut utiliser la [sécurité basée sur les revendications](https://www.oasis-open.org/committees/download.php/60412/amqp-cbs-v1.0-wd03.doc) ou l’[authentification SASL (Simple Authentication and Security Layer)](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).

Les informations suivantes sont requises pour le client d’appareil :

| Information | Valeur |
|-------------|--------------|
| Nom d’hôte du hub IoT | `<iot-hub-name>.azure-devices.net` |
| Clé d’accès | Clé primaire ou secondaire associée à l’appareil |
| Signature d’accès partagé | Signature d’accès partagé à courte durée de vie au format suivant : `SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`. Pour obtenir le code de génération de cette signature, consultez [Contrôler l’accès à IoT Hub](./iot-hub-devguide-security.md#security-token-structure).

L’extrait de code suivant utilise la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) pour se connecter à un hub IoT par le biais d’une liaison d’expéditeur.

```python
import uamqp
import urllib
import uuid

# Use generate_sas_token implementation available here:
# https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security#security-token-structure
from helper import generate_sas_token

iot_hub_name = '<iot-hub-name>'
hostname = '{iot_hub_name}.azure-devices.net'.format(iot_hub_name=iot_hub_name)
device_id = '<device-id>'
access_key = '<primary-or-secondary-key>'
username = '{device_id}@sas.{iot_hub_name}'.format(
    device_id=device_id, iot_hub_name=iot_hub_name)
sas_token = generate_sas_token('{hostname}/devices/{device_id}'.format(
    hostname=hostname, device_id=device_id), access_key, None)

# e.g., '/devices/{device_id}/messages/devicebound'
operation = '<operation-link-name>'
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
send_client = uamqp.SendClient(uri, debug=True)
```

Les chemins de liaisons suivants sont pris en charge en tant qu’opérations d’appareil :

| Créé par | Type de liaison | Chemin de liaison | Description |
|------------|-----------|-----------|-------------|
| Appareils | Liaison de récepteur | `/devices/<deviceID>/messages/devicebound` | Les messages cloud-à-appareil destinés aux appareils sont reçus sur cette liaison par chaque appareil de destination. |
| Appareils | Liaison d’expéditeur | `/devices/<deviceID>/messages/events` | Les messages appareil-à-cloud qui sont envoyés à partir d’un appareil sont envoyés par le biais de cette liaison. |
| Appareils | Liaison d’expéditeur | `/messages/serviceBound/feedback` | Messages de commentaires cloud-à-appareil envoyés au service sur cette liaison par les appareils. |

### <a name="receive-cloud-to-device-commands-device-client"></a>Recevoir des commandes cloud-à-appareil (client d’appareil)

Les commandes cloud-à-appareil qui sont envoyées aux appareils arrivent sur une liaison `/devices/<deviceID>/messages/devicebound`. Les appareils peuvent recevoir ces messages par lots et utiliser la charge utile de données de message, les propriétés de message, les annotations ou les propriétés de l’application dans le message en fonction des besoins.

L’extrait de code suivant utilise la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) pour recevoir des messages cloud-à-appareil sur un appareil.

```python
# ...
# Create a receive client for the cloud-to-device receive link on the device
operation = '/devices/{device_id}/messages/devicebound'.format(
    device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username),
                                  urllib.quote_plus(sas_token), hostname, operation)

receive_client = uamqp.ReceiveClient(uri, debug=True)
while True:
    batch = receive_client.receive_message_batch(max_batch_size=5)
    for msg in batch:
        print('*** received a message ***')
        print(''.join(msg.get_data()))

        # Property 'to' is set to: '/devices/device1/messages/devicebound',
        print('\tto:                     ' + str(msg.properties.to))

        # Property 'message_id' is set to value provided by the service
        print('\tmessage_id:             ' + str(msg.properties.message_id))

        # Other properties are present if they were provided by the service
        print('\tcreation_time:          ' + str(msg.properties.creation_time))
        print('\tcorrelation_id:         ' +
              str(msg.properties.correlation_id))
        print('\tcontent_type:           ' + str(msg.properties.content_type))
        print('\treply_to_group_id:      ' +
              str(msg.properties.reply_to_group_id))
        print('\tsubject:                ' + str(msg.properties.subject))
        print('\tuser_id:                ' + str(msg.properties.user_id))
        print('\tgroup_sequence:         ' +
              str(msg.properties.group_sequence))
        print('\tcontent_encoding:       ' +
              str(msg.properties.content_encoding))
        print('\treply_to:               ' + str(msg.properties.reply_to))
        print('\tabsolute_expiry_time:   ' +
              str(msg.properties.absolute_expiry_time))
        print('\tgroup_id:               ' + str(msg.properties.group_id))

        # Message sequence number in the built-in event hub
        print('\tx-opt-sequence-number:  ' +
              str(msg.annotations['x-opt-sequence-number']))
```

### <a name="send-telemetry-messages-device-client"></a>Envoyer des messages de télémétrie (client d’appareil)

Vous pouvez également envoyer des messages de télémétrie à partir d’un appareil à l’aide d’AMQP. L’appareil peut éventuellement fournir un dictionnaire de propriétés d’application ou différentes propriétés de message telles qu’un ID de message.

L’extrait de code suivant utilise la [bibliothèque uAMQP dans Python](https://github.com/Azure/azure-uamqp-python) pour envoyer des messages appareil-à-cloud à partir d’un appareil.

```python
# ...
# Create a send client for the device-to-cloud send link on the device
operation = '/devices/{device_id}/messages/events'.format(device_id=device_id)
uri = 'amqps://{}:{}@{}{}'.format(urllib.quote_plus(username), urllib.quote_plus(sas_token), hostname, operation)

send_client = uamqp.SendClient(uri, debug=True)

# Set any of the applicable message properties
msg_props = uamqp.message.MessageProperties()
msg_props.message_id = str(uuid.uuid4())
msg_props.creation_time = None
msg_props.correlation_id = None
msg_props.content_type = None
msg_props.reply_to_group_id = None
msg_props.subject = None
msg_props.user_id = None
msg_props.group_sequence = None
msg_props.to = None
msg_props.content_encoding = None
msg_props.reply_to = None
msg_props.absolute_expiry_time = None
msg_props.group_id = None

# Application properties in the message (if any)
application_properties = { "app_property_key": "app_property_value" }

# Create message
msg_data = b"Your message payload goes here"
message = uamqp.Message(msg_data, properties=msg_props, application_properties=application_properties)

send_client.queue_message(message)
results = send_client.send_all_messages()

for result in results:
    if result == uamqp.constants.MessageState.SendFailed:
        print result
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Les connexions AMQP peuvent être interrompues en raison d’un problème réseau ou de l’expiration du jeton d’authentification (généré dans le code). Le client du service doit gérer ces circonstances et rétablir la connexion et les liaisons si nécessaire. Si un jeton d’authentification expire, le client peut éviter la perte de connexion en renouvelant de manière proactive le jeton avant son expiration.

* Votre client doit parfois être capable de gérer correctement les redirections de liaison. Pour comprendre comment effectuer une telle opération, consultez la documentation de votre client AMQP.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le protocole AMQP, consultez la [spécification du protocole AMQP v1.0](https://www.amqp.org/sites/amqp.org/files/amqp.pdf).

Pour en savoir plus sur la messagerie IoT Hub, consultez :

* [Messages cloud-à-appareil](./iot-hub-devguide-messages-c2d.md)
* [Prendre en charge des protocoles supplémentaires](iot-hub-protocol-gateway.md)
* [Prise en charge du protocole MQTT (Message Queuing Telemetry Transport)](./iot-hub-mqtt-support.md)
