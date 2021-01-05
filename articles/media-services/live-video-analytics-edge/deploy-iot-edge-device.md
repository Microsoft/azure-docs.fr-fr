---
title: Déployer Live Video Analytics sur un appareil IoT Edge - Azure
description: Cet article répertorie les étapes permettant de déployer Live Video Analytics sur votre appareil IoT Edge. C’est utile si, par exemple, vous disposez d’un accès à un ordinateur local Linux et que vous avez déjà crée un compte Azure Media Services.
ms.topic: how-to
ms.date: 09/09/2020
ms.openlocfilehash: 4fa4a9643976ba513b025706cacec26b2a50afb5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498317"
---
# <a name="deploy-live-video-analytics-on-an-iot-edge-device"></a>Déployer Live Video Analytics sur un appareil IoT Edge

Cet article répertorie les étapes permettant de déployer Live Video Analytics sur votre appareil IoT Edge. C’est utile si, par exemple, vous disposez d’un accès à un ordinateur local Linux et que vous avez déjà crée un compte Azure Media Services.

> [!NOTE]
> La prise en charge des appareils ARM64 est disponible dans Live Video Analytics sur les builds de IoT Edge `1.0.4` et versions ultérieures.
> La prise en charge de l’exécution du runtime Azure IoT Edge sur des appareils ARM64 est incluse dans la [version préliminaire publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

* Un appareil x86-64 ou ARM64 exécutant l’un des [systèmes d’exploitation Linux pris en charge](../../iot-edge/support.md#operating-systems)
* Abonnement Azure dans lequel vous disposez des [privilèges de propriétaire](../../role-based-access-control/built-in-roles.md#owner)
* [Créer et configurer IoT Hub](../../iot-hub/iot-hub-create-through-portal.md)
* [Enregistrer un appareil IoT Edge](../../iot-edge/how-to-manual-provision-symmetric-key.md)
* [Installer le runtime Azure IoT Edge sur des systèmes Linux Debian](../../iot-edge/how-to-install-iot-edge.md)
* [Créer un compte Azure Media Services](../latest/create-account-howto.md)

    * Utilisez l’une de ces régions : USA Est 2, USA Est, USA Centre, USA Centre Nord, Japon Est, USA Ouest, USA Ouest 2, USA Centre-Ouest, Canada Est, Royaume-Uni Sud, France Centre, France Sud, Suisse Nord, Suisse Ouest et Japon Ouest.
    * Nous vous recommandons d’utiliser des comptes de stockage universels v2 (GPv2).

## <a name="configuring-azure-resources-for-using-live-video-analytics"></a>Configuration des ressources Azure pour utiliser Live Video Analytics

### <a name="create-custom-azure-resource-manager-role"></a>Créer un rôle Azure Resource Manager personnalisé

Voir [Créer un rôle Azure Resource Manager personnalisé](create-custom-azure-resource-manager-role-how-to.md) et l’attribuer à un principal de service pour utiliser Live Video Analytics.

### <a name="set-up-a-premium-streaming-endpoint"></a>Configurer un point de terminaison de streaming Premium

Si vous prévoyez d’utiliser Live Video Analytics pour enregistrer des vidéos en continu dans le cloud, puis de recourir aux [API de requête](playback-recordings-how-to.md#query-api) avant de les rejouer, nous vous recommandons de mettre à niveau Media Service vers un [point de terminaison de streaming Premium](../latest/streaming-endpoint-concept.md#types).  

Il s’agit d’une étape facultative. Pour ce faire, vous pouvez utiliser cette commande Azure CLI :

```azurecli
az ams streaming-endpoint scale --resource-group $RESOURCE_GROUP --account-name $AMS_ACCOUNT -n default --scale-units 1
```

Utilisez cette commande pour commencer le point de terminaison de streaming 

> [!IMPORTANT]
> Votre abonnement vous sera facturé à partir de ce moment.

```azurecli
az ams streaming-endpoint start --resource-group $RESOURCE_GROUP --account-name $AMS_ACCOUNT -n default --no-wait
```

Pour obtenir les informations d'identification et accéder aux API Media Service, suivez les étapes décrites dans cet article : [Accéder aux API Media Service](../latest/access-api-howto.md?tabs=portal). Puis sélectionnez l'onglet Portail.

## <a name="create-and-use-local-user-account-for-deployment"></a>Créer et utiliser un compte d’utilisateur local pour le déploiement
Afin d’exécuter le module Live Video Analytics sur IoT Edge, créez un compte d’utilisateur local doté du moins de privilèges possible. Par exemple, exécutez les commandes suivantes sur votre ordinateur Linux :

```
sudo groupadd -g 1010 localuser
sudo adduser --home /home/edgeuser --uid 1010 -gid 1010 edgeuser
```

## <a name="granting-permissions-to-device-storage"></a>Octroi d’autorisations au stockage de l’appareil

Maintenant que vous avez créé un compte d’utilisateur local, 

* vous aurez besoin d’un dossier local pour stocker les données de configuration de l’application. Créez un dossier et accordez des autorisations d’écriture au compte d’utilisateur local dans ce dossier à l’aide des commandes suivantes :

```
sudo mkdir /var/lib/azuremediaservices
sudo chown -R edgeuser /var/lib/azuremediaservices
```

* Vous aurez également besoin d’un dossier pour [enregistrer des vidéos dans un fichier local](event-based-video-recording-concept.md#video-recording-based-on-events-from-other-sources). Utilisez les commandes suivantes pour créer un dossier local pour le même

```
sudo mkdir /var/media
sudo chown -R edgeuser /var/media
```

## <a name="deploy-live-video-analytics-edge-module"></a>module Edge Deploy Live Video Analytics

Live Video Analytics sur IoT Edge expose les propriétés du jumeau de module qui sont documentées dans [schéma de configuration du jumeau de module](module-twin-configuration-schema.md). 

### <a name="deploy-using-the-azure-portal"></a>Déployer à l’aide du portail Azure

Le Portail Azure vous aide à créer un manifeste de déploiement et à étendre le déploiement à un appareil IoT Edge.  
#### <a name="select-your-device-and-set-modules"></a>Sélectionner votre appareil et définir des modules

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com/) et accédez à votre IoT Hub.
1. Sélectionnez **IoT Edge** dans le menu.
1. Cliquez sur l’ID de l’appareil cible dans la liste des appareils.
1. Sélectionnez **Définir modules**.

#### <a name="configure-a-deployment-manifest"></a>Configurer un manifeste de déploiement

Un manifeste de déploiement est un document JSON qui décrit les modules à déployer, le flux des données entre les modules et les propriétés souhaitées des jumeaux de module. Le portail Azure comprend un Assistant qui vous guide lors de la création du manifeste de déploiement. Le processus comprend trois étapes organisées en onglets : **Modules**, **Itinéraires** et **Vérifier + créer**.

#### <a name="add-modules"></a>Ajouter des modules

1. Dans la section **Modules IoT Edge** de la page, cliquez sur la liste déroulante **Ajouter**, puis sélectionnez **Module IoT Edge** pour afficher la page **Ajouter un module IoT Edge**.
1. Sous l’onglet **Paramètres du module**, fournissez un nom pour le module, puis spécifiez l’URI de l’image conteneur :   
    Exemples :
    
    * **Nom du module IoT Edge** : IvaEdge
    * **URI de l’image** : mcr.microsoft.com/media/live-video-analytics:1.0    
    
    ![La capture d’écran montre l’onglet Paramètres du module.](./media/deploy-iot-edge-device/add.png)
    
    > [!TIP]
    > Ne sélectionnez pas **Ajouter** tant que vous n’avez pas spécifié de valeur sous les onglets **Paramètres du module**, **Options de création de conteneur** et **Paramètres du jumeau de module**, comme décrit dans cette procédure.
    
    > [!WARNING]
    > Azure IoT Edge respecte la casse lorsque vous effectuez des appels de modules. Notez la chaîne exacte que vous utilisez comme nom de module.

1. Ouvrez l’onglet **variables d’environnement**.
   
   Ajoutez les valeurs suivantes dans les champs de saisie qui s’affichent. ![Environment Variables](./media/deploy-iot-edge-device/environment-variables.png) 

1. Ouvrez l’onglet **Options de création de conteneur**.

    ![Options de création de conteneur](./media/deploy-iot-edge-device/container-create-options.png)
 
    Copiez et collez le JSON suivant dans la zone pour limiter la taille des fichiers journaux générés par le module.
    
    ```    
    {
        "HostConfig": {
            "LogConfig": {
                "Type": "",
                "Config": {
                    "max-size": "10m",
                    "max-file": "10"
                }
            },
            "Binds": [
               "/var/lib/azuremediaservices:/var/lib/azuremediaservices",
               "/var/media:/var/media"
            ]
        }
    }
    ````
   
   La section « Lie » du code JSON comporte 2 entrées :
   1. « /var/lib/azuremediaservices:/var/lib/azuremediaservices » : Celle-là permet de lier les données de configuration d’application persistantes du conteneur et de les stocker sur le périphérique.
   1. « /var/media:/var/media » : Celle-ci lie les dossiers multimédias entre le périphérique et le conteneur. Elle est utilisée pour stocker les enregistrements vidéo lorsque vous exécutez une topologie de graphique média qui prend en charge le stockage des clips vidéo sur le périphérique.
   
1. Sous l’onglet **Paramètres de jumeau de module**, copiez le code JSON suivant et collez-le dans la zone.
 
    ![Paramètres de jumeau](./media/deploy-iot-edge-device/twin-settings.png)

    Live Video Analytics sur IoT Edge nécessite un ensemble de propriétés de jumeau obligatoires pour pouvoir être exécuté, comme indiqué dans [Schéma de configuration du jumeau de module](module-twin-configuration-schema.md).  

    Le JSON que vous devez entrer dans la zone d’édition des paramètres du jumeau de module se présente comme suit :    
    ```
    {
        "applicationDataDirectory": "/var/lib/azuremediaservices",
        "azureMediaServicesArmId": "/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/microsoft.media/mediaservices/{AMS-account-name}",
        "aadTenantId": "{the-ID-of-your-tenant}",
        "aadServicePrincipalAppId": "{the-ID-of-the-service-principal-app-for-ams-account}",
        "aadServicePrincipalSecret": "{secret}"
    }
    ```
    Ce sont des propriétés **requises** et à propos du JSON ci-dessus,  
    * {subscriptionID} : il s’agit de votre ID d’abonnement Azure
    * {resourceGroupName} : il s’agit du groupe de ressources auquel appartient votre compte Media Service
    * {AMS-Account-Name} : il s’agit du nom de votre compte Media Services
    
    Pour obtenir les autres valeurs, consultez [Accéder à l'API Azure Media Services](../latest/access-api-howto.md?tabs=portal) et sélectionnez l'onglet Portail.  
    * aadTenantId : l’ID de votre locataire qui est identique au « AadTenantId » du lien ci-dessus.
    * aadServicePrincipalAppId : l’ID de l’application du principal de service pour votre compte Media Service qui est identique au « AadClientId » du lien ci-dessus.
    * aadServicePrincipalSecret : mot de passe du principal du service ; identique au « AadSecret » du lien ci-dessus.

    Ci-dessous, vous trouverez quelques propriétés **recommandées** supplémentaires pouvant être ajoutées au JSON et qui vous aideront à surveiller le module. Pour plus d’informations, consultez [surveillance et enregistrement](monitoring-logging.md) :
    
    ```
    "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
    "OperationalEventsOutputName": "lvaEdgeOperational",
    "logLevel": "Information",
    "logCategories": "Application,Events"
    ```
    
    Voici quelques propriétés **facultatives** que vous pouvez ajouter dans le JSON :
    
    ```
    "aadEndpoint": "https://login.microsoftonline.com",
    "aadResourceId": "https://management.core.windows.net/",
    "armEndpoint": "https://management.azure.com/",
    "allowUnsecuredEndpoints": true
    ```

   > [!Note]
   > La propriété de jumeau **allowUnsecuredEndpoints** est définie sur true dans le cadre des tutoriels et démarrages rapides.   
   Vous devez définir cette propriété sur **false** lors de l’exécution dans l’environnement de production. Cela permet de s’assurer que l’application bloque tous les points de terminaison non sécurisés. De plus, pour exécuter les topologies de graphique, des informations d’identification de connexion valides sont nécessaires.  
   
    Sélectionnez Ajouter pour ajouter les propriétés du jumeau de module.
1. Sélectionnez **Suivant : Itinéraires** pour passer à la section Itinéraires.
    Spécifiez des itinéraires.

Conservez les itinéraires par défaut et sélectionnez **Suivant : Vérifier + créer** pour passer à la section Vérifier.

#### <a name="review-deployment"></a>Vérifier le déploiement

La section de vérification vous montre le manifeste de déploiement JSON qui a été créé en fonction de vos sélections dans les deux sections précédentes. Il existe également deux modules déclarés que vous n’avez pas ajoutés : $edgeAgent et $edgeHub. Ils composent le runtime IoT Edge et sont les valeurs par défaut requises pour tous les déploiements.

Vérifiez les informations de votre déploiement, puis sélectionnez Créer.

### <a name="verify-your-deployment"></a>Vérifier votre déploiement

Une fois le déploiement créé, vous êtes redirigé vers la page IoT Edge de votre hub IoT.

1. Sélectionnez le périphérique IoT Edge que vous avez ciblé avec le déploiement pour accéder à ses informations détaillées.
2. Dans les détails de l’appareil, vérifiez que le module de stockage d’objets blob figure dans **Spécifié lors du déploiement et Signalé par l’appareil**.

Il peut s’écouler quelques instants avant que le module ne démarre sur l’appareil et qu’il ne soit renvoyé à IoT Hub. Actualisez la page pour afficher un état mis à jour.
Code d’état : 200 –OK signifie que [le runtime d’IoT Edge](../../iot-edge/iot-edge-runtime.md) est sain et fonctionne correctement.

![La capture d’écran affiche une valeur d’état pour un runtime IoT Edge.](./media/deploy-iot-edge-device/status.png)

#### <a name="invoke-a-direct-method"></a>Appeler une méthode directe

Testons ensuite l’exemple en appelant une méthode directe. Lisez [Méthodes directes pour Live Video Analytics sur IoT Edge](direct-methods.md) pour comprendre toutes les méthodes directes fournies par notre module IvaEdge.

1. En cliquant sur le module Edge que vous avez créé, vous êtes redirigé vers sa page de configuration.  

    ![La capture d’écran affiche la page de configuration d’un module Edge.](./media/deploy-iot-edge-device/modules.png)
1. Cliquez sur l’option de menu Méthode directe.

    > [!NOTE] 
    > Vous devrez ajouter une valeur dans les sections Chaîne de connexion, comme vous pouvez le voir sur la page actuelle. Vous n’avez pas besoin de masquer ou de modifier quoi que ce soit dans la section **Nom du paramètre**. Vous pouvez laisser ces informations publiques.

    ![Méthode directe](./media/deploy-iot-edge-device/module-details.png)
1. Ensuite, entrez « GraphTopologyList » dans la zone Nom de la méthode.
1. Puis, copiez et collez la charge utile JSON ci-dessous dans la zone Charge utile.
    
    ```
    {
        "@apiVersion" : "1.0"
    }
    ```
1. Cliquez sur l’option « Appeler la méthode » en haut de la page

    ![Méthodes directes](./media/deploy-iot-edge-device/direct-method.png)
1. Vous devez voir le message d’état 200 dans la zone Résultats

    ![Message d’état 200](./media/deploy-iot-edge-device/connection-timeout.png) 

## <a name="next-steps"></a>Étapes suivantes

Essayez [Démarrage rapide : Bien démarrer – Live Video Analytics sur IoT Edge](get-started-detect-motion-emit-events-quickstart.md#deploy-modules-on-your-edge-device)

> [!TIP]
> Si vous poursuivez avec le démarrage rapide ci-dessus, lors de l’appel des méthodes directes à l’aide de Visual Studio Code, vous utiliserez l’appareil qui a été ajouté au IoT Hub via cet article, au lieu de la `lva-sample-device` par défaut.