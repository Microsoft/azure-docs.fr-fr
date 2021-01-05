---
title: Connecter des appareils en aval - Azure IoT Edge | Microsoft Docs
description: Guide pratique pour configurer des appareils en aval ou de nœud terminal afin de les connecter à des appareils de passerelle Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 10/15/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: 35bb17a5cdfcc6aff4a513a594a08283ab1f1305
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96437029"
---
# <a name="connect-a-downstream-device-to-an-azure-iot-edge-gateway"></a>Connecter un appareil en aval à une passerelle Azure IoT Edge

Cet article explique comment établir une connexion approuvée entre des appareils en aval et des passerelles transparentes IoT Edge. Dans un scénario de passerelle transparente, un ou plusieurs appareils peuvent transmettre leurs messages via un appareil de passerelle qui gère la connexion à IoT Hub.

La configuration d’une connexion de passerelle transparente s’effectue en trois grandes étapes. Cet article aborde la troisième étape :

1. Configurez l’appareil de passerelle en tant que serveur pour permettre aux appareils en aval de s’y connecter en toute sécurité. Configurez la passerelle pour recevoir des messages des appareils en aval et les acheminer vers la destination qui convient. Ces étapes sont décrites dans [Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md).
2. Créez une identité d’appareil pour l’appareil en aval afin qu’il puisse s’authentifier sur IoT Hub. Configurez l’appareil en aval pour envoyer des messages par le biais de l’appareil de passerelle. Ces étapes sont décrites dans [Authentifier un appareil en aval auprès d'Azure IoT Hub](how-to-authenticate-downstream-device.md).
3. **Connectez l’appareil en aval à l’appareil de passerelle et commencez à envoyer des messages.**

Cet article présente les concepts de base des connexions d'appareils en aval et vous guide dans la configuration de vos appareils en aval de la façon suivante :

* Présente le protocole TLS (Transport Layer Security) et les principes de base des certificats.
* Présente le fonctionnement des bibliothèques TLS sur différents systèmes d’exploitation et comment chaque système d’exploitation gère les certificats.
* Présente différents exemples Azure IoT dans plusieurs langues pour vous aider à démarrer.

Dans cet article, les termes *passerelle* et *passerelle IoT Edge* font référence à un appareil IoT Edge configuré comme passerelle transparente.

## <a name="prerequisites"></a>Prérequis

* Mettez le fichier de certificat d'autorité de certification racine utilisé pour générer le certificat d'autorité de certification d'appareil dans [Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md) à la disposition de votre appareil en aval. Celui-ci se sert de ce certificat pour valider l’identité de l’appareil de passerelle. Si vous avez utilisé les certificats de démonstration, le certificat d’autorité de certification racine est appelé **azure-iot-test-only.root.ca.cert.pem**.
* Utilisez la chaîne de connexion modifiée qui pointe vers l’appareil de passerelle, comme expliqué dans [Authentifier un appareil en aval auprès d’Azure IoT Hub](how-to-authenticate-downstream-device.md).

## <a name="prepare-a-downstream-device"></a>Préparer un appareil en aval

Un appareil en aval est une application ou une plateforme dont l’identité a été créée avec le service cloud Azure IoT Hub. Dans de nombreux cas, ces applications utilisent [Azure IoT device SDK](../iot-hub/iot-hub-devguide-sdks.md). Un appareil en aval peut même être une application fonctionnant sur l’appareil de passerelle IoT Edge proprement dit. Toutefois, un autre appareil IoT Edge ne peut pas se trouver en aval d’une passerelle IoT Edge.

>[!NOTE]
>Les appareils IoT inscrits auprès d'IoT Hub peuvent utiliser des [jumeaux de module](../iot-hub/iot-hub-devguide-module-twins.md) pour isoler des processus, matériels ou fonctions différents sur un même appareil. Les passerelles IoT Edge prennent en charge les connexions de modules en aval à l’aide de l’authentification par clé symétrique, mais pas à l’aide de l’authentification par certificat X.509.

Pour connecter un appareil en aval à une passerelle Azure IoT Edge, deux éléments sont nécessaires :

* Un appareil ou une application configurés avec une chaîne de connexion d’appareil IoT Hub ajoutée avec les informations pour se connecter à la passerelle.

    Cette étape a été effectuée dans l’article précédent, [Authentifier un appareil en aval auprès d’Azure IoT Hub](how-to-authenticate-downstream-device.md#retrieve-and-modify-connection-string).

* L’appareil ou l’application doit approuver le certificat de l’**autorité de certification racine** de la passerelle afin de valider les connexions TLS (Transport Layer Security) à l’appareil de passerelle.

    Cette étape est expliquée en détail dans la suite de cet article. Cette étape peut être effectuée de deux manières : en installant le certificat de l’autorité de certification dans le magasin de certificats du système d’exploitation, ou (pour certains langages) en référençant le certificat au sein des applications à l’aide des SDK Azure IoT.

## <a name="tls-and-certificate-fundamentals"></a>Principes de base du protocole TLS et des certificats

Le défi qu’impose la connexion en toute sécurité d’appareils en aval à IoT Edge est le même que pour toute autre communication client/serveur sécurisé survenant sur Internet. Un client et un serveur de communiquent en toute sécurité sur Internet à l’aide du protocole [Transport layer security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). Le protocole TLS est généré à l’aide de constructions [Public key infrastructure (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure) standard appelées certificats. TLS est une spécification souvent utilisée, qui convient à de nombreuses situations impliquant la sécurisation de deux points de terminaison. Cette section explique comment connecter des appareils à une passerelle IoT Edge de façon sécurisée.

Lorsqu’un client se connecte à un serveur, le serveur présente une chaîne de certificats, appelée *chaîne de certificats du serveur*. Une chaîne de certificats comprend généralement un certificat d’autorité de certification racine, un ou plusieurs certificats d’autorité de certification intermédiaires, et enfin le certificat du serveur lui-même. Un client établit une relation de confiance avec un serveur en vérifiant par chiffrement l’intégralité de la chaîne de certificats du serveur. Cette validation côté client de la chaîne de certificat du serveur est appelée *validation de la chaîne du serveur*. Le client vérifie que le service possède la clé privée associée au certificat du serveur, dans le cadre d’un processus appelé *preuve de possession*. La combinaison d’une validation de chaîne de serveur et d’une preuve de possession est appelée *authentification du serveur*. Pour valider une chaîne de certificats du serveur, un client a besoin d’une copie du certificat de l’autorité de certification racine qui a servi à créer (ou à émettre) le certificat du serveur. Normalement, lors de la connexion à des sites web, un navigateur est préconfiguré avec des certificats d’autorité de certification couramment utilisés afin de garantir au client un processus transparent.

Lorsqu’un appareil se connecte à Azure IoT Hub, l’appareil est client et le service cloud IoT Hub est le serveur. Le service cloud IoT Hub s’appuie sur un certificat d’autorité de certification racine appelé **Baltimore CyberTrust Root**, disponible publiquement et largement utilisé. Comme le certificat d’autorité de certification IoT Hub est déjà installé sur la plupart des appareils, de nombreuses implémentations TLS (OpenSSL, Schannel, LibreSSL) l’utilisent automatiquement lors de la validation du certificat du serveur. Cela étant, un appareil qui se connecte avec succès à IoT Hub peut rencontrer des problèmes en tentant de se connecter à une passerelle IoT Edge.

Lorsqu’un appareil se connecte à une passerelle IoT Edge, l’appareil en aval est le client et l’appareil de passerelle est le serveur. Azure IoT Edge vous permet de générer des chaînes de certificats de passerelle adaptées à vos besoins. Vous pouvez choisir d'utiliser un certificat d'autorité de certification publique, comme Baltimore, ou un certificat d'autorité de certification racine auto-signé (ou développé en interne). Les certificats d’autorité de certification publique entraînent souvent un coût et sont par conséquent généralement utilisés dans des scénarios de production. Les certificats d’autorité de certification auto-signés sont plus adaptés au développement et aux tests. Si vous utilisez les certificats de démonstration, il s'agit de certificats d'autorité de certification racine auto-signés.

Lorsque vous utilisez un certificat d’autorité de certification racine auto-signé pour une passerelle IoT Edge, ce certificat doit être installé sur tous les appareils en aval qui tentent de se connecter à la passerelle.

![Configuration du certificat de la passerelle](./media/how-to-create-transparent-gateway/gateway-setup.png)

Pour en savoir plus sur les certificats IoT Edge et certaines implications en matière de production, consultez la section [Détails sur l’utilisation d’un certificat IoT Edge](iot-edge-certs.md).

## <a name="provide-the-root-ca-certificate"></a>Fournir le certificat d’autorité de certification racine

Pour vérifier les certificats de l’appareil de passerelle, l’appareil en aval a besoin de sa propre copie du certificat d’autorité de certification racine. Si vous avez utilisé les scripts fournis dans le dépôt Git IoT Edge pour créer des certificats de test, le certificat d’autorité de certification racine est appelé **azure-iot-test-only.root.ca.cert.pem**. Si vous ne l’avez pas déjà fait dans le cadre d’autres étapes de préparation des appareils en aval, vous pouvez déplacer ce fichier de certificat vers n’importe quel répertoire de votre appareil en aval. Vous pouvez utiliser un service comme [Azure Key Vault](../key-vault/index.yml) ou une fonction comme [SCP (Secure Copy Protocol)](https://www.ssh.com/ssh/scp/) pour déplacer le fichier de certificat.

## <a name="install-certificates-in-the-os"></a>Installer des certificats sur le système d’exploitation

Une fois que le certificat d'autorité de certification racine se trouve sur l'appareil en aval, vous devez vérifier que les applications qui se connectent à la passerelle ont accès au certificat.

L’installation du certificat d’autorité de certification racine dans le magasin de certificats du système d’exploitation permet généralement à la plupart des applications d’utiliser le certificat d’autorité de certification racine. Il existe quelques exceptions, notamment les applications NodeJS, qui n’utilisent pas le magasin de certificats du système d’exploitation, mais plutôt le magasin de certificats interne du runtime Node. Si vous ne pouvez pas installer le certificat au niveau du système d’exploitation, passez directement à [Utiliser des certificats avec les kits de développement logiciel Azure IoT](#use-certificates-with-azure-iot-sdks).

### <a name="ubuntu"></a>Ubuntu

Les commandes suivantes montrent un exemple d’installation d’un certificat d’autorité de certification sur un hôte Ubuntu. Cet exemple suppose que vous utilisez le certificat **azure-iot-test-only.root.ca.cert.pem** tiré des articles sur les prérequis et que vous avez copié le certificat dans un emplacement de l’appareil en aval.

```bash
sudo cp <path>/azure-iot-test-only.root.ca.cert.pem /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
sudo update-ca-certificates
```

Vous devez voir un message indiquant « Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done. » (Mise à jour des certificats dans /etc/ssl/certs... 1 ajouté, 0 supprimé ; terminé.)

### <a name="windows"></a>Windows

Les étapes suivantes montrent un exemple d’installation d’un certificat d’autorité de certification sur un hôte Windows. Cet exemple suppose que vous utilisez le certificat **azure-iot-test-only.root.ca.cert.pem** tiré des articles sur les prérequis et que vous avez copié le certificat dans un emplacement de l’appareil en aval.

Vous pouvez installer des certificats en utilisant l’applet de commande PowerShell [Import-Certificate](/powershell/module/pkiclient/import-certificate) en tant qu’administrateur :

```powershell
import-certificate  <file path>\azure-iot-test-only.root.ca.cert.pem -certstorelocation cert:\LocalMachine\root
```

Vous pouvez aussi installer des certificats avec l’utilitaire **certlm** :

1. Dans le menu Démarrer, recherchez et sélectionnez **Manage computer certificates** (Gérer les certificats d’ordinateur). Un utilitaire appelé **certlm** s’ouvre.
2. Accédez à **Certificats - Ordinateur local** > **Autorités de certification racines de confiance**.
3. Cliquez avec le bouton droit sur **Certificats**, puis sélectionnez **Toutes les tâches** > **Importer**. L’Assistant Importation de certificat devrait s’ouvrir.
4. Suivez les étapes indiquées et importez le fichier de certificat `<path>/azure-iot-test-only.root.ca.cert.pem`. Une fois terminé, vous devez voir un message « Importation réussie ».

Vous pouvez également installer des certificats par programmation à l’aide de l’API .NET, comme indiqué dans l’exemple .NET plus loin dans cet article.

En général, les applications utilisent la pile TLS fournie par Windows et appelée [Schannel](/windows/desktop/com/schannel) pour se connecter en toute sécurité via TLS. Schannel *requiert* l’installation de tous les certificats dans le magasin de certificats Windows avant de tenter d’établir une connexion TLS.

## <a name="use-certificates-with-azure-iot-sdks"></a>Utiliser des certificats avec les kits de développement logiciel Azure IoT

Cette section décrit comment les kits Azure IoT SDK se connectent à un appareil IoT Edge à l’aide de simples exemples d’applications. Tous les exemples visent à se connecter au client de l’appareil et à envoyer des messages de télémétrie à la passerelle, avant de fermer la connexion et de quitter le programme.

Avant d’utiliser des exemples au niveau de l’application, préparez les deux éléments suivants :

* La chaîne de connexion IoT Hub de votre appareil en aval modifiée de manière à pointer vers l’appareil de passerelle, et tous les certificats nécessaires pour authentifier votre appareil en aval dans IoT Hub. Pour plus d’informations, consultez [Authentifier un appareil en aval auprès d’Azure IoT Hub](how-to-authenticate-downstream-device.md).

* Le chemin complet vers le certificat d’autorité de certification racine que vous avez copié et enregistré quelque part sur votre appareil en aval.

    Par exemple : `<path>/azure-iot-test-only.root.ca.cert.pem`.

### <a name="nodejs"></a>NodeJS

Cette section fournit un exemple d’application pour se connecter d’un client d’appareil Azure IoT NodeJS à une passerelle IoT Edge. Pour les applications Node.js, vous devez installer le certificat d’autorité de certification racine au niveau de l’application, comme indiqué ici. Les applications NodeJS n’utilisent pas le magasin de certificats du système.

1. Récupérez l’exemple pour **edge_downstream_device.js** à partir du [référentiel d’exemples Azure IoT device SDK pour Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples).
2. Assurez-vous que vous remplissez toutes les conditions préalables pour exécuter l’exemple en passant en revue le fichier **readme.md**.
3. Dans le fichier edge_downstream_device.js, mettez à jour les variables **connectionString** et **edge_ca_cert_path**.
4. Consultez la documentation du Kit de développement logiciel pour obtenir des instructions sur la façon d’exécuter l’exemple sur votre appareil.

Pour comprendre l’exemple que vous exécutez, l’extrait de code suivant explique comment le SDK client lit le fichier de certificat et l’utilise pour établir une connexion TLS sécurisée :

```javascript
// Provide the Azure IoT device client via setOptions with the X509
// Edge root CA certificate that was used to setup the Edge runtime
var options = {
    ca : fs.readFileSync(edge_ca_cert_path, 'utf-8'),
};
```

### <a name="net"></a>.NET

Cette section présente un exemple d’application permettant de connecter un client d’appareil Azure IoT .NET à une passerelle IoT Edge. Toutefois, les applications .NET peuvent automatiquement utiliser les certificats installés dans le magasin de certificats du système sur des hôtes Linux et Windows.

1. Récupérez l’exemple pour **EdgeDownstreamDevice** à partir du [dossier d’exemples IoT Edge .NET](https://github.com/Azure/iotedge/tree/master/samples/dotnet/EdgeDownstreamDevice).
2. Assurez-vous que vous remplissez toutes les conditions préalables pour exécuter l’exemple en passant en revue le fichier **readme.md**.
3. Dans le fichier **Properties / launchSettings.json**, mettez à jour les variables **DEVICE_CONNECTION_STRING** et **CA_CERTIFICATE_PATH**. Si vous souhaitez utiliser le certificat installé dans le magasin de certificats de confiance sur le système hôte, laissez vide cette variable.
4. Consultez la documentation du Kit de développement logiciel pour obtenir des instructions sur la façon d’exécuter l’exemple sur votre appareil.

Pour installer par programmation un certificat approuvé dans le magasin de certificats via une application .NET, reportez-vous à la fonction **InstallCACert()** dans le fichier **EdgeDownstreamDevice / Program.cs**. Cette opération est idempotente et peut donc être exécutée plusieurs fois avec les mêmes valeurs sans aucune autre incidence.

### <a name="c"></a>C

Cette section présente un exemple d’application permettant de connecter un client d’appareil Azure IoT C à une passerelle IoT Edge. Le Kit de développement logiciel C peut fonctionner avec de nombreuses bibliothèques TLS, y compris OpenSSL et Schannel WolfSSL. Pour plus d’informations, consultez le [Kit de développement logiciel (SDK) C Azure IoT](https://github.com/Azure/azure-iot-sdk-c).

1. Récupérez l’application **iotedge_downstream_device_sample** à partir des [exemples Azure IoT device SDK pour C](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples).
2. Assurez-vous que vous remplissez toutes les conditions préalables pour exécuter l’exemple en passant en revue le fichier **readme.md**.
3. Dans le fichier iotedge_downstream_device_sample.c, mettez à jour les variablest **connectionString** et **edge_ca_cert_path**.
4. Consultez la documentation du Kit de développement logiciel pour obtenir des instructions sur la façon d’exécuter l’exemple sur votre appareil.


Azure IoT device SDK pour C fournit une option permettant d’inscrire un certificat d’autorité de certification lors de la configuration du client. Cette opération n’installe pas le certificat n’importe où, mais utilise plutôt un format de chaîne du certificat en mémoire. Le certificat enregistré est transmis à la pile TLS sous-jacente lors de l’établissement d’une connexion.

```C
(void)IoTHubDeviceClient_SetOption(device_handle, OPTION_TRUSTED_CERT, cert_string);
```

>[!NOTE]
> La méthode d’inscription du certificat d’une autorité de certification lors de la configuration du client peut changer si vous utilisez une bibliothèque ou un package [géré(e)](https://github.com/Azure/azure-iot-sdk-c#packages-and-libraries). Par exemple, la [bibliothèque IDE Arduino](https://github.com/azure/azure-iot-arduino) nécessite l’ajout du certificat de l’autorité de certification à un tableau de certificats défini dans un fichier global [certs.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/certs/certs.c), plutôt que l’utilisation de l’opération `IoTHubDeviceClient_LL_SetOption`.  

Sur les hôtes Windows, si vous n’utilisez pas OpenSSL ou une autre bibliothèque TLS, le Kit de développement utilise par défaut Schannel. Pour que Schannel fonctionne, le certificat d’autorité de certification racine IoT Edge doit être installé dans le magasin de certificats Windows, qui n’est défini à l’aide de l’opération `IoTHubDeviceClient_SetOption`.

### <a name="java"></a>Java

Cette section présente un exemple d’application permettant de connecter un client d’appareil Azure IoT Java à une passerelle IoT Edge.

1. Récupérez l’exemple pour **Send-event** à partir des [exemples Azure IoT device SDK pour Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples).
2. Assurez-vous que vous remplissez toutes les conditions préalables pour exécuter l’exemple en passant en revue le fichier **readme.md**.
3. Consultez la documentation du Kit de développement logiciel pour obtenir des instructions sur la façon d’exécuter l’exemple sur votre appareil.

### <a name="python"></a>Python

Cette section présente un exemple d’application permettant de connecter un client d’appareil Azure IoT Python à une passerelle IoT Edge.

1. Récupérez l’exemple pour **send_message_downstream** à partir des [exemples d’Azure IoT device SDK pour Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios).
2. Définissez les variables d’environnement `IOTHUB_DEVICE_CONNECTION_STRING` et `IOTEDGE_ROOT_CA_CERT_PATH` comme indiqué dans les commentaires du script Python.
3. Consultez la documentation du Kit de développement logiciel (SDK) pour obtenir des instructions supplémentaires sur la façon d’exécuter l’exemple sur votre appareil.

## <a name="test-the-gateway-connection"></a>Test de connexion à la passerelle

Utilisez cet exemple de commande sur l'appareil en aval pour le tester et vérifier qu'il peut se connecter à l'appareil de passerelle :

```cmd/sh
openssl s_client -connect mygateway.contoso.com:8883 -CAfile <CERTDIR>/certs/azure-iot-test-only.root.ca.cert.pem -showcerts
```

Cette commande teste les connexions via MQTTS (port 8883). Si vous utilisez un autre protocole, ajustez la commande selon les besoins pour AMQPS (5671) ou HTTPS (433)

La sortie de cette commande peut être longue, car elle comprend des informations sur tous les certificats de la chaîne. Si votre connexion réussit, vous verrez une ligne telle que celle-ci `Verification: OK` ou celle-ci `Verify return code: 0 (ok)`.

![Vérifier la connexion à la passerelle](./media/how-to-connect-downstream-device/verification-ok.png)

## <a name="troubleshoot-the-gateway-connection"></a>Résoudre les problèmes de connexion à la passerelle

Si la connexion entre votre appareil de nœud terminal et son appareil de passerelle est intermittente, essayez les étapes suivantes pour résoudre le problème.

1. Le nom d’hôte de passerelle de la chaîne de connexion est-il identique à celui qui figure dans le fichier config.yaml IoT Edge de l’appareil de passerelle ?
2. Le nom d’hôte de passerelle peut-il être résolu en une adresse IP ? Vous pouvez résoudre les problèmes de connexion intermittente à l’aide d’un DNS ou en ajoutant une entrée de fichier hôte sur l’appareil de nœud terminal.
3. Les ports de communication de votre pare-feu sont-ils ouverts ? Une communication basée sur le protocole utilisé (MQTTS:8883/AMQPS:5671/HTTPS:433) doit pouvoir être établie entre l’appareil en aval et la passerelle transparente IoT Edge.

## <a name="next-steps"></a>Étapes suivantes

Guide pratique pour étendre les [fonctionnalités hors connexion](offline-capabilities.md) IoT Edge aux appareils en aval.