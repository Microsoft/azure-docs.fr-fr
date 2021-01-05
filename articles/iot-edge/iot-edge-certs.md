---
title: Certificats pour la sécurité des appareils - Azure IoT Edge | Microsoft Docs
description: Azure IoT Edge utilise un certificat pour valider les appareils, les modules et les appareils de nœud terminal, et pour établir des connexions sécurisées entre eux.
author: stevebus
manager: philmea
ms.author: stevebus
ms.date: 08/12/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: mqtt
ms.openlocfilehash: d1d4abbcc0768915d7d2e693cfc76a699ed21a91
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89669617"
---
# <a name="understand-how-azure-iot-edge-uses-certificates"></a>Comprendre Azure IoT Edge utilise les certificats

Les certificats IoT Edge sont utilisés par les modules et les appareils IoT en aval afin de vérifier l’identité et la légitimité du module d’exécution du [hub IoT Edge](iot-edge-runtime.md#iot-edge-hub). Ces vérifications permettent une connexion sécurisée TLS entre le runtime, les modules et les appareils IoT. Tout comme IoT Hub, IoT Edge nécessite une connexion sécurisée et chiffrée à partir d’appareils IoT en aval (ou feuilles) et de modules IoT Edge. Pour établir une connexion TLS sécurisée, le module du hub IoT Edge présente une chaîne d’approbation de serveur aux clients qui se connectent pour que ces derniers puissent vérifier son identité.

>[!NOTE]
>Cet article présente les certificats utilisés pour sécuriser les connexions entre les différents composants d’un appareil IoT Edge ou entre un appareil IoT Edge et des appareils de nœuds terminaux. Vous pouvez également utiliser des certificats pour authentifier votre appareil IoT Edge pour IoT Hub. Ces certificats d’authentification sont différents et ne sont pas abordés dans cet article. Pour plus d’informations sur l’authentification de votre appareil avec des certificats, consultez [Créer et provisionner un appareil IoT Edge à l’aide de certificats X.509](how-to-auto-provision-x509-certs.md).

Cet article explique comment les certificats IoT Edge fonctionnent dans un environnement de production, de développement et de test. Bien que les scripts diffèrent (PowerShell/bash), les concepts sont identiques pour Linux et Windows.

## <a name="iot-edge-certificates"></a>Certificats IoT Edge

Il existe deux scénarios courants pour configurer des certificats sur un appareil IoT Edge. Parfois, l’utilisateur final, ou l’opérateur, d’un appareil achète un appareil générique créé par un fabricant, puis gère lui-même les certificats. Dans d’autres cas, le fabricant travaille sous contrat et crée un appareil personnalisé destiné à l’opérateur, puis effectue une signature de certificat initiale avant de lui remettre l’appareil. La conception de certificat IoT Edge tente de tenir compte de ces deux scénarios.

Le schéma suivant illustre l’utilisation des certificats par IoT Edge. Il peut y avoir un ou plusieurs certificats de signature intermédiaires (ou aucun) entre le certificat d’autorité de certification racine et le certificat d’autorité de certification de l’appareil, selon le nombre d’entités impliquées. Ici, nous ne montrons qu’un seul cas.

![Diagramme des relations de certificat standard](./media/iot-edge-certs/edgeCerts-general.png)

> [!NOTE]
> Il existe actuellement dans libiothsm une limitation empêchant l’utilisation de certificats qui expirent le 1er janvier 2038 ou après cette date. Cette limitation s'applique au certificat d'autorité de certification de l'appareil, à tous les certificats du bundle de confiance et aux certificats d'identité d'appareil utilisés pour les méthodes d'approvisionnement X.509.

### <a name="certificate-authority"></a>Autorité de certification

L’autorité de certification (ou « CA ») est une entité qui émet des certificats numériques. Une autorité de certification agit comme un tiers de confiance entre le propriétaire et le destinataire du certificat. Un certificat numérique certifie que le destinataire du certificat détient une clé publique. La chaîne d’approbation de confiance fonctionne en émettant initialement un certificat racine, qui sert de base pour tous les certificats émis par l’autorité. Par la suite, le propriétaire peut utiliser le certificat racine pour émettre des certificats intermédiaires supplémentaires (certificats « feuilles »).

### <a name="root-ca-certificate"></a>Certificat d’autorité de certification racine

Un certificat d’autorité de certification racine est la source de confiance de l’ensemble du processus. Dans les scénarios de production, le certificat d’autorité de certification est généralement acheté auprès d’une autorité de certification commerciale de confiance comme Baltimore, Verisign, DigiCert, etc. Si vous avez un contrôle complet sur les dispositifs qui se connectent à vos appareils IoT Edge, il est possible d’utiliser une autorité de certification au niveau de l’entreprise. Dans les deux cas, la chaîne d’approbation dans son intégralité remonte vers celle-ci à partir du hub IoT Edge. Les appareils de nœud terminal IoT doivent donc approuver le certificat racine. Vous pouvez stocker le certificat d’autorité de certification racine dans le magasin d’autorité de certification racine de confiance, ou fournir les détails du certificat dans le code de votre application.

### <a name="intermediate-certificates"></a>Certificats intermédiaires

Dans un processus de fabrication standard d’appareils sécurisés, les certificats d’autorité de certification racine sont rarement utilisés directement, essentiellement en raison du risque de fuite ou d’exposition. Le certificat d’autorité de certification racine crée et signe numériquement un ou plusieurs certificats d’autorité de certification intermédiaires. Il peut n’y en avoir qu’un, ou il peut y avoir une chaîne de certificats intermédiaires. Les scénarios qui nécessitent une chaîne de certificats intermédiaires sont les suivants :

* Hiérarchie de départements chez un fabricant.

* Plusieurs entreprises impliquées successivement dans la production d’un appareil.

* Client achetant une autorité de certification racine et obtenant un certificat de signature permettant au fabricant de signer les appareils qu’il fabrique pour le compte de ce client.

Dans tous les cas, le fabricant utilise un certificat d’autorité de certification intermédiaire à la fin de cette chaîne pour signer le certificat d’autorité de certification placé sur l’appareil final. En règle générale, ces certificats intermédiaires sont jalousement gardés à l’usine de fabrication. Ils sont soumis à des processus rigoureux, physiques et électroniques, pour leur utilisation.

### <a name="device-ca-certificate"></a>Certificat d’autorité de certification d’appareil

Le certificat d’autorité de certification de l’appareil est généré à partir du certificat d’autorité de certification intermédiaire final (et signé par celui-ci) dans le processus. Ce certificat est installé sur l’appareil IoT Edge, de préférence dans un stockage sécurisé, tel qu’un module de sécurité matériel (HSM). En outre, un certificat d’autorité de certification d’appareil identifie de façon unique un appareil IoT Edge. Le certificat d’autorité de certification d’appareil peut signer d’autres certificats.

### <a name="iot-edge-workload-ca"></a>Autorité de certification pour la charge de travail IoT Edge

Le [gestionnaire de sécurité IoT Edge](iot-edge-security-manager.md) génère le certificat d’autorité de certification de charge de travail (le premier du côté « opérateur » du processus) lors du premier démarrage d’IoT Edge. Ce certificat est généré à partir du certificat d’autorité de certification d’appareil (et signé par celui-ci). Ce certificat, qui est simplement un autre certificat de signature intermédiaire, sert à générer et à signer tous les autres certificats utilisés par le runtime IoT Edge. Aujourd’hui, il s’agit principalement du certificat de serveur du hub IoT Edge abordé dans la section suivante, mais à l’avenir, cela pourra inclure les autres certificats d’authentification des composants IoT Edge.

### <a name="iot-edge-hub-server-certificate"></a>Certificat de serveur du hub IoT Edge

Le certificat de serveur du hub IoT Edge est le certificat qui est présenté aux modules et aux appareils de nœud terminal pour la vérification d’identité lors de l’établissement de la connexion TLS exigée par IoT Edge. Ce certificat présente la chaîne complète des certificats de signature utilisés pour le générer jusqu’au certificat d’autorité de certification racine, que l’appareil IoT feuille doit approuver. Lorsqu’il est généré par le gestionnaire de sécurité IoT Edge, le nom commun du certificat du hub IoT Edge est défini sur la propriété « hostname » dans le fichier config.yaml, après sa conversion en minuscules. Cette configuration est une source courante de confusion avec IoT Edge.

## <a name="production-implications"></a>Implications en matière de production

On peut raisonnablement se demander pourquoi IoT Edge a besoin d’un certificat supplémentaire pour l’autorité de certification de la charge de travail. Pourquoi ne pas utiliser le certificat d’autorité de certification d’appareil pour générer directement le certificat de serveur du hub IoT Edge ? Techniquement, c’est possible. Toutefois, l’objectif de ce certificat intermédiaire de « charge de travail » consiste à séparer les responsabilités entre le fabricant de l’appareil et l’opérateur. Imaginez un scénario où un appareil IoT Edge est vendu ou transféré d’un client à un autre. Il est sans doute préférable que le certificat d’autorité de certification de l’appareil fourni par le fabricant soit immuable. Cependant, les certificats de « charge de travail » propres au fonctionnement de l’appareil doivent être supprimés et recréés pour le nouveau déploiement.

Étant donné que les processus de fabrication et d’opérations sont séparés, tenez compte des conséquences suivantes lorsque vous préparez les appareils de production :

* Avec n’importe quel processus reposant sur un certificat, le certificat d’autorité de certification racine et tous les certificats d’autorité de certification intermédiaires doivent être sécurisés et surveillés pendant tout le processus de déploiement d’un appareil IoT Edge. Le fabricant de l’appareil IoT Edge doit mettre en place des processus stricts pour un stockage et une utilisation appropriés de ses certificats intermédiaires. En outre, le certificat d’autorité de certification de l’appareil doit être conservé dans un emplacement de stockage aussi sécurisé que possible sur l’appareil lui-même, de préférence un module de sécurité matériel.

* Le certificat de serveur du hub IoT Edge est présenté par le hub IoT Edge sur les appareils et les modules des clients qui se connectent. Le nom commun du certificat d’autorité de certification d’appareil **ne doit pas** être le même que le « hostname » qui sera utilisé dans le fichier config.yaml sur l’appareil IoT Edge. Le nom utilisé par les clients pour se connecter à IoT Edge (par exemple, via le paramètre GatewayHostName de la chaîne de connexion ou la commande CONNECT de MQTT) **ne peut pas être** identique au nom commun utilisé dans le certificat d’autorité de certification d’appareil. Cela est dû au fait que le hub IoT Edge présente sa chaîne d’approbation dans son intégralité afin que les clients la vérifient. Si le certificat de serveur du hub IoT Edge et le certificat d’autorité de certification d’appareil ont tous deux le même nom commun, vous obtenez une boucle de vérification, ce qui rend non valide votre certificat.

* Étant donné que le certificat d’autorité de certification d’appareil est utilisé par le démon de sécurité IoT Edge pour générer les certificats IoT Edge finaux, il doit lui-même être un certificat de signature, ce qui signifie qu’il a des fonctionnalités de signature de certificat. L’application du paramètre « V3 Basic constraints CA:True » au certificat d’autorité de certification d’appareil configure automatiquement les propriétés d’utilisation de clé nécessaires.

>[!Tip]
> Si vous avez déjà configuré IoT Edge en tant que passerelle transparente dans un scénario de développement/test à l’aide de nos « scripts de commodité » (voir la section suivante) et utilisé le même nom d’hôte lors de la création du certificat d’autorité de certification d’appareil que pour le nom d’hôte dans le fichier config.yaml, vous vous demandez peut-être pourquoi cela a fonctionné. Dans le but de simplifier l’expérience de développement, les scripts de commodité ajoutent « .ca » à la fin du nom que vous transmettez dans le script. Par conséquent, si vous avez utilisé « mygateway » comme nom de périphérique dans les scripts et comme nom d’hôte dans le fichier config.yaml, la première occurrence devient en mygateway.ca avant d’être utilisée en tant que nom commun (CN) pour le certificat d’autorité de certification d’appareil.

## <a name="devtest-implications"></a>Implications en matière de développement/test

Pour faciliter les scénarios de développement et de test, Microsoft fournit un ensemble de [scripts de commodité](https://github.com/Azure/azure-iot-sdk-c/tree/master/tools/CACertificates) permettant de générer des certificats hors production appropriés pour IoT Edge dans le scénario de passerelle transparente. Pour obtenir des exemples de fonctionnement des scripts, consultez [Créer des certificats de démonstration pour tester les fonctionnalités des appareils IoT Edge](how-to-create-test-certificates.md).

>[!Tip]
> Pour connecter vos appareils IoT feuille et applications utilisant notre Kit de développement logiciel (SDK) d’appareil IoT via IoT Edge, vous devez ajouter le paramètre GatewayHostName facultatif à la fin de la chaîne de connexion de l’appareil. Lorsque le certificat de serveur Edge Hub est généré, il est basé sur une version en minuscules du nom d’hôte indiqué dans le fichier config.yaml. Par conséquent, pour que les noms correspondent et que la vérification du certificat TLS réussisse, vous devez saisir le paramètre GatewayHostName en minuscules.

## <a name="example-of-iot-edge-certificate-hierarchy"></a>Exemple de hiérarchie de certificats IoT Edge

Pour illustrer ce chemin d’accès de certificat, la capture d’écran suivante montre un appareil IoT Edge configuré en tant que passerelle transparente. OpenSSL est utilisé pour se connecter au hub IoT Edge, puis pour valider et vider les certificats.

![Capture d’écran de la hiérarchie de certificats à chaque niveau](./media/iot-edge-certs/iotedge-cert-chain.png)

Observez la hiérarchie des certificats représentée dans la capture d’écran :

| Certificat d’autorité de certification racine         | Certification d’autorité de certification Azure IoT Hub - Test uniquement                                                                           |
|-----------------------------|-----------------------------------------------------------------------------------------------------------|
| Certificat d’autorité de certification intermédiaire | Certification intermédiaire Azure IoT Hub - Test uniquement                                                                 |
| Certificat d’autorité de certification d’appareil       | iotgateway.ca (« iotgateway » a été transmis comme  < nom d’hôte de passerelle > aux scripts de commodité)   |
| Certificat d’autorité de certification de la charge de travail     | Autorité de certification pour la charge de travail IoT Edge                                                                                       |
| Certificat de serveur du hub IoT Edge | iotedgegw.local (correspond au nom d’hôte du fichier config.yaml)                                            |

## <a name="next-steps"></a>Étapes suivantes

[Présentation des modules Azure IoT Edge](iot-edge-modules.md)

[Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md)
