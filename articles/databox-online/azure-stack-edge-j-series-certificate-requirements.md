---
title: Exigences applicables aux certificats et résolution des problèmes avec Azure Stack Edge Pro | Microsoft Docs
description: Décrit les exigences applicables aux certificats et la résolution des erreurs de certificat avec un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/17/2020
ms.author: alkohli
ms.openlocfilehash: de41bd030ea73ac68bfac5fbfbd03ae14cf7980f
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874234"
---
# <a name="certificate-requirements"></a>Configuration requise des certificats

Cet article décrit les exigences applicables aux certificats qui doivent être satisfaites pour que les certificats puissent être installés sur votre appareil Azure Stack Edge Pro. Les exigences ont trait aux certificats PFX, à l’autorité émettrice, au nom de l’objet et à l’autre nom de l’objet du certificat, ainsi qu’aux algorithmes de certificat pris en charge.

## <a name="certificate-issuing-authority"></a>Autorité émettrice du certificat

Les exigences d’émission de certificat sont les suivantes :

* Les certificats doivent être émis par une autorité de certification interne ou une autorité de certification publique.

* L’utilisation de certificats auto-signés n’est pas prise en charge.

* Les champs *Délivré à* et *Délivré par* ne peuvent pas avoir le même contenu, sauf pour les certificats d’autorité de certification racine.


## <a name="certificate-algorithms"></a>Algorithmes de certificat

Les algorithmes de certificat doivent répondre aux exigences suivantes :

* Les certificats doivent utiliser l’algorithme de clé RSA.

* Seuls les certificats RSA associés au Fournisseur de services de chiffrement Microsoft RSA/Schannel sont pris en charge.

* L’algorithme de signature de certificat ne peut pas être Secure Hash Algorithm 1.

* La taille de clé minimale est de 4096.

## <a name="certificate-subject-name-and-subject-alternative-name"></a>Nom de l’objet et autre nom de l’objet du certificat

Le nom de l’objet et autre nom de l’objet du certificat doivent répondre aux exigences suivantes :

* Vous pouvez utiliser un certificat générique unique couvrant tous les espaces de noms dans les champs Autre nom de l’objet (SAN) du certificat. Vous pouvez également utiliser des certificats individuels pour chaque espace de noms. Les deux approches requièrent l’utilisation de caractères génériques pour les points de terminaison si nécessaire, tel que les blobs.

* Assurez-vous que le nom de l’objet (nom commun dans le nom d’objet) fait partie de l’autre nom de l’objet dans l’extension de l’autre nom de l’objet.

* Vous pouvez utiliser un certificat générique unique couvrant tous les espaces de noms dans les champ SAN du certificat.

* Utilisez le tableau suivant lors de la création d’un certificat de point de terminaison :

    |Type |Nom de l’objet (SN)  |Autre nom de l’objet (SAN)  |Exemple de nom de sujet |
    |---------|---------|---------|---------|
    |Azure Resource Manager|`management.<Device name>.<Dns Domain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`|`management.mydevice1.microsoftdatabox.com` |
    |Stockage d'objets blob|`*.blob.<Device name>.<Dns Domain>`|`*.blob.< Device name>.<Dns Domain>`|`*.blob.mydevice1.microsoftdatabox.com` |
    |Interface utilisateur locale| `<Device name>.<DnsDomain>`|`<Device name>.<DnsDomain>`| `mydevice1.microsoftdatabox.com` |
    |Certificat unique à plusieurs SAN pour les deux points de terminaison|`<Device name>.<dnsdomain>`|`<Device name>.<dnsdomain>`<br>`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`<br>`*.blob.<Device name>.<Dns Domain>`|`mydevice1.microsoftdatabox.com` |
    |Nœud|`<NodeSerialNo>.<DnsDomain>`|`*.<DnsDomain>`<br><br>`<NodeSerialNo>.<DnsDomain>`|`mydevice1.microsoftdatabox.com` |
    |VPN|`AzureStackEdgeVPNCertificate.<DnsDomain>`<br><br> * AzureStackEdgeVPNCertificate est codé en dur.  | `*.<DnsDomain>`<br><br>`<AzureStackVPN>.<DnsDomain>` | `edgevpncertificate.microsoftdatabox.com`|
    
## <a name="pfx-certificate"></a>Certificat PFX

Les certificats PFX installés sur votre appareil Azure Stack Edge Pro doivent respecter les conditions suivantes :

* Lorsque vous recevez vos certificats de l’autorité SSL, assurez-vous que vous disposez de la chaîne de signature complète pour les certificats.

* Lorsque vous exportez un certificat PFX, assurez-vous que vous avez sélectionné l’option **Inclure tous les certificats dans la chaîne, si possible**.

* Utilisez un certificat PFX pour le point de terminaison, l’interface utilisateur locale, le nœud, le VPN et le Wi-Fi, car les clés tant publiques que privées sont requises pour Azure Stack Edge Pro. La clé privée doit être définie pour l’attribut de clé Ordinateur local.

* Le chiffrement PFX du certificat doit être 3DES. Il s’agit du chiffrement par défaut utilisé en cas d’exportation à partir d’un client Windows 10 ou d’un magasin de certificats Windows Server 2016. Pour plus d’informations sur 3DES, consultez [Triple DES](https://en.wikipedia.org/wiki/Triple_DES).

* Les fichiers PFX de certificat doivent disposer de valeurs *Signature numérique* et *KeyEncipherment* valides dans le champ *Utilisation de la clé*.

* Les fichiers PFX de certificat doivent avoir les valeurs *Authentification du serveur (1.3.6.1.5.5.7.3.1)* et *Authentification du client (1.3.6.1.5.5.7.3.2)* dans le champ *Utilisation avancée de la clé*.

* Les mots de passe de tous les fichiers PFX de certificat doivent être identiques au moment du déploiement si vous vous servez de l’Outil Azure Stack Readiness Checker. Pour plus d’informations, consultez [Créer des certificats pour votre Azure Stack Edge Pro à l’aide de l’Outil Azure Stack Hub Readiness Checker](azure-stack-edge-j-series-create-certificates-tool.md).

* Le mot de passe pour le fichier PFX de certificat doit être un mot de passe complexe. Notez ce mot de passe, car vous allez l’utiliser comme paramètre de déploiement.

* Utilisez uniquement les certificats RSA associés au Fournisseur de services de chiffrement Microsoft RSA/Schannel.

Pour plus d’informations, consultez [Exporter des certificats PFX avec une clé privée](azure-stack-edge-j-series-manage-certificates.md#export-certificates-as-pfx-format-with-private-key).

## <a name="next-steps"></a>Étapes suivantes

[Utiliser des certificats avec Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md)

[Créer des certificats pour votre instance Azure Stack Edge Pro avec l’outil Azure Stack Hub Readiness Checker](azure-stack-edge-j-series-create-certificates-tool.md)

[Exporter des certificats PFX avec une clé privée](azure-stack-edge-j-series-manage-certificates.md#export-certificates-as-pfx-format-with-private-key)

[Résolution des erreurs de certificat](azure-stack-edge-j-series-certificate-troubleshooting.md)