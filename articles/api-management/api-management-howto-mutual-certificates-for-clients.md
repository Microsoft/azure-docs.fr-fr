---
title: Sécuriser les API à l’aide d’une authentification par certificat client dans Gestion des API
titleSuffix: Azure API Management
description: Apprenez à sécuriser l’accès aux API à l’aide de certificats clients. Vous pouvez utiliser des expressions de stratégie pour valider des certificats entrants.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/13/2020
ms.author: apimpm
ms.openlocfilehash: 4e5522c162e08f0257bd6f20b058bf8bb858cff3
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93099344"
---
# <a name="how-to-secure-apis-using-client-certificate-authentication-in-api-management"></a>Comment sécuriser les API à l'aide d'une authentification par certificat client dans la Gestion des API

La Gestion des API permet de sécuriser l'accès aux API (par ex. client à gestion des API) en utilisant des certificats client. Vous pouvez valider le certificat entrant et en vérifier les propriétés en les comparant aux valeurs souhaitées à l’aide d’expressions de stratégie.

Pour savoir comment sécuriser l’accès au service back-end d’une API à l’aide de certificats clients (par exemple, de la Gestion des API vers le back-end), consultez [Comment sécuriser les services principaux à l’aide d’une authentification par certificat client dans la Gestion des API Azure](./api-management-howto-mutual-certificates.md).

> [!IMPORTANT]
> Pour recevoir et vérifier des certificats clients via HTTP/2 dans les niveaux Developer, Basic, Standard ou Premium, vous devez activer le paramètre « Négocier le certificat client » dans le panneau « Domaines personnalisés », comme indiqué ci-dessous.

![Négocier le certificat client](./media/api-management-howto-mutual-certificates-for-clients/negotiate-client-certificate.png)

> [!IMPORTANT]
> Pour recevoir et vérifier des certificats clients dans le niveau Consommation, vous devez activer le paramètre « Demander un certificat client » dans le panneau « Domaines personnalisés », comme indiqué ci-dessous.

![Demander un certificat client](./media/api-management-howto-mutual-certificates-for-clients/request-client-certificate.png)

## <a name="checking-the-issuer-and-subject"></a>Vérification de l’émetteur et du sujet

Les stratégies suivantes peuvent être configurées pour vérifier l’émetteur et le sujet d’un certificat client :

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Pour désactiver la vérification de la liste de révocation de certificats, utilisez `context.Request.Certificate.VerifyNoRevocation()` et non `context.Request.Certificate.Verify()`.
> Si le certificat client est auto-signé, le ou les certificats de l’autorité de certification racine (ou intermédiaire) doivent être [chargés](api-management-howto-ca-certificates.md) dans la Gestion des API pour `context.Request.Certificate.Verify()` et `context.Request.Certificate.VerifyNoRevocation()` pour fonctionner.

## <a name="checking-the-thumbprint"></a>Vérification de l’empreinte numérique

Les stratégies suivantes peuvent être configurées pour vérifier l’empreinte d’un certificat client :

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Thumbprint != "DESIRED-THUMBPRINT-IN-UPPER-CASE")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Pour désactiver la vérification de la liste de révocation de certificats, utilisez `context.Request.Certificate.VerifyNoRevocation()` et non `context.Request.Certificate.Verify()`.
> Si le certificat client est auto-signé, le ou les certificats de l’autorité de certification racine (ou intermédiaire) doivent être [chargés](api-management-howto-ca-certificates.md) dans la Gestion des API pour `context.Request.Certificate.Verify()` et `context.Request.Certificate.VerifyNoRevocation()` pour fonctionner.

## <a name="checking-a-thumbprint-against-certificates-uploaded-to-api-management"></a>Vérification d’une empreinte par rapport aux certificats téléchargés dans la gestion des API

L’exemple suivant montre comment vérifier l’empreinte d’un certificat client par rapport aux certificats téléchargés dans la gestion des API :

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>

```

> [!NOTE]
> Pour désactiver la vérification de la liste de révocation de certificats, utilisez `context.Request.Certificate.VerifyNoRevocation()` et non `context.Request.Certificate.Verify()`.
> Si le certificat client est auto-signé, le ou les certificats de l’autorité de certification racine (ou intermédiaire) doivent être [chargés](api-management-howto-ca-certificates.md) dans la Gestion des API pour `context.Request.Certificate.Verify()` et `context.Request.Certificate.VerifyNoRevocation()` pour fonctionner.

> [!TIP]
> Le problème de blocage de certificat client décrit dans cet [article](https://techcommunity.microsoft.com/t5/Networking-Blog/HTTPS-Client-Certificate-Request-freezes-when-the-Server-is/ba-p/339672) peut se manifester de différentes manières, notamment par des demandes qui se figent, des demandes qui génèrent un code d’état `403 Forbidden` après une expiration du délai, `context.Request.Certificate` qui a la valeur `null`. Ce problème affecte généralement les demandes `POST` et `PUT` avec une longueur de contenu d’environ 60 Ko ou plus.
> Pour éviter que ce problème ne se reproduise, activez le paramètre « Négocier le certificat client » pour les noms d’hôte souhaités dans le panneau « Domaines personnalisés », comme indiqué dans la première image de ce document. Cette fonctionnalité n’est pas disponible dans le niveau Consommation.


## <a name="next-steps"></a>Étapes suivantes

-   [Comment sécuriser les services principaux à l'aide d'une authentification par certificat client](./api-management-howto-mutual-certificates.md)
-   [Comment télécharger des certificats](./api-management-howto-mutual-certificates.md)
