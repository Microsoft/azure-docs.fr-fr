---
title: Autorités de certification Azure Active Directory
description: Liste des certificats approuvés utilisés dans Azure
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: baselden
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: bea6f4161bcca063cd2c58d4c463473426f159ba
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96859129"
---
# <a name="certificate-authorities-used-by-azure-active-directory"></a>Autorités de certification utilisées par Azure Active Directory

> [!IMPORTANT]
> Les informations contenues dans cette page s’appliquent uniquement aux entités qui spécifient de manière explicite une liste d’autorités de certification acceptables. Cette pratique, appelée épinglage de certificat, ne doit être utilisée que s’il n’existe aucune autre option.

Les certificats des autorités de certification répertoriées ci-dessous sont présentés à toute entité qui tente d’accéder aux services d’identité Azure Active Directory (Azure AD) via les protocoles TLS/SSL. Si l’entité approuve ces autorités de certification, elle peut utiliser les certificats pour vérifier l’identité et la légitimité des services d’identité et établir des connexions sécurisées.

Les autorités de certification peuvent être classées en Autorités de certification racines et Autorités de certification intermédiaires. En général, les autorités de certification racines sont associées à une ou plusieurs autorités de certification intermédiaires. Cet article répertorie les autorités de certification racines utilisées par les services d’identité Azure AD et les autorités de certification intermédiaires associées à chacune de ces autorités racines. Pour chaque autorité de certification, nous indiquons les URI (Uniform Resource Identifiers) pour télécharger les fichiers AIA (Authority Information Access) et CDP (Certificate Revocation List Distribution Point) associés. Le cas échéant, nous indiquons également un URI vers le point de terminaison OCSP (Online Certificate Status Protocol).

## <a name="cas-used-in-azure-public-and-azure-us-government-clouds"></a>Autorités de certification utilisées dans les clouds Azure (public) et Azure US Government

Différents services peuvent utiliser des autorités de certification racines ou intermédiaires différentes. Par conséquent, toutes les entrées listées ci-dessous peuvent être obligatoires.

### <a name="digicert-global-root-g2"></a>DigiCert Global Root G2


| Autorité de certification racine| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - |- |-|-|-|-|
| DigiCert Global Root G2| 033af1e6a711a 9a0bb2864b11d09fae5| 1 août 2013 <br>15 janvier 2038| df3c24f9bfd666761b268 073fe06d1cc8d4f82a4| [AIA](http://cacerts.digicert.com/DigiCertGlobalRootG2.crt)<br>[CDP](http://crl3.digicert.com/DigiCertGlobalRootG2.crl) |


#### <a name="associated-intermediate-cas"></a>Autorités de certification intermédiaires associées

| Autorité de certification émettrice et intermédiaire| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - | 
| Microsoft Azure TLS Issuing CA 01| 0aafa6c5ca63c45141 ea3be1f7c75317| 29 juillet 2020<br>27 juin 2024| 2f2877c5d778c31e0f29c 7e371df5471bd673173| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2001%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2001.crl)|
|Microsoft Azure TLS Issuing CA 02| 0c6ae97cced59983 8690a00a9ea53214| 29 juillet 2020<br>27 juin 2024| e7eea674ca718e3befd 90858e09f8372ad0ae2aa| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2002%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2002.crl) |
| Microsoft Azure TLS Issuing CA 05| 0d7bede97d8209967a 52631b8bdd18bd| 29 juillet 2020<br>27 juin 2024| 6c3af02e7f269aa73a fd0eff2a88a4a1f04ed1e5| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2005%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2005.crl) |
| Microsoft Azure TLS Issuing CA 06| 02e79171fb8021e93fe 2d983834c50c0| 29 juillet 2020<br>27 juin 2024| 30e01761ab97e59a06b 41ef20af6f2de7ef4f7b0| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.cer)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.crl) |


 ### <a name="baltimore-cybertrust-root"></a>Baltimore CyberTrust Root

| Autorité de certification racine| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - | 
| Baltimore CyberTrust Root| 020000b9| 12 mai 2000<br>12 mai 2025| d4de20d05e66fc53fe 1a50882c78db2852cae474|<br>[CDP](http://crl3.digicert.com/Omniroot2025.crl)<br>[OCSP](http://ocsp.digicert.com/) |


#### <a name="associated-intermediate-cas"></a>Autorités de certification intermédiaires associées

| Autorité de certification émettrice et intermédiaire| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - | 
| Microsoft RSA TLS CA 01| 703d7a8f0ebf55aaa 59f98eaf4a206004eb2516a| 21 juillet 2020<br>8 octobre 2024| 417e225037fbfaa4f9 5761d5ae729e1aea7e3a42| [AIA](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2001.crt)<br>[CDP](https://mscrl.microsoft.com/pki/mscorp/crl/Microsoft%20RSA%20TLS%20CA%2001.crl)<br>[OCSP](http://ocsp.msocsp.com/) |
| Microsoft RSA TLS CA 02| b0c2d2d13cdd56cdaa 6ab6e2c04440be4a429c75| 21 juillet 2020<br>20 mai 2024| 54d9d20239080c32316ed 9ff980a48988f4adf2d| [AIA](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2002.crt)<br>[CDP](https://mscrl.microsoft.com/pki/mscorp/crl/Microsoft%20RSA%20TLS%20CA%2002.crl)<br>[OCSP](http://ocsp.msocsp.com/) |


 ### <a name="digicert-global-root-ca"></a>DigiCert Global Root CA

| Autorité de certification racine| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - | 
| DigiCert Global Root CA| 083be056904246 b1a1756ac95991c74a| 9 novembre 2006<br>9 novembre 2031| a8985d3a65e5e5c4b2d7 d66d40c6dd2fb19c5436| [CDP](http://crl3.digicert.com/DigiCertGlobalRootCA.crl)<br>[OCSP](http://ocsp.digicert.com/) |


#### <a name="associated-intermediate-cas"></a>Autorités de certification intermédiaires associées

| Autorité de certification émettrice et intermédiaire| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - |
| DigiCert SHA2 Secure Server CA| 01fda3eb6eca75c 888438b724bcfbc91| 8 mars 2013 8 mars 2023| 1fb86b1168ec743154062 e8c9cc5b171a4b7ccb4| [AIA](http://cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt)<br>[CDP](http://crl3.digicert.com/ssca-sha2-g6.crl)<br>[OCSP](http://ocsp.digicert.com/) |
| DigiCert SHA2 Secure Server CA |02742eaa17ca8e21 c717bb1ffcfd0ca0 |22 septembre 2020<br>22 septembre 2030|626d44e704d1ceabe3bf 0d53397464ac8080142c|[AIA](http://cacerts.digicert.com/DigiCertSHA2SecureServerCA-2.crt)<br>[CDP](http://crl3.digicert.com/DigiCertSHA2SecureServerCA.crl)<br>[OCSP](http://ocsp.digicert.com/)|


## <a name="cas-used-in-azure-china-21vianet-cloud"></a>Autorités de certification utilisées dans le cloud Azure China 21Vianet

### <a name="digicert-global-root-ca"></a>DigiCert Global Root CA


| Autorité de certification racine| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - |
| DigiCert Global Root CA| 083be056904246b 1a1756ac95991c74a| 9 novembre 2006<br>9 novembre 2031| a8985d3a65e5e5c4b2d7 d66d40c6dd2fb19c5436| [CDP](http://ocsp.digicert.com/)<br>[OCSP](http://crl3.digicert.com/DigiCertGlobalRootCA.crl) |


#### <a name="associated-intermediate-ca"></a>Autorité de certification intermédiaire associée

| Autorité de certification émettrice et intermédiaire| Numéro de série| Date d’émission Date d’expiration| Empreinte numérique SHA1| URI |
| - | - | - | - | - | - |
| DigiCert Basic RSA CN CA G2| 02f7e1f982bad 009aff47dc95741b2f6| 4 mars 2020<br>4 mars 2030| 4d1fa5d1fb1ac3917c08e 43f65015e6aea571179| [AIA](http://cacerts.digicert.cn/DigiCertBasicRSACNCAG2.crt)<br>[CDP](http://crl.digicert.cn/DigiCertBasicRSACNCAG2.crl)<br>[OCSP](http://ocsp.digicert.cn/) |

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les chaînes de chiffrement Microsoft 365](/microsoft-365/compliance/encryption-office-365-certificate-chains)
