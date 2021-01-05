---
title: Terminologie utilisée avec le service Azure IoT Hub Device Provisioning | Microsoft Docs
description: Décrit la terminologie courante utilisée avec le service Device Provisioning (DPS) et IoT Hub
author: wesmc7777
ms.author: wesmc
ms.date: 09/18/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: eliotga
ms.openlocfilehash: b9fc37c6589cdd0bc6a5cdce7b7ebebe2c6e9a85
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019444"
---
# <a name="iot-hub-device-provisioning-service-dps-terminology"></a>Terminologie du service Device Provisioning (DPS) IoT Hub

Le service IoT Hub Device Provisioning est un service d’assistance pour IoT Hub qui vous permet de configurer l’approvisionnement sans contact des appareils dans un hub IoT spécifié. Le service Device Provisioning vous permet d’[approvisionner](about-iot-dps.md#provisioning-process) des millions d’appareils de manière sécurisée et évolutive.

Le provisionnement des appareils est un processus en deux parties. La première partie établit la connexion initiale entre l’appareil et la solution IoT en *inscrivant* l’appareil. La deuxième partie applique la *configuration* appropriée à l’appareil selon les exigences spécifiques de la solution. Une fois que ces deux étapes sont terminées, l’appareil est complètement *approvisionné*. Le service Device Provisioning automatise les deux étapes pour fournir une expérience d’approvisionnement fluide pour l’appareil.

Cet article donne une vue d’ensemble des concepts d’approvisionnement applicables en particulier à la gestion du *service*. Cet article concerne particulièrement les personnes impliquées dans l’[étape de configuration du cloud](about-iot-dps.md#cloud-setup-step) qui permet de préparer un appareil pour le déploiement.

## <a name="service-operations-endpoint"></a>Point de terminaison des opérations de service

Le point de terminaison des opérations de service est le point de terminaison de gestion des paramètres de service et de maintenance de la liste d’inscription. Ce point de terminaison est utilisé uniquement par l’administrateur de service. Il n’est pas utilisé par les appareils.

## <a name="device-provisioning-endpoint"></a>Point de terminaison d'approvisionnement des appareils

Le point de terminaison de provisionnement des appareils est le seul point de terminaison sur lequel communiquent tous les appareils pour l’approvisionnement automatique. L’URL est la même pour toutes les instances de service d’approvisionnement de façon à éviter d’avoir à réinitialiser les appareils avec les nouvelles informations de connexion dans les scénarios de chaîne d’approvisionnement. L’étendue de l’ID garantit l’isolation des locataires.

## <a name="linked-iot-hubs"></a>Hubs IoT liés

Le service Device Provisioning peut uniquement provisionner des appareils sur des hubs IoT auxquels il est lié. La liaison d’un hub IoT à une instance du service Device Provisioning donne au Registre d’appareils du hub IoT des autorisations en lecture/écriture sur le service. Avec cette liaison, le service Device Provisioning peut inscrire un ID d’appareil et définir la configuration initiale dans l’appareil jumeau. Les hubs IoT liés peuvent se trouver dans n’importe quelle région Azure. Vous pouvez lier des hubs d’autres abonnements à votre service d’approvisionnement.


## <a name="allocation-policy"></a>Stratégie d’allocation

Paramètre au niveau du service qui détermine la façon dont le service Device Provisioning attribue les appareils à un hub IoT. Trois stratégies d’allocation sont prises en charge :

* **Distribution uniformément pondérée** : Les hubs IoT liés sont susceptibles d’avoir des appareils approvisionnés en proportions égales. Paramètre par défaut. Si vous approvisionnez des appareils sur un seul hub IoT, vous pouvez conserver ce paramètre.

* **Latence la plus faible** : Les appareils sont approvisionnés sur le hub IoT ayant la latence la plus faible sur l’appareil. Si plusieurs hubs IoT liés fournissent la même latence la plus faible, le service d’approvisionnement répartit les appareils sur ces hubs

* **Configuration statique par le biais de la liste d’inscriptions** : La spécification du hub IoT souhaité dans la liste d’inscriptions est prioritaire sur la stratégie d’allocation au niveau du service.

* **Personnalisé (utiliser Fonction Azure)**  : Une stratégie d’allocation personnalisée vous permet de mieux contrôler le processus d’assignation des appareils à un hub IoT. Cette assignation s’effectue par l’ajout de code personnalisé dans une solution Azure Functions. Le service Device Provisioning appelle votre code de Fonction Azure en fournissant toutes les informations appropriées sur l’appareil et l’inscription à votre code. Votre code de fonction est exécuté et retourne les informations sur les hubs IoT utilisés pour le provisionnement de l’appareil.

## <a name="enrollment"></a>Inscription

L’inscription désigne l’enregistrement d’appareils ou de groupes d’appareils susceptibles de s’inscrire via l’approvisionnement automatique. L’enregistrement d’inscription contient des informations sur l’appareil ou le groupe d’appareils, à savoir :
- le [mécanisme d’attestation](#attestation-mechanism) utilisée par l’appareil,
- la configuration souhaitée initiale facultative,
- le Hub IoT souhaité,
- l’ID d’appareil souhaité.

Il existe deux types d’inscription pris en charge par le service Device Provisioning Service :

### <a name="enrollment-group"></a>Groupe d’inscription

Un groupe d’inscription désigne un groupe d’appareils qui partagent un mécanisme d’attestation spécifique. Les groupes d’inscriptions prennent en charge X.509 et la clé symétrique. Tous les appareils du groupe d’inscriptions X.509 présentent des certificats X.509 qui ont été signés par la même autorité de certification racine ou intermédiaire. Chaque appareil du groupe d’inscriptions de clé symétrique présente des jetons SAS dérivés de la clé symétrique du groupe. Le nom du groupe d’inscription et le nom du certificat sont alphanumériques, en minuscules et peuvent contenir des traits d’union.

> [!TIP]
> Nous vous recommandons d’utiliser un groupe d’inscription pour un grand nombre d’appareils qui partagent une configuration initiale souhaitée ou pour des appareils destinés au même locataire.

### <a name="individual-enrollment"></a>Inscription individuelle

Une inscription individuelle désigne une entrée pour un seul appareil pouvant s’inscrire. Les inscriptions individuelles peuvent utiliser des certificats feuille X.509 ou des jetons SAP (à partir d’un module de plateforme sécurisée (TPM) réel ou virtuel) comme mécanismes d’attestation. L’ID d’enregistrement d’une inscription individuelle est alphanumérique, en minuscules et peut contenir des traits d’union. Dans le cas d’inscriptions individuelles, vous pouvez spécifier l’ID de l’appareil IoT Hub souhaité.

> [!TIP]
> Nous vous recommandons d’utiliser des inscriptions individuelles pour les appareils qui nécessitent des configurations initiales uniques ou pour ceux qui peuvent uniquement s’authentifier avec des jetons SAP via l’attestation TPM.


## <a name="attestation-mechanism"></a>Mécanisme d’attestation

Le mécanisme d’attestation est la méthode utilisée pour confirmer l’identité d’un appareil. Le mécanisme d’attestation est configuré sur une entrée d’inscription. Il indique au service d’approvisionnement la méthode à utiliser lors de la vérification de l’identité d’un appareil lors de l’inscription.

> [!NOTE]
> IoT Hub utilise un « schéma d’authentification » pour un concept semblable dans ce service.

Le service Device Provisioning prend en charge les formes d’attestation suivantes :
* **Certificats X.509** basés sur le flux d’authentification de certificat X.509 standard. Pour plus d’informations, consultez [Attestation X.509](concepts-x509-attestation.md).
* **Module de plateforme sécurisée (TPM)** basé sur un défi nonce, utilisant la norme de module de plateforme sécurisée (TPM) pour les clés afin de présenter un jeton de signature d’accès partagé (SAS) signé. Il n’est pas nécessaire d’avoir un TPM physique sur l’appareil, mais le service utilise pour l’attestation la paire de clés de type EK conformément à la [spécification TPM](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/). Pour plus d’informations, consultez [Attestation TPM](concepts-tpm-attestation.md).
* **Clé symétrique** basée sur des [jetons de sécurité](../iot-hub/iot-hub-devguide-security.md#security-tokens) avec signature d’accès partagé (SAP), qui incluent une signature hachée et un délai d’expiration incorporé. Pour plus d’informations, consultez [Attestation de clé symétrique](concepts-symmetric-key-attestation.md).


## <a name="hardware-security-module"></a>Module de sécurité matériel

Le module de sécurité matériel, ou module HSM, est utilisé pour le stockage matériel sécurisé des secrets de l’appareil. C’est la forme de stockage de secrets la plus sécurisée. Les certificats X.509 et les jetons SAP peuvent être stockés dans le module HSM. Les modules HSM peuvent être utilisés avec les deux mécanismes d’attestation que le service d’approvisionnement prend en charge.

> [!TIP]
> Nous vous recommandons vivement d’utiliser un module HSM avec les appareils pour stocker en toute sécurité des secrets sur vos appareils.

Les secrets de l’appareil peuvent également être stockés sous forme logicielle (mémoire), mais cette forme de stockage est moins sécurisée que le module HSM.



## <a name="id-scope"></a>Étendue de l’ID

L’étendue de l’ID est affectée à un service Device Provisioning quand celui-ci est créé par l’utilisateur et utilisé pour identifier de façon unique le service d’approvisionnement spécifique qui permet d’enregistrer l’appareil. L’étendue de l’ID est générée par le service et est immuable, ce qui garantit l’unicité.

> [!NOTE]
> L’unicité est importante pour les opérations de déploiement longues et les scénarios de fusion et acquisition.


## <a name="registration"></a>Inscription

L’inscription désigne l’enregistrement d’un appareil inscrit ou provisionné sur un hub IoT via le service Device Provisioning. Les enregistrements d’inscription sont créés automatiquement. Ils peuvent être supprimés, mais pas mis à jour.


## <a name="registration-id"></a>ID d’enregistrement

L’ID d’inscription est utilisé pour identifier de manière unique l’inscription d’un appareil dans le service Device Provisioning. L’ID d’appareil doit être unique dans l’[étendue de l’ID](#id-scope) du service d’approvisionnement. Chaque appareil doit avoir un ID d’enregistrement. L’ID d’inscription est insensible à la casse ; il peut comporter des caractères alphanumériques et des caractères spéciaux (deux points, point, trait de soulignement et trait d’union).

* Dans le cas d’un module TPM, l’ID d’enregistrement est fourni par le TPM lui-même.
* Dans le cas d’une attestation X.509, l’ID d’enregistrement est le nom de sujet du certificat.

## <a name="device-id"></a>ID d’appareil

L’ID d’appareil est l’ID tel qu’il apparaît dans IoT Hub. L’ID d’appareil de votre choix peut être défini dans l’entrée d’inscription, mais ce n’est pas obligatoire. Il n’est possible de définir l’ID d’appareil souhaité que dans les inscriptions individuelles. Si aucun ID d’appareil souhaité n’est spécifié dans la liste d’inscriptions, l’ID d’inscription est utilisé comme ID d’appareil durant l’enregistrement de l’appareil. Découvrez plus d’informations sur les [ID d’appareil dans IoT Hub](../iot-hub/iot-hub-devguide-identity-registry.md).



## <a name="operations"></a>Operations

Les opérations désignent l’unité de facturation du service Device Provisioning. Une opération est une instruction envoyée au service qui se déroule correctement. Les opérations incluent les inscriptions et réinscriptions d’appareils, ainsi que les changements côté service, comme l’ajout et la mise à jour des entrées de liste d’inscriptions.
