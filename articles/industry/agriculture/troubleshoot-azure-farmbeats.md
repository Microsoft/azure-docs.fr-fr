---
title: Résoudre les problèmes liés à Azure FarmBeats
description: Cet article explique comment résoudre les problèmes liés à Azure FarmBeats.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: dd0ed78c56e4d656a2ecee6395d831ed093e85b5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001607"
---
# <a name="troubleshoot-azure-farmbeats"></a>Résoudre les problèmes liés à Azure FarmBeats

Cet article fournit des solutions aux problèmes courants liés à Azure FarmBeats. Pour obtenir de l’aide supplémentaire, contactez notre [Forum de support questions et réponses](/answers/topics/azure-farmbeats.html) ou envoyez-nous un e-mail à l’adresse farmbeatssupport@microsoft.com.

> [!NOTE]
  > Si vous avez installé FarmBeats au mois d’avril et que vos travaux échouent avec un message d’erreur vide, il se peut que votre installation n’ait pas reçu de quota Batch pour hiérarchiser le support apporté à des organisations critiques en matière de sécurité et de santé. Pour plus d’informations, consultez [cet article](https://azure.microsoft.com/blog/update-2-on-microsoft-cloud-services-continuity/) . Vous devez demander l’allocation de machines virtuelles au compte Batch pour exécuter les tâches.

## <a name="install-issues"></a>Problèmes d’installation

  > [!NOTE]
  > Si vous redémarrez l’installation en raison d’une erreur, veillez à supprimer le **Groupe de ressources** ou toutes les ressources de celui-ci avant de relancer l’installation.

### <a name="invalid-sentinel-credentials"></a>Informations d’identification de Sentinel non valides

Les informations d’identification de Sentinel fournies pendant l’installation sont incorrectes. Redémarrez l’installation avec les informations d’identification correctes.

### <a name="the-regional-account-quota-of-batch-accounts-for-the-specified-subscription-has-been-reached"></a>Le quota régional de comptes Batch pour l’abonnement spécifié a été atteint

Augmentez le quota ou supprimez les comptes Batch inutilisés, puis réexécutez le déploiement.

### <a name="invalid-resource-group-location"></a>Emplacement du groupe de ressources non valide

Assurez-vous que l’emplacement du **Groupe de ressources** est bien celui de la **Région** spécifiée pendant l’installation.

### <a name="other-install-issues"></a>Autres problèmes d’installation

Contactez-nous en vous munissant des informations suivantes :

- Votre ID d’abonnement
- Nom du groupe ressources
- Procédez de la manière décrite ci-dessous pour joindre le fichier journal relatif à l’échec du déploiement :

    1. Dans le portail Azure, accédez au **Groupe de ressources**.

    2. Dans la section **Paramètres** sur le côté gauche, sélectionnez **Déploiements**.

    3. Pour chaque déploiement en **Échec**, sélectionnez et téléchargez les détails. Joignez ce fichier au courrier.

## <a name="sensor-telemetry"></a>Données de télémétrie des capteurs

### <a name="cant-view-telemetry-data"></a>Impossible d’afficher les données de télémétrie

**Symptôme** : Des appareils ou des capteurs sont déployés, et vous avez lié FarmBeats à votre partenaire d’appareil. Toutefois, vous ne pouvez ni obtenir ni afficher les données de télémétrie sur FarmBeats.

**Action corrective** :

1. Accédez à votre groupe de ressources FarmBeats.
2. Sélectionnez l’espace de noms **Event Hub** (« sensor-partner-eh-namespace-xxxx »), cliquez sur « Event Hubs », puis recherchez le nombre de messages entrants dans l’Event Hub attribué au partenaire.
3. Effectuez l'une des opérations suivantes :

   - S’il n’y a *aucun message entrant*, contactez votre partenaire d’appareil.  
   - S’il y a des *messages entrants*, contactez-nous en joignant les journaux Datahub et Accelerator, ainsi que la télémétrie capturée.

Pour comprendre comment télécharger les journaux, consultez la section [« Collecter les journaux manuellement »](#collect-logs-manually).  

### <a name="cant-view-telemetry-data-after-ingesting-historicalstreaming-data-from-your-sensors"></a>Impossible d’afficher les données de télémétrie après l’ingestion des données historique/de diffusion en continu à partir de vos capteurs

**Symptôme** : Des appareils ou des capteurs sont déployés. Vous avez créé les appareils/capteurs sur FarmBeats et ingéré la télémétrie vers l’EventHub, mais vous ne pouvez pas obtenir ni visualiser les données de télémétrie sur FarmBeats.

**Action corrective** :

1. Assurez-vous d’avoir correctement effectué l’inscription du partenaire. Pour le vérifier, accédez à votre Swagger DataHub et à /Partner API, entrez la commande GET et regardez si le partenaire est bien inscrit. Si ce n’est pas le cas, suivez les [étapes suivantes](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats) pour ajouter un partenaire.

2. Vérifiez que vous avez utilisé le bon format de message de télémétrie :

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>":<value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

### <a name="dont-have-the-azure-event-hubs-connection-string"></a>Chaîne de connexion Azure Event Hubs indisponible

**Action corrective** :

1. Dans Datahub Swagger, accédez à l’API Partner.
2. Sélectionnez **Get** (Obtenir) > **Try it out (Essayer)**  > **Execute** (Exécuter).

> [!NOTE]
> Notez l’ID partenaire du partenaire de capteur qui vous intéresse.

3. Revenez dans l’API partenaire et sélectionnez **Get/\<ID>** .
4. Spécifiez l’ID partenaire obtenue à l’étape 3, puis sélectionnez **Exécuter**.

   La réponse de l’API contient normalement la chaîne de connexion Event Hubs.

### <a name="device-appears-offline"></a>L’appareil apparaît hors connexion

**Symptômes** : Les appareils sont installés et vous avez lié FarmBeats à votre partenaire d’appareil. Les appareils sont en ligne et envoient des données de télémétrie, mais ils apparaissent comme étant hors connexion.

**Action corrective** L’intervalle de création de rapports n’est pas configuré pour cet appareil. Pour définir l’intervalle de création de rapports, contactez le fabricant de votre appareil. 

### <a name="error-deleting-a-device"></a>Erreur de suppression d’un appareil

Quand vous supprimez un appareil, vous pouvez être confronté à l’un des scénarios d’erreur suivants :  

**Message** : « L’appareil est référencé dans les capteurs : Un ou plusieurs capteurs sont associés à l’appareil. Supprimez les capteurs, puis supprimez l’appareil. »  

**Signification** : L’appareil est associé à plusieurs capteurs déployés dans la ferme.

**Action corrective** :  

1. Supprimez les capteurs associés à l’appareil avec Accelerator.  
2. Si vous souhaitez associer les capteurs à un autre appareil, demandez à votre partenaire d’appareil d’en faire de même.  
3. Supprimez l’appareil à l’aide d’un appel `DELETE API`, puis définissez le paramètre force avec la valeur *true*.  

**Message** : « L’appareil est référencé dans les appareils en tant que ParentDeviceId : Un ou plusieurs appareils associés à cet appareil sont des appareils enfants. Supprimez-les, puis supprimez cet appareil. »  

**Signification** : D’autres appareils sont associés à votre appareil.  

**Action corrective** :

1. Supprimez les appareils associés à cet appareil spécifique.  
2. Supprimez l’appareil spécifique.  

    > [!NOTE]
    > Vous ne pouvez pas supprimer un appareil s’il est associé à des capteurs. Pour plus d’informations sur la suppression des capteurs associés, consultez la section **Supprimer un capteur** dans [Obtenir des données de capteur auprès de partenaires de capteur](get-sensor-data-from-sensor-partner.md).
    > Les partenaires ne sont pas autorisés à supprimer un appareil ou un capteur. Seuls des administrateurs sont autorisés à supprimer.

## <a name="issues-with-jobs"></a>Problèmes liés aux travaux

### <a name="farmbeats-internal-error"></a>Erreur interne FarmBeats

**Message** : « Erreur interne FarmBeats, consultez le guide de résolution des problèmes pour plus d’informations. »

**Action corrective** Ce problème peut provenir d’une défaillance temporaire dans le pipeline de données. Recréez le travail. Si l’erreur persiste, contactez-nous en joignant le message d’erreur ou les journaux.

## <a name="accelerator-troubleshooting"></a>Résolution des problèmes des accélérateurs

### <a name="access-control"></a>Contrôle d’accès

**Problème** : Vous recevez une erreur pendant que vous ajoutez une attribution de rôle.

**Message** : « Aucun utilisateur correspondant trouvé. »

**Action corrective** Vérifiez l’ID d’e-mail pour lequel vous essayez d’ajouter une attribution de rôle. L’ID d’e-mail doit correspondre exactement à celui inscrit pour cet utilisateur dans Active Directory. Si l’erreur persiste, contactez-nous en joignant le message d’erreur ou les journaux.

### <a name="unable-to-log-in-to-accelerator"></a>Impossible de se connecter à l’accélérateur

**Message** : « Erreur : Vous n’êtes pas autorisé à appeler le service. Contactez l’administrateur pour obtenir une autorisation. »

**Action corrective** Demandez à l’administrateur de vous autoriser à accéder au déploiement FarmBeats. Pour ce faire, vous pouvez effectuer un POST des API RoleAssignment ou utiliser Access Control dans le volet **Settings** (Paramètres) d’Accelerator.  

Si l’accès vous a déjà été accordé et que vous êtes confronté à cette erreur, réessayez en actualisant la page. Si l’erreur persiste, contactez-nous en joignant le message d’erreur ou les journaux.

![Capture d’écran montrant l’erreur d’autorisation.](./media/troubleshoot-azure-farmbeats/accelerator-troubleshooting-1.png)

### <a name="accelerator-issues"></a>Problèmes liés à Accelerator  

**Problème** : Vous recevez une erreur Accelerator dont la cause est inconnue.

**Message** : « Erreur : Une erreur inconnue s’est produite. »

**Action corrective** Cette erreur se produit si vous laissez la page inactive trop longtemps. Actualisez la page. Si l’erreur persiste, contactez-nous en joignant le message d’erreur ou les journaux.

**Problème** : FarmBeats Accelerator n’indique pas la dernière version, même après la mise à niveau de FarmBeatsDeployment.

**Action corrective** Cette erreur se produit en raison de la persistance du worker du service dans le navigateur. Effectuez les actions suivantes :

1. Fermez tous les onglets de navigateur pour lesquels Accelerator est ouvert, puis fermez la fenêtre du navigateur.
2. Démarrez une nouvelle instance du navigateur et rechargez l’URI d’Accelerator. Cette action charge la nouvelle version d’Accelerator.

## <a name="sentinel-imagery-related-issues"></a>Sentinel : Problèmes liés à l’imagerie

### <a name="wrong-username-or-password"></a>Nom d’utilisateur ou mot de passe incorrect

**Message d’échec du travail** : « Une authentification complète est nécessaire pour accéder à cette ressource. »

**Action corrective** : Effectuez l’une des actions suivantes :

- Mettez à jour FarmBeats avec les nom d’utilisateur et mot de passe corrects en suivant des étapes ci-dessous, puis recommencez le travail.

  **Mettre à jour le nom d’utilisateur Sentinel**

    1. Connectez-vous au [portail Azure](https://portal.azure.com).
    2. Dans la zone **Rechercher**, recherchez le groupe de ressources FarmBeats Datahub.
    3. Sélectionnez Stockage du compte de stockage***** > **Conteneurs** > **batch-PREP-Files** > **to_vm** > **config.ini**
    4. Sélectionnez **Modifier**
    5. Mettez à jour le nom d’utilisateur dans la section sentinel_account.

  **Mettre à jour le mot de passe Sentinel**

    1. Connectez-vous au [portail Azure](https://portal.azure.com).
    2. Dans la zone **Rechercher**, recherchez le groupe de ressources FarmBeats Datahub.
    3. Sélectionnez keyvault-*****
    4. Sélectionnez stratégies d’accès sous Paramètres.
    5. Sélectionnez **Ajouter une stratégie d’accès**.
    6. Utilisez **Gestion des secrets** pour configurer à partir du Modèle et vous ajouter au Principal.
    7. Dans la page **Stratégies d’accès**, sélectionnez **Ajouter**, puis **Enregistrer**.
    8. Sous **Paramètres**, sélectionnez **Secrets**.
    9. Sélectionner **Sentinel-Password**.
    10. Créez une nouvelle version de la valeur, puis activez-la.

- Réexécutez le travail ayant échoué ou exécutez un travail d’indices satellite pour une plage de dates de 5 à 7 jours, puis vérifiez si le travail est réussi.

### <a name="sentinel-hub-wrongurlor-site-not-accessible"></a>Hub Sentinel : URL ou site incorrect inaccessible

**Message d’échec du travail** : « Désolé, une erreur s’est produite. La page à laquelle vous tentez d’accéder est (temporairement) indisponible. »

**Action corrective** :

1. Ouvrez [Sentinel](https://scihub.copernicus.eu/dhus/) dans votre navigateur pour voir si le site web est accessible.
2. Si le site web n’est pas accessible, vérifiez si un pare-feu, un réseau d’entreprise ou tout autre logiciel de blocage empêche l’accès au site web, puis prenez les mesures nécessaires pour autoriser l’URL de Sentinel. 
3. Réexécutez le travail ayant échoué ou exécutez un travail d’indices satellite pour une plage de dates de 5 à 7 jours, puis vérifiez si l’exécution du travail a réussi.  

### <a name="sentinel-server-down-for-maintenance"></a>Serveur Sentinel : Hors service pour cause de maintenance

**Message d’échec du travail** : « Le hub d’accès Copernicus Open sera bientôt disponible ! Nous sommes désolés pour ce désagrément, nous effectuons actuellement une opération de maintenance. Le portail sera bientôt de retour en ligne. » 

**Action corrective** :

Ce problème peut se produire si des activités de maintenance sont effectuées sur le serveur Sentinel.

1. Si un travail ou un pipeline échoue en raison d’une maintenance en cours d’exécution, soumettez à nouveau le travail après un certain temps. 

   Pour plus d’informations sur les activités de maintenance Sentinel planifiées ou non, accédez au site [Copernicus Open Access Hub News](https://scihub.copernicus.eu/news/).  

2. Réexécutez le travail ayant échoué ou exécutez un travail d’indices satellite pour une plage de dates de 5 à 7 jours, puis vérifiez si l’exécution du travail a réussi.

### <a name="sentinel-maximum-number-of-connections-reached"></a>Sentinel : Nombre maximal de connexions Sentinel atteint

**Message d’échec du travail** : « Nombre maximal de deux flux simultanés atteints par l’utilisateur ’\<username>’. »

**Signification** : Si un travail échoue parce que le nombre maximal de connexions a été atteint, le même compte Sentinel est utilisé pour plusieurs travaux.

**Action corrective** : Effectuez l’une des actions suivantes :

* Avant de réexécuter un travail qui a échoué, attendez que les autres travaux soient terminés.
* Créez un nouveau compte Sentinel, puis mettez à jour les nom d’utilisateur et mot de passe Sentinel dans FarmBeats.

### <a name="sentinel-server-refused-connection"></a>Serveur Sentinel : Connexion refusée

**Message d’échec du travail** : « Le serveur a refusé la connexion à :  http://172.30.175.69:8983/solr/dhus. »

**Action corrective** : Ce problème peut se produire si des activités de maintenance sont effectuées sur le serveur Sentinel.

1. Si un travail ou un pipeline échoue en raison d’une maintenance en cours d’exécution, soumettez à nouveau le travail après un certain temps.

   Pour plus d’informations sur les activités de maintenance Sentinel planifiées ou non, accédez au site [Copernicus Open Access Hub News](https://scihub.copernicus.eu/news/).  

2. Réexécutez le travail ayant échoué ou exécutez un travail d’indices satellite pour une plage de dates de 5 à 7 jours, puis vérifiez si l’exécution du travail a réussi.

### <a name="soil-moisture-map-has-white-areas"></a>La carte d’humidité des sols a des zones blanches

**Problème** : La **carte d’humidité des sols** a été générée, mais contient principalement des zones blanches.

**Action corrective** : Ce problème peut se produire si les index satellites générés pour la période pour laquelle la carte a été demandée ont des valeurs NDVI inférieures à 0,3. Pour plus d’informations, consultez le [Guide technique de Sentinel](https://earth.esa.int/web/sentinel/technical-guides/sentinel-2-msi/level-2a/algorithm).

1. Réexécutez le travail pour une plage de dates différente, puis vérifiez si les valeurs NDVI dans les index satellites sont supérieures à 0.3.

## <a name="collect-logs-manually"></a>Collecter les journaux manuellement

[Installez et déployez l’Explorateur Stockage Azure]( https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows).

### <a name="collect-azure-data-factory-job-logs-or-app-service-logs-in-datahub"></a>Collecter les journaux de travaux Azure Data Factory ou les journaux d’App Service dans DataHub

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la zone **Rechercher**, recherchez le groupe de ressources FarmBeats Datahub.
3. Dans le tableau de bord **Groupe de ressources**, recherchez le compte de stockage *datahublogs\** . Par exemple, *datahublogsmvxmq*  
4. Dans la colonne **Nom**, sélectionnez le compte de stockage pour afficher le tableau de bord **Compte de stockage**.
5. Dans le volet **datahubblogs\* *_, sélectionnez _* Ouvrir dans l’Explorateur** pour afficher l’application **Explorateur Stockage Azure**.
6. Dans le volet gauche, sélectionnez **Conteneurs d’objets blob**, puis **job-logs** pour les journaux d’Azure Data Factory ou **appinsights-logs** pour les journaux d’App Service.
7. Sélectionnez **Télécharger**, puis téléchargez les journaux dans un dossier local sur votre ordinateur.

    ![Capture d’écran montrant les fichiers journaux téléchargés.](./media/troubleshoot-azure-farmbeats/collecting-logs-manually-1.png)

### <a name="collect-azure-data-factory-job-logs-or-app-service-logs-for-accelerator"></a>Collecter les journaux de travaux d’Azure Data Factory les journaux d’App Service pour Accelerator

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la zone **Rechercher**, recherchez le groupe de ressources FarmBeats Accelerator.
3. Dans le tableau de bord **Groupe de ressources**, recherchez le compte de stockage *storage\** . Par exemple, *storagedop4k\** .
4. Sélectionnez le compte de stockage dans la colonne **Nom** pour afficher le tableau de bord **Compte de stockage**.
5. Dans le volet **storage\* *_, sélectionnez _* Ouvrir dans l’Explorateur** pour ouvrir l’application Explorateur Stockage Azure.
6. Dans le volet gauche, sélectionnez **Conteneurs d’objets blob**, puis **job-logs** pour les journaux d’Azure Data Factory ou **appinsights-logs** pour les journaux d’App Service.
7. Sélectionnez **Télécharger**, puis téléchargez les journaux dans un dossier local sur votre ordinateur.

## <a name="high-cpu-usage"></a>Utilisation élevée du processeur

**Erreur** : Vous recevez une alerte par e-mail qui fait référence à une **alerte d’utilisation élevée du processeur**.

**Action corrective** :

1. Accédez à votre groupe de ressources FarmBeats Datahub.
2. Sélectionnez **App Service**.  
3. Accédez à la [page des prix App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) Scale Up, puis sélectionnez un niveau tarifaire approprié.

## <a name="weather-data-job-failures"></a>Échecs de travaux de données météorologiques

**Erreur** : Vous exécutez des travaux pour obtenir des données météorologiques, mais le travail échoue

### <a name="collect-logs-to-troubleshoot-weather-data-job-failures"></a>Collecter les journaux pour résoudre les échecs des travaux de données météorologiques

1. Accédez au groupe de ressources FarmBeats sur le portail Azure.
2. Cliquez sur le service Data Factory qui fait partie de ce groupe de ressources. Le service a une étiquette « Référence SKU : Datahub »

> [!NOTE]
> Pour voir les étiquettes des services dans le groupe de ressources, cliquez sur « Modifier les colonnes » et ajoutez des « étiquettes » dans la vue du groupe de ressources.

:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-1.png" alt-text="Capture d’écran mettant en évidence l’étiquette sku:Datahub.":::

3. Dans la page de présentation de la fabrique de données, cliquez sur **Créer et surveiller**. Un nouvel onglet s’ouvre dans votre navigateur. Cliquez sur **Surveiller**.

:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-2.png" alt-text="Capture d’écran qui met en évidence l’option de menu Redémarrer.":::

4. Vous voyez la liste des exécutions de pipeline qui font partie de l’exécution du travail météorologique. Cliquez sur le travail pour lequel vous souhaitez collecter les journaux.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-3.png" alt-text="Capture d’écran mettant en évidence l’option de menu Exécutions de pipeline et la tâche sélectionnée.":::

5. Dans la page de présentation du pipeline, vous voyez la liste des exécutions d’activités. Prenez note des ID d’exécution des activités pour lesquelles vous souhaitez collecter les journaux.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-4.png" alt-text="Capture d’écran montrant la liste des exécutions d’activités.":::

6. Revenez à votre groupe de ressources FarmBeats sur le portail Azure, puis cliquez sur le compte de stockage portant le nom **datahublogs-XXXX**.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-5.png" alt-text="Capture d’écran mettant en évidence le compte de stockage nommé datahublogs-XXXX.":::

7. Cliquez sur **Conteneurs** -> **adfjobs**. Dans la zone de recherche, entrez l’ID d’exécution du travail que vous avez noté à l’étape 5 ci-dessus.
 
:::image type="content" source="./media/troubleshoot-Azure-farmbeats/weather-log-6.png" alt-text="Projet FarmBeats":::

8. Le résultat de la recherche inclut le dossier contenant les journaux relatifs au travail. Téléchargez les journaux et envoyez-les à farmbeatssupport@microsoft.com pour obtenir de l’aide afin de résoudre le problème.