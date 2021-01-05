---
title: Concept de la sécurité d’Azure IoT Hub X.509 | Microsoft Docs
description: Présentation du concept des certificats d'autorité de certification X.509 en matière d'authentification et de fabrication de l'appareil IoT.
author: eustacea
manager: arjmands
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/18/2017
ms.author: eustacea
ms.openlocfilehash: 877200cbafbe68fa6161025572abfddad651e172
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490718"
---
# <a name="conceptual-understanding-of-x509-ca-certificates-in-the-iot-industry"></a>Informations conceptuelles sur les certificats de l’autorité de certification X.509 dans l’industrie IoT

Cet article décrit la valeur de l’utilisation des certificats de l’autorité de certification X.509 en matière de fabrication de l’appareil IoT et d’authentification à IoT Hub. Il inclut des informations sur la configuration de la chaîne logistique et met en avant les avantages proposés.

Cet article aborde les points suivants :

* Ce que sont les certificats de l’autorité de certification X.509 et comment les obtenir

* Comment inscrire votre certificat de l’autorité de certification X.509 dans IoT Hub

* Comment configurer une chaîne logistique de fabrication pour l’authentification basée sur l’autorité de certification X.509

* Comment les appareils approuvés par l’autorité de certification X.509 se connectent à IoT Hub

[!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

## <a name="overview"></a>Vue d’ensemble

L’authentification par l’autorité de certification X.509 est une approche permettant d’authentifier les appareils dans IoT Hub à l’aide d’une méthode qui simplifie considérablement la création d’une identité d’appareil et la gestion du cycle de vie dans la chaîne logistique.

Un attribut distinctif de l’authentification de l’autorité de certification X.509 est la relation un-à-plusieurs qu’entretient un certificat de l’autorité de certification avec ses appareils en aval. Cette relation permet d’inscrire autant d’appareils que vous le voulez dans IoT Hub en inscrivant une seule fois un certificat de l’autorité de certification X.509. Dans le cas contraire, un certificat unique doit être préalablement inscrit pour chaque appareil avant sa connexion. Cette relation un-à-plusieurs simplifie également les opérations de gestion du cycle de vie des certificats d’appareil.

Un autre attribut important de l’authentification de l’autorité de certification X.509 est la simplification de la gestion de la chaîne logistique. L’authentification sécurisée des appareils nécessite que chaque appareil soit associé à un secret unique servant de clé pour l’approbation. Dans l’authentification basée sur les certificats, ce secret est une clé privée. Les flux classiques de fabrication d’appareils impliquent plusieurs étapes et opérateurs. La gestion sécurisée des clés privées des appareils entre plusieurs opérateurs et le maintien de la conformité sont difficiles et coûteux. Le recours aux autorités de certification résout ce problème : chaque opérateur s’inscrit dans une chaîne de confiance cryptographique au lieu de se voir remettre des clés privées d’appareils. Chaque opérateur approuve à son tour les appareils à son étape du processus de fabrication. Résultat : une chaîne logistique optimale avec responsabilité intégrée grâce à l’utilisation d’une chaîne de confiance cryptographique. Il est important de noter que ce processus est le plus sécurisé lorsque les appareils protègent leurs clés privées uniques. À cette fin, nous conseillons vivement l’utilisation de modèles de sécurité matériels capables de générer en interne des clés privées qui ne sortiront jamais du système.

Cet article offre une vue complète de l’utilisation de l’authentification de l’autorité de certification X.509, de la configuration de la chaîne logistique à la connexion des appareils, tout en exploitant un exemple réel pour rendre la présentation plus claire.

Vous pouvez également utiliser des groupes d’inscription avec le Service IoT Hub Device Provisioning (DPS) Azure pour gérer l’approvisionnement des appareils aux hubs. Pour plus d’informations sur l’utilisation de DPS pour approvisionner des appareils de certificat X.509, consultez [Didacticiel : Provisionner plusieurs appareils X.509 à l’aide de groupes d’inscriptions](../iot-dps/tutorial-custom-hsm-enrollment-group-x509.md).

## <a name="introduction"></a>Introduction

Le certificat de l’autorité de certification X.509 est un certificat numérique dont le titulaire peut signer d’autres certificats. Ce certificat numérique porte le nom X.509, car il est conforme à une norme de mise en forme des certificats prévue par la norme RFC 5280 de l’IETF. Il s’agit d’une autorité de certification, car son détenteur peut signer d’autres certificats.

L’utilisation de l’autorité de certification X.509 est plus simple à comprendre en s’appuyant sur un exemple concret. Imaginez l’entreprise X, un créateur de widgets Smart-X conçus pour une installation professionnelle. L’entreprise X sous-traite la fabrication et l’installation. Elle fait appel à l’usine Y pour fabriquer les widgets Smart-X et au technicien Z pour les installer. L’entreprise X souhaite que les widgets Smart-X soient directement envoyés de l’usine Y au technicien Z pour l’installation, et qu’ils soient directement reliés à l’instance IoT Hub de l’entreprise X après installation, sans autre intervention de l’entreprise X. Pour ce faire, l’entreprise X doit effectuer quelques opérations de configuration uniques pour préparer le widget Smart-X à la connexion automatique. Suivant le scénario complet, le reste de cet article est structuré comme suit :

* Acquérir le certificat de l’autorité de certification X.509

* Inscrire le certificat de l’autorité de certification X.509 dans IoT Hub

* Signer les appareils dans une chaîne de certificats de confiance

* Connexion de l’appareil

## <a name="acquire-the-x509-ca-certificate"></a>Acquérir le certificat de l’autorité de certification X.509

L’entreprise X a la possibilité d’acheter un certificat de l’autorité de certification X.509 auprès d’une autorité de certification racine publique ou d’en créer un via un processus auto-signé. Selon le scénario d’application, l’une des options peut être meilleure que l’autre. Quelle que soit l’option choisie, le processus implique deux étapes fondamentales : la génération d’une paire de clés publique/privée et la signature de la clé publique dans un certificat.

![Flux pour la génération d’un certificat X509CA](./media/iot-hub-x509ca-concept/csr-flow.png)

La réalisation de ces étapes diffère en fonction des prestataires de services.

### <a name="purchasing-an-x509-ca-certificate"></a>Achat d’un certificat de l’autorité de certification X.509

En achetant un certificat d’autorité de certification, une autorité de certification racine connue agit comme un tiers de confiance pour garantir la légitimité de vos appareils IoT lors de leur connexion. L'entreprise X optera pour cette option si elle souhaite que ses widgets Smart-X interagissent avec des produits ou services tiers après la connexion initiale à IoT Hub.

Pour acheter un certificat de l’autorité de certification X.509, l’entreprise X choisira un prestataire de services de certificats racine. Pour trouver de bons prospects, il suffit de rechercher « Autorité de certification racine » sur Internet. L’autorité de certification racine aidera l’entreprise X à créer la paire de clés publique/privée et à générer une demande de signature de certificat pour ses services. Une demande de signature de certificat correspond au processus formel de demande de certificat auprès d’une autorité de certification. Le résultat de cet achat est un certificat à utiliser comme certificat d’autorité. Étant donné l’omniprésence des certificats X.509, le certificat aura certainement été correctement mis en forme conformément à la norme RFC 5280 de l’IETF.

### <a name="creating-a-self-signed-x509-ca-certificate"></a>Création d’un certificat de l’autorité de certification X.509 auto-signé

Le processus de création d'un certificat d'autorité de certification X.509 auto-signé est similaire au processus d'achat, hormis qu'un signataire tiers, comme l'autorité de certification racine, est impliqué. Dans notre exemple, l’entreprise X signe son certificat d’autorité à la place d’une autorité de certification racine. L’entreprise X peut choisir cette option à des fins de test, jusqu'à ce qu’elle soit prête à acheter un certificat d’autorité. L'entreprise X peut également utiliser un certificat d'autorité de certification X.509 auto-signé en production, si le widget Smart-X n'est pas destiné à se connecter à des services tiers en dehors d'IoT Hub.

## <a name="register-the-x509-certificate-to-iot-hub"></a>Inscrire le certificat X.509 dans IoT Hub

L’entreprise X doit inscrire l’autorité de certification X.509 dans IoT Hub, où elle servira à authentifier les widgets Smart-X lors de leur connexion. Il s’agit d’un processus unique qui permet d’authentifier et de gérer autant de widgets Smart-X que vous le souhaitez. Il s’agit d’un processus unique en raison d’une relation un-à-plusieurs entre le certificat de l’autorité de certification et les certificats de l’appareil signés par le certificat de l’autorité de certification ou un certificat intermédiaire. Cette relation constitue l’un des principaux avantages de l’utilisation de la méthode d’authentification de l’autorité de certification X.509. L’autre solution consiste à télécharger les empreintes de chaque certificat pour chaque widget Smart-X, ce qui augmente les coûts d’exploitation.

L’inscription du certificat de l’autorité de certification X.509 est un processus en deux étapes, qui comprend le chargement du certificat et une preuve de possession du certificat.

![Inscription d’un certificat X509CA](./media/iot-hub-x509ca-concept/pop-flow.png)

### <a name="x509-ca-certificate-upload"></a>Chargement du certificat de l’autorité de certification X.509

Le processus de chargement du certificat de l’autorité de certification X.509 consiste simplement à charger le certificat de l’autorité de certification dans IoT Hub. IoT Hub attend le certificat sous forme de fichier. L’entreprise X transfère simplement le fichier de certificat. Le fichier de certificat NE DOIT en aucun cas contenir des clés privées. Les meilleures pratiques des normes régissant l’infrastructure de clé publique imposent que les clés privées de l’entreprise X soient exclusivement connues au sein de l’entreprise X.

### <a name="proof-of-possession-of-the-certificate"></a>Preuve de possession du certificat

Le certificat de l’autorité de certification X.509, tout comme n’importe quel certificat numérique, constitue des informations publiques qui sont susceptibles d’être volées. Par conséquent, une personne malveillante peut intercepter un certificat et essayer de le charger comme s’il s’agissait du sien. Dans notre exemple, IoT Hub veut s’assurer que le certificat de l’autorité de certification que charge l’entreprise X appartient réellement à l’entreprise X. Pour ce faire, IoT Hub demande à l'entreprise X de prouver qu'elle possède effectivement le certificat via un [flux de preuve de possession](https://tools.ietf.org/html/rfc5280#section-3.1). Le flux de preuve de possession implique qu’IoT Hub génère un numéro aléatoire que l’entreprise X doit signer à l’aide de sa clé privée. Si l’entreprise X a bien suivi les meilleures pratiques de l’infrastructure de clé publique et a correctement protégé sa clé privée, alors elle est la seule à pouvoir répondre correctement au défi de la preuve de possession. IoT Hub poursuit l’inscription du certificat de l’autorité de certification X.509 en cas de réponse correcte au défi de la preuve de possession.

En cas de réponse correcte d’IoT Hub au défi de la preuve de possession, l’inscription de l’autorité de certification X.509 est terminée.

## <a name="sign-devices-into-a-certificate-chain-of-trust"></a>Signer les appareils dans une chaîne de certificats de confiance

IoT nécessite que chaque appareil possède une identité unique. Ces identités correspondent à des certificats pour les schémas d’authentification par certificat. Dans notre exemple, cela signifie que chaque widget Smart-X doit posséder un certificat d’appareil unique. Comment l’entreprise X doit-elle configurer sa chaîne logistique ?

L’une des façons de procéder est de prégénérer des certificats pour les widgets Smart-X et de confier les clés privées d’appareil unique correspondantes aux partenaires de la chaîne logistique. Pour l’entreprise X, cela signifie faire confiance à l’usine Y et au technicien Z. Bien que cette méthode soit valide, elle implique des défis qui doivent être relever pour assurer la confiance :

1. Devoir partager les clés privées des appareils avec les partenaires de la chaîne logistique rend la confiance dans la chaîne logistique coûteuse, outre le fait que cela ne respecte pas les meilleures pratiques de l’infrastructure de clé publique qui indiquent de ne jamais partager les clés privées. Cela signifie que des systèmes cruciaux, par exemple des salles sécurisées pour stocker les clés privées, et des processus tels que des audits de sécurité doivent être mis en place. Ceux-ci ajoutent des coûts à la chaîne logistique.

2. La comptabilité sécurisée des appareils dans la chaîne logistique et leur gestion ultérieure dans le déploiement deviennent une tâche individualisée pour chaque paire clé-appareil, de la génération d’un certificat d’appareil unique (clé privée) au retrait de l’appareil. Cela empêche la gestion de groupe des appareils, à moins que le concept de groupes ne soit explicitement intégré au processus, d’une manière ou d’une autre. Par conséquent, la comptabilité sécurisée et la gestion du cycle de vie des appareils se transforment en une charge d’exploitation lourde. Dans notre exemple, l’entreprise X doit supporter cette charge.

L’authentification par certificat d’autorité de certification X.509 propose des solutions à tous les défis cités, grâce à l’utilisation de chaînes d’approbation. Une chaîne d’approbation est créée lorsqu’une autorité de certification signe une autorité de certification intermédiaire, qui à son tour signe une autre autorité de certification intermédiaire, et ainsi de suite jusqu’à ce qu’une autorité de certification intermédiaire finale signe un appareil. Dans notre exemple, l’entreprise X signe l’usine Y, qui à son tour signe le technicien Z, qui signe le widget Smart-X.

![Hiérarchie de la chaîne d’approbation](./media/iot-hub-x509ca-concept/cert-chain-hierarchy.png)

La cascade de certificats ci-dessus correspond au transfert logique de l’autorité. De nombreuses chaînes logistiques suivent ce transfert logique, selon lequel chaque autorité de certification intermédiaire est signée dans la chaîne tout en recevant tous les certificats d’autorité de certification en amont, et la dernière autorité de certification intermédiaire signe chaque appareil et injecte tous les certificats de l’autorité de la chaîne dans l’appareil. Ce système est courant lorsque le fabricant sous contrat qui dispose d’une hiérarchie d’usines nomme une usine donnée pour la fabrication. Bien que la hiérarchie puisse comporter plusieurs niveaux (par exemple, par région/type de produit/ligne de fabrication), seule l’usine en bout de chaîne peut interagir avec l’appareil, mais la chaîne est conservée à partir du haut de la hiérarchie.

Dans d’autres chaînes, une autre autorité de certification intermédiaire peut interagir avec l’appareil, auquel cas l’interaction de l’autorité de certification avec l’appareil injecte le contenu de la chaîne d’approbation à ce stade. Des modèles hybrides sont également possibles : seules certaines autorités de certification ont alors une interaction physique avec l’appareil.

Dans notre exemple, l’usine Y et le technicien Z interagissent tous deux avec le widget Smart-X. Bien que l’entreprise X soit propriétaire du widget Smart-X, elle n’interagit pas physiquement avec celui-ci dans la chaîne logistique. La chaîne d’approbation de confiance du widget Smart-X implique donc que l’entreprise X signe l’usine Y, qui à son tour signe le technicien Z, qui fournira ensuite la signature finale du widget Smart-X. La fabrication et l’installation du widget Smart-X nécessitent que l’usine Y et le technicien Z utilisent leurs certificats d’autorité de certification intermédiaires respectifs pour signer chaque widget Smart-X. Résultat final : les widgets Smart-X disposent de certificats d’appareil uniques et la chaîne d’approbation de confiance remonte jusqu’au certificat d’autorité de certification de l’entreprise X.

![Chaîne d’approbation depuis les certificats d’une entreprise jusqu’aux certificats d’une autre entreprise](./media/iot-hub-x509ca-concept/cert-mfr-chain.png)

Ce point permet d’évaluer la valeur de la méthode d’autorité de certification X.509. Au lieu de prégénérer et de transmettre les certificats pour chaque widget Smart-X dans la chaîne logistique, l’entreprise X doit simplement effectuer une transmission unique à l’usine Y. Plutôt que de suivre chaque appareil tout au long de son cycle de vie, l’entreprise X peut à présent suivre et gérer les appareils via des groupes qui découlent naturellement du processus de la chaîne logistique, par exemple, les appareils installés par le technicien Z après le mois de juillet d’une année donnée.

Dernier point mais pas des moindres, la méthode d’authentification par l’autorité de certification ajoute la comptabilité sécurisée à la chaîne logistique de fabrication de l’appareil. Du fait du processus de la chaîne d’approbation, les actions de chaque membre de la chaîne sont enregistrées sous forme chiffrée et peuvent être vérifiées.

Ce processus repose sur certaines hypothèses qui doivent être vérifiées par souci d’exhaustivité. Il requiert la création indépendante d’une paire de clés publique/privée propre à l’appareil et la protection de la clé privée au sein de l’appareil. Heureusement, il existe des puces en silicium sécurisées sous forme de modèles de sécurité matériels capables de générer des clés en interne et de protéger les clés privées. L’entreprise X doit simplement ajouter une de ces puces à la nomenclature des composants du widget Smart-X.

## <a name="device-connection"></a>Connexion de l’appareil

Les sections précédentes amènent à la connexion de l’appareil. En inscrivant une seule fois un certificat d’autorité de certification X.509 dans IoT Hub, comment des millions d’appareils peuvent-ils se connecter et être authentifiés dès la première fois ?  C’est simple : via le flux de chargement de certificat et de preuve de possession abordé précédemment lors de l’inscription du certificat d’autorité de certification X.509.

Les appareils fabriqués pour l’authentification via l’autorité de certification X.509 sont équipés de certificats d’appareil uniques et d’une chaîne d’approbation à partir de leur chaîne logistique respective. La connexion de l’appareil, même la première fois, suit un processus en deux étapes : chargement de la chaîne d’approbation et preuve de possession.

Au cours du chargement de la chaîne d’approbation, l’appareil charge son certificat unique d’appareil avec la chaîne d’approbation correspondante dans IoT Hub. À l’aide du certificat de l’autorité de certification X.509 pré-enregistré, IoT Hub peut valider par chiffrement que la chaîne d’approbation chargée est cohérente en interne et qu’elle a été émise par le propriétaire valide du certificat de l’autorité de certification X.509. Avec uniquement le processus d’inscription d’autorité de certification X.509, IoT Hub lancerait un processus de réponse au défi de preuve de possession afin de s’assurer que la chaîne (et donc le certificat de l’appareil) appartient bien à l’appareil qui la charge. Pour ce faire, IoT Hub génère un défi aléatoire que l’appareil doit signer à l’aide de sa clé privée pour la validation par IoT Hub. Si la réponse est correcte, IoT Hub accepte l’appareil comme authentique et l’autorise à se connecter.

Dans notre exemple, chaque widget Smart-X chargerait son certificat d’appareil unique avec les certificats d’autorité de certification X.509 de l’usine Y et du technicien Z, et répondrait ensuite au défi de preuve de possession d’IoT Hub.

![Flux d’un certificat à un autre, négociation à partir du hub](./media/iot-hub-x509ca-concept/device-pop-flow.png)

Notez que la base de la confiance réside dans la protection des clés privées, y compris les clés privées des appareils. Nous ne pourrons jamais insister suffisamment sur l’importance des puces en silicium sécurisées sous la forme de modèles de sécurité matériels afin de protéger les clés privées des appareils et la meilleure pratique globale consistant à ne jamais partager les clés privées, par exemple une usine confiant sa clé privée à une autre.
