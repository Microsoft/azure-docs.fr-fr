---
title: Résoudre les problèmes liés à Synapse Studio
description: Résoudre les problèmes liés à Azure Synapse Studio
author: julieMSFT
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: a96709c7538bda6d6b338c29dc512902e9ca2c87
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445849"
---
# <a name="azure-synapse-studio-troubleshooting"></a>Résolution des problèmes liés à Azure Synapse Studio

Ce guide de résolution des problèmes fournit des instructions sur les informations à fournir lors de l’ouverture d’un ticket de support concernant des problèmes de connectivité réseau. Avec les informations appropriées, nous pouvons peut-être résoudre le problème plus rapidement.

## <a name="serverless-sql-pool-service-connectivity-issue"></a>Problème de connectivité avec le service Pool SQL serverless

### <a name="symptom-1"></a>Symptôme 1

L’option « Pool SQL serverless » est grisée dans la liste déroulante « Se connecter à ».

![Symptôme 1](media/troubleshooting-synapse-studio/symptom1v2.png)

### <a name="symptom-2"></a>Symptôme 2

L’exécution de la requête contenant « Pool SQL serverless » génère le message d’erreur « Échec de l’établissement de la connexion au serveur ».

![Symptôme 2](media/troubleshooting-synapse-studio/symptom2.png)
 

## <a name="troubleshooting-steps"></a>Étapes de dépannage

> [!NOTE] 
>    Les étapes de résolution des problèmes suivantes sont destinées à Chromium Edge et Chrome. Vous pouvez utiliser d’autres navigateurs (tels que FireFox) avec les mêmes étapes de résolution des problèmes, mais la disposition de la fenêtre « Outil développeur » peut différer des captures d’écran de ce guide de résolution des problèmes. Autant que possible, N’UTILISEZ PAS la version classique de Edge pour la résolution des problèmes, car elle peut afficher des informations inexactes dans certaines situations.

Ouvrez le panneau « Informations de diagnostic », puis sélectionnez le bouton « Télécharger le diagnostic ». Conservez les informations téléchargées pour le signalement d’erreurs. Au lieu de cela, vous pouvez copier l’« ID de session » et l’attacher lors de l’ouverture du ticket de support.

![infos-diagnostic](media/troubleshooting-synapse-studio/diagnostic-info-download.png)

Pour commencer la résolution des problèmes, réessayez l’opération que vous avez effectuée dans Azure Synapse Studio.

- Pour le symptôme 1, sous l’onglet « Script SQL », sélectionnez le bouton « Actualiser » à droite de la liste déroulante « Utiliser une base de données », et vérifiez si vous pouvez voir « pool SQL serverless ».
- Pour le symptôme 2, essayez de réexécuter la requête pour voir si elle fonctionne correctement.

Si le problème persiste, appuyez sur F12 dans votre navigateur pour ouvrir « Outils de développement » (DevTools).

Dans la fenêtre « Outils de développement », basculez vers le panneau « Réseau ». Sélectionnez le bouton « Effacer » dans la barre d’outils du panneau « Réseau », si nécessaire.
Assurez-vous que l’option « Désactiver le cache » dans le panneau « Réseau » est activée.

Réessayez l’opération que vous avez effectuée dans Azure Synapse Studio. Vous pouvez voir les nouveaux éléments affichés dans la liste « Réseau » dans « Outils de développement ». Notez votre heure système actuelle à fournir dans le ticket de support.

![panneau-réseau 1](media/troubleshooting-synapse-studio/network-panel.png)

Recherchez l’élément dont la colonne URL correspond au modèle suivant :

`https://[*A*]-ondemand.database.windows.net:1443/databases/[*B*]/query?api-version=2018-08-01-preview&application=ArcadiaSqlOnDemandExplorer`

Où [*A*] est le nom de votre espace de travail, « -ondemand » pourrait être « -sqlod », et où [*B*] doit être un nom de base de données tel que « master ». Il doit y avoir au plus deux éléments avec la même valeur d’URL, mais des valeurs de méthode différentes : OPTIONS et POST. Vérifiez si ces deux éléments contiennent « 200 » ou « 20x » dans la colonne d’état, où « x » peut être n’importe quel chiffre unique.

Si l’un d’eux autre chose que « 20x », et que :

- l’état commence par « (failed) », élargissez la colonne « État » ou pointez sur le texte de l’état pour le voir en entier. Incluez le texte et/ou la capture d’écran lors de l’ouverture du ticket de support.

    ![texte d’état](media/troubleshooting-synapse-studio/status-text.png)

    - Si vous voyez ERR_NAME_NOT_RESOLVED et avez créé votre espace de travail en 10 minutes, patientez 10 minutes, puis réessayez pour voir si le problème persiste.
    - Si vous voyez ERR_INTERNET_DISCONNECTED ou ERR_NETWORK_CHANGED, cela peut indiquer que la connexion réseau de votre PC rencontre des problèmes. Vérifiez votre connexion réseau, puis retentez l’opération.
    - Si vous voyez ERR_CONNECTION_RESET, ERR_SSL_PROTOCOL_ERROR ou d’autres codes d’erreur contenant « SSL », cela peut indiquer que votre configuration SSL locale rencontre des problèmes ou que votre administrateur réseau a bloqué l’accès au serveur du pool SQL serverless. Ouvrez un ticket de support et joignez le code d’erreur dans la description.
    - Si vous voyez ERR_NETWORK_ACCESS_DENIED, vous devrez peut-être vérifier auprès de l’administrateur si la stratégie de votre pare-feu local a bloqué l’accès au domaine *.database.windows.net ou au port distant 1443.
    - Vous pouvez tenter la même opération immédiatement sur un autre ordinateur et/ou un autre environnement réseau pour écarter tout problème de configuration réseau sur votre PC.

- l’état est « 40x », « 50 » ou un autre nombre, sélectionnez le ou les éléments pour afficher les détails. Vous devez voir les détails de l’élément à droite. Recherchez la section « Response Header », puis vérifiez s’il existe un élément nommé « access-control-allow-origin ». Si c’est le cas, vérifiez s’il a l’une des valeurs suivantes :

    - `*` (astérisque unique)
    - https://web.azuresynapse.net/ (ou autre valeur par laquelle commence le texte dans la barre d’adresse de votre navigateur)

Si l’en-tête de réponse contient l’une des valeurs ci-dessus, cela signifie que nous devrions avoir déjà collecté les informations d’échec. Vous pouvez ouvrir un ticket de support si nécessaire et éventuellement joindre la capture d’écran des détails de l’élément.

Si vous ne voyez pas l’en-tête ou si l’en-tête n’a pas l’une des valeurs mentionnées ci-dessus, joignez une capture d’écran des détails de l’élément lorsque vous ouvrez le ticket.

 
![détails de l’élément](media/troubleshooting-synapse-studio/item-details.png)
 
Si les étapes ci-dessus ne résolvent pas votre problème, vous devrez peut-être ouvrir un ticket de support. Lorsque vous soumettez votre ticket de support, incluez l’« ID de session » ou les « informations de diagnostic » téléchargés au début de ce guide.

Lorsque vous signalez le problème, vous pouvez éventuellement prendre une capture d’écran de l’onglet « console » dans « Outils de développement » et l’attacher au ticket de support. Faites défiler le contenu et prenez plusieurs captures d’écran si nécessaire pour capturer l’intégralité du message.

![console d’outils de développement](media/troubleshooting-synapse-studio/developer-tool-console.png)

Si vous joignez des captures d’écran, indiquez l’heure (ou un intervalle de temps estimé) à laquelle vous avez pris les captures d’écran. Cela nous aidera à examiner le problème.

Certains navigateurs prennent en charge l’affichage des horodatages sous l’onglet « Console ». Pour Chromium Edge/Chrome, ouvrez la boîte de dialogue « Paramètres » dans « Outils de développement », puis vérifiez « Afficher les horodateurs » sous l’onglet « Préférences ».

![paramètres de la console d’outils de développement](media/troubleshooting-synapse-studio/developer-tool-console-settings.png)

![afficher l’horodatage](media/troubleshooting-synapse-studio/show-time-stamp.png)

## <a name="next-steps"></a>Étapes suivantes
Si les étapes précédentes ne permettent pas de résoudre le problème, [créez un ticket de support](../../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
