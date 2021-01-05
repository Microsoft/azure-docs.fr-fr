---
title: Prise en charge d’Azure IoT Hub pour les réseaux virtuels
description: Guide pratique pour utiliser le modèle de connectivité des réseaux virtuels avec IoT Hub
services: iot-hub
author: jlian
ms.service: iot-fundamentals
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: jlian
ms.openlocfilehash: f79b03884109ffbd856ff4f60909565daeb0e792
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549109"
---
# <a name="iot-hub-support-for-virtual-networks-with-private-link-and-managed-identity"></a>Prise en charge par IoT Hub des réseaux virtuels avec Private Link et Managed Identity

Par défaut, les noms d'hôtes d’IoT Hub sont mappés à un point de terminaison public avec une adresse IP de routage public sur Internet. Différents clients partagent ce point de terminaison public IoT Hub, et tous les appareils IoT qui se trouvent sur des réseaux étendus et sur des réseaux locaux peuvent y accéder.

![Point de terminaison public IoT Hub](./media/virtual-network-support/public-endpoint.png)

Les fonctionnalités IoT Hub, notamment le [routage de messages](./iot-hub-devguide-messages-d2c.md), le [chargement de fichiers](./iot-hub-devguide-file-upload.md) et [l'importation/exportation d’appareils en bloc](./iot-hub-bulk-identity-mgmt.md) ont également besoin d’une connectivité IoT Hub vers une ressource Azure appartenant au client via son point de terminaison public. L’ensemble de ces chemins de connectivité constitue le trafic de sortie entre IoT Hub et les ressources du client.

Plusieurs raisons peuvent vous pousser à restreindre la connectivité aux ressources Azure (y compris IoT Hub) via votre propre réseau virtuel. Les raisons sont les suivantes :

* Introduire un isolement réseau pour votre hub IoT en empêchant l’exposition de la connectivité à l’Internet public.

* Permettre une expérience de connectivité privée à partir de vos ressources réseau sur site en garantissant que vos données et votre trafic sont transmis directement au réseau principal Azure.

* Empêcher les attaques par exfiltration à partir de réseaux locaux sensibles. 

* Suivre des modèles de connectivité établis à l'échelle d’Azure en utilisant des [points de terminaison privés](../private-link/private-endpoint-overview.md).

Cet article explique comment atteindre ces objectifs en utilisant [Azure Private Link](../private-link/private-link-overview.md) pour la connectivité d'entrée vers IoT Hub et l'exception des services Microsoft approuvés pour la connectivité de sortie d’IoT Hub vers d'autres ressources Azure.

## <a name="ingress-connectivity-to-iot-hub-using-azure-private-link"></a>Connectivité d’entrée vers IoT Hub avec Azure Private Link

Un point de terminaison privé est une adresse IP privée attribuée à l'intérieur d'un réseau virtuel appartenant au client, qui permet l’accès à une ressource Azure. Avec Azure Private Link, vous avez la possibilité de configurer un point de terminaison privé grâce auquel votre hub IoT pourra autoriser les services situés à l'intérieur de votre réseau virtuel à atteindre IoT Hub sans avoir besoin d'envoyer du trafic au point de terminaison public d’IoT Hub. De même, vos appareils locaux peuvent utiliser un [réseau privé virtuel (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou un peering [ExpressRoute](https://azure.microsoft.com/services/expressroute/) pour obtenir une connectivité à votre réseau virtuel et à votre hub IoT (via son point de terminaison privé). Il vous est ainsi possible de restreindre ou de bloquer entièrement la connectivité aux points de terminaison publics de votre hub IoT à l’aide du [filtre IP IoT Hub](./iot-hub-ip-filtering.md) ou du [bouton bascule d’accès au réseau public](iot-hub-public-network-access.md). Cette approche permet de maintenir la connectivité des appareils à votre hub à l’aide du point de terminaison privé. Cette configuration s’adresse principalement aux appareils situés à l'intérieur d'un réseau local. Cette configuration n'est pas conseillée pour les appareils déployés sur un réseau étendu.

![Entrée du réseau virtuel IoT Hub](./media/virtual-network-support/virtual-network-ingress.png)

Avant de commencer, assurez-vous que les conditions préalables suivantes sont remplies :

* Vous avez [créé un réseau virtuel Azure](../virtual-network/quick-create-portal.md) avec un sous-réseau dans lequel sera créé le point de terminaison privé.

* Pour les appareils qui fonctionnent sur des réseaux locaux, configurez un [réseau privé virtuel (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou un peering privé [ExpressRoute](https://azure.microsoft.com/services/expressroute/) dans votre réseau virtuel Azure.

### <a name="set-up-a-private-endpoint-for-iot-hub-ingress"></a>Configurer un point de terminaison privé pour l'entrée IoT Hub

Un point de terminaison privé fonctionne pour des API d’appareil IoT Hub (comme des messages appareil-à-cloud), ainsi que des API de service (comme la création et la mise à jour d’appareils).

1. Sur le Portail Azure, sélectionnez **Mise en réseau**, **Connexion des points de terminaison privés**, puis cliquez sur **+ Point de terminaison privé**.

    :::image type="content" source="media/virtual-network-support/private-link.png" alt-text="Capture d’écran montrant où ajouter un point de terminaison privé pour IoT Hub":::

1. Indiquez l’abonnement, le groupe de ressources, le nom et la région dans laquelle sera créé le nouveau point de terminaison privé. Idéalement, il doit s’agir de la même région que celle de votre hub.

1. Cliquez sur **Suivant : Ressource**, précisez l’abonnement de votre ressource IoT Hub, puis sélectionnez le type de ressource **Microsoft.Devices/IotHubs**, le nom de votre hub IoT comme **ressource** et la sous-ressource cible **iotHub**.

1. Cliquez sur **Suivant : Configuration** et indiquez le réseau virtuel et le sous-réseau où le point de terminaison privé sera créé. Sélectionnez l'option d'intégration dans zone DNS privée Azure, si vous le souhaitez.

1. Cliquez sur **Suivant : Balises**, et spécifiez si nécessaire les balises de votre ressource.

1. Cliquez sur **Vérifier + Créer** pour créer votre ressource de liaison privée.

### <a name="built-in-event-hub-compatible-endpoint"></a>Point de terminaison intégré compatible avec Event Hubs 

Le [point de terminaison intégré compatible avec Event Hubs](iot-hub-devguide-messages-read-builtin.md) est également accessible via un point de terminaison privé. Lorsque la liaison privée est configurée, vous devez voir une connexion de point de terminaison privé supplémentaire pour le point de terminaison intégré. C’est celle qui inclut `servicebus.windows.net` dans le nom de domaine complet.

:::image type="content" source="media/virtual-network-support/private-built-in-endpoint.png" alt-text="Image présentant deux points de terminaison privés en fonction de chaque liaison privée IoT Hub":::


### <a name="pricing-for-private-link"></a>Tarifs de la liaison privée

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="egress-connectivity-from-iot-hub-to-other-azure-resources"></a>Connectivité entrante d’IoT Hub vers d'autres ressources Azure

IoT Hub peut se connecter à vos ressources Stockage Blob Azure, Event Hubs et Service Bus pour le [routage des messages](./iot-hub-devguide-messages-d2c.md), le [chargement des fichiers](./iot-hub-devguide-file-upload.md) et [l'importation/exportation d’appareils en bloc](./iot-hub-bulk-identity-mgmt.md), via le point de terminaison public des ressources. Le fait de lier une ressource à un réseau virtuel bloque par défaut la connectivité à la ressource. Par conséquent, cette configuration empêche IoT Hub d’envoyer des données à vos ressources. Pour corriger ce problème, activez la connectivité entre votre ressource IoT Hub et vos ressources de compte de stockage, Event Hubs ou Service Bus avec l'option du **service Microsoft approuvé**.

### <a name="turn-on-managed-identity-for-iot-hub"></a>Activation de l’identité managée pour IoT Hub

Votre hub IoT doit disposer d’une identité managée affectée par le système pour que d’autres services puissent le trouver en tant que service Microsoft approuvé.

1. Accédez à **Identité** sur votre portail IoT Hub.

1. Sous **État**, sélectionnez **Activé**, puis cliquez sur **Enregistrer**.

    :::image type="content" source="media/virtual-network-support/managed-identity.png" alt-text="Capture d’écran montrant comment activer l’identité managée pour IoT Hub":::

### <a name="assign-managed-identity-to-your-iot-hub-at-creation-time-using-arm-template"></a>Affecter une identité gérée à votre IoT Hub au moment de la création à l’aide du modèle ARM

Pour affecter une identité gérée à votre IoT Hub au moment du provisionnement des ressources, utilisez le modèle ARM ci-dessous :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Devices/IotHubs",
      "apiVersion": "2020-03-01",
      "name": "<provide-a-valid-resource-name>",
      "location": "<any-of-supported-regions>",
      "identity": {
        "type": "SystemAssigned"
      },
      "sku": {
        "name": "<your-hubs-SKU-name>",
        "tier": "<your-hubs-SKU-tier>",
        "capacity": 1
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-02-01",
      "name": "updateIotHubWithKeyEncryptionKey",
      "dependsOn": [
        "<provide-a-valid-resource-name>"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "0.9.0.0",
          "resources": [
            {
              "type": "Microsoft.Devices/IotHubs",
              "apiVersion": "2020-03-01",
              "name": "<provide-a-valid-resource-name>",
              "location": "<any-of-supported-regions>",
              "identity": {
                "type": "SystemAssigned"
              },
              "sku": {
                "name": "<your-hubs-SKU-name>",
                "tier": "<your-hubs-SKU-tier>",
                "capacity": 1
              }
            }
          ]
        }
      }
    }
  ]
}
```

Après avoir substitué les valeurs de votre ressource `name`, `location`, `SKU.name` et `SKU.tier`, vous pouvez utiliser l’interface de ligne de commande Azure pour déployer la ressource dans un groupe de ressources existant en utilisant :

```azurecli-interactive
az deployment group create --name <deployment-name> --resource-group <resource-group-name> --template-file <template-file.json>
```

Une fois la ressource créée, vous pouvez récupérer l'identité de service géré attribuée à votre hub en utilisant l’interface de ligne de commande Azure :

```azurecli-interactive
az resource show --resource-type Microsoft.Devices/IotHubs --name <iot-hub-resource-name> --resource-group <resource-group-name>
```

### <a name="pricing-for-managed-identity"></a>Tarifs de l’identité managée

La fonctionnalité d'exception des services internes Microsoft approuvés est gratuite. Les frais pour les comptes de stockage provisionnés, les hubs d'événements ou les ressources Service Bus s'appliquent séparément.

### <a name="egress-connectivity-to-storage-account-endpoints-for-routing"></a>Connectivité sortante vers les points de terminaison de stockage pour le routage

IoT Hub peut router les messages vers un compte de stockage détenu par le client. Pour permettre à la fonctionnalité de routage d'accéder à un compte de stockage alors que des restrictions de pare-feu sont en place, votre hub IoT doit disposer d’une [identité managée](#turn-on-managed-identity-for-iot-hub). Une fois qu’une identité managée est configurée, suivez la procédure ci-dessous pour donner à l’identité de ressource de votre hub l’autorisation Azure RBAC d’accéder à votre compte de stockage.

1. Dans le portail Azure, accédez à l'onglet **Contrôle d'accès (IAM)** de votre compte de stockage, puis cliquez sur **Ajouter** dans la section **Ajouter une attribution de rôle**.

2. Sélectionnez **Contributeur aux données Blob du stockage** ([et *non* Contributeur ou Contributeur de compte de stockage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) comme **rôle**, **Utilisateur, groupe ou principal du service Azure AD** comme **Attribution de l’accès à**, puis sélectionnez le nom de la ressource de votre hub IoT dans la liste déroulante. Cliquez sur le bouton **Enregistrer** .

3. Accédez à l'onglet **Pare-feux et réseaux virtuels** de votre compte de stockage, puis activez l'option **Autoriser l'accès à partir des réseaux sélectionnés**. Sous la liste **Exceptions**, cochez la case **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage**. Cliquez sur le bouton **Enregistrer** .

4. Sur la page des ressources de votre hub IoT, accédez à l'onglet **Routage de messages**.

5. Accédez à la section **Points de terminaison personnalisés** et cliquez sur **Ajouter**. Sélectionnez **Stockage** comme type de point de terminaison.

6. Sur la page qui s'affiche, nommez votre point de terminaison, sélectionnez le conteneur que vous souhaitez utiliser dans votre stockage d’objets blob, indiquez l'encodage et le format du nom de fichier. Sélectionnez **Basée sur l'identité** comme **type d'authentification** de votre point de terminaison de stockage. Cliquez sur le bouton **Créer**.

Votre point de terminaison de stockage personnalisé est maintenant configuré pour utiliser l'identité attribuée par le système à votre hub, et il est autorisé à accéder à votre ressource de stockage malgré les restrictions de son pare-feu. Vous pouvez maintenant utiliser ce point de terminaison pour définir une règle de routage.

### <a name="egress-connectivity-to-event-hubs-endpoints-for-routing"></a>Connectivité sortante vers les points de terminaison de hubs d’événements pour le routage

IoT Hub peut être configuré pour router les messages vers un espace de noms de hubs d’événements appartenant au client. Pour permettre à la fonctionnalité de routage d'accéder à une ressource Event Hubs alors que des restrictions de pare-feu sont en place, votre hub IoT doit disposer d’une identité managée. Une fois qu’une identité managée est créée, suivez la procédure ci-dessous pour donner à l’identité de ressource de votre hub l’autorisation Azure RBAC d’accéder à vos Event Hubs.

1. Dans le portail Azure, accédez à l'onglet **Contrôle d'accès (IAM)** de vos hubs d’événements, puis cliquez sur **Ajouter** dans la section **Ajouter une attribution de rôle**.

2. Sélectionnez **Expéditeur de données Azure Event Hubs** comme **rôle**, **Utilisateur, groupe ou principal du service Azure AD** comme option **Attribution de l’accès aux applications**, puis sélectionnez le nom de la ressource de votre hub IoT dans la liste déroulante. Cliquez sur le bouton **Enregistrer** .

3. Accédez à l'onglet **Pare-feux et réseaux virtuels** de vos hubs d’événements, puis activez l'option **Autoriser l'accès à partir des réseaux sélectionnés**. Sous la liste **Exceptions**, cochez la case **Autoriser les services Microsoft approuvés à accéder aux hubs d’événements**. Cliquez sur le bouton **Enregistrer** .

4. Sur la page des ressources de votre hub IoT, accédez à l'onglet **Routage de messages**.

5. Accédez à la section **Points de terminaison personnalisés** et cliquez sur **Ajouter**. Sélectionnez **Hubs d’événements** comme type de point de terminaison.

6. Sur la page qui s’affiche, entrez un nom pour votre point de terminaison, et sélectionnez l’espace de noms et l’instance de vos hubs d’événements. Sélectionnez **Basée sur l’identité** comme **Type d’authentification**, puis cliquez sur le bouton **Créer**.

Votre point de terminaison de hubs d’événements personnalisé est maintenant configuré pour utiliser l'identité attribuée par le système à votre hub, et il est autorisé à accéder à votre ressource de hubs d’événements malgré les restrictions de son pare-feu. Vous pouvez maintenant utiliser ce point de terminaison pour définir une règle de routage.

### <a name="egress-connectivity-to-service-bus-endpoints-for-routing"></a>Connectivité sortante vers les points de terminaison Service Bus pour le routage

IoT Hub peut être configuré pour router les messages vers un espace de noms Service Bus appartenant au client. Pour permettre à la fonctionnalité de routage d'accéder à une ressource Service Bus alors que des restrictions de pare-feu sont en place, votre hub IoT doit disposer d’une identité managée. Une fois qu’une identité managée est configurée, suivez la procédure ci-dessous pour donner à l’identité de ressource de votre hub l’autorisation Azure RBAC d’accéder à votre bus de services.

1. Dans le portail Azure, accédez à l'onglet **Contrôle d'accès (IAM)** de Service Bus, puis cliquez sur **Ajouter** dans la section **Ajouter une attribution de rôle**.

2. Sélectionnez **Expéditeur de données Azure Service Bus** comme **rôle**, **Utilisateur, groupe ou principal du service Azure AD** comme option **Attribution de l’accès aux applications**, puis sélectionnez le nom de la ressource de votre hub IoT dans la liste déroulante. Cliquez sur le bouton **Enregistrer** .

3. Accédez à l'onglet **Pare-feux et réseaux virtuels** de Service Bus, puis activez l'option **Autoriser l'accès à partir des réseaux sélectionnés**. Sous la liste **Exceptions**, cochez la case **Autoriser les services Microsoft approuvés à accéder à Service Bus**. Cliquez sur le bouton **Enregistrer** .

4. Sur la page des ressources de votre hub IoT, accédez à l'onglet **Routage de messages**.

5. Accédez à la section **Points de terminaison personnalisés** et cliquez sur **Ajouter**. Sélectionnez **File d'attente Service Bus** ou **Rubrique Service Bus** (selon le cas) comme type de point de terminaison.

6. Sur la page qui s'affiche, nommez votre point de terminaison, sélectionnez l'espace de noms ainsi que la file d'attente ou la rubrique Service Bus (selon le cas). Sélectionnez **Basée sur l’identité** comme **Type d’authentification**, puis cliquez sur le bouton **Créer**.

Votre point de terminaison Service Bus personnalisé est maintenant configuré pour utiliser l'identité attribuée par le système à votre hub, et il est autorisé à accéder à votre ressource Service Bus malgré les restrictions de son pare-feu. Vous pouvez maintenant utiliser ce point de terminaison pour définir une règle de routage.

### <a name="egress-connectivity-to-storage-accounts-for-file-upload"></a>Connectivité sortante vers les comptes de stockage pour le chargement de fichiers

La fonction de chargement de fichiers d’IoT Hub permet aux appareils de charger des fichiers sur un compte de stockage appartenant au client. Pour que le chargement de fichiers puisse fonctionner, les deux appareils et IoT Hub doivent être connectés au compte de stockage. Si des restrictions de pare-feu sont en place sur le compte de stockage, vos appareils doivent utiliser l'un des mécanismes du compte de stockage pris en charge (notamment les [points de terminaison privés](../private-link/tutorial-private-endpoint-storage-portal.md), les [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) ou la [configuration directe du pare-feu](../storage/common/storage-network-security.md)) pour obtenir une connectivité. De même, si des restrictions de pare-feu sont en place sur le compte de stockage, IoT Hub doit être configuré pour accéder à la ressource de stockage via l'exception des services Microsoft approuvés. À cet effet, votre hub IoT doit disposer d’une identité managée. Une fois qu’une identité managée est configurée, suivez la procédure ci-dessous pour donner à l’identité de ressource de votre hub l’autorisation Azure RBAC d’accéder à votre compte de stockage.

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

1. Dans le portail Azure, accédez à l'onglet **Contrôle d'accès (IAM)** de votre compte de stockage, puis cliquez sur **Ajouter** dans la section **Ajouter une attribution de rôle**.

2. Sélectionnez **Contributeur aux données Blob du stockage** ([et *non* Contributeur ou Contributeur de compte de stockage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) comme **rôle**, **Utilisateur, groupe ou principal du service Azure AD** comme **Attribution de l’accès à**, puis sélectionnez le nom de la ressource de votre hub IoT dans la liste déroulante. Cliquez sur le bouton **Enregistrer** .

3. Accédez à l'onglet **Pare-feux et réseaux virtuels** de votre compte de stockage, puis activez l'option **Autoriser l'accès à partir des réseaux sélectionnés**. Sous la liste **Exceptions**, cochez la case **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage**. Cliquez sur le bouton **Enregistrer** .

4. Sur la page des ressources de votre hub IoT, accédez à l'onglet **Chargement de fichiers**.

5. Sur la page qui s'affiche, sélectionnez le conteneur que vous comptez utiliser dans votre Stockage Blob, puis configurez les options **Paramètres de notification de fichier**, **Durée de vie SAS**, **Durée de vie par défaut** et **Nombre maximal de distributions** comme vous le souhaitez. Sélectionnez **Basée sur l'identité** comme **type d'authentification** de votre point de terminaison de stockage. Cliquez sur le bouton **Créer**. Si vous recevez une erreur à cette étape, configurez temporairement votre compte de stockage afin d’autoriser l’accès à partir de **tous les réseaux**, puis réessayez. Vous pouvez configurer le pare-feu sur le compte de stockage une fois la configuration du chargement de fichiers terminée.

Votre point de terminaison de stockage pour chargement de fichiers est maintenant configuré pour utiliser l'identité attribuée par le système à votre hub, et il est autorisé à accéder à votre ressource de stockage malgré les restrictions de son pare-feu.

### <a name="egress-connectivity-to-storage-accounts-for-bulk-device-importexport"></a>Connectivité sortante vers les comptes de stockage pour l’importation/exportation d’appareils en bloc

IoT Hub prend en charge la fonctionnalité permettant d'[importer/exporter](./iot-hub-bulk-identity-mgmt.md) en bloc les informations des appareils depuis/vers un stockage d’objets blob fourni par le client. Pour que la fonctionnalité d’importation/exportation en bloc puisse fonctionner, les deux appareils et IoT Hub doivent être connectés au compte de stockage.

Cette fonctionnalité nécessite une connectivité entre IoT Hub et le compte de stockage. Pour accéder à une ressource Service Bus alors que des restrictions de pare-feu sont en place, votre hub IoT doit disposer d’une identité managée. Une fois qu’une identité managée est configurée, suivez la procédure ci-dessous pour donner à l’identité de ressource de votre hub l’autorisation Azure RBAC d’accéder à votre bus de services.

1. Dans le portail Azure, accédez à l'onglet **Contrôle d'accès (IAM)** de votre compte de stockage, puis cliquez sur **Ajouter** dans la section **Ajouter une attribution de rôle**.

2. Sélectionnez **Contributeur aux données Blob du stockage** ([et *non* Contributeur ou Contributeur de compte de stockage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) comme **rôle**, **Utilisateur, groupe ou principal du service Azure AD** comme **Attribution de l’accès à**, puis sélectionnez le nom de la ressource de votre hub IoT dans la liste déroulante. Cliquez sur le bouton **Enregistrer** .

3. Accédez à l'onglet **Pare-feux et réseaux virtuels** de votre compte de stockage, puis activez l'option **Autoriser l'accès à partir des réseaux sélectionnés**. Sous la liste **Exceptions**, cochez la case **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage**. Cliquez sur le bouton **Enregistrer** .

Il est maintenant possible d’utiliser les API REST Azure IoT pour [créer des travaux d'importation/exportation](/rest/api/iothub/service/jobs/getimportexportjobs) et obtenir des informations sur l’utilisation de la fonctionnalité d'importation/exportation en bloc. Vous devrez indiquer `storageAuthenticationType="identityBased"` dans le corps de votre demande et utiliser respectivement `inputBlobContainerUri="https://..."` et `outputBlobContainerUri="https://..."` comme URL d'entrée et de sortie de votre compte de stockage.

Les kits SDK IoT Hub Azure prennent également en charge cette fonctionnalité dans le gestionnaire de registre du client du service. L’extrait de code suivant indique comment créer une tâche d’importation ou d’exportation en utilisant le SDK C#.

```csharp
// Call an import job on the IoT Hub
JobProperties importJob = 
await registryManager.ImportDevicesAsync(
  JobProperties.CreateForImportJob(inputBlobContainerUri, outputBlobContainerUri, null, StorageAuthenticationType.IdentityBased), 
  cancellationToken);

// Call an export job on the IoT Hub to retrieve all devices
JobProperties exportJob = 
await registryManager.ExportDevicesAsync(
    JobProperties.CreateForExportJob(outputBlobContainerUri, true, null, StorageAuthenticationType.IdentityBased),
    cancellationToken);
```

Pour utiliser cette version des kits SDK Azure IoT avec prise en charge des réseaux virtuels pour C#, Java et Node.js :

1. Créez une variable d'environnement nommée `EnableStorageIdentity` et définissez sa valeur sur `1`.

2. Téléchargez le SDK :  [Java](https://aka.ms/vnetjavasdk) | [C#](https://aka.ms/vnetcsharpsdk) | [Node.js](https://aka.ms/vnetnodesdk)
 
Pour Python, téléchargez notre version limitée à partir de GitHub.

1. Accédez à la [page de mise en production GitHub](https://aka.ms/vnetpythonsdk).

2. Téléchargez le fichier suivant, que vous trouverez au bas de la page de mise en production sous l'en-tête **assets**.
    > *azure_iot_hub-2.2.0_limited-py2.py3-none-any.whl*

3. Ouvrez un terminal et accédez au dossier contenant le fichier téléchargé.

4. Exécutez la commande suivante pour installer le SDK du service Python avec prise en charge des réseaux virtuels :
    > pip install ./azure_iot_hub-2.2.0_limited-py2.py3-none-any.whl


## <a name="next-steps"></a>Étapes suivantes

Utilisez les liens ci-dessous pour en savoir plus sur les fonctionnalités IoT Hub :

* [Routage de messages](./iot-hub-devguide-messages-d2c.md)
* [Chargement de fichiers](./iot-hub-devguide-file-upload.md)
* [Importation/exportation d’appareils en bloc](./iot-hub-bulk-identity-mgmt.md)
