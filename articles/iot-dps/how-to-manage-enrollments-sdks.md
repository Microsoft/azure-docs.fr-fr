---
title: Gérer les inscriptions d’appareils à l’aide des Kits de développement logiciel (SDK) Azure DPS
description: Guide pratique pour gérer les inscriptions d’appareil dans le service IoT Hub Device Provisioning (DPS) à l’aide des SDK du service
author: robinsh
ms.author: robinsh
ms.date: 04/04/2018
ms.topic: conceptual
ms.service: iot-dps
ms.custom: fasttrack-edit, iot
services: iot-dps
ms.openlocfilehash: 45a2b7a64006ab6963290be3ac86a3a5d1e4916d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010977"
---
# <a name="how-to-manage-device-enrollments-with-azure-device-provisioning-service-sdks"></a>Guide pratique pour gérer les inscriptions d’appareil avec les SDK du service de provisionnement des appareils Azure
Une *inscription d’appareil* crée un enregistrement d’un appareil ou d’un groupe d’appareils susceptibles d’être inscrits au service de provisionnement des appareils à un moment donné. L’enregistrement contient la configuration initiale souhaitée pour le ou les appareils dans le cadre de cette inscription, y compris le hub IoT souhaité. Cet article explique comment gérer les inscriptions d’appareils pour votre service de provisionnement par programmation en utilisant les SDK du service de provisionnement des appareils Azure IoT.  Les SDK sont disponibles sur GitHub dans le même dépôt que les SDK Azure IoT.

## <a name="prerequisites"></a>Conditions préalables requises
* Récupérez la chaîne de connexion à partir de votre instance du service Device Provisioning.
* Récupérez des artefacts de sécurité des appareils pour le [mécanisme d’attestation](concepts-service.md#attestation-mechanism) utilisé :
    * [**Module de plateforme sécurisée (TPM)**](./concepts-tpm-attestation.md) :
        * Inscription individuelle : ID d’inscription et paire de clés de type EK du module de plateforme sécurisée à partir d’un appareil physique ou du simulateur TPM.
        * Le groupe d’inscriptions ne s’applique pas à l’attestation TPM.
    * [**X.509**](./concepts-service.md#attestation-mechanism) :
        * Inscription individuelle : [certificat feuille](./concepts-service.md#attestation-mechanism) à partir de l’appareil physique ou de l’émulateur [DICE](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) du SDK.
        * Groupe d’inscriptions : [certificat racine/de l’autorité de certification](./concepts-x509-attestation.md#root-certificate) ou [certificat intermédiaire](./concepts-x509-attestation.md#intermediate-certificate), utilisé pour générer le certificat d’appareil sur un appareil physique.  Il peut également être généré à partir de l’émulateur DICE du SDK.
* Les appels d’API peuvent différer d’un langage à l’autre. Pour plus d’informations, consultez les exemples fournis sur GitHub :
   * [Exemples de provisionnement de client de service en Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-samples)
   * [Exemples de provisionnement de client de service en Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/service/samples)
   * [Exemples de provisionnement de client de service en .NET](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/provisioning/service/samples)

## <a name="create-a-device-enrollment"></a>Créer une inscription d’appareil
Il existe deux façons de procéder à l’inscription de vos appareils auprès du service d’approvisionnement :

* Un **groupe d’inscriptions** est une entrée pour un groupe d’appareils partageant un mécanisme commun d’attestation de certificats X.509 signés par le [certificat racine](./concepts-x509-attestation.md#root-certificate) ou le [certificat intermédiaire](./concepts-x509-attestation.md#intermediate-certificate). Nous vous recommandons d’utiliser un groupe d’inscription pour un grand nombre d’appareils qui partagent une configuration initiale souhaitée ou pour des appareils destinés au même locataire. Notez que vous pouvez uniquement inscrire les appareils qui utilisent le mécanisme d’attestation X.509 comme *groupe d’inscriptions*. 

    Vous pouvez créer un groupe d’inscriptions avec les SDK en suivant ce flux de travail :

    1. Pour le groupe d’inscriptions, le mécanisme d’attestation utilise le certificat racine X.509.  Appelez l’API de SDK de service ```X509Attestation.createFromRootCertificate``` avec un certificat racine afin de créer l’attestation pour l’inscription.  Le certificat racine X.509 est fourni dans un fichier PEM ou sous forme de chaîne.
    1. Créez une variable ```EnrollmentGroup``` à l’aide de l’```attestation``` créée et d’un ```enrollmentGroupId``` unique.  Si vous le souhaitez, vous pouvez définir des paramètres tels que ```IoTHubHostName``` et ```ProvisioningStatus```.
    2. Appelez l’API de SDK de service ```createOrUpdateEnrollmentGroup``` dans votre application backend avec ```EnrollmentGroup``` pour créer un groupe d’inscriptions.

* L’**inscription individuelle** est une entrée permettant d’enregistrer un seul appareil. Les inscriptions individuelles peuvent utiliser des certificats X.509 ou des jetons SAP (à partir d’un module de plateforme sécurisée réel ou virtuel) comme mécanismes d’attestation. Nous vous recommandons d’utiliser des inscriptions individuelles pour les appareils qui nécessitent une configuration initiale unique ou pour les appareils qui peuvent uniquement utiliser des jetons SAP par le biais du module TPM ou du module TPM virtuel comme mécanisme d’attestation. Dans le cas d’inscriptions individuelles, vous pouvez spécifier l’ID de l’appareil IoT Hub souhaité.

    Vous pouvez créer une inscription individuelle avec les SDK en suivant ce flux de travail :
    
    1. Choisissez votre mécanisme ```attestation```, qui peut être TPM ou X.509.
        1. **TPM** : en utilisant comme entrée la paire de clés de type EK d’un appareil physique ou du simulateur TPM, vous pouvez appeler l’API de SDK de service ```TpmAttestation``` afin de créer l’attestation pour l’inscription. 
        2. **X.509** : en utilisant le certificat client comme entrée, vous pouvez appeler l’API de SDK de service ```X509Attestation.createFromClientCertificate``` afin de créer l’attestation pour l’inscription.
    2. Créez une variable ```IndividualEnrollment``` à l’aide de l’```attestation``` créée et d’un ```registrationId``` unique comme entrée, qui se trouve sur votre appareil ou qui est généré à partir du simulateur TPM.  Si vous le souhaitez, vous pouvez définir des paramètres tels que ```Device ID```, ```IoTHubHostName```et ```ProvisioningStatus```.
    3. Appelez l’API de SDK de service ```createOrUpdateIndividualEnrollment``` dans votre application backend avec ```IndividualEnrollment``` pour créer une inscription individuelle.

Une fois que vous avez créé une inscription, le service Device Provisioning retourne un résultat d’inscription. Ce flux de travail est illustré dans les exemples [mentionnés précédemment](#prerequisites).

## <a name="update-an-enrollment-entry"></a>Mettre à jour une entrée d’inscription

Après avoir créé une entrée d’inscription, vous pouvez mettre à jour l’inscription.  Les scénarios possibles incluent la mise à jour d’une propriété spécifique, la mise à jour de la méthode d’attestation ou la révocation de l’accès des appareils.  Il existe différentes API pour l’inscription individuelle et l’inscription de groupe, mais pas pour le mécanisme d’attestation.

Vous pouvez mettre à jour une entrée d’inscription en suivant ce flux de travail :
* **Inscription individuelle** :
    1. Dans un premier temps, obtenez la dernière inscription auprès du service de provisionnement avec l’API de SDK de service ```getIndividualEnrollment```.
    2. Modifiez le paramètre de la dernière inscription selon les besoins. 
    3. À l’aide de la dernière inscription, appelez l’API de SDK de service ```createOrUpdateIndividualEnrollment``` pour mettre à jour votre entrée d’inscription.
* **Inscription de groupe** :
    1. Dans un premier temps, obtenez la dernière inscription auprès du service de provisionnement avec l’API de SDK de service ```getEnrollmentGroup```.
    2. Modifiez le paramètre de la dernière inscription selon les besoins.
    3. À l’aide de la dernière inscription, appelez l’API de SDK de service ```createOrUpdateEnrollmentGroup``` pour mettre à jour votre entrée d’inscription.

Ce flux de travail est illustré dans les exemples [mentionnés précédemment](#prerequisites).

## <a name="remove-an-enrollment-entry"></a>Supprimer une entrée d’inscription

* Vous pouvez supprimer une **inscription individuelle** en appelant l’API de SDK de service ```deleteIndividualEnrollment``` avec ```registrationId```.
* Vous pouvez supprimer une **inscription de groupe** en appelant l’API de SDK de service ```deleteEnrollmentGroup``` avec ```enrollmentGroupId```.

Ce flux de travail est illustré dans les exemples [mentionnés précédemment](#prerequisites).

## <a name="bulk-operation-on-individual-enrollments"></a>Opération en bloc sur des inscriptions individuelles

Vous pouvez effectuer une opération en bloc pour créer, mettre à jour ou supprimer plusieurs inscriptions individuelles en suivant ce flux de travail :

1. Créez une variable qui contient plusieurs ```IndividualEnrollment```.  L’implémentation de cette variable varie d’un langage à l’autre.  Pour plus d’informations, examinez l’exemple d’opération en bloc sur GitHub.
2. Appelez l’API de SDK de service ```runBulkOperation``` avec un ```BulkOperationMode``` pour l’opération souhaitée et votre variable représentant les inscriptions individuelles. Quatre modes sont pris en charge : create, update, updateIfMatchEtag et delete.

Une fois l’opération correctement effectuée, le service de provisionnement des appareils retourne un résultat d’opération en bloc.

Ce flux de travail est illustré dans les exemples [mentionnés précédemment](#prerequisites).