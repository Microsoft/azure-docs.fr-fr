---
title: Vue d’ensemble de la sécurité de l’autorité de certification Azure IoT Hub X.509 | Microsoft Docs
description: Vue d’ensemble - guide pratique pour authentifier les appareils auprès d’IoT Hub à l’aide d’autorités de certification X.509.
author: eustacea
manager: arjmands
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/18/2017
ms.author: eustacea
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: System Architecture'
ms.openlocfilehash: 455f1f3e1c8181646fbe165d0f1aea6a2fb2fb13
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150709"
---
# <a name="device-authentication-using-x509-ca-certificates"></a>Authentification des appareils à l’aide de certificats d’autorité de certification X.509

Cet article décrit comment utiliser des certificats d’autorité de certification (CA) X.509 pour authentifier les appareils se connectant IoT Hub.  Cet article vous apprendra les éléments suivants :

* Guide pratique pour obtenir un certificat d’autorité de certification X.509
* Guide pratique pour inscrire le certificat de l’autorité de certification X.509 dans IoT Hub
* Comment signer des appareils à l’aide de certificats d’autorité de certification X.509
* Comment les appareils signés à l’aide d’une autorité de certification X.509 sont authentifiés

[!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

## <a name="overview"></a>Vue d’ensemble

La fonctionnalité d’autorité de certification X.509 permet l’authentification des appareils auprès d’IoT Hub à l’aide d’une autorité de certification. Elle simplifie considérablement le processus d’inscription initial des appareils et la logistique de la chaîne d’approvisionnement lors de la fabrication de l’appareil. [Pour en savoir plus, consultez cet article de scénario sur l’importance d’utiliser des certificats d’autorités de certification X.509](iot-hub-x509ca-concept.md) pour l’authentification des appareils.  Nous vous invitons à lire cet article de scénario avant de poursuivre car il explique l’objectif des étapes qui suivent.

## <a name="prerequisite"></a>Configuration requise

La fonctionnalité d’autorité de certification X.509 nécessite un compte IoT Hub.  [Découvrez comment créer une instance IoT Hub](quickstart-send-telemetry-dotnet.md) si vous n’en avez pas.

## <a name="how-to-get-an-x509-ca-certificate"></a>Guide pratique pour obtenir un certificat d’autorité de certification X.509

Le certificat de l’autorité de certification X.509 apparaît en haut de la chaîne des certificats pour chacun de vos appareils.  Vous pouvez acheter un certificat ou en créer un, selon la façon dont vous souhaitez l’utiliser.

Pour un environnement de production, nous vous recommandons d’acheter un certificat d’autorité de certification X.509 après d’une autorité de certification racine publique. En achetant un certificat d’autorité de certification, l’autorité de certification racine agit comme un tiers de confiance pour garantir la légitimité de vos appareils. Choisissez cette option si vos appareils feront partie d’un réseau IoT ouvert sur lequel ils interagiront avec des produits ou des services tiers.

Vous pouvez également créer une autorité de certification X.509 auto-signée à des fins de tests ou pour une utilisation sur des réseaux IoT fermés.

Quelle que soit la provenance de votre certificat d’autorité de certification X.509, veillez constamment à garder secrète et à protéger sa clé privée.  Cette condition est nécessaire pour établir une relation de confiance dans le cadre de l’authentification de l’autorité de certification X.509.

Découvrez comment [créer un certificat d’autorité de certification auto-signé](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) à des fins d’expérimentation tout au long de la description de cette fonctionnalité.

## <a name="sign-devices-into-the-certificate-chain-of-trust"></a>Signature d’appareils dans la chaîne de certificats de confiance

Le propriétaire d’un certificat d’autorité de certification X.509 peut signer par chiffrement une autorité de certification intermédiaire qui peut à son tour signer une autre autorité de certification intermédiaire, et ainsi de suite, jusqu’à ce que la dernière autorité de certification intermédiaire mette fin à ce processus en signant un appareil. Vous obtenez ainsi une chaîne de certificats en cascade appelée chaîne de certificats de confiance. En réalité, il s’agit d’une délégation d’approbation pour la signature d’appareils. Cette délégation est importante car elle établit une chaîne de responsabilités variable par chiffrement et évite le partage des clés de signature.

![img-generic-cert-chain-of-trust](./media/generic-cert-chain-of-trust.png)

Sur le certificat d’appareil (également appelé certificat feuille), le *Nom du sujet* doit être défini sur l’**ID d’appareil** (`CN=deviceId`) utilisé lors de l’inscription de l’appareil IoT dans le hub Azure IoT. Ce paramètre est requis pour l’authentification.

Découvrez ici comment [créer une chaîne de certificats](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) lors de la signature des appareils.

## <a name="how-to-register-the-x509-ca-certificate-to-iot-hub"></a>Guide pratique pour inscrire le certificat de l’autorité de certification X.509 dans IoT Hub

Inscrivez votre certificat d’autorité de certification X.509 dans IoT Hub, où il servira à authentifier vos appareils lors de l’inscription et de la connexion.  L’inscription du certificat d’autorité de certification X.509 est un processus en deux étapes, qui comprend le chargement d’un fichier de certificat et une preuve de possession.

Le processus de chargement implique le chargement d’un fichier contenant votre certificat.  Ce fichier ne doit jamais contenir toutes les clés privées.

La preuve de possession nécessite une vérification par chiffrement et un processus de réponse entre vous et IoT Hub.  Comme le contenu du certificat numérique est public et peut donc faire l’objet d’écoutes clandestines, IoT Hub souhaite s’assurer que vous possédez réellement le certificat d’autorité de certification.  Pour cela, il vous invite de façon aléatoire à vous connecter avec la clé privée correspondant au certificat d’autorité de certification.  Si vous avez gardé secrète et protégé la clé privée comme conseillé précédemment, vous êtes alors la seule personne à pouvoir effectuer cette étape. Dans cette méthode, la capacité à garder secrètes les clés privées est la base de la confiance.  Après la signature de vérification, effectuez cette étape en chargeant un fichier contenant les résultats.

Découvrez comment [enregistrer votre certificat d’autorité de certification](iot-hub-security-x509-get-started.md#register-x509-ca-certificates-to-your-iot-hub)

## <a name="how-to-create-a-device-on-iot-hub"></a>Guide pratique pour créer un appareil sur IoT Hub

Afin d’éviter l’emprunt d’identité de l’appareil, IoT Hub vous oblige à l’informer des appareils que vous utiliserez.  Pour cela, vous créez une entrée dans le registre des appareils d’IoT Hub.  Ce processus est automatique si vous utilisez le service IoT Hub [Device Provisioning](https://azure.microsoft.com/blog/azure-iot-hub-device-provisioning-service-preview-automates-device-connection-configuration/). 

Découvrez ici comment [créer manuellement un appareil dans IoT Hub](iot-hub-security-x509-get-started.md#create-an-x509-device-for-your-iot-hub).

Créer un appareil X.509 pour votre IoT Hub

## <a name="authenticating-devices-signed-with-x509-ca-certificates"></a>Authentification d’appareils signés avec des certificats d’autorité de certification X.509

Lorsque les appareils sont enregistrés à l’aide d’un certificat d’autorité de certificat X.509 et signés dans une chaîne de certificats de confiance, la dernière étape est l’authentification des appareils lorsqu’ils se connectent, même pour la première fois.  Lorsqu’un appareil signé à l’aide d’un certificat d’autorité de certification X.509 se connecte, il charge sa chaîne de certificats pour validation. La chaîne inclut tous les certificats intermédiaires d’autorité de certification et d’appareil.  Grâce à ces informations, IoT Hub authentifie l’appareil dans un processus en deux étapes.  IoT Hub valide par chiffrement la chaîne de certificats pour la cohérence interne, puis lance une procédure de vérification de la preuve de possession sur l’appareil.  IoT Hub authentifie l’appareil si celui-ci répond de façon correcte à la demande de preuve de possession.  Cette déclaration suppose que la clé privée de l’appareil est protégée et que seul l’appareil peut répondre correctement à cette demande.  Nous recommandons d’installer sur les appareils des puces sécurisées de type Hardware Secure Modules (HSM) afin de protéger les clés privées.

La connexion réussie d’un appareil à IoT Hub termine le processus d’authentification et constitue un bon indicateur d’une configuration correcte.

Découvrez ici comment [effectuer cette étape de connexion de l’appareil](iot-hub-security-x509-get-started.md#authenticate-your-x509-device-with-the-x509-certificates).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [l’importance de l’authentification par autorité de certification X.509](iot-hub-x509ca-concept.md) dans IoT.

Bien démarrer avec [Service IoT Hub Device Provisioning](../iot-dps/index.yml).