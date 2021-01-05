---
title: Gérer un compte d'identification Azure Automation
description: Cet article décrit comment gérer votre compte d’identification avec PowerShell ou à partir du portail Azure.
services: automation
ms.subservice: shared-capabilities
ms.date: 09/28/2020
ms.topic: conceptual
ms.openlocfilehash: 0849eb0c421883ecb0510451ff81b604538c9cc3
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92069889"
---
# <a name="manage-an-azure-automation-run-as-account"></a>Gérer un compte d'identification Azure Automation

Les comptes d’identification d’Azure Automation assurent l’authentification pour la gestion des ressources sur le modèle de déploiement Azure Resource Manager ou Azure Classic à l’aide de runbooks Automation et d’autres fonctionnalités Automation. Cet article aide à gérer un compte d’identification ou un compte d’identification Classic.

Pour plus d’informations sur l’authentification des comptes Azure Automation et des conseils relatifs aux scénarios d’automatisation des processus, consultez [Vue d’ensemble de l’authentification des comptes Automation](automation-security-overview.md).

## <a name="run-as-account-permissions"></a><a name="permissions"></a>Autorisations de compte d’identification

Cette section définit les autorisations pour les comptes d’identification standard et les comptes d’identification Classic.

Pour créer ou mettre à jour un compte d’identification, vous devez disposer d’autorisations et de privilèges spécifiques. Un administrateur d’application dans Azure Active Directory et un propriétaire dans un abonnement peuvent effectuer toutes les tâches. Dans une situation où les responsabilités sont partagées, le tableau suivant donne une liste des tâches, avec les applets de commande équivalentes et les autorisations nécessaires :

|Tâche|Applet de commande  |Autorisations minimales  |Où vous définissez les autorisations|
|---|---------|---------|---|
|Créer une application Azure AD|[New-AzADApplication](/powershell/module/az.resources/new-azadapplication)     | Rôle de développeur d’applications<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Accueil > Azure AD > Inscriptions d’applications |
|Ajoutez les informations d’identification à l’application.|[New-AzADAppCredential](/powershell/module/az.resources/new-azadappcredential)     | Administrateur d’application ou administrateur général<sup>1</sup>         |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Accueil > Azure AD > Inscriptions d’applications|
|Créer et obtenir un principal du service Azure AD|[New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal)</br>[Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal)     | Administrateur d’application ou administrateur général<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>Accueil > Azure AD > Inscriptions d’applications|
|Attribuer ou obtenir le rôle Azure pour le principal spécifié|[New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment)</br>[Get-AzRoleAssignment](/powershell/module/Az.Resources/Get-AzRoleAssignment)      | Administrateur ou propriétaire de l’accès utilisateur, ou détenteur des autorisations suivantes :</br></br><code>Microsoft.Authorization/Operations/read</br>Microsoft.Authorization/permissions/read</br>Microsoft.Authorization/roleDefinitions/read</br>Microsoft.Authorization/roleAssignments/write</br>Microsoft.Authorization/roleAssignments/read</br>Microsoft.Authorization/roleAssignments/delete</code></br></br> | [Abonnement](../role-based-access-control/role-assignments-portal.md)</br>Accueil > Abonnements > \<subscription name\> - Contrôle d’accès (IAM)|
|Créer ou supprimer un certificat Automation|[New-AzAutomationCertificate](/powershell/module/Az.Automation/New-AzAutomationCertificate)</br>[Remove-AzAutomationCertificate](/powershell/module/az.automation/remove-azautomationcertificate)     | Contributeur sur le groupe de ressources         |Groupe de ressources du compte Automation|
|Créer ou supprimer une connexion Automation|[New-AzAutomationConnection](/powershell/module/az.automation/new-azautomationconnection)</br>[Remove-AzAutomationConnection](/powershell/module/az.automation/remove-azautomationconnection)|Contributeur sur le groupe de ressources |Groupe de ressources du compte Automation|

<sup>1</sup> Les utilisateurs non-administrateurs dans votre locataire Azure AD peuvent [inscrire des applications AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app) si l’option **Les utilisateurs peuvent inscrire des applications** du locataire Azure AD dans la page des Paramètres utilisateur a la valeur **Oui**. Si le paramètre d’inscription des applications est **Non**, l’utilisateur qui effectue cette action doit être tel que défini dans ce tableau.

Si vous n’êtes pas membre de l’instance Active Directory de l’abonnement avant d’être ajouté au rôle Administrateur général de l’abonnement, vous êtes ajouté en tant qu’invité. Dans ce cas, vous recevez un avertissement `You do not have permissions to create…` sur la page **Ajouter un compte Automation**.

Si vous êtes membre de l’instance Active Directory de l’abonnement sur laquelle le rôle Administrateur général est attribué, vous pouvez également recevoir un avertissement `You do not have permissions to create…` sur la page **Ajouter un compte Automation**. Dans ce cas, vous pouvez demander la suppression à partir de l’instance Active Directory de l’abonnement, puis demander à être ajouté de nouveau, afin de devenir un utilisateur à part entière dans Active Directory.

Pour vérifier que la situation produisant le message d’erreur a été corrigée :

1. Dans le volet Azure Active Directory du portail Azure, sélectionnez **Utilisateurs et groupes**.
2. Sélectionnez **Tous les utilisateurs**.
3. Choisissez votre nom, puis sélectionnez **Profil**.
4. Vérifiez que la valeur de l’attribut **Usertype** sous votre profil d’utilisateur n’est pas définie sur **Invité**.

### <a name="permissions-required-to-create-or-manage-classic-run-as-accounts"></a><a name="permissions-classic"></a>Autorisations nécessaires pour créer ou gérer des comptes d’identification Classic

Pour configurer ou renouveler des comptes d’identification Classic, vous devez disposer du rôle Coadministrateur au niveau de l’abonnement. Pour en savoir plus sur les autorisations d’abonnement Classic, consultez [Administrateurs d’abonnement Azure Classic](../role-based-access-control/classic-administrators.md#add-a-co-administrator).

## <a name="create-a-run-as-account-in-azure-portal"></a>Créer un compte d’identification dans le portail Azure

Effectuez les étapes suivantes pour mettre à jour votre compte Azure Automation dans le portail Azure. Créez individuellement les comptes d’identification standard et les comptes d’identification Classic. Si vous n’avez pas besoin de gérer des ressources classiques, vous pouvez simplement créer le compte d’identification Azure.

1. Connectez-vous au portail Azure avec un compte membre du rôle Administrateurs des abonnements et coadministrateur de l’abonnement.

2. Recherchez et sélectionnez **Comptes Automation**.

3. Dans la page Comptes Automation, sélectionnez votre compte Automation dans la liste.

4. Dans le volet gauche, sélectionnez **Comptes d’identification** dans la section **Paramètres de compte**.

    :::image type="content" source="media/manage-runas-account/automation-account-properties-pane.png" alt-text="Sélection de l’option Compte d’identification":::

5. Selon le compte dont vous avez besoin, choisissez le volet **+ Compte d’identification Azure** ou le volet **+ Compte d’identification Azure Classic**. Après avoir passé en revue les informations générales, cliquez sur **Créer**.

    :::image type="content" source="media/manage-runas-account/automation-account-create-runas.png" alt-text="Sélection de l’option Compte d’identification":::

6. Pour suivre la progression de la création du compte d’identification, accédez à l’onglet **Notifications** du menu. Une bannière s’affiche également indiquant que le compte est en cours de création. L’exécution de ce processus peut prendre plusieurs minutes.

## <a name="create-a-run-as-account-using-powershell"></a>Créer un compte d’identification avec PowerShell

La liste suivante indique la configuration nécessaire pour créer un compte d’identification dans PowerShell à l’aide d’un script fourni. Ces conditions requises s’appliquent aux deux types de comptes d’identification.

* Windows 10 ou Windows Server 2016 avec les modules Azure Resource Manager 3.4.1 et ultérieur. Le script PowerShell ne prend pas en charge les versions antérieures de Windows.
* Azure PowerShell 6.2.4 ou version ultérieure. Pour plus d’informations, consultez [Guide pratique pour installer et configurer Azure PowerShell](/powershell/azure/install-az-ps).
* Un compte Automation, qui est référencé comme valeur pour les paramètres `AutomationAccountName` et `ApplicationDisplayName`.
* Les autorisations équivalentes à celles qui sont listées dans [Autorisations nécessaires pour configurer des comptes d’identification](#permissions).

Pour obtenir les valeurs de `AutomationAccountName`, `SubscriptionId` et `ResourceGroupName`, qui sont des paramètres obligatoires pour le script PowerShell, procédez comme suit.

1. Dans le portail Azure, sélectionnez **Comptes Automation**.

1. Dans la page Comptes Automation, sélectionnez votre compte Automation.

1. Dans la section des paramètres de compte, sélectionnez **Propriétés**.

1. Notez les valeurs **Nom**, **ID d’abonnement** et **Groupe de ressources** sur la page **Propriétés**.

   ![Page des propriétés du compte Automation](media/manage-runas-account/automation-account-properties.png)

### <a name="powershell-script-to-create-a-run-as-account"></a>Script PowerShell pour créer un compte d’identification

Le script PowerShell comprend la prise en charge de plusieurs configurations.

* Créez un compte d’identification à l’aide d’un certificat auto-signé.
* Créez un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat auto-signé.
* Créer un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat émis par votre AC d’entreprise.
* Créez un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat auto-signé dans le cloud Azure Government.

1. Téléchargez et enregistrez le script dans un dossier local à l’aide de la commande suivante.

    ```powershell
    wget https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/AzRunAs/Create-RunAsAccount.ps1 -outfile Create-RunAsAccount.ps1
    ```

2. Lancez PowerShell avec des droits d’utilisateur élevés et accédez au dossier contenant le script.

3. Exécutez l’une des commandes suivantes pour créer un compte d’identification ou un compte d’identification Classic en fonction de vos besoins.

    * Créez un compte d’identification à l’aide d’un certificat auto-signé.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $false
        ```

    * Créez un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat auto-signé.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true
        ```

    * Créez un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat d’entreprise.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication>  -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnterpriseCertPathForRunAsAccount <EnterpriseCertPfxPathForRunAsAccount> -EnterpriseCertPlainPasswordForRunAsAccount <StrongPassword> -EnterpriseCertPathForClassicRunAsAccount <EnterpriseCertPfxPathForClassicRunAsAccount> -EnterpriseCertPlainPasswordForClassicRunAsAccount <StrongPassword>
        ```

        Si vous avez créé un compte d’identification Classic avec un certificat public d’entreprise (fichier .cer), utilisez ce certificat. Le script le crée et l’enregistre dans le dossier de fichiers temporaires sur votre ordinateur, sous le profil utilisateur `%USERPROFILE%\AppData\Local\Temp` que vous avez utilisé pour exécuter la session PowerShell. Consultez [Chargement d’un certificat d’API de gestion dans le portail Azure](../cloud-services/cloud-services-configure-ssl-certificate-portal.md).

    * Créer un compte d’identification standard et un compte d’identification Classic à l’aide d’un certificat auto-signé dans le cloud Azure Government

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnvironmentName AzureUSGovernment
        ```

4. Une fois le script exécuté, vous êtes invité à vous authentifier auprès d’Azure. Connectez-vous avec un compte membre du rôle Administrateurs des abonnements. Si vous créez un compte d’identification Classic, votre compte doit être coadministrateur de l’abonnement.

## <a name="delete-a-run-as-or-classic-run-as-account"></a>Supprimer un compte d’identification standard ou Classic

Cette section décrit comment supprimer un compte d’identification standard ou Classic. Lorsque vous effectuez cette opération, le compte Automation est conservé. Après avoir supprimé le compte d’identification, vous pouvez le recréer sur le Portail Azure ou avec le script PowerShell fourni.

1. Dans le portail Azure, ouvrez le compte Automation.

2. Dans le volet gauche, sélectionnez **Comptes d’identification** dans la section des paramètres de compte.

3. Dans la page de propriétés Comptes d’identification, sélectionnez le compte d’identification standard ou le compte d’identification Classic que vous voulez supprimer.

4. Dans le volet Propriétés, pour le compte sélectionné, cliquez sur **Supprimer**.

   ![Supprimer un compte d’identification](media/manage-runas-account/automation-account-delete-runas.png)

5. Pour suivre la progression de la suppression du compte, accédez à l’onglet **Notifications** du menu.

## <a name="renew-a-self-signed-certificate"></a><a name="cert-renewal"></a>Renouveler un certificat auto-signé

Le certificat auto-signé que vous avez créé pour le compte d’identification expire au bout d’un an après la date de création. Avant que votre compte d’identification n’expire, vous devez renouveler le certificat. Vous pouvez le renouveler à tout moment avant qu’il n’expire.

Lorsque vous renouvelez le certificat auto-signé, le certificat valide en cours est conservé afin de garantir que les Runbooks en file d’attente ou en cours d’exécution, qui s’authentifient avec le compte d’identification, ne sont pas affectés. Le certificat reste valide jusqu’à sa date d’expiration.

>[!NOTE]
>Si vous pensez que le compte d’identification a été compromis, vous pouvez supprimer et recréer le certificat auto-signé.

>[!NOTE]
>Si vous avez configuré votre compte d’identification pour utiliser un certificat émis par votre autorité de certification d’entreprise, et que vous utilisez l’option permettant de renouveler un certificat auto-signé, le certificat d’entreprise est remplacé par un certificat auto-signé.

Effectuez les étapes suivantes pour renouveler le certificat auto-signé.

1. Dans le portail Azure, ouvrez le compte Automation.

1. Sélectionnez **Comptes d’identification** dans la section des paramètres de compte.

    ![Panneau des propriétés du compte Automation](media/manage-runas-account/automation-account-properties-pane.png)

1. Dans la page de propriétés des Comptes d’identification, sélectionnez le compte d’identification standard ou le compte d’identification Classic pour lequel renouveler le certificat.

1. Dans le volet Propriétés, pour le compte sélectionné, cliquez sur **Renouveler le certificat**.

    ![Renouveler le certificat pour le compte d’identification](media/manage-runas-account/automation-account-renew-runas-certificate.png)

1. Pour suivre la progression du renouvellement du certificat, accédez à l’onglet **Notifications** du menu.

## <a name="limit-run-as-account-permissions"></a>Limiter les autorisations d’un compte d’identification

Pour contrôler le ciblage de l’automatisation sur des ressources dans Azure, vous pouvez exécuter le script [Update-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug8). Ce script modifie votre principal du service du compte d’identification existant pour créer et utiliser une définition de rôle personnalisée. Ce rôle dispose d’autorisations pour toutes les ressources, à l’exception de [Key Vault](../key-vault/index.yml).

>[!IMPORTANT]
>Après l’exécution du script **Update-AutomationRunAsAccountRoleAssignments.ps1**, les runbooks qui accèdent à Key Vault par le biais de l’utilisation de comptes d’identification ne fonctionnent plus. Avant d’exécuter le script, vous devez examiner les runbooks de votre compte pour repérer les appels à Azure Key Vault. Pour activer l’accès à Key Vault à partir des runbooks Azure Automation, vous devez [ajouter le compte d’identification aux autorisations de Key Vault](#add-permissions-to-key-vault).

Si vous devez restreindre davantage ce que le principal du service du compte d’identification peut faire, vous pouvez ajouter d’autres types de ressources à l’élément `NotActions` de la définition de rôle personnalisée. L’exemple suivant restreint l’accès à `Microsoft.Compute/*`. Si vous ajoutez ce type de ressource à l’élément `NotActions` de la définition de rôle, le rôle ne sera pas en mesure d’accéder à une ressource de calcul. Pour en savoir plus sur les définitions de rôle, consultez [Comprendre les définitions de rôle relatives aux ressources Azure](../role-based-access-control/role-definitions.md).

```powershell
$roleDefinition = Get-AzRoleDefinition -Name 'Automation RunAs Contributor'
$roleDefinition.NotActions.Add("Microsoft.Compute/*")
$roleDefinition | Set-AzRoleDefinition
```

Vous pouvez déterminer si le principal du service utilisé par votre compte d’identification se trouve dans la définition du rôle Contributeur ou dans une autre définition personnalisée.

1. Accédez à votre compte Automation, puis sélectionnez **Comptes d’identification** dans la section des paramètres de compte.
2. Sélectionnez **Compte d'identification Azure**.
3. Sélectionnez **Rôle** pour rechercher la définition de rôle en cours d’utilisation.

:::image type="content" source="media/manage-runas-account/verify-role.png" alt-text="Sélection de l’option Compte d’identification" lightbox="media/manage-runas-account/verify-role-expanded.png":::

Vous pouvez également déterminer la définition de rôle utilisée par les comptes d’identification pour plusieurs abonnements ou comptes Automation. Pour ce faire, utilisez le script [Check-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug5) dans PowerShell Gallery.

### <a name="add-permissions-to-key-vault"></a>Ajouter des autorisations à Key Vault

Vous pouvez autoriser Azure Automation à vérifier si Key Vault et votre principal du service du compte d’identification utilisent une définition de rôle personnalisée. Vous devez respecter les consignes suivantes :

* accorder les autorisations à Key Vault ;
* définir la stratégie d’accès.

Vous pouvez utiliser le script [Extend-AutomationRunAsAccountRoleAssignmentToKeyVault.ps1](https://aka.ms/AA5hugb) dans PowerShell Gallery pour donner à votre compte d’identification des autorisations sur Key Vault. Pour plus d’informations sur la définition des autorisations sur Key Vault, consultez [Attribuer une stratégie d'accès Key Vault](../key-vault/general/assign-access-policy-powershell.md).

## <a name="resolve-misconfiguration-issues-for-run-as-accounts"></a>Résoudre des problèmes de configuration incorrecte pour les comptes d’identification

Certains éléments de configuration nécessaires à un compte d’identification standard ou à un compte d’identification Classic ont peut-être été supprimés, ou n’ont pas été créés correctement lors de l’installation initiale. Les cas possibles de configuration incorrecte sont les suivants :

* Ressource de certificat
* Ressource de connexion
* Compte d’identification supprimé du rôle Contributeur
* Principal du service ou application dans Azure AD

Pour de tels cas de configuration incorrecte, le compte Automation détecte les modifications et affiche pour le compte l’état *Incomplet* dans le volet des propriétés des comptes d’identification.

![État Incomplet pour la configuration du compte d’identification](media/manage-runas-account/automation-account-runas-config-incomplete.png)

Lorsque vous sélectionnez le compte d’identification, le volet des propriétés du compte affiche le message d’erreur suivant :

```text
The Run As account is incomplete. Either one of these was deleted or not created - Azure Active Directory Application, Service Principal, Role, Automation Certificate asset, Automation Connect asset - or the Thumbprint is not identical between Certificate and Connection. Please delete and then re-create the Run As Account.
```

Vous pouvez rapidement résoudre ces problèmes liés au compte d’identification en supprimant et en recréant le compte d’identification.

## <a name="next-steps"></a>Étapes suivantes

* [Objets principal de service et d’application](../active-directory/develop/app-objects-and-service-principals.md).
* [Vue d’ensemble des certificats pour Azure Cloud Services](../cloud-services/cloud-services-certs-create.md).