---
title: Tutoriel – Créer une instance IoT Edge pour l’analytique vidéo Azure IoT Central (machine virtuelle Linux)
description: Ce tutoriel explique comment créer une instance IoT Edge pour l’analytique vidéo sur une machine virtuelle Linux afin de l’utiliser avec le modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: f798e65b1517430bc67af793ebb517c586d5d58f
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763874"
---
# <a name="tutorial-create-an-iot-edge-instance-for-video-analytics-linux-vm"></a>Tutoriel : Créer une instance IoT Edge pour l’analytique vidéo (machine virtuelle Linux)

Azure IoT Edge est un service entièrement géré qui permet de bénéficier de l’intelligence du cloud localement en déployant et en exécutant les ressources suivantes :

* logique personnalisée ;
* services Azure ;
* intelligence artificielle.

Dans IoT Edge, ces services s’exécutent directement sur des appareils IoT multiplateformes, ce qui vous permet d’exécuter votre solution IoT en toute sécurité et à grande échelle, tant dans le cloud que hors connexion.

Ce tutoriel montre comment préparer un appareil IoT Edge dans une machine virtuelle Azure. L’instance IoT Edge exécute les modules d’analytique vidéo en direct qu’utilise le modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement Azure IoT Central.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * créer une machine virtuelle Azure avec le runtime Azure IoT Edge installé ;
> * préparer l’installation d’IoT Edge afin d’héberger le module d’analytique vidéo en direct et de vous connecter à IoT Central.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez avoir suivi le tutoriel précédent [Créer une application d’analytique vidéo dans Azure IoT Central](./tutorial-video-analytics-create-app-yolo-v3.md) ou [Créer une analytique vidéo dans Azure IoT Central (OpenVINO)&trade;](tutorial-video-analytics-create-app-openvino.md).

Vous avez également besoin d’un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un gratuitement dans la [page d’inscription à Azure](https://aka.ms/createazuresubscription).

## <a name="deploy-azure-iot-edge"></a>Déployer Azure IoT Edge

Pour créer une machine virtuelle Azure avec les derniers runtime IoT Edge et modules d’analytique vidéo en direct installés, sélectionnez le bouton suivant :

[![Bouton Déployer sur Azure pour iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Flive-video-analytics%2Fmaster%2Fref-apps%2Flva-edge-iot-central-gateway%2Fvm_deploy%2FedgeModuleVMDeploy.json)

Utilisez les informations du tableau suivant pour remplir le formulaire de **déploiement personnalisé** :

| Champ | Valeur |
| ----- | ----- |
| Abonnement | Sélectionnez votre abonnement Azure. |
| Resource group | *lva-rg* : groupe de ressources que vous avez créé dans le tutoriel précédent. |
| Région       | *USA Est* |
| DNS Label Prefix | Choisissez un préfixe DNS unique pour la machine virtuelle. Utilisez uniquement des caractères alphabétiques ; les chiffres et caractères spéciaux ne sont pas autorisés. |
| Nom d’utilisateur d’administrateur | *AzureUser* |
| Mot de passe d’administrateur | Entrez un mot de passe. Notez le mot de passe dans le fichier *scratchpad.txt*. Vous l’utiliserez plus tard. |
| ID d’étendue | **ID d’étendue** que vous avez noté dans le fichier *scratchpad.txt* dans le tutoriel précédent lorsque vous avez ajouté l’appareil de passerelle. |
| ID de périphérique | *gateway-001* : appareil de passerelle que vous avez créé dans le tutoriel précédent. |
| Clé de périphérique | **Clé primaire de l’appareil** que vous avez notée dans le fichier *scratchpad.txt* dans le tutoriel précédent lorsque vous avez ajouté l’appareil de passerelle. |
| Hôte d’application IoT Central | **URL de l’application** que vous avez notée dans le fichier *scratchpad.txt* dans le tutoriel précédent. Par exemple, *traders.azureiotcentral.com*. |
| Jeton d’API d’application IoT Central | **Jeton d’API d’opérateur** que vous avez noté dans le fichier *scratchpad.txt* dans le tutoriel précédent. |
| Clé d’approvisionnement d’appareil IoT Central | **Clé primaire du groupe SAS-IoT-Devices** que vous avez notée dans le fichier *scratchpad.txt* dans le tutoriel précédent. |
| Taille de la machine virtuelle | *Standard_DS1_v2* |
| Version du système d’exploitation Ubuntu | *18.04-LTS* |
| Emplacement | *[resourceGroup().location]* |

Sélectionnez **Vérifier + créer**. Une fois la validation terminée, sélectionnez **Créer**. Le déploiement prend généralement environ trois minutes. Une fois le déploiement terminé, accédez au groupe de ressources **lva-rg** dans le portail Azure.

## <a name="ensure-the-iot-edge-runtime-loads-the-modules"></a>Vérifier que le runtime IoT Edge charge les modules

Dans le portail Azure, accédez au groupe de ressources **lva-rg** et sélectionnez la machine virtuelle. Dans la section **Support + dépannage**, sélectionnez **Console série**.

Appuyez sur **Entrée** pour afficher l’invite `login:`. Tapez le nom d'utilisateur *AzureUser* et le mot de passe que vous avez choisi lors de la création de la machine virtuelle.

Exécutez la commande suivante pour vérifier la version du runtime IoT Edge. Au moment de la rédaction de cette documentation, la version était 1.0.9 :

```bash
sudo iotedge --version
```

Répertoriez vos modules IoT Edge à l’aide de la commande suivante :

```bash
sudo iotedge list
```

Le déploiement a configuré les cinq modules IoT Edge suivants pour exécution :

* LvaEdgeGatewayModule
* edgeAgent
* edgeHub
* lvaEdge
* lvaYolov3

Le déploiement a créé un environnement IoT Edge personnalisé avec les modules requis pour l’analytique vidéo en direct. Le déploiement a mis à jour le fichier **config.yaml** par défaut pour s’assurer que le runtime IoT Edge a utilisé le service d’approvisionnement d’appareil IoT pour se connecter à IoT Central. Le déploiement a également créé un fichier appelé **state.json** dans le dossier **/data/storage** pour fournir des données de configuration supplémentaires aux modules. Pour plus d’informations, consultez le tutoriel [Créer une instance IoT Edge pour l’analytique vidéo (Intel NUC)](./tutorial-video-analytics-iot-edge-nuc.md).

Pour résoudre les problèmes de l’appareil IoT Edge, consultez [Résoudre les problèmes de votre appareil IoT Edge](../../iot-edge/troubleshoot.md).

## <a name="use-the-rtsp-simulator"></a>Utiliser le simulateur RTSP

Si vous ne disposez pas de caméra réelle à connecter à votre appareil IoT Edge, vous pouvez utiliser les deux caméras simulées du modèle d’application d’analytique vidéo. Cette section vous montre comment utiliser un flux vidéo simulé dans votre appareil IoT Edge.

Ces instructions utilisent le [Serveur multimédia Live555](http://www.live555.com/mediaServer/) en tant que simulateur RTSP dans un conteneur Docker.

> [!NOTE]
> Les références à des logiciels tiers dans ce référentiel sont fournies uniquement à titre d’information et de commodité. Microsoft ne garantit pas et ne concède pas de droits sur des logiciels tiers. Pour plus d’informations, consultez [Serveur multimédia Live555](http://www.live555.com/mediaServer/).

Utilisez la commande suivante pour exécuter l’utilitaire **rtspvideo** dans un conteneur Docker sur votre machine virtuelle IoT Edge. Le conteneur Docker crée un flux RTSP en arrière-plan :

```bash
sudo docker run -d --name live555 --rm -p 554:554 mcr.microsoft.com/lva-utilities/rtspsim-live555:1.2
```

Utilisez la commande suivante pour répertorier les conteneurs Docker :

```bash
sudo docker ps
```

La liste comprend un conteneur nommé **live555**.

## <a name="next-steps"></a>Étapes suivantes

Vous avez déployé le runtime IoT Edge, les modules d’analytique vidéo en direct et le flux de simulation Live555 sur une machine virtuelle Linux exécutée sur Azure.

Pour gérer les caméras, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Surveiller et gérer une application d’analytique vidéo pour la détection d’objet et de mouvement](./tutorial-video-analytics-manage.md)