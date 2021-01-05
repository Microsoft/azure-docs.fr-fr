---
title: Tutoriel - Provisionner un appareil à l’aide du service Azure IoT Hub Device Provisioning
description: Ce tutoriel montre comment provisionner votre appareil sur un seul hub IoT à l’aide du service Azure IoT Hub Device Provisioning
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 876fd8260b64fba4d3d34a766b4259323c660b76
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968075"
---
# <a name="tutorial-provision-the-device-to-an-iot-hub-using-the-azure-iot-hub-device-provisioning-service"></a>Tutoriel : Approvisionner l’appareil sur un hub IoT avec le service IoT Hub Device Provisioning

Dans le didacticiel précédent, vous avez appris à configurer un appareil pour vous connecter à votre service Device Provisioning. Dans ce didacticiel, vous apprenez à utiliser ce service pour approvisionner votre appareil sur un seul IoT Hub à l’aide de l’approvisionnement automatique et de **_listes d’inscriptions_** . Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Inscrire l’appareil
> * Démarrer l’appareil
> * Vérifier que l’appareil est enregistré

## <a name="prerequisites"></a>Prérequis

Avant de continuer, assurez-vous de configurer votre appareil comme indiqué dans le didacticiel [Configurer un appareil à provisionner à l’aide du service IoT Hub Device Provisioning](./tutorial-set-up-device.md).

Si vous ne connaissez pas le processus de provisionnement automatique, révisez la présentation du [provisionnement](about-iot-dps.md#provisioning-process) avant de poursuivre.

<a id="enrolldevice"></a>
## <a name="enroll-the-device"></a>Inscrire l’appareil

Cette étape implique l’ajout des artefacts de sécurité uniques de l’appareil au service Device Provisioning. Ces artefacts de sécurité sont basés sur le [mécanisme d’attestation](concepts-service.md#attestation-mechanism) de l’appareil, comme suit :

- Pour les appareils TPM, vous avez besoin des éléments suivants :
    - La *paire de clés de type EK* qui est unique à chaque simulation ou processeur TPM, obtenue auprès du fournisseur de processeurs TPM.  Pour plus d’informations, consultez [Comprendre la paire de clés de type EK (Endorsement Key) du module de plateforme sécurisée](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770443(v=ws.11)).
    - *L’ID d’enregistrement* qui est utilisé pour identifier un appareil dans l’espace de noms ou l’étendue. Cet ID peut ou non être le même que l’ID de l’appareil. L’ID est obligatoire pour chaque appareil. Pour les appareils basés sur TPM, l’ID d’enregistrement peut être dérivé du module TPM lui-même, par exemple un hachage SHA-256 de la paire de clés de type EK TPM.

      [![Informations d’inscription pour le module TPM dans le portail](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png)](./media/tutorial-provision-device-to-hub/tpm-device-enrollment.png#lightbox)  

- Pour les appareils X.509, vous avez besoin des éléments suivants :
    - Le [certificat délivré à X.509](/windows/win32/seccertenroll/about-x-509-public-key-certificates) (processeur ou simulation), sous la forme d’un fichier *.pem* ou *.cer*. Pour les inscriptions individuelles, vous devez utiliser le *certificat signé* de chaque appareil de votre système X.509. Pour les groupes d’inscriptions, vous devez utiliser le *certificat racine*. 

      [![Ajouter une inscription individuelle pour l’attestation X.509 dans le portail](./media/tutorial-provision-device-to-hub/individual-enrollment.png)](./media/tutorial-provision-device-to-hub/individual-enrollment.png#lightbox)

Il existe deux façons d’inscrire l’appareil auprès du service Device Provisioning :

- **Groupe d’inscriptions** : représente un groupe d’appareils qui partagent un mécanisme d’attestation spécifique. Nous recommandons d’utiliser un groupe d’inscriptions pour un grand nombre d’appareils qui partagent une configuration initiale souhaitée ou pour des appareils destinés au même locataire. Pour plus d’informations sur l’attestation d’identité dans les groupes d’inscription, consultez [Sécurité](concepts-x509-attestation.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates).

    [![Ajouter une inscription de groupe pour l’attestation X.509 dans le portail](./media/tutorial-provision-device-to-hub/group-enrollment.png)](./media/tutorial-provision-device-to-hub/group-enrollment.png#lightbox)

- **Inscription individuelle** : représente une entrée d’un seul appareil pouvant être enregistré auprès du service Device Provisioning. Les inscriptions individuelles peuvent utiliser des certificats x509 ou des jetons SAP (dans un module de plateforme sécurisée réel ou virtuel) comme mécanismes d’attestation. Nous vous recommandons d’utiliser des inscriptions individuelles pour les appareils qui nécessitent une configuration initiale unique et pour les appareils qui peuvent uniquement utiliser des jetons SAP par le biais du TPM ou TPM virtuel comme mécanisme d’attestation. Dans le cas d’inscriptions individuelles, vous pouvez spécifier l’ID de l’appareil IoT Hub souhaité.

Vous inscrivez l’appareil auprès de votre instance Device Provisioning Service à l’aide des artefacts de sécurité requis en fonction du mécanisme d’attestation de l’appareil : 

1. Connectez-vous au portail Azure, cliquez sur le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre instance Device Provisioning Service.

2. Dans le panneau de résumé du service Device Provisioning, sélectionnez **Gérer les inscriptions**. Suivant la configuration de votre appareil, sélectionnez l’onglet **Inscriptions individuelles** ou **Groupes d’inscriptions**. Cliquez sur le bouton **Ajouter** en haut. Sélectionnez **TPM** ou **X.509** comme *mécanisme* d’attestation de l’identité, puis entrez les artefacts de sécurité appropriés, comme indiqué précédemment. Vous pouvez entrer un nouvel **ID d’appareil IoT Hub**. Cela fait, cliquez sur le bouton **Enregistrer**. 

3. Une fois l’appareil correctement inscrit, il doit apparaître comme suit dans le portail :

    ![Inscription TPM réussie dans le portail](./media/tutorial-provision-device-to-hub/tpm-enrollment-success.png)

Après l’inscription, le service d’approvisionnement attend que l’appareil démarre et s’y connecte plus tard. Au premier démarrage de votre appareil, la bibliothèque du Kit de développement logiciel (SDK) client interagit avec votre processeur pour extraire les artefacts de sécurité de l’appareil et vérifie l’inscription auprès de votre instance Device Provisioning Service. 

## <a name="start-the-iot-device"></a>Démarrer l’appareil IoT

Votre appareil IoT peut être un appareil réel ou un appareil simulé. Étant donné que l’appareil IoT est maintenant inscrit avec une instance de service Device Provisioning, l’appareil peut désormais démarrer et appeler le service d’approvisionnement pour être reconnu à l’aide du mécanisme d’attestation. Une fois l’appareil reconnu par le service d’approvisionnement, il est affecté à un hub IoT. 

Des exemples d’appareil simulé, avec des attestations TPM et X.509, sont inclus pour C, Java, C#, Node.js et Python. Par exemple, un appareil simulé utilisant un TPM et le [kit de développement logiciel (SDK) Azure IoT](https://github.com/Azure/azure-iot-sdk-c) suit le processus traité dans la section [Simuler la première séquence de démarrage d’un appareil](quick-create-simulated-device.md#simulate-first-boot-sequence-for-the-device). Le même appareil utilisant l’attestation de certificats X.509 fait référence à la section [Séquence de démarrage](quick-create-simulated-device-x509.md#simulate-first-boot-sequence-for-the-device).

Reportez-vous au [Guide de procédure pour le DevKit IoT MXChip](how-to-connect-mxchip-iot-devkit.md) comme exemple pour un appareil réel.

Démarrez l’appareil pour autoriser votre application cliente d’appareil à lancer l’inscription auprès de votre service Device Provisioning.  

## <a name="verify-the-device-is-registered"></a>Vérifier que l’appareil est enregistré

Une fois que votre appareil démarre, voici les actions qui doivent se produire :

1. L’appareil envoie une demande d’enregistrement à votre service Device Provisioning.
2. Pour les appareils TPM, le service Device Provisioning envoie une demande d’enregistrement à laquelle répond votre appareil. 
3. Une fois l’inscription réussie, le service Device Provisioning envoie l’URI du hub IoT, l’ID de l’appareil et la clé chiffrée à l’appareil. 
4. L’application cliente IoT Hub sur l’appareil peut alors se connecter à votre hub. 
5. Une fois la connexion au hub établie, l’appareil doit apparaître dans l’Explorateur **Appareils IoT** du hub IoT. 

    ![Connexion réussie au hub dans le portail](./media/tutorial-provision-device-to-hub/hub-connect-success.png)

Pour plus d’informations, consultez l’exemple de provisionnement de client d’appareil, [prov_dev_client_sample.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/samples/prov_dev_client_sample/prov_dev_client_sample.c). L’exemple illustre le provisionnementd’un appareil simulé à l’aide du module de plateforme sécurisée (TPM), de certificats X.509 et de clés symétriques. Pour obtenir des instructions pas à pas sur l’utilisation de cet exemple, consultez les guides de démarrage rapide d’attestation [TPM](./quick-create-simulated-device.md), [X.509](./quick-create-simulated-device-x509.md) et [Clé symétrique](./quick-create-simulated-device-symm-key.md).

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Inscrire l’appareil
> * Démarrer l’appareil
> * Vérifier que l’appareil est enregistré

Passez au didacticiel suivant pour apprendre à approvisionner plusieurs appareils sur des hubs à charge équilibrée. 

> [!div class="nextstepaction"]
> [Approvisionner des appareils sur des hubs IoT à charge équilibrée](./tutorial-provision-multiple-hubs.md)