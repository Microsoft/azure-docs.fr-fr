---
title: Configurer un abonnement de la Place de marché Azure pour les versions d'évaluation hébergées
description: Explique comment configurer un abonnement de la Place de marché Azure pour les versions d'évaluation de Dynamics 365 for Customer Engagement et Dynamics 365 for Operations
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: trkeya
ms.author: trkeya
ms.date: 11/09/2020
ms.openlocfilehash: 6f74f3b1f5aad153903ba5d290a290973203a875
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489384"
---
# <a name="set-up-an-azure-marketplace-subscription-for-hosted-test-drives"></a>Configurer un abonnement de la Place de marché Azure pour les versions d'évaluation hébergées

Cet article explique comment configurer un abonnement de la Place de marché Azure et un environnement **Dynamics 365 for Customer Engagement** ou **Dynamics 365 for Operations** environnement pour les versions d'évaluation.

## <a name="set-up-for-dynamics-365-for-customer-engagement"></a>Configuration de Dynamics 365 for Customer Engagement

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) avec un compte d’administrateur.
2. Vérifiez que vous êtes dans le client associé à votre instance de version d'évaluation de Dynamics 365 en pointant sur l’icône de votre compte dans le coin supérieur droit. Si vous n’êtes pas dans le locataire approprié, sélectionnez l’icône de compte pour basculer vers le locataire approprié.

    :::image type="content" source="./media/test-drive/select-correct-tenant.png" alt-text="Sélection du locataire approprié.":::

3. Vérifiez que la licence **Offre Dynamics 365 Customer Engagement** est disponible.

    [![Vérification de la licence de l’offre.](media/test-drive/check-plan-license.png)](media/test-drive/check-plan-license.png#lightbox)

4. Créez une application Azure Active Directory (AD) dans Azure. AppSource utilise cette application pour approvisionner et déprovisionner l’utilisateur de la version d'évaluation dans votre locataire.
    1. Dans le volet de filtre, sélectionnez **Azure Active Directory**.
    2. Sélectionnez **Inscriptions d’applications**.

        [![Sélection des inscriptions d’applications.](media/test-drive/app-registrations.png)](media/test-drive/app-registrations.png#lightbox)

    3. Sélectionnez **Nouvelle inscription**.
    4. Fournissez un nom d’application approprié.

        :::image type="content" source="./media/test-drive/register-an-application.png" alt-text="Inscription d’une application.":::

    5. Sous Types de comptes pris en charge, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
    6. Sélectionnez **Créer** et attendez que votre application soit créée.
    7. Une fois l’application créée, notez l’**ID d’application**  affiché sur l’écran Vue d’ensemble. Vous aurez besoin de cette valeur ultérieurement lors de la configuration de votre version d'évaluation.
    8. Pour ajouter un URI de redirection NativeClient, sélectionnez le panneau **Authentification**. Sous **Configuration de la plateforme**, sélectionnez la vignette d’application **Ajouter une plateforme** > **Mobile** > **Bureau**. Choisissez l’URI de redirection **NativeClient** et sélectionnez **Configurer**.

        :::image type="content" source="./media/test-drive/configure-desktop-devices.png" alt-text="Ajout d’un URI de redirection NativeClient.":::

    9. Sous **Gérer l’application**, sélectionnez **Autorisations de l’API**.
    10. Sélectionnez **Ajouter une autorisation**, puis **Microsoft Graph API**.
    11. Sélectionnez la catégorie d’autorisation **Application**, puis les autorisations **Directory.Read.All** et **Directory.ReadWrite.All**.

        :::image type="content" source="./media/test-drive/microsoft-graph.png" alt-text="Définition des autorisations de l’application.":::

    12. Pour ajouter l’accès **Dynamics CRM - Emprunt d’identité d’utilisateur** à la liste d'autorisations pour l’application Azure AD, sélectionnez à nouveau **Ajouter une autorisation** .

        :::image type="content" source="./media/test-drive/request-api-permissions.png" alt-text="Demande des autorisations de l’application.":::

    13. Une fois l’autorisation ajoutée, sélectionnez **Accorder le consentement de l’administrateur pour Microsoft**.
    14. Dans la fenêtre de message d’alerte, sélectionnez **Oui**.

        [![Indique que les autorisations de l’application ont été accordées.](media/test-drive/api-permissions-confirmation-customer.png)](media/test-drive/api-permissions-confirmation-customer.png#lightbox)

    15. Pour générer une clé secrète pour l’Azure AD App :
        1. Dans **Gérer l’application**, sélectionnez **Certificat et secrets**.
        2. Sous Secrets client, sélectionnez **Nouveau secret client**.
        3. Entrez une description, par exemple *Version d'évaluation*, puis sélectionnez une durée appropriée. La version d'évaluation s’interrompt une fois cette clé expirée. Vous devez alors générer et fournir à AppSource une nouvelle clé.
        4. Sélectionnez **Ajouter** pour générer le secret de l’application Azure. Copiez cette valeur, car elle sera masquée dès que vous quitterez ce panneau. Vous aurez besoin de cette valeur ultérieurement lors de la configuration de votre version d'évaluation.

            :::image type="content" source="./media/test-drive/add-client-secret.png" alt-text="Ajout d’un secret client.":::

5. Parfois, il faut plus de temps que prévu pour synchroniser un utilisateur d’Azure AD à une instance CRM. Pour faciliter cette opération, nous avons ajouté un processus pour forcer la synchronisation de l’utilisateur, mais cela nécessite que l’application Azure AD soit dans la liste autorisée de l’Espace partenaires. Pour ce faire, consultez [Synchronisation d’utilisateurs avec l’instance Customer Engagement](https://github.com/microsoft/AppSource/blob/master/Microsoft%20Hosted%20Test%20Drive/CDS_Utility_to_ForceUserSync_in_CRM_Instance.md).
6. Ajoutez le rôle de principal de service à l’application pour permettre à l’application Azure AD de supprimer des utilisateurs de votre locataire Azure.
    1. Ouvrez une invite de commandes PowerShell d’administration.
    2. Install-Module MSOnline (exécutez cette commande si MSOnline n’est pas installé).
    3. Connect-MsolService (cette opération permet d’afficher une fenêtre contextuelle ; connectez-vous avec le locataire d’organisation nouvellement créé).
    4. $applicationId = **<VOTRE_ID_APPLICATION>** .
    5. $sp = Get-MsolServicePrincipal -AppPrincipalId $applicationId.
    6. Add-MsolRoleMember -RoleObjectId fe930be7-5e62-47db-91af-98c3a49a38b1 -RoleMemberObjectId $sp.ObjectId -RoleMemberType servicePrincipal.

        :::image type="content" source="./media/test-drive/sign-in-to-account.png" alt-text="Connexion à votre compte.":::

7. Ajoutez l’application Azure créée ci-dessus en tant qu’utilisateur d’application à votre instance de CRM de version d'évaluation.
    1. Ajoutez un nouvel utilisateur dans **Azure Active Directory**. Seules les valeurs **Nom** et **Nom d’utilisateur** (appartenant au même locataire) sont nécessaires pour créer cet utilisateur. Laissez les autres champs à leurs valeurs par défaut. Copiez la valeur de nom d’utilisateur.
    2. Connectez-vous à **Instance CRM** et sélectionnez **Paramètre** > **Sécurité** > **Utilisateurs**.
    3. Modifiez la vue sur **Utilisateurs de l’application**.

        :::image type="content" source="./media/test-drive/application-users.png" alt-text="Définition des informations de compte pour un utilisateur.":::

    4. Ajoutez un nouvel utilisateur (Assurez-vous que le formulaire est destiné à l’UTILISATEUR DE L’APPLICATION).
    5. Entrez le même nom d’utilisateur dans les champs **E-mail principal** et **Nom d’utilisateur**. Ajoutez l’**Azure ApplicationId** dans **ID d’application**.
    6. Donnez le **Nom complet** de votre choix.
    7. Sélectionnez **Enregistrer**.
    8. Sélectionnez **Gérer les rôles**.
    9. Attribuez un rôle de sécurité personnalisé ou OOB qui contient des privilèges de lecture, d’écriture et d’attribution de rôles, par exemple *Administrateur système*.

        :::image type="content" source="./media/test-drive/security-roles-selection.png" alt-text="Sélection des privilèges de rôle.":::

    10. Affectez à l’utilisateur de l’application le rôle de sécurité personnalisé que vous avez créé pour votre version d'évaluation.

## <a name="set-up-for-dynamics-365-for-operations"></a>Configurer pour Dynamics 365 for Operations

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) avec un compte d’administrateur.
2. Vérifiez que vous êtes dans le client associé à votre instance de version d'évaluation de Dynamics 365 en pointant sur l’icône de votre compte dans le coin supérieur droit. Si vous n’êtes pas dans le locataire approprié, sélectionnez l’icône de compte pour basculer vers le locataire approprié.

    :::image type="content" source="./media/test-drive/select-correct-tenant.png" alt-text="Sélection du locataire approprié.":::

3. Créez une application Azure AD dans Azure. AppSource utilise cette application pour approvisionner et déprovisionner l’utilisateur de la version d'évaluation dans votre locataire.
    1. Dans le volet de filtre, sélectionnez **Azure Active Directory**.
    2. Sélectionnez **Inscriptions d’applications**.

        :::image type="content" source="./media/test-drive/app-registrations.png" alt-text="Sélection d’une inscription d’application.":::

    3. Sélectionnez **Nouvelle inscription**.
    4. Fournissez un nom d’application approprié.

        :::image type="content" source="./media/test-drive/register-an-application.png" alt-text="Inscription d’une application.":::

    5. Sous Types de comptes pris en charge, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
    6. Sélectionnez **Créer** et attendez que votre application soit créée.
    7. Une fois l’application créée, notez l’**ID d’application**  affiché sur l’écran Vue d’ensemble. Vous aurez besoin de cette valeur ultérieurement lors de la configuration de votre version d'évaluation.
    8. Sous **Gérer l’application**, sélectionnez **Autorisations de l’API**.
    9. Sélectionnez **Ajouter une autorisation**, puis **Microsoft Graph API**.
    10. Sélectionnez la catégorie d’autorisation **Application**, puis les autorisations **Directory.Read.All** et **Directory.ReadWrite.All**.

        :::image type="content" source="./media/test-drive/microsoft-graph.png" alt-text="Définition des autorisations de l’application.":::

    11. Sélectionnez **Ajouter une autorisation**.
    12. Une fois l’autorisation ajoutée, sélectionnez **Accorder le consentement de l’administrateur pour Microsoft**.
    13. Dans la fenêtre de message d’alerte, sélectionnez **Oui**.

        [![Indique les autorisations de l’application accordées.](media/test-drive/api-permissions-confirmation-operations.png)](media/test-drive/api-permissions-confirmation-operations.png#lightbox)

    14. Pour générer une clé secrète pour l’Azure AD App :
        1. Dans **Gérer l’application**, sélectionnez **Certificat et secrets**.
        2. Sous Secrets client, sélectionnez **Nouveau secret client**.
        3. Entrez une description, par exemple *Version d'évaluation*, puis sélectionnez une durée appropriée. La version d'évaluation s’interrompt une fois cette clé expirée. Vous devez alors générer et fournir à AppSource une nouvelle clé.
        4. Sélectionnez **Ajouter** pour générer le secret de l’application Azure. Copiez cette valeur, car elle sera masquée dès que vous quitterez ce panneau. Vous aurez besoin de cette valeur ultérieurement lors de la configuration de votre version d'évaluation.

            :::image type="content" source="./media/test-drive/add-client-secret.png" alt-text="Ajout d’un secret client.":::

4. Ajoutez le rôle de principal de service à l’application pour permettre à l’application Azure AD de supprimer des utilisateurs de votre locataire Azure.
    1. Ouvrez une invite de commandes PowerShell d’administration.
    2. Install-Module MSOnline (exécutez cette commande si MSOnline n’est pas installé).
    3. Connect-MsolService (cette opération permet d’afficher une fenêtre contextuelle ; connectez-vous avec le locataire d’organisation nouvellement créé).
    4. $applicationId = **<VOTRE_ID_APPLICATION>** .
    5. $sp = Get-MsolServicePrincipal -AppPrincipalId $applicationId.
    6. Add-MsolRoleMember -RoleObjectId fe930be7-5e62-47db-91af-98c3a49a38b1 -RoleMemberObjectId $sp.ObjectId -RoleMemberType servicePrincipal.

        :::image type="content" source="./media/test-drive/sign-in-to-account.png" alt-text="Connexion à votre compte.":::

5. Ajoutez maintenant l’application ci-dessus à **Dynamics 365 for Operations** pour permettre à l’application de gérer les utilisateurs.
    1. Recherchez votre instance **Dynamics 365 for Operations**.
    2. Dans le coin supérieur gauche, cliquez sur le menu de trois lignes (hamburger).
    3. Sélectionnez **Administration système**.
    4. Sélectionnez **Applications Azure Active Directory**.
    5. Sélectionnez **+Nouveau**.
    6. Entrez l’**ID client de l’application Azure AD** qui va effectuer les actions au nom de l’utilisateur.

    > [!NOTE]
    > L’ID d’utilisateur au nom duquel les actions seront exécutées (en général, l’administrateur système de l’instance ou un utilisateur disposant de privilèges pour ajouter d’autres utilisateurs).

    [![L’ID d’utilisateur au nom duquel les actions seront exécutées (en général, l’administrateur système de l’instance ou un utilisateur disposant de privilèges pour ajouter d’autres utilisateurs).](media/test-drive/system-admin-user-id.png)](media/test-drive/system-admin-user-id.png#lightbox)
<!--
## Next steps

- [Commercial marketplace partner and customer usage attribution](azure-partner-customer-usage-attribution.md) -->
