---
title: Gérer les captures de paquets – Portail Azure
titleSuffix: Azure Network Watcher
description: Découvrez comment gérer la fonctionnalité de capture de paquets de Network Watcher à l’aide du portail Azure.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: damendo
ms.openlocfilehash: 099ead37b8d6b46b767ef59ff24a7e7ff9dc9e3c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966443"
---
# <a name="manage-packet-captures-with-azure-network-watcher-using-the-portal"></a>Gérer les captures de paquets avec Azure Network Watcher à l’aide du portail

La fonctionnalité de capture des paquets Network Watcher vous permet de créer des sessions de capture afin d’effectuer le suivi du trafic en direction et en provenance d’une machine virtuelle. Les filtres sont fournis pour la session de capture afin de vous garantir que vous capturez uniquement le trafic souhaité. La capture des paquets permet de diagnostiquer des anomalies du réseau de manières proactive et réactive. Elle permet aussi de collecter des statistiques réseau, d’obtenir des informations sur les intrusions, de déboguer des communications client-serveur, et bien plus. La possibilité de déclencher des captures de paquets à distance allège le fardeau de l’exécution manuelle de captures de paquets sur des machines virtuelles spécifiques, permettant ainsi d’économiser un temps précieux.

Cet article décrit comment démarrer, arrêter, télécharger et supprimer une capture de paquets. 

## <a name="before-you-begin"></a>Avant de commencer

La capture de paquets requiert la connectivité TCP sortante suivante :
- vers le compte de stockage choisi sur le port 443
- vers 169.254.169.254 sur le port 80
- vers 168.63.129.16 sur le port 8037

> [!NOTE]
> Les ports mentionnés dans les deux derniers cas ci-dessus sont communs à toutes les fonctionnalités Network Watcher qui impliquent l’extension Network Watcher et peuvent parfois changer.


Si un groupe de sécurité réseau est associé à l’interface réseau ou à un sous-réseau dans lequel figure l’interface réseau, assurez-vous qu’il existe des règles autorisant les ports ci-dessus. De même, l’ajout d’itinéraires de trafic définis par l’utilisateur à votre réseau peut empêcher la connectivité aux adresses IP et ports précités. Vérifiez qu’ils sont accessibles. 

## <a name="start-a-packet-capture"></a>Démarrer une capture de paquets

1. Dans votre navigateur, accédez au [portail Azure](https://portal.azure.com), sélectionnez **Tous les services**, puis, dans la section **Mise en réseau**, choisissez **Network Watcher**.
2. Sélectionnez **Capture de paquets** sous **Outils de diagnostic réseau**. Les captures de paquets existantes sont répertoriées, indépendamment de leur état.
3. Sélectionnez **Ajouter** pour créer une capture de paquets. Vous pouvez sélectionner des valeurs pour les propriétés suivantes :
   - **Abonnement**: abonnement dans lequel figure la machine virtuelle pour laquelle vous voulez créer la capture de paquets.
   - **Groupe de ressources** : groupe de ressources de la machine virtuelle.
   - **Machine virtuelle cible** : machine virtuelle pour laquelle vous voulez créer la capture de paquets.
   - **Nom de la capture de paquets** : nom pour la capture de paquets.
   - **Compte ou fichier de stockage** : Sélectionnez **Compte de stockage**, **Fichier**, ou les deux. Si vous sélectionnez **Fichier**, la capture est écrite dans un chemin d’accès à l’intérieur de la machine virtuelle.
   - **Chemin de fichier local** : chemin d’accès local, sur la machine virtuelle, de l’emplacement où la capture de paquets sera enregistrée (uniquement quand *Fichier* est sélectionné). Il doit s’agir d’un chemin d’accès valide. Si vous utilisez une machine virtuelle Linux, le chemin doit commencer par */var/captures*.
   - **Comptes de stockage** : si vous avez choisi *Compte de stockage*, sélectionnez un compte de stockage. Cette option est disponible uniquement si vous avez sélectionné **Stockage**.
   
     > [!NOTE]
     > Les comptes de stockage Premium ne sont actuellement pas pris en charge pour le stockage des captures de paquets.

   - **Nombre maximal d’octets par paquet** : nombre d’octets capturés dans chaque paquet. Si ce champ est vide, tous les octets sont capturés.
   - **Nombre maximal d’octets par session** : nombre total d’octets capturés. Une fois la valeur atteinte, la capture de paquets s’arrête.
   - **Délai imparti (secondes)**  : délai précédant l’arrêt de la capture de paquets. La valeur par défaut est 18 000 secondes.
   - Filtrage (facultatif). Sélectionnez **+ Ajouter un filtre**
     - **Protocole** : Protocole permettant de filtrer la capture de paquets. Valeurs possibles : TCP, UDP et Quelconque.
     - **Adresse IP locale** : filtre la capture de paquets en la limitant à ceux dont l’adresse IP locale correspond à cette valeur.
     - **Port local** : filtre la capture de paquets en la limitant à ceux dont le port local correspond à cette valeur.
     - **Adresse IP distante** : filtre la capture de paquets en la limitant à ceux dont l’adresse IP distante correspond à cette valeur.
     - **Port distant** : filtre la capture de paquets en la limitant à ceux dont le port distant correspond à cette valeur.
    
     > [!NOTE]
     > Les valeurs d’adresse IP et de port peuvent être une valeur unique, une série de valeurs ou une plage de valeurs telle que 80-1024 pour le port. Vous pouvez définir autant de filtres que nécessaire.

4. Sélectionnez **OK**.

Une fois le délai imparti défini pour la capture de paquets expiré, la capture de paquets s’arrête et peut être examinée. Vous pouvez également arrêter manuellement une session de capture de paquets.

> [!NOTE]
> Le portail effectue automatiquement les actions suivantes :
>  * Crée un observateur réseau dans la région où réside la machine virtuelle que vous avez sélectionnée, s’il n’y a pas encore d’observateur réseau dans cette région.
>  * Ajoute l’extension de machine virtuelle *AzureNetworkWatcherExtension* [Linux](../virtual-machines/extensions/network-watcher-linux.md) ou [Windows](../virtual-machines/extensions/network-watcher-windows.md) à la machine virtuelle, si elle n’est pas encore installée.

## <a name="delete-a-packet-capture"></a>Supprimer une capture de paquets

1. Dans l’affichage des captures de paquets, sélectionnez **...**  à droite d’une capture de paquets, ou cliquez avec le bouton droit sur une capture de paquets, puis sélectionnez **Supprimer**.
2. Vous êtes invité à confirmer la suppression de la capture de paquets. Sélectionnez **Oui**.

> [!NOTE]
> La suppression d’une capture de paquets n’a pas pour effet de supprimer le fichier de capture dans le compte de stockage sur la machine virtuelle.

## <a name="stop-a-packet-capture"></a>Arrêter une capture de paquets

Dans l’affichage des captures de paquets, sélectionnez **...**  à droite d’une capture de paquets, ou cliquez avec le bouton droit sur une capture de paquets, puis sélectionnez **Arrêter**.

## <a name="download-a-packet-capture"></a>Télécharger une capture de paquets

Une fois votre session de capture de paquets terminée, le fichier de capture est chargé dans le stockage d'objets blob ou dans un fichier local sur la machine virtuelle. L’emplacement de stockage de la capture de paquets est défini lors de la création de la capture de paquets. Un outil pratique pour accéder aux fichiers de capture enregistrés dans un compte de stockage est l’Explorateur Stockage Microsoft Azure que vous pouvez [télécharger](https://storageexplorer.com/).

Si un compte de stockage est spécifié, les fichiers de capture de paquets sont enregistrés dans un compte de stockage à l’emplacement suivant :

```
https://{storageAccountName}.blob.core.windows.net/network-watcher-logs/subscriptions/{subscriptionId}/resourcegroups/{storageAccountResourceGroup}/providers/microsoft.compute/virtualmachines/{VMName}/{year}/{month}/{day}/packetCapture_{creationTime}.cap
```

Si vous avez sélectionné **Fichier** lors de la création la capture, vous pouvez afficher ou télécharger le fichier à partir du chemin d’accès que vous avez configuré sur la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment automatiser les captures de paquets avec des alertes de machine virtuelle, voir [Créer une capture de paquets déclenchée par alerte](network-watcher-alert-triggered-packet-capture.md).
- Pour déterminer si un trafic entrant ou sortant spécifique est autorisé pour une machine virtuelle, voir [Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle](diagnose-vm-network-traffic-filtering-problem.md).