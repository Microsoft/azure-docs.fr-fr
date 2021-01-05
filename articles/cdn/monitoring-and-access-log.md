---
title: Surveillance, métriques et journaux bruts pour Azure CDN
description: Cet article explique comment configurer et utiliser la surveillance, les métriques et les journaux bruts Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/23/2020
ms.author: yuajia
ms.openlocfilehash: 43f53d1098e08a0f913e3baec2c6aaf3d65054d0
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501291"
---
# <a name="real-time-monitoring-metrics-and-access-logs-for-azure-cdn"></a>Surveillance en temps réel, métriques et journaux d’accès pour Azure CDN
Grâce à Azure CDN de Microsoft, vous pouvez surveiller les ressources des manières suivantes pour vous aider à résoudre, suivre et déboguer les problèmes : 

* Les journaux bruts, qui fournissent des informations détaillées sur chaque requête reçue par CDN. Les journaux bruts diffèrent des journaux d’activité. Les journaux d’activité apportent une visibilité dans les opérations effectuées sur des ressources Azure.
* Les métriques, qui affichent quatre métriques clés relatives à CDN, notamment le taux d’accès par octet, le nombre de requêtes, la taille de la réponse et la latence totale. Différentes dimensions sont également fournies pour décomposer les métriques.
* Les alertes, qui permettent au client de configurer une alerte pour les métriques clés.
* Des métriques supplémentaires, qui permettent aux clients d’utiliser Azure Log Analytics pour activer des métriques supplémentaires de valeur. Nous fournissons également des exemples de requêtes pour quelques autres métriques sous Azure Log Analytics.

> [!IMPORTANT]
> La fonctionnalité des journaux bruts HTTP est disponible pour Azure CDN à partir de Microsoft.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="configuration---azure-portal"></a>Configuration – Portail Azure

Pour configurer les journaux bruts de votre instance Azure CDN à partir du profil Microsoft : 

1. Dans le menu du portail Azure, sélectionnez **Toutes les ressources** >>  **\<your-CDN-profile>** .

2. Sous **Supervision**, sélectionnez **Paramètres de diagnostic**.

3. Sélectionnez **+ Ajouter le paramètre de diagnostic**.

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-01.png" alt-text="Ajouter un paramètre de diagnostic pour le profil CDN." border="true":::
    
    > [!IMPORTANT]
    > Les journaux bruts sont disponibles uniquement au niveau du profil, tandis que les journaux agrégés des codes d’état http sont disponibles au niveau du point de terminaison.

4. Sous **Paramètres de diagnostic**, entrez un nom pour le paramètre de diagnostic sous **Nom des paramètres de diagnostic**.

5. Sélectionnez le journal **AzureCdnAccessLog** et définissez la rétention en jours.

6. Sélectionnez les **Détails de la destination**. Les options de destination sont les suivantes :
    * **Envoyer à Log Analytics**
        * Sélectionnez l’**Abonnement** et l’**Espace de travail Log Analytics**.
    * **Archiver dans un compte de stockage**
        * Sélectionnez l’**Abonnement** et le **Compte de stockage**.
    * **Diffuser vers un hub d’événements**
        * Sélectionnez l’**Abonnement**, l’**Espace de noms du hub d’événements**, le **Nom du hub d’événements (facultatif)** et le **Nom de la stratégie du hub d’événements**.

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-02.png" alt-text="Configurer la destination pour les paramètres du journal." border="true":::

7. Sélectionnez **Enregistrer**.

## <a name="configuration---azure-powershell"></a>Configuration – Azure PowerShell

Utilisez [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) pour configurer le paramètre de diagnostic des journaux bruts.

Les données conservées sont définies par l’option **-RetentionInDays** dans la commande.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="enable-diagnostic-logs-in-a-storage-account"></a>Activer les journaux de diagnostic dans un compte de stockage

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```

2. Pour activer les journaux de diagnostic dans un compte de stockage, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $storageacct = <your-storage-account-name>
    $diagname = <your-diagnostic-setting-name>
    $days = '30'

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $storage = Get-AzStorageAccount -ResourceGroupName $rsg -Name $storageacct

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -StorageAccountId $storage.id -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```

### <a name="enable-diagnostics-logs-for-log-analytics-workspace"></a>Activer les journaux de diagnostic pour l’espace de travail Log Analytics

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Pour activer les journaux de diagnostic pour un espace de travail Log Analytics, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $workspacename = <your-log-analytics-workspace-name>
    $diagname = <your-diagnostic-setting-name>
    $days = '30'

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $rsg -Name $workspacename

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -WorkspaceId $workspace.ResourceId -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```
### <a name="enable-diagnostics-logs-for-event-hub-namespace"></a>Activer les journaux de diagnostic pour l’espace de noms Event Hub

1. Connectez-vous à Azure PowerShell :

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Pour activer les journaux de diagnostic pour un espace de noms Event Hubs, entrez ces commandes. Remplacez les variables par vos valeurs :

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $evthubnamespace = <your-event-hub-namespace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $eventhub = Get-AzEventHubNamespace -ResourceGroupName $rsg -Name $eventhubname

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -EventHubName $eventhub.id -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```

## <a name="raw-logs-properties"></a>Propriétés des journaux bruts

Azure CDN du service Microsoft fournit actuellement des journaux bruts. Les journaux bruts présentent chaque requête d’API individuellement, chaque entrée appliquant le schéma suivant : 

| Propriété  | Description |
| ------------- | ------------- |
| BackendHostname | Si la requête est transférée à un serveur principal, ce champ représente le nom d’hôte du serveur principal. Ce champ est vide si la requête est redirigée ou transférée vers un cache régional (lorsque la mise en cache est activée pour la règle d’acheminement). |
| CacheStatus | Pour les scénarios de mise en cache, ce champ définit une absence/une correspondance dans le cache au niveau POP |
| ClientIp | Adresse IP du client à l’origine de la demande. S’il existait un en-tête X-Forwarded-For dans la demande, l’adresse IP du client est sélectionnée de la même façon. |
| ClientPort | Adresse IP du port du client qui a effectué la requête. |
| HttpMethod | Méthode HTTP utilisée par la requête. |
| HttpStatusCode | Code d’état HTTP retourné par le proxy. |
| HttpStatusDetails | État résultant de la requête. Vous trouverez la signification de cette valeur de chaîne dans la Table de référence des états. |
| HttpVersion | Type de la requête ou de la connexion. |
| POP | Nom abrégé de la périphérie où la demande est arrivée. |
| RequestBytes | Taille du message de requête HTTP en octets, en-têtes de requête et corps de requête compris. |
| RequestUri | URI de la requête reçue. |
| ResponseBytes | Octets envoyés en tant que réponse par le serveur back-end.  |
| RoutingRuleName | Nom de la règle de routage correspondant à la requête. |
| RulesEngineMatchNames | Noms des règles correspondant à la demande. |
| SecurityProtocol | Version du protocole TLS/SSL utilisée par la requête, ou Null si aucun chiffrement. |
| SentToOriginShield </br> (déconseillé) * **Consultez les notes sur la désapprobation dans la section suivante.**| Si la valeur est true, cela signifie que la requête a été traitée à partir du cache de protection d’origine au lieu du pop de périphérie. La protection d’origine est un cache parent utilisé pour améliorer le taux d’accès au cache. |
| isReceivedFromClient | Si la valeur est true, cela signifie que la requête provient du client. Si la valeur est false, la requête est en échec dans la périphérie (POP enfant) et reçoit une réponse à partir de la protection d’origine (POP parent). |
| TimeTaken | Durée, en secondes, écoulée entre le premier octet de la requête Front Door et le dernier octet de la réponse. |
| TrackingReference | Chaîne de référence unique qui identifie une requête traitée par Front Door, également envoyée en tant qu’en-tête X-Azure-Ref au client. Nécessaire pour pouvoir effectuer une recherche détaillée dans les journaux d’accès pour une requête spécifique. |
| UserAgent | Type de navigateur utilisé par le client. |
| ErrorInfo | Ce champ contient le type d’erreur spécifique permettant de réduire la zone de résolution des problèmes. </br> Les valeurs possibles incluent : </br> **NoError** : Indique qu’aucune erreur n’a été trouvée. </br> **CertificateError** : Erreur de certificat SSL générique.</br> **CertificateNameCheckFailed** : Le nom d’hôte dans le certificat SSL n’est pas valide ou ne correspond pas. </br> **ClientDisconnected** : Échec de la requête en raison de la connexion réseau du client. </br> **UnspecifiedClientError** : Erreur du client générique. </br> **InvalidRequest** : Requête non valide. Cela peut se produire en raison d’un en-tête, d’un corps et d’une URL incorrect(e)s. </br> **DNSFailure** : Échec DNS. </br> **DNSNameNotResolved** : Le nom ou l’adresse du serveur n’a pas pu être résolue. </br> **OriginConnectionAborted** : La connexion avec l’origine a été arrêtée brusquement. </br> **OriginConnectionError** : Erreur de connexion d’origine générique. </br> **OriginConnectionRefused** : La connexion avec l’origine n’a pas pu être établie. </br> **OriginError** : Erreur d’origine générique. </br> **OriginInvalidResponse** : L’origine a renvoyé une réponse non valide ou non reconnue. </br> **OriginTimeout** : Le délai d’expiration de la requête d’origine a expiré. </br> **ResponseHeaderTooBig** : L’origine a retourné un en-tête de réponse trop grand. </br> **RestrictedIP** : La requête a été bloquée en raison d’une adresse IP restreinte. </br> **SSLHandshakeError** : Impossible d’établir la connexion avec l’origine en raison d’un échec d’établissement d’une liaison SSL. </br> **UnspecifiedError** : Une erreur ne correspondant à aucune des erreurs dans le tableau s’est produite. |
| TimeToFirstByte | Durée, en millisecondes, entre le moment où Microsoft CDN reçoit la requête et le moment où le premier octet est envoyé au client. La durée est mesurée uniquement du côté de Microsoft. Les données côté client ne sont pas mesurées. |
> [!NOTE]
> Vous pouvez voir les journaux sous votre profil Log Analytics en exécutant une requête. Voici un exemple de requête :
    ```
    AzureDiagnostics | where Category == "AzureCdnAccessLog"
    ```

### <a name="sent-to-origin-shield-deprecation"></a>Dépréciation de Sent to origin shield
La propriété de journal brut **isSentToOriginShield** est déconseillée et remplacée par un nouveau champ **isReceivedFromClient**. Utilisez le nouveau champ si vous utilisez déjà le champ déconseillé. 

Les journaux bruts incluent les journaux générés à partir de la périphérie de CDN (POP enfant) et de la protection d’origine. La protection d’origine fait référence aux nœuds parents qui se trouvent stratégiquement dans le monde entier. Ces nœuds communiquent avec les serveurs d’origine et réduisent la charge du trafic sur l’origine. 

Pour chaque requête qui accède à la protection d’origine, il y a deux entrées de journal :

* une pour les nœud de périphérie
* l’autre pour la protection d’origine 

Pour différencier la sortie ou les réponses des nœuds de périphérie par rapport à la protection d’origine, vous pouvez utiliser le champ **isReceivedFromClient** pour récupérer les données correctes. 

Si la valeur est false, cela signifie que la requête reçoit une réponse de la protection d’origine jusqu’aux nœuds de périphérie. Cette approche est efficace pour comparer les journaux bruts aux données de facturation. Les frais ne sont pas facturés pour la sortie de la protection d’origine jusqu’aux nœuds de périphérie. Des frais sont facturés pour la sortie des nœuds de périphérie jusqu’aux clients. 

**Exemple de requête Kusto pour exclure les journaux générés sur la protection d’origine dans Log Analytics.**

```kusto
AzureDiagnostics 
| where OperationName == "Microsoft.Cdn/Profiles/AccessLog/Write" and Category == "AzureCdnAccessLog"  
| where isReceivedFromClient == true

```

> [!IMPORTANT]
> La fonctionnalité des journaux bruts HTTP est disponible automatiquement pour tous les profils créés ou mis à jour après le **25 février 2020**. Pour les profils CDN créés précédemment, vous devez mettre à jour le point de terminaison CDN après la configuration de la journalisation. Par exemple, vous pouvez accéder au filtrage géographique sous les points de terminaison CDN et bloquer tous les pays/régions qui ne sont pas pertinents pour votre charge de travail, puis cliquer sur Enregistrer.


## <a name="metrics"></a>Mesures
Azure CDN de Microsoft est intégré à Azure Monitor et publie quatre métriques CDN pour faciliter le suivi, la résolution et le débogage des problèmes. 

Les métriques sont affichées dans des graphiques et sont accessibles via PowerShell, l’interface CLI et l’API. Les métriques CDN sont gratuites.

Azure CDN de Microsoft mesure et envoie ses métriques par intervalles de 60 secondes. Les métriques peuvent prendre jusqu’à 3 minutes pour s’afficher dans le portail. 

Pour plus d’informations, voir [Mesures Azure Monitor](../azure-monitor/platform/data-platform-metrics.md).

**Métriques prises en charge par Azure CDN de Microsoft**

| Mesures  | Description | Dimensions |
| ------------- | ------------- | ------------- |
| Taux d’accès par octet* | Pourcentage de sortie du cache CDN, calculé par rapport à la sortie totale. | Point de terminaison |
| RequestCount | Nombre de requêtes de clients prises en charge par CDN. | Point de terminaison </br> Pays du client. </br> Région du client. </br> État HTTP </br> Groupe d’états HTTP. |
| ResponseSize | Nombre d’octets envoyés en tant que réponses de la périphérie de CDN aux clients. |Point de terminaison </br> Pays du client. </br> Région du client. </br> État HTTP </br> Groupe d’états HTTP. |
| TotalLatency | Durée totale de la requête du client reçue par CDN **jusqu’au dernier octet de réponse envoyé de CDN au client**. |Point de terminaison </br> Pays du client. </br> Région du client. </br> État HTTP </br> Groupe d’états HTTP. |

**_Taux d'accès par octet = (sortie de la périphérie - la sortie de l'origine)/sortie de la périphérie_*

Scénarios exclus du calcul du taux d’accès par octet :

* Vous ne configurez explicitement aucun cache par le biais du comportement de mise en cache des chaînes de requête ou du moteur de règles.
* Vous configurez explicitement une directive de contrôle du cache avec un cache privé ou sans magasin.

### <a name="metrics-configuration"></a>Configuration des métriques

1. Dans le menu du portail Azure, sélectionnez **Toutes les ressources** >>  **\<your-CDN-profile>** .

2. Sous **Supervision**, sélectionnez **Métriques** :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-03.png" alt-text="Métriques pour le profil CDN." border="true":::

3. Sélectionnez **Ajouter une métrique**, puis sélectionnez la métrique à ajouter :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-04.png" alt-text="Ajouter et sélectionner une métrique pour le profil CDN." border="true":::

4. Sélectionnez **Ajouter un filtre** pour ajouter un filtre :
    
    :::image type="content" source="./media/cdn-raw-logs/raw-logs-05.png" alt-text="Appliquer le filtre à la métrique." border="true":::

5. Sélectionnez **Appliquer le fractionnement** pour afficher la tendance selon différentes dimensions :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-06.png" alt-text="Appliquer le fractionnement à la métrique." border="true":::

6. Sélectionnez **Nouveau graphique** pour ajouter un nouveau graphique :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-07.png" alt-text="Ajouter un nouveau graphique à l’affichage des métriques." border="true":::

### <a name="alerts"></a>Alertes

Vous pouvez configurer des alertes sur Microsoft CDN en sélectionnant **Supervision** >> **Alertes**.

Sélectionnez **Nouvelle règle d’alerte** pour les métriques répertoriées dans la section Métriques :

:::image type="content" source="./media/cdn-raw-logs/raw-logs-08.png" alt-text="Configurer des alertes pour le point de terminaison CDN." border="true":::

L’alerte sera facturée conformément à Azure Monitor. Pour plus d’informations sur les alertes, consultez [Alertes Azure Monitor](../azure-monitor/platform/alerts-overview.md).

### <a name="additional-metrics"></a>Métriques supplémentaires
Vous pouvez activer des métriques supplémentaires à l’aide d’Azure Log Analytics et des journaux bruts pour un coût supplémentaire.

1. Suivez les étapes ci-dessus pour permettre aux diagnostics d’envoyer un journal brut à Log Analytics.

2. Sélectionnez l’espace de travail Log Analytics que vous avez créé :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-09.png" alt-text="Sélectionner l’espace de travail Log Analytics." border="true":::   

3. Sous **Général** dans l’espace de travail Log Analytics, sélectionnez **Journaux**.  Sélectionnez ensuite **Prise en main**.

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-10.png" alt-text="Espace de travail des ressources Log Analytics." border="true":::   
 
4. Sélectionnez **Profils CDN**.  Sélectionnez un exemple de requête à exécuter ou fermez l’écran d’exemple pour entrer une requête personnalisée :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-11.png" alt-text="Exemple d’écran de requête." border="true":::   

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-12.png" alt-text="Exécution d’une requête." border="true":::   

4. Pour afficher les données par graphique, sélectionnez **Graphique**.  Sélectionnez **Épingler au tableau de bord** pour épingler le graphique au tableau de bord Azure :

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-13.png" alt-text="Épinglez un graphique au tableau de bord." border="true"::: 

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez activé les journaux bruts HTTP pour le service CDN Microsoft.

Pour plus d’informations sur Azure CDN et les autres services Azure mentionnés dans cet article, consultez :

* [Analyser](cdn-log-analysis.md) des modèles d’utilisation Azure CDN.

* En savoir plus sur [Azure Monitor](../azure-monitor/overview.md).

* Configurer [Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md).