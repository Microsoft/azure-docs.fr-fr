---
title: Fichier include
description: Fichier include
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 04/24/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 234407e6973657ba3ad0d78189e7cb1d363c15e2
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95564141"
---
L’Internet des objets (IoT, Internet of Things) confronte les entreprises du monde entier à des défis uniques en termes de sécurité, de confidentialité et de conformité. Contrairement à la technologie informatique traditionnelle où ces problèmes sont axés sur les logiciels et leur mode d’implémentation, l’IoT porte sur les effets de la convergence entre le monde informatique et le monde physique. La protection des solutions IoT implique un approvisionnement sécurisé des appareils, une connexion sécurisée entre ces appareils et le cloud et une protection efficace des données dans le cloud, dans le cadre du traitement et du stockage. Cependant, les appareils avec contraintes de ressources, la répartition géographique des déploiements et le grand nombre d’appareils inclus au sein d’une solution vont à l’encontre de ces fonctionnalités.

Cet article étudie de quelle façon les accélérateurs de solution IoT fournissent une solution Internet des objets sécurisée et privée dans le cloud. Les accélérateurs de solution offrent une solution complète et de bout en bout qui intègre la sécurité à chaque étape. Enraciné dans des décennies d’expérience, le développement de logiciels sécurisés fait partie intégrante des pratiques de Microsoft en matière d’ingénierie logicielle. À cet égard, l’approche Security Development Lifecycle (SDL) représente la méthodologie de développement fondamentale. Elle est associée à une multitude de services de sécurité au niveau de l’infrastructure, comme le processus OSA (Operational Security Assurance), la Microsoft Digital Crimes Unit, Microsoft Security Response Center et le Centre de protection Microsoft contre les programmes malveillants.

Les accélérateurs de solution offrent des fonctionnalités uniques qui font de l’approvisionnement des appareils IoT, de la connexion à ces appareils et du stockage des données qu’ils fournissent une opération simple et transparente et plus que tout, sécurisée. Cet article passe en revue les fonctionnalités de sécurité et les stratégies de déploiement des accélérateurs de solution Azure IoT afin de garantir sécurité, confidentialité et conformité.

## <a name="introduction"></a>Introduction

L’Internet des objets (IoT, Internet of Things) marque le début d’une nouvelle ère. Il offre aux entreprises des opportunités réelles et immédiates pour réduire leurs coûts, augmenter leur chiffre d’affaires et transformer leur activité. Beaucoup d’entreprises, cependant, hésitent à déployer l’IoT dans leurs organisations, préoccupées par la sécurité, la confidentialité et la conformité. La nature unique de l’infrastructure IoT représente un point de préoccupation majeur : cette infrastructure fusionne les mondes physique et informatique, réunissant les risques inhérents à chacun d’eux. La sécurité de l’IoT consiste à garantir l’intégrité du code s’exécutant sur les appareils, à assurer l’authentification des appareils et des utilisateurs, à définir clairement la propriété des appareils (et des données générées par ces appareils) et à résister aux cyber-attaques et aux attaques physiques.

La confidentialité représente également un réel problème. Les entreprises exigent de la transparence vis-à-vis de la collecte de données. Elles veulent savoir ce qui est collecté, pourquoi, qui peut voir les données, qui en contrôle l’accès, etc. Enfin, on observe également des problèmes de sécurité générale liés aux équipements et aux personnes qui les utilisent, ainsi que des difficultés à respecter les normes de conformité du secteur.

Au vu de ces préoccupations en termes de sécurité, de confidentialité, de transparence et de conformité, le choix du fournisseur de solutions IoT approprié demeure un véritable défi. L’assemblage d’éléments individuels issus de logiciels et services IoT de différents fournisseurs entraîne des failles en matière de sécurité, de confidentialité, de transparence et de conformité. Il peut être difficile de les détecter et encore plus de les résoudre. Le choix du fournisseur de logiciels et services IoT approprié repose sur différents aspects. Celui-ci doit afficher une expérience solide dans l’exécution de services, couvrant non seulement différents marchés verticaux, mais également différentes zones géographiques et capables d’effectuer un scale-in de façon sûre et transparente. De même, il est préférable que le fournisseur choisi bénéficie de plusieurs dizaines d’années d’expérience dans le développement de logiciels sécurisés et que ces logiciels soient exécutés sur des milliards d’ordinateurs dans le monde. Il doit également être en mesure d’évaluer les menaces inhérentes à ce nouveau monde de l’Internet des objets.

## <a name="secure-infrastructure-from-the-ground-up"></a>Infrastructure sécurisée de bout en bout

L’infrastructure [Microsoft Cloud](https://azure.microsoft.com) prend en charge plus d’un milliard de clients dans 127 pays et régions. Fort de plusieurs décennies d’expérience de Microsoft en création de logiciels d’entreprise et fourniture de services en ligne, parmi les plus vastes du monde, Microsoft Cloud apporte aux clients des niveaux de sécurité, de confidentialité et de conformité, ainsi que des pratiques d’atténuation de menaces, qu’ils ne pourraient pas mettre en œuvre par eux-mêmes.

Le concept [Security Development Lifecycle (SDL)](https://www.microsoft.com/sdl/) propose un processus de développement obligatoire dans toute l’entreprise, qui intègre les exigences de sécurité dans le cycle de vie des logiciels. Pour garantir que les activités opérationnelles suivent le même niveau de sécurité, SDL utilise des instructions de sécurité rigoureuses, présentées dans le processus Operational Security Assurance (OSA) de Microsoft. Microsoft travaille également avec des cabinets d’audit tiers, qui veillent en permanence à ce que la société respecte ses obligations de conformité. De plus, Microsoft met en œuvre des efforts considérables en matière de sécurité en créant des centres d’excellence, notamment la Microsoft Digital Crimes Unit, Microsoft Security Response Center et le Centre de protection Microsoft contre les programmes malveillants.

## <a name="microsoft-azure---secure-iot-infrastructure-for-your-business"></a>Microsoft Azure : une infrastructure IoT sécurisée pour votre entreprise

Microsoft Azure offre une solution cloud complète, qui associe un ensemble de services cloud intégrés en constante évolution (analyse, Machine Learning, stockage, sécurité, mise en réseau et web) et un engagement envers la protection et la confidentialité de vos données, qui fait référence dans le secteur. La stratégie [Assume Breach](https://azure.microsoft.com/blog/red-teaming-using-cutting-edge-threat-simulation-to-harden-the-microsoft-enterprise-cloud/) de Microsoft repose sur une *équipe dédiée*, composée d’experts en sécurité logicielle, qui simule des attaques pour tester la capacité d’Azure à détecter les menaces émergentes, à les contrer et à rétablir un fonctionnement correct après-coup. L’équipe de [réponse globale aux incidents](https://www.microsoft.com/en-us/TrustCenter/Security/DesignOpSecurity) de Microsoft travaille en permanence pour atténuer les conséquences des attaques et activités malveillantes. Elle applique des procédures établies pour la gestion des incidents, la communication et la récupération et utilise des interfaces détectables et prévisibles avec les partenaires internes et externes.

Les systèmes de Microsoft assurent une détection des intrusions et une prévention continues, une prévention contre les attaques de service et des tests d’intrusion réguliers. Ils offrent également des outils d’investigation qui permettent d’identifier et d’atténuer les menaces. [L’authentification multifacteur](../articles/active-directory/authentication/concept-mfa-howitworks.md) fournit une couche de sécurité supplémentaire pour l’accès au réseau par les utilisateurs finaux. Pour l’application et le fournisseur de l’hôte, Microsoft offre le contrôle d’accès, la surveillance, les logiciels anti-programme malveillant, l’analyse de vulnérabilité, les correctifs et la gestion de la configuration.

Les accélérateurs de solution tirent parti de la sécurité et de la confidentialité intégrées à la plateforme Azure, mais aussi des processus SDL et OSA pour le développement et l’utilisation sécurisés de tous les logiciels Microsoft. Ces procédures fournissent des fonctionnalités de protection de l’infrastructure, de protection du réseau et d’identité et gestion fondamentales pour la sécurité de n’importe quelle solution.

[Azure IoT Hub](../articles/iot-hub/about-iot-hub.md), qui fait partie des [accélérateurs de solution IoT](../articles/iot-fundamentals/iot-introduction.md), offre un service intégralement géré qui garantit une communication bidirectionnelle fiable et sécurisée entre les appareils IoT et les services Azure, par exemple [Azure Machine Learning](../articles/machine-learning/classic/index.yml) et [Azure Stream Analytics](../articles/stream-analytics/stream-analytics-introduction.md). Il utilise pour cela des informations d’identification de sécurité et un contrôle d’accès pour chaque appareil.

Pour présenter clairement les fonctionnalités de sécurité et de confidentialité intégrées aux accélérateurs de solution Azure IoT, cet article fractionne cette suite selon les trois principaux domaines de sécurité.

![Accélérateurs de solution Azure IoT](media/iot-security-ground-up/securing-iot-ground-up-fig3.png)

### <a name="secure-device-provisioning-and-authentication"></a>Approvisionnement et authentification sécurisés des appareils

Les accélérateurs de solution sécurisent les appareils emportés sur le terrain en fournissant une clé d’identité unique pour chacun d’eux. Cette clé peut être utilisée par l’infrastructure IoT pour communiquer avec l’appareil, lorsqu’il est en cours d’utilisation. La configuration du processus est simple et rapide. La clé générée avec un ID d’appareil sélectionné par l’utilisateur constitue la base d’un jeton utilisé pour toutes les communications entre l’appareil et Azure IoT Hub.

Les ID d’appareil peuvent être associés à un appareil au cours de la fabrication (autrement dit, flashés dans un module matériel de confiance) ou utiliser une identité fixe existante en tant que proxy (par exemple, les numéros de série de processeur). Il est complexe de modifier ces informations d’identification dans l’appareil. Il convient donc d’introduire des ID d’appareil logiques, au cas où le matériel sous-jacent subit des modifications et où l’appareil logique reste le même. Dans certains cas, l’association d’une identité d’appareil peut se produire au moment du déploiement de l’appareil (par exemple, un ingénieur de terrain authentifié configure physiquement un nouvel appareil en communiquant avec le serveur principal de solution). Le [registre des identités Azure IoT Hub](../articles/iot-hub/iot-hub-devguide.md) stocke de manière sécurisée les identités des appareils et les clés de sécurité d’une solution. Des identités d’appareil peuvent être ajoutées à une liste verte ou une liste rouge, individuellement ou en groupe, permettant un contrôle total de l’accès à l’appareil.

Les stratégies de contrôle d’accès d’Azure IoT Hub dans le cloud permettent d’activer et de désactiver l’identité de tout appareil, offrant le moyen de dissocier un appareil d’un déploiement IoT si nécessaire. Cette association et cette dissociation d’appareils sont basées sur l’identité de chaque appareil.

D’autres fonctionnalités de sécurité des appareils sont disponibles :

* Les appareils n’acceptent pas les connexions réseau non sollicitées. Ils établissent tous les itinéraires et connexions pour le trafic sortant uniquement. Pour qu’un appareil reçoive une commande de la part d’un serveur principal, il doit établir une connexion pour vérifier toutes les commandes en attente de traitement. Lorsqu’une connexion entre l’appareil et IoT Hub est établie en toute sécurité, l’échange de messages entre le cloud et l’appareil peut se faire en toute transparence.

* Les appareils se connectent ou établissent des itinéraires vers des services bien connus auxquels ils sont couplés, par exemple Azure IoT Hub.

* L’autorisation et l’authentification au niveau du système utilisent des identités par appareil. Les informations d’identification et autorisations d’accès sont ainsi révocables quasi instantanément.

### <a name="secure-connectivity"></a>Connectivité sécurisée

La durabilité de la messagerie est une fonctionnalité importante de toute solution IoT. La nécessité de fournir des commandes et/ou de recevoir des données à partir des appareils de façon durable est d’autant plus importante que les appareils IoT sont connectés par le biais d’Internet ou d’autres réseaux similaires, qui ne sont pas toujours fiables. Azure IoT Hub assure la durabilité de la messagerie entre le cloud et les appareils par le biais d’un système d’accusés de réception en réponse aux messages. Cette durabilité peut être renforcée en mettant en cache les messages dans IoT Hub pendant sept jours maximum pour la télémétrie et deux jours maximum pour les commandes.

L’efficacité est essentielle pour assurer la conservation des ressources et le fonctionnement dans un environnement avec contraintes de ressources. Pris en charge par Azure IoT Hub, le protocole HTTPS (HTTP Secure), version sécurisée standard du protocole populaire http, permet une communication efficace. Les protocoles AMQP (Advanced Message Queuing Protocol) et MQTT (Message Queuing Telemetry Transport), pris en charge par Azure IoT Hub, sont conçus non seulement pour leur efficacité en termes d’utilisation des ressources, mais également pour leur fiabilité en matière de remise des messages. 

L’évolutivité implique la possibilité d’interagir en toute sécurité avec un large éventail d’appareils. Azure IoT Hub assure une connexion sécurisée avec les appareils IP et non-IP. Les appareils IP peuvent se connecter et communiquer directement avec IoT Hub par le biais d’une connexion sécurisée. Les appareils non-IP présentent des contraintes de ressources et se connectent uniquement avec des protocoles de communication de courte distance, tels que Zwave, ZigBee et Bluetooth. Une passerelle de champ est utilisée pour agréger ces appareils. Elle effectue la traduction de protocole pour établir une communication bidirectionnelle sécurisée avec le cloud.

D’autres fonctionnalités de sécurité des connexions sont disponibles :

* L’itinéraire de communication entre les appareils et Azure IoT Hub, ou entre les passerelles et Azure IoT Hub, est sécurisé à l’aide du protocole standard TLS (Transport Layer Security), Azure IoT Hub étant authentifié à l’aide du protocole X.509.

* Afin de protéger les appareils contre les connexions entrantes non sollicitées, Azure IoT Hub n’ouvre aucune connexion vers l’appareil. L’appareil lance toutes les connexions.

* Azure IoT Hub stocke durablement les messages pour les appareils et attend leur connexion. Ces commandes sont stockées pendant deux jours. Les appareils confrontés à des problèmes de connectivité ou d’alimentation peuvent ainsi se connecter de façon sporadique pour recevoir ces commandes. Azure IoT Hub gère une file d’attente spécifique pour chaque appareil.

### <a name="secure-processing-and-storage-in-the-cloud"></a>Traitement et stockage sécurisés dans le cloud

Des communications chiffrées au traitement des données dans le cloud, les accélérateurs de solution veillent à la sécurité des données. Cette solution vous offre la possibilité d’implémenter un chiffrement supplémentaire et de gérer vos clés de sécurité.

En utilisant Azure Active Directory (AAD) pour l’authentification et l’autorisation des utilisateurs, les accélérateurs de solution Azure IoT peuvent fournir un modèle d’autorisation basé sur une stratégie pour les données dans le cloud, garantissant ainsi une gestion facilitée des accès, pouvant être auditée et vérifiée. Ce modèle permet également de révoquer quasi instantanément l’accès aux données dans le cloud et les appareils connectés aux accélérateurs de solution Azure IoT.

Une fois les données dans le cloud, elles peuvent être traitées et stockées dans n’importe quel flux de travail défini par l’utilisateur. L’accès à chaque partie des données est contrôlé avec Azure Active Directory, en fonction du service de stockage utilisé.

Toutes les clés utilisées par l’infrastructure IoT sont stockées dans le cloud de manière sécurisée. Les clés peuvent être substituées si un réapprovisionnement est nécessaire. Les données peuvent être stockées dans [Azure Cosmos DB](../articles/cosmos-db/introduction.md) ou dans [SQL Database](../articles/azure-sql/database/sql-database-paas-overview.md), ce qui permet de définir le niveau de sécurité souhaité. En outre, Azure fournit un moyen de surveiller et d’auditer tous les accès à vos données pour vous informer de toute intrusion ou de tout accès non autorisé.

## <a name="conclusion"></a>Conclusion

L’Internet des objets concerne avant tout vos propres activités, les choses essentielles pour l’entreprise. L’IoT peut apporter une formidable valeur ajoutée à une entreprise, en lui permettant de réduire ses coûts, d’augmenter son chiffre d’affaires et de transformer son activité. La réussite de cette transformation repose en grande partie sur le choix du fournisseur de logiciels et de services IoT approprié. Il s’agit de trouver un fournisseur qui va non seulement déclencher cette transformation en comprenant les besoins et les exigences de l’entreprise, mais également fournir des services et des logiciels intégrant la sécurité, la confidentialité, la transparence et la conformité comme éléments de conception essentiels. Fort d’une expérience incomparable en développement et déploiement de services et logiciels sécurisés, Microsoft continue de s’imposer comme véritable leader dans cette nouvelle ère de l’Internet des objets.

De par leur conception, les accélérateurs de solution intègrent des mesures de sécurité. Ils assurent la surveillance sécurisée des ressources pour améliorer l’efficacité, favorisent les performances opérationnelles pour permettre l’innovation et emploient des analyses de données avancées pour transformer les activités. Grâce à leur approche en couche de la sécurité, à leurs très nombreuses fonctionnalités de sécurité et à leurs modèles de conception, les accélérateurs de solution permettent de déployer une infrastructure de confiance, à même de transformer tout type d’activité.

## <a name="additional-information"></a>Informations supplémentaires

Chaque accélérateur de solution crée des instances de services Azure, telles que :

* [**Azure IoT Hub**](https://azure.microsoft.com/services/iot-hub/) : votre passerelle connectant le cloud aux appareils. Vous pouvez évoluer vers des millions de connexions par hub et traiter des volumes de données de grande envergure en bénéficiant de la prise en charge de l’authentification par appareil, qui vous aide à sécuriser votre solution.

* [**Azure Cosmos DB**](https://azure.microsoft.com/services/cosmos-db/) : service de base de données évolutif et entièrement indexé pour données semi-structurées, qui gère les métadonnées (attributs, configuration, propriétés de sécurité, etc.) des appareils que vous provisionnez. Azure Cosmos DB assure un traitement hautes performances et à débit élevé, ainsi qu’une indexation des données indépendante du schéma. Ce service offre également une interface de requête SQL enrichie.

* [**Azure Stream Analytics**](https://azure.microsoft.com/services/stream-analytics/) : traitement des flux en temps réel dans le cloud, permettant de développer et de déployer rapidement une solution d’analyse à faible coût pour obtenir des informations en temps réel de la part des appareils, capteurs, infrastructures et applications. Les données de ce service entièrement géré peuvent être mises à l’échelle selon n’importe quel volume. Vous continuez à bénéficier d’un débit élevé, d’une faible latence et de la résilience.

* [**Azure App Services**](https://azure.microsoft.com/services/app-service/) : plateforme cloud qui permet de créer des applications web et mobiles puissantes qui se connectent aux données n’importe où, que ce soit localement ou dans le cloud. Créez des applications mobiles attrayantes pour iOS, Android et Windows. Assurez l’intégration avec vos applications SaaS (Software as a Service) et d’entreprise grâce à une connectivité prête à l’emploi à des dizaines d’applications et services basés sur le cloud. Encodez dans votre environnement de développement intégré et votre langage favoris (.NET, Node.js, PHP, Python ou Java) pour créer rapidement des applications web et des API.

* [**Logic Apps**](https://azure.microsoft.com/services/app-service/logic/) : fonctionnalité Azure App Service qui vous aide à intégrer votre solution IoT dans vos systèmes métier existants et à automatiser les processus de flux de travail. Logic Apps permet aux développeurs de concevoir des flux de travail qui démarrent à partir d’un déclencheur et exécutent une série d’étapes — des règles et actions utilisant des connecteurs puissants pour l’intégration à vos processus d’entreprise. Logic Apps offre une connectivité prête à l’emploi vers un vaste écosystème d’applications SaaS, basées sur le cloud et locales.

* [**Stockage Blob Azure**](https://azure.microsoft.com/services/storage/) : stockage cloud fiable et économique pour les données que vos appareils envoient au cloud.