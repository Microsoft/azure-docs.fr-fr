---
title: Compression des fichiers dans Azure CDN pour améliorer les performances | Microsoft Docs
description: Découvrez comment améliorer la vitesse de transfert de fichiers et les performances de chargement de page en compressant vos fichiers dans Azure CDN.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: af1cddff-78d8-476b-a9d0-8c2164e4de5d
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/28/2018
ms.author: allensu
ms.openlocfilehash: ceed62d466627d6a23554229bd6f4b96c674c7e9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993667"
---
# <a name="improve-performance-by-compressing-files-in-azure-cdn"></a>Compression des fichiers dans Azure CDN pour améliorer les performances
La compression de fichiers est une méthode simple et efficace qui vise à améliorer la vitesse de transfert des fichiers et à accroître les performances de chargement des pages en réduisant la taille des fichiers avant leur envoi à partir du serveur. La compression de fichiers peut réduire les coûts de bande passante et offre à vos utilisateurs davantage de réactivité.

Il existe deux façons d’activer la compression de fichiers :

- Activer la compression sur votre serveur d’origine. Dans ce cas, Azure CDN fait transiter les fichiers compressés et les remet aux clients qui les demandent.
- Activez la compression directement sur les serveurs POP CDN (*compression à la volée*). Dans ce cas, le CDN compresse les fichiers et les sert aux utilisateurs finals, même s’ils ne sont pas compressés par le serveur d’origine.

> [!IMPORTANT]
> La propagation sur le réseau des changements de configuration d’Azure CDN peut prendre un certain temps : 
> - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
> - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
> - Dans le cas des profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 10 minutes. 
> 
> Si vous configurez la compression de votre point de terminaison CDN pour la première fois, patientez 1 à 2 heures avant de chercher à résoudre un problème, pour être certain que les paramètres de compression ont été propagés aux POP.

## <a name="enabling-compression"></a>Activation de la compression
Les niveaux Standard et Premium de CDN fournissent les mêmes fonctionnalités, mais l’interface utilisateur est différente. Pour plus d’informations sur les différences entre les niveaux Standard et Premium de CDN, consultez [Vue d’ensemble du réseau de distribution de contenu (CDN) Azure](cdn-overview.md).

### <a name="standard-cdn-profiles"></a>Profils CDN Standard 
> [!NOTE]
> Cette section s’applique aux profils **Azure CDN Standard fourni par Microsoft**, **Azure CDN Standard fourni par Verizon** et **Azure CDN Standard fourni par Akamai**.
> 
> 

1. Depuis la page de profil CDN, sélectionnez le point de terminaison CDN que vous souhaitez gérer.

    ![Points de terminaison du profil CDN](./media/cdn-file-compression/cdn-endpoints.png)

    La page du point de terminaison CDN s’ouvre.
2. Sélectionnez **Compression**.

    ![Capture d’écran représentant un point de terminaison avec l’option Compression sélectionnée dans le menu du portail.](./media/cdn-file-compression/cdn-compress-select-std.png)

    La page de compression s’ouvre.
3. Sélectionnez **Activer** pour activer la compression.

    ![Capture d’écran représentant l’activation de Compression.](./media/cdn-file-compression/cdn-compress-standard.png)
4. Utilisez les types MIME par défaut ou modifiez la liste en supprimant ou en ajoutant des types MIME.

   > [!TIP]
   > Bien que cela soit possible, il n’est pas recommandé d’appliquer la compression aux formats compressés. Par exemple, ZIP, MP3, MP4 ou JPG.
   > 

   > [!NOTE]
   > La modification de la liste par défaut des types MIME n’est actuellement pas prise en charge dans Azure CDN Standard fourni par Microsoft.
   > 

5. Une fois vos modifications effectuées, sélectionnez **Enregistrer**.

### <a name="premium-cdn-profiles"></a>Profils CDN Premium
> [!NOTE]
> Cette section s’applique uniquement aux profils du **CDN Azure CDN Premium fourni par Verizon**.
> 

1. Dans la page de profil CDN, sélectionnez **Gérer**.

    ![Sélectionner Gestion du CDN](./media/cdn-file-compression/cdn-manage-btn.png)

    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **HTTP volumineux**, puis pointez sur le menu volant **Paramètres de cache**. Sélectionnez **Compression**.

    ![Sélection de fichiers CDN](./media/cdn-file-compression/cdn-compress-select.png)

    Les options de compression sont affichées.

    ![Options de compression de fichiers CDN](./media/cdn-file-compression/cdn-compress-files.png)
3. Activez la compression en sélectionnant **Compression activée**. Entrez les types MIME que vous souhaitez compresser sous forme de liste délimitée par des virgules (sans espace) dans la zone **Types de fichiers**.

   > [!TIP]
   > Bien que cela soit possible, il n’est pas recommandé d’appliquer la compression aux formats compressés. Par exemple, ZIP, MP3, MP4 ou JPG.
   > 

4. Après avoir apporté vos modifications, sélectionnez **Mettre à jour**.

## <a name="compression-rules"></a>Règles de compression

### <a name="azure-cdn-standard-from-microsoft-profiles"></a>Profils CDN Azure Standard fourni par Microsoft

Pour les profils du **CDN Azure Standard fourni par Microsoft**, seuls les fichiers éligibles sont compressés. Pour être éligible pour la compression, un fichier doit :
- Être d’un type MIME [configuré pour la compression](#enabling-compression).
- Être supérieur à 1 Ko
- Être inférieur à 8 Mo

Ces profils prennent en charge les encodages de compression suivants :
- gzip (GNU zip)
- brotli 

Si la requête prend en charge plusieurs types de compression, la compression brotli est prioritaire.

Lorsqu’une demande liée à un composant spécifie la compression gzip et que cette demande cause une absence dans le cache, le CDN Azure effectue une compression gzip du composant directement sur le serveur POP. Ensuite, le fichier compressé est servi à partir du cache.

Si l’origine utilise l’encodage de transfert mémorisé en bloc (CTE) pour envoyer des données compressées au CDN POP, les réponses supérieures à 8 Mo ne sont pas prises en charge. 

### <a name="azure-cdn-from-verizon-profiles"></a>Profils CDN Azure fourni par Verizon

Pour les profils **CDN Azure Standard fourni par Verizon** et **CDN Azure Premium fourni par Verizon**, seuls les fichiers éligibles sont compressés. Pour être éligible pour la compression, un fichier doit :
- Être supérieur à 128 octets
- Être inférieur à 3 Mo

Ces profils prennent en charge les encodages de compression suivants :
- gzip (GNU zip)
- DEFLATE
- bzip2
- brotli 

Si la requête prend en charge plusieurs types de compression, ces types de compression sont prioritaires sur la compression brotli.

Lorsqu’une demande liée à un composant spécifie la compression brotli (en-tête HTTP `Accept-Encoding: br`) et que cette demande cause une absence dans le cache, le CDN Azure effectue une compression brotli du composant directement sur le serveur POP. Ensuite, le fichier compressé est servi à partir du cache.

### <a name="azure-cdn-standard-from-akamai-profiles"></a>Profils CDN Azure Standard fourni par Akamai

Pour les profils **CDN Azure Standard fourni par Akamai**, tous les fichiers sont éligibles à la compression. Cependant, un fichier doit être de type MIME et [configuré pour la compression](#enabling-compression).

Ces profils prennent en charge uniquement l’encodage par compression gzip. Lorsqu’un point de terminaison de profil demande des fichiers encodés gzip, ils sont toujours demandés en provenance de l’origine, quelle que soit la requête du client. 

## <a name="compression-behavior-tables"></a>Tables de comportement de compression
Les tableaux suivants décrivent le comportement de compression du CDN Azure pour chaque scénario :

### <a name="compression-is-disabled-or-file-is-ineligible-for-compression"></a>La compression est désactivée ou le fichier n’est pas éligible pour la compression
| Format demandé par le client (via l’en-tête Accept-Encoding) | Format de fichier mis en cache | La réponse du CDN au client | Notes&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
| --- | --- | --- | --- |
| Compressé |Compressé |Compressé | |
| Compressé |Non compressé |Non compressé | |
| Compressé |Non mis en cache |Compressés ou non compressé |La réponse d’origine détermine si le CDN effectue une compression. |
| Non compressé |Compressé |Non compressé | |
| Non compressé |Non compressé |Non compressé | |
| Non compressé |Non mis en cache |Non compressé | |

### <a name="compression-is-enabled-and-file-is-eligible-for-compression"></a>La compression est activée et le fichier est éligible pour la compression
| Format demandé par le client (via l’en-tête Accept-Encoding) | Format de fichier mis en cache | Réponse du CDN au client | Notes |
| --- | --- | --- | --- |
| Compressé |Compressé |Compressé |Transcode CDN entre les formats pris en charge. |
| Compressé |Non compressé |Compressé |CDN effectue une compression. |
| Compressé |Non mis en cache |Compressé |CDN effectue une compression si l’origine renvoie un fichier non compressé. <br/>**Azure CDN fourni par Verizon** transmet le fichier non compressé à la première demande, puis compresse et met en cache le fichier pour les demandes suivantes. <br/>Les fichiers avec l’en-tête `Cache-Control: no-cache` ne sont jamais compressés. |
| Non compressé |Compressé |Non compressé |CDN effectue la décompression. |
| Non compressé |Non compressé |Non compressé | |
| Non compressé |Non mis en cache |Non compressé | |

## <a name="media-services-cdn-compression"></a>Compression CDN Media Services
Pour les points de terminaison activés pour la diffusion en continu CDN de Media Services, la compression est activée par défaut pour les types MIME suivants : 
- application/vnd.ms-sstr+xml 
- application/dash+xml
- application/vnd.apple.mpegurl
- application/f4m+xml 

## <a name="see-also"></a>Voir aussi
* [Résolution des problèmes de compression des fichiers CDN](cdn-troubleshoot-compression.md)    

