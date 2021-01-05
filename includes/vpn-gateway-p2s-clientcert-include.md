---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/28/2020
ms.author: cherylmc
ms.openlocfilehash: 34986ac80a309bcfd495e5782496ba560f84c5f7
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041557"
---
Chaque ordinateur client que vous connectez à un réseau virtuel avec une connexion point à site doit avoir un certificat client installé. Ce certificat doit être généré à partir du certificat racine, puis installé sur chaque ordinateur client. Si vous n’installez pas de certificat client valide, l’authentification échoue lorsque le client essaie de se connecter au réseau virtuel.

Vous pouvez soit générer un certificat unique pour chaque client, soir utiliser le même certificat pour plusieurs clients. Le fait de générer des certificats clients uniques vous offre la possibilité de révoquer un seul certificat. Dans le cas contraire, si plusieurs clients utilisent le même certificat client pour s’authentifier et que vous révoquez ce dernier, vous devrez générer et installer de nouveaux certificats pour chaque client qui utilise ce certificat.

Vous pouvez générer des certificats clients à l’aide des méthodes suivantes :

* **Certificat d’entreprise :**

  * Si vous utilisez une solution de certificat d’entreprise, générez un certificat client avec le format de valeur de nom commun *name\@yourdomain.com*. Utilisez ce format au lieu du format *domain name\username*.

  * Assurez-vous que le certificat client repose sur un modèle de certificat utilisateur qui indique *Authentification client* comme premier élément dans la liste d’utilisateurs. Vérifiez le certificat en double-cliquant dessus et en affichant **Utilisation avancée de la clé** dans l’onglet **Détails**.

* **Certificat racine auto-signé :** Suivez la procédure décrite dans l’un des articles concernant les certificats P2S ci-dessous pour créer des certificats clients compatibles avec vos connexions P2S.

  Lorsque vous générez un certificat client à partir d’un certificat racine auto-signé, ce certificat est automatiquement installé sur l’ordinateur que vous avez utilisé pour le générer. Si vous souhaitez installer un certificat client sur un autre ordinateur client, exportez-le en tant que fichier .pfx, avec l’intégralité de la chaîne du certificat. Cette opération crée un fichier .pfx contenant les informations de certificat racine requises pour l’authentification du client.

  Les procédures décrites dans ces articles permettent de générer un certificat client compatible, que vous pouvez ensuite exporter et distribuer.

  * [Instructions pour PowerShell sur Windows 10](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientcert) : ces instructions requièrent Windows 10 et PowerShell pour générer des certificats. Les certificats qui sont générés peuvent être installés sur n’importe quel client P2S pris en charge.

  * [Instructions pour MakeCert](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md) : Utilisez MakeCert si vous n’avez pas accès à un ordinateur Windows 10 pour générer des certificats. Même si MakeCert est déconseillé, vous pouvez toujours l’utiliser pour générer des certificats. Vous pouvez installer les certificats générés sur n’importe quel client P2S pris en charge.

  * [Instructions Linux](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-linux.md).