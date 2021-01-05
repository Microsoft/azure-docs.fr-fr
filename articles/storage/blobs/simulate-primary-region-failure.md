---
title: Tutoriel - Simuler une défaillance lors de la lecture de données à partir de la région primaire
titleSuffix: Azure Storage
description: Simulez une erreur lors de la lecture de données à partir de la région primaire dans laquelle le stockage géoredondant interzone avec accès en lecture (RA-GZRS) est activé.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 04/16/2020
ms.author: tamram
ms.reviewer: artek
ms.custom: devx-track-js
ms.openlocfilehash: 5a0bef4fbfd4ee9aa720dab430a33bbfcd0b918b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91280349"
---
# <a name="tutorial-simulate-a-failure-in-reading-data-from-the-primary-region"></a>Tutoriel : Simuler une défaillance lors de la lecture de données à partir de la région primaire

Ce tutoriel est le deuxième d’une série. Il vous permet de découvrir les avantages d’un [stockage géographiquement redondant interzone avec accès en lecture](../common/storage-redundancy.md) (RA-GZRS) en simulant une défaillance.

Pour simuler une défaillance, vous pouvez utiliser un [routage statique](#simulate-a-failure-with-an-invalid-static-route) ou [Fiddler](#simulate-a-failure-with-fiddler). Les deux méthodes vous permettent de simuler l’échec des demandes d’accès au point de terminaison principal de votre compte de stockage [géographiquement redondant interzone avec accès en lecture](../common/storage-redundancy.md) (RA-GZRS), pour que l’application lise plutôt le point de terminaison secondaire.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Dans ce deuxième volet, vous apprenez à :

> [!div class="checklist"]
> * Exécuter et interrompre l’application
> * Simuler une défaillance avec un [routage statique non valide](#simulate-a-failure-with-an-invalid-static-route) ou [Fiddler](#simulate-a-failure-with-fiddler)
> * Simuler la restauration du point de terminaison principal

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, suivez le tutoriel précédent : [Rendre vos données d’application hautement disponibles avec le stockage Azure][previous-tutorial].

Pour simuler une défaillance avec un routage statique, vous allez utiliser une invite de commandes avec élévation de privilèges.

Pour simuler une défaillance à l’aide de Fiddler, téléchargez et [installez Fiddler](https://www.telerik.com/download/fiddler)

## <a name="simulate-a-failure-with-an-invalid-static-route"></a>Simuler une défaillance avec un itinéraire statique non valide

Vous pouvez créer un itinéraire statique non valide pour toutes les demandes d’accès au point de terminaison principal de votre compte de stockage RA-GZRS [(géoredondant interzone avec accès en lecture)](../common/storage-redundancy.md). Dans ce didacticiel, l’hôte local sert de passerelle pour acheminer les demandes de routage vers le compte de stockage. Le recours à l’hôte local en tant que passerelle amène toutes les demandes d’accès au point de terminaison principal de votre compte de stockage à effectuer une boucle dans l’hôte, ce qui engendre une défaillance. Suivez les étapes ci-après pour simuler une défaillance ainsi que la restauration du point de terminaison principal avec un itinéraire statique non valide.

### <a name="start-and-pause-the-application"></a>Démarrer et interrompre l’application

Suivez les instructions du [tutoriel précédent][previous-tutorial] pour lancer l’exemple et télécharger le fichier de test, en confirmant qu’il provient du stockage principal. Selon votre plateforme cible, vous pouvez ensuite manuellement mettre en pause l’exemple ou attendre l’affichage d’une invite de commandes.

### <a name="simulate-failure"></a>Simuler une défaillance

Une fois l’application mise en pause, ouvrez une invite de commandes sur Windows en tant qu’administrateur ou exécutez le terminal en tant qu’utilisateur racine sur Linux.

Pour obtenir plus d’informations sur le domaine du point de terminaison principal du compte de stockage, entrez la commande suivante dans une invite de commandes ou un terminal, en remplaçant `STORAGEACCOUNTNAME` par le nom du compte de stockage.

```
nslookup STORAGEACCOUNTNAME.blob.core.windows.net
```

Copiez l’adresse IP de votre compte de stockage dans un éditeur de texte pour une utilisation ultérieure.

Pour obtenir l’adresse IP de l’hôte local, entrez `ipconfig` dans l’invite de commandes Windows ou `ifconfig` dans le terminal Linux.

Pour ajouter un routage statique pour un hôte de destination, tapez la commande suivante sur l’invite de commandes Windows ou le terminal Linux, en remplaçant `<destination_ip>` par l’adresse IP de votre compte de stockage et `<gateway_ip>` par l’adresse de l’hôte local.

#### <a name="linux"></a>Linux

```
route add <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```
route add <destination_ip> <gateway_ip>
```

Dans la fenêtre affichant l’exemple exécuté, redémarrez l’application ou appuyez sur la touche appropriée pour télécharger l’exemple de fichier et vérifier qu’il provient du stockage secondaire. Vous pouvez ensuite remettre en pause l’exemple ou attendre l’affichage de l’invite.

### <a name="simulate-primary-endpoint-restoration"></a>Simuler la restauration du point de terminaison principal

Pour simuler le fait que le point de terminaison principal devient à nouveau fonctionnel, supprimez le routage statique non valide de la table de routage. Ainsi, toutes les demandes d’accès au point de terminaison principal peuvent être acheminées au moyen de la passerelle par défaut. Entrez la commande suivante dans une invite de commandes Windows ou un terminal Linux.

#### <a name="linux"></a>Linux

```bash
route del <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```console
route delete <destination_ip>
```

Vous pouvez ensuite redémarrer l’application ou appuyer sur la touche appropriée pour retélécharger l’exemple de fichier, cette fois en confirmant qu’il est redevenu le stockage principal.

## <a name="simulate-a-failure-with-fiddler"></a>Simuler une défaillance à l’aide de Fiddler

Pour simuler une défaillance avec Fiddler, vous injectez un échec de réponse vis-à-vis des demandes d’accès au point de terminaison principal de votre compte de stockage RA-GZRS.

Les sections suivantes décrivent la manière de simuler une défaillance ainsi que la restauration du point de terminaison principal avec Fiddler.

### <a name="launch-fiddler"></a>Lancer Fiddler

Ouvrez Fiddler, sélectionnez **Rules** (Règles) et **Customize Rules** (Personnaliser les règles).

![Personnaliser les règles de Fiddler](media/simulate-primary-region-failure/figure1.png)

Fiddler ScriptEditor se lance et affiche le fichier **SampleRules.js**. Ce fichier sert à personnaliser Fiddler.

Collez l’exemple de code suivant dans la fonction `OnBeforeResponse`, en remplaçant `STORAGEACCOUNTNAME` par le nom de votre compte de stockage. En fonction de l’exemple, vous devez également remplacer `HelloWorld` par le nom du fichier de test (ou un préfixe comme `sampleFile`) en cours de téléchargement. Le nouveau code est commenté pour qu’il ne soit pas exécuté immédiatement.

Une fois l’opération effectuée, sélectionnez **File** (Fichier) et **Save** (Enregistrer) pour enregistrer les changements apportés. Laissez la fenêtre ScriptEditor ouverte car vous allez l’utiliser dans les étapes suivantes.

```javascript
    /*
        // Simulate data center failure
        // After it is successfully downloading the blob, pause the code in the sample,
        // uncomment these lines of script, and save the script.
        // It will intercept the (probably successful) responses and send back a 503 error.
        // When you're ready to stop sending back errors, comment these lines of script out again
        //     and save the changes.

        if ((oSession.hostname == "STORAGEACCOUNTNAME.blob.core.windows.net")
            && (oSession.PathAndQuery.Contains("HelloWorld"))) {
            oSession.responseCode = 503;
        }
    */
```

![Coller la règle personnalisée](media/simulate-primary-region-failure/figure2.png)

### <a name="start-and-pause-the-application"></a>Démarrer et interrompre l’application

Suivez les instructions du [tutoriel précédent][previous-tutorial] pour lancer l’exemple et télécharger le fichier de test, en confirmant qu’il provient du stockage principal. Selon votre plateforme cible, vous pouvez ensuite manuellement mettre en pause l’exemple ou attendre l’affichage d’une invite de commandes.

### <a name="simulate-failure"></a>Simuler une défaillance

Pendant que l’application est en pause, revenez à Fiddler et décommentez la règle personnalisée que vous avez enregistrée dans la fonction `OnBeforeResponse`. Veillez à sélectionner **Fichier** et **Enregistrer** pour enregistrer vos changements afin que la règle prenne effet. Ce code recherche les demandes d’accès au compte de stockage RA-GZRS. Si le chemin contient le nom de l’exemple de fichier, il retourne le code de réponse `503 - Service Unavailable`.

Dans la fenêtre affichant l’exemple exécuté, redémarrez l’application ou appuyez sur la touche appropriée pour télécharger l’exemple de fichier et vérifier qu’il provient du stockage secondaire. Vous pouvez ensuite remettre en pause l’exemple ou attendre l’affichage de l’invite.

### <a name="simulate-primary-endpoint-restoration"></a>Simuler la restauration du point de terminaison principal

Dans Fiddler, supprimez ou commentez la règle personnalisée à nouveau. Sélectionnez **Fichier** et **Enregistrer** pour vous assurer que la règle n’est plus en vigueur.

Dans la fenêtre avec l’exemple exécuté, redémarrez l’application ou appuyez sur la touche appropriée pour télécharger l’exemple de fichier et vérifier qu’il provient à nouveau du stockage principal. Vous pouvez à présent quitter l’exemple.

## <a name="next-steps"></a>Étapes suivantes

Dans ce deuxième volet, vous avez appris à simuler une défaillance pour tester le stockage géoredondant avec accès en lecture.

Pour découvrir comment fonctionne le stockage géographiquement redondant avec accès en lecture (RA-GZRS) ainsi que les risques qui y sont liés, lisez l’article suivant :

> [!div class="nextstepaction"]
> [Conception d’applications à haute disponibilité à l’aide de RA-GZRS](../common/geo-redundant-design.md)

[previous-tutorial]: storage-create-geo-redundant-storage.md
