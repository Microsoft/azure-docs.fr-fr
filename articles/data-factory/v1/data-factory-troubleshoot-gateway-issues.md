---
title: Résoudre les problèmes des passerelles de gestion des données
description: Fournit des conseils pour résoudre les problèmes liés à la passerelle de gestion des données.
services: data-factory
author: nabhishek
manager: anandsub
ms.assetid: c6756c37-4e5a-4d1e-ab52-365f149b4128
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/01/2017
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 7b12ff431b2d164baf4f70fa5341f538b16bca51
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896563"
---
# <a name="troubleshoot-issues-with-using-data-management-gateway"></a>Résoudre les problèmes liés à l’utilisation de la passerelle de gestion des données
Cet article fournit des informations sur la résolution des problèmes liés à l’utilisation de la passerelle de gestion des données.

> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Guide pratique pour créer et configurer le runtime d’intégration auto-hébergé dans Data Factory](../create-self-hosted-integration-runtime.md).

Pour obtenir des informations détaillées sur la passerelle, consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md). Pour connaître la procédure de déplacement de données d’une base de données SQL Server vers un stockage Blob Microsoft Azure avec la passerelle, consultez l’article [Déplacer des données entre un site local et le cloud](data-factory-move-data-between-onprem-and-cloud.md).

## <a name="failed-to-install-or-register-gateway"></a>Échec de l’inscription ou de l’installation de la passerelle
### <a name="1-problem"></a>1. Problème
Ce message d’erreur s’affiche lors de l’installation et de l’enregistrement d’une passerelle, en particulier lors du téléchargement du fichier d’installation de la passerelle.

`Unable to connect to the remote server". Please check your local settings (Error Code: 10003).`

#### <a name="cause"></a>Cause
L’ordinateur sur lequel vous essayez d’installer la passerelle n’a pas pu télécharger le fichier d’installation de la dernière version de la passerelle à partir du Centre de téléchargement en raison d’un problème réseau.

#### <a name="resolution"></a>Résolution
Vérifiez si les paramètres de serveur proxy de votre pare-feu bloquent la connexion réseau de l’ordinateur vers le [Centre de téléchargement](https://download.microsoft.com/) et modifiez ces paramètres si nécessaire.

Vous pouvez également télécharger le fichier d’installation de la dernière version de la passerelle à partir du [Centre de téléchargement](https://www.microsoft.com/download/details.aspx?id=39717) sur d’autres ordinateurs qui peuvent accéder à ce dernier. Copiez ensuite le fichier du programme d’installation sur l’ordinateur hôte de la passerelle et exécutez-le manuellement pour installer et mettre à jour la passerelle.

### <a name="2-problem"></a>2. Problème
Cette erreur s’affiche lorsque vous tentez d’installer une passerelle en cliquant sur **Installer directement sur cet ordinateur** dans le portail Azure.

`Error:  Abort installing a new gateway on this computer because this computer has an existing installed gateway and a computer without any installed gateway is required for installing a new gateway.`  

#### <a name="cause"></a>Cause
Une passerelle est déjà installée sur l’ordinateur.

#### <a name="resolution"></a>Résolution
Désinstallez la passerelle existante sur l’ordinateur et cliquez de nouveau sur le lien **Installer directement sur cet ordinateur**.

### <a name="3-problem"></a>3. Problème
Cette erreur peut s’afficher lorsque vous inscrivez une nouvelle passerelle.

`Error: The gateway has encountered an error during registration.`

#### <a name="cause"></a>Cause
Ce message peut s’afficher pour l’une des raisons suivantes :

* Le format de la clé de passerelle n’est pas valide.
* La clé de passerelle a été invalidée.
* La clé de passerelle a été régénérée à partir du portail.  

#### <a name="resolution"></a>Résolution
Vérifiez que vous utilisez la bonne clé de passerelle du portail. Si besoin, régénérez une clé et utilisez cette clé pour inscrire la passerelle.

### <a name="4-problem"></a>4. Problème
Le message d’erreur suivant peut s’afficher lors de l’inscription d’une passerelle.

`Error: The content or format of the gateway key "{gatewayKey}" is invalid, please go to azure portal to create one new gateway or regenerate the gateway key.`



![Contenu ou format de clé non valide](media/data-factory-troubleshoot-gateway-issues/invalid-format-gateway-key.png)

#### <a name="cause"></a>Cause
Le contenu ou le format de la clé de passerelle est incorrect. Vous avez peut-être copié uniquement une partie de la clé dans le portail ou vous utilisez une clé non valide.

#### <a name="resolution"></a>Résolution
Générez une clé de passerelle dans le portail et utilisez le bouton Copier pour copier la clé entière. Collez-la ensuite dans cette fenêtre pour inscrire la passerelle.

### <a name="5-problem"></a>5. Problème
Le message d’erreur suivant peut s’afficher lors de l’inscription d’une passerelle.

`Error: The gateway key is invalid or empty. Specify a valid gateway key from the portal.`

![Capture d’écran mettant en évidence le message d’erreur indiquant que la clé de passerelle n’est pas valide ou est vide.](media/data-factory-troubleshoot-gateway-issues/gateway-key-is-invalid-or-empty.png)

#### <a name="cause"></a>Cause
La clé de passerelle a été régénérée ou la passerelle a été supprimée dans le portail Azure. Cela peut également se produire si vous n’utilisez pas la configuration de passerelle de gestion des données la plus récente.

#### <a name="resolution"></a>Résolution
Vérifiez que la configuration de la passerelle de gestion des données correspond à la dernière version (vous trouverez la dernière version dans le [centre de téléchargement](https://go.microsoft.com/fwlink/p/?LinkId=271260) Microsoft).

Si la configuration est à jour/correspond à la dernière version et que la passerelle existe encore sur le portail, régénérez la clé de la passerelle dans le portail Azure, utilisez le bouton Copier pour copier la clé et collez-la dans cette fenêtre pour inscrire la passerelle. Sinon, recréez la passerelle et recommencez.

### <a name="6-problem"></a>6. Problème
Le message d’erreur suivant peut s’afficher lors de l’inscription d’une passerelle.

`Error: Gateway has been online for a while, then shows “Gateway is not registered” with the status “Gateway key is invalid”`

![La clé de passerelle n’est pas valide ou est vide.](media/data-factory-troubleshoot-gateway-issues/gateway-not-registered-key-invalid.png)

#### <a name="cause"></a>Cause
Cette erreur peut se produire si la passerelle a été supprimée ou la clé de passerelle associée a été régénérée.

#### <a name="resolution"></a>Résolution
Si la passerelle a été supprimée, recréez-la à partir du portail, cliquez sur **Inscrire** , copiez la clé dans le portail, collez-la et essayez d’inscrire la passerelle.

Si la passerelle existe encore, mais que sa clé a été régénérée, utilisez la nouvelle clé pour inscrire la passerelle. Si vous n’avez pas la clé, régénérez-la sur le portail.

### <a name="7-problem"></a>7. Problème
Lorsque vous inscrivez une passerelle, il se peut que vous deviez entrer le chemin d’accès et le mot de passe d’un certificat.

![Capture d’écran montrant où entrer le chemin d’accès et le mot de passe du certificat.](media/data-factory-troubleshoot-gateway-issues/specify-certificate.png)

#### <a name="cause"></a>Cause
La passerelle a été inscrite sur d’autres ordinateurs auparavant. Lors de l’inscription initiale de la passerelle, un certificat de chiffrement lui a été associé. Le certificat peut avoir été généré automatiquement par la passerelle ou fourni par l’utilisateur.  Ce certificat est utilisé pour chiffrer les informations d’identification de la banque de données (service lié).  

![Exportation du certificat](media/data-factory-troubleshoot-gateway-issues/export-certificate.png)

Lorsque vous restaurez la passerelle sur un autre ordinateur hôte, l’Assistant Inscription demande ce certificat afin de déchiffrer les informations d’identification précédemment chiffrées avec ce certificat.  Sans ce certificat, les informations d’identification ne peuvent pas être déchiffrées par la nouvelle passerelle et les exécutions d’activités de copie suivantes associées à cette nouvelle passerelle échouent.  

#### <a name="resolution"></a>Résolution
Si vous avez exporté le certificat d’informations d’identification de l’ordinateur d’origine de la passerelle avec le bouton **Exporter** de l’onglet **Paramètres** du Gestionnaire de configuration de passerelle de gestion des données, utilisez le certificat ici.

Vous ne pouvez pas ignorer cette étape lors de la récupération d’une passerelle. Si le certificat est manquant, vous devez supprimer la passerelle depuis le portail et en recréer une.  Mettez également à jour tous les services liés associés à la passerelle en entrant à nouveau leurs informations d’identification.

### <a name="8-problem"></a>8. Problème
Le message d’erreur suivant peut s’afficher.

`Error: The remote server returned an error: (407) Proxy Authentication Required.`

#### <a name="cause"></a>Cause
Cette erreur se produit lorsque votre passerelle se trouve dans un environnement qui exige un proxy HTTP pour accéder aux ressources Internet ou lorsque le mot de passe d’authentification de votre serveur proxy est modifié, mais n’est pas actualisé dans votre passerelle.

#### <a name="resolution"></a>Résolution
Suivez les instructions de la section Considérations relatives aux serveurs proxy de cet article et configurez les paramètres de proxy à l’aide du Gestionnaire de configuration de passerelle de gestion des données.

## <a name="gateway-is-online-with-limited-functionality"></a>La passerelle est en ligne avec des fonctionnalités limitées
### <a name="1-problem"></a>1. Problème
Vous constatez que la passerelle est connectée avec des fonctionnalités limitées.

#### <a name="cause"></a>Cause
La passerelle peut être connectée avec des fonctionnalités limitées pour l’une des raisons suivantes :

* La passerelle ne peut pas se connecter au service cloud via Azure Service Bus.
* Le service cloud ne peut pas se connecter à la passerelle via Service Bus.

Lorsque la passerelle est connectée avec des fonctionnalités limitées, il se peut que vous ne soyez pas en mesure d’utiliser l’Assistant de copie Data Factory afin de créer des pipelines de données pour la copie de données vers des banques de données locales ou à partir de celles-ci. Pour résoudre ce problème, vous pouvez utiliser Data Factory Editor dans le portail, Visual Studio ou Azure PowerShell.

#### <a name="resolution"></a>Résolution
La procédure de résolution de ce problème (passerelle connectée avec des fonctionnalités limitées) n’est pas la même si la passerelle ne peut pas se connecter au service cloud ou si c’est l’inverse. Les sections suivantes fournissent des solutions.

### <a name="2-problem"></a>2. Problème
Vous obtenez l’erreur suivante.

`Error: Gateway cannot connect to cloud service through service bus`

![La passerelle ne peut pas se connecter au service cloud](media/data-factory-troubleshoot-gateway-issues/gateway-cannot-connect-to-cloud-service.png)

#### <a name="cause"></a>Cause
La passerelle ne peut pas se connecter au service cloud via Service Bus.

#### <a name="resolution"></a>Résolution
Suivez ces étapes pour remettre la passerelle en ligne :

1. Autorisez les règles de trafic sortant d’adresse IP sur l’ordinateur de la passerelle et le pare-feu d’entreprise. Vous pouvez trouver les adresses IP dans le journal des événements Windows (ID == 401) : Tentative d’accès à un socket de manière interdite par ses autorisations d’accès XX.XX.XX.XX:9350.
1. Configurez les paramètres de proxy de la passerelle. Pour plus d’informations, consultez la section Considérations relatives aux serveurs proxy.
1. Activez les ports sortants 5671 et 9350 à 9354 sur le pare-feu Windows de l’ordinateur passerelle et le pare-feu d’entreprise. Pour plus d’informations, consultez la section Ports et pare-feu. Cette étape est facultative, mais elle est recommandée pour des questions de performances.

### <a name="3-problem"></a>3. Problème
Vous obtenez l’erreur suivante.

`Error: Cloud service cannot connect to gateway through service bus.`

#### <a name="cause"></a>Cause
erreur temporaire de connectivité réseau.

#### <a name="resolution"></a>Résolution
Suivez ces étapes pour remettre la passerelle en ligne :

1. Attendez quelques minutes. La connectivité est récupérée automatiquement une fois que l’erreur a disparu.
1. Si l’erreur persiste, redémarrez le service de passerelle.

## <a name="failed-to-author-linked-service"></a>Impossible de créer le service lié
### <a name="problem"></a>Problème
Cette erreur peut s’afficher lorsque vous essayez d’utiliser le Gestionnaire d’informations d’identification dans le portail pour entrer les informations d’identification d’un nouveau service lié ou pour mettre à jour les informations d’identification d’un service lié existant.

`Error: The data store '<Server>/<Database>' cannot be reached. Check connection settings for the data source.`

Lorsque vous obtenez cette erreur, la page de paramètres du Gestionnaire de configuration de passerelle de gestion des données peut se présenter comme dans la capture d’écran suivante.

![Impossible de contacter la base de données](media/data-factory-troubleshoot-gateway-issues/database-cannot-be-reached.png)

#### <a name="cause"></a>Cause
Le certificat TLS/SSL a peut-être été perdu sur l’ordinateur de la passerelle. L’ordinateur de la passerelle ne peut pas charger le certificat actuellement utilisé pour le chiffrement TLS. Un message d’erreur similaire au message suivant peut également être affiché dans le journal des événements.

 `Unable to get the gateway settings from cloud service. Check the gateway key and the network connection. (Certificate with thumbprint cannot be loaded.)`

#### <a name="resolution"></a>Résolution
Procédez comme suit pour résoudre le problème :

1. Lancez le Gestionnaire de configuration de passerelle de gestion des données.
2. Basculez vers l’onglet **Paramètres** .  
3. Cliquez sur le bouton **Modifier** pour modifier le certificat TLS/SSL.

   ![Bouton de modification du certificat](media/data-factory-troubleshoot-gateway-issues/change-button-ssl-certificate.png)
4. Sélectionnez un nouveau certificat comme certificat TLS/SSL. Vous pouvez utiliser n’importe quel certificat TLS/SSL généré par vos soins ou par une organisation quelconque.

   ![Indiquer un certificat](media/data-factory-troubleshoot-gateway-issues/specify-http-end-point.png)

## <a name="copy-activity-fails"></a>Échec de l’activité de copie
### <a name="problem"></a>Problème
Vous pouvez obtenir l’erreur « UserErrorFailedToConnectToSqlserver » suivante après avoir configuré un pipeline dans le portail.

`Error: Copy activity encountered a user error: ErrorCode=UserErrorFailedToConnectToSqlServer,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Cannot connect to SQL Server`

#### <a name="cause"></a>Cause
Cela peut se produire pour différentes raisons et la procédure de résolution varie en conséquence.

#### <a name="resolution"></a>Résolution
Autorisez les connexions TCP sortantes sur le port TCP/1433 côté client de la passerelle de gestion des données avant la connexion à une base de données SQL.

Si la base de données cible se trouve dans Azure SQL Database, vérifiez également les paramètres de pare-feu SQL Server pour Azure.

Consultez la section suivante pour tester la connexion à la banque de données locale.

## <a name="data-store-connection-or-driver-related-errors"></a>Erreurs liées à la connexion à la banque de données ou au pilote
Si vous obtenez des erreurs liées à la connexion à la banque de données ou au pilote, procédez comme suit :

1. Lancez le Gestionnaire de configuration de passerelle de gestion des données sur l’ordinateur de la passerelle.
2. Basculez vers l’onglet **Diagnostics** .
3. Dans **Tester la connexion** , ajoutez les valeurs de groupe de la passerelle.
4. Cliquez sur **Tester la connexion** pour vérifier si vous pouvez vous connecter à la source de données locale à partir de l’ordinateur de la passerelle en utilisant les informations de connexion et d’identification. Si le test de connexion échoue encore après l'installation d'un pilote, redémarrez la passerelle pour récupérer les dernières modifications.

![Tester la connexion dans l’onglet Diagnostics](media/data-factory-troubleshoot-gateway-issues/test-connection-in-diagnostics-tab.png)

## <a name="gateway-logs"></a>Journaux d’activité de la passerelle
### <a name="send-gateway-logs-to-microsoft"></a>Envoyer les journaux d’activité de la passerelle à Microsoft
Lorsque vous contactez le Support Microsoft pour résoudre des problèmes de passerelle, il peut vous être demandé de fournir les journaux d’activité de votre passerelle. Avec la dernière version de la passerelle, vous pouvez partager les journaux d’activité de passerelle requis en deux clics de bouton dans le Gestionnaire de configuration de passerelle de gestion des données.    

1. Basculez vers l’onglet **Diagnostics** du Gestionnaire de configuration de passerelle de gestion des données.

    ![Onglet Diagnostics de la passerelle de gestion des données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-diagnostics-tab.png)
2. Cliquez sur **Envoyer des journaux d’activité** pour afficher la boîte de dialogue suivante.

    ![Lien Envoyer des journaux d’activité de la passerelle de gestion des données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-dialog.png)
3. (Facultatif) Cliquez sur **Afficher les journaux d’activité** pour consulter les journaux d’activité dans l’Observateur d’événements.
4. (Facultatif) Cliquez sur **Confidentialité** pour consulter la déclaration de confidentialité relative aux services web Microsoft.
5. Lorsque vous êtes satisfait des éléments que vous vous apprêtez à charger, cliquez sur **Envoyer des journaux d’activité** pour envoyer les journaux d’activité des sept derniers jours à Microsoft en vue de résoudre les problèmes. L’état de l’opération d’envoi des journaux devrait s’afficher comme dans la capture d’image suivante.

    ![Capture d’écran montrant où afficher l’état de l’opération d’envoi de journaux.](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-status.png)
6. Une fois l’opération terminée, la boîte de dialogue illustrée dans la capture d’écran suivante s’affiche.

    ![État de l’opération Envoyer des journaux d’activité pour la passerelle de gestion des données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-result.png)
7. Notez l’ **ID du rapport** et communiquez-le au Support Microsoft. L’ID du rapport permet de localiser les journaux d’activité de la passerelle que vous avez chargés pour la résolution des problèmes.  L’ID du rapport est également enregistré dans l’Observateur d’événements.  Vous pouvez le trouver en recherchant l’ID d’événement « 25 » et en vérifiant la date et l’heure.

    ![ID du rapport de l’opération Envoyer des journaux d’activité pour la passerelle de gestion des données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-send-logs-report-id.png)    

### <a name="archive-gateway-logs-on-gateway-host-machine"></a>La passerelle d’archive ouvre une session sur l’ordinateur hôte de la passerelle
Dans certains cas, il se peut que vous ayez des problèmes avec la passerelle et que vous ne puissiez pas partager directement les journaux d’activité de la passerelle :

* Vous installez et inscrivez manuellement la passerelle.
* Vous essayez d’inscrire la passerelle avec une clé régénérée dans le Gestionnaire de configuration de passerelle de gestion des données.
* Vous tentez d’envoyer des journaux d’activité et le service hôte de la passerelle ne peut pas être connecté.

Dans ces cas de figure, vous pouvez enregistrer les journaux d’activité de la passerelle dans un fichier zip et fournir celui-ci au Support Microsoft. Par exemple, si vous recevez une erreur lorsque vous inscrivez la passerelle, comme illustré dans la capture d’écran suivante.   

![Erreur d’inscription de la passerelle de gestion des données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-registration-error.png)

Cliquez sur le lien **Archiver les journaux d’activité de la passerelle** pour archiver et enregistrer les journaux d’activité, puis fournissez le fichier zip au Support Microsoft.

![Lien Archiver les journaux d’activité de la passerelle de gestion de données](media/data-factory-troubleshoot-gateway-issues/data-management-gateway-archive-logs.png)

### <a name="locate-gateway-logs"></a>Rechercher dans les journaux d’activité de la passerelle
Vous pouvez accéder à des informations détaillées sur les journaux d’activité de la passerelle dans les journaux d’événements Windows.

1. Démarrez l’ **Observateur d’événements** Windows.
2. Localisez les journaux d’activité dans le dossier **Journaux des applications et services** > **Passerelle de gestion des données**.

   Lors de la résolution de problèmes liés à la passerelle, recherchez les événements de niveau Erreur dans l’Observateur d’événements.

![Journaux d’activité de la passerelle de gestion des données dans l’Observateur d’événements](media/data-factory-troubleshoot-gateway-issues/gateway-logs-event-viewer.png)
