---
title: Modification des certificats Azure TLS
description: Modification des certificats Azure TLS
services: security
author: msmbaldwin
tags: azure-resource-manager
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.date: 11/10/2020
ms.author: mbaldwin
ms.openlocfilehash: 4e64d866b5bd2f725db3be31d0fdd2f8663cfd7c
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029691"
---
# <a name="azure-tls-certificate-changes"></a>Modification des certificats Azure TLS  

Microsoft met à jour les services Azure pour qu’ils utilisent des certificats TLS issus d’un autre ensemble d’autorités de certification racines. Cette modification est effectuée, car les certificats d’autorité de certification actuels ne sont pas conformes à l’une des exigences de base du CA/Browser Forum.

## <a name="when-will-this-change-happen"></a>Quand cette modification aura-t-elle lieu ?

Les points de terminaison Azure existants sont en transition de manière progressive depuis le 13 août 2020. Tous les points de terminaison Azure TLS/SSL nouvellement créés contiennent des certificats mis à jour, liés aux nouvelles autorités de certification racines.

Détails spécifiques au service :

- Les services [Azure Active Directory](../../active-directory/index.yml) (Azure AD) ont commencé cette transition le 7 juillet 2020.
- [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub) et [DPS](../../iot-dps/index.yml) resteront dans l’autorité de certification racine Baltimore CyberTrust, mais leurs autorités de certification intermédiaires changeront. [Pour plus d’informations, cliquez ici](https://techcommunity.microsoft.com/t5/internet-of-things/azure-iot-tls-changes-are-coming-and-why-you-should-care/ba-p/1658456).
- [Stockage Azure](../../storage/index.yml) restera dans l’autorité de certification racine Baltimore CyberTrust mais ses autorités de certification intermédiaires changeront. [Pour plus d’informations, cliquez ici](https://techcommunity.microsoft.com/t5/azure-storage/azure-storage-tls-changes-are-coming-and-why-you-care/ba-p/1705518).
- [Azure Cache pour Redis](../../azure-cache-for-redis/index.yml) restera dans l’autorité de certification racine Baltimore CyberTrust, mais ses autorités de certification intermédiaires changeront. [Pour plus d’informations, cliquez ici](../../azure-cache-for-redis/cache-whats-new.md).
- Azure Instance Metadata Service restera dans l’autorité de certification racine Baltimore CyberTrust mais ses autorités de certification intermédiaires changeront. [Pour plus d’informations, cliquez ici](https://docs.microsoft.com/answers/questions/172717/action-required-for-attested-data-tls-with-azure-i.html).

> [!IMPORTANT]
> Les clients devront peut-être mettre à jour leurs applications après cette modification pour éviter les échecs de connectivité quand ils tentent de se connecter aux services Azure.

## <a name="what-is-changing"></a>Qu’est-ce qui change ?

Aujourd’hui, la plupart des certificats TLS utilisés par les services Azure sont liés à l’autorité de certification racine suivante :

| Nom commun de l’autorité de certification | Empreinte (SHA1) |
|--|--|
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |

Les certificats TLS utilisés par les services Azure seront liés à l’une des autorités de certification racines suivantes :

| Nom commun de l’autorité de certification | Empreinte (SHA1) |
|--|--|
| [DigiCert Global Root G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |
| [DigiCert Global Root CA](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt) | a8985d3a65e5e5c4b2d7d66d40c6dd2fb19c5436 |
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [D-TRUST Root Class 3 CA 2 2009](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt) | 58e8abb0361533fb80f79b1b6d29d3ff8d5f00f0 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 | 
| [Microsoft ECC Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20ECC%20Root%20Certificate%20Authority%202017.crt) | 999a64c37ff47d9fab95f14769891460eec4c3c5 |

## <a name="when-can-i-retire-the-old-intermediate-thumbprint"></a>Quand pourrai-je supprimer l’ancienne empreinte intermédiaire ?

Les certificats d’autorité de certification actuels ne seront *pas* révoqués avant le 15 février 2021. Après cette date, vous pourrez supprimer les anciennes empreintes de votre code.

Si cette date change, vous serez informé de la nouvelle date de révocation.

## <a name="will-this-change-affect-me"></a>Ce changement m’affecte-t-il ? 

Nous pensons que **la plupart des clients Azure ne seront pas impactés**.  Toutefois, votre application peut être impactée si elle spécifie explicitement une liste d’autorités de certification acceptables. Cette pratique est appelée « épinglage de certificat ».

Voici quelques méthodes permettant de déterminer si votre application est impactée :

- Recherchez dans votre code source l’empreinte, le nom commun et les autres propriétés de certificat de toutes les autorités de certification Microsoft IT TLS mentionnées [ici](https://www.microsoft.com/pki/mscorp/cps/default.htm). S’il existe une correspondance, votre application sera impactée. Pour résoudre ce problème, mettez à jour le code source en incluant les nouvelles autorités de certification. Une bonne pratique consiste à vous assurer que les autorités de certification peuvent être ajoutées ou modifiées dans un court délai. Les réglementations du secteur exigent que les certificats d’autorité de certification soient remplacés sous 7 jours. Les clients qui s’appuient sur l’épinglage doivent donc réagir rapidement.

- Si vous disposez d’une application qui s’intègre à des API Azure ou d’autres services Azure et que vous ne savez pas si elle utilise l’épinglage de certificat, contactez le fournisseur de l’application.

- Les différents systèmes d’exploitation et runtimes de langage qui communiquent avec les services Azure peuvent nécessiter des étapes supplémentaires pour générer correctement la chaîne de certificats avec ces nouvelles racines :
    - **Linux** : de nombreuses distributions vous obligent à ajouter des autorités de certification à /etc/ssl/certs. Pour obtenir des instructions spécifiques, reportez-vous à la documentation de la distribution.
    - **Java** : assurez-vous que le magasin de clés Java contient les autorités de certification listées ci-dessus.
    - **Windows exécuté dans des environnements déconnectés** : pour les systèmes qui s’exécutent dans des environnements déconnectés, de nouvelles racines doivent être ajoutées au magasin d’autorités de certification racines de confiance et les intermédiaires au magasin d’autorités de certification intermédiaires.
    - **Android** : consultez la documentation de votre appareil et de votre version d’Android.
    - **Autres appareils, en particulier IoT** : contactez le fabricant de l’appareil.

- Si vous disposez d’un environnement dans lequel les règles de pare-feu sont configurées pour autoriser les appels sortants uniquement vers des emplacements de téléchargement de liste de révocation de certificats et/ou de vérification de protocole OCSP (Online Certificate Status Protocol) spécifiques, vous devez autoriser les URL correspondantes suivantes :

    - http://crl3&#46;digicert&#46;com
    - http://crl4&#46;digicert&#46;com
    - http://ocsp&#46;digicert&#46;com
    - http://www&#46;d-trust&#46;net
    - http://root-c3-ca2-2009&#46;ocsp&#46;d-trust&#46;net
    - http://crl&#46;microsoft&#46;com
    - http://oneocsp&#46;microsoft&#46;com
    - http://ocsp&#46;msocsp&#46;com
    - http://www&#46;microsoft&#46;com/pkiops

## <a name="next-steps"></a>Étapes suivantes

Si vous avez d’autres questions, contactez-nous par le biais du [support](https://azure.microsoft.com/support/options/).
