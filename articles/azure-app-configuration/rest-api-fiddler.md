---
title: API REST Azure Active Directory - Test à l’aide de Fiddler
description: Utiliser Fiddler pour tester l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 1142aa25212d87c5484963cda4e172df3d1fbafc
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932606"
---
# <a name="test-the-azure-app-configuration-rest-api-using-fiddler"></a>Tester l’API REST Azure App Configuration à l’aide de Fiddler

Pour tester l’API REST à l’aide de [Fiddler](https://www.telerik.com/fiddler), vous devez inclure les en-têtes HTTP requis pour l’[authentification](./rest-api-authentication-hmac.md) dans vos demandes. Voici la procédure à suivre pour configurer Fiddler de manière à tester l’API REST en générant automatiquement les en-têtes d’authentification :

1. Assurez-vous que TLS 1.2 est un protocole autorisé :
    1. Accédez à **Outils** > **Options** > **HTTPS**).
    1. Vérifiez que l’option **Déchiffrer le trafic HTTPS** est cochée.
    1. Dans la liste des protocoles, ajoutez **tls1.2** s’il n’est pas présent.
1. Ouvrez **Éditeur de script Fiddler** ou appuyez sur **Ctrl-R** dans Fiddler
1. Ajoutez le code suivant dans la classe `Handlers` avant la fonction `OnBeforeRequest`

    ```js
    static function SignRequest(oSession: Session, credential: String, secret: String) {
        var utcNow = DateTimeOffset.UtcNow.ToString("r", System.Globalization.DateTimeFormatInfo.InvariantInfo);
        var contentHash = ComputeSHA256Hash(oSession.RequestBody);
        var stringToSign = oSession.RequestMethod.ToUpperInvariant() + "\n" + oSession.PathAndQuery + "\n" + utcNow +";" + oSession.hostname + ";" + contentHash;
        var signature = ComputeHMACHash(secret, stringToSign);

        oSession.oRequest.headers["x-ms-date"] = utcNow;
        oSession.oRequest.headers["x-ms-content-sha256"] = contentHash;
        oSession.oRequest.headers["Authorization"] = "HMAC-SHA256 Credential=" + credential + "&SignedHeaders=x-ms-date;host;x-ms-content-sha256&Signature=" + signature;
    }

    static function ComputeSHA256Hash(content: Byte[]) {
        var sha256 = System.Security.Cryptography.SHA256.Create();
        try {
            return Convert.ToBase64String(sha256.ComputeHash(content));
        }
        finally {
            sha256.Dispose();
        }
    }

    static function ComputeHMACHash(secret: String, content: String) {
        var hmac = new System.Security.Cryptography.HMACSHA256(Convert.FromBase64String(secret));
        try {
            return Convert.ToBase64String(hmac.ComputeHash(System.Text.Encoding.ASCII.GetBytes(content)));
        }
        finally {
            hmac.Dispose();
        }
    }
    ```

1. Ajoutez le code suivant à la fin de la fonction `OnBeforeRequest`. Mettez à jour la clé d’accès comme indiqué par le commentaire TODO.

    ```js
    if (oSession.isFlagSet(SessionFlags.RequestGeneratedByFiddler) &&
        oSession.hostname.EndsWith(".azconfig.io", StringComparison.OrdinalIgnoreCase)) {

        // TODO: Replace the following placeholders with your access key
        var credential = "<Credential>"; // Id
        var secret = "<Secret>"; // Value

        SignRequest(oSession, credential, secret);
    }
    ```
1. Utilisez l’éditeur [Composer de Fiddler](https://docs.telerik.com/fiddler/Generate-Traffic/Tasks/CreateNewRequest) pour générer et envoyer une demande
