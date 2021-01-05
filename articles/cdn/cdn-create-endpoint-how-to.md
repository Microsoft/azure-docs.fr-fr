---
title: Créer un point de terminaison CDN Azure | Microsoft Docs
description: Cet article explique comment créer un point de terminaison Azure Content Delivery Network (CDN), y compris les paramètres avancés.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/12/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 43718f8ebc851f27035f2999bfb4ff3ec12ca5b4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021960"
---
# <a name="create-an-azure-cdn-endpoint"></a>Créer un point de terminaison CDN Azure
Cet article décrit tous les paramètres de création d’un point de terminaison [Azure Content Delivery Network (CDN)](cdn-overview.md) dans un profil CDN existant. Après avoir créé un profil et un point de terminaison, vous pouvez commencer à distribuer du contenu à vos clients. Pour créer rapidement un profil et un point de terminaison, consultez [Démarrage rapide : Créer un point de terminaison et un profil de réseau de distribution de contenu Azure](cdn-create-new-endpoint.md).

## <a name="prerequisites"></a>Prérequis
Avant de créer un point de terminaison CDN, vous devez avoir créé au moins un profil CDN, qui peut contenir un ou plusieurs points de terminaison CDN. Vous pouvez utiliser plusieurs profils pour organiser vos points de terminaison CDN par domaine Internet, application web ou d’autres critères. La tarification CDN s’applique au niveau du profil CDN. Vous devez donc créer plusieurs profils CDN si vous souhaitez utiliser plusieurs niveaux tarifaires Azure CDN. Pour créer un profil CDN, consultez [Créer un profil CDN](cdn-create-new-endpoint.md#create-a-new-cdn-profile).

## <a name="log-in-to-the-azure-portal"></a>Se connecter au portail Azure.
Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

## <a name="create-a-new-cdn-endpoint"></a>Créer un point de terminaison CDN

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre profil CDN. Vous l'avez peut-être épinglé au tableau de bord à l'étape précédente. Sinon, vous pouvez le trouver en sélectionnant **Tous les services**, puis **Profils CDN**. Dans le volet **Profils CDN**, sélectionnez le profil auquel vous avez prévu d’ajouter votre point de terminaison. 
   
    Le volet du profil CDN s’affiche.

2. Sélectionnez **Point de terminaison**.
   
    ![Sélection d’un point de terminaison CDN](./media/cdn-create-endpoint-how-to/cdn-select-endpoint.png)
   
    La page **Ajouter un point de terminaison** s’affiche.
   
    ![Page Ajouter un point de terminaison](./media/cdn-create-endpoint-how-to/cdn-add-endpoint-page.png)

3. Comme **Nom**, entrez un nom unique pour le nouveau point de terminaison CDN. Ce nom sert à accéder à vos ressources en cache au niveau du domaine _\<endpointname>_ .azureedge.net.

4. Pour **Type d’origine**, choisissez l’un des types d’origine suivants : 
   - **Stockage** pour Stockage Azure
   - **Service cloud** pour Azure Cloud Services
   - **Application web** pour Azure Web Apps
   - **Origine personnalisée** pour n’importe quel autre serveur web d’origine publiquement accessible (hébergé sur Azure ou ailleurs)

5. Pour **Nom d’hôte de l’origine**, sélectionnez ou entrez le domaine du serveur d’origine. La liste déroulante répertorie tous les serveurs d’origine disponibles pour le type spécifié à l’étape 4. Si vous avez sélectionné **Origine personnalisée** comme type d’origine, saisissez le domaine de votre serveur d’origine personnalisé.
    
6. Pour **Chemin d’accès d’origine**, entrez le chemin d’accès aux ressources que vous souhaitez mettre en cache. Pour permettre la mise en cache de n’importe quelle ressource au niveau du domaine que vous avez spécifié à l’étape 5, laissez ce paramètre vide.
    
7. Pour **En-tête de l’hôte d’origine**, entrez l’en-tête de l’hôte que le CDN doit envoyer avec chaque requête ou conservez la valeur par défaut.
   
   > [!NOTE]
   > Certains types d’origine, tels que Azure Storage et Web Apps, nécessitent l’en-tête de l’hôte pour correspondre au domaine de l’origine. À moins d’avoir une origine nécessitant un en-tête d’hôte différent de son domaine, vous devriez laisser la valeur par défaut.
   > 
    
8. Pour **Protocole** et **Port de l’origine**, spécifiez les protocoles et les ports utilisés pour accéder à vos ressources au niveau du serveur d’origine. Vous devez sélectionner au moins un protocole (HTTP ou HTTPS). Utilisez le domaine fourni par CDN ( _\<endpointname>_ .azureedge.net) pour accéder au contenu HTTPS. 
   
   > [!NOTE]
   > La valeur **Port de l’origine** ne concerne que le port utilisé par le point de terminaison pour récupérer des informations à partir du serveur d’origine. Le point de terminaison est uniquement disponible pour les clients sur les ports HTTP et HTTPS par défaut (80 et 443), quel que soit la valeur **port d’origine**.  
   > 
   > Les points de terminaison **CDN Azure fournis par Akamai** n’autorisent pas la plage de ports TCP complète pour les ports d’origine. Pour obtenir la liste des ports d’origine non autorisés, consultez l’article [Azure CDN from Akamai Allowed Origin Ports](/previous-versions/azure/mt757337(v=azure.100))(Ports d’origine autorisés du CDN Azure fourni par Akamai).  
   > 
   > HTTPS pour les domaines personnalisés Azure CDN n’est pas pris en charge sur les produits **Azure CDN fourni par Akamai**. Pour plus d’informations, consultez [Activer ou désactiver HTTPS sur un domaine personnalisé Azure CDN](cdn-custom-ssl.md).
    
9. Pour **Optimisé pour**, sélectionnez le type d’optimisation qui correspond le mieux au scénario et au type de contenu que le point de terminaison doit distribuer. Pour plus d’informations, consultez [Optimiser Azure CDN pour le type de distribution de contenu](cdn-optimization-overview.md).

    Les paramètres de type d’optimisation suivants sont pris en charge, en fonction du type de profil :
    - Profils **CDN Azure Standard fourni par Microsoft** :
       - [**Livraison web générale**](cdn-optimization-overview.md#general-web-delivery)

    - Profils **CDN Azure Standard fourni par Verizon** et **CDN Azure Premium fourni par Verizon** :
       - [**Livraison web générale**](cdn-optimization-overview.md#general-web-delivery)
       - [**Accélération de site dynamique**](cdn-optimization-overview.md#dynamic-site-acceleration)

    - Profils **CDN Azure Standard fourni par Akamai** :
       - [**Livraison web générale**](cdn-optimization-overview.md#general-web-delivery)
       - [**Diffusion multimédia en continu générale**](cdn-optimization-overview.md#general-media-streaming)
       - [**Diffusion multimédia en continu de vidéos à la demande**](cdn-optimization-overview.md#video-on-demand-media-streaming)
       - [**Téléchargement de fichiers volumineux**](cdn-optimization-overview.md#large-file-download)
       - [**Accélération de site dynamique**](cdn-optimization-overview.md#dynamic-site-acceleration)

10. Sélectionnez **Ajouter** pour créer un point de terminaison.
   
    Une fois le point de terminaison créé, il s'affiche dans la liste des points de terminaison pour le profil.
    
    ![Point de terminaison CDN](./media/cdn-create-new-endpoint/cdn-endpoint-success.png)
    
    Le point de terminaison n’est pas disponible immédiatement, car la propagation de l’enregistrement peut prendre du temps : 
    - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
    - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
    - Dans le cas des profils du **CDN Azure Standard fourni par Verizon** et du **CDN Azure Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 90 minutes. 
   
    Si vous tentez d’utiliser le nom de domaine CDN avant la propagation de la configuration du point de terminaison aux serveurs de point de présence (POP), vous recevrez probablement un état de réponse HTTP 404. Si plusieurs heures se sont écoulées depuis la création de votre point de terminaison et si vous recevez toujours un état de réponse 404, consultez [Dépannage des points de terminaison de CDN Azure renvoyant un code d’état 404](cdn-troubleshoot-endpoint.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer un point de terminaison qui n’est plus nécessaire, sélectionnez-le, puis sélectionnez **Supprimer**. 

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les domaines personnalisés, passez au didacticiel consacré à l’ajout d’un domaine personnalisé à votre point de terminaison CDN.

> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](cdn-map-content-to-custom-domain.md)


