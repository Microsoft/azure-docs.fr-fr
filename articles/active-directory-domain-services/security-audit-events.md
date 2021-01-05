---
title: Activer les audits de sécurité pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment activer les audits de sécurité pour centraliser la journalisation des événements à des fins d’analyse et de gestion des alertes dans Azure AD Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 662362c3-1a5e-4e94-ae09-8e4254443697
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 13bdc8797af8facaa73d3e43ecfbe504a6bd1dc2
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618873"
---
# <a name="enable-security-audits-for-azure-active-directory-domain-services"></a>Activer les audits de sécurité pour Azure Active Directory Domain Services

Les audits de sécurité Azure Active Directory Domain Services (Azure AD DS) permettent à Azure de diffuser en continu les événements de sécurité aux ressources ciblées. Ces ressources incluent le Stockage Microsoft Azure, les espaces de travail Azure Log Analytics ou Azure Event Hub. Après avoir activé les événements d’audit de sécurité, le service de domaine Azure AD DS envoie tous les événements audités pour la catégorie sélectionnée à la ressource ciblée.

Vous pouvez archiver les événements dans les événements de diffusion en continu et de Stockage Microsoft Azure à un logiciel SIEM (Security Information and Event Management) (ou équivalent) à l’aide d’Azure Event Hubs ou effectuer votre propre analyse et utiliser des espaces de travail Log Analytics, à partir du Portail Microsoft Azure.

> [!IMPORTANT]
> Les audits de sécurité Azure AD DS sont disponibles uniquement pour les domaine gérés basés sur Azure Resource Manager. Pour en savoir plus sur la migration, voir [Migrer Azure Active Directory Domain Services depuis le modèle de réseau virtuel classique vers Resource Manager][migrate-azure-adds].

## <a name="security-audit-destinations"></a>Destinations des audits de sécurité

Vous pouvez utiliser des espaces de travail Stockage Azure, Azure Event Hubs ou Azure Log Analytics comme ressources cibles pour les audits de sécurité Azure Active Directory Domain Services. Et vous pouvez combiner ces destinations. Par exemple, vous pouvez un espace de travail Stockage Azure pour l’archivage d’événements d’audit de sécurité, et un espace de travail Azure Log Analytics pour analyser et créer des rapport sur les informations à court terme.

La table suivante présente les scénarios pour chaque type de ressource de destination.

> [!IMPORTANT]
> Vous devez créer la ressource cible avant d’activer les audits de sécurité Azure Active Directory Domain Services. Vous pouvez créer ces ressources en utilisant le Portail Microsoft Azure, Azure PowerShell ou Azure CLI.

| Ressource cible | Scénario |
|:---|:---|
|Stockage Azure| Utilisez cette cible si votre besoin principal repose sur le stockage des événements d’audit de sécurité à des fins d’archivage. Les autres cibles peuvent être utilisées à des fins d’archivage ; toutefois, ces cibles fournissent des fonctionnalités au-delà du besoin principal d’archivage. <br /><br />Avant d’activer les événements d’audit de sécurité Azure AD DS, vous devez d’abord [créer un compte de stockage Azure](../storage/common/storage-account-create.md).|
|Hubs d'événements Azure| Utilisez cette cible si votre besoin principal repose sur le partage des événements d’audit de sécurité avec un autre logiciel tel qu’un logiciel d’analyse de données ou un logiciel SIEM (Security Information and Event Management).<br /><br />Avant d’activer les événements d’audit de sécurité Azure AD DS, [créez un hub Event Hub avec le Portail Microsoft Azure](../event-hubs/event-hubs-create.md).|
|Espace de travail Azure Log Analytics| Utilisez cette cible si votre besoin principal repose sur l’analyse et le passage en revue des audits de sécurité directement à partir du Portail Microsoft Azure.<br /><br />Avant d’activer les événements d’audit de sécurité Azure AD DS, [créez un espace de travail Log Analytics dans le Portail Microsoft Azure](../azure-monitor/learn/quick-create-workspace.md).|

## <a name="enable-security-audit-events-using-the-azure-portal"></a>Activer des événements d’audit de sécurité à l’aide du Portail Microsoft Azure

Pour activer les événements d’audit de sécurité Azure AD DS à l’aide du Portail Microsoft Azure, procédez comme suit.

> [!IMPORTANT]
> Les audits de sécurité Azure AD DS ne sont pas rétroactifs. Vous ne pouvez pas récupérer ou répéter des événements du passé. Le service Active Directory Domain Services ne peut envoyer que des événements qui se produisent une fois les audits de sécurité activés.

1. Connectez-vous au portail Azure sur https://portal.azure.com.
1. Sur le Portail Microsoft Azure, recherchez et sélectionnez **Azure AD Domain Services**. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans la fenêtre Azure AD DS, sélectionnez **Paramètres de diagnostic** sur la partie gauche.
1. Aucun diagnostic n’est configuré par défaut. Pour vous lancer, sélectionnez **Ajouter le paramètre de diagnostic**.

    ![Ajouter un paramètre de diagnostic pour Azure AD Domain Services](./media/security-audit-events/add-diagnostic-settings.png)

1. Saisissez un nom pour la configuration de diagnostic, par exemple *aadds-audit*.

    Cochez la case correspondant à la destination de l’audit de sécurité de votre choix. Vous pouvez choisir un compte de Stockage Microsoft Azure, un hub Azure Event Hub ou un espace de travail Log Analytics. Ces ressources de destination doivent déjà figurer dans votre abonnement Azure. Vous ne pouvez pas créer de ressources de destination dans cet assistant.

    ![Activer la destination et le type d’événements d’audit requis pour la capture](./media/security-audit-events/diagnostic-settings-page.png)

    * **Azure Storage**
        * Sélectionnez **Archiver dans un compte de stockage**, puis **Configurer**.
        * Sélectionnez l’**abonnement** et le **compte de stockage** que vous souhaitez utiliser pour archiver les événements d’audit de sécurité.
        * Lorsque vous êtes prêt, sélectionnez **OK**.
    * **Hubs Azure Event Hub**
        * Sélectionnez **Diffuser vers Event Hub**, puis choisissez **Configurer**.
        * Sélectionnez **l’abonnement** et **l’espace de noms Event Hub**. Si nécessaire, choisissez également un **nom de hub Event Hub**, puis un **nom de stratégie Event Hub**.
        * Lorsque vous êtes prêt, sélectionnez **OK**.
    * **Espaces de travail Azure Log Analytics**
        * Sélectionnez **Envoyer à Log Analytics**, puis choisissez **l’abonnement** et **l’espace de travail Log Analytics** que vous souhaitez utiliser pour stocker les événements d’audit de sécurité.

1. Sélectionnez les catégories de journaux à inclure pour la ressource cible particulière. Si vous envoyez les événements d’audit à un compte de Stockage Microsoft Azure, vous pouvez également configurer une stratégie de rétention qui définisse le nombre de jours pendant lesquels conserver les données. Un paramètre par défaut de *0* conserve toutes les données et ne fait pas pivoter les événements après un certain laps de temps.

    Vous pouvez sélectionner différentes catégories de journaux pour chaque ressource cible dans une même configuration. Cela vous permet par exemple de choisir les catégories de journaux à conserver pour Log Analytics, ainsi que les catégories de journaux à archiver.

1. Cela fait, sélectionnez **Enregistrer** pour valider vos modifications. Les ressources cibles commencent à recevoir des événements d’audit de sécurité Azure AD DS peu après l’enregistrement de la configuration.

## <a name="enable-security-audit-events-using-azure-powershell"></a>Activation des événements d’audit de sécurité via Azure PowerShell

Pour activer les événements d’audit de sécurité Azure AD DS à l’aide d’Azure PowerShell, procédez comme suit. Si nécessaire, commencez par [installer le module Azure PowerShell et vous connecter à votre abonnement Azure](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Les audits de sécurité Azure AD DS ne sont pas rétroactifs. Vous ne pouvez pas récupérer ou répéter des événements du passé. Le service Active Directory Domain Services ne peut envoyer que des événements qui se produisent une fois les audits de sécurité activés.

1. Connectez-vous à votre abonnement Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount). À l’invite, saisissez vos informations d’identification de compte.

    ```azurepowershell
    Connect-AzAccount
    ```

1. Créez la ressource cible pour les événements d’audit de sécurité.

    * **Stockage Microsoft Azure** - [Créer un compte de stockage avec Azure PowerShell](../storage/common/storage-account-create.md?tabs=azure-powershell)
    * **Azure Event Hub** - [Créer un hub Event Hub à l’aide d’Azure PowerShell](../event-hubs/event-hubs-quickstart-powershell.md). Vous devrez peut-être également utiliser la cmdlet [New-AzEventHubAuthorizationRule](/powershell/module/az.eventhub/new-azeventhubauthorizationrule) pour créer une règle d’autorisation qui accorde à Azure Active Directory Domain Services des autorisations d’accès à *l’espace de noms* Event Hub. La règle d’autorisation doit inclure les droits **Manage** (Gérer), **Listen** (Écouter) et **Send** (Envoyer).

        > [!IMPORTANT]
        > Vérifiez que vous définissez la règle d’autorisation sur l’espace de noms du hub Event Hub, et non sur le hub lui-même.

    * **Espaces de travail Azure Log Analytics** - [Créer un espace de travail Log Analytics avec Azure PowerShell](../azure-monitor/platform/powershell-workspace-configuration.md).

1. Récupérez l’ID de ressource de votre domaine Azure AD DS géré à l’aide de la cmdlet [Get-AzResource](/powershell/module/Az.Resources/Get-AzResource). Créez une variable nommée *$aadds.ResourceId* pour contenir la valeur :

    ```azurepowershell
    $aadds = Get-AzResource -name aaddsDomainName
    ```

1. Configurez les paramètres de diagnostic Azure à l’aide de la cmdlet [Set-AzDiagnosticSetting](/powershell/module/Az.Monitor/Set-AzDiagnosticSetting) afin d’utiliser la ressource cible pour les événements d’audit de sécurité Azure AD Domain Services. Dans les exemples suivants, la variable *$aadds.ResourceId* de l’étape précédente est utilisée.

    * **Stockage Microsoft Azure** : remplacez *storageAccountId* par le nom de votre compte de stockage :

        ```powershell
        Set-AzDiagnosticSetting `
            -ResourceId $aadds.ResourceId `
            -StorageAccountId storageAccountId `
            -Enabled $true
        ```

    * **Azure Event Hubs** : remplacez *eventHubName* par le nom de votre hub Event Hub, et *eventHubRuleId* par l’ID de votre règle d’autorisation :

        ```powershell
        Set-AzDiagnosticSetting -ResourceId $aadds.ResourceId `
            -EventHubName eventHubName `
            -EventHubAuthorizationRuleId eventHubRuleId `
            -Enabled $true
        ```

    * **Espaces de travail Azure Log Analytics** : remplacez *workspaceId* par l’ID de l’espace de travail Log Analytics :

        ```powershell
        Set-AzureRmDiagnosticSetting -ResourceId $aadds.ResourceId `
            -WorkspaceID workspaceId `
            -Enabled $true
        ```

## <a name="query-and-view-security-audit-events-using-azure-monitor"></a>Interroger et afficher les événements d’audit de sécurité à l’aide d’Azure Monitor

Les espaces de travail log Analytics vous permettent d’afficher et d’analyser les événements d’audit de sécurité à l’aide d’Azure Monitor et du langage de requête Kusto. Ce langage de requête est conçu pour être utilisé en lecture seule, offrant des capacités d’analyse puissantes et une syntaxe facile à lire. Pour en savoir plus sur la prise en main des langages de requête Kusto, consultez les articles suivants :

* [Documentation Azure Monitor](../azure-monitor/index.yml)
* [Prise en main de Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md)
* [Bien démarrer avec les requêtes de journal Azure Monitor](../azure-monitor/log-query/get-started-queries.md).
* [Créer et partager des tableaux de bord de données Log Analytics](../azure-monitor/learn/tutorial-logs-dashboards.md)

Les exemples de requêtes suivants peuvent être utilisés pour lancer l’analyse des événements d’audit de sécurité à partir de Microsoft Azure AD DS.

### <a name="sample-query-1"></a>Exemple de requête 1

Affichez tous les événements de verrouillage de compte au cours des sept derniers jours :

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= ago(7d)
| where OperationName has "4740"
```

### <a name="sample-query-2"></a>Exemple de requête 2

Affichez tous les événements de verrouillage de compte (*4740*) survenus entre le 3 juin 2020 à 9 heures et le 10 juin 2020 à minuit, triés par ordre croissant de date et d’heure :

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= datetime(2020-06-03 09:00) and TimeGenerated <= datetime(2020-06-10)
| where OperationName has "4740"
| sort by TimeGenerated asc
```

### <a name="sample-query-3"></a>Exemple de requête 3

Affichez les événements de connexion de compte survenus sept jours plus tôt pour l’utilisateur de compte nommé :

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-4"></a>Exemple de requête 4

Affichez les événements de connexion de compte survenus sept jours plus tôt pour l’utilisateur de compte nommé qui a tenté de se connecter à l’aide d’un mot de passe incorrect (*0xC0000006a*) :

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc000006a" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-5"></a>Exemple de requête 5

Affichez les événements de connexion de compte survenus sept jours plus tôt pour l’utilisateur de compte nommé qui a tenté de se connecter alors que le compte était verrouillé (*0xC0000234*) :

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-6"></a>Exemple de requête 6

Affichez le nombre d’événements de connexion de compte survenus sept jours plus tôt pour toutes les tentatives de connexion qui se sont produites pour tous les utilisateurs verrouillés :

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| summarize count()
```

## <a name="audit-event-categories"></a>Catégories d’événements d’audit

Les audits de sécurité Azure AD DS s’alignent sur l’audit traditionnel des contrôleurs de domaine AD DS traditionnels. Dans les environnements hybrides, la réutilisation de modèles d’audit existants garantit que la même logique peut être utilisée lors de l’analyse des événements. Selon le scénario que vous devez dépanner ou analyser, différentes catégories d’événements d’audit doivent être ciblées.

Les catégories d’événements d’audit suivantes sont disponibles :

| Nom de la catégorie d’audit | Description |
|:---|:---|
| Connexion de compte|Audite les tentatives d’authentification de données de compte sur un contrôleur de domaine ou sur un gestionnaire de comptes de sécurité local.</p>Les événements et les paramètres de stratégie d’ouverture et de fermeture de session suivent les tentatives d’accès à un ordinateur particulier. Les paramètres et les événements de cette catégorie se concentrent sur la base de données de compte qui est utilisée. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer la validation des informations d’identification](/windows/security/threat-protection/auditing/audit-credential-validation)</li><li>[Auditer le service d’authentification Kerberos](/windows/security/threat-protection/auditing/audit-kerberos-authentication-service)</li><li>[Auditer les opérations de ticket du service Kerberos](/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations)</li><li>[Auditer d’autres événements d’ouverture/fermeture de session](/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li></ul>|
| Gestion de compte|Audite les modifications apportées aux groupes et comptes d’utilisateurs et d’ordinateurs. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer la gestion des groupes d’applications](/windows/security/threat-protection/auditing/audit-application-group-management)</li><li>[Auditer la gestion des comptes d’ordinateur](/windows/security/threat-protection/auditing/audit-computer-account-management)</li><li>[Auditer la gestion des groupes de distribution](/windows/security/threat-protection/auditing/audit-distribution-group-management)</li><li>[Auditer d’autres événements d’ouverture de session](/windows/security/threat-protection/auditing/audit-other-account-management-events)</li><li>[Auditer la gestion des groupes de sécurité](/windows/security/threat-protection/auditing/audit-security-group-management)</li><li>[Auditer la gestion des comptes d’utilisateur](/windows/security/threat-protection/auditing/audit-user-account-management)</li></ul>|
| Suivi des détails|Audit les activités des applications et utilisateurs individuels sur l’ordinateur concerné et le fonctionnement de cet ordinateur. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer l’activité DPAPI](/windows/security/threat-protection/auditing/audit-dpapi-activity)</li><li>[Auditer l’activité Plug-and-Play](/windows/security/threat-protection/auditing/audit-pnp-activity)</li><li>[Auditer la création du processus](/windows/security/threat-protection/auditing/audit-process-creation)</li><li>[Auditer la fin du processus](/windows/security/threat-protection/auditing/audit-process-termination)</li><li>[Auditer les événements RPC](/windows/security/threat-protection/auditing/audit-rpc-events)</li></ul>|
| Accès aux services d’annuaire|Audite les tentatives d’accès aux objets dans Active Directory Domain Services (AD DS) et de modification de ces derniers. Ces événements d’audit sont journalisés uniquement sur des contrôleurs de domaine. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer la réplication du service d’annuaire détaillé](/windows/security/threat-protection/auditing/audit-detailed-directory-service-replication)</li><li>[Auditer l’accès au service d’annuaire](/windows/security/threat-protection/auditing/audit-directory-service-access)</li><li>[Auditer les modifications du service d’annuaire](/windows/security/threat-protection/auditing/audit-directory-service-changes)</li><li>[Auditer la réplication du service d’annuaire](/windows/security/threat-protection/auditing/audit-directory-service-replication)</li></ul>|
| Ouverture/fermeture de session|Audite les tentatives d’ouverture de session sur un ordinateur de manière interactive ou par le biais d’un réseau. Ces événements sont utiles pour le suivi de l’activité de l’utilisateur et l’identification des attaques potentielles sur les ressources réseau. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer le verrouillage du compte](/windows/security/threat-protection/auditing/audit-account-lockout)</li><li>[Auditer les revendications utilisateur/de périphérique](/windows/security/threat-protection/auditing/audit-user-device-claims)</li><li>[Auditer le mode étendu IPsec](/windows/security/threat-protection/auditing/audit-ipsec-extended-mode)</li><li>[Auditer l’appartenance à un groupe](/windows/security/threat-protection/auditing/audit-group-membership)</li><li>[Auditer le mode principal IPsec](/windows/security/threat-protection/auditing/audit-ipsec-main-mode)</li><li>[Auditer le mode rapide IPsec](/windows/security/threat-protection/auditing/audit-ipsec-quick-mode)</li><li>[Auditer la fermeture de session](/windows/security/threat-protection/auditing/audit-logoff)</li><li>[Auditer l’ouverture de session](/windows/security/threat-protection/auditing/audit-logon)</li><li>[Auditer le serveur NPS (Network Policy Server)](/windows/security/threat-protection/auditing/audit-network-policy-server)</li><li>[Auditer d’autres événements d’ouverture/fermeture de session](/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li><li>[Auditer l’ouverture de session spéciale](/windows/security/threat-protection/auditing/audit-special-logon)</li></ul>|
|Accès aux objets| Audite les tentatives d’accès à des objets ou types d’objets spécifiques sur un ordinateur ou réseau. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer l’application générée](/windows/security/threat-protection/auditing/audit-application-generated)</li><li>[Auditer les services de certification](/windows/security/threat-protection/auditing/audit-certification-services)</li><li>[Auditer le partage de fichiers détaillé](/windows/security/threat-protection/auditing/audit-detailed-file-share)</li><li>[Auditer le partage de fichiers](/windows/security/threat-protection/auditing/audit-file-share)</li><li>[Auditer le système de fichiers](/windows/security/threat-protection/auditing/audit-file-system)</li><li>[Auditer la connexion de la plateforme de filtrage](/windows/security/threat-protection/auditing/audit-filtering-platform-connection)</li><li>[Auditer le rejet de paquet par la plateforme de filtrage](/windows/security/threat-protection/auditing/audit-filtering-platform-packet-drop)</li><li>[Auditer la manipulation de handle](/windows/security/threat-protection/auditing/audit-handle-manipulation)</li><li>[Auditer l’objet de noyau](/windows/security/threat-protection/auditing/audit-kernel-object)</li><li>[Auditer d’autres événements d’accès à l’objet](/windows/security/threat-protection/auditing/audit-other-object-access-events)</li><li>[Auditer le Registre](/windows/security/threat-protection/auditing/audit-registry)</li><li>[Auditer le stockage amovible](/windows/security/threat-protection/auditing/audit-removable-storage)</li><li>[Auditer SAM](/windows/security/threat-protection/auditing/audit-sam)</li><li>[Auditer la stratégie d’accès centralisée intermédiaire](/windows/security/threat-protection/auditing/audit-central-access-policy-staging)</li></ul>|
|Modification de stratégie|Audite les modifications apportées aux stratégies de sécurité importantes sur un réseau ou un système local. Les stratégies sont généralement établies par les administrateurs pour faciliter la sécurisation des ressources réseau. La supervision des modifications ou des tentatives de modification de ces stratégies peut être un aspect important de la gestion de la sécurité pour un réseau. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer la modification de la stratégie d’audit](/windows/security/threat-protection/auditing/audit-audit-policy-change)</li><li>[Auditer la modification de stratégie d’authentification](/windows/security/threat-protection/auditing/audit-authentication-policy-change)</li><li>[Auditer la modification de la stratégie d’autorisation](/windows/security/threat-protection/auditing/audit-authorization-policy-change)</li><li>[Auditer la modification de la stratégie de plateforme de filtrage](/windows/security/threat-protection/auditing/audit-filtering-platform-policy-change)</li><li>[Auditer la modification de la stratégie de niveau règle MPSSVC](/windows/security/threat-protection/auditing/audit-mpssvc-rule-level-policy-change)</li><li>[Auditer d’autres événements de modification de stratégie](/windows/security/threat-protection/auditing/audit-other-policy-change-events)</li></ul>|
|Utilisation des privilèges| Audite l’utilisation de certaines autorisations sur un ou plusieurs systèmes. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer l’utilisation de privilèges non sensibles](/windows/security/threat-protection/auditing/audit-non-sensitive-privilege-use)</li><li>[Auditer l’utilisation de privilèges sensibles](/windows/security/threat-protection/auditing/audit-sensitive-privilege-use)</li><li>[Auditer d’autres événements d’utilisation de privilèges](/windows/security/threat-protection/auditing/audit-other-privilege-use-events)</li></ul>|
|Système| Audite les modifications de niveau système apportées à un ordinateur non incluses dans les autres catégories et qui ont des implications de sécurité potentielles. Cette catégorie comprend les sous-catégories suivantes :<ul><li>[Auditer le pilote IPSEC](/windows/security/threat-protection/auditing/audit-ipsec-driver)</li><li>[Auditer d’autres événements système](/windows/security/threat-protection/auditing/audit-other-system-events)</li><li>[Auditer la modification de l’état de la sécurité](/windows/security/threat-protection/auditing/audit-security-state-change)</li><li>[Auditer l’extension du système de sécurité](/windows/security/threat-protection/auditing/audit-security-system-extension)</li><li>[Auditer l’intégrité du système](/windows/security/threat-protection/auditing/audit-system-integrity)</li></ul>|

## <a name="event-ids-per-category"></a>ID d’événements par catégorie

 Les audits de sécurité Azure AD DS enregistrent les ID d’événement suivants lorsque l’action spécifique déclenche un événement pouvant être audité :

| Nom de la catégorie d’événements | ID d’événement |
|:---|:---|
|Sécurité de la connexion de compte|4767, 4774, 4775, 4776, 4777|
|Sécurité de la gestion des comptes|4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4730, 4731, 4732, 4733, 4734, 4735, 4737, 4738, 4740, 4741, 4742, 4743, 4754, 4755, 4756, 4757, 4758, 4764, 4765, 4766, 4780, 4781, 4782, 4793, 4798, 4799, 5376, 5377|
|Sécurité du suivi des détails|None|
|Sécurité de l’accès DS|5136, 5137, 5138, 5139, 5141|
|Sécurité de l’ouverture/fermeture de session|4624, 4625, 4634, 4647, 4648, 4672, 4675, 4964|
|Sécurité de l’accès aux objets|None|
|Sécurité de la modification de stratégie|4670, 4703, 4704, 4705, 4706, 4707, 4713, 4715, 4716, 4717, 4718, 4719, 4739, 4864, 4865, 4866, 4867, 4904, 4906, 4911, 4912|
|Sécurité de l’utilisation des privilèges|4985|
|Sécurité système|4612, 4621|

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Kusto, consultez les articles suivants :

* [Vue d’ensemble](/azure/kusto/query/) du langage de requête Kusto.
* [Tutoriel Kusto](/azure/kusto/query/tutorial) pour vous familiariser avec les principes fondamentaux des requêtes.
* [Exemples de requêtes](/azure/kusto/query/samples) qui vous aident à découvrir de nouvelles façons de voir vos données.
* [Meilleures pratiques](/azure/kusto/query/best-practices) de Kusto pour optimiser vos requêtes.

<!-- LINKS - Internal -->
[migrate-azure-adds]: migrate-from-classic-vnet.md