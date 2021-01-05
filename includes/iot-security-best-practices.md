---
title: Fichier include
description: Fichier include
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 08/07/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 4fdb891d668d99644d8a9ed9c15d158e65d53ba5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "72793126"
---
Sécuriser une infrastructure IoT implique la mise en œuvre d’une stratégie complète et rigoureuse. Dans le cadre de cette stratégie, vous devez sécuriser les données dans le cloud, protéger l’intégrité des données transitant sur le réseau Internet public et configurer des appareils en toute sécurité. Chaque couche contribue à garantir la sécurité de l’infrastructure globale.

## <a name="secure-an-iot-infrastructure"></a>Sécuriser une infrastructure IoT

Cette stratégie de sécurité complète peut être développée et appliquée avec la participation active des différents acteurs impliqués dans la fabrication, le développement et le déploiement de l’infrastructure et des appareils IoT. Voici une description générale de ces acteurs.

* **Fabricant/intégrateur de matériel IoT** : il s’agit généralement de fabricants de matériels IoT déployés, d’intégrateurs qui assemblent le matériel de différents fabricants ou de fournisseurs de matériels dédiés à un déploiement IoT réalisé ou intégré par d’autres fournisseurs.

* **Développeur de solutions IoT** : le développement d’une solution IoT est généralement assuré par un développeur de solutions. Ce développeur peut faire partie d’une équipe interne ou être un intégrateur système spécialisé dans cette activité. Le développeur de solutions IoT peut développer plusieurs composants de la solution IoT en partant de zéro, intégrer de nombreux composants du commerce ou open source ou encore adopter des accélérateurs de solution moyennant une légère adaptation.

* **Responsable du déploiement de solutions IoT** : une fois développée, une solution IoT doit être déployée sur le terrain. Ce processus implique le déploiement du matériel, l’interconnexion des appareils et le déploiement des solutions sur les appareils ou dans le cloud.

* **Opérateur de solutions IoT** : une fois déployée, la solution IoT nécessite une exploitation, une surveillance, des mises à niveau et une maintenance sur le long terme. Ces tâches peuvent être assurées par une équipe interne comprenant des spécialistes en technologies de l’information, des équipes d’exploitation et de maintenance du matériel et des spécialistes du domaine qui contrôlent le comportement de l’infrastructure IoT globale.

Les sections qui suivent décrivent les bonnes pratiques associées à chacun de ces acteurs et facilitant le développement, le déploiement et l’exploitation d’une infrastructure IoT sécurisée.

## <a name="iot-hardware-manufacturerintegrator"></a>Fabricant/intégrateur de matériel IoT

Voici les bonnes pratiques pour les intégrateurs de matériel et les fabricants de matériel IoT.

* **Concevoir le matériel selon les exigences minimales** : la conception du matériel doit répondre aux fonctionnalités minimales requises pour son fonctionnement, et rien de plus. Par exemple, des ports USB sont inclus uniquement s’ils sont nécessaires au fonctionnement de l’appareil. Les fonctionnalités supplémentaires exposent l’appareil à des vecteurs d’attaque indésirables qui doivent être évités.

* **Protéger le matériel contre les falsifications** : intégrez un mécanisme permettant de détecter toute altération physique telle que l’ouverture du capot ou la suppression d’une partie de l’appareil. Les signaux de falsification peuvent être inclus dans le flux de données chargé sur le cloud, permettant d’informer les opérateurs de ces événements.

* **Intégrer la sécurité au matériel** : si le coût des marchandises vendues le permet, intégrez des fonctionnalités de sécurité, telles qu’un stockage sécurisé et chiffré, ou une fonctionnalité de démarrage basée sur un module de plateforme sécurisée (TPM). Ces fonctionnalités renforcent la sécurité des appareils et facilitent la protection de l’infrastructure IoT globale.

* **Sécuriser les mises à jour** : le microprogramme doit faire l’objet de mises à jour durant toute la durée de vie de l’appareil. Prévoyez des chemins d’accès sécurisés pour les mises à jour et un chiffrement garanti des versions du microprogramme. L’appareil sera ainsi sécurisé pendant et après les mises à jour.

## <a name="iot-solution-developer"></a>Développeur de solutions IoT

Voici les bonnes pratiques pour les développeurs de solutions IoT :

* **Respecter la méthodologie de développement de logiciels sécurisés** : le développement de logiciels sécurisés requiert une réflexion de bout en bout sur la sécurité, depuis le lancement du projet jusqu’à son implémentation, aux tests et au déploiement. Les choix des plateformes, des langages et des outils sont influencés par cette méthodologie. Microsoft Security Development Lifecycle offre une approche pas à pas de la création de logiciels sécurisés.

* **Choisir des logiciels open source avec précaution** : les logiciels open source offrent la possibilité de développer des solutions rapidement. Quand vous choisissez des logiciels open source, tenez compte du niveau d’activité de la communauté vis-à-vis de chaque composant open source. Avec une communauté active, les logiciels bénéficient d’un vaste soutien, et les problèmes sont identifiés et traités. Ce n’est nécessairement pas le cas d’un projet logiciel open source obscur et inactif : les problèmes ne sont probablement pas identifiés.

* **Intégrer les composants avec précaution** : les failles de sécurité logicielle se situent, pour nombre d’entre elles, à la limite des bibliothèques et des API. Des fonctionnalités qui ne sont pas requises pour le déploiement concerné peuvent rester disponibles par le biais d’une couche API. Pour garantir la sécurité globale, vérifiez que toutes les interfaces des composants intégrés sont exemptes de failles de sécurité.

## <a name="iot-solution-deployer"></a>Responsable du déploiement de solutions IoT

Voici les bonnes pratiques pour les responsables du déploiement de solutions IoT :

* **Déployer du matériel de façon sécurisée** : les déploiements IoT peuvent nécessiter un déploiement du matériel dans des lieux non sécurisés, par exemple dans des espaces publics ou des régions non supervisées. Dans ce cas, assurez-vous que le déploiement matériel est protégé contre les falsifications, autant que faire se peut. Si le matériel est doté de ports, notamment de ports USB, vérifiez qu’ils sont correctement protégés. De nombreux vecteurs d’attaque peuvent les utiliser comme point d’entrée.

* **Protéger les clés d’authentification** : au cours du déploiement, chaque appareil requiert des ID et les clés d’authentification associées, générées par le service cloud. Protégez physiquement ces clés, même après le déploiement. Une clé compromise pourrait être utilisée par un appareil malveillant pour usurper l’identité d’un appareil existant.

## <a name="iot-solution-operator"></a>Opérateur de solutions IoT

Voici les bonnes pratiques pour les opérateurs de solutions IoT :

* **Garder le système à jour** : vérifiez que les systèmes d’exploitation et tous les pilotes des appareils sont au niveau des dernières versions. Si vous activez les mises à jour automatiques dans Windows 10 (IoT ou autres références), Microsoft le met constamment à jour. Les appareils IoT bénéficient ainsi d’un système d’exploitation sécurisé. Mettre régulièrement à jour les autres systèmes d’exploitation (tels que Linux) permet également de les protéger contre les attaques malveillantes.

* **Protéger le système contre les activités malveillantes** : installez les dernières fonctionnalités de protection antivirus et anti-programme malveillant sur le système d’exploitation de chaque appareil, quand celui-ci le permet. Cette pratique peut contribuer à atténuer la plupart des menaces externes. Vous pouvez protéger la plupart des systèmes d’exploitation contre les menaces en prenant les mesures appropriées.

* **Effectuer des audits fréquemment** : il est essentiel de rechercher les problèmes de sécurité en effectuant un audit de l’infrastructure IoT pour traiter les incidents de sécurité. La plupart des systèmes d’exploitation offrent une journalisation des événements intégrée. Examinez les journaux fréquemment pour vous assurer qu’aucune violation de sécurité n’a été détectée. Les informations d’audit peuvent être envoyées sous la forme d’un flux de télémétrie au service cloud, où elles peuvent être analysées.

* **Protéger physiquement l’infrastructure IoT** : les attaques de sécurité les plus menaçantes pour l’infrastructure IoT sont lancées par le biais d’un accès physique aux appareils. Une pratique de sécurité essentielle consiste à protéger les appareils contre l’utilisation malveillante des ports USB et d’autres modes d’accès physiques. Pour détecter toute violation, vous devez notamment journaliser l’accès physique, comme l’utilisation des ports USB. Là encore, Windows 10 (IoT et autres références) peut assurer une journalisation détaillée de ces événements.

* **Protéger les informations d’identification cloud** : les informations d’identification cloud utilisées pour configurer et exploiter un déploiement IoT constituent peut-être le moyen le plus simple d’accéder à un système IoT et de le compromettre. Protégez les informations d’identification en modifiant fréquemment le mot de passe et en évitant de les utiliser sur des machines publiques.

Les fonctionnalités des différents appareils IoT peuvent varier. Certains appareils peuvent être des ordinateurs exécutant des systèmes d’exploitation courants, tandis que d’autres peuvent exécuter des systèmes d’exploitation très légers. Les bonnes pratiques en matière de sécurité décrites précédemment peuvent s’appliquer à ces appareils selon différents degrés. Il se peut que les fabricants de ces appareils établissent leurs propres bonnes pratiques en matière de sécurité et de déploiement. Le cas échéant, elles doivent être observées également.

Certains appareils existants et limités n’ont peut-être pas été conçus spécifiquement pour un déploiement IoT. Il se peut qu’ils ne soient pas en mesure de chiffrer les données, de se connecter à Internet ou d’assurer un audit avancé. Dans ce cas, une passerelle de champ moderne et sécurisée peut agréger les données des appareils existants et offrir la sécurité requise pour connecter ces appareils à Internet. Les passerelles de champ peuvent assurer l’authentification sécurisée, la négociation de sessions chiffrées, la réception des commandes du cloud et beaucoup d’autres fonctionnalités de sécurité.