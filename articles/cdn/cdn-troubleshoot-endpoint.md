---
title: Résolution des problèmes des points de terminaison Azure CDN – Code de statut 404
description: Apprenez à résoudre les problèmes liés aux points de terminaison Azure Content Delivery Network (CDN) Azure qui retournent des codes d’état HTTP 404.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: b588a1eb-ab69-4fc7-ae4d-157c3e46f4a8
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: d6ad0b8b37bd4f04c22ed52d4ac6717202f22889
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88192487"
---
# <a name="troubleshooting-azure-cdn-endpoints-that-return-a-404-status-code"></a>Dépannage des points de terminaison de CDN Azure renvoyant un code d’état 404
Cet article vous permet de résoudre les problèmes liés aux points de terminaison du réseau de distribution de contenu (CDN) Azure qui retournent des codes d’état HTTP 404.

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir un support**.

## <a name="symptom"></a>Symptôme
Vous avez créé un profil et un point de terminaison de CDN, mais votre contenu ne semble pas être disponible sur le CDN. Les utilisateurs qui tentent d’accéder à votre contenu par le biais de l’URL du CDN reçoivent un code d’état HTTP 404. 

## <a name="cause"></a>Cause :
Il existe plusieurs causes possibles, y compris :

* L’origine du fichier n’est pas visible pour le CDN.
* Le point de terminaison est mal configuré et le CDN effectue donc la recherche au mauvais endroit.
* L’hôte rejette l’en-tête de l’hôte du CDN.
* Le point de terminaison n’a pas eu le temps de se propager dans le CDN.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
> [!IMPORTANT]
> Lorsqu’un point de terminaison de CDN est créé, il n’est pas disponible immédiatement, car la propagation de l’enregistrement dans le CDN peut prendre du temps :
> - Pour les profils **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de dix minutes. 
> - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
> - Dans le cas des profils du **CDN Azure Standard fourni par Verizon** et du **CDN Azure Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 90 minutes. 
> 
> Si vous suivez les étapes de ce document et que vous obtenez toujours des réponses 404, patientez quelques heures, puis vérifiez à nouveau avant d’ouvrir un ticket de support.
> 
> 

### <a name="check-the-origin-file"></a>Vérifier le fichier d’origine
Tout d’abord, vérifiez que le fichier à mettre en cache est disponible sur le serveur d’origine et qu’il est accessible publiquement sur Internet. Pour cela, le moyen le plus rapide consiste à ouvrir une session de navigateur en mode privé ou incognito, et d’accéder directement au fichier. Entrez ou collez l’URL dans la zone d’adresse, puis vérifiez que vous accédez bien au fichier attendu. Supposons que vous disposiez d’un fichier dans un compte de stockage Azure, accessible à l’adresse https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt. Si vous pouvez charger le contenu de ce fichier, le test est réussi.

![Opération réussie.](./media/cdn-troubleshoot-endpoint/cdn-origin-file.png)

> [!WARNING]
> Même s’il s’agit du moyen le plus rapide et le plus simple de vérifier que votre fichier est disponible publiquement, certaines configurations réseau de votre organisation peuvent donner l’impression qu’un fichier est accessible publiquement, alors qu’il n’est visible que pour les utilisateurs de votre réseau (même s’il est hébergé dans Azure). Pour vérifier que ce n’est pas le cas, testez le fichier avec un navigateur externe, par exemple avec un appareil mobile qui n’est pas connecté au réseau de votre organisation ou une machine virtuelle dans Azure.
> 
> 

### <a name="check-the-origin-settings"></a>Vérifier les paramètres d’origine
Maintenant que vous avez vérifié la disponibilité publique du fichier sur Internet, vérifiez les paramètres d’origine. Dans le [portail Azure](https://portal.azure.com), accédez à votre profil CDN et sélectionnez le point de terminaison que vous dépannez. Dans la page **Point de terminaison** qui apparaît, sélectionnez l’origine.  

![Page Point de terminaison avec origine mise en surbrillance](./media/cdn-troubleshoot-endpoint/cdn-endpoint.png)

La page **Origine** s’affiche. 

![Page Origine](./media/cdn-troubleshoot-endpoint/cdn-origin-settings.png)

#### <a name="origin-type-and-hostname"></a>Type et nom d’hôte de l’origine
Vérifiez que les valeurs du **type d’origine** et du **nom d’hôte d’origine** sont correctes. Dans cet exemple, https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt, la partie de nom d’hôte de l’URL est *cdndocdemo.blob.core.windows.net*, qui est correcte. Étant donné que les origines de stockage Azure, Web App et service Cloud utilisent une valeur de la liste déroulante pour le champ **Nom d’hôte de l’origine**, les erreurs orthographiques ne sont pas un problème. Toutefois, si vous utilisez une origine personnalisée, assurez-vous que le nom d’hôte est correctement orthographié.

#### <a name="http-and-https-ports"></a>Ports HTTP et HTTPS
Vérifiez vos ports **HTTP** et **HTTPS**. Dans la plupart des cas, les ports 80 et 443 sont corrects, et aucune modification n’est nécessaire.  Toutefois, si le serveur d’origine est à l’écoute sur un port différent, cela doit être représenté ici. En cas de doute, affichez l’URL de votre fichier d’origine. Les spécifications HTTP et HTTPS utilisent les ports 80 et 443 par défaut. Dans l’exemple d’URL https:\//cdndocdemo.blob.core.windows.net/publicblob/lorem.txt, aucun port n’est spécifié. La valeur par défaut 443 est donc supposée et les paramètres sont corrects.  

Toutefois, supposons que l’URL du fichier d’origine que vous avez testé précédemment est http:\//www.contoso.com:8080/file.txt. Remarquez la partie *:8080* qui termine le segment du nom d’hôte. Elle indique au navigateur d’utiliser le port 8080 pour se connecter au serveur web sur \.www.contoso.com. Vous devez donc entrer *8080* dans le champ **Port HTTP**. Il est important de noter que ces paramètres de port affectent uniquement le port utilisé par le point de terminaison pour récupérer des informations à partir de l’origine.

> [!NOTE]
> Les points de terminaison **CDN Azure Standard fourni par Akamai** n’autorisent pas la plage de ports TCP complète pour les origines.  Pour obtenir la liste des ports d’origine non autorisés, consultez l’article [Azure CDN from Akamai Allowed Origin Ports](/previous-versions/azure/mt757337(v=azure.100))(Ports d’origine autorisés du CDN Azure fourni par Akamai).  
> 
> 

### <a name="check-the-endpoint-settings"></a>Vérifier les paramètres de point de terminaison
Dans la page **Point de terminaison**, sélectionnez le bouton **Configurer**.

![Page Point de terminaison avec bouton Configurer mis en surbrillance](./media/cdn-troubleshoot-endpoint/cdn-endpoint-configure-button.png)

Le point de terminaison CDN de la page **Configurer** s’affiche.

![Page Configurer](./media/cdn-troubleshoot-endpoint/cdn-configure.png)

#### <a name="protocols"></a>Protocoles
Sous **Protocoles**, vérifiez que le protocole utilisé par les clients est sélectionné. Étant donné que le protocole utilisé par le client est celui utilisé pour accéder à l’origine, il est important que les ports d’origine soient correctement configurés dans la section précédente. Le point de terminaison CDN écoute uniquement les ports HTTP et HTTPS par défaut (80 et 443), quels que soient les ports d’origine.

Revenons à notre exemple avec http:\//www.contoso.com:8080/file.txt.  Vous savez que Contoso a spécifié *8080* comme port HTTP. Cependant, supposons qu’il a aussi spécifié *44300* comme port HTTPS.  S’il avait créé un point de terminaison nommé *contoso*, le nom d’hôte du point de terminaison de son CDN serait *contoso.azureedge.net*.  Une requête pour http:\//contoso.azureedge.net/file.txt est une requête HTTP. Le point de terminaison utilisera donc le protocole HTTP sur le port 8080 pour le récupérer à partir de l’origine.  Avec une requête sécurisée sur HTTPS, https:\//contoso.azureedge.net/file.txt, le point de terminaison utilise le protocole HTTPS sur le port 44300 lors de la récupération du fichier à partir de l’origine.

#### <a name="origin-host-header"></a>En-tête de l’hôte d’origine
Le champ **En-tête de l’hôte d’origine** indique la valeur d’en-tête d’hôte envoyée à l’origine avec chaque demande.  Dans la plupart des cas, elle doit être identique au **Nom d’hôte d’origine** que nous avons vérifié précédemment.  Généralement, une valeur incorrecte dans ce champ ne provoque pas d’état 404. Cela peut cependant provoquer d’autres états 4xx, selon ce qu’attend l’origine.

#### <a name="origin-path"></a>Chemin d’accès d’origine
Enfin, nous devons vérifier le champ **Chemin d’accès d’origine**,  qui est vide par défaut.  Vous devez l’utiliser uniquement si vous souhaitez limiter les ressources hébergées par l’origine, que vous souhaitez rendre disponibles sur le CDN.  

Dans cet exemple de point de terminaison, nous voulions que toutes les ressources du compte de stockage soient disponibles. Nous avons donc laissé le champ **Chemin d’accès d’origine** vide.  Cela signifie qu’une requête auprès de https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt entraîne une connexion entre le point de terminaison et cdndocdemo.core.windows.net qui interroge */publicblob/lorem.txt*.  De même, une requête auprès de https:\//cdndocdemo.azureedge.net/donotcache/status.png amène le point de terminaison à interroger */donotcache/status.png* à partir de l’origine.

Que se passe-t-il si vous ne voulez pas utiliser le CDN pour chaque chemin d’accès de votre origine ?  Supposons que vous vouliez uniquement exposer le chemin d’accès *publicblob*.  Si nous entrons */publicblob* dans le champ **Chemin d’accès d’origine**, le point de terminaison insère */publicblob* avant chaque requête adressée à l’origine.  Cela signifie que la requête auprès de https:\//cdndocdemo.azureedge.net/publicblob/lorem.txt prend la partie de la requête de l’URL */publicblob/lorem.txt*et la fait précéder de */publicblob*. Cela entraîne une requête auprès de */publicblob/publicblob/lorem.txt* à partir de l’origine.  Si ce chemin d’accès ne se résout pas en fichier réel, l’origine retournera un état 404.  L’URL correcte pour récupérer lorem.txt dans cet exemple serait en fait https:\//cdndocdemo.azureedge.net/lorem.txt.  Nous n’incluons pas du tout le chemin */publicblob*, car la partie requête de l’URL est */lorem.txt* et le point de terminaison ajoute */publicblob*. Par conséquent, la requête transmise à l’origine est */publicblob/lorem.txt*.

