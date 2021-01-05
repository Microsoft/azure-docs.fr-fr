---
title: Double chiffrement dans Microsoft Azure
description: Cet article explique comment Microsoft Azure fournit un double chiffrement pour les données au repos et en transit.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/13/2020
ms.author: terrylan
ms.openlocfilehash: 020e8249f57ccb1a14da798a717a00dcc3962389
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88226828"
---
# <a name="double-encryption"></a>Double chiffrement
On parle de double chiffrement quand deux couches de chiffrement indépendantes sont activées pour protéger contre les compromissions d’une couche de chiffrement quelconque. L’utilisation de deux couches de chiffrement atténue les menaces associées au chiffrement des données. Par exemple :

- erreurs de configuration dans le chiffrement des données ;
- erreurs d’implémentation dans l’algorithme de chiffrement ;
- compromission d’une clé de chiffrement.

Azure fournit un double chiffrement pour les données au repos et en transit.

## <a name="data-at-rest"></a>Données au repos
L’approche de Microsoft pour activer deux couches de chiffrement pour les données au repos est la suivante :

- **Chiffrement de disque à l’aide de clés gérées par le client**. Vous fournissez votre propre clé pour le chiffrement de disque. Vous pouvez apporter vos propres clés à votre Key Vault (BYOK) ou générer de nouvelles clés dans Azure Key Vault pour chiffrer les ressources souhaitées.
- **Chiffrement de l’infrastructure à l’aide de clés gérées par la plateforme**.  Par défaut, les disques sont automatiquement chiffrés au repos à l’aide de clés de chiffrement gérées par la plateforme.

## <a name="data-in-transit"></a>Données en transit
L’approche de Microsoft pour activer deux couches de chiffrement pour des données en transit est la suivante :

- **Chiffrement des données en transit à l’aide du protocole TLS (Transport Layer Security) 1.2 afin protéger les données lors de leur déplacement entre les services cloud et vous**. Tout le trafic sortant d’un centre de données est chiffré en transit, même si la destination du trafic est un autre contrôleur de domaine situé dans la même région. TLS 1.2 est le protocole de sécurité par défaut utilisé. TLS fournit une authentification forte, la confidentialité et l’intégrité des messages (activation de la détection de falsification et d’interception des messages), l’interopérabilité, la flexibilité des algorithmes, ainsi que la facilité de déploiement et d’utilisation.
- **Couche supplémentaire de chiffrement fournie au niveau de la couche d’infrastructure**. Cette méthode de chiffrement de la couche de liaison de données utilisant les normes de sécurité MAC IEEE 802.1 AE (également appelées MACsec) est appliquée de point à point sur le matériel réseau sous-jacent. Chaque fois que le trafic des clients Azure circule entre des centres de données (en dehors des limites physiques non contrôlées par Microsoft, ou pour le compte de Microsoft), les paquets sont chiffrés et déchiffrés sur les appareils avant leur envoi. Cela permet d’éviter les attaques physiques de l’intercepteur ou les attaques par snooping/écoutes téléphoniques. Étant donné que cette technologie est intégrée au matériel réseau, elle fournit un chiffrement de débit de ligne sur le matériel réseau sans augmentation mesurable de la latence de la liaison. Ce chiffrement MACsec est activé par défaut pour tout le trafic Azure au sein d’une région ou entre des régions, et aucune intervention des clients n’est nécessaire pour l’activer.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [utiliser le chiffrement dans Azure](encryption-overview.md).
