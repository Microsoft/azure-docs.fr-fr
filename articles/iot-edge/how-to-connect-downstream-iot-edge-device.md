---
title: Connexion d’appareils IoT Edge en aval – Azure IoT Edge | Microsoft Docs
description: Guide pratique pour configurer un appareil IoT Edge en aval de sorte qu’il se connecte à des appareils de passerelle Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 37c237cdaf6c0d4f766d4b2e39c10e3e96215463
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187831"
---
# <a name="connect-a-downstream-iot-edge-device-to-an-azure-iot-edge-gateway-preview"></a>Connexion d’un appareil IoT Edge en aval à une passerelle Azure IoT Edge (préversion)

Cet article explique comment établir une connexion approuvée entre une passerelle IoT Edge et un appareil IoT Edge en aval.

>[!NOTE]
>Les conteneurs Linux doivent posséder la version 1.2 de IoT Edge, en préversion publique.

Dans un scénario de type passerelle, un appareil IoT Edge peut être à la fois une passerelle et un appareil en aval. Il est possible de superposer plusieurs passerelles IoT Edge pour créer une hiérarchie d’appareils. Les appareils en aval (ou enfants) peuvent s’authentifier et envoyer ou recevoir des messages par le biais de leur appareil de passerelle (ou parent).

Il existe deux configurations différentes pour les appareils IoT Edge d’une hiérarchie de passerelle. Cet article aborde les deux. La première est l’appareil IoT Edge de la **couche supérieure**. Lorsque plusieurs appareils IoT Edge sont connectés les uns aux autres, un appareil dépourvu d’appareil parent qui se connecte directement à IoT Hub est considéré comme situé dans la couche supérieure. Il est chargé de gérer les demandes de tous les appareils qui se trouvent en aval. L’autre configuration s’applique à tous les appareils IoT Edge des **couches inférieures** de la hiérarchie. Ils peuvent servir de passerelle pour d’autres appareils IoT et IoT Edge en aval, mais ils doivent également acheminer les communications via leurs propres appareils parents.

Certaines architectures réseau exigent que seul le premier appareil IoT Edge d’une hiérarchie puisse se connecter au cloud. Dans cette configuration, les appareils IoT Edge des couches inférieures de la hiérarchie ne peuvent communiquer qu’avec leur appareil de passerelle (ou parent) et tous les appareils en aval (ou enfants).

Toutes les étapes de cet article s’appuient sur la procédure [Configuration d’un appareil IoT Edge comme passerelle transparente](how-to-create-transparent-gateway.md), qui vise à configurer un appareil IoT Edge comme une passerelle pour les appareils IoT en aval. Tous les scénarios de passerelle comportent les mêmes étapes de base :

* **Authentification** : créez des identités IoT Hub pour tous les appareils de la hiérarchie de passerelle.
* **Autorisation** : configurez la relation parent/enfant dans IoT Hub de façon à autoriser les appareils enfants à se connecter à leur appareil parent de la même manière qu’à IoT Hub.
* **Détection de passerelle** : veillez à ce que l’appareil enfant puisse trouver son appareil parent sur le réseau local.
* **Connexion sécurisée** : établissez une connexion sécurisée avec des certificats approuvés faisant partie de la même chaîne.

## <a name="prerequisites"></a>Prérequis

* Un hub IoT gratuit ou standard.
* Au moins deux **appareils IoT Edge** : l’un qui deviendra l’appareil de la couche supérieure, et le ou les autres les appareils de couche inférieure. Si vous ne possédez pas d’appareils IoT Edge disponibles, vous pouvez [exécuter Azure IoT Edge sur des machines virtuelles Ubuntu](how-to-install-iot-edge-ubuntuvm.md).
* Si vous utilisez Azure CLI pour créer et gérer les appareils, veillez à installer la version 2.3.1 de l’interface, ainsi que la version 0.10.6 (ou une version ultérieure) de l’extension Azure IoT.

Cet article vous donne la procédure détaillée et les options permettant de créer une hiérarchie de passerelle adaptée à votre scénario. Pour un tutoriel guidé, consultez [Création d’une hiérarchie d’appareils IoT Edge à l’aide de passerelles](tutorial-nested-iot-edge.md).

## <a name="create-a-gateway-hierarchy"></a>Création d’une hiérarchie de passerelle

Pour créer une hiérarchie de passerelle IoT Edge, définissez des relations parent/enfant des appareils IoT Edge dans le scénario. Vous pouvez définir un appareil parent lorsque vous créez une identité d’appareil, ou bien gérer le parent et les enfants d’une identité d’appareil existante.

L’étape de configuration des relations parent/enfant permet d’autoriser les appareils enfants à se connecter à leur appareil parent de la même manière qu’à IoT Hub.

Seuls les appareils IoT Edge peuvent être des appareils parents, tandis que peuvent être enfants les appareils IoT Edge et IoT. Un parent peut avoir de nombreux enfants, alors qu’un enfant ne peut avoir qu’un seul parent. Une hiérarchie de passerelle est créée en chaînant des ensembles parent/enfant de sorte que l’enfant d’un appareil constitue le parent d’un autre.

<!-- TODO: graphic of gateway hierarchy -->

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sur le Portail Azure, vous pouvez gérer la relation parent/enfant lorsque vous créez de nouvelles identités d’appareil ou en modifiant des appareils existants.

Lors de la création d’un appareil IoT Edge, il est possible de choisir des appareils parents et enfants dans la liste des appareils IoT Edge existants dans ce hub.

1. Accédez à votre IoT Hub dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **IoT Edge** dans le menu de navigation.
1. Sélectionnez **Ajouter un appareil IoT Edge**.
1. En plus de l’ID de l’appareil et des paramètres d’authentification, vous pouvez **Définir un appareil parent** ou **Choisir des appareils enfants**.
1. Choisissez le ou les appareils parent et enfants.

Vous pouvez également créer ou gérer les relations parent/enfant d’appareils existants.

1. Accédez à votre IoT Hub dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **IoT Edge** dans le menu de navigation.
1. Sélectionnez l’appareil que vous souhaitez gérer dans la liste des **Appareils IoT Edge**.
1. Sélectionnez **Définir un appareil parent** ou **Gérer des appareils enfants**.
1. Ajoutez ou supprimez des appareils parent ou enfants.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

L’extension [azure-iot](/cli/azure/ext/azure-iot) pour Azure CLI offre des commandes permettant de gérer les ressources IoT. Vous pouvez gérer la relation parent/enfant d’appareils IoT et IoT Edge lorsque vous créez de nouvelles identités d’appareil ou en modifiant des appareils existants.

L’ensemble de commandes [az iot hub device-identity](/cli/azure/ext/azure-iot/iot/hub/device-identity) permet de gérer les relations parent/enfant d’un appareil donné.

La commande `create` comprend des paramètres relatifs à l’ajout d’appareils enfants et à la définition d’un appareil parent au moment de la création de l’appareil.

D’autres commandes device-identity, notamment `add-children`, `list-children` et `remove-children`, ou `get-parent` et `set-parent`, permettent de gérer les relations parent/enfant d’appareils existants.

---

## <a name="prepare-certificates"></a>Préparer les certificats

Il est nécessaire qu’une chaîne de certificats cohérente soit installée sur les appareils de la même hiérarchie de passerelle pour établir une communication sécurisée entre ces appareils. Chacun des appareils de la hiérarchie (appareil IoT Edge ou appareil de nœud terminal IoT) a besoin d’une copie du même certificat d’autorité de certification racine. Chaque appareil IoT Edge de la hiérarchie utilise alors ce certificat comme racine de son certificat d’autorité de certification d’appareil.

Avec cette configuration, chacun des appareils IoT Edge et appareils de nœud terminal IoT en aval peut connaître l’identité de son parent en vérifiant que le module edgeHub auquel il se connecte possède un certificat de serveur signé par le certificat d’autorité de certification racine partagée.

<!-- TODO: certificate graphic -->

Créez les certificats suivants :

* Un **certificat d’autorité de certification racine**, à savoir le plus haut certificat partagé pour tous les appareils d’une hiérarchie de passerelle donnée. Il est installé sur tous les appareils.
* Les **certificats intermédiaires** que vous souhaitez inclure dans la chaîne de certificats racines.
* Un **certificat d’autorité de certification d’appareil** et sa **clé privée**, générés par les certificats racine et intermédiaires. Un certificat d’autorité de certification d’appareil unique est nécessaire pour chacun des appareils IoT Edge de la hiérarchie de passerelle.

>[!NOTE]
>Il existe actuellement dans libiothsm une limitation empêchant l’utilisation de certificats qui expirent le 1er janvier 2038 ou après cette date.

Vous pouvez soit utiliser une autorité de certification auto-signée, soit en acheter un auprès d’une autorité de certification commerciale approuvée comme Baltimore, Verisign, DigiCert ou GlobalSign.

Si vous ne disposez pas de certificats propres, vous pouvez [créer des certificats de démonstration pour tester les fonctionnalités de l’appareil IoT Edge](how-to-create-test-certificates.md). Suivez la procédure de cet article pour créer un ensemble de certificats racine et intermédiaires, puis des certificats d’autorité de certification d’appareil IoT Edge pour chacun de vos appareils.

## <a name="configure-iot-edge-on-devices"></a>Configuration de IoT Edge sur les appareils

La procédure de configuration d’IoT Edge en tant que passerelle est très similaire à la procédure de configuration en tant qu’appareil en aval.

Pour activer la détection de passerelle, il est nécessaire que chacun des appareils de passerelle IoT Edge soit configuré avec un **nom d’hôte** permettant à ses appareils enfants de le trouver sur le réseau local. Chaque appareil IoT Edge en aval doit être configuré avec un **parent_hostname** auquel il se connectera. Un appareil IoT Edge à la fois parent et enfant a besoin des deux paramètres.

La condition pour permettre les connexions sécurisées est que chacun des appareils IoT Edge d’un scénario de passerelle soit configuré avec un unique certificat d’autorité de certification d’appareil et une copie du certificat d’autorité de certification racine partagé par tous les appareils de la hiérarchie de passerelle.

IoT Edge doit être déjà installé sur l’appareil. Si ce n’est pas le cas, suivez la procédure [Installation du runtime Azure IoT Edge](how-to-install-iot-edge.md), puis provisionnez votre appareil avec [l’authentification par clé symétrique](how-to-manual-provision-symmetric-key.md) ou [l’authentification par certificat X.509](how-to-manual-provision-x509.md).

La procédure de cette section fait référence au **certificat d’autorité de certification racine** ainsi qu’au **certificat d’autorité de certification d’appareil et à la clé privée** qui ont été abordés plus haut dans cet article. Si vous avez créé ces certificats sur un autre appareil, rendez-les disponibles sur celui-ci. Vous pouvez transférer les fichiers physiquement (par exemple, avec un lecteur USB), avec un service comme [Azure Key Vault](../key-vault/general/overview.md) ou avec une fonction comme la [Copie de fichiers sécurisée](https://www.ssh.com/ssh/scp/).

Procédez comme suit pour configurer IoT Edge sur votre appareil.

Sur Linux, veillez à ce que l’utilisateur **iotedge** dispose d’autorisations d’accès en lecture au répertoire contenant les certificats et les clés.

1. Installez le **certificat d’autorité de certification racine** sur cet appareil IoT Edge.

   ```bash
   sudo cp <path>/<root ca certificate>.pem /usr/local/share/ca-certificates/<root ca certificate>.pem
   ```

1. Mettez à jour le magasin de certificats.

   ```bash
   sudo update-ca-certificates
   ```

   Cette commande doit indiquer en sortie qu’un certificat a été ajouté dans /etc/ssl/certs.

1. Ouvrez le fichier de configuration du démon de sécurité IoT Edge.

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

1. Recherchez la section **certificates** du fichier config.yaml. Mettez à jour les trois champs de certificat de sorte qu’ils pointent vers vos certificats. Indiquez les chemins d’URI de fichier, au format `file:///<path>/<filename>`.

   * **device_ca_cert** : chemin d’URI de fichier du certificat d’autorité de certification d’appareil propre à l’appareil.
   * **device_ca_pk** : chemin d’URI de fichier de la clé privée d’autorité de certification d’appareil propre à l’appareil.
   * **trusted_ca_certs** : chemin d’URI de fichier du certificat d’autorité de certification racine partagé par tous les appareils de la hiérarchie de passerelle.

1. Recherchez le paramètre **hostname** dans le fichier config.yaml. Remplacez la valeur du nom d’hôte par le nom de domaine complet (FQDN) ou l’adresse IP de l’appareil IoT Edge.

   C’est la valeur de ce paramètre qui sera utilisée par les appareils en aval pour se connecter à cette passerelle. Le nom d’hôte prend le nom de l’ordinateur par défaut, mais le nom de domaine complet ou l’adresse IP est requis pour connecter les appareils en aval.

   Utilisez un nom d’hôte inférieur à la limite de caractères d’un nom commun de certificat de serveur, soit 64 caractères.

   Maintenez une cohérence avec le modèle de nom d’hôte dans toute la hiérarchie de passerelle. Utilisez soit des noms de domaine complets, soit des adresses IP, non les deux.

1. **Si cet appareil est un appareil enfant**, recherchez le paramètre **parent_hostname**. Remplacez la valeur du champ **parent_hostname** par le nom de domaine complet ou l’adresse IP de l’appareil parent, en fonction de ce qui a été indiqué comme nom d’hôte dans le fichier config.yaml du parent.

1. Bien que cette fonctionnalité soit en préversion publique, vous devez configurer votre appareil IoT Edge de sorte qu’il utilise la préversion publique de l’agent IoT Edge au démarrage.

   Recherchez la section YAML **agent** et remplacez la valeur image par l’image de la préversion publique :

   ```yml
   agent:
     name: "edgeAgent"
     type: "docker"
     env: {}
     config:
       image: "mcr.microsoft.com/azureiotedge-agent:1.2.0-rc2"
       auth: {}
   ```

1. Enregistrez (`Ctrl+O`) et fermez (`Ctrl+X`) le fichier config.yaml.

1. Si vous avez déjà utilisé d’autres certificats pour IoT Edge, supprimez les fichiers dans les deux répertoires suivants pour que vos nouveaux certificats s’appliquent :

   * `/var/lib/iotedge/hsm/certs`
   * `/var/lib/iotedge/hsm/cert_keys`

1. Redémarrez le service IoT Edge pour appliquer vos modifications.

   ```bash
   sudo systemctl restart iotedge
   ```

1. Recherchez les éventuelles erreurs dans la configuration.

   ```bash
   sudo iotedge check --verbose
   ```

   >[!TIP]
   >L’outil de vérification IoT Edge utilise un conteneur pour effectuer une partie du contrôle diagnostics. Si vous souhaitez vous servir de cet outil sur des appareils IoT Edge en aval, veillez à ce qu’ils puissent accéder à `mcr.microsoft.com/azureiotedge-diagnostics:latest`, ou placez l’image conteneur dans votre registre de conteneurs privé.

## <a name="configure-runtime-modules-for-public-preview"></a>Configuration des modules runtime pour la préversion publique

Bien que cette fonctionnalité soit en préversion publique, vous devez configurer votre appareil IoT Edge de sorte qu’il utilise la préversion publique des modules runtime IoT Edge. La section précédente donne la procédure de configuration d’edgeAgent au démarrage. Il vous faut également configurer les modules runtime dans les déploiements pour votre appareil.

1. Configurez le module edgeHub de façon à utiliser l’image en préversion publique : `mcr.microsoft.com/azureiotedge-hub:1.2.0-rc2`.

1. Configurez les variables d’environnement suivantes pour le module edgeHub :

   | Nom | Valeur |
   | - | - |
   | `experimentalFeatures__enabled` | `true` |
   | `experimentalFeatures__nestedEdgeEnabled` | `true` |

1. Configurez le module edgeAgent de façon à utiliser l’image en préversion publique : `mcr.microsoft.com/azureiotedge-hub:1.2.0-rc2`.

## <a name="network-isolate-downstream-devices"></a>Isolement réseau des appareils en aval

Dans cet article ont été indiquées les procédures visant à configurer les appareils IoT Edge comme une passerelle ou un appareil en aval, et à créer une connexion approuvée entre eux. L’appareil de passerelle gère les interactions entre l’appareil en aval et IoT Hub, notamment l’authentification et le routage des messages. Les modules déployés sur des appareils IoT Edge en aval peuvent toujours créer leurs propres connexions aux services cloud.

Certaines architectures réseau, comme celles qui suivent la norme ISA-95, cherchent à réduire le nombre de connexions Internet. Dans ces scénarios, vous pouvez configurer des appareils IoT Edge en aval sans connectivité Internet directe. Au-delà du routage des communications IoT Hub via leur appareil de passerelle, les appareils IoT Edge en aval peuvent s’appuyer sur cet appareil pour toutes les connexions au cloud.

Dans cette configuration réseau, seul l’appareil IoT Edge de la couche supérieure de la hiérarchie de passerelle doit posséder des connexions directes au cloud. Les appareils IoT Edge des couches inférieures ne peuvent communiquer qu’avec leur appareil parent et leurs appareils enfants. Les modules spéciaux sur les appareils de passerelle permettent ce scénario :

* Le **module proxy d’API** est requis sur toutes les passerelles IoT Edge sous lesquelles se trouve un autre appareil IoT Edge. Il doit donc se trouver sur *chaque couche* d’une hiérarchie de passerelle, à l’exception de la couche inférieure. Il utilise un proxy inverse [nginx](https://nginx.org) pour acheminer les données HTTP à travers les couches réseau sur un port unique. Il est extrêmement configurable par le biais de son jumeau de module et de ses variables d’environnement. Il peut donc être adapté en fonction des exigences de votre scénario de passerelle.

* Le **module registre Docker** peut être déployé sur la passerelle IoT Edge au niveau de la *couche supérieure* d’une hiérarchie de passerelle. Il est responsable de la récupération et de la mise en cache des images conteneur pour le compte de tous les appareils IoT Edge des couches inférieures. En dehors du déploiement de ce module au niveau de la couche supérieure, il existe d’autres possibilités : utiliser un registre local, ou charger manuellement les images conteneur sur les appareils et définir la stratégie de tirage (pull) de module sur **never**.

* Le module **Stockage Blob Azure sur IoT Edge** peut être déployé sur la passerelle IoT Edge au niveau de la *couche supérieure* d’une hiérarchie de passerelle. Il est responsable du chargement des objets blob pour le compte de tous les appareils IoT Edge des couches inférieures. La possibilité de charger des objets blob offre également des fonctionnalités de résolution des problèmes utiles pour les appareils IoT Edge des couches inférieures, notamment le chargement du journal de module et du Bundle de support.

### <a name="network-configuration"></a>Configuration réseau

Pour chacun des appareils de passerelle de la couche supérieure, les opérateurs réseau doivent effectuer plusieurs actions :

* Fournir une adresse IP statique ou un nom de domaine complet (FQDN)
* Autoriser les communications sortantes de cette adresse IP vers votre nom d’hôte Azure IoT Hub sur les ports 443 (HTTPS) et 5671 (AMQP)
* Autoriser les communications sortantes de cette adresse IP vers votre nom d’hôte Azure Container Registry sur le port 443 (HTTPS)

  Le module proxy d’API ne peut gérer les connexions qu’à un seul registre de conteneurs à la fois. Nous vous recommandons de stocker toutes les images conteneur, y compris les images publiques fournies par Microsoft Container Registry (mcr.microsoft.com), dans votre registre de conteneurs privé.

Pour chacun des appareils de passerelle des couches inférieures, les opérateurs réseau doivent effectuer plusieurs actions :

* Fournir une adresse IP statique
* Autoriser les communications sortantes de cette adresse IP vers l’adresse IP de la passerelle parente sur les ports 443 (HTTPS) et 5671 (AMQP)

### <a name="deploy-modules-to-top-layer-devices"></a>Déploiement des modules sur les appareils de la couche supérieure

Il existe un ensemble de modules requis qui doivent être déployés sur l’appareil IoT Edge de la couche supérieure d’une hiérarchie de passerelle, en plus des éventuels modules de charge de travail qui s’exécutent sur l’appareil.

Le module proxy d’API a été conçu pour être personnalisé afin de gérer la plupart des scénarios de passerelle courants. Cet article donne un exemple de configuration de base des modules. Pour plus d’informations et d’exemples, consultez [Configuration du module proxy d’API pour un scénario de hiérarchie de passerelle](how-to-configure-api-proxy-module.md).

1. Accédez à votre IoT Hub dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **IoT Edge** dans le menu de navigation.
1. Sélectionnez l’appareil de la couche supérieure que vous configurez dans la liste **Appareils IoT Edge**.
1. Sélectionnez **Définir des modules**.
1. Dans la section **Modules IoT Edge**, sélectionnez **Ajouter**, puis choisissez **Module Place de marché**.
1. Recherchez et sélectionnez le module **Proxy d’API IoT Edge**.
1. Sélectionnez le nom du module proxy d’API dans la liste des modules déployés et mettez à jour les paramètres de module suivants :
   1. Dans l’onglet **Variables d’environnement**, remplacez la valeur de **NGINX_DEFAULT_PORT** par `443`.
   1. Dans l’onglet **Options de création de conteneur**, mettez à jour les liaisons de port en indiquant le port 443.

      ```json
      {
        "HostConfig": {
          "PortBindings": {
            "443/tcp": [
              {
                "HostPort": "443"
              }
            ]
          }
        }
      }
      ```

   Ces modifications permettent de configurer le module proxy d’API de sorte qu’il écoute le port 443. Pour éviter les collisions de liaison de port, vous devez configurer le module edgeHub de sorte qu’il n’écoute pas le port 443. Le module proxy d’API acheminera tout le trafic edgeHub sur le port 443.

1. Sélectionnez **Paramètres de runtime** et recherchez les options de création du module edgeHub. Supprimez la liaison du port 443, en laissant celles des ports 5671 et 8883.

   ```json
   {
     "HostConfig": {
       "PortBindings": {
         "5671/tcp": [
           {
             "HostPort": "5671"
           }
         ],
         "8883/tcp": [
           {
             "HostPort": "8883"
           }
         ]
       }
     }
   }
   ```

1. Sélectionnez **Enregistrer** pour enregistrer les modifications apportées aux paramètres de runtime.
1. Sélectionnez à nouveau **Ajouter**, puis choisissez **Module IoT Edge**.
1. Indiquez les valeurs suivantes pour ajouter le module registre Docker à votre déploiement :
   1. **Nom du module IoT Edge** : `registry`.
   1. Dans l’onglet **Paramètres du module**, **URI de l’image** : `registry:latest`.
   1. Dans l’onglet **Variables d’environnement**, ajoutez les variables d’environnement suivantes :

      * **Nom** : `REGISTRY_PROXY_REMOTEURL` **Valeur** : URL du registre de conteneurs auquel vous souhaitez associer ce module registre. Par exemple : `https://myregistry.azurecr`.

        Le module registre ne peut être associé qu’à un seul registre de conteneurs. Nous vous recommandons donc de conserver toutes les images conteneur dans un même registre de conteneurs privé.

      * **Nom** : `REGISTRY_PROXY_USERNAME` **Valeur** : nom d’utilisateur pour l’authentification auprès du registre de conteneurs.

      * **Nom** : `REGISTRY_PROXY_PASSWORD` **Valeur** : mot de passe pour l’authentification auprès du registre de conteneurs.

   1. Dans l’onglet **Options de création de conteneur**, collez :

      ```json
      {
          "HostConfig": {
              "PortBindings": {
                  "5000/tcp": [
                      {
                          "HostPort": "5000"
                      }
                  ]
              }
          }
      }
      ```

1. Sélectionnez **Ajouter** pour ajouter le module au déploiement.
1. Sélectionnez **Suivant : Itinéraires** pour accéder à l’étape suivante.
1. Pour permettre aux messages appareil-à-cloud issus des appareils en aval d’atteindre IoT Hub, ajoutez un itinéraire qui transmet tous les messages à IoT Hub. Par exemple :
    1. **Nom** : `Route`
    1. **Valeur** : `FROM /messages/* INTO $upstream`
1. Sélectionnez **Vérifier + créer** pour passer à l’étape finale.
1. Sélectionnez **Créer** pour effectuer le déploiement sur votre appareil.

### <a name="deploy-modules-to-lower-layer-devices"></a>Déploiement de modules sur les appareils de couche inférieure

Il existe un module requis qui doit être déployé sur les appareils IoT Edge des couches inférieures d’une hiérarchie de passerelle, en plus des éventuels modules de charge de travail qui s’exécutent sur les appareils.

#### <a name="route-container-image-pulls"></a>Acheminement des tirages (pull) d’images conteneur

Avant d’aborder le module proxy requis pour les appareils IoT Edge des hiérarchies de passerelle, il est important de comprendre comment les appareils IoT Edge des couches inférieures obtiennent leur image de module.

Si vos appareils de couche inférieure ne peuvent pas se connecter au cloud, mais que vous souhaitez qu’ils tirent (pull) les images de module comme d’habitude, vous devez configurer l’appareil de la couche supérieure de la hiérarchie de passerelle de façon à gérer ces demandes. Il doit exécuter un module **registre** Docker associé à votre registre de conteneurs. Configurez ensuite le module proxy d’API de sorte qu’il achemine les demandes de conteneur vers celui-ci. Ces informations sont abordées dans les sections précédentes de cet article. Dans cette configuration, les appareils de couche inférieure ne doivent pas pointer vers les registres de conteneurs cloud, mais vers le registre qui s’exécute dans la couche supérieure.

Par exemple, au lieu d’appeler `mcr.microsoft.com/azureiotedge-api-proxy:latest`, les appareils de couche inférieure doivent appeler `$upstream:443/azureiotedge-api-proxy:latest`.

Le paramètre **$upstream** pointe vers le parent d’un appareil de couche inférieure. La demande est donc acheminée à travers toutes les couches jusqu’à la couche supérieure possédant un environnement proxy qui route les demandes de conteneur au module registre. Le port `:443` de cet exemple doit être remplacé par celui qu’écoute le module proxy d’API de l’appareil parent.

Le module proxy d’API ne peut acheminer les demandes que vers un module registre. Par ailleurs, un module registre ne peut être associé qu’à un seul registre de conteneurs. Il faut par conséquent que toutes les images que les appareils de couche inférieure doivent tirer (pull) soient stockées dans un même registre de conteneurs.

Si vous ne souhaitez pas que les appareils de couche inférieure effectuent des demandes de tirage (pull request) de module à travers une hiérarchie de passerelle, il existe une autre solution, qui consiste à gérer une solution de registre locale. Vous pouvez également envoyer (push) les images de module sur les appareils avant de créer des déploiements, puis définir **imagePullPolicy** sur **never**.

#### <a name="bootstrap-the-iot-edge-agent"></a>Démarrage de l’agent IoT Edge

L’agent IoT Edge est le premier composant de runtime à démarrer sur un appareil IoT Edge. Vous devez veiller à ce que les appareils IoT Edge en aval puissent accéder à l’image de module edgeAgent au démarrage, puis accéder aux déploiements et lancer les autres images de module.

Lorsque vous accédez au fichier config.yaml sur un appareil IoT Edge pour fournir ses informations d’authentification, ses certificats et son nom d’hôte parent, mettez également à jour l’image conteneur edgeAgent.

Si l’appareil de passerelle de la couche supérieure est configuré pour gérer les demandes d’images conteneur, remplacez `mcr.microsoft.com` par le nom d’hôte parent et le port d’écoute du proxy d’API. Dans le manifeste de déploiement, vous pouvez utiliser `$upstream` comme raccourci, mais il faut pour cela que le module edgeHub gère le routage et qu’il n’ait pas encore démarré. Par exemple :

```yml
agent:
  name: "edgeAgent"
  type: "docker"
  env: {}
  config:
    image: "{Parent FQDN or IP}:443/azureiotedge-agent:1.2.0-rc2"
    auth: {}
```

Si vous utilisez un registre de conteneurs local ou que vous fournissez manuellement les images conteneur sur l’appareil, mettez à jour le fichier config.yaml en conséquence.

#### <a name="configure-runtime-and-deploy-proxy-module"></a>Configuration du runtime et déploiement du module proxy

Le **module proxy d’API** est requis pour acheminer toutes les communications entre le cloud et tous les appareils IoT Edge en aval. Un appareil IoT Edge de la couche inférieure de la hiérarchie, dépourvu d’appareils IoT Edge en aval, n’a pas besoin de ce module.

Le module proxy d’API a été conçu pour être personnalisé afin de gérer la plupart des scénarios de passerelle courants. Cet article aborde brièvement la procédure de configuration de base des modules. Pour plus d’informations et d’exemples, consultez [Configuration du module proxy d’API pour un scénario de hiérarchie de passerelle](how-to-configure-api-proxy-module.md).

1. Accédez à votre IoT Hub dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **IoT Edge** dans le menu de navigation.
1. Sélectionnez l’appareil de couche inférieure que vous configurez dans la liste **Appareils IoT Edge**.
1. Sélectionnez **Définir des modules**.
1. Dans la section **Modules IoT Edge**, sélectionnez **Ajouter**, puis choisissez **Module Place de marché**.
1. Recherchez et sélectionnez le module **Proxy d’API IoT Edge**.
1. Sélectionnez le nom du module proxy d’API dans la liste des modules déployés et mettez à jour les paramètres de module suivants :
   1. Dans l’onglet **Paramètres du module**, mettez à jour la valeur **URI d’image**. Remplacez `mcr.microsoft.com` par `$upstream:443`.
   1. Dans l’onglet **Variables d’environnement**, remplacez la valeur de **NGINX_DEFAULT_PORT** par `443`.
   1. Dans l’onglet **Options de création de conteneur**, mettez à jour les liaisons de port en indiquant le port 443.

      ```json
      {
        "HostConfig": {
          "PortBindings": {
            "443/tcp": [
              {
                "HostPort": "443"
              }
            ]
          }
        }
      }
      ```

   Ces modifications permettent de configurer le module proxy d’API de sorte qu’il écoute le port 443. Pour éviter les collisions de liaison de port, vous devez configurer le module edgeHub de sorte qu’il n’écoute pas le port 443. Le module proxy d’API acheminera tout le trafic edgeHub sur le port 443.

1. Sélectionnez **Paramètres du runtime**.
1. Mettez à jour les paramètres du module edgeHub :

   1. Dans le champ **Image**, remplacez `mcr.microsoft.com` par `$upstream:443`.
   1. Dans le champ **Options de création**, supprimez la liaison du port 443, en laissant celles des ports 5671 et 8883.

   ```json
   {
     "HostConfig": {
       "PortBindings": {
         "5671/tcp": [
           {
             "HostPort": "5671"
           }
         ],
         "8883/tcp": [
           {
             "HostPort": "8883"
           }
         ]
       }
     }
   }
   ```

1. Mettez à jour les paramètres du module edgeAgent :
   1. Dans le champ **Image**, remplacez `mcr.microsoft.com` par `$upstream:443`.

1. Sélectionnez **Enregistrer** pour enregistrer les modifications apportées aux paramètres de runtime.
1. Sélectionnez **Suivant : Itinéraires** pour accéder à l’étape suivante.
1. Pour permettre aux messages appareil-à-cloud issus des appareils en aval d’atteindre IoT Hub, ajoutez un itinéraire qui transmet tous les messages à `$upstream`. Le paramètre en amont pointe vers l’appareil parent dans le cas des appareils de couche inférieure. Par exemple :
    1. **Nom** : `Route`
    1. **Valeur** : `FROM /messages/* INTO $upstream`
1. Sélectionnez **Vérifier + créer** pour passer à l’étape finale.
1. Sélectionnez **Créer** pour effectuer le déploiement sur votre appareil.

## <a name="next-steps"></a>Étapes suivantes

[Guide pratique pour utiliser un appareil IoT Edge en tant que passerelle](iot-edge-as-gateway.md)

[Configuration du module proxy d’API pour un scénario de hiérarchie de passerelle](how-to-configure-api-proxy-module.md)