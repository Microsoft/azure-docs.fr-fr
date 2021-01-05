---
title: Nouveautés d’Azure Cache pour Redis
description: Mises à jour récentes pour Azure Cache pour Redis
author: yegu-ms
ms.service: cache
ms.topic: reference
ms.date: 09/28/2020
ms.author: yegu
ms.openlocfilehash: b30e83b89b25e6400b8c7e0419406631fa1edcd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91491670"
---
# <a name="whats-new-in-azure-cache-for-redis"></a>Nouveautés d’Azure Cache pour Redis

## <a name="azure-tls-certificate-change"></a>Changement de certificat Azure TLS

Microsoft met à jour les services Azure pour qu’ils utilisent des certificats de serveur TLS issus d’un autre ensemble d’autorités de certification (CA). Cette modification est déployée dans les phases du 13 août 2020 au 26 octobre 2020 (estimation). Azure apporte ce changement parce que [les certificats d’autorité de certification actuels ne sont pas conformes à l’une des exigences de base du forum sur l’autorité de certification/navigateur](https://bugzilla.mozilla.org/show_bug.cgi?id=1649951). Le problème a été signalé le 1er juillet 2020 et s’applique à plusieurs fournisseurs d’infrastructure à clé publique (PKI) populaires dans le monde entier. La plupart des certificats TLS qu’utilisent les services Azure proviennent aujourd’hui de l’infrastructure à clé publique *Baltimore CyberTrust Root*. Le service Azure Cache pour Redis continue d’être chaîné à Baltimore CyberTrust Root. Toutefois, ses certificats de serveur TLS seront émis par de nouvelles autorités de certification intermédiaires (ICA) à partir du 12 octobre 2020.

> [!NOTE]
> Cette modification est limitée aux services dans les [régions Azure](https://azure.microsoft.com/global-infrastructure/geographies/) publiques. Elle exclut les clouds souverains (par ex., de Chine) ou gouvernementaux.
>
>

### <a name="does-this-change-affect-me"></a>Ce changement m’affecte-t-il ?

Nous pensons que la plupart des clients d’Azure Cache pour Redis ne sont pas affectés par la modification. Votre application peut être affectée si elle spécifie explicitement une liste de certificats acceptables, pratique appelée « épinglage de certificat ». En cas d’épinglage à un certificat intermédiaire ou feuille au lieu de Baltimore CyberTrust Root, vous devez **prendre des mesures immédiates** pour modifier la configuration du certificat.

Le tableau suivant fournit des informations sur les certificats en cours de déploiement. En fonction du certificat que votre application utilise, il se peut que vous deviez le mettre à jour pour éviter tout perte de connectivité à votre instance Azure Cache pour Redis.

| Type d'autorité de certification | Actuel | Post déploiement (12 octobre 2020) | Action |
| ----- | ----- | ----- | ----- |
| Root | Thumbprint : d4de20d05e66fc53fe1a50882c78db2852cae474<br><br> Expiration : Lundi 12 mai 2025, 16:59:00<br><br> Nom de l’objet :<br> CN = Baltimore CyberTrust Root<br> OU = CyberTrust<br> O = Baltimore<br> C = IE | Pas de modification | Aucun |
| Intermédiaires | Thumbprints :<br> CN = Microsoft IT TLS CA 1<br> Empreinte : 417e225037fbfaa4f95761d5ae729e1aea7e3a42<br><br> CN = Microsoft IT TLS CA 2<br> Empreinte : 54d9d20239080c32316ed9ff980a48988f4adf2d<br><br> CN = Microsoft IT TLS CA 4<br> Empreinte : 8a38755d0996823fe8fa3116a277ce446eac4e99<br><br> CN = Microsoft IT TLS CA 5<br> Empreinte : Ad898ac73df333eb60ac1f5fc6c4b2219ddb79b7<br><br> Expiration : Vendredi 20 mai 2024 5:52:38<br><br> Nom de l’objet :<br> OU = Microsoft IT<br> O = Microsoft Corporation<br> L = Redmond<br> S = Washington<br> C = US<br> | Thumbprints :<br> CN = Microsoft RSA TLS CA 01<br> Empreinte : 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a<br><br> CN = Microsoft RSA TLS CA 02<br> Thumbprint : b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75<br><br> Expiration : Mardi 8 octobre 2024 12:00:00 ;<br><br> Nom de l’objet :<br> O = Microsoft Corporation<br> C = US<br> | Obligatoire |

### <a name="what-actions-should-i-take"></a>Que dois-je faire ?

Si votre application utilise le magasin de certificats du système d’exploitation ou épingle la racine Baltimore parmi d’autres, aucune action n’est nécessaire. En revanche, si votre application épingle un certificat TLS intermédiaire ou feuille, nous vous recommandons d’épingler les racines suivantes :

| Certificat | Empreinte numérique |
| ----- | ----- |
| [Autorité de certification racine Baltimore](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 |
| [Digicert Global Root G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |

> [!TIP]
> Les certificats intermédiaires et feuilles sont censés changer fréquemment. Nous vous recommandons de ne pas pendre de dépendance sur ceux-ci. Au lieu de cela, épinglez votre application à un certificat racine, car il est déployé moins fréquemment.
>
>

Pour continuer à épingler des certificats intermédiaires, ajoutez ce qui suit à la liste des certificats intermédiaires épinglés, qui en comprend quelques autres pour minimiser les modifications futures :

| Nom commun de l’autorité de certification | Empreinte numérique |
| ----- | ----- |
| [Microsoft RSA TLS CA 01](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2001.crt) | 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a |
| [Microsoft RSA TLS CA 02](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2002.crt) | b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75 |
| [Microsoft Azure TLS Issuing CA 01](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2001.cer) | 2f2877c5d778c31e0f29c7e371df5471bd673173 |
| [Microsoft Azure TLS Issuing CA 02](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2002.cer) | e7eea674ca718e3befd90858e09f8372ad0ae2aa |
| [Microsoft Azure TLS Issuing CA 05](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2005.cer) | 6c3af02e7f269aa73afd0eff2a88a4a1f04ed1e5 |
| [Microsoft Azure TLS Issuing CA 06](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.cer) | 30e01761ab97e59a06b41ef20af6f2de7ef4f7b0 |

Si votre application valide le certificat dans le code, vous devez le modifier pour reconnaître les propriétés (par exemple, émetteurs, thumbprint) des certificats nouvellement épinglés. Cette vérification supplémentaire doit couvrir tous les certificats épinglés pour être plus sécurisés.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez d’autres questions, contactez-nous par le biais du [support](https://azure.microsoft.com/support/options/).  
