---
title: Solution de gestion Office 365 dans Azure
description: Cet article donne des informations sur la configuration et l’utilisation de la solution Office 365 dans Azure.  Il inclut une description détaillée des enregistrements d’Office 365 créés dans Azure Monitor.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/30/2020
ms.openlocfilehash: eb20bf4164cb2153f6786dbec04f79453554fa25
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995860"
---
# <a name="office-365-management-solution-in-azure-preview"></a>Solution de gestion Office 365 dans Azure (préversion)

![Logo Office 365](media/solution-office-365/icon.png)

> [!IMPORTANT]
> ## <a name="solution-update"></a>Mise à jour de la solution
> Cette solution a été remplacée par la solution [Office 365](../../sentinel/connect-office-365.md) en disponibilité générale dans [Azure Sentinel](../../sentinel/overview.md) et par la [solution de supervision et de création de rapports Azure AD](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md). Combinées, ces deux solutions fournissent une version mise à jour de la solution Azure Monitor Office 365 précédente qui offre une expérience de configuration optimisée. Vous pouvez continuer à utiliser la solution existante jusqu’au 31 octobre 2020.
> 
> Azure Sentinel est une solution de gestion des informations et des événements de sécurité (SIEM) native dans le cloud qui ingère les journaux et offre des fonctionnalités SIEM supplémentaires, comme les détections, les investigations, la chasse et les insights basés sur le machine learning. Azure Sentinel permet maintenant l’ingestion des journaux d’activité Office 365 SharePoint et de gestion Exchange.
> 
> La création de rapports Azure AD vous permet d’avoir une vue plus complète des journaux de l’activité Azure AD dans votre environnement, y compris des événements de connexion, des événements d’audit et des changements apportés à votre annuaire. Pour connecter des journaux Azure AD, vous pouvez utiliser le [connecteur Azure Sentinel Azure AD](../../sentinel/connect-azure-active-directory.md) ou configurer l’[intégration des journaux Azure AD à Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). 
>
> La collecte des journaux Azure AD est facturée aux tarifs d’Azure Monitor.  Pour plus d’informations, consultez [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
>
> Pour utiliser la solution Azure Sentinel Office 365 :
> 1. L’utilisation du connecteur Office 365 dans Azure Sentinel impacte la tarification de votre espace de travail. Pour plus d’informations, consultez [Tarification Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).
> 2. Si vous utilisez déjà la solution Azure Monitor Office 365, vous devez d’abord la désinstaller à l’aide du script fourni dans la [section Désinstallation ci-après](#uninstall).
> 3. [Activez la solution Azure Sentinel](../../sentinel/quickstart-onboard.md) sur votre espace de travail.
> 4. Accédez à la page **Connecteurs de données** dans Azure Sentinel et activez le connecteur **Office 365**.
>
> ## <a name="frequently-asked-questions"></a>Forum aux questions
> 
> ### <a name="q-is-it-possible-to-on-board-the-office-365-azure-monitor-solution-between-now-and-october-31"></a>Q : Est-il possible d’intégrer la solution Azure Monitor Office 365 entre aujourd’hui et le 31 octobre ?
> Non, car les scripts d’intégration de la solution Azure Monitor Office 365 ne sont plus disponibles. La solution sera supprimée le 31 octobre.
> 
> ### <a name="q-will-the-tables-and-schemas-be-changed"></a>Q : Les tables et les schémas seront-ils modifiés ?
> Le nom et le schéma de la table **OfficeActivity** resteront les mêmes que dans la solution actuelle. Vous pourrez continuer à utiliser les mêmes requêtes dans la nouvelle solution, à l’exception des requêtes qui référencent des données Azure AD.
> 
> Les journaux de la nouvelle [solution de supervision et de création de rapports Azure AD](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md) seront ingérés dans les tables [SigninLogs](../../active-directory/reports-monitoring/concept-sign-ins.md) et [AuditLogs](../../active-directory/reports-monitoring/concept-audit-logs.md) au lieu de la table **OfficeActivity**. Pour plus d’informations, consultez la page [Analyser les journaux Azure AD](../../active-directory/reports-monitoring/howto-analyze-activity-logs-log-analytics.md), qui s’applique aussi aux utilisateurs d’Azure Sentinel et d’Azure Monitor.
> 
> Les exemples ci-dessous montrent la conversion de requêtes **OfficeActivity** en **SigninLogs** :
> 
> **Rechercher les connexions ayant échoué, par utilisateur :**
> 
> ```Kusto
> OfficeActivity
> | where TimeGenerated >= ago(1d) 
> | where OfficeWorkload == "AzureActiveDirectory"                      
> | where Operation == 'UserLoginFailed'
> | summarize count() by UserId    
> ```
> 
> ```Kusto
> SigninLogs
> | where ConditionalAccessStatus == "failure" or ConditionalAccessStatus == "notApplied"
> | summarize count() by UserDisplayName
> ```
> 
> **Afficher les opérations Azure AD :**
> 
> ```Kusto
> OfficeActivity
> | where OfficeWorkload =~ "AzureActiveDirectory"
> | sort by TimeGenerated desc
> | summarize AggregatedValue = count() by Operation
> ```
> 
> ```Kusto
> AuditLogs
> | summarize count() by OperationName
> ```
> 
> ### <a name="q-how-can-i-on-board-azure-sentinel"></a>Q : Comment puis-je intégrer Azure Sentinel ?
> Azure Sentinel est une solution que vous pouvez activer sur tout espace de travail Log Analytics, nouveau ou existant. Pour plus d’informations, consultez la [documentation sur l’intégration d’Azure Sentinel](../../sentinel/quickstart-onboard.md).
>
> ### <a name="q-do-i-need-azure-sentinel-to-connect-the-azure-ad-logs"></a>Q : Ai-je besoin de connecter Azure Sentinel et les journaux Azure AD ?
> Vous pouvez configurer l’[intégration des journaux Azure AD avec Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) indépendamment de la solution Azure Sentinel. Azure Sentinel fournit un connecteur natif et du contenu prêt à l’emploi pour les journaux Azure AD. Pour plus d’informations, reportez-vous à la question posée plus bas sur le contenu de sécurité prêt à l’emploi.
>
> ###    <a name="q-what-are-the-differences-when-connecting-azure-ad-logs-from-azure-sentinel-and-azure-monitor"></a>Q : Quelles sont les différences entre la connexion des journaux Azure AD à partir d’Azure Sentinel et à partir d’Azure Monitor ?
> Azure Sentinel et Azure Monitor se connectent à des journaux Azure AD basés sur la même [solution de supervision et de création de rapports Azure AD](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md). Azure Sentinel fournit un connecteur natif qui connecte en un clic les mêmes données et fournit des informations de supervision.
>
> ###    <a name="q-what-do-i-need-to-change-when-moving-to-the-new-azure-ad-reporting-and-monitoring-tables"></a>Q : Que dois-je modifier pour passer aux nouvelles tables de supervision et de création de rapports Azure AD ?
> Vous devez recréer toutes les requêtes qui utilisent des données Azure AD, y compris les requêtes dans les alertes, les tableaux de bord et tout contenu existant avec des données Office 365 Azure AD, dans les nouvelles tables.
>
> Azure Sentinel et Azure AD fournissent du contenu prédéfini que vous pouvez utiliser quand vous passez à la solution de supervision et de création de rapports Azure AD. Pour plus d’informations, consultez la question suivante sur le contenu de sécurité prêt à l’emploi et l’article [Comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory](../../active-directory/reports-monitoring/howto-use-azure-monitor-workbooks.md). 
>
> ### <a name="q-how-i-can-use-the-azure-sentinel-out-of-the-box-security-oriented-content"></a>Q : Comment utiliser le contenu de sécurité prêt à l’emploi d’Azure Sentinel ?
> Azure Sentinel fournit diverses fonctionnalités de sécurité prêtes à l’emploi, comme les tableaux de bord, les requêtes d’alertes personnalisées, les requêtes de chasse, les investigations et les réponses automatisées, basées sur les journaux Office 365 et Azure AD. Explorez le dépôt GitHub et les tutoriels Azure Sentinel pour en savoir plus :
>
> - [Détection des menaces prête à l’emploi](../../sentinel/tutorial-detect-threats-built-in.md)
> - [Créer des règles d’analytique personnalisées pour détecter les menaces suspectes](../../sentinel/tutorial-detect-threats-custom.md)
> - [Superviser vos données](../../sentinel/tutorial-monitor-your-data.md)
> - [Examiner les incidents avec Azure Sentinel](../../sentinel/tutorial-investigate-cases.md)
> - [Configurer des réponses automatisées aux menaces dans Azure Sentinel](../../sentinel/tutorial-respond-threats-playbook.md)
> - [Dépôt GitHub de la communauté Azure Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks)
> 
> ### <a name="q-does-azure-sentinel-provide-additional-connectors-as-part-of-the-solution"></a>Q : Azure Sentinel inclut-il des connecteurs supplémentaires avec la solution ?
> Oui. Consultez les [sources de données connectables à Azure Sentinel](../../sentinel/connect-data-sources.md).
> 
> ###    <a name="q-what-will-happen-on-october-31-do-i-need-to-offboard-beforehand"></a>Q : Que va-t-il se passer le 31 octobre ? Dois-je faire l’intégration avant cette date ?
> 
> - Vous ne pourrez plus recevoir de données de la solution **Office 365**. La solution sera supprimée de votre espace de travail et ne sera plus disponible sur la Place de marché.
> - Pour les clients d’Azure Sentinel, la **solution Office 365** de l’espace de travail Log Analytics sera incluse dans la solution Azure Sentinel **SecurityInsights**.
> - Si vous ne retirez pas votre solution manuellement avant le 31 octobre, vos données seront déconnectées automatiquement et la table **OfficeActivity** sera supprimée. Vous pourrez toujours restaurer la table en activant le connecteur Office 365 dans Azure Sentinel (voir ci-dessous).
> 
> ### <a name="q-will-my-data-transfer-to-the-new-solution"></a>Q : Mes données seront-elles transférées vers la nouvelle solution ?
> Oui. Quand vous supprimerez la solution **Office 365** de votre espace de travail, les données associées seront temporairement indisponibles en raison de la suppression du schéma. Lorsque vous activerez le nouveau connecteur **Office 365** dans Sentinel, le schéma sera restauré dans l’espace de travail et toutes les données déjà collectées seront disponibles. 
 

La solution de gestion Office 365 vous permet d’effectuer le monitoring de votre environnement Office 365 dans Azure Monitor.

- Surveillez les activités des utilisateurs dans vos comptes Office 365 pour analyser les modèles d’utilisation ainsi que pour identifier les tendances de comportement. Par exemple, vous pouvez extraire des scénarios d’utilisation spécifiques, tels que les fichiers partagés en dehors de votre organisation ou les sites SharePoint les plus populaires.
- Analysez les activités d’administrateur pour effectuer le suivi des modifications de configuration ou d’opérations de privilèges élevés.
- Détectez et analysez le comportement des utilisateurs indésirables, qui peut être personnalisé pour les besoins de votre organisation.
- Présentation d’audit et de conformité. Par exemple, vous pouvez surveiller les opérations d’accès aux fichiers sur des fichiers confidentiels, ce qui peut vous aider dans le processus d’audit et de conformité.
- Effectuez un dépannage opérationnel à l’aide des [requêtes dans les journaux](../log-query/log-query-overview.md) en haut des données d’activité Office 365 de votre organisation.


## <a name="uninstall"></a>Désinstaller l’interface

Vous pouvez supprimer la solution de gestion Office 365 suivant le processus décrit dans [Supprimer une solution de gestion](solutions.md#remove-a-monitoring-solution). Néanmoins, cela n’empêchera pas la collecte des données d’Office 365 dans Azure Monitor. Suivez la procédure ci-dessous pour vous désabonner d’Office 365 et arrêter la collecte de données.

1. Enregistrez le script suivant sous le nom *office365_unsubscribe.ps1*.

    ```powershell
    param (
        [Parameter(Mandatory=$True)][string]$WorkspaceName,
        [Parameter(Mandatory=$True)][string]$ResourceGroupName,
        [Parameter(Mandatory=$True)][string]$SubscriptionId,
        [Parameter(Mandatory=$True)][string]$OfficeTennantId
    )
    $line='#-------------------------------------------------------------------------------------------------------------------------------------------------------------------------'
    
    $line
    IF ($Subscription -eq $null)
        {Login-AzAccount -ErrorAction Stop}
    $Subscription = (Select-AzSubscription -SubscriptionId $($SubscriptionId) -ErrorAction Stop)
    $Subscription
    $option = [System.StringSplitOptions]::RemoveEmptyEntries 
    $Workspace = (Set-AzOperationalInsightsWorkspace -Name $($WorkspaceName) -ResourceGroupName $($ResourceGroupName) -ErrorAction Stop)
    $Workspace
    $WorkspaceLocation= $Workspace.Location
    
    # Client ID for Azure PowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2"
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    $domain='login.microsoftonline.com'
    $adTenant =  $Subscription[0].Tenant.Id
    $authority = "https://login.windows.net/$adTenant";
    $ARMResource ="https://management.azure.com/";
    $xms_client_tenant_Id ='55b65fb5-b825-43b5-8972-c8b6875867c1'
    
    switch ($WorkspaceLocation) {
           "USGov Virginia" { 
                             $domain='login.microsoftonline.us';
                              $authority = "https://login.microsoftonline.us/$adTenant";
                              $ARMResource ="https://management.usgovcloudapi.net/"; break} # US Gov Virginia
           default {
                    $domain='login.microsoftonline.com'; 
                    $authority = "https://login.windows.net/$adTenant";
                    $ARMResource ="https://management.azure.com/";break} 
                    }
    
    Function RESTAPI-Auth { 
    
    $global:SubscriptionID = $Subscription.SubscriptionId
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURIARM=$ARMResource;
    # Authenticate and Acquire Token 
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $platformParameters = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList "Auto"
    $global:authResultARM = $authContext.AcquireTokenAsync($resourceAppIdURIARM, $clientId, $redirectUri, $platformParameters)
    $global:authResultARM.Wait()
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $authHeader
    }
    
    Function Office-UnSubscribe-Call{
    
    #----------------------------------------------------------------------------------------------------------------------------------------------
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $ResourceName = "https://manage.office.com"
    $SubscriptionId   = $Subscription[0].Subscription.Id
    $OfficeAPIUrl = $ARMResource + 'subscriptions/' + $SubscriptionId + '/resourceGroups/' + $ResourceGroupName + '/providers/Microsoft.OperationalInsights/workspaces/' + $WorkspaceName + '/datasources/office365datasources_'  + $SubscriptionId + $OfficeTennantId + '?api-version=2015-11-01-preview'
    
    $Officeparams = @{
        ContentType = 'application/json'
        Headers = @{
        'Authorization'="$($authHeader)"
        'x-ms-client-tenant-id'=$xms_client_tenant_Id
        'Content-Type' = 'application/json'
        }
        Method = 'Delete'
        URI = $OfficeAPIUrl
      }
    
    $officeresponse = Invoke-WebRequest @Officeparams 
    $officeresponse
    
    }
    
    #GetDetails 
    RESTAPI-Auth -ErrorAction Stop
    Office-UnSubscribe-Call -ErrorAction Stop
    ```

2. Exécutez le script avec la commande suivante :

    ```
    .\office365_unsubscribe.ps1 -WorkspaceName <Log Analytics workspace name> -ResourceGroupName <Resource Group name> -SubscriptionId <Subscription ID> -OfficeTennantID <Tenant ID> 
    ```

    Exemple :

    ```powershell
    .\office365_unsubscribe.ps1 -WorkspaceName MyWorkspace -ResourceGroupName MyResourceGroup -SubscriptionId '60b79d74-f4e4-4867-b631-yyyyyyyyyyyy' -OfficeTennantID 'ce4464f8-a172-4dcf-b675-xxxxxxxxxxxx'
    ```

Vous êtes invité à entrer des informations d’identification. Fournissez les informations d’identification de votre espace de travail Log Analytics.

## <a name="data-collection"></a>Collecte de données

La collecte initiale des données peut prendre quelques heures. Dès qu’elle a commencé, Office 365 envoie une [notification Webhook](/office/office-365-management-api/office-365-management-activity-api-reference#receiving-notifications) avec des données détaillées à Azure Monitor chaque fois qu’un enregistrement est créé. Cet enregistrement est disponible dans Azure Monitor quelques minutes après sa réception.

## <a name="using-the-solution"></a>Utilisation de la solution

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

Quand vous ajoutez la solution Office 365 à votre espace de travail Log Analytics, la vignette **Office 365** est ajoutée à votre tableau de bord. Cette mosaïque affiche une valeur et une représentation graphique du nombre d’ordinateurs de votre environnement et leur conformité de mise à jour.<br><br>
![Vignette de résumé Office 365](media/solution-office-365/tile.png)  

Cliquez sur la vignette **Office 365** pour ouvrir le tableau de bord **Office 365**.

![Tableau de bord Office 365](media/solution-office-365/dashboard.png)  

Le tableau de bord comprend les colonnes figurant dans le tableau suivant. Chaque colonne répertorie les dix premières alertes (classées par nombre d’alertes) correspondant aux critères de cette colonne pour l’étendue et l’intervalle de temps spécifiés. Vous pouvez exécuter une recherche de journaux qui fournit la liste complète. Pour cela, cliquez sur Afficher tout en bas de la colonne ou cliquez sur l’en-tête de colonne.

| Colonne | Description |
|:--|:--|
| Opérations | Fournit des informations sur les utilisateurs actifs depuis vos abonnements entièrement surveillés Office 365. Vous serez également en mesure de voir le nombre d’activités qui se produisent au fil du temps.
| Exchange | Affiche la répartition des activités d’Exchange Server telles que l’autorisation ajouter une boîte aux lettres ou Définir une boîte aux lettres. |
| SharePoint | Affiche les activités supérieures que les utilisateurs exécutent sur les documents SharePoint. Quand vous explorez depuis cette vignette, la page de recherche affiche les détails de ces activités, tels que le document cible et l’emplacement de cette activité. Par exemple, pour un événement Fichier ouvert, vous pouvez afficher le document ouvert, le nom de compte qui lui est associé et son adresse IP. |
| Azure Active Directory | Inclut les activités utilisateur supérieures, telles que la réinitialisation du mot de passe utilisateur et les tentatives de connexion. Quand vous explorez, vous pouvez afficher les détails de ces activités, telles que l’état du résultat. Ceci est particulièrement utile si vous souhaitez surveiller les activités suspectes sur votre Azure Active Directory. |




## <a name="azure-monitor-log-records"></a>Enregistrements de journaux Azure Monitor

Tous les enregistrements créés dans l’espace de travail Log Analytics dans Azure Monitor par la solution Office 365 ont un **Type** de **OfficeActivity**.  La propriété **OfficeWorkload** détermine le service Office 365 auquel fait référence l’enregistrement : Exchange, AzureActiveDirectory, SharePoint ou OneDrive.  La propriété **RecordType** spécifie le type d’opération.  Les propriétés varient pour chaque type d’opération et sont affichées dans les tables ci-dessous.

### <a name="common-properties"></a>Propriétés communes

Les propriétés suivantes sont communes à tous les enregistrements d’Office 365.

| Propriété | Description |
|:--- |:--- |
| Type | *OfficeActivity* |
| ClientIP | L’adresse IP du terminal qui a été utilisée lorsque l’activité a été enregistrée. L’adresse IP s’affiche au format d’adresse IPv4 ou IPv6. |
| OfficeWorkload | Service d’Office 365 auquel fait référence l’enregistrement.<br><br>AzureActiveDirectory<br>Exchange<br>SharePoint|
| Opération | Le nom de l’activité utilisateur ou administrateur.  |
| OrganizationId | Le GUID pour le client Office 365 de votre organisation. Cette valeur est toujours la même pour votre organisation, quel que soit le service Office 365 dans lequel il se produit. |
| RecordType | Type d’opération effectuée. |
| ResultStatus | Indique si l’action (spécifiée dans la propriété Operation) a réussi ou non. Les valeurs possibles sont Réussie, Partiellement réussie ou Échec. Pour l’activité de l’administrateur Exchange, la valeur est True ou False. |
| UserId | L’UPN (nom d’utilisateur principal) de l’utilisateur qui a exécuté l’action enregistrée dans l’enregistrement ; par exemple, my_name@my_domain_name. Notez que les enregistrements pour l’activité exécutée par les comptes système (tels que SHAREPOINT\system ou NTAUTHORITY\SYSTEM) sont également inclus. | 
| UserKey | Un autre ID pour l’utilisateur identifié dans la propriété UserId.  Par exemple, cette propriété est remplie par l’ID unique du passeport (PUID) pour les événements exécutés par les utilisateurs dans SharePoint, OneDrive entreprise et Exchange. Cette propriété peut également spécifier la même valeur que la propriété UserID pour les événements qui se produisent dans d’autres services et les événements exécutés par des comptes système|
| UserType | Le type d’utilisateur qui a effectué l’opération.<br><br>Admin<br>Application<br>DcAdmin<br>Normal<br>Réservé<br>ServicePrincipal<br>Système |


### <a name="azure-active-directory-base"></a>Base Azure Active Directory

Les propriétés suivantes sont communes à tous les enregistrements d’Azure Active Directory.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AzureActiveDirectory_EventType | Le type de client Azure AD. |
| ExtendedProperties | Les propriétés étendues de l’événement d’Azure AD. |


### <a name="azure-active-directory-account-logon"></a>Connexion au compte Azure Active Directory

Ces enregistrements sont créés lorsqu’un utilisateur Active Directory tente de se connecter.

| Propriété | Description |
|:--- |:--- |
| `OfficeWorkload` | AzureActiveDirectory |
| `RecordType`     | AzureActiveDirectoryAccountLogon |
| `Application` | L’application qui a déclenché l’événement de connexion du compte, telles que Office 15. |
| `Client` | Détails sur le terminal client, le système d’exploitation du terminal et le navigateur du terminal qui a été utilisé pour l’événement de connexion du compte. |
| `LoginStatus` | Cette propriété est directement issue de OrgIdLogon.LoginStatus. Le mappage de différentes connexions intéressantes peut être effectué par les alertes des algorithmes. |
| `UserDomain` | Les informations d’identité client (TII). | 


### <a name="azure-active-directory"></a>Azure Active Directory

Ces enregistrements sont créés lors de la modification ou des ajouts apportés aux objets Azure Active Directory.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AADTarget | L’utilisateur sur lequel l’action est exécutée (identifiée par la propriété Opération). |
| Acteur | L'utilisateur ou le principal du service qui a effectué l'action. |
| ActorContextId | Le GUID de l’organisation à laquelle appartient l’acteur. |
| ActorIpAddress | L’adresse IP de l’acteur au format d’adresse IPV4 ou IPV6. |
| InterSystemsId | Le GUID qui effectuent le suivi des actions sur des composants au sein du service Office 365. |
| IntraSystemId |     Le GUID généré par Azure Active Directory pour effectuer le suivi de l’action. |
| SupportTicketId | L’ID du-ticket de support client pour l’action dans les situations de « agir-au-nom-de ». |
| TargetContextId | Le GUID de l’organisation à laquelle appartient l’utilisateur cible. |


### <a name="data-center-security"></a>Sécurité du centre de données

Ces enregistrements sont créés à partir des données d’audit de sécurité du centre de données.  

| Propriété | Description |
|:--- |:--- |
| EffectiveOrganization | Le nom du client vers lequel l’élévation/l’applet de commande est ciblée. |
| ElevationApprovedTime | L’horodatage de l’approbation de l’élévation. |
| ElevationApprover | Le nom d’un gestionnaire Microsoft. |
| ElevationDuration | La durée pendant laquelle l’élévation est active. |
| ElevationRequestId |     Un identifiant unique pour la requête d’élévation. |
| ElevationRole | Le rôle pour lequel l’élévation a été demandé. |
| ElevationTime | L’heure de début de l’élévation. |
| start_time | L’heure de début de l’exécution de l’applet de commande. |


### <a name="exchange-admin"></a>Administrateur Exchange

Ces enregistrements sont créés lorsque des modifications sont apportées à la configuration Exchange.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeAdmin |
| ExternalAccess |     Spécifie si l’applet de commande a été exécutée par un utilisateur dans votre organisation, par le personnel du centre de données Microsoft ou un compte de service du centre de données ou par un administrateur délégué. La valeur False indique que l’applet de commande a été exécuté par une personne dans votre organisation. La valeur True indique que l’applet de commande a été exécutée par le personnel du centre de données, un compte de service du centre de données ou un administrateur délégué. |
| ModifiedObjectResolvedName |     Il s’agit du nom convivial de l’objet qui a été modifié par l’applet de commande. Cette opération est enregistrée uniquement si l’applet de commande modifie l’objet. |
| OrganizationName | Le nom du client. |
| OriginatingServer | Le nom du serveur à partir duquel l’applet de commande a été exécutée. |
| Paramètres | Le nom et la valeur de tous les paramètres utilisés avec l’applet de commande qui est identifiée dans la propriété Operations. |


### <a name="exchange-mailbox"></a>Boîte aux lettres Exchange

Ces enregistrements sont créés lorsque des modifications ou ajouts sont apportés aux boîtes aux lettres Exchange.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| ClientInfoString | Informations sur le client de messagerie utilisé pour effectuer l’opération, telles que les informations sur la version du navigateur, la version d’Outlook et l’appareil mobile. |
| Client_IPAddress | L’adresse IP du terminal utilisée lorsque l’activité a été enregistrée. L’adresse IP s’affiche au format d’adresse IPv4 ou IPv6. |
| ClientMachineName | Le nom de l’ordinateur qui héberge le client Outlook. |
| ClientProcessName | Le client de messagerie utilisé pour accéder à la boîte aux lettres. |
| ClientVersion | La version du client de messagerie. |
| InternalLogonType | Réservé à un usage interne. |
| Logon_Type | Indique le type d’utilisateur qui a accédé à la boîte aux lettres et effectué l’opération qui a été enregistrée. |
| LogonUserDisplayName |     Le nom convivial de l’utilisateur qui a effectué l’opération. |
| LogonUserSid | Le SID de l’utilisateur qui a effectué l’opération. |
| MailboxGuid | Le GUID Exchange de la boîte aux lettres ouverte. |
| MailboxOwnerMasterAccountSid | SID de compte principal du compte du propriétaire de boîte aux lettres. |
| MailboxOwnerSid | Le SID du propriétaire de la boîte aux lettres. |
| MailboxOwnerUPN | L’adresse de messagerie de la personne qui possède la boîte ouverte. |


### <a name="exchange-mailbox-audit"></a>Audit de la boîte aux lettres Exchange

Ces enregistrements sont créés lors de la création d’une entrée d’audit de boîte aux lettres.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| Élément | Représente l’élément sur lequel l’opération a été effectuée | 
| SendAsUserMailboxGuid | Le GUID Exchange de la boîte aux lettres ouverte pour envoyer un e-mail. |
| SendAsUserSmtp | Adresse SMTP de l’utilisateur dont le nom a été emprunté. |
| SendonBehalfOfUserMailboxGuid | Le GUID Exchange de la boîte aux lettres ouverte pour envoyer un e-mail au nom de. |
| SendOnBehalfOfUserSmtp | Adresse SMTP de l’utilisateur au nom duquel le courrier électronique est envoyé. |


### <a name="exchange-mailbox-audit-group"></a>Groupe d’audit de la boîte aux lettres Exchange

Ces enregistrements sont créés lorsque des modifications ou des ajouts sont apportés aux groupes Exchange.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | Exchange |
| OfficeWorkload | ExchangeItemGroup |
| AffectedItems | Informations sur chaque élément dans le groupe. |
| CrossMailboxOperations | Indique si l’opération a impliqué plusieurs boîtes aux lettres. |
| DestMailboxId | Définie uniquement si le paramètre CrossMailboxOperations a la valeur True. Spécifie le GUID de la boîte aux lettres cible. |
| DestMailboxOwnerMasterAccountSid | Définie uniquement si le paramètre CrossMailboxOperations a la valeur True. Spécifie le SID pour le SID du compte principal du propriétaire de la boîte aux lettres cible. |
| DestMailboxOwnerSid | Définie uniquement si le paramètre CrossMailboxOperations a la valeur True. Spécifie le SID de la boîte aux lettres cible. |
| DestMailboxOwnerUPN | Définie uniquement si le paramètre CrossMailboxOperations a la valeur True. Spécifie le nom UPN du propriétaire de la boîte aux lettres cible. |
| DestFolder | Le dossier de destination, pour les opérations telles que Déplacer. |
| Dossier | Le dossier où se trouve un groupe d’éléments. |
| Dossiers |     Plus d’informations sur les dossiers sources impliqués dans une opération ; par exemple, si les dossiers sont sélectionnés, puis supprimés. |


### <a name="sharepoint-base"></a>SharePoint Base

Ces propriétés sont communes à tous les enregistrements SharePoint.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| EventSource | Identifie qu’un événement s’est produit dans SharePoint. Les valeurs possibles sont ObjectModel ou SharePoint. |
| itemType | Le type d’objet qui a été ouvert ou modifié. Consultez la table ItemType pour plus d’informations sur les types d’objets. |
| MachineDomainInfo | Informations sur les opérations de synchronisation de terminal. Ces informations sont enregistrées uniquement si elles sont présente dans la requête. |
| MachineId |     Informations sur les opérations de synchronisation de terminal. Ces informations sont enregistrées uniquement si elles sont présente dans la requête. |
| Site_ | Le GUID du site où se trouve le fichier ou le dossier ouvert par l’utilisateur. |
| Source_Name | L’entité qui a déclenché l’opération d’audit. Les valeurs possibles sont ObjectModel ou SharePoint. |
| UserAgent | Informations sur le client ou le navigateur de l’utilisateur. Ces informations sont fournies par le client ou le navigateur. |


### <a name="sharepoint-schema"></a>Schéma de SharePoint

Ces enregistrements sont créés lorsque des modifications de configuration sont apportées à SharePoint.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| CustomEvent | Chaîne facultative pour les événements personnalisés. |
| Event_Data |     Charge facultative pour les événements personnalisés. |
| ModifiedProperties | La propriété est incluse pour les événements d’administrateur, tels que l’ajout d’un utilisateur en tant que membre d’un site ou un groupe administrateur de collection de sites. La propriété inclut le nom de la propriété modifiée (par exemple, le groupe d’administrateurs de site), la nouvelle valeur de la propriété modifiée (l’utilisateur ajouté en tant qu’un administrateur de site) et la valeur précédente de l’objet modifié. |


### <a name="sharepoint-file-operations"></a>Opérations sur les fichiers SharePoint

Ces enregistrements sont créés en réponse aux opérations de fichiers dans SharePoint.

| Propriété | Description |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePointFileOperation |
| DestinationFileExtension | L’extension de fichier d’un fichier qui est copié ou déplacé. Cette propriété s’affiche uniquement pour les événements FileCopied et FileMoved. |
| DestinationFileName | Le nom du fichier qui est copié ou déplacé. Cette propriété s’affiche uniquement pour les événements FileCopied et FileMoved. |
| DestinationRelativeUrl | L’URL du dossier de destination dans lequel un fichier est copié ou déplacé. La combinaison des valeurs pour les paramètres SiteURL, DestinationRelativeURL et DestinationFileName est identique à la valeur de la propriété ObjectID, qui est le nom de chemin d’accès complet du fichier qui a été copié. Cette propriété s’affiche uniquement pour les événements FileCopied et FileMoved. |
| SharingType | Le type d’autorisations de partage attribuées à l’utilisateur avec lequel la ressource a été partagée. Cet utilisateur est identifié par le paramètre UserSharedWith. |
| Site_Url | L’URL du site où se trouve le fichier ou le dossier ouvert par l’utilisateur. |
| SourceFileExtension | L’extension de fichier du fichier qui est ouvert par l’utilisateur. Cette propriété est vide si l’objet a été ouvert est un dossier. |
| SourceFileName |     Le nom du fichier ou du dossier ouvert par l’utilisateur. |
| SourceRelativeUrl | L’URL du dossier qui contient le fichier ouvert par l’utilisateur. La combinaison des valeurs pour les paramètres SiteURL, DestinationRelativeURL et DestinationFileName est identique à la valeur de la propriété ObjectID, qui est le nom de chemin d’accès complet du fichier ouvert par l’utilisateur. |
| UserSharedWith |     L’utilisateur avec lequel une ressource a été partagée. |




## <a name="sample-log-queries"></a>Exemples de requêtes de journal

Le tableau suivant fournit des exemples de requêtes dans les journaux d’enregistrements de mise à jour collectés par cette solution.

| Requête | Description |
| --- | --- |
|Nombre de toutes les opérations sur votre abonnement Office 365 |OfficeActivity &#124; summarize count() by Operation |
|Utilisation des sites SharePoint|OfficeActivity &#124; où OfficeWorkload =~ "sharepoint" &#124; summarize count() by SiteUrl \| sort by Count asc|
|Opérations d’accès de fichier par type d’utilisateur | OfficeActivity &#124; summarize count() by UserType |
|Analyser des actions externes sur Exchange|OfficeActivity &#124; where OfficeWorkload =~ "exchange" and ExternalAccess == true|



## <a name="next-steps"></a>Étapes suivantes

* Utilisez les [requêtes de journaux dans Azure Monitor](../log-query/log-query-overview.md) pour afficher des données détaillées sur les mises à jour.
* [Créez vos propres tableaux de bord](../learn/tutorial-logs-dashboards.md) pour afficher vos requêtes de recherche favoris Office 365.
* [Créez des alertes](../platform/alerts-overview.md) pour être informé de façon proactive des activités importantes Office 365.  
