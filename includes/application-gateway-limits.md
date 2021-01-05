---
author: vhorne
ms.service: application-gateway
ms.topic: include
ms.date: 03/04/2020
ms.author: victorh
ms.openlocfilehash: ff97aa6c6f04ad41ba6e1b986f3cc0734ec7a326
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92526113"
---
| Ressource | Limite | Remarque |
| --- | --- | --- |
| Azure Application Gateway |1 000 par abonnement | |
| Configuration d’adresses IP frontales |2 |1 publique et 1 privée |
| Ports frontaux |100<sup>1</sup> | |
| Pool d'adresses principales |100<sup>1</sup> | |
| Serveurs principaux par pool |1,200 | |
| Écouteurs HTTP |200<sup>1</sup> |Limité à 100 écouteurs actifs qui routent le trafic. Écouteurs actifs = nombre total d’écouteurs - écouteurs non actifs.<br>Si une configuration par défaut à l’intérieur d’une règle de routage est définie pour router le trafic (avec, par exemple, un écouteur, un pool de back-ends et des paramètres HTTP), elle est également considérée comme un écouteur.|
| Règles d’équilibrage de charge HTTP |100<sup>1</sup> | |
| Paramètres HTTP du serveur principal |100<sup>1</sup> | |
| Instances par passerelle |Référence SKU V1 - 32<br>Référence SKU V2 - 125 | |
| Certificats SSL |100<sup>1</sup> |1 par écouteur HTTP |
| Taille maximale du certificat SSL |RÉFÉRENCE SKU V1 : 10 Ko<br>RÉFÉRENCE SKU V2 : 16 Ko| |
| Certificats d’authentification |100 | |
| Certificats racines approuvés |100 | |
| Délai minimum d’expiration de la demande |1 seconde | |
| Délai maximum d’expiration de la demande |24 heures | |
| Nombre de sites |100<sup>1</sup> |1 par écouteur HTTP |
| Mappages d’URL par écouteur |1 | |
| Nombre maximal de règles basées sur le chemin par mappage d’URL|100||
| Configurations de redirection |100<sup>1</sup>| |
| Connexions WebSocket simultanées |Passerelles moyenne 20 Ko<br> Passerelles volumineuses 50 Ko| |
| Longueur maximale d’URL|32 Ko| |
| Taille d’en-tête maximale pour HTTP/2 |4 Ko| |
| Taille de téléchargement maximale Standard |2 Go | |
| Taille de téléchargement maximale WAF |Passerelles v1 WAF moyennes, 100 Mo<br>Grandes passerelles v1 WAF, 500 Mo<br>WAF v2, 750 Mo| |
| Limite de taille de corps de WAF, sans les fichiers|128 Ko||
| Règles personnalisées WAF maximales|100||
| Exclusions WAF maximales par passerelle applicative|40||

<sup>1</sup> Dans le cas des références SKU compatibles WAF, vous devez limiter le nombre de ressources à 40.
