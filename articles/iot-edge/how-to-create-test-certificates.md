---
title: Créer des certificats de test-Azure IoT Edge | Microsoft Docs
description: Créez des certificats de test et apprenez à les installer sur un appareil Azure IoT Edge pour préparer le déploiement de production.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/02/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: c5af77da0ed2c579a478c8ebaaa924882d9a15c6
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927700"
---
# <a name="create-demo-certificates-to-test-iot-edge-device-features"></a>Créer des certificats de démonstration pour tester les fonctionnalités de l’appareil IoT Edge

Les appareils IoT Edge requièrent des certificats pour sécuriser la communication entre le runtime, les modules et tous les appareils en aval.
Si vous n’avez pas d’autorité de certification pour créer les certificats requis, vous pouvez utiliser des certificats de démonstration pour essayer les fonctionnalités IoT Edge dans votre environnement de test.
Cet article décrit les fonctionnalités des scripts de génération de certificat que IoT Edge fournit pour le test.

Ces certificats expirent au bout de 30 jours et ne doivent pas être utilisés dans un scénario de production.

Vous pouvez créer des certificats sur n’importe quel ordinateur, puis les copier sur votre appareil IoT Edge.
Il est plus facile d’utiliser votre ordinateur principal pour créer les certificats qu’en les générant sur votre appareil IoT Edge lui-même.
À l'aide de votre ordinateur principal, vous pouvez configurer les scripts, puis les utiliser afin de créer des certificats pour plusieurs appareils.

Procédez comme suit pour créer des certificats de démonstration afin de tester votre scénario IoT Edge :

1. [Configurez des scripts](#set-up-scripts) pour générer des certificats sur votre appareil.
2. [Créez le certificat d’autorité de certification racine](#create-root-ca-certificate) que vous utilisez pour signer tous les autres certificats de votre scénario.
3. Générez les certificats dont vous avez besoin dans le cadre du scénario à tester :
   * [Créer des certificats d’identité d’appareil IoT Edge](#create-iot-edge-device-identity-certificates) pour l’approvisionnement automatique avec Service IoT Hub Device Provisioning.
   * [Créer des certificats d’autorité de certification d’appareil IoT Edge](#create-iot-edge-device-ca-certificates) pour les appareils IoT Edge dans les scénarios de passerelle.
   * [Créer des certificats d’appareil en aval](#create-downstream-device-certificates) pour l’authentification des appareils en aval pour IoT Hub dans un scénario de passerelle.

## <a name="prerequisites"></a>Prérequis

Ordinateur de développement sur lequel git est installé.

## <a name="set-up-scripts"></a>Configuration de scripts

Le référentiel IoT Edge sur GitHub comprend des scripts de génération de certificat que vous pouvez utiliser pour créer des certificats de démonstration.
Cette section fournit des instructions pour préparer les scripts à exécuter sur votre ordinateur, sous Windows ou Linux.
Si vous êtes sur une machine Linux, passez directement à la [Configuration sur un ordinateur Linux](#set-up-on-linux).

### <a name="set-up-on-windows"></a>Configurer sur Windows

Pour créer des certificats de démonstration sur un appareil Windows, vous devez installer OpenSSL, puis cloner les scripts de génération et les configurer pour qu’ils s’exécutent localement dans PowerShell.

#### <a name="install-openssl"></a>Installer OpenSSL

Installez OpenSSL pour Windows sur l’ordinateur que vous utilisez pour générer les certificats.
Si vous avez déjà installé OpenSSL sur votre appareil Windows, assurez-vous que le fichier openssl.exe est disponible dans la variable d’environnement PATH.

Il existe plusieurs façons d’installer OpenSSL, y compris les options suivantes :

* **Plus facile :** Téléchargez et installez les [binaires OpenSSL de fournisseurs tiers](https://wiki.openssl.org/index.php/Binaries), par exemple, à partir d’ [OpenSSL sur SourceForge](https://sourceforge.net/projects/openssl/). Ajoutez le chemin d’accès complet à openssl.exe à votre variable d’environnement PATH.

* **Recommandé :** Téléchargez le code source OpenSSL et générez les fichiers binaires sur votre machine vous-même ou en utilisant [vcpkg](https://github.com/Microsoft/vcpkg). Les instructions ci-dessous utilisent vcpkg pour télécharger le code source, compiler et installer OpenSSL sur votre machine Windows en suivant des étapes simples.

   1. Accédez au répertoire où vous souhaitez installer vcpkg. Suivez les instructions pour télécharger et installer [vcpkg](https://github.com/Microsoft/vcpkg).

   2. Une fois vcpkg installé, exécutez la commande suivante à partir d’une invite PowerShell afin d’installer le package OpenSSL pour Windows x64. L’installation prend généralement 5 minutes environ.

      ```powershell
      .\vcpkg install openssl:x64-windows
      ```

   3. Ajoutez `<vcpkg path>\installed\x64-windows\tools\openssl` à votre variable d’environnement PATH afin que le fichier openssl.exe soit appelable.

#### <a name="prepare-scripts-in-powershell"></a>Préparer les scripts dans PowerShell

Le dépôt Git Azure IoT Edge contient des scripts permettant de générer des certificats de test.
Dans cette section, vous clonez le dépôt IoT Edge et exécutez les scripts.

1. Ouvrez une fenêtre PowerShell en mode administrateur.

2. Clonez le référentiel Git IoT Edge, qui contient des scripts pour générer des certificats de démonstration. Utilisez la commande `git clone` ou [téléchargez le fichier zip](https://github.com/Azure/iotedge/archive/master.zip).

   ```powershell
   git clone https://github.com/Azure/iotedge.git
   ```

3. Accédez au répertoire dans lequel vous souhaitez travailler. Ce répertoire de travail sera appelé *\<WRKDIR>* tout au long de cet article. L’ensemble des certificats et des clés seront créés dans ce répertoire de travail.

4. Copiez les fichiers de configuration et de script du dépôt cloné dans votre répertoire de travail.

   ```powershell
   copy <path>\iotedge\tools\CACertificates\*.cnf .
   copy <path>\iotedge\tools\CACertificates\ca-certs.ps1 .
   ```

   Si vous avez téléchargé le dépôt sous forme de fichier zip, le nom du dossier est `iotedge-master` et le reste du chemin est le même.

5. Activez PowerShell pour exécuter les scripts.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
   ```

6. Intégrez les fonctions utilisées par les scripts à l’espace de noms global de PowerShell.

   ```powershell
   . .\ca-certs.ps1
   ```

   La fenêtre PowerShell affiche un avertissement signalant que les certificats générés par ce script sont destinés aux tests uniquement et ne doivent pas être utilisés dans les scénarios de production.

7. Vérifiez qu’OpenSSL est correctement installé et qu’il n’y aura pas de collisions de noms avec des certificats existants. Si des problèmes se produisent, la sortie du script doit décrire comment les corriger sur votre système.

   ```powershell
   Test-CACertsPrerequisites
   ```

### <a name="set-up-on-linux"></a>Configurer sur Linux

Pour créer des certificats de démonstration sur un appareil Windows, vous devez cloner les scripts de génération et les configurer pour qu’ils s’exécutent localement dans bash.

1. Clonez le référentiel Git IoT Edge, qui contient des scripts pour générer des certificats de démonstration.

   ```bash
   git clone https://github.com/Azure/iotedge.git
   ```

2. Accédez au répertoire dans lequel vous souhaitez travailler. Ce répertoire sera appelé *\<WRKDIR>* tout au long de cet article. Tous les fichiers de clés et de certificats seront créés dans ce répertoire.
  
3. Copiez les fichiers de configuration et de script du dépôt IoT Edge cloné dans votre répertoire de travail.

   ```bash
   cp <path>/iotedge/tools/CACertificates/*.cnf .
   cp <path>/iotedge/tools/CACertificates/certGen.sh .
   ```

<!--
4. Configure OpenSSL to generate certificates using the provided script. 

   ```bash
   chmod 700 certGen.sh 
   ```
-->

## <a name="create-root-ca-certificate"></a>Créer un certificat d’autorité de certification racine

Le certificat d’autorité de certification racine est utilisé pour créer tous les autres certificats de démonstration en vue de tester un scénario IoT Edge.
Vous pouvez continuer à utiliser le même certificat d’autorité de certification racine pour créer des certificats de démonstration pour plusieurs appareils IoT Edge ou appareils en aval.

Si vous avez déjà un certificat d’autorité de certification racine dans votre dossier de travail, n’en créez pas un nouveau.
Le nouveau certificat d’autorité de certification racine remplacera l’ancien, et tous les certificats en aval générés à partir de l’ancien certificat cesseront de fonctionner.
Si vous souhaitez plusieurs certificats d’autorité de certification racine, assurez-vous de les gérer dans des dossiers distincts.

Avant de suivre les étapes de cette section, suivez les étapes décrites dans la section [Configurer des scripts](#set-up-scripts) pour préparer un répertoire de travail avec les scripts de génération de certificat de démonstration.

### <a name="windows"></a>Windows

1. Accédez au répertoire de travail où vous avez placé les scripts de génération de certificat.

1. Créez le certificat d’autorité de certification racine et faites-le signer un certificat intermédiaire. Les certificats sont tous placés dans le répertoire de travail.

   ```powershell
   New-CACertsCertChain rsa
   ```

   Cette commande de script crée plusieurs fichiers de certificat et de clé, mais lorsque les articles demandent le **certificat d’autorité de certification racine** , utilisez le fichier suivant :

   * `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`

### <a name="linux"></a>Linux

1. Accédez au répertoire de travail où vous avez placé les scripts de génération de certificat.

1. Créez le certificat d’autorité de certification racine et un certificat intermédiaire.

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

   Cette commande de script crée plusieurs fichiers de certificat et de clé, mais lorsque les articles demandent le **certificat d’autorité de certification racine** , utilisez le fichier suivant :

   * `<WRKDIR>/certs/azure-iot-test-only.root.ca.cert.pem`  

## <a name="create-iot-edge-device-identity-certificates"></a>Créer des certificats d'identité d’appareil IoT Edge

Les certificats d’identité d'appareil sont utilisés pour approvisionner les appareils IoT Edge via le service Azure IoT Hub Device Provisioning.

Les certificats d’identité d’appareil sont placés dans la section **Approvisionnement** du fichier config.yaml de l’appareil IoT Edge.

Avant de suivre les étapes de cette section, suivez les étapes décrites dans les sections [Configurer des scripts](#set-up-scripts) et [Créer un certificat d’autorité de certification racine](#create-root-ca-certificate).

### <a name="windows"></a>Windows

Créez la clé privée et le certificat d'identité d’appareil IoT Edge avec la commande suivante :

```powershell
New-CACertsEdgeDeviceIdentity "<name>"
```

Le nom que vous transmettez dans cette commande correspond à l’ID de l'appareil IoT Edge dans IoT Hub.

La nouvelle commande d’identité d’appareil crée plusieurs fichiers de certificats et de clés, y compris trois que vous utiliserez lors de la création d’une inscription individuelle dans DPS et de l’installation du runtime IoT Edge :

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>\certs\iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>\private\iot-edge-device-identity-<name>.key.pem`

### <a name="linux"></a>Linux

Créez la clé privée et le certificat d'identité d’appareil IoT Edge avec la commande suivante :

```bash
./certGen.sh create_edge_device_identity_certificate "<name>"
```

Le nom que vous transmettez dans cette commande correspond à l’ID de l'appareil IoT Edge dans IoT Hub.

Le script crée plusieurs fichiers de certificats et de clés, y compris trois que vous utiliserez lors de la création d’une inscription individuelle dans DPS et de l’installation du runtime IoT Edge :

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>/certs/iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>/private/iot-edge-device-identity-<name>.key.pem`

## <a name="create-iot-edge-device-ca-certificates"></a>Créer des certificats d’autorité de certification d’appareil IoT Edge

Chaque appareil IoT Edge destiné à la production a besoin d’un certificat d’autorité de certification d’appareil référencé à partir du fichier config.yaml.
Le certificat d’autorité de certification de l’appareil est chargé de créer des certificats pour les modules en cours d’exécution sur l’appareil.
Il est également nécessaire pour les scénarios de passerelle, car c'est grâce au certificat de l'autorité de certification de l'appareil que l'appareil IoT Edge vérifie son identité auprès des appareils en aval.

Les certificats d’autorité de certification d'appareil sont placés dans la section **Certificat** du fichier config.yaml de l’appareil IoT Edge.

Avant de suivre les étapes de cette section, suivez les étapes décrites dans les sections [Configurer des scripts](#set-up-scripts) et [Créer un certificat d’autorité de certification racine](#create-root-ca-certificate).

### <a name="windows"></a>Windows

1. Accédez au répertoire de travail qui contient les scripts de génération de certificats et le certificat d’Autorité de certification racine.

2. Créez la clé privée et le certificat d’autorité de certification d’appareil IoT Edge avec la commande suivante. Fournissez un nom pour le certificat de l'autorité de certification.

   ```powershell
   New-CACertsEdgeDevice "<CA cert name>"
   ```

   Cette commande crée plusieurs fichiers de certificat et de clé. Le certificat et la paire de clés suivants doivent être copiés sur un appareil IoT Edge et référencés dans le fichier config.yaml :

   * `<WRKDIR>\certs\iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>\private\iot-edge-device-<CA cert name>.key.pem`

Le nom d'appareil transmis à la commande **New-CACertsEdgeDevice** ne doit pas être le même que le paramètre hostname du fichier config.yaml ou que l'ID de l'appareil dans IoT Hub.

### <a name="linux"></a>Linux

1. Accédez au répertoire de travail qui contient les scripts de génération de certificats et le certificat d’autorité de certification racine.

2. Créez la clé privée et le certificat d’autorité de certification d’appareil IoT Edge avec la commande suivante. Fournissez un nom pour le certificat de l'autorité de certification.

   ```bash
   ./certGen.sh create_edge_device_ca_certificate "<CA cert name>"
   ```

   Cette commande de script crée plusieurs fichiers de certificats et de clés. Le certificat et la paire de clés suivants doivent être copiés sur un appareil IoT Edge et référencés dans le fichier config.yaml :

   * `<WRKDIR>/certs/iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>/private/iot-edge-device-<CA cert name>.key.pem`

Le nom transmis à la commande **create_edge_device_ca_certificate** ne doit pas être le même que le paramètre hostname du fichier config.yaml ni que l’ID de l’appareil dans IoT Hub.

## <a name="create-downstream-device-certificates"></a>Créer des certificats d’appareils en aval

Si vous configurez un appareil IoT en aval pour un scénario de passerelle et souhaitez utiliser l'authentification X.509, vous pouvez générer des certificats de démonstration pour l'appareil en aval.
Si vous souhaitez utiliser l’authentification par clé symétrique, vous ne devez créer aucun certificat pour l’appareil en aval.
Il existe deux façons d’authentifier un appareil IoT à l’aide de certificats X.509 : à l’aide de certificats auto-signés ou à l’aide de certificats signés par une autorité de certification.
Pour l’authentification par certificat X.509 autosigné, parfois appelée authentification par empreinte numérique, vous devez créer de nouveaux certificats et les placer sur votre appareil IoT.
Ces certificats contiennent une empreinte numérique que vous partagez avec IoT Hub pour l’authentification.
Pour l’authentification par certificat X.509 signés par l’autorité de certification, vous avez besoin d’un certificat d’autorité de certification racine inscrit dans IoT Hub, que vous utilisez pour signer les certificats de votre appareil IoT.
Tous les appareils qui utilisent un certificat émis par le certificat d’autorité de certification racine ou l’un de ses certificats intermédiaires sont autorisés à s’authentifier.

Les scripts de génération de certificats peuvent vous aider à créer des certificats de démonstration pour tester l’un de ces scénarios d’authentification.

Avant de suivre les étapes de cette section, suivez les étapes décrites dans les sections [Configurer des scripts](#set-up-scripts) et [Créer un certificat d’autorité de certification racine](#create-root-ca-certificate).

### <a name="self-signed-certificates"></a>Certificats auto-signés

Lorsque vous authentifiez un appareil IoT avec des certificats auto-signés, vous devez créer des certificats d’appareil basés sur le certificat d’autorité de certification racine de votre solution.
Ensuite, vous récupérez une « empreinte digitale » hexadécimale à partir des certificats à fournir à IoT Hub.
Votre appareil IoT a également besoin d’une copie de ses certificats d’appareil afin qu’il puisse s’authentifier auprès de IoT Hub.

#### <a name="windows"></a>Windows

1. Accédez au répertoire de travail qui contient les scripts de génération de certificats et le certificat d’autorité de certification racine.

2. Créez deux certificats pour l’appareil en aval (un primaire et un secondaire). Une convention d’affectation de noms pratique consiste à créer les certificats avec le nom de l’appareil IoT suivi de l’étiquette primaire ou secondaire. Par exemple :

   ```PowerShell
   New-CACertsDevice "<device name>-primary"
   New-CACertsDevice "<device name>-secondary"
   ```

   Cette commande de script crée plusieurs fichiers de certificats et de clés. Le certificat et les paires de clés suivants doivent être copiés sur l’appareil IoT en aval et référencés dans les applications qui se connectent à IoT Hub :

   * `<WRKDIR>\certs\iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>\private\iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>\private\iot-device-<device name>-secondary.key.pem`

3. Récupérez l’empreinte digitale SHA1 (appelée empreinte dans les contextes IoT Hub) à partir de chaque certificat. L’empreinte digitale est une chaîne hexadécimale d’une longueur de 40 caractères. Pour afficher le certificat et trouver l’empreinte digitale, utilisez la commande openssl suivante :

   ```PowerShell
   openssl x509 -in <WRKDIR>\certs\iot-device-<device name>-primary.cert.pem -text -fingerprint
   ```

   Exécutez cette commande deux fois, une fois pour le certificat principal et une fois pour le certificat secondaire. Vous fournissez des empreintes digitales pour les deux certificats lorsque vous inscrivez un nouvel appareil IoT à l’aide de certificats X.509 auto-signés.

#### <a name="linux"></a>Linux

1. Accédez au répertoire de travail qui contient les scripts de génération de certificats et le certificat d’autorité de certification racine.

2. Créez deux certificats pour l’appareil en aval (un primaire et un secondaire). Une convention d’affectation de noms pratique consiste à créer les certificats avec le nom de l’appareil IoT suivi de l’étiquette primaire ou secondaire. Par exemple :

   ```bash
   ./certGen.sh create_device_certificate "<device name>-primary"
   ./certGen.sh create_device_certificate "<device name>-secondary"
   ```

   Cette commande de script crée plusieurs fichiers de certificats et de clés. Le certificat et les paires de clés suivants doivent être copiés sur l’appareil IoT en aval et référencés dans les applications qui se connectent à IoT Hub :

   * `<WRKDIR>/certs/iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>/private/iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>/private/iot-device-<device name>-secondary.key.pem`

3. Récupérez l’empreinte digitale SHA1 (appelée empreinte dans les contextes IoT Hub) à partir de chaque certificat. L’empreinte digitale est une chaîne hexadécimale d’une longueur de 40 caractères. Pour afficher le certificat et trouver l’empreinte digitale, utilisez la commande openssl suivante :

   ```bash
   openssl x509 -in <WRKDIR>/certs/iot-device-<device name>-primary.cert.pem -text -fingerprint | sed 's/[:]//g'
   ```

   Vous fournissez des empreintes digitales primaire et secondaire lorsque vous inscrivez un nouvel appareil IoT à l’aide de certificats X.509 auto-signés.

### <a name="ca-signed-certificates"></a>Certificats signés par une autorité de certification

Lorsque vous authentifiez un appareil IoT avec des certificats auto-signés, vous devez charger le certificat d’autorité de certification racine de votre solution vers IoT Hub.
Ensuite, vous effectuez une vérification pour prouver à IoT Hub que vous êtes propriétaire du certificat d’autorité de certification racine.
Enfin, vous utilisez le même certificat d’autorité de certification racine pour créer des certificats d’appareil à placer sur votre appareil IoT afin qu’il puisse s’authentifier avec IoT Hub.

Les certificats de cette section concernent la procédure [Configurer la sécurité X.509 dans votre Azure IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md).

#### <a name="windows"></a>Windows

1. Chargez le fichier de certificat d’autorité de certification racine à partir de votre répertoire de travail, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, vers votre IoT Hub.

2. Utilisez le code fourni dans le Portail Azure pour vérifier que vous êtes propriétaire de ce certificat d’autorité de certification racine.

   ```PowerShell
   New-CACertsVerificationCert "<verification code>"
   ```

3. Créez une chaîne de certificats pour votre appareil en aval. Utilisez le même ID d’appareil que celui dans lequel l’appareil est inscrit auprès d’IoT Hub.

   ```PowerShell
   New-CACertsDevice "<device id>"
   ```

   Cette commande de script crée plusieurs fichiers de certificats et de clés. Le certificat et les paires de clés suivants doivent être copiés sur l’appareil IoT en aval et référencés dans les applications qui se connectent à IoT Hub :

   * `<WRKDIR>\certs\iot-device-<device id>.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device id>.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>\private\iot-device-<device id>.key.pem`

#### <a name="linux"></a>Linux

1. Chargez le fichier de certificat d’autorité de certification racine à partir de votre répertoire de travail, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, vers votre IoT Hub.

2. Utilisez le code fourni dans le Portail Azure pour vérifier que vous êtes propriétaire de ce certificat d’autorité de certification racine.

   ```bash
   ./certGen.sh create_verification_certificate "<verification code>"
   ```

3. Créez une chaîne de certificats pour votre appareil en aval. Utilisez le même ID d’appareil que celui dans lequel l’appareil est inscrit auprès d’IoT Hub.

   ```bash
   ./certGen.sh create_device_certificate "<device id>"
   ```

   Cette commande de script crée plusieurs fichiers de certificats et de clés. Le certificat et les paires de clés suivants doivent être copiés sur l’appareil IoT en aval et référencés dans les applications qui se connectent à IoT Hub :

   * `<WRKDIR>/certs/iot-device-<device id>.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device id>.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>/private/iot-device-<device id>.key.pem`
