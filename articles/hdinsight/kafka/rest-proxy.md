---
title: Proxy REST Apache Kafka - Azure HDInsight
description: Découvrez comment effectuer des opérations Apache Kafka à l’aide d’un proxy REST Kafka sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: how-to
ms.custom: has-adal-ref, devx-track-python
ms.date: 04/03/2020
ms.openlocfilehash: d6c45a5c8062c3b6441309361037f8755a552074
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95791899"
---
# <a name="interact-with-apache-kafka-clusters-in-azure-hdinsight-using-a-rest-proxy"></a>Interagir avec des clusters Apache Kafka dans Azure HDInsight à l’aide d’un proxy REST

Le proxy REST Kafka vous permet d’interagir avec votre cluster Kafka via une API REST sur HTTPS. Cette action signifie que vos clients Kafka peuvent se trouver en dehors du réseau virtuel. Les clients peuvent effectuer des appels HTTPS simples et sécurisés au cluster Kafka, au lieu de s’appuyer sur des bibliothèques Kafka. Cet article montre comment créer un cluster Kafka avec un proxy REST activé. Il contient également un exemple de code qui montre comment effectuer des appels au proxy REST.

## <a name="rest-api-reference"></a>Référence d’API REST

Pour plus d’informations sur les opérations prises en charge par l’API REST Kafka, consultez la [référence de l’API de proxy REST HDInsight Kafka](/rest/api/hdinsight-kafka-rest-proxy).

## <a name="background"></a>Arrière-plan

![Conception du proxy REST Kafka](./media/rest-proxy/rest-proxy-architecture.png)

Pour obtenir la spécification complète des opérations prises en charge par l’API, consultez [API de proxy REST Apache Kafka](/rest/api/hdinsight-kafka-rest-proxy).

### <a name="rest-proxy-endpoint"></a>Point de terminaison de proxy REST

La création d’un cluster HDInsight Kafka avec un proxy REST entraîne la création d’un point de terminaison public pour votre cluster. Celui-ci apparaît dans les **Propriétés** de votre cluster HDInsight dans le portail Azure.

### <a name="security"></a>Sécurité

L’accès au proxy REST Kafka est géré avec des groupes de sécurité Azure Active Directory. Quand vous créez le cluster Kafka, spécifiez le groupe de sécurité Azure AD avec un accès au point de terminaison REST. Les clients Kafka qui ont besoin d’accéder au proxy REST doivent être inscrits auprès de ce groupe par le propriétaire du groupe, par le biais du portail ou de PowerShell.

Pour les requêtes de point de terminaison de proxy REST, les applications clientes doivent obtenir un jeton OAuth. Le jeton sert à vérifier l’appartenance au groupe de sécurité. Vous trouverez ci-dessous un [exemple d’application cliente](#client-application-sample) qui montre comment obtenir un jeton OAuth. L’application cliente transmet le jeton OAuth dans la requête HTTPS au proxy REST.

> [!NOTE]
> Consultez [Gérer l’accès aux applications et aux ressources à l’aide de groupes Azure Active Directory](../../active-directory/fundamentals/active-directory-manage-groups.md) pour en savoir plus sur les groupes de sécurité AAD. Pour plus d’informations sur le fonctionnement des jetons OAuth, consultez [Autoriser l’accès aux applications web Azure Active Directory à l’aide du flux d’octroi de code OAuth 2.0](../../active-directory/azuread-dev/v1-protocols-oauth-code.md).

## <a name="kafka-rest-proxy-with-network-security-groups"></a>Proxy REST Kafka avec des groupes de sécurité réseau
Si vous apportez votre propre réseau virtuel et contrôlez le trafic réseau avec des groupes de sécurité réseau, autorisez le trafic **entrant** sur le port **9400** en plus du port 443. Cela garantit que le serveur proxy REST Kafka est accessible.

## <a name="prerequisites"></a>Prérequis

1. inscrivez une application auprès d’Azure AD ; Les applications clientes que vous écrivez pour interagir avec le proxy REST Kafka utilisent l’identificateur et le secret de cette application pour s’authentifier auprès d’Azure.

1. Créez un groupe de sécurité Azure AD. Ajoutez l’application que vous avez inscrite auprès d’Azure AD au groupe de sécurité en tant que **membre** du groupe. Ce groupe de sécurité sera utilisé pour contrôler les applications qui sont autorisées à interagir avec le proxy REST. Pour plus d’informations sur la création de groupes Azure AD, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

    Confirmez que le groupe est de type **Sécurité**.
    ![Groupe de sécurité](./media/rest-proxy/rest-proxy-group.png)

    Confirmez que l’application est membre du groupe.
    ![Vérifier l’appartenance](./media/rest-proxy/rest-proxy-membergroup.png)

## <a name="create-a-kafka-cluster-with-rest-proxy-enabled"></a>Créer un cluster Kafka avec le proxy REST activé

Les étapes ci-dessous utilisent le portail Azure. Pour obtenir un exemple utilisant Azure CLI, consultez [Créer un cluster proxy REST Apache Kafka en utilisant Azure CLI](tutorial-cli-rest-proxy.md).

1. Pendant le workflow de création du cluster Kafka, sous l’onglet **Sécurité et réseau**, activez l’option **Activer le proxy REST Kafka**.

     ![Capture d'écran représentant la page Créer un cluster HDInsight, sur laquelle l'option Sécurité et réseau est sélectionnée.](./media/rest-proxy/azure-portal-cluster-security-networking-kafka-rest.png)

1. Cliquez sur **Sélectionner un groupe de sécurité**. Dans la liste des groupes de sécurité, sélectionnez le groupe de sécurité qui doit avoir accès au proxy REST. Vous pouvez utiliser la zone de recherche pour retrouver le groupe de sécurité approprié. Cliquez sur le bouton **Sélectionner** en bas.

     ![Capture d'écran représentant la page Créer un cluster HDInsight, qui contient une option de sélection du groupe de sécurité.](./media/rest-proxy/azure-portal-cluster-security-networking-kafka-rest2.png)

1. Effectuez les étapes restantes pour créer votre cluster, comme décrit dans [Créer un cluster Apache Kafka dans Azure HDInsight à l’aide du portail Azure](./apache-kafka-get-started.md).

1. Une fois le cluster créé, accédez aux propriétés du cluster pour enregistrer l’URL du proxy REST Kafka.

     ![afficher l’URL du proxy REST](./media/rest-proxy/apache-kafka-rest-proxy-view-proxy-url.png)

## <a name="client-application-sample"></a>Exemple d’application cliente

Vous pouvez utiliser le code Python ci-dessous pour interagir avec le proxy REST sur votre cluster Kafka. Pour exécuter l’exemple de code, procédez comme suit :

1. Enregistrez l’exemple de code sur un ordinateur sur lequel Python est installé.
1. Installez les dépendances Python requises en exécutant `pip3 install msal`.
1. Modifiez la section de code **Configure these properties** et mettez à jour les propriétés suivantes pour votre environnement :

    |Propriété |Description |
    |---|---|
    |ID client|Locataire Azure où se trouve votre abonnement.|
    |ID client|ID de l’application que vous avez inscrite dans le groupe de sécurité.|
    |Clé secrète client|Secret de l’application que vous avez inscrite dans le groupe de sécurité.|
    |Kafkarest_endpoint|Récupérez cette valeur sous l’onglet **Propriétés** de la vue d’ensemble du cluster, comme décrit dans la [section de déploiement](#create-a-kafka-cluster-with-rest-proxy-enabled). Elle doit respecter le format suivant : `https://<clustername>-kafkarest.azurehdinsight.net`|

1. Depuis la ligne de commande, exécutez le fichier Python en exécutant `sudo python3 <filename.py>`

Ce code effectue l’action suivante :

1. Il récupère un jeton OAuth à partir d’Azure AD.
1. Il montre comment effectuer une requête auprès du proxy REST Kafka.

Pour plus d’informations sur l’obtention de jetons OAuth en Python, consultez [Classe AuthenticationContext Python](/python/api/adal/adal.authentication_context.authenticationcontext). Vous pouvez constater un délai avant l’affichage ici des `topics` qui ne sont pas créées ou supprimées par le biais du proxy REST Kafka. Ce délai est dû à l’actualisation du cache. Le champ **Valeur** de l’API de producteur a été amélioré. À présent, il accepte les objets JSON et tout formulaire sérialisé.

```python
#Required python packages
#pip3 install msal

import json
import msal
import random
import requests
import string
import sys
import time

def get_random_string():
    letters = string.ascii_letters
    random_string = ''.join(random.choice(letters) for i in range(7))

    return random_string


#--------------------------Configure these properties-------------------------------#
# Tenant ID for your Azure Subscription
tenant_id = 'ABCDEFGH-1234-1234-1234-ABCDEFGHIJKL'
# Your Client Application Id
client_id = 'XYZABCDE-1234-1234-1234-ABCDEFGHIJKL'
# Your Client Credentials
client_secret = 'password'
# kafka rest proxy -endpoint
kafkarest_endpoint = "https://<clustername>-kafkarest.azurehdinsight.net"
#--------------------------Configure these properties-------------------------------#

# Get access token
# Scope
scope = 'https://hib.azurehdinsight.net/.default'
#Authority
authority = 'https://login.microsoftonline.com/' + tenant_id

app = msal.ConfidentialClientApplication(
    client_id , client_secret, authority,
    #cache - For details on how look at this example: https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/app.py
)

# The pattern to acquire a token looks like this.
result = None
result = app.acquire_token_for_client(scopes=[scope])
accessToken = result['access_token']
verify_https = True
request_timeout = 10

# Print access token
print("Access token: " + accessToken)

# API format
api_version = 'v1'
api_format = kafkarest_endpoint + '/{api_version}/{rest_api}'
get_topic_api = 'metadata/topics'
topic_api_format = 'topics/{topic_name}'
producer_api_format = 'producer/topics/{topic_name}'
consumer_api_format = 'consumer/topics/{topic_name}/partitions/{partition_id}/offsets/{offset}?count={count}'  # by default count = 1

# Request header
headers = {
    'Authorization': 'Bearer ' + accessToken,
    'Content-type': 'application/json'          # set Content-type to 'application/json'
}

# New topic
new_topic = 'hello_topic_' + get_random_string()
print("Topic " + new_topic + " is going to be used for demo.")

topics = []

# Create a  new topic
# Example of topic config
topic_config = {
    "partition_count": 1,
    "replication_factor": 1,
    "topic_properties": {
        "retention.ms": 604800000,
        "min.insync.replicas": "1"
    }
}

create_topic_url = api_format.format(api_version=api_version, rest_api=topic_api_format.format(topic_name=new_topic))
response = requests.put(create_topic_url, headers=headers, json=topic_config, timeout=request_timeout, verify=verify_https)
print(response.content)

if response.ok:
    while new_topic not in topics:
        print("The new topic " + new_topic + " is not visible yet. sleep 30 seconds...")
        time.sleep(30)
        # List Topic
        get_topic_url = api_format.format(api_version=api_version, rest_api=get_topic_api)

        response = requests.get(get_topic_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
        topic_list = response.json()
        topics = topic_list.get("topics", [])
else:
    print("Topic " + new_topic + " was created. Exit.")
    sys.exit(1)

# Produce messages to new_topic
# Example payload of Producer REST API
payload_json = {
    "records": [
        {
            "key": "key1",
            "value": "**********"         # A string                              
        },
        {
            "partition": 0,
            "value": 5                    # An integer
        },
        {
            "value": 3.14                 # A floating number
        },
        {
            "value": {                    # A JSON object
                "id": 1,
                "name": "HDInsight Kafka REST proxy"
            }
        },
        {
            "value": [                    # A list of JSON objects
                {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1"
                },
                {
                    "id": 2,
                    "name": "HDInsight Kafka REST proxy 2"
                },
                {
                    "id": 3,
                    "name": "HDInsight Kafka REST proxy 3"
                }
            ]
        },
        {
            "value": {                  # A nested JSON object
                "group id": 1,
                "HDI Kafka REST": {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1"
                },
                "HDI Kafka REST server info": {
                    "id": 1,
                    "name": "HDInsight Kafka REST proxy 1",
                    "servers": [
                        {
                            "server id": 1,
                            "server name": "HDInsight Kafka REST proxy server 1"
                        },
                        {
                            "server id": 2,
                            "server name": "HDInsight Kafka REST proxy server 2"
                        },
                        {
                            "server id": 3,
                            "server name": "HDInsight Kafka REST proxy server 3"
                        }
                    ]
                }
            }
        }
    ]
}

print("Payloads in a Producer request: \n", payload_json)
producer_url = api_format.format(api_version=api_version, rest_api=producer_api_format.format(topic_name=new_topic))
response = requests.post(producer_url, headers=headers, json=payload_json, timeout=request_timeout, verify=verify_https)
print(response.content)

# Consume messages from the topic
partition_id = 0
offset = 0
count = 2

while True:
    consumer_url = api_format.format(api_version=api_version, rest_api=consumer_api_format.format(topic_name=new_topic, partition_id=partition_id, offset=offset, count=count))
    print("Consuming " + str(count) + " messages from offset " + str(offset))

    response = requests.get(consumer_url, headers=headers, timeout=request_timeout, verify=verify_https)

    if response.ok:
        messages = response.json()
        print("Consumed messages: \n" + json.dumps(messages, indent=2))
        next_offset = response.headers.get("NextOffset")
        if offset == next_offset or not messages.get("records", []):
            print("Consumer caught up with producer. Exit for now...")
            break

        offset = next_offset

    else:
        print("Error " + str(response.status_code))
        break
        
# List partitions
get_partitions_url = api_format.format(api_version=api_version, rest_api=partitions_api_format.format(topic_name=new_topic))
print("Fetching partitions from  " + get_partitions_url)

response = requests.get(get_partitions_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
partition_list = response.json()
print("Partition list: \n" + json.dumps(partition_list, indent=2))

# List a partition
get_partition_url = api_format.format(api_version=api_version, rest_api=partition_api_format.format(topic_name=new_topic, partition_id=partition_id))
print("Fetching metadata of a partition from  " + get_partition_url)

response = requests.get(get_partition_url, headers={'Authorization': 'Bearer ' + accessToken}, timeout=request_timeout, verify=verify_https)
partition = response.json()
print("Partition metadata: \n" + json.dumps(partition, indent=2))

```

Vous trouverez ci-dessous un autre exemple illustrant comment obtenir un jeton à partir d’Azure pour le proxy REST à l’aide d’une commande curl. **Notez que nous avons besoin du `scope=https://hib.azurehdinsight.net/.default` spécifié lors de l’obtention d’un jeton.**

```cmd
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=<clientid>&client_secret=<clientsecret>&grant_type=client_credentials&scope=https://hib.azurehdinsight.net/.default' 'https://login.microsoftonline.com/<tenantid>/oauth2/v2.0/token'
```

## <a name="next-steps"></a>Étapes suivantes

* [Documents d’informations de référence de l’API Proxy REST Kafka](/rest/api/hdinsight-kafka-rest-proxy/)
