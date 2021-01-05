---
title: Utiliser les outils du kit de développement logiciel (SDK) Service IoT Hub Device Provisioning
description: Ce document passe en revue les outils fournis dans les SDK du service Azure IoT Hub Device Provisioning (DPS) pour le développement.
author: wesmc7777
ms.author: wesmc
ms.date: 04/09/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: 984d38752df93f233c6d87458e3c9ba713696177
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967242"
---
# <a name="how-to-use-tools-provided-in-the-sdks-to-simplify-development-for-provisioning"></a>Comment utiliser les outils fournis dans les SDK pour simplifier le développement pour l’approvisionnement
Le service IoT Hub Device Provisioning simplifie le processus d’[approvisionnement](about-iot-dps.md#provisioning-process) en prenant en charge l’activité d’une manière transparente, ponctuelle, sécurisée et évolutive.  Une attestation de sécurité sous la forme d’un certificat X.509 ou un module de plateforme sécurisée (TPM) est requise.  Microsoft collabore également avec d’[autres partenaires fournisseurs de matériel de sécurité](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) afin d’améliorer la fiabilité de la sécurisation des déploiements IoT. La compréhension des exigences de sécurité du matériel est un processus qui peut s’avérer complexe pour les développeurs. Un ensemble de SDK du service Azure IoT Provisioning est mis à disposition des développeurs, qui recourent à une couche de commodité pour l’écriture des clients interagissant avec le service de provisionnement. Les kits de développement logiciel fournissent également des exemples de scénarios communs ainsi qu’un ensemble d’outils simplifiant l’attestation de sécurité dans le développement.

## <a name="trusted-platform-module-tpm-simulator"></a>Simulateur de module de plateforme sécurisée (TPM)
Un module [TPM](./concepts-service.md#attestation-mechanism) peut faire référence à un standard de stockage sécurisé pour les clés utilisées dans l’authentification de la plateforme ou à l’interface d’E/S utilisée pour interagir avec les modules qui implémentent le standard. Les modules TPM peuvent exister sous forme de matériel distinct, de matériel intégré, basés sur un microprogramme ou basés sur un logiciel.  En production, le module TPM est hébergé sur l’appareil, en tant que matériel discret, intégré ou sur microprogramme. Durant la phase de test, un simulateur TPM sur logiciel est fourni aux développeurs.  Actuellement, ce simulateur est disponible uniquement pour le développement sur plateforme Windows.

Les étapes de l’utilisation du simulateur TPM sont les suivantes :
1. [Préparez l’environnement de développement](./quick-enroll-device-x509-java.md) et clonez le référentiel GitHub :
   ```
   git clone https://github.com/Azure/azure-iot-sdk-java.git
   ```
2. Accédez au dossier du simulateur TPM, sous ```azure-iot-sdk-java/provisioning/provisioning-tool/tpm-simulator/```.
3. Exécutez le fichier Simulator.exe avant d’exécuter toute application cliente pour l’approvisionnement de l’appareil.
4. Laissez le simulateur s’exécuter en arrière-plan tout au long du processus d’approvisionnement afin d’obtenir l’ID d’inscription et la paire de clés de type EK (Endorsement Key).  Les deux valeurs sont valides uniquement pour une instance de l’exécution.

## <a name="x509-certificate-generator"></a>Générateur de certificats X.509
Utilisez des [certificats X.509](./concepts-x509-attestation.md#x509-certificates) comme mécanisme d’attestation pour mettre à l’échelle la production et simplifier le provisionnement des appareils.  Il existe [différentes manières](../iot-hub/iot-hub-x509ca-overview.md#how-to-get-an-x509-ca-certificate) d’obtenir un certificat X.509 :
* Pour un environnement de production, nous vous recommandons d’acheter un certificat d’autorité de certification X.509 après d’une autorité de certification racine publique.
* Pour un environnement de test, vous pouvez générer un certificat racine X.509 ou une chaîne de certificats X.509 par les biais suivant :
    * OpenSSL : vous pouvez utiliser des scripts pour la génération de certificats :
        * [Node.JS](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/tools)
        * [PowerShell ou Bash](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)
        
    * Émulateur DICE (Device Identity Composition Engine) : DICE peut être utilisé pour l’identité et l’attestation du périphérique de chiffrement basées sur le protocole TLS et les certificats clients X.509.  [En savoir](https://www.microsoft.com/research/publication/device-identity-dice-riot-keys-certificates/) plus sur l’identité du périphérique avec DICE.

### <a name="using-x509-certificate-generator-with-dice-emulator"></a>Utilisation du générateur de certificats X.509 avec l’émulateur DICE
Les SDK fournissent un générateur de certificats X.509 avec l’émulateur DICE, dans le [SDK Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-tools/provisioning-x509-cert-generator).  Ce générateur fonctionne sur plusieurs plateformes.  Le certificat généré peut être utilisé pour le développement dans d’autres langues.

Actuellement, l’émulateur DICE génère un certificat racine, un certificat intermédiaire, un certificat feuille et une clé privée associée.  Toutefois, le certificat racine ou le certificat intermédiaire ne peuvent pas être utilisés pour la signature d’un certificat feuille séparé.  Si vous souhaitez tester un scénario d’inscription de groupe au sein duquel un certificat de signature est utilisé pour signer les certificats feuilles de plusieurs appareils, vous pouvez utiliser OpenSSL afin de produire une chaîne de certificats.

Pour générer un certificat X.509 à l’aide de ce générateur :
1. [Préparez l’environnement de développement](./quick-enroll-device-x509-java.md) et clonez le référentiel GitHub :
   ```
   git clone https://github.com/Azure/azure-iot-sdk-java.git
   ```
2. Modifiez la racine en azure-iot-sdk-java.
3. Exécutez ```mvn install -DskipTests=true``` pour télécharger l’ensemble des packages requis et compiler le SDK.
4. Accédez à la racine du générateur de certificats X.509 dans ```azure-iot-sdk-java/provisioning/provisioning-tools/provisioning-x509-cert-generator```.
5. Générez avec ```mvn clean install```.
6. Exécutez l’outil en utilisant les commandes suivantes :
   ```
   cd target
   java -jar ./provisioning-x509-cert-generator-{version}-with-deps.jar
   ```
7. Lorsque vous y êtes invité, vous pouvez également entrer un _nom commun_ pour vos certificats.
8. L’outil génère localement un **certificat client**, la **clé privée du certificat client**, le **certificat intermédiaire** et le **certificat racine**.

Le **certificat client** est le certificat feuille sur l’appareil.  Le **certificat client** et la **clé privée du certificat client** sont requis dans le client d’appareil. Selon la langue choisie, le mécanisme d’intégration dans l’application client peut être différent.  Pour plus d’informations, consultez les [Démarrages rapides](./quick-create-simulated-device-x509.md) sur la création d’un appareil simulé à l’aide de X.509.

Les certificats racine ou intermédiaire peuvent être utilisés pour créer un groupe d’inscriptions ou une inscription individuelle par [programmation](./how-to-manage-enrollments-sdks.md) ou à l’aide du [portail](./how-to-manage-enrollments.md).

## <a name="next-steps"></a>Étapes suivantes
* Développer à l’aide du [SDK Azure IoT]( https://github.com/Azure/azure-iot-sdks) pour Azure IoT Hub et le service Azure IoT Hub Device Provisioning