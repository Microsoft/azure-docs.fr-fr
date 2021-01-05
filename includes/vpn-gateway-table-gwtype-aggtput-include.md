---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 11/12/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 1643b20c6c157c43e93967cef364e703dbf4478e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010766"
---
|**Génération<br>de passerelle<br>VPN** |**Référence (SKU)**   | **S2S/VNet-to-VNet<br>Tunnels** | **P2S<br> connexions SSTP** | **P2S<br> connexions IKEv2/OpenVPN** | **Agrégat<br>Référence de débit** | **BGP** | **Redondant interzone** |
|---            |---         | ---        | ---       | ---            | ---       | ---       | ---|
|**Génération1**|**De base**   | Bande passante 10    | Bande passante 128  | Non pris en charge  | 100 Mbits/s  | Non pris en charge| Non |
|**Génération1**|**VpnGw1**  | Bande passante 30*   | Bande passante 128  | Bande passante 250       | 650 Mbits/s  | Prise en charge | Non |
|**Génération1**|**VpnGw2**  | Bande passante 30*   | Bande passante 128  | Bande passante 500       | 1 Gbit/s    | Prise en charge | Non |
|**Génération1**|**VpnGw3**  | Bande passante 30*   | Bande passante 128  | Bande passante 1 000      | 1,25 Gbits/s | Prise en charge | Non |
|**Génération1**|**VpnGw1AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 250       | 650 Mbits/s  | Prise en charge | Oui |
|**Génération1**|**VpnGw2AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 500       | 1 Gbit/s    | Prise en charge | Oui |
|**Génération1**|**VpnGw3AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 1 000      | 1,25 Gbits/s | Prise en charge | Oui |
|        |            |            |           |                |           |           |     |
|**Génération2**|**VpnGw2**  | Bande passante 30*   | Bande passante 128  | Bande passante 500       | 1,25 Gbits/s | Prise en charge | Non |
|**Génération2**|**VpnGw3**  | Bande passante 30*   | Bande passante 128  | Bande passante 1 000      | 2,5 Gbits/s  | Prise en charge | Non |
|**Génération2**|**VpnGw4**  | Bande passante 30*   | Bande passante 128  | Bande passante 5 000      | 5 Gbit/s    | Prise en charge | Non |
|**Génération2**|**VpnGw5**  | Bande passante 30*   | Bande passante 128  | Bande passante 10000      | 10 Gbits/s   | Prise en charge | Non |
|**Génération2**|**VpnGw2AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 500       | 1,25 Gbits/s | Prise en charge | Oui |
|**Génération2**|**VpnGw3AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 1 000      | 2,5 Gbits/s  | Prise en charge | Oui |
|**Génération2**|**VpnGw4AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 5 000      | 5 Gbit/s    | Prise en charge | Oui |
|**Génération2**|**VpnGw5AZ**| Bande passante 30*   | Bande passante 128  | Bande passante 10000      | 10 Gbits/s   | Prise en charge | Oui |

(*) Utilisez le [WAN virtuel](../articles/virtual-wan/virtual-wan-about.md) si vous avez besoin de plus de 30 tunnels VPN S2S.

* Le redimensionnement des références SKU VpnGw est autorisé dans la même génération, à l’exception du redimensionnement de la référence SKU De base. La référence SKU De base est une référence SKU héritée et présente des limitations en termes de fonctionnalités. Pour passer de la référence SKU De base à une autre référence SKU VpnGw, vous devez supprimer la passerelle VPN de la référence SKU De base et créer une passerelle avec la combinaison de taille de référence SKU et de génération souhaitée.

* Ces limites de connexion sont séparées. Par exemple, vous pouvez avoir 128 connexions SSTP et 250 connexions IKEv2 sur une référence SKU VpnGw1.

* Pour des informations sur les prix, consultez la page [Tarification](https://azure.microsoft.com/pricing/details/vpn-gateway) .

* Vous trouverez des informations relatives au contrat de niveau de service (SLA) sur la page [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/).

* Dans un tunnel unique, un débit maximal de 1 Gbit/s peut être atteint. La référence de débit agrégée indiquée dans le tableau ci-dessus se base sur les mesures de plusieurs tunnels agrégés par le biais d’une passerelle unique. Le test d’évaluation du débit d’agrégat pour une passerelle VPN est S2S + P2S combinés. **Si vous avez un grand nombre de connexions P2S, cela peut avoir un impact négatif sur la connexion S2S à cause des limites de débit.** La référence de débit agrégée n’est pas garantie en raison des conditions de trafic Internet et des comportements de votre application.

Pour aider nos clients à comprendre les performances relatives des références SKU à l’aide de différents algorithmes, nous avons utilisé les outils disponibles publiquement iPerf et CTSTraffic pour mesurer les performances. Le tableau ci-dessous liste les résultats des tests de performances des références SKU VpnGw de génération 1. Comme vous pouvez le voir, les meilleures performances sont obtenues quand nous utilisons l’algorithme GCMAES256 pour le chiffrement IPsec et pour l’intégrité. Nous obtenons des performances moyennes lors de l’utilisation d’AES256 pour le chiffrement IPsec et de SHA256 pour l’intégrité. Quand nous utilisons DES3 pour le chiffrement IPsec et SHA256 pour l’intégrité, nous obtenons les performances les plus faibles.

|**Génération**|**Référence (SKU)**   | **Algorithmes<br>utilisés** | **Débit<br>observé** | **Paquets par seconde<br>observés** |
|---           |---       | ---                 | ---            | ---                    |
|**Génération1**|**VpnGw1**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 650 Mbits/s<br>500 Mbits/s<br>120 Mbits/s   | 58 000<br>50 000<br>50 000|
|**Génération1**|**VpnGw2**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 1 Gbit/s<br>500 Mbits/s<br>120 Mbits/s | 90 000<br>80 000<br>55 000|
|**Génération1**|**VpnGw3**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 1,25 Gbits/s<br>550 Mbits/s<br>120 Mbits/s | 105 000<br>90 000<br>60 000|
|**Génération1**|**VpnGw1AZ**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 650 Mbits/s<br>500 Mbits/s<br>120 Mbits/s   | 58 000<br>50 000<br>50 000|
|**Génération1**|**VpnGw2AZ**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 1 Gbit/s<br>500 Mbits/s<br>120 Mbits/s | 90 000<br>80 000<br>55 000|
|**Génération1**|**VpnGw3AZ**| GCMAES256<br>AES256 et SHA256<br>DES3 et SHA256| 1,25 Gbits/s<br>550 Mbits/s<br>120 Mbits/s | 105 000<br>90 000<br>60 000|
