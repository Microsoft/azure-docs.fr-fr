---
title: Résoudre les problèmes d’une instance Application Gateway dans Azure – ILB ASE | Microsoft Docs
description: Découvrez comment résoudre les problèmes d’une passerelle d’application à l’aide d’un équilibreur de charge interne (ILB) avec un environnement App Service dans Azure
services: vpn-gateway
documentationCenter: na
author: genlin
manager: dcscontentpm
editor: ''
tags: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/09/2020
ms.author: genli
ms.openlocfilehash: 8861e850e168169762d95c44a54b6a88a036f396
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84628529"
---
# <a name="back-end-server-certificate-is-not-allow-listed-for-an-application-gateway-using-an-internal-load-balancer-with-an-app-service-environment"></a>Un certificat du serveur back-end ne figure pas dans la liste verte pour une passerelle d’application qui utilise un équilibreur de charge interne avec un environnement ASE (App Service Environment)

Cet article permet de résoudre le problème suivant : un certificat n’est pas mis en liste verte lorsque vous créez une passerelle d’application en utilisant un équilibreur de charge interne (ILB) avec un environnement ASE (App Service Environment) sur le back end et que le chiffrement TLS de bout en bout est activé dans Azure.

## <a name="symptoms"></a>Symptômes

Lorsque vous créez une passerelle d’application à l’aide d’un équilibreur de charge interne avec un environnement App Service sur le serveur principal, le serveur principal peut devenir défectueux. Ce problème se produit si le certificat d’authentification de la passerelle d’application ne correspond pas au certificat configuré sur le serveur principal. Par exemple, considérez le scénario suivant :

**Configuration d’une passerelle d’application :**

- **Écouteur :** Multi-sites
- **Port :** 443
- **Nom d’hôte :** test.appgwtestase.com
- **Certificat SSL :** CN=test.appgwtestase.com
- **Pool principal :** Adresse IP ou nom de domaine complet
- **Adresse IP :** 10.1.5.11
- **Paramètres HTTP :** HTTPS
- **Port :** 443
- **Sonde personnalisée :** Nom d’hôte : test.appgwtestase.com
- **Certificat d’authentification :** .cer de test.appgwtestase.com
- **Intégrité du serveur principal :** Non sain : le certificat du serveur back-end ne figure pas dans la liste verte pour Application Gateway.

**Configuration ASE :**

- **IP ILB :** 10.1.5.11
- **Nom de domaine :** appgwtestase.com
- **App Service :** test.appgwtestase.com
- **Liaison SSL :** SNI SSL – CN=test.appgwtestase.com

Lorsque vous accédez à la passerelle d’application, le message d’erreur suivant s’affiche car le serveur principal est défectueux :

**502 - Le serveur Web a reçu une réponse erronée lors de son utilisation en tant que passerelle ou serveur proxy.**

## <a name="solution"></a>Solution

Lorsque vous n’utilisez pas un nom d’hôte pour accéder à un site web HTTPS, le serveur principal retournera le certificat configuré sur le site web par défaut si l’indication du nom du serveur est désactivée. Pour un environnement App Service ILB, le certificat par défaut est fourni par l’équilibreur de charge interne. S’il n’y a aucun certificat configuré pour l’équilibrage de charge interne, le certificat est fourni par l’application ASE.

Lorsque vous utilisez un nom de domaine complet (FQDN) pour accéder à l’équilibreur de charge interne, le serveur principal retournera le certificat approprié chargé dans les paramètres HTTP. Dans ce cas, envisagez les options suivantes :

- Utilisez le nom de domaine complet dans le pool principal de la passerelle d’application pour pointer vers l’adresse IP de l’équilibreur de charge interne. Cette option fonctionne uniquement si vous disposez d’une zone DNS privée ou d’un DNS personnalisé configuré. Sinon, vous devez créer un enregistrement « A » pour un service DNS public.

- Utilisez le certificat chargé sur l’ILB ou le certificat par défaut (certificat ILB) dans les paramètres HTTP. La passerelle d’application obtient le certificat lorsqu’elle accède à l’adresse IP de l’équilibreur de charge interne pour la sonde.

- Utilisez un certificat générique sur l’ILB et le serveur principal afin que le certificat soit commun à tous les sites web. Toutefois, cette solution est possible uniquement dans le cas de sous-domaine, et pas si chacun des sites Web nécessite des noms d’hôtes différents.

- Effacez l’option **Utilisation pour App service** pour la passerelle d’application si vous utilisez l’adresse IP de l’ILB.

Pour réduire la surcharge, vous pouvez charger le certificat de l’équilibreur de charge interne dans les paramètres HTTP pour que le chemin d’accès de la sonde fonctionne. (Cette étape sert uniquement à la mise en liste verte. Elle ne sera pas utilisée pour la communication TLS.) Vous pouvez récupérer le certificat ILB en accédant à l’ILB via son adresse IP depuis votre navigateur sur le protocole HTTPS, en exportant le certificat TLS/SSL au format CER encodé en base 64, puis en le chargeant dans les paramètres HTTP respectifs.

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.
