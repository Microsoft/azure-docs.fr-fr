---
title: Déployer un module de sécurité IoT Edge
description: Apprenez à déployer un agent de sécurité Defender pour IoT sur IoT Edge.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 1/30/2020
ms.author: mlottner
ms.openlocfilehash: c2b440413599ce07112231af17daa0bc14817b76
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832775"
---
# <a name="deploy-a-security-module-on-your-iot-edge-device"></a>Déployer un module de sécurité sur votre appareil IoT Edge

Le module **Defender pour IoT** constitue une solution de sécurité complète pour vos appareils IoT Edge.
Ce module de sécurité collecte, agrège et analyse des données de sécurité brutes tirées du système d’exploitation et du système de conteneur pour produire des alertes et des suggestions de sécurité actionnables.
Pour plus d’informations, consultez [Module de sécurité pour IoT Edge](security-edge-architecture.md).

Dans ce article, vous apprendrez à déployer un module de sécurité sur votre appareil IoT Edge.

## <a name="deploy-security-module"></a>Déployer un module de sécurité

Procédez comme suit pour déployer un module de sécurité Defender pour IoT sur IoT Edge.

### <a name="prerequisites"></a>Prérequis

1. Dans votre instance d’IoT Hub, assurez-vous que votre appareil est [inscrit en tant qu’appareil IoT Edge](../iot-edge/how-to-manual-provision-symmetric-key.md#register-a-new-device).

1. Le module Defender pour IoT Edge nécessite l'installation du [framework AuditD](https://linux.die.net/man/8/auditd) sur l'appareil IoT Edge.

    - Installez le framework en exécutant la commande suivante sur votre appareil IoT Edge :

    `sudo apt-get install auditd audispd-plugins`

    - Exécutez la commande suivante pour vérifier qu’AuditD est actif :

    `sudo systemctl status auditd`<br>
    - La réponse attendue est : `active (running)`

### <a name="deployment-using-azure-portal"></a>Déploiement à l’aide du Portail Microsoft Azure

1. À partir du portail Microsoft Azure, ouvrez la **Place de marché**.

1. Sélectionnez **Internet des objets**, puis recherchez **Defender pour IoT** et sélectionnez-le.

   :::image type="content" source="media/howto/edge-onboarding-8.png" alt-text="Sélectionnez Defender pour IoT.":::

1. Cliquez sur **Créer** pour configurer le déploiement.

1. Choisissez l’**abonnement** Azure de votre instance d’IoT Hub, puis sélectionnez votre instance d’**IoT Hub**.<br>Sélectionnez **Déployer sur un appareil** pour cibler un appareil unique ou **Déployer à l’échelle** pour cibler plusieurs appareils, puis cliquez sur **Créer**. Pour plus d’informations sur le déploiement à l’échelle, consultez [Comment déployer](../iot-edge/how-to-deploy-at-scale.md).

    >[!Note]
    >Si vous avez sélectionné **Déployer à l’échelle**, ajoutez le nom de l’appareil et d’autres informations avant d’accéder à l’onglet **Ajouter des modules** dans les instructions suivantes.

Suivez les différentes étapes pour procéder au déploiement IoT Edge de Defender pour IoT.

#### <a name="step-1-modules"></a>Étape 1 : Modules

1. Sélectionnez le module **AzureSecurityCenterforIoT**.
1. Sous l’onglet **Paramètres du module**, remplacez le **nom** par **azureiotsecurity**.
1. Sous l’onglet **Variables d’environnement**, ajoutez une variable si nécessaire (par exemple niveau de débogage).
1. Sous l’onglet **Options de création de conteneur**, ajoutez la configuration suivante :

    ``` json
    {
        "NetworkingConfig": {
            "EndpointsConfig": {
                "host": {}
            }
        },
        "HostConfig": {
            "Privileged": true,
            "NetworkMode": "host",
            "PidMode": "host",
            "Binds": [
                "/:/host"
            ]
        }
    }
    ```

1. Sous l’onglet **Paramètres de jumeau de module**, ajoutez la configuration suivante :

   Propriété de jumeau de module :
   
   ``` json
     "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration"
   ```

   Contenu de la propriété de jumeau de module : 

   ```json
     {

     }
   ```
    
   Pour plus d’informations sur la configuration de l’agent, consultez [Configurer des agents de sécurité](./how-to-agent-configuration.md).

1. Sélectionnez **Update**.

#### <a name="step-2-runtime-settings"></a>Étape 2 : Paramètres du runtime

1. Sélectionnez **Paramètres du runtime**.
1. Sous **Edge Hub**, remplacez **Image** par **mcr.microsoft.com/azureiotedge-hub:1.0.8.3**.
1. Vérifiez que **Options de création** est défini sur la configuration suivante :

    ``` json
    {
       "HostConfig":{
          "PortBindings":{
             "8883/tcp":[
                {
                   "HostPort":"8883"
                }
             ],
             "443/tcp":[
                {
                   "HostPort":"443"
                }
             ],
             "5671/tcp":[
                {
                   "HostPort":"5671"
                }
             ]
          }
       }
    }
    ```

1. Sélectionnez **Enregistrer**.

1. Sélectionnez **Suivant**.

#### <a name="step-3-specify-routes"></a>Étape 3 : Spécifier des routes

1. Sous l’onglet **Spécifier des routes**, vérifiez que vous avez une route (explicite ou implicite) qui va transférer les messages depuis le module **azureiotsecurity** vers **$upstream** conformément aux exemples suivants. Quand la route est en place, sélectionnez **Suivant**.

   Exemples de routes :

    ```Default implicit route
    "route": "FROM /messages/* INTO $upstream"
    ```

    ```Explicit route
    "ASCForIoTRoute": "FROM /messages/modules/azureiotsecurity/* INTO $upstream"
    ```

1. Sélectionnez **Suivant**.

#### <a name="step-4-review-deployment"></a>Étape 4 : Vérifier le déploiement

- Sous l’onglet **Passer en revue le déploiement**, examinez les informations de votre déploiement, puis sélectionnez **Créer** pour procéder au déploiement.

## <a name="diagnostic-steps"></a>Étapes de diagnostic

Si vous rencontrez un problème, les journaux de conteneur sont la meilleure façon d’en savoir plus sur l’état d’un appareil de module de sécurité IoT Edge. Utilisez les commandes et les outils de cette section pour recueillir des informations.

### <a name="verify-the-required-containers-are-installed-and-functioning-as-expected"></a>Vérifiez que les conteneurs requis sont installés et qu’ils fonctionnent comme prévu

1. Exécutez les commandes suivantes sur votre appareil IoT Edge :

    `sudo docker ps`

1. Vérifiez que les conteneurs suivants sont en cours d’exécution :

   | Nom | IMAGE |
   | --- | --- |
   | azureiotsecurity | mcr.microsoft.com/ascforiot/azureiotsecurity:1.0.2 |
   | edgeHub | mcr.microsoft.com/azureiotedge-hub:1.0.8.3 |
   | edgeAgent | mcr.microsoft.com/azureiotedge-agent:1.0.1 |

   Si les conteneurs minimum requis ne sont pas présents, vérifiez si votre manifeste de déploiement IoT Edge correspond aux paramètres recommandés. Pour plus d’informations, consultez [Déployer votre module IoT Edge](#deployment-using-azure-portal).

### <a name="inspect-the-module-logs-for-errors"></a>Recherchez d’éventuelles erreurs dans journaux de module

1. Exécutez les commandes suivantes sur votre appareil IoT Edge :

   `sudo docker logs azureiotsecurity`

1. Pour consulter des journaux d’activité plus détaillés, ajoutez la variable d’environnement suivante au déploiement du module **azureiotsecurity** : `logLevel=Debug`.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les options de configuration, accédez au guide pratique de configuration du module.
> [!div class="nextstepaction"]
> [Guide pratique de configuration du module](./how-to-agent-configuration.md)