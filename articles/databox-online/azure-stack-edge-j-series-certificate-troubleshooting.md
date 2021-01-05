---
title: Résolution des problèmes liés aux certificats avec Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit la résolution des erreurs de certificat avec les appareils Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 11/20/2020
ms.author: alkohli
ms.openlocfilehash: a4848f5e8b542d80b382abe39f4882a26e0f534c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015678"
---
# <a name="troubleshooting-certificate-errors"></a>Résolution des erreurs de certificat

L’article explique comment résoudre les erreurs de certificat courantes lors de l’installation de certificats sur votre appareil Azure Stack Edge Pro.

## <a name="common-certificate-errors"></a>Erreurs de certificat courantes

Le tableau suivant répertorie les erreurs de certificat courantes et des informations détaillées sur ces erreurs et les solutions possibles :

> [!NOTE]
> Les occurrences de &#8220;{0}, {1}, ... , {n}&#8221; indiquent les paramètres positionnels. Les paramètres positionnels prennent des valeurs en fonction des certificats que vous utilisez.

| Code d'erreur | Description |
|---|---|
| CertificateManagement_UntrustedCertificate | Le certificat avec le nom d’objet {0} a une chaîne de certificats rompue. Téléchargez le certificat de chaîne de signature avant de charger ce certificat.|
| CertificateManagement_DeviceNotRegistered| Votre appareil n’est pas activé. Vous pouvez télécharger un certificat de support uniquement après l’activation.|
| CertificateManagement_ExpiredCertificate | Le certificat avec le type {0} a expiré ou arrive bientôt à expiration. Vérifiez l’expiration du certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_FormatMismatch | Le format brut n'est pas pris en charge. Vérifiez le format du certificat et, si nécessaire, apportez un nouveau certificat.  {0} attendu, {1} trouvé. |
| CertificateManagement_GenericError | Impossible d’effectuer l’opération de gestion des certificats. Recommencez cette opération après quelques minutes. Si le problème persiste, contactez le support technique Microsoft. |
| CertificateManagement_HttpsBindingFailure | Le certificat avec le nom d’objet {0} n’a pas pu créer un canal HTTPS sécurisé. Vérifiez le certificat que vous avez téléchargé et, si nécessaire, apportez un nouveau certificat. Cette erreur se produit avec le certificat du nœud de l’appareil.|
| CertificateManagement_IncorrectKeyCertSignKeyUsage | Le certificat avec le nom d’objet {0} ne doit pas avoir une signature de certificat d’utilisation de clé. Vérifiez l’utilisation de la clé du certificat et, si nécessaire, apportez un nouveau certificat. Cette erreur se produit avec le certificat de chaîne de signature. |
| CertificateManagement_IncorrectKeyNumber | Le certificat avec le nom d’objet {0} a un numéro de clé incorrect {1}. Vérifiez le numéro de clé du certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_InvalidP7b | Le fichier de certificat téléchargé n’est pas valide.  Vérifiez le format du certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_KeyAlgorithmNotRSA | Ce certificat n’utilise pas l’algorithme de clé RSA. Seuls les certificats RSA sont pris en charge. |
| CertificateManagement_KeySizeNotSufficient | Le certificat avec le nom d’objet {0} a une taille de clé insuffisante {1}. La taille de clé minimale est de 4096.|
| CertificateManagement_MissingClientOid | Le certificat avec le nom d’objet {0} n’a pas d’OID d’authentification du client. Vérifiez les propriétés de votre certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_MissingDigitalSignatureKeyUsage | Le certificat avec le nom d’objet {0} n’a pas de signature numérique dans l’utilisation de la clé. Vérifiez les propriétés de votre certificat et, si nécessaire, apportez un nouveau certificat. |
| CertificateManagement_MissingKeyCertSignKeyUsage | Le certificat avec le nom d’objet {0} n’a pas de signature de certificat dans l’utilisation de la clé. Vérifiez les propriétés de votre certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_MissingKeyEnciphermentKeyUsage | Le certificat avec le nom d’objet {0} n’a pas de chiffrement de clé dans l’utilisation de la clé. Vérifiez les propriétés de votre certificat et, si nécessaire, apportez un nouveau certificat. |
| CertificateManagement_MissingServerOid | Le certificat avec le nom d’objet {0} n’a pas d’OID d’authentification du serveur. Vérifiez les propriétés de votre certificat et, si nécessaire, apportez un nouveau certificat.|
| CertificateManagement_NameMismatch | Incompatibilité de type de certificat. Étendue attendue : {0}, valeur {1} trouvée. Téléchargez le certificat approprié.|
| CertificateManagement_NoPrivateKeyPresent | Le certificat avec le nom d’objet {0} n’a pas de clé privée présente. Téléchargez un certificat .pfx avec une clé privée.|
| CertificateManagement_NoRSACryptoPrivateKey | La clé privée pour le certificat avec le nom d’objet {0} n’est pas accessible. Assurez-vous d’utiliser un certificat pris en charge. Seul le certificat Fournisseur de services de chiffrement Microsoft RSA/SChannel Microsoft est pris en charge. |
| CertificateManagement_NotSelfSignedCertificate | Le certificat avec le nom d’objet {0} n’est pas auto-signé. Les certificats racines doivent être auto-signés |
| CertificateManagement_NotSupportedOnVirtualAppliance | Cette opération n'est pas prise en charge sur l'appareil virtuel. Cette erreur indique que la signature se produira uniquement avec Data Box Gateway s’exécutant dans une appliance cloud tactique. Cette erreur se produit lors de la gestion de l’appareil via Windows PowerShell.|
| CertificateManagement_SelfSignedCertificate | Le certificat avec le nom d’objet {0} est auto-signé. Téléchargez un certificat qui est correctement signé.|
| CertificateManagement_SignatureAlgorithmSha1 | Le certificat avec le nom d’objet {0} a un algorithme de signature non pris en charge. SHA1-RSA n’est pas pris en charge. |
| CertificateManagement_SubjectNamesInvalid | Le certificat avec le nom d’objet {0} ne possède pas le bon nom d’objet ou les bons autres noms d’objet le certificat {1}. Vérifiez le certificat que vous avez téléchargé et, si nécessaire, apportez un nouveau certificat. Vous devez également vérifier que le nom DNS correspond aux noms des SAN.|
| CertificateManagement_UnreadableCertificate | Impossible de lire le certificat avec le type {0}. Cette erreur se produit lorsque le certificat est illisible ou endommagé. Apportez un nouveau certificat.|
| CertificateSubjectNotFound | Le certificat avec le nom d’objet {0} est introuvable. Apportez un nouveau certificat.|
| CertificateRotationGenericFailure | Échec d’une ou de plusieurs rotations de certificats. Réessayez dans quelques minutes. Si le problème persiste, contactez le support technique Microsoft.|
| CertificateImportFailure | Le certificat avec l’empreinte numérique {0} n’a pas été importé dans le nœud {1}. Si le problème persiste, contactez le support technique Microsoft. |
| CertificateApplyFailure | Le certificat avec l’empreinte numérique {0} n’a pas été appliqué au nœud {1}. Si le problème persiste, contactez le support technique Microsoft.|
| NodeNotReachable | Impossible de valider le certificat sur {0}. Vérifiez l’intégrité des composants et des logiciels du système.|


## <a name="next-steps"></a>Étapes suivantes

[Conditions requises pour les certificats](azure-stack-edge-j-series-certificate-requirements.md)