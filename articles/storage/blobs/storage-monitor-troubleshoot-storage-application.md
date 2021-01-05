---
title: Surveiller et résoudre les problèmes d’une application de stockage cloud dans Azure | Microsoft Docs
description: Utilisez des outils de diagnostic, des métriques et la génération d’alertes pour surveiller et résoudre les problèmes d’une application cloud.
author: normesta
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 07/20/2018
ms.author: normesta
ms.reviewer: fryu
ms.openlocfilehash: 6c1d9b813db15900521968865c1036d53ad53f41
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544783"
---
# <a name="monitor-and-troubleshoot-a-cloud-storage-application"></a>Surveiller et résoudre les problèmes d’une application de stockage cloud

Ce didacticiel est le quatrième et dernier volet d’une série. Vous apprenez à surveiller et résoudre les problèmes d’une application de stockage cloud.

Dans ce quatrième volet, vous apprenez à :

> [!div class="checklist"]
> * Activer la journalisation et les métriques
> * Activer les alertes pour les erreurs d’autorisation
> * Exécuter un trafic de test avec des jetons SAS incorrects
> * Télécharger et analyser des journaux d’activité

[Azure Storage Analytics](../common/storage-analytics.md) fournit la journalisation et les données de métriques d’un compte de stockage. Ces données fournissent des insights sur l’intégrité de votre compte de stockage. Pour collecter des données Azure Storage Analytics, vous pouvez configurer la journalisation, les mesures et les alertes. Ce processus implique l’activation de la journalisation, la configuration des métriques et l’activation des alertes.

La journalisation et les métriques des comptes de stockage sont activées sous l’onglet **Diagnostics** du portail Azure. La journalisation du stockage permet d’enregistrer les détails des demandes ayant réussi ou échoué dans votre compte de stockage. Ces journaux d’activité permettent d’afficher les détails des opérations de lecture, d’écriture et de suppression sur vos tables, files d’attente et objets blob Azure. Ils permettent également de connaître la raison des échecs de demande comme les délais d’attente, la limitation et les erreurs d’autorisation.

## <a name="log-in-to-the-azure-portal"></a>Se connecter au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com)

## <a name="turn-on-logging-and-metrics"></a>Activer la journalisation et les métriques

Dans le menu de gauche, sélectionnez **Groupes de ressources**, **myResourceGroup**, puis votre compte de stockage dans la liste des ressources.

Sous les **paramètres de Diagnostics (classique)** , définissez l’**État** sur **Activé**. Vérifiez que toutes les options sous **Propriétés de l’objet blob** sont activées.

Quand vous avez terminé, cliquez sur **Enregistrer**

![Capture d’écran qui met en évidence la section qui contient les paramètres de configuration permettant d’activer la journalisation et les métriques.](media/storage-monitor-troubleshoot-storage-application/enable-diagnostics.png)

## <a name="enable-alerts"></a>Activer les alertes

Les alertes sont un moyen d’envoyer par e-mail aux administrateurs ou de déclencher un webhook quand une métrique dépasse un seuil. Dans cet exemple, vous activez une alerte pour la métrique `SASClientOtherError`.

### <a name="navigate-to-the-storage-account-in-the-azure-portal"></a>Accédez à votre compte de stockage dans le Portail Azure

Sous la section **Surveillance**, sélectionnez **Alertes (classique)** .

Sélectionnez **Ajouter une alerte métrique (classique)** et terminez le formulaire **Ajouter une règle** en renseignant les informations requises. À partir de la liste déroulante **Métrique**, sélectionnez `SASClientOtherError`. Pour autoriser le déclenchement de votre alerte lors de la première erreur, à partir de la liste déroulante **Condition** sélectionnez **Supérieur ou égal à**.

![Volet Diagnostics](media/storage-monitor-troubleshoot-storage-application/add-alert-rule.png)

## <a name="simulate-an-error"></a>Simuler une erreur

Pour simuler une alerte valide, vous pouvez tenter de demander un objet blob qui n’existe pas à partir de votre compte de stockage. La commande suivante requiert un nom de conteneur de stockage. Vous pouvez utiliser le nom d’un conteneur existant ou en créer un pour les besoins de cet exemple.

Remplacez les espaces réservés par des valeurs réelles (assurez-vous que `<INCORRECT_BLOB_NAME>` est défini sur une valeur qui n’existe pas) et exécutez la commande.

```azurecli-interactive
sasToken=$(az storage blob generate-sas \
    --account-name <STORAGE_ACCOUNT_NAME> \
    --account-key <STORAGE_ACCOUNT_KEY> \
    --container-name <CONTAINER_NAME> \
    --name <INCORRECT_BLOB_NAME> \
    --permissions r \
    --expiry `date --date="next day" +%Y-%m-%d`)

curl https://<STORAGE_ACCOUNT_NAME>.blob.core.windows.net/<CONTAINER_NAME>/<INCORRECT_BLOB_NAME>?$sasToken
```

L’illustration suivante représente un exemple d’alerte basée sur l’échec simulé exécuté dans l’exemple précédent.

 ![Exemple d’alerte](media/storage-monitor-troubleshoot-storage-application/email-alert.png)

## <a name="download-and-view-logs"></a>Télécharger et afficher des journaux d’activité

Les journaux d’activité de stockage stockent des données dans un ensemble d’objets blob dans un conteneur d’objets blob nommé **$logs** dans votre compte de stockage. Ce conteneur n’apparaît pas si vous répertoriez tous les conteneurs d’objets blob de votre compte, mais vous pouvez voir son contenu si vous y accédez directement.

Dans ce scénario, vous utilisez [Microsoft Message Analyzer](/message-analyzer/microsoft-message-analyzer-operating-guide) pour interagir avec votre compte de stockage Azure.

### <a name="download-microsoft-message-analyzer"></a>Télécharger Microsoft Message Analyzer

Téléchargez [Microsoft Message Analyzer](/message-analyzer/installing-and-upgrading-message-analyzer) et installez l’application.

Lancez l’application et choisissez **Fichier** > **Ouvrir** > **À partir d’autres sources de fichiers**.

Dans la boîte de dialogue **Sélecteur de fichiers**, sélectionnez **+ Ajouter une connexion Azure**. Entrez le **nom du compte de stockage** et la **clé de compte**, puis cliquez sur **OK**.

![Boîte de dialogue Microsoft Message Analyzer - Ajouter une connexion de stockage Azure](media/storage-monitor-troubleshoot-storage-application/figure3.png)

Une fois que vous êtes connecté, développez les conteneurs dans l’arborescence de stockage pour afficher les objets blob de journal. Sélectionnez le dernier journal et cliquez sur **OK**.

![Capture d’écran qui montre Microsoft Message Analyzer et met en évidence le fichier journal sélectionné.](media/storage-monitor-troubleshoot-storage-application/figure4.png)

Dans la boîte de dialogue **Nouvelle session**, cliquez sur **Démarrer** pour afficher le journal.

Une fois le journal ouvert, vous pouvez consulter les événements de stockage. Comme vous pouvez le voir dans l’image suivante, un événement `SASClientOtherError` a été déclenché sur le compte de stockage. Pour plus d’informations sur la journalisation du stockage, visitez [Storage Analytics](../common/storage-analytics.md).

![Microsoft Message Analyzer - Affichage des événements](media/storage-monitor-troubleshoot-storage-application/figure5.png)

L’[Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/) est un autre outil qui peut être utilisé pour interagir avec vos comptes de stockage, y compris le conteneur **$logs** et les fichiers journaux d’activité qu’il contient.

## <a name="next-steps"></a>Étapes suivantes

Dans le quatrième et dernier volet de la série, vous avez appris à surveiller et résoudre les problèmes de votre compte de stockage, notamment :

> [!div class="checklist"]
> * Activer la journalisation et les métriques
> * Activer les alertes pour les erreurs d’autorisation
> * Exécuter un trafic de test avec des jetons SAS incorrects
> * Télécharger et analyser des journaux d’activité

Suivez ce lien pour consulter des exemples de stockage préconçus.

> [!div class="nextstepaction"]
> [Exemples de script de stockage Azure](storage-samples-blobs-cli.md)