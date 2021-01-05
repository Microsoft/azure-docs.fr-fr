---
title: Résolution des problèmes de compression de fichiers dans Azure CDN | Microsoft Docs
description: Découvrez comment résoudre les problèmes de compression de fichiers dans Azure Content Delivery Network. Cet article aborde plusieurs causes possibles.
services: cdn
documentationcenter: ''
author: sohamnc
manager: danielgi
editor: ''
ms.assetid: a6624e65-1a77-4486-b473-8d720ce28f8b
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: f49af1488a0c044639a72fc2ea52ba0a47727a24
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996149"
---
# <a name="troubleshooting-cdn-file-compression"></a>Résolution des problèmes de compression des fichiers CDN
Cet article vous aide à résoudre les problèmes de [compression des fichiers CDN](cdn-improve-performance.md).

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site de support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.

## <a name="symptom"></a>Symptôme
La compression pour votre point de terminaison est activée, mais les fichiers sont renvoyés non compressés.

> [!TIP]
> Pour vérifier si vos fichiers sont retournés sous une forme compressée, vous devez utiliser un outil tel que [Fiddler](https://www.telerik.com/fiddler) ou les [outils de développement](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) de votre navigateur.  Vérifiez les en-têtes de réponse HTTP retournés avec votre contenu CDN mis en cache.  S’il existe un en-tête nommé `Content-Encoding` avec une valeur **gzip**, **bzip2**, ou **deflate**, votre contenu est compressé.
> 
> ![En-tête d’encodage de contenu](./media/cdn-troubleshoot-compression/cdn-content-header.png)
> 
> 

## <a name="cause"></a>Cause
Il existe plusieurs causes possibles, y compris :

* Le contenu demandé n’est pas éligible pour la compression.
* La compression n’est pas activée pour le type de fichier demandé.
* La requête HTTP ne comprenait pas un en-tête demandant un type de compression valide.
* L’origine envoie du contenu segmenté.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
> [!TIP]
> Comme lors du déploiement de nouveaux points de terminaison, les modifications de configuration CDN prennent un certain temps à se propager sur le réseau.  En règle générale, les modifications sont appliquées dans les 90 minutes.  S’il s’agit de la première fois que vous avez configuré la compression pour votre point de terminaison CDN, vous devez envisager d’attendre 1 à 2 heures pour vous assurer que les paramètres de compression ont été transmis aux POP. 
> 
> 

### <a name="verify-the-request"></a>Vérifier la requête
Tout d’abord, effectuez une vérification rapide de l’intégrité de la requête.  Vous pouvez utiliser les [outils de développement](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) de votre navigateur pour afficher les requêtes en cours.

* Vérifiez que la requête est envoyée à l’URL de point de terminaison, `<endpointname>.azureedge.net`, et non à l’origine.
* Vérifiez que la requête contient un en-tête **Accept-Encoding** et que la valeur de cet en-tête contient **gzip**, **deflate** ou **bzip2**.

> [!NOTE]
> Les profils du **CDN Azure fourni par Akamai** prennent uniquement en charge l’encodage **gzip**.
> 
> 

![En-têtes de requête CDN](./media/cdn-troubleshoot-compression/cdn-request-headers.png)

### <a name="verify-compression-settings-standard-cdn-profiles"></a>Vérifier les paramètres de compression (profils CDN Standard)
> [!NOTE]
> Cette étape s’applique uniquement si votre profil CDN est un profil du **CDN Azure Standard de Microsoft**, du **CDN Azure Standard de Verizon** ou du **CDN Azure Standard fourni d’Akamai**. 
> 
> 

Accédez à votre point de terminaison dans le [portail Azure](https://portal.azure.com) et cliquez sur le bouton **Configurer** .

* Vérifiez que la compression est activée.
* Vérifiez que le type MIME pour le contenu à compresser est inclus dans la liste des formats compressés.

![Paramètres de compression CDN](./media/cdn-troubleshoot-compression/cdn-compression-settings.png)

### <a name="verify-compression-settings-premium-cdn-profiles"></a>Vérifier les paramètres de compression (profils CDN Premium)
> [!NOTE]
> Cette étape s’applique uniquement si votre profil CDN est un profil du **CDN Azure Premium de Verizon**.
> 
> 

Accédez à votre point de terminaison dans le [portail Azure](https://portal.azure.com) et cliquez sur le bouton **Gérer** .  Le portail supplémentaire s’ouvre.  Pointez sur l’onglet **HTTP volumineux**, puis pointez sur le menu volant **Paramètres de cache**.  Cliquez sur **Compression**. 

* Vérifiez que la compression est activée.
* Vérifiez que la liste **Types de fichiers** contient une liste séparée par des virgules (sans espace) des types MIME.
* Vérifiez que le type MIME pour le contenu à compresser est inclus dans la liste des formats compressés.

![Paramètres de compression Premium CDN](./media/cdn-troubleshoot-compression/cdn-compression-settings-premium.png)

### <a name="verify-the-content-is-cached-verizon-cdn-profiles"></a>Vérifier que le contenu est mis en cache (profils CDN Verizon)
> [!NOTE]
> Cette étape s’applique uniquement si votre profil CDN est un profil du **CDN Azure Standard de Verizon** ou du **CDN Azure Premium de Verizon**.
> 
> 

À l’aide des outils de développement de votre navigateur, vérifiez les en-têtes de réponse pour vous assurer que le fichier est mis en cache dans la région où il est demandé.

* Vérifiez l’en-tête de réponse **Server** .  L’en-tête doit être au format **Plateforme (ID serveur/POP)**, comme indiqué dans l’exemple suivant.
* Vérifiez l’en-tête de réponse **X-Cache** .  L’en-tête doit indiquer **HIT**.  

![En-têtes de réponse CDN](./media/cdn-troubleshoot-compression/cdn-response-headers.png)

### <a name="verify-the-file-meets-the-size-requirements-verizon-cdn-profiles"></a>Vérifier que le fichier respecte les critères de taille (profils CDN Verizon)
> [!NOTE]
> Cette étape s’applique uniquement si votre profil CDN est un profil du **CDN Azure Standard de Verizon** ou du **CDN Azure Premium de Verizon**.
> 
> 

Afin d’être éligible pour la compression, un fichier doit respecter les exigences de taille suivantes :

* Plus de 128 octets (Longueur du contenu : 128)
* Moins de 3 Mo.

### <a name="check-the-request-at-the-origin-server-for-a-via-header"></a>Vérifiez la requête sur le serveur d’origine pour un en-tête **Via**
L’en-tête HTTP **Via** indique au serveur web que la requête est transmise par un serveur proxy.  Par défaut, des serveurs web Microsoft IIS ne compressent pas les réponses lorsque la requête contient un en-tête **Via** .  Pour modifier ce comportement, procédez comme suit :

* **IIS 6**: [définissez HcNoCompressionForProxies="FALSE" dans les propriétés de la métabase IIS](/previous-versions/iis/6.0-sdk/ms525390(v=vs.90))
* **IIS 7 et versions ultérieures** : [définissez **noCompressionForHttp10** et **noCompressionForProxies** sur False dans la configuration du serveur](https://www.iis.net/configreference/system.webserver/httpcompression)

