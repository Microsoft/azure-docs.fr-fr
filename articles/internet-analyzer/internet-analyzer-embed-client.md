---
title: Incorporer le client Internet Analyzer | Microsoft Docs
description: Dans cet article, découvrez comment incorporer le client Internet Analyzer JavaScript dans votre application.
services: internet-analyzer
author: mattcalder
ms.service: internet-analyzer
ms.topic: how-to
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 0d4b27b85ac7bc61e14a79f29e4e26ec4973ced1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84744049"
---
# <a name="embed-the-internet-analyzer-client"></a>Incorporer le client Internet Analyzer

Cet article explique comment incorporer le client JavaScript dans votre application. L’installation de ce client est nécessaire pour exécuter des tests et recevoir des analyses de tableaux de bord. **Le client JavaScript spécifique au profil est fourni après la configuration du premier test.** À partir de là, vous pouvez continuer à ajouter ou supprimer des tests dans ce profil sans avoir à incorporer de nouveaux scripts. Pour plus d’informations sur Internet Analyzer, consultez la [vue d’ensemble](internet-analyzer-overview.md). 

> [!IMPORTANT]
> Cette préversion publique est fournie sans contrat de niveau de service et ne doit pas être utilisée pour les charges de travail de production. Certaines fonctionnalités peuvent ne pas être prises en charge, disposer de capacités limitées ou ne pas être disponibles dans tous les emplacements Azure. Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Avant de commencer

Internet Analyzer requiert l’accès à Azure et à d’autres services Microsoft pour fonctionner correctement. Autorisez l’accès réseau à `fpc.msedge.net` et à toutes les URL de point de terminaison préconfigurées (visibles via [CLI](internet-analyzer-cli.md)) avant d’incorporer le client.

## <a name="find-the-client-script-url"></a>Rechercher l’URL du script client

L’URL du script est accessible via le Portail Azure ou Azure CLI après la configuration d’un test. Pour en savoir plus, consultez [Create an Internet Analyzer Resource](internet-analyzer-create-test-portal.md) (Créer une ressource Internet Analyzer).

Option 1. Dans le Portail Azure, utilisez [ce lien](https://aka.ms/InternetAnalyzerPreviewPortal) pour ouvrir la page du portail en préversion pour Azure Internet Analyzer. Accédez à votre profil Internet Analyzer pour afficher l’URL du script en vous rendant dans **Paramètres > Configuration**.

Option 2. À l’aide d’Azure CLI, vérifiez la propriété `scriptFileUri`.
```azurecli-interactive
    az extension add --name internet-analyzer    
    az internet-analyzer test list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
```

## <a name="client-details"></a>Détails du client

Le script est généré spécifiquement pour votre profil et vos tests. Une fois chargé, le script s’exécute après un délai de 2 secondes. Tout d’abord, il contacte le service Internet Analyzer pour extraire la liste des points de terminaison configurés dans vos tests. Il exécute ensuite les mesures et charge à nouveau les résultats chronométrés dans le service Internet Analyzer.

## <a name="client-examples"></a>Exemples de clients

Ces exemples illustrent quelques méthodes de base pour incorporer le client JavaScript dans votre page web ou application. Nous utilisons `0bfcb32638b44927935b9df86dcfe397` comme exemple d’ID de profil pour l’URL de script.

### <a name="run-on-page-load"></a>Exécuter au chargement de la page
La méthode la plus simple consiste à utiliser la balise de script dans le bloc de balises meta. Cette balise va exécuter le script une fois par chargement de page.

```html
<html>
<meta>
    <script src="//fpc.msedge.net/client/v2/0bfcb32638b44927935b9df86dcfe397/ab.min.js"></script>
</meta>
<body></body>
</html>
```

## <a name="next-steps"></a>Étapes suivantes

Consultez la [FAQ sur Internet Analyzer](internet-analyzer-faq.md)
