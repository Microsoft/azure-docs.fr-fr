---
title: Gestionnaire de sécurité Azure IoT Edger - Azure IoT Edge
description: Gère le principe de sécurité d’appareil IoT Edge et l’intégrité des services de sécurité.
services: iot-edge
keywords: sécurité, élément sécurisé, enclave, TEE, IoT Edge
author: eustacea
manager: philmea
ms.author: eustacea
ms.date: 08/30/2019
ms.topic: conceptual
ms.service: iot-edge
ms.openlocfilehash: d50ff37c3d29ae8e9e25d8759ef3c55787a58047
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92042998"
---
# <a name="azure-iot-edge-security-manager"></a>Gestionnaire de sécurité Azure IoT Edge

Le Gestionnaire de sécurité Azure IoT Edge est un cœur de sécurité bien délimité de protection de l’appareil IoT Edge et tous ses composants en faisant abstraction du matériel de silicium de sécurité. Le Gestionnaire de sécurité, point focal du renforcement de la sécurité, fournit un point d’intégration technologique aux fabricants OEM.

![Gestionnaire de sécurité Azure IoT Edge](media/edge-security-manager/iot-edge-security-manager.png)

Le Gestionnaire de sécurité IoT Edge vise à protéger l’intégrité de l’appareil IoT Edge et toutes les opérations de logiciel inhérentes. Le gestionnaire de sécurité fait passer l’approbation de la racine matérielle sous-jacente du matériel de confiance (s’il est disponible) pour démarrer le runtime IoT Edge et superviser les opérations en cours.  Le gestionnaire de sécurité IoT Edge est un logiciel qui fonctionne avec des processeurs en silicium sécurisés (si disponibles) pour offrir les garanties de sécurité les plus élevées possible.  

Les responsabilités du gestionnaire de sécurité IoT Edge sont notamment :

* Démarrage sécurisé et mesuré de l’appareil Azure IoT Edge.
* Approvisionnement de l’identité de l’appareil et la transition de confiance, le cas échéant.
* Héberger et protéger les composants de l’appareil de services cloud comme Service Device Provisioning.
* Provisionnement sécurisé des modules IoT Edge avec des identités uniques.
* Opérateur de contrôle à la racine matérielle d’appareil de confiance via les services de notaire.
* Surveiller l’intégrité des opérations IoT Edge lors de l’exécution.

Le gestionnaire de sécurité IoT Edge comprend trois composants :

* Démon de sécurité IoT Edge.
* Abstraction calque de la plateforme du module de sécurité matérielle (PAL HSM).
* Racine matérielle de silicium de confiance facultative mais fortement recommandée ou HSM.

## <a name="the-iot-edge-security-daemon"></a>Démon de sécurité IoT Edge

Le démon de sécurité IoT Edge est responsable des opérations logiques du gestionnaire de sécurité IoT Edge. Il représente une partie significative du TCB (Trusted Computing Base) de l’appareil IoT Edge.

### <a name="design-principles"></a>Principes de conception

Le démon de sécurité IoT Edge suit deux principes de base : optimiser l’intégrité opérationnelle, et réduire le ballonnement et l’évolution.

#### <a name="maximize-operational-integrity"></a>Optimiser l’intégrité opérationnelle

Le démon de sécurité IoT Edge fonctionne avec l’intégrité la plus élevée possible dans la fonctionnalité de défense de n’importe quelle racine donnée du matériel de confiance. Avec l’intégration appropriée, la racine du matériel de confiance mesure et surveille le démon de sécurité statiquement et lors de l’exécution pour résister aux falsifications.

L’accès physique constitue toujours une menace pour les appareils IoT. La racine matérielle de confiance joue un rôle important dans la protection de l’intégrité du démon de sécurité IoT Edge.  La racine matérielle de confiance se présente dans deux versions :

* éléments sécurisés pour la protection des informations sensibles telles que les secrets et clés de chiffrement.
* enclaves sécurisées pour la protection des secrets tels que les clés et des charges de travail sensibles telles que les métriques et les facturations.

Il existe deux types d’environnements d’exécution pour utiliser la racine matérielle de confiance :

* L’environnement d’exécution standard ou riche (EER) qui s’appuie sur l’utilisation d’éléments sécurisés pour protéger les informations sensibles.
* L’environnement d’exécution approuvé (EEA) qui s’appuie sur l’utilisation de la technologie d’enclave sécurisée pour protéger les informations sensibles et offrir une protection à l’exécution de logiciels.

Pour les appareils qui utilisent des enclaves sécurisées comme racine matérielle de confiance, la logique sensible au sein du démon de sécurité IoT Edge doit se trouver à l’intérieur de l’enclave.  Les parties non sensibles du démon de sécurité peuvent se trouver en dehors de l’environnement d’exécution approuvé.  Dans tous les cas, les fabricants ODM et les fabricants OEM doivent élargir la confiance à partir de leur HSM pour mesurer et défendre l’intégrité du démon de sécurité IoT Edge au moment du démarrage et de l’exécution.

#### <a name="minimize-bloat-and-churn"></a>Réduire le ballonnement et l’évolution

Un autre principe de base pour le démon de sécurité IoT Edge consiste à réduire l’évolution.  Pour le niveau le plus élevé de confiance, le démon de sécurité IoT Edge peut étroitement s’associer à la racine matérielle de confiance de l’appareil et fonctionner comme du code natif.  Il est courant pour ces types de réalisations de mettre à jour le démon via le chemin de mise à jour sécurisée de la racine matérielle de confiance (par opposition à des mécanismes de mise à jour fournis par le système d’exploitation), ce qui peut s’avérer difficile dans certains scénarios.  Tandis que le renouvellement de la sécurité est recommandé pour les appareils IoT, des exigences de mise à jour excessives ou des charges utiles de mise à jour importantes peuvent développer la surface d’attaque de nombreuses façons.  Ignorer des mises à jour pour optimiser la disponibilité opérationnelle ou la racine du matériel de confiance trop limitée pour traiter des misse à jour de charges utiles importantes.  Par conséquent, la conception du démon de sécurité IoT Edge est concise, de façon à conserver une taille réduite pour l’empreinte et pour le TCB, et à réduire les exigences de mise à jour.

### <a name="architecture-of-iot-edge-security-daemon"></a>L’architecture du démon de sécurité IoT Edge

![Démon de sécurité Azure IoT Edge](media/edge-security-manager/iot-edge-security-daemon.png)

Le démon de sécurité IoT Edge est conçu pour tirer parti de n’importe quelle technologie de racine matérielle de confiance disponible pour le renforcement de la sécurité.  Il permet également un fonctionnement distinct entre un environnement d’exécution Standard/riche (REE) et un environnement d’exécution approuvé (TEE) quand celui-ci est proposé par des technologies matérielles. Les interfaces spécifiques à un rôle permettent aux composants principaux d’IoT Edge d’assurer l’intégrité de l’appareil IoT Edge et de son fonctionnement.

#### <a name="cloud-interface"></a>Interface Cloud

L’interface cloud permet au démon de sécurité IoT Edge d’accéder à des services cloud comme les compléments cloud pour la sécurité des appareils, par exemple le renouvellement de la sécurité.  Par exemple, le démon de sécurité IoT Edge utilise actuellement cette interface pour accéder au service Azure IoT Hub [Device Provisioning](../iot-dps/index.yml) pour la gestion du cycle de vie des identités d’appareil.  

#### <a name="management-api"></a>API de gestion

Le démon de sécurité IoT Edge offre une API de gestion, qui est appelée par l’agent IoT Edge lors de la création, du démarrage, de l’arrêt et de la suppression d’un module IoT Edge. Le démon de sécurité stocke les « inscriptions » pour tous les modules actifs. Ces inscriptions mappent les identités d’un module à certaines propriétés du module. Ces propriétés du module sont, par exemple, l’identificateur du processus (pid) en cours d’exécution dans le conteneur ou le hachage du contenu du conteneur Docker.

Elles sont utilisées par l’API de charge de travail (décrite ci-dessous) pour vérifier que l’appelant est autorisé à effectuer une action.

L’API de gestion est une API avec privilèges, qui peut être appelée seulement depuis l’agent IoT Edge.  Étant donné que le démon de sécurité IoT Edge amorce et démarre l’agent IoT Edge, il vérifie que l’agent IoT Edge n’a pas été falsifié, puis il peut créer une inscription implicite pour l’agent IoT Edge. Le même processus d’attestation qui utilise des API de charge de travail est utilisé pour restreindre l’accès de l’API de gestion à l’agent IoT Edge uniquement.

#### <a name="container-api"></a>API de conteneur

L’API de conteneur interagit avec le système de conteneur utilisé pour la gestion des modules, comme Moby ou Docker.

#### <a name="workload-api"></a>API de charge de travail

L’API de charge de travail est accessible à tous les modules. Il fournit à un module une preuve d’identité (un jeton signé avec la racine HSM ou un certificat X509) et le bundle de confiance correspondant. Le bundle de confiance contient les certificats d’autorité de certification de tous les autres serveurs que les modules doivent approuver.

Le démon de sécurité IoT Edge utilise un processus d’attestation pour protéger cette API. Quand un module appelle cette API, le démon de sécurité tente de trouver une inscription pour l’identité. En cas de réussite, il utilise les propriétés de l’inscription pour mesurer le module. Si le résultat du processus de mesure correspond à l’inscription, une nouvelle preuve d’identité est générée. Les certificats d’autorité de certification correspondants (bundle de confiance) sont retournés au module.  Le module utilise ce certificat pour vous connecter à IoT Hub, les autres modules, ou démarrer un serveur. Quand la date d’expiration du jeton signé ou du certificat approche, il est de la responsabilité du module de demander un nouveau certificat.

### <a name="integration-and-maintenance"></a>Intégration et maintenance

Microsoft tient à jour le code principal pour le [démon de sécurité IoT Edge sur GitHub](https://github.com/Azure/iotedge/tree/master/edgelet).

#### <a name="installation-and-updates"></a>Installation et mises à jour

L’installation et les mises à jour du démon de sécurité IoT Edge sont gérées via le système de gestion des packages du système d’exploitation. Les appareils IoT Edge avec racine matérielle de confiance doivent fournir un renforcement de l’intégrité du démon en gérant le cycle de vie de celui-ci via le démarrage sécurisé et les systèmes de gestion des mises à jour. Les fabricants d’appareils doivent explorer ces voies en fonction des capacités de leurs appareils respectifs.

#### <a name="versioning"></a>Contrôle de version

Le runtime IoT Edge effectue le suivi et signale la version du démon de sécurité IoT Edge. La version est signalée comme l’attribut *runtime.platform.version* de la propriété rapportée du module d’agent IoT Edge.

### <a name="hardware-security-module-platform-abstraction-layer-hsm-pal"></a>Abstraction calque de la plateforme du module de sécurité matériel (PAL HSM)

La publication de HSM PAL extrait toutes les racines du matériel de confiance pour isoler le développeur ou l’utilisateur de IoT Edge de leur complexité.  Il comprend une combinaison d’API et de procédures de communications entre domaines, par exemple la communication entre un environnement d’exécution standard et une enclave sécurisée.  L’implémentation réelle de la PAL HSM varie selon le matériel sécurisé en cours d’utilisation. Son existence permet d’utiliser n’importe quel processeur de silicium sécurisé.

## <a name="secure-silicon-root-of-trust-hardware"></a>Racine de silicium sécurisé du matériel de confiance

Le silicium sécurisé est nécessaire à l’approbation d’ancrage dans le matériel de l’appareil IoT Edge.  Un silicium sécurisé se présente en plusieurs variétés pour inclure le Module de plateforme sécurisée (TPM), l’élément sécurisé incorporé (eSE), ARM TrustZone, Intel SGX et des technologies de silicium sécurisé personnalisé.  L’utilisation de la racine de confiance « silicium » sécurisée sur les appareils est recommandée, en raison des menaces associées à l’accessibilité physique des appareils IoT.

## <a name="iot-edge-security-manager-integration-and-maintenance"></a>Maintenance et intégration de gestionnaire de sécurité IoT Edge

Le gestionnaire de sécurité IoT Edge a pour but d’identifier et d’isoler les composants chargés d’assurer la sécurité et l’intégrité de la plateforme Azure IoT Edge pour un renforcement personnalisé. Les tierces parties, telles que les fabricants d’appareils, doivent utiliser les fonctionnalités de sécurité personnalisées qui sont disponibles pour les composants matériels de leurs appareils.  Reportez-vous à la section Étapes suivantes pour obtenir des liens qui montrent comment renforcer le gestionnaire de sécurité Azure IoT avec le module de plateforme sécurisée (TPM) sur les plateformes Linux et Windows. Ces exemples utilisent des logiciels ou des modules de plateforme sécurisée virtuels, mais ils s’appliquent directement à l’utilisation des appareils TPM discrets.  

## <a name="next-steps"></a>Étapes suivantes

Lire le blog sur [Sécuriser la périphérie intelligente](https://azure.microsoft.com/blog/securing-the-intelligent-edge/).

Créer et provisionner un [appareil IoT Edge à l’aide d’un TPM virtuel sur une machine virtuelle Linux](how-to-auto-provision-simulated-device-linux.md)

Créer et provisionner un [appareil IoT Edge avec un TPM simulé sur Windows](how-to-auto-provision-simulated-device-windows.md)