---
title: Infrastructure de sécurité - Azure IoT Edge | Microsoft Docs
description: Découvrez les normes de sécurité, d’authentification et d’autorisation qui ont été utilisées pour développer Azure IoT Edge et que vous devriez prendre en compte pour concevoir votre solution
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/30/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 3212493963805de3c8845ec494d87fc92d72998a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76760111"
---
# <a name="security-standards-for-azure-iot-edge"></a>Normes de sécurité pour Azure IoT Edge

Azure IoT Edge répond aux risques qui sont inhérents au déplacement de vos données et de votre analytique vers la périphérie intelligente. Les standards de sécurité IoT Edge apportent un compromis entre la flexibilité pour différents types de scénarios de déploiement et la protection que vous attendez de tous les services Azure.

IoT Edge s’exécute sur différents modèles de matériels, prend en charge plusieurs systèmes d’exploitation et s’applique à différents scénarios de déploiement. Plutôt que de proposer des solutions concrètes pour des scénarios spécifiques, IoT Edge offre une infrastructure de sécurité extensible basée sur des principes concrets conçus pour la mise à l’échelle. Le risque lié à un scénario de déploiement donné dépend de nombreux facteurs, notamment :

* Propriété de la solution
* Géographie du déploiement
* Sensibilité des données
* Confidentialité
* Verticalité des applications
* Exigences réglementaires

Cet article fournit une vue d’ensemble du framework de sécurité IoT Edge. Pour plus d’informations, consultez [Securing the intelligent edge (Sécurisation de la périphérie intelligente)](https://azure.microsoft.com/blog/securing-the-intelligent-edge/).

## <a name="standards"></a>Standards

Les standards facilitent l’examen et l’implémentation, qui sont des éléments clés de la sécurité. Une solution de sécurité doit pouvoir être examinée et évaluée afin de gagner la confiance des utilisateurs, et ne doit pas être un obstacle au déploiement. La conception du framework permettant de sécuriser Azure IoT Edge est basée sur des protocoles de sécurité testés visant à vous familiariser avec elle et à l’utiliser plus facilement.

## <a name="authentication"></a>Authentification

Quand vous déployez une solution IoT, vous devez être sûr que seuls les acteurs, les appareils et les modules approuvés ont accès à votre solution. L’authentification par certificat est le mécanisme principal d’authentification pour la plateforme Azure IoT Edge. Ce mécanisme est dérivé d’un ensemble de standards de l’IETF (Internet Engineering Task Force) régissant l’infrastructure à clé publique (PKiX).

Tous les appareils, modules et acteurs qui interagissent avec l’appareil Azure IoT Edge doivent disposer d'identités de certificat uniques. Ces conseils s'appliquent en présence d'interactions physiques ou via une connexion réseau. Tous les scénarios ou composants ne se prêtent pas nécessairement à l’authentification basée sur un certificat : l’extensibilité de l’infrastructure de sécurité offre donc des alternatives sécurisées.

Pour plus d’informations, consultez [Utilisation des certificats par Azure IoT Edge](iot-edge-certs.md).

## <a name="authorization"></a>Autorisation

Selon le principe des privilèges minimum, les utilisateurs et les composants d’un système doivent uniquement avoir accès à un ensemble minimal de ressources et aux données dont ils ont besoin pour exécuter leurs rôles. Les appareils, modules et acteurs doivent uniquement accéder aux ressources et aux données qui se trouvent dans leur étendue d’autorisations, et uniquement quand l’architecture le permet. Certaines autorisations sont configurables avec des privilèges suffisants, et d’autres sont appliquées au niveau de l’architecture. Par exemple, certains modules peuvent être autorisés à se connecter à Azure IoT Hub. Cependant, il n’y a aucune raison pour qu’un module d’un appareil IoT Edge doive accéder au jumeau d’un module sur un autre appareil IoT Edge.

Parmi les autres schémas d’autorisation figurent les droits de signature de certificat et le contrôle d’accès en fonction du rôle (RBAC).

## <a name="attestation"></a>Attestation

L’attestation garantit l’intégrité des bits logiciels, ce qui est important pour la détection et la prévention des programmes malveillants. L’infrastructure de sécurité Azure IoT Edge classe l’attestation dans trois catégories principales :

* Attestation statique
* Attestation d’exécution
* Attestation logicielle

### <a name="static-attestation"></a>Attestation statique

L’attestation statique vérifie l’intégrité de tous les logiciels présents sur un appareil lors du démarrage, notamment le système d’exploitation, tous les runtimes et les informations de configuration. Comme l’attestation statique se produit lors du démarrage, elle est souvent appelée « démarrage sécurisé ». L’infrastructure de sécurité des appareils IoT Edge s’étend aux fabricants et incorpore des fonctionnalités matérielles sécurisées qui assurent les processus d’attestation statique. Ces processus incluent le démarrage sécurisé et la mise à niveau sécurisée du microprogramme. Le fait de collaborer étroitement avec les fournisseurs de silicium permet d’éliminer les couches de microprogramme superflues, réduisant ainsi la surface d’attaque.

### <a name="runtime-attestation"></a>Attestation d’exécution

Une fois qu’un système a démarré de façon sécurisée, des systèmes conçus pour être fiables doivent détecter les tentatives d’injection de logiciels malveillants et prendre les contre-mesures appropriées. Les attaques de programmes malveillants peuvent cibler les ports et les interfaces du système. Si des personnes malveillantes ont un accès physique à un appareil, elles peuvent falsifier son contenu ou utiliser des attaques par canal auxiliaire pour y accéder. Ce contenu malveillant, qui peut se présenter sous la forme de logiciels malveillants ou de modifications non autorisées de la configuration, ne peut pas être détecté par l’attestation statique, car il est injecté après le processus de démarrage. Les contre-mesures proposées ou appliquées par les composants matériels de l’appareil permettent d’atténuer ces menaces. Le framework de sécurité pour IoT Edge appelle explicitement des extensions afin de combattre les menaces au moment de l’exécution.  

### <a name="software-attestation"></a>Attestation logicielle

Tous les systèmes sains, notamment les systèmes de périphérie intelligente, ont besoin de correctifs et de mises à niveau. La sécurité est importante pour les processus de mise à jour, car ceux-ci peuvent sinon constituer des vecteurs de menaces potentiels. L’infrastructure de sécurité pour IoT Edge appelle les mises à jour via des packages mesurés et signés, afin de garantir l’intégrité et l’authentification de la source des packages. Ce standard s’applique à tous les systèmes d’exploitation et logiciels d’application.

## <a name="hardware-root-of-trust"></a>Racine matérielle de confiance

Pour de nombreux appareils de périphérie intelligente, en particulier ceux qui sont accessibles par des acteurs potentiellement malveillants, la sécurité matérielle constitue la dernière défense pour la protection. Pour de tels déploiements, il est essentiel de disposer de composants matériels inviolables. Azure IoT Edge encourage les fournisseurs de composants matériels en silicium sécurisés à proposer différentes versions de racine matérielle de confiance afin de prendre en charge différents scénarios de déploiement et profils de risque. La confiance accordée aux composants matériels est garantie par les standards de protocole de sécurité courants tels que le Trusted Platform Module (ISO/IEC 11889) et le Device Identifier Composition Engine (DICE) du Trusted Computing Group. Les technologies intégrées et sécurisées comme TrustZones et Software Guard Extensions (SGX) permettent également aux composants matériels d’être considérés comme étant de confiance.

## <a name="certification"></a>Certification

Pour aider les clients à prendre des décisions en connaissance de cause quand ils font l’acquisition d’appareils Azure IoT Edge pour leur déploiement, l’infrastructure IoT Edge inclut des exigences de certification. À la base de ces exigences figurent des certifications relatives aux revendications de sécurité et des certifications relatives à la validation de l’implémentation de sécurité. Par exemple, une certification de revendication de sécurité signifie que l’appareil IoT Edge utilise du matériel sécurisé connu pour résister aux attaques de démarrage. Un certificat de validation signifie que le matériel sécurisé a été correctement implémenté pour offrir cette fonctionnalité dans l’appareil. Conformément au principe de simplicité, le framework vise à réduire au maximum la charge de certification.

## <a name="extensibility"></a>Extensibilité

Étant donné que la technologie IoT pilote différents types de transformations des entreprises, la sécurité doit évoluer en parallèle afin de gérer les scénarios émergents. L’infrastructure de sécurité Azure IoT Edge repose sur une base solide et extensible dans différentes dimensions de façon à inclure ce qui suit :

* Services de sécurité internes tels que le service Device Provisioning pour Azure IoT Hub.
* Services tiers, comme des services de sécurité managés pour différents domaines d’application (l’industrie ou les soins de santé) ou centrés sur la technologie (surveillance de la sécurité sur les réseaux à maillage ou les services d’attestation matérielle « silicium ») via un important réseau de partenaires.
* Systèmes hérités devant inclure la récupération avec des stratégies de sécurité de substitution, comme l’utilisation d’une technologie sécurisée autre que les certificats pour la gestion de l’authentification et de l’identité.
* Matériel sécurisé pour l’adoption des nouvelles technologies matérielles sécurisées et des contributions des fournisseurs de silicium.

Au final, la sécurisation de la périphérie intelligente nécessite des contributions d’une communauté ouverte motivée par un intérêt commun pour la sécurisation de l’IoT. Ces contributions peuvent se présenter sous la forme de technologies sécurisées ou de services. L’infrastructure de sécurité Azure IoT Edge offre une base solide pour la sécurité, qui est extensible afin de fournir une couverture maximale, et permet d’offrir le même niveau de confiance et d’intégrité dans la périphérie intelligente que le cloud Azure.  

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la façon dont Azure IoT Edge [sécurise la périphérie intelligente](https://azure.microsoft.com/blog/securing-the-intelligent-edge/).
