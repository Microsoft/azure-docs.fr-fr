---
title: Service Azure IoT Hub Device Provisioning - Attestation TPM
description: Cet article fournit une vue d’ensemble conceptuelle du flux de l’attestation TPM utilisant le service IoT Device Provisioning (DPS).
author: nberdy
ms.author: nberdy
ms.date: 04/04/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: briz
ms.openlocfilehash: 12860629d78391ed271306daba29a51aeb326c1d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90531591"
---
# <a name="tpm-attestation"></a>Attestation TPM

Le service IoT Hub Device Provisioning est un service d’assistance pour IoT Hub qui vous permet de configurer l’approvisionnement sans contact des appareils dans un hub IoT spécifié. Avec le service Device Provisioning, vous pouvez approvisionner des millions d’appareils de manière sécurisée.

Cet article décrit le processus d’attestation d’identité lorsque vous utilisez un module de plateforme de confiance (TPM). Un TPM est un type de module de sécurité matériel (HSM). Cet article suppose que vous utilisez un microprogramme discret, ou un TPM intégré. Les TPM émulés logiciels sont parfaitement adaptés pour le prototypage ou le test, mais ils n’offrent pas le même niveau de sécurité que les TPM intégrés ou les microprogrammes discrets. Nous vous déconseillons d’utiliser les TPM logiciels en production. Pour plus d’informations sur les types de TPM, consultez la section [Brève introduction au TPM](https://trustedcomputinggroup.org/wp-content/uploads/TPM-2.0-A-Brief-Introduction.pdf).

Cet article concerne uniquement les appareils utilisant un TPM 2.0 avec la prise en charge de clé HMAC et leurs paires de clés de type EK. Il n’est pas destiné aux appareils utilisant des certificats X.509 pour l’authentification. TPM est une norme ISO du secteur issue de Trusted Computing Group, et vous pouvez en apprendre davantage sur TPM en lisant la [spécification TPM 2.0 complète](https://trustedcomputinggroup.org/tpm-library-specification/) ou la [spécification ISO/IEC 11889](https://www.iso.org/standard/66510.html). Cet article suppose également que vous connaissez les paires de clés privées et publiques, ainsi que la façon dont elles sont utilisées pour le chiffrement.

Les Kits de développement logiciel (SDK) d’appareil du service Device Provisioning gèrent pour vous tout ce qui est décrit dans cet article. Vous n’avez pas besoin d’implémenter quoi que ce soit si vous utilisez les Kits de développement logiciel (SDK) sur vos appareils. Cet article vous aidera à comprendre, d’un point de vue conceptuel, ce qui se passe avec votre processeur de sécurité TPM lorsque votre appareil est approvisionné et pourquoi cette opération est si sécurisée.

## <a name="overview"></a>Vue d’ensemble

Les TPM utilisent une paire de clés de type EK en tant que clé racine sécurisée de confiance. La paire de clés de type EK est unique au TPM et sa modification transforme l’appareil en un autre appareil.

Les TPM ont également un autre type de clé, nommé la clé racine de stockage. Une clé racine de stockage peut être générée par le propriétaire du TPM après qu’il a pris possession du TPM. Prendre possession du TPM revient à dire « quelqu’un définit un mot de passe sur le HSM » en langage TPM. Si un appareil TPM est vendu à un autre propriétaire, celui-ci peut prendre possession du TPM pour générer une nouvelle clé racine de stockage. L’opération de génération d’une nouvelle clé racine de stockage permet de s’assurer que le propriétaire précédent ne peut pas utiliser le TPM. Étant donné que la clé racine de stockage est unique au propriétaire du TPM, elle peut être utilisée pour sceller des données dans le TPM lui-même pour ce propriétaire. La clé de stockage racine fournit un bac à sable au propriétaire pour qu’il stocke ses clés et propose une fonctionnalité de suppression de l’accès si l’appareil ou le TPM est vendu. C’est comme si vous emménagiez dans une nouvelle maison : en devenant propriétaire, vous changez les serrures et détruisez tous les meubles laissés par les anciens propriétaires (clé racine de stockage), mais vous ne pouvez pas changer l’adresse de la maison (paire de clés de type EK).

Une fois qu’un appareil a été configuré et qu’il est prêt à être utilisé, la paire de clés EK et la clé racine de stockage sont disponibles.

![Prise de possession d’un TPM](./media/concepts-tpm-attestation/tpm-ownership.png)

Une remarque sur la prise de possession du module TPM : Prendre possession d’un TPM dépend de nombreux facteurs, notamment du fabricant du TPM, de l’ensemble d’outils TPM utilisés et du système d’exploitation de l’appareil. Suivez les instructions relatives à votre système pour la prise de possession.

Le service Device Provisioning utilise la partie publique de la paire de clés de type EK (EK_pub) pour identifier et inscrire des appareils. Le fournisseur de l’appareil peut lire la partie EK_pub pendant la fabrication ou le test final et charger la partie EK_pub dans le service d’approvisionnement de façon que l’appareil soit reconnu lorsqu’il se connecte pour l’approvisionnement. Le service Device Provisioning ne vérifie pas la clé racine de stockage ni le propriétaire. Par conséquent, le TPM efface les données clientes mais la paire de clés de type EK (et les autres données du fournisseur) est conservée et l’appareil sera toujours reconnu par le service Device Provisioning lorsqu’il se connecte pour l’approvisionnement.

## <a name="detailed-attestation-process"></a>Processus d’attestation détaillé

Quand un appareil avec un TPM se connecte pour la première fois au service Device Provisioning, le service commence par vérifier la partie EK_pub par rapport à la partie EK_pub stockée dans la liste d’inscription. Si les parties EK_pub ne correspondent pas, l’approvisionnement de l’appareil est refusé. Si les parties EK_pub correspondent, le service demande à l’appareil de prouver qu’il possède la partie privée de la paire de clés EK via une stimulation nonce, qui permet de prouver l’identité de façon sécurisée. Le service Device Provisioning génère une valeur nonce, qui est ensuite chiffrée avec la clé racine de stockage et la partie EK_pub, qui sont fournies par l’appareil lors de l’appel d’inscription initial. Le TPM conserve toujours la partie privée de la paire de clés EK sécurisée. Cela permet d’éviter les contrefaçons et de s’assurer que les jetons SAP sont approvisionnés de façon sûre pour les appareils autorisés.

Nous allons étudier en détail le processus d’attestation.

### <a name="device-requests-an-iot-hub-assignment"></a>L’appareil demande une affectation IoT Hub

Tout d’abord, l’appareil se connecte au service Device Provisioning et demande à être approvisionné. Ce faisant, l’appareil fournit au service son ID d’inscription, une étendue d’ID et les parties EK_pub et SRK_pub du TPM. Le service transmet la valeur once chiffrée à l’appareil et demande à l’appareil de déchiffrer cette valeur once et de l’utiliser pour signer un jeton SAP pour se connecter une fois de plus et terminer l’approvisionnement.

![L’appareil demande l’approvisionnement](./media/concepts-tpm-attestation/step-one-request-provisioning.png)

### <a name="nonce-challenge"></a>Stimulation nonce

L’appareil récupère la valeur nonce et utilise les parties privées de la paire de clés de type EK et de la clé racine de stockage pour déchiffrer la valeur nonce dans le TPM ; l’ordre du chiffrement nonce délègue la confiance de la paire de clés de type EK, qui est immuable, à la clé racine de stockage, qui peut être modifiée si une nouvelle personne prend possession du TPM.

![Déchiffrement de la valeur nonce](./media/concepts-tpm-attestation/step-two-nonce.png)

### <a name="validate-the-nonce-and-receive-credentials"></a>Valider la valeur nonce et recevoir des informations d’identification

L’appareil peut ensuite signer un jeton SAP à l’aide de la valeur nonce déchiffrée et rétablir une connexion au service Device Provisioning en utilisant le jeton SAP signé. Une fois la stimulation nonce terminée, le service autorise l’approvisionnement de l’appareil.

![L’appareil rétablit la connexion au service de provisionnement des appareils pour valider la possession de la paire de clés de type EK.](./media/concepts-tpm-attestation/step-three-validation.png)

## <a name="next-steps"></a>Étapes suivantes

L’appareil se connecte maintenant à IoT Hub et vous êtes sûr que les clés de vos appareils sont stockées en toute sécurité. Maintenant que vous savez comment le service Device Provisioning vérifie de façon sécurisée l’identité d’un appareil à l’aide d’un TPM, consultez les articles suivants pour en savoir plus :

* [Découvrir plus d’informations sur les concepts du provisionnement](about-iot-dps.md#provisioning-process)
* [Configurer le service d’approvisionnement d’appareil IoT Hub avec le portail Azure](./quick-setup-auto-provision.md) à l’aide de Kits de développement logiciel (SDK) pour prendre en charge le flux.
