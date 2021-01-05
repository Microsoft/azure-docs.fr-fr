---
title: Accéder aux objets blob de stockage à l’aide d’un domaine personnalisé Azure CDN via HTTPS
description: Découvrez comment ajouter un domaine personnalisé Azure CDN et activer HTTPS sur ce domaine pour votre point de terminaison de stockage Blob personnalisé.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 6061de0a330518baaa829a9a1c8a05f196d68dcb
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92777841"
---
# <a name="tutorial-access-storage-blobs-using-an-azure-cdn-custom-domain-over-https"></a>Tutoriel : Accéder aux objets blob de stockage à l’aide d’un domaine personnalisé Azure CDN via HTTPS

Une fois que vous avez intégré votre compte de stockage Azure à Azure Content Delivery Network (CDN), vous pouvez ajouter un domaine personnalisé et activer le protocole HTTPS sur ce domaine pour le point de terminaison de votre stockage d’objets blob personnalisé. 

## <a name="prerequisites"></a>Prérequis

Avant de suivre la procédure indiquée dans ce didacticiel, vous devez intégrer votre compte de stockage Azure à Azure CDN. Pour plus d’informations, consultez [Démarrage rapide : Intégrer un compte de stockage Azure à Azure CDN](cdn-create-a-storage-account-with-cdn.md).

## <a name="add-a-custom-domain"></a>Ajouter un domaine personnalisé
Lorsque vous créez un point de terminaison CDN dans votre profil, le nom du point de terminaison, qui est un sous-domaine de azureedge.net, est inclus dans l’URL de diffusion de contenu CDN par défaut. Vous avez également la possibilité d’associer un domaine personnalisé à un point de terminaison CDN. Grâce à cette option, vous distribuez votre contenu avec un domaine personnalisé dans votre URL au lieu d’un nom de point de terminaison. Pour ajouter un domaine personnalisé à votre point de terminaison, suivez les instructions de ce tutoriel : [Ajouter un domaine personnalisé à votre point de terminaison Azure CDN](cdn-map-content-to-custom-domain.md).

## <a name="configure-https"></a>Configurer le protocole HTTPS
En utilisant le protocole HTTPS sur votre domaine personnalisé, vous vous assurez que vos données sont distribuées sur internet en toute sécurité via le chiffrement TLS/SSL. Lorsque votre navigateur web est connecté à un site web via HTTPS, ce protocole valide le certificat de sécurité du site et vérifie qu’il est fourni par une autorité de certification légitime. Pour configurer le protocole HTTPS sur votre domaine personnalisé, suivez les instructions de ce tutoriel : [Configurer HTTPS sur un domaine personnalisé Azure CDN](cdn-custom-ssl.md).

## <a name="shared-access-signatures"></a>Les signatures d’accès partagé
Si le point de terminaison de votre stockage d’objets blob est configuré de sorte que l’accès en lecture anonyme est interdit, vous devez fournir un jeton [Signature d’accès partagé (SAP)](cdn-sas-storage-support.md) pour chaque requête effectuée auprès de votre domaine personnalisé. Par défaut, les points de terminaison de stockage d’objets blob interdisent l’accès en lecture anonyme. Pour en savoir plus sur SAP, consultez la section [Gestion de l’accès en lecture anonyme aux conteneurs et aux objets blob](../storage/blobs/anonymous-read-access-configure.md).

Azure CDN ignore les restrictions ajoutées au jeton SAP. Par exemple, tous les jetons SAP disposent d’un délai d’expiration. Ainsi, le contenu reste accessible avec une SAP qui a expiré tant que le contenu n’est pas vidé des serveurs de point de présence (POP) du CDN. Vous pouvez contrôler la durée de mise en cache sur Azure CDN en définissant l’en-tête de réponse du cache. Pour en savoir plus, consultez la rubrique [Gérer l’expiration des objets blob de stockage Azure CDN](cdn-manage-expiration-of-blob-content.md).

Si vous créez plusieurs URL de SAP pour le même point de terminaison d’objets blob, il convient d’activer la mise en cache de la chaîne de requête. Ce paramètre vous permet de garantir que chaque URL est traitée en tant qu’entité unique. Pour plus d’informations, consultez l’article [Contrôler le comportement de mise en cache d’Azure CDN avec des chaînes de requête](cdn-query-string.md).

## <a name="http-to-https-redirection"></a>Redirection HTTP vers HTTPS
Vous pouvez choisir de rediriger le trafic HTTP vers HTTPS en créant une règle de redirection d’URL à l’aide du [moteur de règles standard](cdn-standard-rules-engine.md) ou du [moteur de règles Verizon Premium](cdn-verizon-premium-rules-engine.md). Le moteur de règles standard est disponible uniquement pour les profils Azure CDN de Microsoft, tandis que le moteur de règles Verizon Premium n’est disponible qu’à partir des profils Azure CDN Premium de Verizon.

![Règle de redirection Microsoft](./media/cdn-storage-custom-domain-https/cdn-standard-redirect-rule.png)

Dans la règle ci-dessus, si vous conservez les valeurs Nom d’hôte, Chemin, Chaîne de requête et Fragment, les valeurs entrantes sont utilisées dans la redirection. 

![Règle de redirection Verizon](./media/cdn-storage-custom-domain-https/cdn-url-redirect-rule.png)

Dans la règle ci-dessus, *Cdn-endpoint-name* fait référence au nom que vous avez configuré pour votre point de terminaison CDN. Vous pouvez le sélectionner dans la liste déroulante. La valeur du paramètre *origin-path* désigne, au sein de votre compte de stockage d’origine, le chemin où est hébergé votre contenu statique. Si vous hébergez tout le contenu statique dans un seul conteneur, remplacez *origin-path* par le nom de ce conteneur.

## <a name="pricing-and-billing"></a>Tarification et facturation
Lorsque vous accédez à des objets blob via Azure CDN, vous payez [le prix du stockage d’objets Blob](https://azure.microsoft.com/pricing/details/storage/blobs/) pour le trafic entre les serveurs POP et l’origine (stockage d’objets Blob), et [la tarification Azure CDN](https://azure.microsoft.com/pricing/details/cdn/) pour l’accès aux données à partir des serveurs POP.

Par exemple, si vous disposez d’un compte de stockage aux États-Unis, accessible à l’aide d’Azure CDN, et qu’une personne en Europe tente d’accéder à l’un des objets blob dans ce compte de stockage via Azure CDN, Azure CDN vérifie d’abord le serveur POP européen le plus proche pour cet objet blob. S’il le trouve, Azure CDN accède à cette copie de l’objet blob et utilise la tarification CDN, car l’accès se fait via Azure CDN. Dans le cas contraire, Azure CDN copie l’objet blob vers le serveur POP, ce qui entraîne des frais de sortie et de transaction tels que spécifiés dans la tarification du stockage d’objets Blob, et accède ensuite au fichier sur le serveur POP, ce qui entraîne la facturation Azure CDN.

## <a name="next-steps"></a>Étapes suivantes
[Tutoriel : Définir des règles de mise en cache d’Azure CDN](cdn-caching-rules-tutorial.md)