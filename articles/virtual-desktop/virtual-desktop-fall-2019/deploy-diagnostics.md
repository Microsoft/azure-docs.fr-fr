---
title: Déployer l’outil de diagnostic pour Windows Virtual Desktop (classique) – Azure
description: Comment déployer l’outil de diagnostic de l’expérience utilisateur pour Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: how-to
ms.date: 12/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 9672d18fe99d8baae3c73fa3782531eb84114713
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97515634"
---
# <a name="deploy-the-windows-virtual-desktop-classic-diagnostics-tool"></a>Déployer l’outil de diagnostic Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager.

Voici ce que l’outil de diagnostic pour Windows Virtual Desktop peut faire pour vous :

- Recherchez des activités de diagnostic (gestion, connexion ou flux) pour un seul utilisateur sur une période d’une semaine.
- Rassemblez les informations sur l’hôte de session pour les activités de connexion à partir de votre espace de travail Log Analytics.
- Examinez les détails des performances des machines virtuelles pour un hôte particulier.
- Consultez quels utilisateurs sont connectés à l’hôte de session.
- Envoyez un message aux utilisateurs actifs sur un hôte de session spécifique.
- Déconnectez des utilisateurs d’un hôte de session.

## <a name="prerequisites"></a>Prérequis

Vous devez créer une inscription de l’application Azure Active Directory et un espace de travail Log Analytics avant de pouvoir déployer le modèle Azure Resource Manager pour l’outil. Vous ou l’administrateur avez besoin de ces autorisations pour effectuer cette opération :

- Propriétaire de l’abonnement Azure
- Autorisation de créer des ressources dans votre abonnement Azure
- Autorisation de créer une application Azure AD
- Droits de propriétaire ou contributeur aux Services Bureau à distance

Vous devez également installer ces deux modules PowerShell avant de commencer :

- [Module Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.4.0/&preserve-view=true)
- [Module Azure AD](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0/&preserve-view=true)

Assurez-vous que votre ID d’abonnement est prêt lorsque vous vous connectez.

Une fois que tout est en ordre, vous pouvez créer l’inscription de l’application Azure AD.

## <a name="create-an-azure-active-directory-app-registration"></a>Créer une inscription d’application Azure Active Directory

Cette section vous montre comment utiliser PowerShell pour créer l’application Azure Active Directory avec un principal du service et obtenir des autorisations d’API pour celle-ci.

>[!NOTE]
>Les autorisations d’API sont Windows Virtual Desktop, les autorisations d’API Log Analytics et Microsoft Graph sont ajoutées à l’application Azure Active Directory.

1. Ouvrez PowerShell en tant qu’administrateur.
2. Connectez-vous à Azure avec un compte disposant des autorisations Propriétaire ou Collaborateur sur l’abonnement Azure que vous souhaitez utiliser pour l’outil de diagnostic :
   ```powershell
   Login-AzAccount
   ```
3. Connectez-vous à Azure AD avec le même compte :
   ```powershell
   Connect-AzureAD
   ```
4. Accédez au [dépôt GitHub RDS-Templates](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy/scripts) et exécutez le script **CreateADAppRegistrationforDiagnostics.ps1** dans PowerShell.
5.  Lorsque le script vous demande de nommer votre application, entrez un nom d’application unique.


Une fois que le script s’exécute correctement, il doit afficher les éléments suivants dans sa sortie :

-  Un message confirmant votre application a maintenant une attribution de rôle de principal du service.
-  Votre ID client et votre clé secrète client dont vous aurez besoin quand vous déploierez l’outil de diagnostic.

Maintenant que vous avez inscrit votre application, il est temps de configurer votre espace de travail Log Analytics.

## <a name="configure-your-log-analytics-workspace"></a>Configurer votre espace de travail Log Analytics

Pour une expérience optimale, nous vous recommandons de configurer votre espace de travail Log Analytics avec les compteurs de performances suivants qui vous permettent de dériver des instructions de l’expérience utilisateur dans une session à distance. Pour obtenir une liste des compteurs recommandés avec des seuils suggérés, consultez [seuils des compteurs de performances Windows](deploy-diagnostics.md#windows-performance-counter-thresholds).

### <a name="create-an-azure-log-analytics-workspace-using-powershell"></a>Créer un espace de travail Log Analytics dans Azure à l’aide de PowerShell

Vous pouvez exécuter un script PowerShell pour créer un espace de travail Log Analytics et configurer les compteurs de performances Windows recommandés pour surveiller l’expérience utilisateur et les performances de l’application.

>[!NOTE]
>Si vous disposez déjà d’un espace de travail Log Analytics que vous avez créé sans le script PowerShell que vous souhaitez utiliser, passez directement à [Valider les résultats du script dans le Portail Azure](#validate-the-script-results-in-the-azure-portal).

Pour exécuter le script PowerShell :

1.  Ouvrez PowerShell en tant qu’administrateur.
2.  Accédez au [dépôt GitHub RDS-Templates](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy/scripts) et exécutez le script **CreateLogAnalyticsWorkspaceforDiagnostics.ps1** dans PowerShell.
3. Entrez les valeurs suivantes pour les paramètres :

    - Pour **ResourceGroupName**, entrez le nom du groupe de ressources.
    - Pour **LogAnalyticsWorkspaceName**, entrez un nom unique pour votre espace de travail Log Analytics.
    - Pour **Emplacement**, saisissez la région Azure que vous utilisez.
    - Entrez l'**ID d’abonnement Azure**, que vous trouverez dans le Portail Azure sous **Abonnements**.

4. Entrez les informations d’identification d’un utilisateur disposant d’un accès administrateur délégué.
5. Connectez-vous au Portail Azure avec les mêmes informations d’identification d’utilisateur.
6. Notez l’ID LogAnalyticsWorkspace ou mémorisez-le pour une utilisation ultérieure.
7. Si vous configurez l’espace de travail Log Analytics avec le script PowerShell, vos compteurs de performances doivent déjà être configurés et vous pouvez passer directement à [Valider des résultats du script dans le Portail Azure](#validate-the-script-results-in-the-azure-portal). Sinon, passez à la section suivante.

### <a name="configure-windows-performance-counters-in-your-existing-log-analytics-workspace"></a>Configurer les compteurs de performances Windows dans votre espace de travail Log Analytics existant

Cette section est destinée aux utilisateurs qui souhaitent utiliser un espace de travail Log Analytics existant créé sans le script PowerShell dans la section précédente. Si vous n’avez pas utilisé le script, vous devez alors configurer manuellement les compteurs de performances Windows recommandés.

Voici comment configurer manuellement les compteurs de performances recommandés :

1. Ouvrez votre navigateur Internet et connectez-vous au [Portail Azure](https://portal.azure.com/) avec votre compte d’administrateur.
2. Accédez ensuite aux **espaces de travail Log Analytics** pour examiner les compteurs de performances Windows configurés.
3. Dans la section **Paramètres**, sélectionnez **Paramètres avancés**.
4. Ensuite, accédez à **Données** >  **Compteurs de performances Windows** et ajoutez les compteurs suivants :

    -   LogicalDisk(\*)\\%Espace libre
    -   LogicalDisk(C:)\\Moy. Longueur de file d'attente de disque
    -   Mémoire(\*)\\Mo disponibles
    -   Informations processeur (\*)\\Temps processeur
    -   Retard d’entrée utilisateur par session(\*)\\Délai d’entrée maximal

Découvrez-en plus sur les compteurs de performances sur [Source de données des performances Windows et Linux dans Azure Monitor](../../azure-monitor/platform/data-sources-performance-counters.md).

>[!NOTE]
>Les compteurs supplémentaires que vous configurez n'apparaîtront pas dans l'outil de diagnostic lui-même. Pour qu’il apparaisse dans l’outil de diagnostic, vous devez configurer le fichier config de l’outil. Les instructions pour effectuer cette opération avec l’administration avancé seront disponibles dans GitHub à une date ultérieure.

## <a name="validate-the-script-results-in-the-azure-portal"></a>Valider les résultats du script dans le Portail Azure

Avant de poursuivre le déploiement de l’outil de diagnostic, nous vous recommandons de vérifier que votre application Azure Active Directory possède des autorisations d’API et que votre espace de travail Log Analytics dispose des compteurs de performances Windows préconfigurés.

### <a name="review-your-app-registration"></a>Consulter votre inscription d’application

Pour vous assurer que votre inscription d’application possède des autorisations d’API :

1. Ouvrez un navigateur et connectez-vous au [Portail Azure](https://portal.azure.com/) avec votre compte d’administrateur.
2. Accédez à **Azure Active Directory**.
3. Accédez à **Inscriptions d'applications** et sélectionnez **Toutes les applications**.
4. Recherchez votre inscription d'application Azure AD avec le nom d'application que vous avez entré à l'étape 5 de [Créer une inscription d'application Azure Active Directory](deploy-diagnostics.md#create-an-azure-active-directory-app-registration).

### <a name="review-your-log-analytics-workspace"></a>Consulter votre espace de travail Log Analytics

Pour vous assurer que votre espace de travail Log Analytics dispose des compteurs de performances Windows préconfigurés :

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à **espace de travail Log Analytics** pour consulter les compteurs de performances Windows configurés.
2. Sous **Paramètres**, sélectionnez **Paramètres avancés**.
3. Ensuite, accédez à **Données** > **Compteur de performances Windows**.
4. Assurez-vous que les compteurs suivants sont préconfigurés :

   - LogicalDisk(\*)\\%Espace libre : Affiche la quantité d’espace libre de l’espace total utilisable sur le disque sous forme de pourcentage.
   - LogicalDisk(C:)\\Moy. Longueur de file d'attente du disque : La longueur de la requête de transfert sur le disque pour votre lecteur C. La valeur ne doit pas dépasser 2 très longtemps.
   - Mémoire(\*)\\Mo disponibles : Mémoire disponible pour le système en mégaoctets.
   - Informations processeur(\*)\\Temps processeur : pourcentage de durée calendaire passée par le processeur pour exécuter des threads actives.
   - Retard d’entrée utilisateur par session(\*)\\Délai d’entrée maximal

### <a name="connect-to-vms-in-your-log-analytics-workspace"></a>Se connecter aux machines virtuelles dans votre espace de travail Log Analytics

Pour pouvoir afficher l’intégrité des machines virtuelles, vous devez activer la connexion Log Analytics. Suivez ces étapes pour connecter vos machines virtuelles :

1. Ouvrez un navigateur et connectez-vous au [Portail Azure](https://portal.azure.com/) avec votre compte d’administrateur.
2. Accédez à votre espace de travail Log Analytics.
3. Dans le menu de gauche, sous Sources de données de l’espace de travail, sélectionnez **Machines virtuelles**.
4. Sélectionnez le nom de la machine virtuelle à laquelle vous souhaitez vous connecter.
5. Sélectionnez **Connecter**.

## <a name="deploy-the-diagnostics-tool"></a>Déployer l’outil de diagnostic

Pour déployer le modèle Azure Resource Management pour l’outil de diagnostic :

1.  Accédez à la [page Azure RDS-Templates de GitHub ](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy).
2.  Déployez le modèle sur Azure et suivez les instructions dans le modèle. Assurez-vous de disposer des informations suivantes :

    -   ID-Client
    -   Secret-Client
    -   ID d’espace de travail Log Analytics

3.  Une fois les paramètres d’entrée fournis, acceptez les conditions générales, puis sélectionnez **Acheter**.

Le déploiement prendra 2 à 3 minutes. Une fois le déploiement réussi, accédez au groupe de ressources et assurez-vous que les ressources de l’application web et du plan de service des applications sont disponibles.

Après cela, vous devez définir l’URI de redirection.

### <a name="set-the-redirect-uri"></a>Définie l’URI de redirection

Pour définir l’URI de redirection :

1.  Dans le [Portail Azure](https://portal.azure.com/), accédez à **App services** et recherchez l’application que vous avez créée.
2.  Accédez à la page de présentation et copiez l’URL que vous y trouverez.
3.  Accédez à **inscriptions des applications**, puis sélectionnez l’application que vous souhaitez déployer.
4.  Dans le volet gauche, sous la section Gérer, sélectionnez **Authentification**.
5.  Entrez l’URI de redirection souhaité dans la zone de texte **URI de redirection**, puis sélectionnez **Enregistrer** dans le coin supérieur gauche du menu.
6. Sélectionnez **Web** dans le menu déroulant sous Type.
7. Entrez l’URL à partir de la page de présentation de l’application et ajoutez **/security/signIn-callback** à la fin. Par exemple : `https://<yourappname>.azurewebsites.net/security/signin-callback`.

   > [!div class="mx-imgBorder"]
   > ![La page URI de redirection](../media/redirect-uri-page.png)

8. Accédez à présent à vos ressources Azure, sélectionnez la ressource Azure App Services avec le nom fourni dans le modèle, puis accédez à l’URL y associée. (Par exemple, si le nom de l’application que vous avez utilisé dans le modèle était `contosoapp45`, votre URL associée est <http://contoso.azurewebsites.net>).
9. Connectez-vous en utilisant le compte d’utilisateur Azure Active Directory approprié.
10.   Sélectionnez **Accepter**.

## <a name="distribute-the-diagnostics-tool"></a>Distribuer l’outil de diagnostic

Avant de rendre l’outil de diagnostic disponible pour vos utilisateurs, assurez-vous qu’ils disposent des autorisations suivantes :

- Les utilisateurs doivent avoir un accès en lecture pour l’analytique des journaux d'activité. Pour plus d’informations, consultez [Prise en main des rôles, des autorisations et de la sécurité dans Azure Monitor](../../azure-monitor/platform/roles-permissions-security.md).
-  Les utilisateurs ont également besoin d’un accès en lecture pour l’abonné Windows Virtual Desktop (rôle de lecteur des Services Bureau à distance). Pour plus d’informations, consultez [Accès délégué dans Windows Virtual Desktop](delegated-access-virtual-desktop-2019.md).

Vous devez également fournir aux utilisateurs les informations suivantes :

- L’URL de l’application
- Les noms de l’abonné individuel du groupe d’abonnés auquel ils peuvent accéder.

## <a name="use-the-diagnostics-tool"></a>Utiliser l’outil de diagnostic

Une fois que vous êtes connecté à votre compte à l’aide des informations que vous avez reçues de votre organisation, préparez le nom d'utilisateur principal pour l’utilisateur pour lequel vous souhaitez demander des activités. Une recherche vous donnera toutes les activités sous le type d’activité spécifié qui s’est produit au cours de la dernière semaine.

### <a name="how-to-read-activity-search-results"></a>Comment lire les résultats de la recherche d’une activité

Les activités sont triées par timestamp, avec l’activité la plus récente en premier. Si les résultats retournent une erreur, commencez par vérifier s’il s’agit d’une erreur de service. Pour les erreurs de service, créez un ticket de support avec les informations d’activité pour nous aider à déboguer le problème. Tous les autres types d’erreurs peuvent généralement être résolus par l’utilisateur ou l’administrateur. Pour obtenir la liste des scénarios d’erreur les plus courants et savoir comment les résoudre, consultez [Identifier et diagnostiquer les problèmes](diagnostics-role-service-2019.md#common-error-scenarios).

>[!NOTE]
>Les erreurs de service sont appelées « erreurs externes » dans la documentation liée. Cela sera modifié lors de la mise à jour de la référence PowerShell.

Les activités de connexion peuvent avoir plus d’une erreur. Vous pouvez développer le type d’activité pour consulter toutes les autres erreurs rencontrées par l’utilisateur. Sélectionnez le nom du code d’erreur pour ouvrir une boîte de dialogue qui vous permet d’obtenir plus d’informations.

### <a name="investigate-the-session-host"></a>Examiner l’hôte de la session

Dans les résultats de la recherche, recherchez et sélectionnez l’hôte de la session sur lequel vous souhaitez obtenir des informations.

Vous pouvez analyser l’intégrité de l’hôte de la session :

- En fonction d’un seuil prédéfini, vous pouvez récupérer les informations sur l’intégrité de l’hôte de la session que Log Analytics demande.
- Lorsqu’il n’y a pas d’activité ou que l’hôte de la session n’est pas connecté à Log Analytics, les informations ne sont pas disponibles.

Vous pouvez également interagir avec les utilisateurs sur l’hôte de la session :

- Vous pouvez vous déconnecter ou envoyer un message aux utilisateurs connectés.
- L’utilisateur que vous avez recherché à l’origine est sélectionné par défaut, mais vous pouvez également sélectionner des utilisateurs supplémentaires pour envoyer des messages ou déconnecter plusieurs utilisateurs à la fois.

### <a name="windows-performance-counter-thresholds"></a>Seuils du compteur de performances Windows

- LogicalDisk(\*)\\%Espace libre :

    - Affiche le pourcentage de l’espace total utilisable sur le disque logique qui est libre.
    - Seuil : Moins de 20 % sont marqués comme non sains.

- LogicalDisk(C:)\\Moy. Longueur de file d'attente du disque :

    - Représente les conditions du système de stockage.
    - Seuil : Plus de 5 sont marqués comme non sains.

- Mémoire(\*)\\Mo disponibles :

    - Mémoire disponible pour le système.
    - Seuil : Moins de 500 mégaoctets sont marqués comme non sains.

- Informations processeur(\*)\\Temps processeur :

    - Seuil : Plus de 80 % sont marqués comme non sains.

- [Retard d’entrée utilisateur par session(\*)\\Délai d’entrée maximal](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/) :

    - Seuil : Plus de 2 000 ms sont marqués comme non sains.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment surveiller les journaux d’activité dans [Utiliser les diagnostics avec Log Analytics](diagnostics-log-analytics-2019.md).
- Consultez des scénarios d’erreur courants et découvrez comment y remédier dans [Identifier et diagnostiquer les problèmes](diagnostics-role-service-2019.md).