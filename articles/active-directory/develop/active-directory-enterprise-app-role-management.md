---
title: Configurer la revendication de rôle pour les applications Azure AD d’entreprise | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer les revendications de rôle émises dans le jeton SAML pour les applications d’entreprise dans Azure Active Directory
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: how-to
ms.date: 12/07/2020
ms.author: jeedes
ms.openlocfilehash: 46a676f90eb9b628551c50f9c3bf664056dd6ca7
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97355942"
---
# <a name="how-to-configure-the-role-claim-issued-in-the-saml-token-for-enterprise-applications"></a>Procédure : Configurer les revendications de rôle émises dans le jeton SAML pour les applications d'entreprise

Grâce à Azure Active Directory (Azure AD), vous pouvez personnaliser le type de revendications de rôle que vous recevez après avoir autorisé une application.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure AD avec l’installation des répertoires.
- Un abonnement pour lequel l’authentification unique (SSO) est activée. Vous devez configurer l’authentification unique avec votre application.

## <a name="when-to-use-this-feature"></a>Quand utiliser cette fonctionnalité

Si votre application s’attend à voir passer dans une réponse SAML des rôles personnalisés, vous devez utiliser cette fonctionnalité. Vous pouvez créer autant de rôles que nécessaire à passer à nouveau d’Azure AD vers votre application.

## <a name="create-roles-for-an-application"></a>Créer des rôles pour une application

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez l’icône **Azure Active Directory**.

    ![Icône Azure Active Directory][1]

2. Sélectionnez **Applications d’entreprise**. Sélectionnez ensuite **Toutes les applications**.

    ![Volet Applications d’entreprise][2]

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton « Nouvelle application »][3]

4. Dans la zone de recherche, saisissez le nom de votre application, puis sélectionnez votre application dans le panneau de résultats. Sélectionnez le bouton **Ajouter** pour ajouter l’application.

    ![Application dans la liste des résultats](./media/active-directory-enterprise-app-role-management/tutorial_app_addfromgallery.png)

5. Lorsque l’application est ajoutée, allez à la page **Propriétés** et copiez l’ID de l’objet.

    ![Page Propriétés](./media/active-directory-enterprise-app-role-management/tutorial_app_properties.png)

6. Ouvrez l’[afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) dans une autre fenêtre et procédez comme suit :

    a. Connectez-vous au site de l’Explorateur graphique en utilisant les informations d’identification d’administrateur global ou de coadministrateur de votre locataire.

    b. Vous devez disposer des autorisations suffisantes pour créer les rôles. Sélectionnez **Modifier les autorisations** pour obtenir les autorisations.

      ![Bouton « Modifier les autorisations »](./media/active-directory-enterprise-app-role-management/graph-explorer-new9.png)

    >[!NOTE]
    >Les rôles Administrateur d’application et Administrateur Cloud App ne sont pas appropriés pour ce scénario, puisque nous avons besoin d’autorisations d’administrateur général pour effectuer des opérations d’écriture et de lecture dans le répertoire.

    c. Sélectionnez les autorisations suivantes dans la liste (si vous ne l’avez pas déjà fait) et sélectionnez **Modifier les autorisations**.

      ![Liste des autorisations et bouton « Modifier les autorisations »](./media/active-directory-enterprise-app-role-management/graph-explorer-new10.png)

    d. Acceptez le consentement. Vous êtes à nouveau connecté au système.

    e. Modifiez la version sur **bêta** et extrayez la liste des principaux du service à partir de votre locataire à l’aide de la requête suivante :

     `https://graph.microsoft.com/beta/servicePrincipals`

      Si vous utilisez plusieurs répertoires, suivez ce modèle : `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

      ![Boîte de dialogue de l’Explorateur graphique, avec la requête d’extraction des principaux du service](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

      
    f. Dans la liste des principaux du service extraits, obtenez celui que vous devez modifier. Vous pouvez également utiliser les touches Ctrl + F pour rechercher l’application dans la liste des principaux du service. Recherchez l’ID de l’objet que vous avez copié à partir de la page **Propriétés** et utilisez la requête suivante pour accéder au principal du service :

      `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

      ![Requête d’obtention du principal du service que vous devez modifier](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

    g. Extrayez la propriété **appRoles** à partir de l’objet du principal du service.

      ![Détails de la propriété appRoles](./media/active-directory-enterprise-app-role-management/graph-explorer-new3.png)

      
      Si vous utilisez l’application personnalisée (non l’application de la Place de marché Azure), les deux rôles par défaut s’affichent : utilisateur et msiam_access. Pour l’application de la Place de marché, msiam_access est le seul rôle par défaut. Vous n’avez pas besoin d’apporter des modifications aux rôles par défaut.

    h. Générez de nouveaux rôles pour votre application.

      Le JSON suivant est un exemple d’objet **appRoles**. Créez un objet similaire pour ajouter les rôles que vous voulez pour votre application.

      ```
      {
         "appRoles": [
          {
              "allowedMemberTypes": [
                  "User"
              ],
              "description": "msiam_access",
              "displayName": "msiam_access",
              "id": "b9632174-c057-4f7e-951b-be3adc52bfe6",
              "isEnabled": true,
              "origin": "Application",
              "value": null
          },
          {
              "allowedMemberTypes": [
                  "User"
              ],
              "description": "Administrators Only",
              "displayName": "Admin",
              "id": "4f8f8640-f081-492d-97a0-caf24e9bc134",
              "isEnabled": true,
              "origin": "ServicePrincipal",
              "value": "Administrator"
          }
      ]
      }
      ```

      
      Vous ne pouvez ajouter de nouveaux rôles qu’après msiam_access de l’opération de correction. Vous pouvez également ajouter autant de rôles selon les besoins de votre organisation. Azure AD envoie la valeur de ces rôles conformément à la valeur de revendication dans la réponse SAML. Pour générer les valeurs de GUID pour l’ID de nouveaux rôles, utilisez les outils web tels que [this](https://www.guidgenerator.com/)

    i. Revenez à l’Explorateur graphique et modifiez la méthode de **GET** à **PATCH**. Corrigez l’objet du principal du service pour obtenir les rôles souhaités en mettant à jour la propriété **appRoles** comme celle affichée dans l’exemple précédent. Sélectionnez **Exécuter la requête** pour exécuter l’opération de correction. Un message de réussite confirme la création du rôle.

      ![Opération de correction avec le message de réussite](./media/active-directory-enterprise-app-role-management/graph-explorer-new11.png)

7. Une fois le principal du service corrigé avec d’autres rôles, vous pouvez assigner des utilisateurs aux rôles respectifs. Vous pouvez assigner les utilisateurs en accédant au portail et à l’application. Sélectionnez l’onglet **Utilisateurs et groupes**. Cet onglet répertorie tous les utilisateurs et groupes déjà assignés à l’application. Vous pouvez ajouter de nouveaux utilisateurs sur les nouveaux rôles. Vous pouvez également sélectionner un utilisateur existant et sélectionnez **Modifier** pour modifier le rôle.

    ![Onglet « Utilisateurs et groupes »](./media/active-directory-enterprise-app-role-management/graph-explorer-new5.png)

    Pour assigner le rôle à n’importe quel utilisateur, sélectionnez le nouveau rôle puis le bouton **Assigner** dans la partie inférieure de la page.

    ![Volets « Modifier l’affectation » et « Sélectionner un rôle »](./media/active-directory-enterprise-app-role-management/graph-explorer-new6.png)

    
    Vous devez actualiser votre session dans le portail Azure pour afficher les nouveaux rôles.

8. Mettez à jour la table **Attributs** table pour définir un mappage personnalisé de la revendication de rôle.

9. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur**, effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous :

    | Nom de l’attribut | Valeur d'attribut |
    | -------------- | ----------------|
    | Nom de rôle  | user.assignedroles |

    
    Si la valeur de revendication de rôle est null, Azure AD n’enverra pas cette valeur dans le jeton ; il s’agit du comportement par défaut normal.

    a. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Attributs et revendications de l’utilisateur**.

      ![Capture d’écran mettant en évidence l’icône Modifier qui sert à ouvrir la boîte de dialogue Attributs et revendications de l’utilisateur.](./media/active-directory-enterprise-app-role-management/editattribute.png)

    b. Dans la boîte de dialogue **Gérer les revendications des utilisateurs**, ajoutez l’attribut de jeton SAML en cliquant sur **Ajouter une nouvelle revendication**.

      ![Bouton « Ajouter un attribut »](./media/active-directory-enterprise-app-role-management/tutorial_attribute_04.png)

      ![Volet « Ajouter un attribut »](./media/active-directory-enterprise-app-role-management/tutorial_attribute_05.png)

    c. Dans la zone **Nom**, saisissez le nom de l’attribut, si nécessaire. Cet exemple utilise **Nom de rôle** comme nom de revendication.

    d. Laissez la zone **Espace de noms** vide.

    e. Dans la liste **Attribut de la source**, tapez la valeur d’attribut indiquée pour cette ligne.

    f. Sélectionnez **Enregistrer**.

10. Pour tester votre application dans une authentification unique initiée par un fournisseur d’identité, connectez-vous au [volet d’accès](https://myapps.microsoft.com) et sélectionnez la vignette de votre application. Dans le jeton SAML, vous devez voir tous les rôles assignés à l’utilisateur avec le nom de la revendication que vous avez attribué.

## <a name="update-an-existing-role"></a>Mettre à jour un rôle existant

Pour mettre à jour un rôle existant, procédez comme suit :

1. Ouvrez l’[afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer).

2. Connectez-vous au site de l’Explorateur graphique en utilisant les informations d’identification d’administrateur global ou de coadministrateur de votre locataire.

3. Modifiez la version sur **bêta** et extrayez la liste des principaux du service à partir de votre locataire à l’aide de la requête suivante :

    `https://graph.microsoft.com/beta/servicePrincipals`

    Si vous utilisez plusieurs répertoires, suivez ce modèle : `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

    ![Boîte de dialogue de l’Explorateur graphique, avec la requête d’extraction des principaux du service](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

4. Dans la liste des principaux du service extraits, obtenez celui que vous devez modifier. Vous pouvez également utiliser les touches Ctrl + F pour rechercher l’application dans la liste des principaux du service. Recherchez l’ID de l’objet que vous avez copié à partir de la page **Propriétés** et utilisez la requête suivante pour accéder au principal du service :

    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

    ![Requête d’obtention du principal du service que vous devez modifier](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

5. Extrayez la propriété **appRoles** à partir de l’objet du principal du service.

    ![Détails de la propriété appRoles](./media/active-directory-enterprise-app-role-management/graph-explorer-new3.png)

6. Pour mettre à jour le rôle existant, procédez comme suit.

    ![Corps de la demande pour « PATCH », avec « description » et « displayname » mis en surbrillance](./media/active-directory-enterprise-app-role-management/graph-explorer-patchupdate.png)

    a. Modifiez la méthode de **GET** à **PATCH**.

    b. Copiez les rôles existants et collez-les sous **Corps de la demande**.

    c. Mettez à jour la valeur d’un rôle en mettant à jour la description du rôle, la valeur du rôle ou le nom d’affichage du rôle en fonction des besoins.

    d. Lorsque vous avez mis à jour tous les rôles requis, sélectionnez **Exécuter la requête**.

## <a name="delete-an-existing-role"></a>Supprimer un rôle existant

Pour supprimer un rôle existant, procédez comme suit :

1. Ouvrez l’[afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) dans une autre fenêtre.

2. Connectez-vous au site de l’Explorateur graphique en utilisant les informations d’identification d’administrateur global ou de coadministrateur de votre locataire.

3. Modifiez la version sur **bêta** et extrayez la liste des principaux du service à partir de votre locataire à l’aide de la requête suivante :

    `https://graph.microsoft.com/beta/servicePrincipals`

    Si vous utilisez plusieurs répertoires, suivez ce modèle : `https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

    ![Boîte de dialogue de l’Explorateur graphique, avec la requête d’extraction de la liste des principaux du service](./media/active-directory-enterprise-app-role-management/graph-explorer-new1.png)

4. Dans la liste des principaux du service extraits, obtenez celui que vous devez modifier. Vous pouvez également utiliser les touches Ctrl + F pour rechercher l’application dans la liste des principaux du service. Recherchez l’ID de l’objet que vous avez copié à partir de la page **Propriétés** et utilisez la requête suivante pour accéder au principal du service :

    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

    ![Requête d’obtention du principal du service que vous devez modifier](./media/active-directory-enterprise-app-role-management/graph-explorer-new2.png)

5. Extrayez la propriété **appRoles** à partir de l’objet du principal du service.

    ![Détails de la propriété appRoles à partir de l’objet du principal du service](./media/active-directory-enterprise-app-role-management/graph-explorer-new7.png)

6. Pour supprimer le rôle existant, procédez comme suit.

    ![Corps de la demande pour « PATCH », avec IsEnabled défini sur false](./media/active-directory-enterprise-app-role-management/graph-explorer-new8.png)

    a. Modifiez la méthode de **GET** à **PATCH**.

    b. Copiez les rôles existants à partir de l’application et collez-les sous **Corps de la demande**.

    c. Définissez la valeur **IsEnabled** sur **false** pour le rôle que vous souhaitez supprimer.

    d. Sélectionnez **Run Query** (Exécuter la requête).

    
    Assurez-vous que vous disposez du rôle d’utilisateur msiam_access et que l’ID correspond au rôle généré.

7. Lorsque le rôle est désactivé, supprimez ce bloc de rôle de la section **appRoles**. Conservez la méthode **PATCH**, puis sélectionnez **Exécuter la requête**.

8. Lorsque vous avez exécuté la requête, le rôle est supprimé.

    
    Le rôle doit être désactivé pour pouvoir être supprimé.

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les étapes supplémentaires, consultez la [documentation de l’application](../saas-apps/tutorial-list.md).

<!--Image references-->
<!--Image references-->

[1]: ./media/active-directory-enterprise-app-role-management/tutorial_general_01.png
[2]: ./media/active-directory-enterprise-app-role-management/tutorial_general_02.png
[3]: ./media/active-directory-enterprise-app-role-management/tutorial_general_03.png
[4]: ./media/active-directory-enterprise-app-role-management/tutorial_general_04.png
