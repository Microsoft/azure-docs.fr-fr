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
ms.openlocfilehash: 28609ad27330ae4ea5ea7c0d02d5a61181fbe0df
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95561470"
---
Lorsque vous concevez un système, il est important de comprendre les menaces potentielles qui pèsent sur ce dernier et d’ajouter les défenses appropriées en conséquence, à mesure de sa conception et de la création de son architecture. Il est particulièrement important de concevoir le produit dès le début dans une optique de sécurité : comprendre comment une personne malveillante peut compromettre un système contribue à s’assurer que les préventions adéquates sont en place dès le début.

## <a name="security-starts-with-a-threat-model"></a>La sécurité commence par un modèle de menace

Microsoft a recours depuis longtemps à des modèles de menace pour ses produits, et a rendu publiquement disponible le processus de modélisation des menaces de l’entreprise. L’expérience de la société démontre que la modélisation présente des avantages inattendus qui vont bien au-delà de la compréhension immédiate des menaces les plus préoccupantes. Par exemple, elle permet également d’ouvrir une discussion avec d’autres personnes non-membres de l’équipe de développement, ce qui peut conduire à de nouvelles idées et à des améliorations du produit.

L’objectif de la modélisation des menaces est de comprendre comment une personne malveillante peut être en mesure de compromettre un système avant de s’assurer que les préventions appropriées sont en place. La modélisation des menaces impose à l’équipe de conception de tenir compte des préventions lors de la conception du système plutôt qu’à l’issue de son déploiement. Ce point est extrêmement important, car dans la pratique modifier des solutions de défense de la sécurité sur une multitude d’appareils est infaisable, sujet aux erreurs et rend les clients vulnérables.

De nombreuses équipes de développement font un excellent travail en prenant en compte les exigences fonctionnelles du système dont bénéficient les clients. Néanmoins, l’identification des modes non évidents de compromission du système par une personne malveillante se révèle plus difficile. La modélisation des menaces peut aider les équipes de développement à comprendre ce qu’une personne malveillante peut effectuer et pourquoi. La modélisation des menaces est un processus structuré qui crée une discussion sur les décisions en matière de conception de la sécurité dans le système, et qui apporte des modifications tout au long de la conception qui ont des incidences sur la sécurité. Si un modèle de menace consiste tout simplement en un document, celui-ci représente un moyen idéal de s’assurer de la continuité des connaissances et de la conservation des leçons retenues. Par ailleurs, il contribue à l’intégration rapide des nouvelles équipes. Enfin, la modélisation des menaces vous permet de prendre en considération d’autres aspects de la sécurité, par exemple les engagements en termes de sécurité que vous souhaitez fournir à vos clients. Ces engagements pris conjointement avec la modélisation des menaces documentent et orientent les tests de votre solution IoT.

### <a name="when-to-do-threat-modeling"></a>Quand modéliser les menaces

La [modélisation des menaces](https://www.microsoft.com/en-us/sdl/adopt/threatmodeling.aspx) apporte davantage de valeur si elle est incorporée dans la phase de conception. Lors de la conception, vous bénéficiez d’une plus grande flexibilité pour apporter les modifications qui élimineront les menaces. L’élimination des menaces au cours de la conception est en effet le résultat souhaité. C’est en effet bien plus facile que d’ajouter des préventions, les tester et s’assurer qu’elles restent en cours. Par ailleurs, procéder à de telles éliminations n’est pas toujours possible. Il est plus difficile d’éliminer des menaces lorsqu’un produit progresse dans son cycle de vie, et cela nécessite plus de travail et des compromis beaucoup plus difficiles que la mise en place d’une modélisation des menaces au début du développement.

### <a name="what-to-consider-for-threat-modeling"></a>Quels facteurs intégrer dans la modélisation des menaces

Vous devez envisager la solution dans son ensemble dans le modèle de menace et vous concentrer sur les domaines suivants :

* Les fonctionnalités de sécurité et de confidentialité
* Les fonctionnalités dont les échecs relèvent de la sécurité
* Les fonctionnalités qui touchent une délimitation d’approbation

### <a name="who-performs-threat-modeling"></a>Qui effectue la modélisation des menaces

La modélisation des menaces est un processus comme un autre. Il est judicieux de traiter le document de modèle de menace comme tout autre composant de la solution et de le valider. De nombreuses équipes de développement font un excellent travail en prenant en compte les exigences fonctionnelles du système dont bénéficient les clients. Néanmoins, l’identification des modes non évidents de compromission du système par une personne malveillante se révèle plus difficile. La modélisation des menaces peut aider les équipes de développement à comprendre ce qu’une personne malveillante peut effectuer et pourquoi.

### <a name="how-to-perform-threat-modeling"></a>Comment effectuer la modélisation des menaces

Le processus de modélisation des menaces est composé des quatre étapes suivantes :

* Modéliser l’application
* Énumérer les menaces
* Prévenir les menaces
* Valider les préventions

#### <a name="the-process-steps"></a>Étapes du processus

Vous devez garder à l’esprit trois règles élémentaires lorsque vous créez un modèle de menace :

1. Créez un diagramme de l’architecture de référence.

2. Commencez par la logique de largeur. Obtenez d’abord une vue d’ensemble du système pour le comprendre avant de l’explorer en profondeur. Cette approche garantit que vous explorez en profondeur les bons emplacements.

3. C’est vous qui conduisez le processus, et non l’inverse. Si vous rencontrez un problème lors de la phase de modélisation et souhaitez l’examiner, n’hésitez pas ! Ne vous sentez pas obligé de suivre ces étapes à la lettre.

#### <a name="threats"></a>Menaces

Les quatre éléments principaux d’un modèle de menace sont les suivants :

* Processus, comme les services web, services Win32 et démons *nix. Certaines entités complexes (comme les passerelles et capteurs sur site) peuvent être résumées sous la forme d’un processus lorsque l’exploration technique de ces zones n’est pas possible.

* Magasins de données (tout emplacement de stockage des données, par exemple un fichier de configuration ou une base de données)

* Flux de données (lorsque des données circulent entre d’autres éléments de l’application)

* Entités externes (tout ce qui interagit avec le système sans être sous le contrôle de l’application, par exemple des flux d’utilisateur et des programmes transmis par satellite)

Tous les éléments du diagramme architectural sont exposés à différentes menaces ; nous allons utiliser le mnémonique STRIDE dans cet article. Lisez le blog [Threat Modeling Again, STRIDE](/archive/blogs/larryosterman/threat-modeling-again-stride) (Modélisation des menaces, STRIDE) pour en savoir plus sur les éléments STRIDE.

Divers éléments du diagramme d’application sont exposés à certaines menaces STRIDE :

* Les processus sont exposés aux menaces STRIDE.
* Les flux de données sont exposés aux menaces TID.
* Les magasins de données sont exposés aux menaces TID et parfois R, si ces magasins sont des fichiers journaux.
* Les entités externes sont exposées aux menaces SRD.

## <a name="security-in-iot"></a>Sécurité des solutions IoT

Les périphériques connectés à usage spécifique présentent un nombre important de surfaces d’exposition d’interaction potentielle et de modèles d’interaction, qui doivent être pris en considération pour fournir une infrastructure afin de sécuriser l’accès numérique à ces périphériques. Le terme « accès numérique » est utilisé ici pour faire la distinction par rapport aux opérations effectuées via l’interaction directe des périphériques où la sécurité d’accès est assurée par le biais d’un contrôle d’accès physique. C’est le cas par exemple, lorsque vous placez le périphérique dans une pièce dont la porte est équipée d’un verrou. Si l’accès physique ne peut pas être refusé à l’aide de logiciels et de matériels, des mesures peuvent être prises pour interdire l’accès physique au système.

À mesure que vous explorez les modèles d’interaction, examinez les termes « contrôle d’appareil » et « données d’appareil » avec le même niveau d’attention. Le terme « contrôle de périphérique » désigne les informations qui sont fournies à un périphérique par un tiers dans le but de modifier ou d’influencer son comportement quant à son état ou l’état de son environnement. Le terme « données de périphérique » désigne les informations qu’un périphérique transmet à un tiers sur son état et l’état observé de son environnement.

Pour optimiser les meilleures pratiques en matière de sécurité, il est recommandé de répartir une architecture IoT standard en plusieurs zones/composants dans le cadre de l’exercice de modélisation des menaces. Ces zones sont décrites en détail dans cette section et incluent les éléments suivants :

* périphérique,
* passerelle de champ,
* passerelles cloud,
* prenant en charge les revendications.

Les zones constituent un moyen de segmenter une solution au sens large. Chaque zone possède souvent ses propres données et exigences d’authentification et d’autorisation. Les zones permettent également d’isoler les dommages et de restreindre l’impact des zones de confiance faible sur les zones de confiance plus élevée.

Chaque zone est séparée par une délimitation d’approbation, qui est indiquée par un trait rouge en pointillés dans le diagramme suivant. Elle représente une transition des données/informations d’une source vers une autre. Durant cette transition, les données/informations peuvent être exposées aux éléments STRIDE (usurpation d’identité, falsification, répudiation, divulgation d’informations, déni de service et élévation de privilège).

![Zones de sécurité IoT](media/iot-security-architecture/iot-security-architecture-fig1.png) 

Les composants représentés au sein de chaque délimitation sont également exposés aux éléments STRIDE, ce qui permet d’obtenir une vue complète de la modélisation des menaces de la solution. Les sections suivantes exposent dans le détail les composants, les problèmes de sécurité spécifiques, ainsi que les solutions qui doivent être mises en place.

Les sections suivantes présentent les composants standard qui se trouvent généralement dans ces zones.

### <a name="the-device-zone"></a>Zone de périphérique

L’environnement du périphérique est l’espace physique immédiat entourant le périphérique, dans lequel l’accès physique et/ou l’accès numérique pair à pair du « réseau local » au périphérique sont possibles. Un « réseau local » est supposé être un réseau distinct et isolé de l’Internet public, mais potentiellement relié par un pont à celui-ci. Il inclut une technologie de radio sans fil à courte portée qui permet la communication d’égal à égal des périphériques. Il n’inclut *aucune* technologie de virtualisation créant l’illusion d’un tel réseau. Il n’inclut pas non plus de réseaux d’opérateurs publics nécessitant la communication de deux appareils dans l’espace du réseau public s’ils doivent établir une relation de communication d’égal à égal.

### <a name="the-field-gateway-zone"></a>Zone de passerelle de champ

Une passerelle de champ est un périphérique/équipement ou un logiciel serveur à usage général qui agit en tant qu’activateur de communication, mais elle peut également servir de système de contrôle de périphérique et de concentrateur de traitement des données du périphérique. La zone de passerelle de champ inclut la passerelle de champ proprement dite et tous les périphériques qui y sont connectés. Comme son nom l’indique, une passerelle de champ fonctionne hors des installations de traitement des données dédiées, est généralement associée à un site, peut être exposée à une intrusion physique et dispose d’une redondance opérationnelle limitée. En résumé, une passerelle de champ est généralement un élément que l’on peut toucher et saboter tout en connaissant sa fonction.

Une passerelle de champ diffère d’un routeur de trafic simple dans la mesure où il a un rôle actif dans la gestion de l’accès et du flux d’informations, ce qui signifie qu’il s’agit d’un terminal de session ou de connexion réseau et d’entité adressable d’application. En revanche, un périphérique NAT ou un pare-feu n’est pas qualifié de passerelle de champ, car il ne s’agit pas de terminaux de connexion ou de session. Ils acheminent (ou bloquent) les connexions ou les sessions qui transitent par eux. La passerelle de champ comporte deux surfaces d’exposition distinctes. L’une expose les périphériques qui y sont connectés et représente l’intérieur de la zone, et l’autre expose toutes les parties externes et constitue la périphérie de la zone.

### <a name="the-cloud-gateway-zone"></a>Zone de passerelle cloud

Une passerelle cloud est un système qui permet une communication distante entre des appareils ou des passerelles de champ à partir de plusieurs sites via l’espace réseau public, généralement vers un système d’analyse de données et de contrôle basé sur le cloud, une fédération de tels systèmes. Dans certains cas, une passerelle cloud peut faciliter l’accès immédiat aux appareils à usage spécifique à partir de terminaux comme les tablettes ou les téléphones. Dans le contexte présenté ici, le « cloud » désigne un système de traitement des données dédié qui n’est pas lié au même site que les périphériques connectés ou les passerelles de champ. Dans une zone de passerelle cloud, des mesures opérationnelles empêchent l’accès physique ciblé. Une zone de passerelle cloud n’est pas nécessairement exposée à une infrastructure de « cloud public ».  

Une passerelle cloud peut potentiellement être mappée à une superposition de virtualisation de réseau pour isoler la passerelle cloud et tous ses périphériques connectés ou passerelles de champ de tout autre trafic réseau. La passerelle cloud proprement dite n’est pas un système de contrôle d’appareil ni une fonction de traitement ou de stockage des données d’appareil. Ces fonctions servent d’interface avec la passerelle cloud. La zone de passerelle cloud inclut la passerelle cloud proprement dite ainsi que l’ensemble des passerelles de champ et des périphériques directement ou indirectement connectés à elle. La périphérie de la zone est une surface d’exposition distincte via laquelle toutes les parties externes communiquent.

### <a name="the-services-zone"></a>Zone des services

Pour ce contexte, un « service » est défini comme un composant ou un module logiciel qui interface avec des périphériques via une passerelle de champ ou cloud pour la collecte et l’analyse des données, ainsi que pour la commande et le contrôle. Les services sont des médiateurs. Sous leur identité, ils agissent pour les passerelles et les autres sous-systèmes, stockent et analysent les données, émettent de façon autonome des commandes vers les périphériques en fonction des lectures des données ou des planifications et exposent les informations et les fonctionnalités de contrôle aux utilisateurs finaux autorisés.

### <a name="information-devices-versus-special-purpose-devices"></a>Appareils d’information et appareils à usage spécifique

Les PC, les téléphones et les tablettes sont des appareils d’information essentiellement interactifs. Les téléphones et les tablettes sont explicitement optimisés via l’optimisation de la durée de vie de la batterie. De préférence, ils s’éteignent partiellement lorsque leur propriétaire n’est pas en communication ou lorsqu’ils ne fournissent pas de services tels que la lecture de musique ou le guidage de leur propriétaire vers un emplacement particulier. Du point de vue des systèmes, ces appareils de technologie de l’information font principalement office de proxy pour les personnes. Ce sont des « actionneurs » qui suggèrent des actions et des « capteurs » qui collectent des saisies.

Les périphériques à usage spécifique, des simples capteurs de température aux chaînes de fabrication complexes comportant des milliers de composants sont différents. Ils sont beaucoup plus étendus dans leur usage et même s’ils fournissent une interface utilisateur, ils sont largement étendus pour interfacer avec ou être intégrés aux ressources du monde physique. Ils mesurent et signalent des conditions d’environnement, activent des vannes, contrôlent des servomécanismes, déclenchent des alarmes, allument des lumières et effectuent bien d’autres tâches. Ils permettent d’effectuer des tâches pour lesquelles un appareil d’information est trop générique, trop coûteux, trop grand ou trop fragile. L’objectif réel permet de dicter immédiatement leur conception technique ainsi que le budget disponible pour leur production et leur durée de vie planifiée. La combinaison de ces deux facteurs clés limite le bilan énergétique opérationnel disponible, l’encombrement physique et donc les capacités de stockage, de calcul et de sécurité disponibles.

Si un problème se produit avec des appareils automatisés ou des périphériques contrôlables à distance, par exemple des défauts physiques ou des défauts de logique de contrôle, des intrusions et des manipulations non autorisées intentionnelles peuvent avoir lieu. Les lots de production peuvent être détruits, des bâtiments peuvent être pillés ou réduits en cendres, et des personnes peuvent être blessées, voire tuées. Il s’agit là d’une classe de dommages totalement différente de ceux rencontrés par une personne atteignant le plafond d’une carte de crédit volée. Le niveau de sécurité des périphériques qui permettent de déplacer des éléments, et des données de capteur qui commandent le déplacement d’éléments doit être supérieur à celui d’un commerce électronique ou d’une banque.

### <a name="device-control-and-device-data-interactions"></a>Interactions du contrôle de périphérique et des données de périphérique

Les périphériques connectés à usage spécifique présentent un nombre important de surfaces d’exposition d’interaction potentielle et de modèles d’interaction, qui doivent être pris en considération pour fournir une infrastructure afin de sécuriser l’accès numérique à ces périphériques. Le terme « accès numérique » est utilisé ici pour faire la distinction par rapport aux opérations effectuées via l’interaction directe des périphériques où la sécurité d’accès est assurée par le biais d’un contrôle d’accès physique. C’est le cas par exemple, lorsque vous placez le périphérique dans une pièce dont la porte est équipée d’un verrou. Si l’accès physique ne peut pas être refusé à l’aide de logiciels et de matériels, des mesures peuvent être prises pour interdire l’accès physique au système.

À mesure que vous explorez les modèles d’interaction, examinez les termes « contrôle d’appareil » et « données d’appareil » avec le même niveau d’attention en créant la modélisation des menaces. Le terme « contrôle de périphérique » désigne les informations qui sont fournies à un périphérique par un tiers dans le but de modifier ou d’influencer son comportement quant à son état ou l’état de son environnement. Le terme « données de périphérique » désigne les informations qu’un périphérique transmet à un tiers sur son état et l’état observé de son environnement.

## <a name="performing-threat-modeling-for-the-azure-iot-reference-architecture"></a>Modélisation des menaces pour l’architecture de référence Microsoft Azure IoT

Microsoft utilise l’infrastructure décrite précédemment pour effectuer la modélisation des menaces pour Azure IoT. La section suivante utilise l’exemple concret de l’architecture de référence Azure IoT pour montrer comment réfléchir à la modélisation des menaces pour IoT et comment traiter les menaces identifiées. Cet exemple identifie quatre domaines principaux :

* Périphériques et sources de données
* Transport de données
* Périphérique et traitement des événements
* Présentation

![Modélisation des menaces pour Azure IoT](media/iot-security-architecture/iot-security-architecture-fig2.png)

Le diagramme suivant présente une vue simplifiée de l’architecture IoT de Microsoft établie à l’aide un modèle de diagramme de flux de données utilisé par l’outil Microsoft de modélisation des menaces :

![Modélisation des menaces pour Azure IoT à l’aide de l’outil MS de modélisation des menaces](media/iot-security-architecture/iot-security-architecture-fig3.png)

Il est important de noter que l’architecture sépare les fonctionnalités des périphériques et des passerelles. Cette approche permet à l’utilisateur de tirer parti des appareils de passerelle qui sont plus sécurisés : ils sont en mesure de communiquer avec la passerelle cloud à l’aide de protocoles sécurisés, ce qui requiert généralement une charge de traitement supérieure à celle qu’un appareil natif (par exemple un thermostat) peut fournir. Dans la zone des services Azure, nous partons du principe que la passerelle cloud est représentée par le service Azure IoT Hub.

### <a name="device-and-data-sourcesdata-transport"></a>Périphériques et sources de données/transport de données

Cette section explore l’architecture présentée précédemment à travers l’objectif de la modélisation des menaces et donne une vue d’ensemble de la façon dont nous traitons certains des problèmes inhérents. Cet exemple est centré sur les éléments principaux d’un modèle de menace :

* Processus (ceux qui relèvent de notre contrôle et éléments externes)
* Communication (également appelée flux de données)
* Stockage (également appelé magasins de données)

#### <a name="processes"></a>Processus

Dans chacune des catégories présentées dans l’architecture Azure IoT, cet exemple essaye de prévenir un certain nombre de menaces dans les différentes étapes impliquant des données/informations : processus, communication et stockage. Voici une vue d’ensemble des menaces les plus courantes existant pour la catégorie « processus », suivie d’une vue d’ensemble des meilleures préventions contre ces menaces :

**Usurpation (S)**  : Un attaquant peut extraire une clé de chiffrement d’un appareil, au niveau logiciel ou matériel, puis accéder au système avec un autre appareil physique ou virtuel sous l’identité de l’appareil à partir duquel la clé a été extraite. Les télécommandes qui peuvent allumer toutes les TV et qui sont des outils de plaisanterie populaires en sont une bonne illustration.

**Déni de service (D)**  : Un appareil peut être dans l’incapacité de fonctionner ou de communiquer en raison d’interférences avec des fréquences radio ou d’une coupure de câbles. Par exemple, une caméra de surveillance dont l’alimentation ou la connexion réseau ont été intentionnellement interrompues ne communiquera pas de données du tout.

**Falsification (T)**  : Un attaquant peut remplacer partiellement ou totalement le logiciel exécuté sur l’appareil, ce qui permet éventuellement au logiciel remplacé d’endosser l’identité réelle de l’appareil si les éléments de clé ou les fonctionnalités de chiffrement contenant les éléments de clé ont été mis à disposition du programme illicite. Par exemple, une personne malveillante peut tirer parti des éléments de clé extraits pour intercepter et supprimer des données de l’appareil sur le chemin de communication, et les remplacer par des données erronées authentifiées avec les éléments de clé volés.

**Divulgation d’informations (I)**  : Si l’appareil exécute un logiciel manipulé, celui-ci peut créer une fuite de données vers des tiers non autorisés. Par exemple, une personne malveillante peut tirer parti des éléments de clé extraits pour les injecter dans le chemin de communication entre le périphérique et un contrôleur, une passerelle de champ ou une passerelle cloud afin de détourner les informations.

**Élévation de privilège (E)**  : Un appareil qui exécute une fonction spécifique peut être contraint d’en exécuter une autre. Par exemple, une vanne programmée pour s’ouvrir à mi-chemin peut à tort s’ouvrir tout le temps.

| **Composant** | **Menace** | **Atténuation** | **Risque** | **Implémentation** |
| --- | --- | --- | --- | --- |
| Appareil |S |Affectation d’une identité au périphérique et authentification de ce dernier. |Remplacement du périphérique ou d’une partie du périphérique par un autre périphérique. Comment savoir si vous vous adressez à l’appareil approprié ? |Authentification du périphérique à l’aide du protocole TLS (Transport Layer Security) ou IPSec. L’infrastructure doit prendre en charge l’utilisation d’une clé prépartagée (PSK) sur les périphériques qui ne peuvent pas gérer le chiffrement asymétrique complet. Exploitation d’Azure AD, [OAuth](https://www.rfc-editor.org/pdfrfc/rfc6755.txt.pdf) |
|| TRID |Application de mécanismes inviolables à l’appareil, par exemple, en rendant difficile, voire impossible, l’extraction des clés et des autres éléments de chiffrement de cet appareil. |Falsification du périphérique (interférence physique) par un tiers. Comment être sûr de la non-falsification de l’appareil. |La prévention la plus efficace consiste à utiliser la fonctionnalité de module de plateforme sécurisée (TPM) qui permet de stocker des clés dans un ensemble de circuits de processeur spécial à partir duquel les clés ne peuvent pas être lues, et peuvent être utilisées uniquement pour les opérations de chiffrement qui utilisent la clé sans jamais la divulguer. Chiffrement de la mémoire du périphérique. Gestion des clés du périphérique. Signature du code. |
|| E |Disposer du contrôle d’accès du périphérique. Schéma d’autorisation. |Si l’appareil permet d’effectuer des actions individuelles basées sur des commandes provenant d’une source externe, voire de capteurs compromis, il permet aux personnes malveillantes d’effectuer des opérations non accessibles par ailleurs. |Disposer du schéma d’autorisation pour le périphérique. |
| Passerelle de champ |S |Authentification de la passerelle de champ auprès de la passerelle cloud (par ex. basée sur les certificats, clé prépartagée, basée sur les revendications...) |Si quelqu’un peut usurper l’identité de la passerelle de champ, elle peut alors se présenter comme un périphérique. |TLS RSA/PSK, IPSe, [RFC 4279](https://tools.ietf.org/html/rfc4279). Mêmes préoccupations de stockage et d’attestation de clés des périphériques en général ; la meilleure approche consiste à utiliser le module de plateforme sécurisée. Extension 6LowPAN pour IPSec afin de prendre en charge des réseaux de capteurs sans fil (WSN). |
|| TRID |Protection de la passerelle de champ contre la falsification (TPM ?) |Les attaques par usurpation d’identité qui induisent la passerelle cloud en erreur en lui faisant « croire » qu’elle communique avec une passerelle de champ peuvent entraîner la divulgation d’informations et la falsification des données. |Chiffrement de la mémoire, module de plateforme sécurisée, authentification. |
|| E |Mécanisme de contrôle d’accès pour la passerelle de champ | | |

Voici quelques exemples de menaces existant dans cette catégorie :

**Usurpation** : Un attaquant peut extraire une clé de chiffrement d’un appareil, au niveau logiciel ou matériel, puis accéder au système avec un autre appareil physique ou virtuel sous l’identité de l’appareil à partir duquel la clé a été extraite.

**Déni de service** : Un appareil peut être dans l’incapacité de fonctionner ou de communiquer en raison d’interférences avec des fréquences radio ou d’une coupure de câbles. Par exemple, une caméra de surveillance dont l’alimentation ou la connexion réseau ont été intentionnellement interrompues ne communiquera pas de données du tout.

**Falsification** : Un attaquant peut remplacer partiellement ou totalement le logiciel exécuté sur l’appareil, ce qui permet éventuellement au logiciel remplacé d’endosser l’identité réelle de l’appareil si les éléments de clé ou les fonctionnalités de chiffrement contenant les éléments de clé ont été mis à disposition du programme illicite.

**Falsification** : Une caméra de surveillance montrant une image (en spectre visible) d’un couloir vide peut être forcée d’afficher une photo de ce couloir. Un capteur de détection de fumée ou d’incendie peut signaler la présence d’une personne utilisant un briquet. Dans les deux cas, l’appareil peut s’avérer techniquement tout à fait fiable pour le système, alors qu’il rapporte des informations manipulées.

**Falsification** : Un attaquant peut tirer parti des éléments de clé extraits pour intercepter et supprimer des données de l’appareil sur le chemin de communication et les remplacer par des données erronées authentifiées avec les éléments de clé volés.

**Falsification** : Un attaquant peut remplacer partiellement ou complètement le logiciel exécuté sur l’appareil, ce qui permet éventuellement au logiciel remplacé d’endosser l’identité réelle de l’appareil si les éléments de clé ou les fonctionnalités de chiffrement contenant les éléments de clé ont été mis à disposition du programme illicite.

**Divulgation d’informations** : Si l’appareil exécute un logiciel manipulé, celui-ci peut créer une fuite de données vers des tiers non autorisés.

**Divulgation d’informations** : Un attaquant peut tirer parti des éléments de clé extraits pour s’introduire dans le chemin de communication entre l’appareil et un contrôleur, une passerelle locale ou une passerelle cloud, afin de détourner des informations.

**Déni de service** : L’appareil peut être désactivé ou activé dans un mode interdisant les communications (ce qui est intentionnel dans de nombreuses machines industrielles).

**Falsification** : L’appareil peut être reconfiguré pour fonctionner dans un état inconnu du système de contrôle (hors des paramètres d’étalonnage connus) et fournir ainsi des données pouvant être mal interprétées

**Élévation de privilège** : Un appareil qui exécute une fonction spécifique peut être contraint d’en exécuter une autre. Par exemple, une vanne programmée pour s’ouvrir à mi-chemin peut à tort s’ouvrir tout le temps.

**Déni de service** : L’appareil peut être mis dans un état interdisant les communications.

**Falsification** : L’appareil peut être reconfiguré pour fonctionner dans un état inconnu du système de contrôle (hors des paramètres d’étalonnage connus) et fournir ainsi des données pouvant être mal interprétées.

**Usurpation/Falsification/Répudiation** : Si l’appareil n’est pas sécurisé (ce qui est souvent le cas avec les télécommandes grand public), un attaquant peut manipuler de façon anonyme l’état d’un appareil. Les télécommandes qui peuvent allumer toutes les TV et qui sont des outils de plaisanterie populaires en sont une bonne illustration.

#### <a name="communication"></a>Communication

Menaces pesant sur le chemin de communication entre des appareils, des passerelles d’appareil et de champ, et des passerelles d’appareil et cloud. Le tableau suivant présente quelques conseils sur les sockets ouverts de l’appareil/du réseau privé virtuel :

| **Composant** | **Menace** | **Atténuation** | **Risque** | **Implémentation** |
| --- | --- | --- | --- | --- |
| IoT Hub de périphérique |TID |(D)TLS (PSK/RSA) pour chiffrer le trafic |Écoute électronique ou interférence de la communication entre le périphérique et la passerelle. |Sécurité au niveau du protocole. Dans le cas de protocoles personnalisés, vous devez déterminer comment les protéger. Dans la plupart des cas, la communication s’effectue du périphérique vers l’IoT Hub (le périphérique établit la connexion). |
| Appareil vers appareil |TID |(D)TLS (PSK/RSA) pour chiffrer le trafic. |Lecture des données en transit entre les périphériques. Falsification des données. Surcharge du périphérique avec de nouvelles connexions. |Sécurité au niveau du protocole (MQTT/AMQP/HTTP/CoAP). Dans le cas de protocoles personnalisés, vous devez déterminer comment les protéger. La prévention des attaques par déni de service consiste à homologuer des périphériques via une passerelle cloud ou de champ et à les faire agir comme des clients pour le réseau. Le peering peut entraîner une connexion directe entre les unités appairées après avoir été réparti par la passerelle. |
| Périphérique d’entité externe |TID |Couplage fort de l’entité externe au périphérique. |Écoute électronique de la connexion au périphérique. Interférence de la communication avec le périphérique. |Couplage sécurisé de l’entité externe au périphérique NFC/Bluetooth LE. Contrôle du panneau de commandes du périphérique (physique). |
| Passerelle de champ Passerelle cloud |TID |TLS (PSK/RSA) pour chiffrer le trafic. |Écoute électronique ou interférence de la communication entre le périphérique et la passerelle. |Sécurité au niveau du protocole (MQTT/AMQP/HTTP/CoAP). Dans le cas de protocoles personnalisés, vous devez déterminer comment les protéger. |
| Passerelle cloud de périphérique |TID |TLS (PSK/RSA) pour chiffrer le trafic. |Écoute électronique ou interférence de la communication entre le périphérique et la passerelle. |Sécurité au niveau du protocole (MQTT/AMQP/HTTP/CoAP). Dans le cas de protocoles personnalisés, vous devez déterminer comment les protéger. |

Voici quelques exemples de menaces existant dans cette catégorie :

**Déni de service** : Les appareils soumis à des contraintes sont généralement vulnérables à un déni de service s’ils écoutent activement les connexions entrantes ou les datagrammes non sollicités sur un réseau, car un attaquant peut ouvrir plusieurs connexions en parallèle et ne pas les traiter, ou les traiter lentement. Autre cas de figure : l’appareil peut être saturé par un trafic non sollicité. Dans les deux cas, le périphérique peut être effectivement rendu inutilisable sur le réseau.

**Usurpation, divulgation d’informations** : Les appareils soumis à des contraintes et ayant un usage spécifique ont souvent des fonctionnalités de sécurité uniques, par exemple la protection par mot de passe ou par code PIN. Parfois, ils reposent entièrement sur l’approbation du réseau, ce qui signifie qu’ils accordent l’accès aux informations quand un appareil se trouve sur le même réseau, lequel est souvent uniquement protégé par une clé partagée. En d’autres termes, si le secret partagé pour le périphérique ou le réseau est divulgué, il est possible de contrôler le périphérique ou d’observer les données émises par celui-ci.  

**Usurpation d’identité** : une personne malveillante peut intercepter ou remplacer partiellement la diffusion et usurper l’identité du donneur d’ordre (intercepteur)

**Falsification** : une personne malveillante peut intercepter ou partiellement remplacer la diffusion et envoyer de fausses informations 

**Divulgation d’informations :** une personne malveillante peut écouter une diffusion et obtenir des informations sans autorisation. **Déni de service :** une personne malveillante peut brouiller le signal de diffusion et bloquer la distribution d’informations

#### <a name="storage"></a>Stockage

Chaque passerelle de périphérique et de champ dispose d’une certaine forme de stockage (temporaire pour la mise en file d’attente des données, le stockage d’images de système d’exploitation).

| **Composant** | **Menace** | **Atténuation** | **Risque** | **Implémentation** |
| --- | --- | --- | --- | --- |
| Stockage du périphérique |TRID |Chiffrement du stockage, signature des journaux d’activité. |Lecture des données de l’espace de stockage (données d’identification personnelles PII), falsification des données de télémétrie. Falsification des données de contrôle de commande en file d’attente ou mises en cache. La falsification des packages de configuration ou de mise à jour de microprogramme alors que des données sont mises en cache ou en file d’attente peut entraîner la compromission des composants du système d’exploitation et/ou des composants système |Chiffrement, code d’authentification de message (MAC) ou signature numérique. Si possible, contrôle d’accès renforcé via des listes de contrôle d’accès aux ressources (ACL) ou des autorisations. |
| Image de système d’exploitation de périphérique |TRID | |Falsification du système d’exploitation/remplacement des composants du système d’exploitation. |Partition du système d’exploitation en lecture seule, image du système d’exploitation signée, chiffrement. |
| Stockage de passerelle de champ (mise en file d’attente des données) |TRID |Chiffrement du stockage, signature des journaux d’activité. |Lecture des données de l’espace de stockage (données d’identification personnelles PII), falsification des données de télémétrie, falsification des données de contrôle de commande en file d’attente ou mises en cache. La falsification des packages de configuration ou de mise à jour de microprogramme (destinés aux appareils ou à la passerelle de champ) alors que des données sont mises en cache ou en file d’attente peut entraîner la compromission des composants du système d’exploitation et/ou des composants système |BitLocker |
| Image de système d’exploitation de passerelle de champ |TRID | |Falsification du système d’exploitation/remplacement des composants du système d’exploitation. |Partition du système d’exploitation en lecture seule, image du système d’exploitation signée, chiffrement. |

### <a name="device-and-event-processingcloud-gateway-zone"></a>Périphérique et traitement des événements/zone de passerelle cloud

Une passerelle cloud est un système qui permet une communication distante entre des appareils ou des passerelles de champ à partir de plusieurs sites via l’espace réseau public, généralement vers un système d’analyse de données et de contrôle basé sur le cloud, une fédération de tels systèmes. Dans certains cas, une passerelle cloud peut faciliter l’accès immédiat aux appareils à usage spécifique à partir de terminaux comme les tablettes ou les téléphones. Dans le contexte présenté ici, le « cloud » désigne un système de traitement des données dédié qui n’est pas lié au même site que les périphériques connectés ou les passerelles de champ, et dans lequel des mesures opérationnelles empêchent l’accès physique ciblé. Il ne s’agit pas nécessairement de « cloud public ». Une passerelle cloud peut potentiellement être mappée à une superposition de virtualisation de réseau pour isoler la passerelle cloud et tous ses périphériques connectés ou passerelles de champ de tout autre trafic réseau. La passerelle cloud proprement dite n’est pas un système de contrôle d’appareil ni une fonction de traitement ou de stockage des données d’appareil. Ces fonctions servent d’interface avec la passerelle cloud. La zone de passerelle cloud inclut la passerelle cloud proprement dite ainsi que l’ensemble des passerelles de champ et des périphériques directement ou indirectement connectés à elle.

Une passerelle cloud est essentiellement un élément logiciel personnalisé qui s’exécute en tant que service comportant des points de terminaison exposés, auxquels se connectent les périphériques et la passerelle de champ. Par conséquent, il convient de la concevoir dans une optique de sécurité. Suivez le processus [Security Development Lifecycle (SDL)](https://www.microsoft.com/sdl) pour concevoir et créer ce service.

#### <a name="services-zone"></a>Zone des services

Un système de contrôle (ou contrôleur) est une solution logicielle qui interagit avec un périphérique, une passerelle de champ ou une passerelle cloud afin de contrôler un ou plusieurs périphériques et/ou de collecter et/ou de stocker et/ou d’analyser les données de périphérique à des fins de présentation ou à des fins de contrôle ultérieur. Les systèmes de contrôle sont les seules entités s’inscrivant dans le cadre de cette discussion, qui peuvent faciliter immédiatement l’interaction avec des personnes. Les surfaces de contrôle physique intermédiaires sur les appareils sont les exceptions, par exemple un commutateur qui permet de désactiver l’appareil ou de modifier d’autres propriétés, et pour lequel il n’existe pas d’équivalent fonctionnel accessible numériquement.

Les surfaces de contrôle physique intermédiaires sont des surfaces où une logique limite la fonction de la surface de contrôle physique, par exemple une fonction équivalente peut être lancée à distance ou des conflits d’entrée avec des saisies distantes peuvent être évités. Ces surfaces de contrôle intermédiaires sont conceptuellement attachées à un système de contrôle local qui exploite la même fonctionnalité sous-jacente que tout autre système de contrôle à distance auquel l’appareil peut être connecté en parallèle. Pour connaître les principales menaces auxquelles le cloud computing est exposé, consultez [Cloud Security Alliance (CSA)](https://cloudsecurityalliance.org/articles/csa-releases-top-threats-to-cloud-computing-deep-dive/).

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les articles suivants :

* [Outil SDL de modélisation des menaces](https://www.microsoft.com/sdl/adopt/threatmodeling.aspx)
* [Microsoft Azure IoT reference architecture (Architecture de référence Microsoft Azure IoT)](https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/)