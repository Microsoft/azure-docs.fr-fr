---
title: Applications approuvées sous forme de modules Azure IoT Edge
description: Utilisez le kit de développement logiciel (SDK) et l'API Open Enclave afin de créer des applications approuvées et de les déployer sous forme de modules IoT Edge pour l'informatique confidentielle
author: kgremban
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: kgremban
ms.openlocfilehash: d81cc6c94c04c683362fd12cd6777c304a4b0a84
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91361974"
---
# <a name="confidential-computing-at-the-edge"></a>Informatique confidentielle à la périphérie

Azure IoT Edge prend en charge les applications approuvées exécutées dans des enclaves sécurisées de l'appareil. Le chiffrement assure la sécurité des données en transit ou au repos, tandis que les enclaves assurent la sécurité des données et des charges de travail en cours d'utilisation. IoT Edge prend en charge Open Enclave en guise de norme pour le développement d'applications approuvées.

La sécurité a toujours été un aspect important de l'Internet des objets (IoT) car les appareils IoT sont rarement situés à l'intérieur d'une installation privée. Exposés, les appareils peuvent faire l'objet de tentatives d'altération et de falsification car ils sont physiquement accessibles à des personnes malveillantes. En termes d'approbation et d'intégrité, les besoins des appareils IoT Edge sont d'autant plus importants qu'ils permettent d'exécuter des charges de travail sensibles à la périphérie. Contrairement aux capteurs et aux mécanismes de positionnement courants, ces appareils de périphérie intelligente exposent potentiellement des charges de travail sensibles qui étaient auparavant exécutées dans des environnements cloud ou locaux protégés.

Le [Gestionnaire de sécurité IoT Edge](iot-edge-security-manager.md) répond à une partie du défi posé par l'informatique confidentielle. Le gestionnaire de sécurité utilise un module de sécurité matériel (HSM) pour protéger les charges de travail liées à l'identité et les processus en cours d'un appareil IoT Edge.

La protection des données utilisées à la périphérie est un autre aspect de l'informatique confidentielle. Un *environnement d'exécution approuvé* (TEE) est un environnement sécurisé et isolé sur un processeur ; c'est ce qu'on appelle parfois une *enclave*. Une *application approuvée* est une application exécutée dans une enclave. Compte tenu de la nature des enclaves, les applications approuvées sont protégées des autres applications exécutées sur le processeur principal ou dans l'environnement TEE.

## <a name="trusted-applications-on-iot-edge"></a>Applications approuvées sur IoT Edge

Les applications approuvées sont chiffrées lorsqu'elles sont en transit et au repos ; elles ne sont déchiffrées que pour s'exécuter dans un environnement d'exécution approuvé. Cette norme s'applique aux applications approuvées déployées sous forme de modules IoT Edge.

Le développeur crée l'application approuvée et l'empaquette sous forme de module IoT Edge. L'application est chiffrée avant d'être envoyée (push) au registre de conteneurs. L'application reste chiffrée tout au long du processus de déploiement IoT Edge jusqu'à ce que le module soit démarré sur l'appareil IoT Edge. Une fois dans l'environnement TEE de l'appareil, l'application approuvée est déchiffrée et son exécution peut débuter.

![Diagramme - Les applications approuvées sont chiffrées dans des modules IoT Edge jusqu'à leur déploiement dans l'enclave sécurisée](./media/deploy-trusted-applications/trusted-applications-encrypted.png)

Les applications approuvées sur IoT Edge sont une extension logique de l'[informatique confidentielle Azure](../confidential-computing/overview.md). Les charges de travail exécutées dans des enclaves sécurisées du cloud peuvent également être déployées pour être exécutées dans des enclaves sécurisées à la périphérie.

## <a name="open-enclave"></a>Open Enclave

Le kit de développement logiciel (SDK) [Open enclave](https://openenclave.io/sdk/) est un projet open source qui permet aux développeurs de créer des applications approuvées pour différents environnements et plateformes. Le kit de développement logiciel (SDK) Open Enclave fonctionne dans l'environnement d'exécution approuvé d'un appareil, tandis que l'API Open Enclave sert d'interface entre le l'environnement TEE et l'environnement de traitement non TEE.

Open Enclave prend en charge différentes plateformes matérielles. La prise en charge des enclaves par IoT Edge requiert actuellement le système d'exploitation Open Portable TEE (OP-TEE OS). Pour en savoir plus, consultez [SDK Open Enclave pour OP-TEE OS](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/OP-TEE/Introduction.md).

Le référentiel Open Enclave contient également des exemples pour aider les développeurs à démarrer. Pour plus d'informations, choisissez l'un des articles de présentation suivants :

* [SDK Open Enclave - Création d'exemples sous Linux](https://github.com/openenclave/openenclave/blob/master/samples/BuildSamplesLinux.md)
* [SDK Open Enclave - Création d'exemples sous Windows](https://github.com/openenclave/openenclave/blob/master/samples/BuildSamplesWindows.md)

## <a name="hardware"></a>Matériel

Actuellement, la [TrustBox de Scalys](https://scalys.com/trustbox-industrial/) est le seul appareil pris en charge par les contrats de service du fabricant pour le déploiement d'applications approuvées sous forme de modules IoT Edge. La TrustBox est basée sur les appareils TrustBox Edge et TrustBox EdgeXL, sur lesquels le kit de développement logiciel (SDK) Open Enclave et Azure IoT Edge sont préchargés.

Pour plus d'informations, consultez [Prise en main d'Open Enclave pour la TrustBox de Scalys](https://aka.ms/scalys-trustbox-edge-get-started).

## <a name="develop-and-deploy"></a>Développer et déployer

Lorsque vous êtes prêt à développer et à déployer votre application approuvée, l'extension [Microsoft Open Enclave](https://marketplace.visualstudio.com/items?itemName=ms-iot.msiot-vscode-openenclave) pour Visual Studio Code peut vous aider. Vous pouvez utiliser un ordinateur de développement Linux ou Windows afin de développer des modules pour la TrustBox.

## <a name="next-steps"></a>Étapes suivantes

Apprenez à développer des applications approuvées sous forme de modules IoT Edge avec l'[extension Open Enclave pour Visual Studio Code](https://github.com/openenclave/openenclave/tree/master/devex/vscode-extension).
