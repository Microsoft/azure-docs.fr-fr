---
title: Résoudre des problèmes d’expérience utilisateur dans Azure Data Factory
description: Découvrez comment résoudre des problèmes d’expérience utilisateur dans Azure Data Factory.
services: data-factory
author: ceespino
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 09/03/2020
ms.author: ceespino
ms.reviewer: daperlov
ms.openlocfilehash: 0bd0421a74679ff0c9498540d722a74ebf3d58af
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632565"
---
# <a name="troubleshoot-azure-data-factory-ux-issues"></a>Résoudre des problèmes d’expérience utilisateur dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article présente des méthodes de résolution des problèmes couramment utilisée pour l’expérience utilisateur dans Azure Data Factory.

## <a name="adf-ux-not-loading"></a>Non-chargement de l’expérience utilisateur ADF

> [!NOTE]
> L’expérience utilisateur Azure Data Factory prend officiellement en charge Microsoft Edge et Google Chrome. L’utilisation d’autres navigateurs web peut entraîner un comportement inattendu ou non documenté.

### <a name="third-party-cookies-blocked"></a>Cookies tiers bloqués

L’expérience utilisateur ADF utilise des cookies de navigateurs pour conserver une session utilisateur et permettre des expériences de développement et de supervision interactives. Il est possible que votre navigateur bloque les cookies tiers, parce que vous utilisez une session incognito ou avez activé un bloqueur de publicités. Le blocage des cookies tiers peut provoquer des problèmes lors du chargement du portail, comme la redirection vers une page vierge, https://adf.azure.com/accesstoken.html, ou l’obtention d’un message d’avertissement indiquant que les cookies tiers sont bloqués. Pour résoudre ce problème, activez les options de cookies tiers sur votre navigateur en procédant comme suit :

### <a name="google-chrome"></a>Google Chrome

#### <a name="allow-all-cookies"></a>Autoriser tous les cookies

1. Visitez **chrome://settings/cookies** dans votre navigateur.
1. Sélectionner l’option **Autoriser tous les cookies** 

    ![Autoriser tous les cookies dans Chrome](media/data-factory-ux-troubleshoot-guide/chrome-allow-all-cookies.png)
1. Actualisez l’expérience utilisateur ADF, puis réessayez.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Autoriser uniquement l’expérience utilisateur ADF à utiliser des cookies
Si vous ne souhaitez pas autoriser tous les cookies, vous pouvez n’autoriser que l’expérience utilisateur ADF :
1. Visitez **chrome://settings/cookies** .
1. Sélectionner **ajouter** sous l’option **Sites qui peuvent toujours utiliser des cookies** 

    ![Ajouter l’expérience utilisateur ADF aux sites autorisés dans Chrome](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-1.png)
1. Ajoutez le site **adf.azure.com** , cochez l’option **tous les cookies** , puis enregistrez. 

    ![Autoriser tous les cookies du site d’expérience utilisateur ADF](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-2.png)
1. Actualisez l’expérience utilisateur ADF, puis réessayez.

### <a name="microsoft-edge"></a>Microsoft Edge

1. Visitez **edge://settings/content/cookies** dans votre navigateur.
1. Assurez-vous que l’option **Autoriser les sites à enregistrer et à lire les données de cookie** est activée, et que l’option **Bloquer les cookies tiers** est désactivée 

    ![Autoriser tous les cookies dans Edge](media/data-factory-ux-troubleshoot-guide/edge-allow-all-cookies.png)
1. Actualisez l’expérience utilisateur ADF, puis réessayez.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Autoriser uniquement l’expérience utilisateur ADF à utiliser des cookies

Si vous ne souhaitez pas autoriser tous les cookies, vous pouvez n’autoriser que l’expérience utilisateur ADF :

1. Visitez **edge://settings/content/cookies** .
1. Sous la section **Autoriser** , sélectionnez **Ajouter** , puis ajoutez le site **adf.azure.com** . 

    ![Ajouter l’expérience utilisateur ADF aux sites autorisés dans Edge](media/data-factory-ux-troubleshoot-guide/edge-allow-adf-cookies.png)
1. Actualisez l’expérience utilisateur ADF, puis réessayez.

## <a name="connection-failed-on-adf-ux"></a>Échec de la connexion dans l’expérience utilisateur ADF

Parfois, dans l’expérience utilisateur ADF, vous pouvez voir des erreurs « Échec de la connexion » similaires à la capture d’écran ci-dessous après avoir cliqué sur **Tester la connexion** , **Aperçu** , etc.

![Échec de la connexion](media/data-factory-ux-troubleshoot-guide/connection-failed.png)

Dans ce cas, vous pouvez d’abord essayer la même opération avec le mode de navigation InPrivate dans votre navigateur.

Si cela ne fonctionne toujours pas, dans le navigateur, appuyez sur F12 pour ouvrir **Outils de développement** . Accédez à l’onglet **Réseau** , cochez **Désactiver le cache** , recommencez l’opération qui a échoué et recherchez la requête ayant échoué (en rouge).

![Requête ayant échoué](media/data-factory-ux-troubleshoot-guide/failed-request.png)

Recherchez ensuite le **nom d’hôte** (dans ce cas, **dpnortheurope.svc.datafactory.azure.com** ) à partir de **l’URL de requête** de la requête ayant échoué.

Tapez le **nom d’hôte** directement dans la barre d’adresses de votre navigateur. Si la mention 404 s’affiche dans le navigateur, cela signifie généralement que tout est correct côté client et que le problème se trouve au niveau du service ADF. Envoyez une demande de ticket de support avec **l’ID d’activité** provenant du message d’erreur dans l’expérience utilisateur ADF.

![Ressource introuvable](media/data-factory-ux-troubleshoot-guide/status-code-404.png)

Si vous voyez une erreur similaire à celle ci-dessous dans le navigateur, cela signifie généralement qu’il y a un problème côté client. Suivez les étapes de dépannage.

![Erreur côté client](media/data-factory-ux-troubleshoot-guide/client-side-error.png)

Ouvrez **l’invite de commandes** et tapez **nslookup dpnortheurope.svc.datafactory.azure.com** . Une réponse normale doit ressembler à ce qui suit :

![Réponse de la commande 1](media/data-factory-ux-troubleshoot-guide/command-response-1.png)

Si vous voyez une réponse DNS normale, contactez votre support informatique local pour vérifier les paramètres de pare-feu afin de savoir si la connexion HTTPS à ce nom d’hôte est bloquée ou non. Si le problème ne peut pas être résolu, envoyez une demande de ticket de support avec **l’ID d’activité** provenant du message d’erreur dans l’expérience utilisateur ADF.

Si autre chose s’affiche, cela signifie généralement qu’il y a un problème avec votre serveur DNS lors de la résolution du nom DNS. En règle générale, la modification du fournisseur de services Internet (ISP) ou du DNS (par exemple, sur Google DNS 8.8.8.8) peut être une solution de contournement possible. Si le problème persiste, vous pouvez essayer **nslookup datafactory.azure.com** et **nslookup azure.com** pour voir à quel niveau votre résolution DNS a échoué et envoyer toutes les informations à votre support informatique local ou à votre fournisseur de services Internet pour résoudre le problème. S’ils pensent que le problème se trouve toujours du côté de Microsoft, envoyez une demande de ticket de support avec **l’ID d’activité** provenant du message d’erreur dans l’expérience utilisateur ADF.

![Réponse de la commande 2](media/data-factory-ux-troubleshoot-guide/command-response-2.png)

## <a name="change-linked-service-type-in-datasets"></a>Modifier le type de service lié dans les jeux de données

Le jeu de données de format de fichier peut être utilisé avec tous les connecteurs basés sur des fichiers. Par exemple, vous pouvez configurer un jeu de données Parquet sur Azure Blob ou Azure Data Lake Storage Gen2. Notez que chaque connecteur prend en charge différents jeux de paramètres liés au magasin de données sur l’activité, et avec un modèle d’application différent. 

Dans l’interface utilisateur de création ADF, lorsque vous utilisez un jeu de données de format de fichier dans une activité (notamment les activités Copy, Lookup, GetMetadata, Delete) et que dans le jeu de données, vous souhaitez pointer vers un service lié de type différent de celui de la version actuelle (par exemple, passer du système de fichiers à ADLS Gen2), le message d’avertissement suivant s’affiche. Pour vous assurer qu’il s’agit d’un commutateur approprié, à la suite de votre consentement, les pipelines et les activités qui font référence à ce jeu de données sont modifiés pour utiliser également le nouveau type, et tous les paramètres de magasin de données existants incompatibles avec le nouveau type sont effacés car ils ne s’appliquent plus.

Pour en savoir plus sur les paramètres de stockage de données pris en charge pour chaque connecteur, vous pouvez accéder à l’article du connecteur correspondant-> propriétés de l’activité de copie pour afficher la liste détaillée des propriétés. Consultez [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [Système de fichiers](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) et [SFTP](connector-sftp.md).

![Message d'avertissement](media/data-factory-ux-troubleshoot-guide/warning-message.png)

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire, essayez les ressources suivantes :

* [Blog Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
* [Demandes de fonctionnalités Data Factory](https://feedback.azure.com/forums/270578-data-factory)
* [Forum Stack Overflow pour Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
* [Informations Twitter sur Data Factory](https://twitter.com/hashtag/DataFactory)
* [Vidéos Azure](https://azure.microsoft.com/resources/videos/index/)
* [Page de questions Microsoft Q&R](/answers/topics/azure-data-factory.html)