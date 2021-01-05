---
title: Service IoT Hub Device Provisioning - Rôles et opérations
description: Cet article fournit une vue d’ensemble conceptuelle des opérations et des rôles impliqués lors du développement d’une solution IoT à l’aide du service de provisionnement des appareils (DPS) IoT.
author: wesmc7777
ms.author: wesmc
ms.date: 09/14/2020
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotga
ms.openlocfilehash: 52c0dcf4bd9f7b2ca8928eab5272a6500cbb379d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532956"
---
# <a name="roles-and-operations"></a>Rôles et opérations

Les phases de développement d’une solution IoT peuvent s’étendre sur plusieurs semaines ou mois en raison de facteurs tels que les délais de fabrication, l’expédition, les procédures douanières et ainsi de suite. De plus, elles peuvent englober les activités de plusieurs rôles, étant donné les diverses entités impliquées. Cette rubrique examine plus en détail les différents rôles et opérations liés à chaque phase, puis montre le flux dans un diagramme de séquence. 

Le provisionnement impose également des conditions au fabricant de l’appareil, qui sont propres à l’activation du [mécanisme d’attestation](concepts-service.md#attestation-mechanism). Les opérations de fabrication peuvent également se produire indépendamment du planning des phases de provisionnement automatique, en particulier dans les cas où de nouveaux appareils sont fournis une fois que le provisionnement automatique a déjà été établi.

Dans la table des matières à gauche, vous trouverez une série de guides de démarrage rapide qui expliquent le provisionnement automatique par le biais d’une expérience pratique. Afin de simplifier le processus d’apprentissage, un logiciel est utilisé pour simuler un appareil physique pour l’inscription et l’enregistrement. Certains guides de démarrage rapide vous obligent à effectuer des opérations pour plusieurs rôles, notamment des opérations pour des rôles inexistants, en raison de la nature simulée de ces guides.

| Role | Opération | Description |
|------| --------- | ------------|
| Fabricant | Encoder l’URL d’inscription et l’identité | En fonction du mécanisme d’attestation utilisé, le fabricant est responsable de l’encodage des informations d’identités d’appareils et de l’URL d’inscription auprès du service Device Provisioning.<br><br>**Guides de démarrage rapide** : l’appareil étant simulé, il n’existe aucun rôle Fabricant. Pour plus d’informations sur la façon d’obtenir ces informations, qui sont utilisées pour le codage d’un exemple d’application d’inscription, consultez le rôle Développeur. |
| | Fournir l’identité d’appareil | Étant à l’origine des informations d’identités d’appareils, le fabricant est chargé de les transmettre à l’opérateur (ou à un agent désigné), ou de les inscrire directement auprès du service Device Provisioning par le biais des API.<br><br>**Guides de démarrage rapide** : l’appareil étant simulé, il n’existe aucun rôle Fabricant. Pour plus d’informations sur la façon d’obtenir l’identité de l’appareil, qui est utilisée pour inscrire un appareil simulé dans votre instance du service Device Provisioning, consultez le rôle Opérateur. |
| Opérateur | Configurer le provisionnement automatique | Cette opération correspond à la première phase de provisionnement automatique.<br><br>**Guides de démarrage rapide :**  : vous endossez le rôle d’opérateur et vous configurez le service Device Provisioning et les instances IoT Hub dans votre abonnement Azure. |
|  | Inscrire l’identité de l’appareil | Cette opération correspond à la deuxième phase de provisionnement automatique.<br><br>**Guides de démarrage rapide :**  : vous endossez le rôle d’opérateur et vous inscrivez votre appareil simulé auprès de votre instance du service Device Provisioning. L’identité de l’appareil est déterminée par la méthode d’attestation simulée dans le guide de démarrage rapide (TPM ou X.509). Pour plus d’informations sur l’attestation, consultez le rôle Développeur. |
| Service Device Provisioning,<br>IoT Hub | \<all operations\> | Pour une implémentation en production avec des appareils physiques et pour les guides de démarrage rapide avec des appareils simulés, ces rôles sont endossés par le biais des services IoT que vous configurez dans votre abonnement Azure. Les rôles/opérations fonctionnent exactement de la même manière, car les services IoT sont indifférents au fait qu’il s’agisse d’un provisionnement d’appareils physiques ou simulés. |
| Développeur | Générer/déployer le logiciel d’inscription | Cette opération correspond à la troisième phase de provisionnement automatique. Le développeur est chargé de générer et de déployer le logiciel d’inscription sur l’appareil à l’aide du kit SDK approprié.<br><br>**Guides de démarrage rapide :**  : l’exemple d’application d’inscription que vous créez simule un appareil réel, pour la plateforme/le langage de votre choix, qui s’exécute sur votre station de travail (au lieu de le déployer sur un appareil physique). L’application d’inscription effectue les mêmes opérations qu’en cas de déploiement sur un appareil physique. Vous spécifiez la méthode d’attestation (certificat TPM ou X.509), ainsi que l’URL d’inscription et « l’étendue de l’ID » de votre instance du service Device Provisioning. L’identité de l’appareil est déterminée par la logique d’attestation du SDK au moment de l’exécution, en fonction de la méthode que vous spécifiez : <ul><li>**Attestation TPM** : votre station de travail de développement exécute une [application de simulation de module de plateforme sécurisée](how-to-use-sdk-tools.md#trusted-platform-module-tpm-simulator). Une fois celle-ci en cours d’exécution, une application distincte est utilisée pour extraire la « paire de clés de type EK (Endorsement Key) » et l’« ID d’inscription » du module TPM en vue de les utiliser pour l’inscription de l’identité de l’appareil. La logique d’attestation du SDK utilise également le simulateur pendant l’inscription, afin de présenter un jeton SAP signé pour l’authentification et la vérification de l’inscription.</li><li>**Attestation X509** : vous utilisez un outil pour [générer un certificat](how-to-use-sdk-tools.md#x509-certificate-generator). Une fois le certificat généré, vous créez le fichier de certificat qui doit être utilisé lors de l’inscription. La logique d’attestation du SDK utilise également le certificat pendant l’inscription, afin de le présenter pour l’authentification et la vérification de l’inscription.</li></ul> |
| Appareil | Démarrer et inscrire | Cette opération correspond à la troisième phase de provisionnement automatique, assurée par le logiciel d’inscription d’appareil généré par le développeur. Pour plus d’informations, consultez le rôle Développeur. Lors du premier démarrage : <ol><li>L’application se connecte avec l’instance du service Device Provisioning, conformément à l’URL globale et à « l’étendue de l’ID » du service spécifiés pendant le développement.</li><li>Une fois connecté, l’appareil est authentifié par rapport à la méthode d’attestation et à l’identité spécifiées lors de l’inscription.</li><li>Une fois authentifié, l’appareil est inscrit auprès de l’instance IoT Hub spécifiée par l’instance du service de provisionnement.</li><li>Une fois l’inscription réussie, un ID d’appareil et un point de terminaison IoT Hub uniques sont retournés à l’application d’inscription pour communiquer avec IoT Hub.</li><li> À partir de là, l’appareil peut extraire son état de [jumeau d’appareil](~/articles/iot-hub/iot-hub-devguide-device-twins.md) initial pour la configuration, et lancer le processus de création de rapports des données de télémétrie.</li></ol>**Guides de démarrage rapide** : l’appareil étant simulé, le logiciel d’inscription s’exécute sur votre station de travail de développement.|

Le diagramme suivant récapitule les rôles et l’ordre des opérations lors du provisionnement automatique d’un appareil :
<br><br>
[![Séquence d’approvisionnement automatique d’un appareil](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png)](./media/concepts-auto-provisioning/sequence-auto-provision-device-vs.png#lightbox) 

> [!NOTE]
> Éventuellement, le fabricant peut aussi effectuer l’opération « Inscrire l’identité de l’appareil » à l’aide des API du service Device Provisioning (plutôt que par le biais de l’Opérateur). Pour une discussion détaillée de cette mise en séquence et des informations supplémentaires, consultez la [vidéo sur l’inscription d’appareils sans intervention avec Azure IoT](https://youtu.be/cSbDRNg72cU?t=2460) (à partir du marqueur 41:00).

## <a name="roles-and-azure-accounts"></a>Rôles et comptes Azure

La façon dont chaque rôle est mappé à un compte Azure dépend du scénario, et il existe un bon nombre de scénarios qui peuvent être impliqués. Les modèles courants ci-dessous doivent vous donner une compréhension générale de la façon dont les rôles sont généralement mappés à un compte Azure.

#### <a name="chip-manufacturer-provides-security-services"></a>Un fabricant de puces fournit des services de sécurité

Dans ce scénario, le fabricant gère la sécurité pour les clients de niveau 1. Ce scénario peut être préféré par ces clients de niveau 1, car ils n’ont pas à gérer les détails de la sécurité. 

Le fabricant introduit les fonctionnalités de sécurité dans les modules de sécurité matériels (HSM). Cette sécurité peut inclure l’obtention par le fabricant de clés, de certificats, etc. auprès des clients potentiels qui ont déjà une configuration d’instances DPS et de groupes d’inscription. Le fabricant peut également générer ces informations de sécurité pour ses clients.

Dans ce scénario, deux comptes Azure peuvent être impliqués :

- **Compte #1** : Probablement partagé dans une certaine mesure entre les rôles d’opérateur et de développeur. Ce tiers peut acheter les puces HSM auprès du fabricant. Ces puces pointent vers les instances DPS associées au compte #1. Avec les inscriptions DPS, ce tiers peut louer des appareils à plusieurs clients de niveau 2 en reconfigurant les paramètres d’inscription des appareils dans DPS. Ce tiers peut également avoir des hubs IoT alloués pour des systèmes back-ends d’utilisateurs finaux avec lesquels s’interfacer afin d’accéder aux données de télémétrie des appareils, etc. Dans ce dernier cas, un deuxième compte peut ne pas être nécessaire.

- **Compte #2** : Les utilisateurs finaux et les clients de niveau 2 peuvent avoir leurs propres hubs IoT. Le tiers associé au compte #1 fait pointer les appareils loués vers le hub approprié dans ce compte. Cette configuration nécessite une liaison de DPS et des hubs IoT entre les comptes Azure, ce qui peut être effectué avec des modèles Azure Resource Manager.

#### <a name="all-in-one-oem"></a>OEM tout-en-un

Le fabricant peut être un « OEM tout-en-un », où un seul compte de fabricant serait nécessaire. Le fabricant gère la sécurité et le provisionnement de bout en bout.

Le fabricant peut fournir une application cloud pour les clients qui achètent des appareils. Cette application sert d’interface avec le hub IoT alloué par le fabricant.

Des distributeurs ou des machines à café automatiques sont des exemples pour ce scénario.




## <a name="next-steps"></a>Étapes suivantes

Nous vous conseillons de mettre cet article dans vos Favoris afin de pouvoir le consulter facilement à mesure que vous parcourez les guides de démarrage rapide de provisionnement automatique correspondants. 

Commencez par suivre le guide de démarrage rapide « Configuration du provisionnement automatique » qui convient le mieux à vos préférences d’outil de gestion. Il vous guidera tout au long de la phase de « Configuration du service » :

- [Configurer le provisionnement automatique avec Azure CLI](quick-setup-auto-provision-cli.md)
- [Configurer le provisionnement automatique avec le portail Azure](quick-setup-auto-provision.md)
- [Configurer le provisionnement automatique à l’aide d’un modèle Resource Manager](quick-setup-auto-provision-rm.md)

Ensuite, continuez avec le guide de démarrage rapide « Provisionner un appareil » qui convient le mieux à votre mécanisme d’attestation d’appareil et votre préférence en matière de langage/SDK de service Device Provisioning. Ce guide de démarrage rapide concerne les phases « Inscription d’appareils » et « Configuration et inscription d’appareils » : 

| Mécanisme d’attestation d’appareil | SDK/langage de guide de démarrage rapide | 
| ------------------------------- | -------------------- |
| Clé symétrique | [C](quick-create-simulated-device-symm-key.md)<br>[Java](quick-create-simulated-device-symmetric-key-java.md)<br>[Python](quick-create-device-symmetric-key-python.md) |
| Certificat X.509 | [C](quick-create-simulated-device-x509.md)<br>[Java](quick-create-simulated-device-x509-java.md)<br>[C#](quick-create-simulated-device-x509-csharp.md)<br>[Node.JS](quick-create-simulated-device-x509-node.md)<br>[Python](quick-create-simulated-device-x509-python.md) |
| Module de plateforme sécurisée (TPM) simulé | [C](quick-create-simulated-device.md)<br>[Java](quick-create-simulated-device-tpm-java.md)<br>[C#](quick-create-simulated-device-tpm-csharp.md)<br>[Python](quick-create-simulated-device-tpm-python.md) |




