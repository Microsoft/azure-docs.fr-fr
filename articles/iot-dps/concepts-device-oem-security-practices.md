---
title: Pratiques de sécurité pour les fabricants – Service d’approvisionnement des appareils Azure IoT
description: Présente les pratiques de sécurité courantes pour les fabricants OEM qui préparent des appareils en vue de leur inscription dans le service d’approvisionnement des appareils Azure IoT.
author: timlt
ms.author: timlt
ms.date: 3/02/2020
ms.topic: conceptual
ms.service: iot-dps
ms.custom: iot-p0-scenario, iot-devices-deviceOEM
ms.reviewer: nberdy
ms.openlocfilehash: 788738082cbf9995fb2f7282bc3f574903275528
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90527205"
---
# <a name="security-practices-for-azure-iot-device-manufacturers"></a>Pratiques de sécurité pour les fabricants d’appareils Azure IoT
À l’heure où de plus en plus de fabricants mettent en production des appareils IoT, il est utile d’identifier des conseils en lien avec les pratiques courantes. Cet article résume les pratiques de sécurité recommandées à prendre en compte lorsque vous fabriquez des appareils destinés à une utilisation avec le service d’approvisionnement d’appareils Azure IoT.  

> [!div class="checklist"]
> * Sélection des options d’authentification des appareils
> * Installation de certificats sur des appareils IoT
> * Intégration d’un module de plateforme sécurisée dans le processus de fabrication

## <a name="selecting-device-authentication-options"></a>Sélection des options d’authentification d’appareil
L’objectif ultime de toute mesure de sécurité d’appareil IoT est de créer une solution IoT sécurisée. Toutefois, des problèmes tels que les limitations matérielles, le coût et le niveau de l’expertise en matière de sécurité ont un impact sur les options que vous choisissez. De plus, votre approche de la sécurité influe sur la façon dont vos appareils IoT se connectent au cloud. Si les [aspects de la sécurité IoT](https://www.microsoft.com/research/publication/seven-properties-highly-secure-devices/) à prendre en considération sont multiples, une question clé que tout client en vient forcément à se poser a trait au type d’authentification à utiliser. 

Les trois types d’authentification largement utilisés sont les certificats X.509, les modules de plateforme sécurisée et les clés symétriques. Bien qu’il existe d’autres types d’authentification, la plupart des clients qui créent des solutions sur Azure IoT utilisent l’un des trois types précités. La suite de cet article présente les avantages et les inconvénients associés à chaque type d’authentification.

### <a name="x509-certificate"></a>Certificat X.509
Les certificats X.509 sont un type d’identité numérique que vous pouvez utiliser pour l’authentification. La norme de certification X.509 est documentée dans [IETF RFC 5280](https://tools.ietf.org/html/rfc5280). Dans Azure IoT, il existe deux façons d’authentifier des certificats :
- Empreinte. Un algorithme d’empreinte est exécuté sur un certificat pour générer une chaîne hexadécimale. Celle-ci est un identificateur unique du certificat. 
- Authentification par une autorité de certification basée sur une chaîne complète. Une chaîne de certificats est une liste hiérarchique de tous les certificats nécessaires pour authentifier un certificat d’entité finale. Pour authentifier un certificat d’entité finale, il est nécessaire d’authentifier chaque certificat figurant dans la chaîne, qui inclut une autorité de certification racine de confiance. 

Avantages des certificats X.509 :
- X.509 est le type d’authentification le plus sécurisé pris en charge dans Azure IoT.
- X.509 offre un niveau de contrôle élevé pour la gestion des certificats.
- De nombreux fournisseurs sont disponibles pour proposer des solutions d’authentification de type X.509.

Inconvénients des certificats X.509 :
- De nombreux clients peuvent dépendent de fournisseurs externes pour leurs certificats.
- La gestion des certificats peut être coûteuse et augmente le coût total de la solution.
- La gestion du cycle de vie des certificats peut être difficile si la logistique n’est pas bien pensée. 

### <a name="trusted-platform-module-tpm"></a>Module de plateforme sécurisée
Un module de plateforme sécurisée, également appelé [ISO/IEC 11889](https://www.iso.org/standard/66510.html), est une norme régissant la génération et le stockage sécurisés de clés de chiffrement. Un module de plateforme sécurisée est également un périphérique d’E/S virtuel ou physique qui interagit avec des modules qui implémentent la norme. Un module de plateforme sécurisée peut exister en tant que matériel séparé, matériel intégré, module basé sur un microprogramme ou module basé sur un logiciel. 

Il existe deux différences clés entre les module de plateforme sécurisée et les clés symétriques : 
- Des puces modules de plateforme sécurisée peuvent également stocker des certificats X.509.
- Une attestation de module de plateforme sécurisée dans le service d’approvisionnement des appareils utilise la paire de clés de type EK (Endorsement Key) de module de plateforme sécurisée, une forme d’authentification asymétrique. Avec une authentification asymétrique, une clé publique est utilisée pour le chiffrement, et une clé privée pour le déchiffrement. Par contre, les clés symétriques utilisent un authentification symétrique, où la clé privée est utilisée tant pour le chiffrement que le déchiffrement. 

Avantages du module de plateforme sécurisée :
- Des modules de plateforme sécurisée sont inclus en tant que matériel standard sur de nombreux appareils Windows, avec une prise en charge intégrée pour le système d’exploitation. 
- Une attestation de module de plateforme sécurisée est plus facile à sécuriser qu’une attestation de clé symétrique basée sur un jeton de signature d’accès partagé (SAP).
- Vous pouvez facilement faire expirer et renouveler, ou déployer, des informations d’identification d’appareil. Le service d’approvisionnement des appareils déploie automatiquement les informations d’identification de l’IoT Hub chaque fois qu’un appareil de module de plateforme sécurisée doit être réapprovisionné.

Inconvénients du module de plateforme sécurisée : 
- Les modules de plateforme sécurisée sont complexes et parfois difficiles à utiliser. 
- Le développement d’applications avec des modules de plateforme sécurisée est difficile, sauf si vous disposez d’un module de plateforme sécurisée physique ou d’un émulateur de qualité.
- Vous devrez peut-être reconcevoir la carte de votre appareil pour inclure un module de plateforme sécurisée dans le matériel. 
- Si vous déployez la paire de clés de type EK (Endorsement Key) sur un module de plateforme sécurisée, l’identité de celui-ci est détruite et une nouvelle identité créée. Si le processeur physique reste le même, il dispose d’une nouvelle identité dans votre solution IoT.

### <a name="symmetric-key"></a>Clé symétrique
Avec des clés symétriques, la même clé est utilisée pour chiffrer et déchiffrer les messages. Par conséquent, la même clé est connue de l’appareil et du service qui l’authentifie. Azure IoT prend en charge les connexions de clés symétriques basées sur un jeton SAP. Une authentification par clé symétrique implique une responsabilité importante du propriétaire pour sécuriser les clés et atteindre un niveau de sécurité égal à celui d’une authentification X. 509. Si vous utilisez des clés symétriques, il est recommandé de protéger les clés à l’aide d’un module de sécurité matériel (HSM).

Avantages de la clé symétrique :
- L’utilisation de clés symétriques est le mode d’authentification le plus simple et le plus économique.
- Elle simplifie le processus, car il n’y a d’autre rien à générer. 

Inconvénients de la clé symétrique : 
- La sécurisation des clés symétriques nécessite un effort important. La même clé étant partagée entre l’appareil et le cloud, la clé doit être protégée à deux endroits. Par contre, le difficulté avec le module de plateforme sécurisée et les certificats X. 509 est de prouver la possession de la clé publique sans révéler la clé privée.
- Les clés symétriques facilitent l’adoption de pratiques de sécurité médiocres. Une tendance courante avec les clés symétriques consiste à coder en dur les clés non chiffrées sur les appareils. Bien que cette pratique soit commode, elle rend les clés vulnérables. Vous pouvez atténuer le risque en stockant la clé symétrique de façon sécurisée sur l’appareil. Cependant, si votre priorité est en définitive la sécurité plutôt que la commodité, utilisez plutôt des certificats X. 509 ou un module de plateforme sécurisée pour l’authentification. 

### <a name="shared-symmetric-key"></a>Clé symétrique partagée
Il existe une variante de l’authentification par clé symétrique appelée clé symétrique partagée. Cette approche implique l’utilisation de la même clé symétrique sur tous les appareils. Nous vous recommandons d’éviter d’utiliser des clés symétriques partagées sur vos appareils. 

Avantage de la clé symétrique partagée :
- Facile à implémenter et peu coûteuse à produire à grande échelle. 

Inconvénients de la clé symétrique partagée : 
- Très vulnérable aux attaques. L’avantage de la facilité d’implémentation est largement éclipsé par le risque. 
- N’importe qui peut usurper l’identité de vos appareils s’il obtient la clé partagée.
- Si vous vous fiez à une clé symétrique partagée qui devient compromise, vous risquez de perdre le contrôle des appareils. 

### <a name="making-the-right-choice-for-your-devices"></a>Le bon choix pour vos appareils
Pour choisir une méthode d’authentification, veillez à prendre en compte les avantages et les coûts de chaque approche pour votre processus de fabrication unique.  Pour l’authentification des appareils, il existe généralement une relation inverse entre le degré de sécurisation d’une approche donnée et son degré de commodité.  

## <a name="installing-certificates-on-iot-devices"></a>Installation de certificats sur des appareils IoT
Si vous utilisez des certificats X. 509 pour authentifier vos appareils IoT, cette section fournit des conseils sur la façon d’intégrer des certificats dans votre processus de fabrication. Vous devez prendre plusieurs décisions.  Il s’agit notamment de décisions concernant les variables de certificat courantes, le moment auquel générer les certificats et celui auquel les installer. 

Si vous avez l’habitude d’utiliser des mots de passe, vous vous demandez peut-être pourquoi vous ne pouvez pas utiliser le même certificat sur tous vos appareils, de la même façon que vous pourriez utiliser le même mot de passe. Tout d’abord, l’utilisation du même mot de passe partout est dangereuse. Cette pratique a déjà exposé des entreprises à des attaques majeures par déni de service, notamment celle qui a fait tomber le DNS sur la côte Est des États-Unis il y a quelques années. N’utilisez jamais le même mot de passe partout, même avec des comptes personnels. Deuxièmement, un certificat n’est pas un mot de passe, mais une identité unique. Un mot de passe est semblable à un code secret que tout le monde peut utiliser pour ouvrir une porte de bâtiment sécurisé.  C’est quelque chose que vous connaissez et que vous pourriez donner à n’importe qui lui permettre d’entrer.  Un certificat est comme un permis de conduire sur lequel figure votre photo et d’autres détails personnels, que vous pouvez présenter à un gardien pour accéder à un bâtiment sécurisé. Il est lié à qui vous êtes.  Pour autant que le gardien vérifie soigneusement les permis de conduire, vous seul pouvez utiliser votre permis (identité) pour accéder. 

### <a name="variables-involved-in-certificate-decisions"></a>Variables impliquées dans les décisions en matière de certificat
Prenez en considération les variables suivantes et l’incidence de chacune d’elle sur le processus de fabrication global. 

#### <a name="where-the-certificate-root-of-trust-comes-from"></a>Origine de la racine de confiance du certificat
La gestion d’une infrastructure à clé publique (PKI) peut être coûteuse et complexe.  C’est particulièrement vrai si votre entreprise n’a aucune expérience de gestion d’infrastructure à clé publique. Vos options sont les suivantes :
- Utiliser une infrastructure à clé publique tierce. Vous pouvez acheter un certificat de signature intermédiaire auprès d’un fournisseur de certificats tiers. Ou vous pouvez utiliser une autorité de certification privée. 
- Utiliser une infrastructure à clé publique gérée par vous. Vous pouvez gérer votre propre système d’infrastructure à clé publique et générer vos propres certificats.
- Utiliser le service de sécurité [Azure Sphere](https://azure.microsoft.com/services/azure-sphere/). Cette option s’applique uniquement aux appareils Azure Sphere. 

#### <a name="where-certificates-are-stored"></a>Emplacement de stockage des certificats
Quelques facteurs déterminent le choix de l’emplacement de stockage des certificats. Ces facteurs sont le type d’appareil, les marges bénéficiaires attendues (avez-vous les moyens d’acquérir un stockage sécurisé), les capacités de l’appareil et la technologie de sécurité disponible sur l’appareil, que vous pourriez peut-être utiliser. Considérez les options suivantes :
- Dans un module de sécurité matériel. L’utilisation d’un module de sécurité matériel est fortement recommandée. Vérifiez si un module de sécurité matériel est déjà installé sur le panneau de contrôle de votre appareil. Si vous savez que vous n’avez pas de module de sécurité matériel, contactez le fabricant de votre matériel pour identifier un module répondant à vos besoins.
- Dans un emplacement sécurisé sur disque, tel qu’un environnement d’exécution de confiance.
- Dans le système de fichiers local ou dans un magasin de certificats. Par exemple, le magasin de certificats Windows. 

#### <a name="connectivity-at-the-factory"></a>Connectivité au niveau de la fabrique
La connectivité au niveau de la fabrique détermine comment et quand vous obtiendrez les certificats à installer sur les appareils. Les options de connectivité sont les suivantes :
- Connectivité. Une connectivité optimale simplifie le processus de génération des certificats localement. 
- Aucune connectivité Dans ce cas, vous utilisez un certificat signé d’une autorité de certification pour générer les certificats d’appareils localement et hors connexion. 
- Aucune connectivité Dans ce cas, vous pouvez obtenir des certificats générés à l’avance. Ou vous pouvez utiliser une infrastructure à clé publique hors connexion pour générer des certificats localement.

#### <a name="audit-requirement"></a>Exigence d’audit
Selon le type d’appareil que vous produisez, il se peut que vous ayez l’obligation réglementaire de créer une piste d’audit pour vérifier la manière dont les identités sont installées sur vos appareils. L’audit augmente sensiblement le coût de production. Donc, dans la plupart des cas, ne le faites que si c’est indispensable. Si vous n’êtes pas certain qu’un audit est requis, contactez le service juridique de votre entreprise. Les options d’audit sont les suivantes : 
- Secteur non sensible. Aucun audit n’est requis.
- Secteur sensible. Les certificats doivent être installés dans une salle sécurisée conformément aux exigences de certification de conformité. Si vous avez besoin d’une salle sécurisée pour installer des certificats, vous savez probablement déjà comment les certificats sont installés sur vos appareils. Et vous disposez probablement déjà d’un système d’audit. 

#### <a name="length-of-certificate-validity"></a>Durée de validité du certificat
À l’instar d’un permis de conduite, un certificat a une date d’expiration définie lors de sa création. Voici les options pour la durée de validité du certificat :
- Renouvellement non requis.  Cette approche utilise une période de renouvellement longue. Vous n’aurez donc jamais à renouveler le certificat pendant la durée de vie de l’appareil. Bien qu’une telle approche soit pratique, elle est également risquée.  Vous pouvez réduire le risque en utilisant un stockage sécurisé tel un module de sécurité matériel sur vos appareils. Toutefois, la pratique recommandée consiste à éviter d’utiliser des certificats à durée de vie longue.
- Renouvellement requis.  Vous devrez renouveler le certificat pendant la durée de vie de l’appareil. La durée de validité du certificat dépend du contexte et vous aurez besoin d’une stratégie de renouvellement.  La stratégie doit inclure l’origine des certificats et le type de fonctionnalité OTA que vos appareils doivent utiliser dans le cadre du processus de renouvellement. 

### <a name="when-to-generate-certificates"></a>Quand générer les certificats
Les capacités de connectivité Internet au niveau de votre fabrique ont une incidence sur le processus de génération de certificats. Vous avez plusieurs options concernant le moment auquel générer des certificats : 

- Certificats préchargés  Certains fournisseurs de module de sécurité matériel offrent un service Premium dans le cadre duquel le fournisseur installe les certificats pour le client. Tout d’abord, le clients accorde au fournisseur du module de sécurité matériel l’accès à un certificat de signature. Ensuite, le fournisseur installe les certificats signés par ce certificat de signature sur chaque module de sécurité matériel acheté par le client. Il suffit à celui-ci d’installer le module de sécurité matériel sur l’appareil. Bien que ce service augmente le coût, il permet de simplifier le processus de fabrication.  Et il résout la question de savoir quand installer les certificats.
- Certificats générés par l’appareil.  Si vos appareils génèrent des certificats en interne, vous devez extraire le certificat X. 509 public de l’appareil pour l’inscrire auprès du service d’approvisionnement des appareils. 
- Fabrique connectée.  Si votre fabrique dispose d’une connectivité, vous pouvez générer des certificats d’appareil chaque fois que vous en avez besoin.
- Fabrique hors connexion avec votre propre infrastructure à clé publique. Si votre fabrique n’a pas de connectivité et que vous utilisez votre propre infrastructure à clé publique avec prise en charge hors connexion, vous pouvez générer les certificats lorsque vous en avez besoin.
- Fabrique hors connexion avec infrastructure à clé publique tierce. Si votre fabrique n’a pas de connectivité et que vous utilisez une infrastructure à clé publique tierce, vous devez générer les certificats à l’avance. Et vous devez les générer à partir d’un emplacement disposant d’une connectivité. 

### <a name="when-to-install-certificates"></a>Quand installer les certificats
Une fois que vous avez généré les certificats pour vos appareils IoT, vous pouvez les installer sur ceux-ci. 

Si vous utilisez des certificats préchargés avec un module de sécurité matériel, le processus est simplifié. Une fois le module de sécurité matériel installé sur l’appareil, le code de l’appareil peut y accéder. Ensuite, vous appelez les API de module de sécurité matériel pour accéder au certificat stocké dans le module de sécurité matériel. Cette approche est la plus pratique pour votre processus de fabrication. 

Si vous n’utilisez pas de certificat préchargé, vous devez installer le certificat dans le cadre de votre processus de production. L’approche la plus simple consiste à installer le certificat dans le module de sécurité matériel au moment où vous flashez l’image initiale du microprogramme. Votre processus doit ajouter une étape pour installer l’image sur chaque appareil. À l’issue de cette étape, vous pouvez exécuter les contrôles de qualité finaux et toutes les autres étapes éventuelles avant de conditionner et d’expédier l’appareil. 

Il existe des outils logiciels pour vous permettre d’exécuter le processus d’installation et le contrôle de qualité final en une seule étape. Vous pouvez modifier ces outils pour générer un certificat ou extraire un certificat d’un magasin de certificats pré-générés. Le logiciel peut alors installer le certificat là où vous devez l’installer. Les outils logiciels de ce type vous permettent d’effectuer une fabrication de qualité à grande échelle. 

Une fois les certificats installés sur vos appareils, l’étape suivante consiste à apprendre comment inscrire les appareils auprès du [service d’approvisionnement des appareils](about-iot-dps.md). 

## <a name="integrating-a-tpm-into-the-manufacturing-process"></a>Intégration d’un module de plateforme sécurisée dans le processus de fabrication
Si vous utilisez un module de plateforme sécurisée pour authentifier vos appareils IoT, cette section contient des conseils. Ceux-ci couvrent les appareils avec module de plateforme sécurisée 2.0 largement utilisés qui prennent en charge les clés HMAC (Hash-based Message Authentication Code). La spécification de module de plateforme sécurisée pour les puces modules de plateforme sécurisée est une norme ISO gérée par le Trusted Computing Group. Pour plus d’informations sur module de plateforme sécurisée, consultez les spécifications des normes de [module de plateforme sécurisée 2.0](https://trustedcomputinggroup.org/tpm-library-specification/) et [ISO/IEC 11889](https://www.iso.org/standard/66510.html). 

### <a name="taking-ownership-of-the-tpm"></a>Prise de possession du module de plateforme sécurisée
Une étape critique dans la fabrication d’un appareil avec une puce module de plateforme sécurisée consiste à prendre possession du module de plateforme sécurisée. Cette étape est requise pour pouvoir fournir une clé au propriétaire de l’appareil. La première étape consiste à extraire la paire de clés de type EK (Endorsement Key) de l’appareil. L’étape suivante consiste à revendiquer la propriété. Le mode opératoire dépend du module de plateforme sécurisée et du système d’exploitation que vous utilisez. Si nécessaire, contactez le fabricant du module de plateforme sécurisée ou le développeur du système d’exploitation de l’appareil pour savoir comment revendiquer la propriété. 

Dans votre processus de fabrication, vous pouvez extraire la paire de clés de type EK (Endorsement Key) et revendiquer la propriété à des moments différents, ce qui augmente la flexibilité. De nombreux fabricants profitent de cette flexibilité pour ajouter un module de sécurité matériel afin d’améliorer la sécurité de leurs appareils. Cette section fournit des conseils sur le moment auquel extraire la paire de clés de type EK (Endorsement Key) et le moment auquel réclamer la propriété du module de plateforme sécurisée, ainsi que des considérations relatives à l’intégration de ces étapes dans une chronologie de fabrication. 

> [!IMPORTANT]
> Les conseils suivants supposent que vous utilisez un module de plateforme sécurisée séparé, de type microprogramme ou intégré. Là où c’est opportun, les conseils ajoutent des remarques sur l’utilisation d’un module de plateforme sécurisée non séparé ou logiciel. Si vous utilisez un module de plateforme sécurisée logiciel, il se peut que des étapes supplémentaires soient nécessaires qui ne sont pas décrites dans ces conseils. Les modules de plateforme sécurisée logiciels ont divers types d’implémentation qui dépassent le cadre de cet article.  En général, il est possible d’intégrer un module de plateforme sécurisée logiciel dans la chronologie de fabrication générale suivante. Toutefois, si un module de plateforme sécurisée émulé est approprié pour le prototypage et le test, il ne peut pas offrir le même niveau de sécurité qu’un module de plateforme sécurisée séparé, de type microprogramme ou intégré. En règle générale, évitez d’utiliser un module de plateforme sécurisée logiciel en production.

### <a name="general-manufacturing-timeline"></a>Chronologie de fabrication générale
La chronologie suivante montre le processus de production d’un module de plateforme sécurisée. Chaque processus de fabrication est unique et cette chronologie présente les schémas les plus courants. La chronologie fournit des conseils sur la façon de prendre certaines mesures en lien avec les clés. 

#### <a name="step-1-tpm-is-manufactured"></a>Étape 1 : Le module de plateforme sécurisée est fabriqué
- Si vous achetez des modules de plateforme sécurisée auprès d’un fabricant pour les utiliser dans vos appareils, vérifiez s’il extraira les paires de clés de type EK (Endorsement Key) publiques pour vous. Il est utile que le fabricant fournisse la liste de ces clé avec les appareils livrés. 
    > [!NOTE]
    > Vous pouvez accorder au fabricant du module de plateforme sécurisée un accès en écriture à votre liste d’inscriptions en utilisant des stratégies d’accès partagé dans votre service d’approvisionnement.  Cette approche lui permet d’ajouter les modules de plateforme sécurisée à votre liste d’inscriptions.  Mais cela doit se faire tôt dans le processus de fabrication et nécessite que vous fassiez confiance au fabricant du module de plateforme sécurisée. C’est à vos propres risques et périls. 

- Si vous fabriquez des modules de plateforme sécurisée que vous vendez à des fabricants d’appareils, envisagez de fournir à vos clients une liste de paires de clés de type EK (Endorsement Key) publiques avec leurs modules de plateforme sécurisée physiques.  La fourniture aux clients des paires de clés de type EK (Endorsement Key) publiques leur permet d’économiser une étape dans leur processus. 
- Si vous fabriquez des modules de plateforme sécurisée à utiliser avec vos propres appareils, identifiez le stade de votre processus le plus approprié pour extraire la paire de clés de type EK (Endorsement Key) publique. Vous pouvez l’extraire à n’importe quel stade de la chronologie. 

#### <a name="step-2-tpm-is-installed-into-a-device"></a>Étape 2 : Le module de plateforme sécurisée est installé dans un appareil
À ce stade du processus de production, vous devez connaître l’instance de service d’approvisionnement des appareils avec laquelle l’appareil sera utilisé. Par conséquent, vous pouvez ajouter des appareils à la liste d’inscriptions pour l’approvisionnement automatique. Pour plus d’informations sur l’approvisionnement automatique des appareils, consultez la [documentation sur le service d’approvisionnement des appareils](about-iot-dps.md).
- Si vous n’avez pas extrait la paire de clés de type EK (Endorsement Key) publique, c’est le moment idéal pour le faire. 
- En fonction du processus d’installation du module de plateforme sécurisée, cette étape est également appropriée pour prendre possession du module de plateforme sécurisée. 

#### <a name="step-3-device-has-firmware-and-software-installed"></a>Étape 3 : Le microprogramme et le logiciel sont installés sur l’appareil
À ce stade du processus, installez le client du service d’approvisionnement des appareils avec l’étendue de l’ID et l’URL globale pour l’approvisionnement.
- C’est maintenant votre dernière chance d’extraire la paire de clés de type EK (Endorsement Key) publique. Si un tiers installe le logiciel sur votre appareil, il est judicieux d’extraire la paire de clés de type EK (Endorsement Key) publique au préalable.
- Ce stade du processus de fabrication est idéal pour prendre possession du module de plateforme sécurisée.  
    > [!NOTE]
    > Si vous utilisez un module de plateforme sécurisée logiciel, vous pouvez l’installer maintenant.  Extrayez la paire de clés de type EK (Endorsement Key) publique au même moment.

#### <a name="step-4-device-is-packaged-and-sent-to-the-warehouse"></a>Étape 4 : L’appareil est conditionné et envoyé à l’entrepôt
Un appareil peut séjourner dans un entrepôt pendant de 6 à 12 mois avant son déploiement. 

#### <a name="step-5-device-is-installed-into-the-location"></a>Étape 5 : L’appareil est installé à son emplacement
Une fois que l’appareil arrive à son emplacement final, il passe par le processus d’approvisionnement automatique du service d’approvisionnement des appareils.

Pour plus d’informations, consultez [Approvisionnement](about-iot-dps.md#provisioning-process) et [Attestation de module de plateforme sécurisée](concepts-tpm-attestation.md). 

## <a name="resources"></a>Ressources

En plus des pratiques de sécurité recommandées dans cet article, Azure IoT fournit des ressources pour vous aider à sélectionner du matériel sécurisé et à créer des déploiements IoT sécurisés : 
- [Recommandations de sécurité](../iot-fundamentals/security-recommendations.md) d’Azure pour guider le processus de déploiement. 
- L’[Azure Security Center](https://azure.microsoft.com/services/security-center/) offre un service qui vous aide à créer des déploiements IoT sécurisés. 
- Pour obtenir de l’aide en lien avec l’évaluation de votre environnement matériel, consultez le livre blanc sur l’[évaluation de votre de sécurité IoT](https://download.microsoft.com/download/D/3/9/D3948E3C-D5DC-474E-B22F-81BA8ED7A446/Evaluating_Your_IOT_Security_whitepaper_EN_US.pdf). 
- Pour obtenir de l’aide sur la sélection de matériel sécurisé, consultez le document concernant [le matériel sécurisé adapté pour votre déploiement IoT](https://download.microsoft.com/download/C/0/5/C05276D6-E602-4BB1-98A4-C29C88E57566/The_right_secure_hardware_for_your_IoT_deployment_EN_US.pdf). 