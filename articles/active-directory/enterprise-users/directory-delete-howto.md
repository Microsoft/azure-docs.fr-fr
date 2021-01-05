---
title: Supprimer une organisation Azure AD (locataire) – Azure Active Directory | Microsoft Docs
description: Explique comment préparer la suppression d’une organisation Azure AD (locataire), y compris une organisation en libre-service
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: addimitu
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2edc6fb98359c5360836bc369e5ae1928464df92
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861028"
---
# <a name="delete-a-tenant-in-azure-active-directory"></a>Supprimer un locataire dans Azure Active Directory

La suppression d’une organisation Azure AD (locataire) a également pour effet de supprimer toutes les ressources qu’elle contient. Vous devez préparer votre organisation en réduisant les ressources associées à un annuaire avant de le supprimer. Seul un administrateur général Azure Active Directory (Azure AD) peut supprimer un organisation Azure AD du portail.

## <a name="prepare-the-organization"></a>Préparer l’organisation

Avant de supprimer une organisation d’Azure AD, vous devez faire quelques vérifications. Celles-ci limitent le risque d’un impact négatif de la suppression de l’organisation Azure AD sur l’accès des utilisateurs, par exemple sur leur capacité à se connecter à Microsoft 365 ou à accéder à des ressources dans Azure. En effet, en cas de suppression d’une organisation associée à un abonnement, les utilisateurs ne peuvent plus accéder aux ressources Azure de cet abonnement. Les conditions à remplir sont les suivantes :

* L’organisation Azure AD (locataire) ne peut contenir aucun utilisateur, à l’exception de l’administrateur général qui doit la supprimer. Tous les autres utilisateurs doivent être supprimés au préalable. Si des utilisateurs se synchronisent à partir d’un emplacement local, la synchronisation doit être désactivée, et les utilisateurs supprimés de l’organisation cloud via le portail Azure ou à l’aide de cmdlets Azure PowerShell.
* L’organisation ne doit contenir aucune application. Toute application doit être supprimée de l’organisation avant la suppression de celle-ci.
* Aucun fournisseur d’authentification multifacteur ne peut être lié à l’organisation.
* Aucun abonnement aux services en ligne Microsoft tels que Microsoft Azure, Microsoft 365 ou Azure AD Premium, ne peut être associé à l’organisation. Par exemple, si une organisation Azure AD par défaut a été créée pour vous dans Azure, vous ne pouvez pas supprimer celle-ci si votre abonnement Azure en dépend toujours pour l’authentification. De même, vous ne pouvez pas supprimer une organisation à laquelle un autre utilisateur a associé un abonnement.

## <a name="delete-the-organization"></a>Supprimer l’organisation

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte d’administrateur général pour votre organisation.

2. Sélectionnez **Azure Active Directory**.

3. Basculez vers l’organisation à supprimer.
  
   ![Confirmez l’organisation avant de lancer la suppression](./media/directory-delete-howto/delete-directory-command.png)

4. Sélectionnez **Supprimer le locataire**.
  
   ![Sélectionnez la commande pour supprimer l’organisation](./media/directory-delete-howto/delete-directory-list.png)

5. Si votre organisation échoue à une ou plusieurs vérifications, vous recevez un lien vers des informations supplémentaires sur la façon d’y remédier. Une fois toutes les vérifications effectuées, sélectionnez **Supprimer** pour terminer le processus.

## <a name="if-you-cant-delete-the-organization"></a>Si vous ne pouvez pas supprimer l’organisation

Quand vous avez configuré votre organisation Azure AD, vous avez peut-être aussi activé pour celle-ci des abonnements assortis d’une licence, comme Azure AD Premium P2, Microsoft 365 Business Standard ou Enterprise Mobility + Security E5. Pour éviter toute perte accidentelle de données, vous ne pouvez pas supprimer une organisation tant que tous les abonnements associés à celle-ci n’ont pas été complètement supprimés. Pour que la suppression de l’organisation soit autorisée, les abonnements doivent être dans un état **Annulé**. Un abonnement **Expiré** ou **Annulé** passe à l’état **Désactivé**, et l’étape finale est l’état **Approvisionnement annulé**.

S’il s’agit de l’expiration d’un abonnement d’essai à Microsoft 365 (à l’exception des programmes payants Partenaire/CSP, Contrat Entreprise ou Licence en volume), consultez le tableau suivant. Pour plus d’informations sur la conservation des données et le cycle de vie des abonnements Microsoft 365, consultez [Qu’arrive-t-il à mes données et à mon accès à la fin de mon abonnement Microsoft 365 pour les entreprises ?](https://support.office.com/article/what-happens-to-my-data-and-access-when-my-office-365-for-business-subscription-ends-4436582f-211a-45ec-b72e-33647f97d8a3). 

État de l’abonnement | Données | Accès aux données
----- | ----- | -----
Actif (30 jours pour l’essai gratuit) | Données accessibles à tous | Les utilisateurs ont un accès normal aux fichiers ou applications Microsoft 365<br>Les administrateurs ont un accès normal au centre d’administration et aux ressources Microsoft 365 
Expiré (30 jours) | Données accessibles à tous| Les utilisateurs ont un accès normal aux fichiers ou applications Microsoft 365<br>Les administrateurs ont un accès normal au centre d’administration et aux ressources Microsoft 365
Désactivé (30 jours) | Données accessibles à l’administrateur uniquement | Les utilisateurs ne peuvent pas accéder aux fichiers ou aux applications Microsoft 365<br>Les administrateurs peuvent accéder au centre d’administration Microsoft 365, mais ne peuvent pas attribuer de licences ou mettre à jour des utilisateurs
Déprovisionné (30 jours après l’état Désactivé) | Données supprimées (automatiquement supprimées si aucun autre service n’est en cours d’utilisation) | Les utilisateurs ne peuvent pas accéder aux fichiers ou aux applications Microsoft 365<br>Les administrateurs peuvent accéder au centre d’administration Microsoft 365 pour acheter et gérer d’autres abonnements

## <a name="delete-a-subscription"></a>Supprimer un abonnement

Vous pouvez placer un abonnement dans un état **Approvisionnement annulé** à supprimer trois jours après à l’aide du centre d’administration Microsoft 365.

1. Connectez-vous au [Centre d’administration Microsoft 365](https://admin.microsoft.com) avec un compte d’administrateur général pour votre organisation. Si vous essayez de supprimer l’organisation « Contoso » qui a le domaine initial par défaut contoso.onmicrosoft.com, connectez-vous avec un UPN tel que admin@contoso.onmicrosoft.com.

2. Affichez un aperçu du nouveau Centre d’administration Microsoft 365 en vous assurant que la bascule **Découvrez le nouveau centre d’administration** est activée.

   ![Afficher un aperçu de la nouvelle expérience du centre d’administration M365](./media/directory-delete-howto/preview-toggle.png)

3. Une fois le nouveau centre d’administration activé, vous devez annuler un abonnement pour pouvoir le supprimer. Sélectionnez **Facturation** et sélectionnez **Produits et services**, puis sélectionnez **Annuler l’abonnement** pour l’abonnement que vous souhaitez annuler. Vous êtes redirigé vers une page de commentaires.

   ![Choisir l’abonnement à annuler](./media/directory-delete-howto/cancel-choose-subscription.png)

4. Remplissez le formulaire de commentaires et sélectionnez **Annuler l’abonnement** pour annuler l’abonnement.

   ![Commande Annuler dans la version préliminaire de l’abonnement](./media/directory-delete-howto/cancel-command.png)

5. Vous pouvez maintenant supprimer l’abonnement. Sélectionnez **Supprimer** pour l’abonnement que vous souhaitez supprimer. Si vous ne trouvez pas l’abonnement dans la page **Produits et services** , assurez-vous que **État de l'abonnement** est défini sur **Tous**.

   ![Supprimer un lien pour supprimer l’abonnement](./media/directory-delete-howto/delete-command.png)

6. Sélectionnez **Supprimer l’abonnement** pour supprimer l’abonnement et accepter les conditions générales. Toutes les données sont définitivement supprimées dans un délai de trois jours. Vous pouvez [réactiver l’abonnement](/office365/admin/subscriptions-and-billing/reactivate-your-subscription) pendant la période de trois jours si vous changez d’avis.
  
   ![Lisez attentivement les termes et conditions](./media/directory-delete-howto/delete-terms.png)

7. À présent, l’état de l’abonnement a changé et il est marqué pour suppression. L’abonnement passe à l’état **Déprovisionné** 72 heures plus tard.

8. Une fois que vous avez supprimé un abonnement dans votre organisation et que le délai de 72 heures s’est écoulé, vous pouvez vous reconnecter au centre d’administration Azure AD pour vérifier qu’aucune action n’est nécessaire et qu’aucun abonnement ne bloque plus la suppression de votre organisation. Vous devriez alors pouvoir supprimer votre organisation Azure AD.
  
   ![écran de vérification d’abonnement pour la suppression](./media/directory-delete-howto/delete-checks-passed.png)

## <a name="i-have-a-trial-subscription-that-blocks-deletion"></a>J’ai un abonnement d’évaluation qui bloque la suppression

L’inscription à certains produits tels que Microsoft Power BI, Rights Management Services, Microsoft Power Apps ou Dynamics 365, est en [libre-service](/office365/admin/misc/self-service-sign-up). Des utilisateurs individuels peuvent s’inscrire par le biais de Microsoft 365, qui crée également un utilisateur invité pour l’authentification dans votre organisation Azure AD. Afin d’éviter toute perte de données, ces produits en libre-service bloquent la suppression de l’organisation tant qu’ils ne sont pas complètement supprimés de celle-ci. Peu importe que l’utilisateur se soit inscrit de lui-même, ou qu’on lui ait attribué le produit : seul l’administrateur Azure AD pourra supprimer un tel abonnement.

Il existe deux types de produits disponibles à l’inscription en libre service, selon la façon dont ils sont attribués : 

* Attribution au niveau de l’organisation : Un administrateur Azure AD attribue le produit à toute l’organisation et un utilisateur peut utiliser le service grâce à cette attribution, même s’il ne possède pas la licence de ce produit.
* Attribution de niveau de l’utilisateur : Un utilisateur effectue une inscription en libre-service et s’attribue le produit, sans l’assistance d’un administrateur. Quand l’organisation est gérée par un administrateur (voir [Prendre le contrôle d’une organisation non gérée en tant qu’administrateur](domains-admin-takeover.md)), celui-ci peut attribuer directement le produit à des utilisateurs, sans inscription en libre-service.  

Lorsque vous déclenchez la suppression d’un produit attribué à des utilisateurs par une inscription en libre-service, cette action supprime définitivement les données et tous les accès utilisateur du service. Tous les utilisateurs a qui ce produit a été attribué, que ce soit individuellement ou au niveau de l’organisation, ne peuvent plus s’y connecter, ni accéder à leurs données liées à ce produit. Si vous souhaitez éviter toute perte de données liées à un produit attribué par une inscription en libre-service, comme [des tableaux de bord Microsoft Power BI](/power-bi/service-export-to-pbix) ou [une configuration de stratégie Rights Management Services](/azure/information-protection/configure-policy#how-to-configure-the-azure-information-protection-policy), avant de déclencher la suppression, vérifiez que les données sont sauvegardées et enregistrées ailleurs.

Pour plus d’informations sur les services et produits disponibles avec une inscription en libre-service, consultez la page [Programmes libre-service disponibles](/office365/admin/misc/self-service-sign-up#available-self-service-programs).

S’il s’agit de l’expiration d’un abonnement d’essai à Microsoft 365 (à l’exception des programmes payants Partenaire/CSP, Contrat Entreprise ou Licence en volume), consultez le tableau suivant. Pour plus d’informations sur la conservation des données et le cycle de vie des abonnements Microsoft 365, consultez [Qu’arrive-t-il à mes données et à mon accès à la fin de mon abonnement Microsoft 365 pour les entreprises ?](/office365/admin/subscriptions-and-billing/what-if-my-subscription-expires).

État du produit | Données | Accès aux données
------------- | ---- | --------------
Actif (30 jours pour l’essai gratuit) | Données accessibles à tous | Les utilisateurs peuvent s’inscrire normalement en libre-service pour accéder aux produits, fichiers ou applications<br>Les administrateurs ont un accès normal au centre d’administration et aux ressources Microsoft 365
Deleted | Données supprimées | Les utilisateurs ne peuvent pas s’inscrire en libre-service pour accéder aux produits, fichiers ou applications<br>Les administrateurs peuvent accéder au centre d’administration Microsoft 365 pour acheter et gérer d’autres abonnements

## <a name="how-can-i-delete-a-self-service-sign-up-product-in-the-azure-portal"></a>Comment puis-je supprimer une inscription à un produit en libre-service dans le portail Microsoft Azure ?

Vous pouvez définir un produit accessible via une inscription en libre-service, comme Microsoft Power BI ou Azure Rights Management Services, dans un état **Supprimer**. Il sera alors immédiatement supprimé du portail Microsoft Azure AD.

1. Connectez-vous au [centre d’administration Azure AD](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) avec un compte d’administrateur général de votre organisation. Si vous essayez de supprimer l’organisation « Contoso » qui a le domaine initial par défaut contoso.onmicrosoft.com, connectez-vous avec un UPN tel que admin@contoso.onmicrosoft.com.

2. Sélectionnez **Licences**, puis **Produits à l’inscription en libre-service**. Vous pouvez afficher tous les produits disponibles à l’inscription en libre-service séparément des abonnements par postes. Choisissez le produit que vous souhaitez supprimer définitivement. Voici un exemple dans Microsoft Power BI :

    ![Capture d’écran représentant la page « Licenses - Self-service sign-up products » (« Licences - Produits d’inscription en libre-service »).](./media/directory-delete-howto/licenses-page.png)

3. Sélectionnez **Supprimer** pour supprimer le produit, puis acceptez les termes du contrat stipulant que les données sont supprimées immédiatement et définitivement. Cette action de suppression supprime tous les utilisateurs et supprime l’accès de l’organisation au produit. Cliquez sur Oui pour poursuivre la suppression.  

    ![Capture d’écran représentant la page « Licenses - Self-service sign-up products » (« Licences - Produits d’inscription en libre-service ») dans laquelle la fenêtre « Delete self-service sign-up product » (« Supprimer le produit d’inscription en libre-service ») est ouverte.](./media/directory-delete-howto/delete-product.png)

4. Lorsque vous sélectionnez **Oui**, la suppression du produit accessible en libre-service sera lancée. Une notification s’affiche, pour vous indiquer que la suppression est en cours d’exécution.  

    ![Capture d’écran représentant la page « Licenses - Self-service sign-up products » (« Licences - Produits d’inscription en libre-service ») dans laquelle la notification « Deletion in progress » (« Suppression en cours ») est affichée.](./media/directory-delete-howto/progress-message.png)

5. À présent, l’état du produit dont l’inscription est disponible en libre-service est défini sur **Supprimé**. Normalement, lorsque vous actualisez la page **Produits à l’inscription en libre-service**, le produit n’y apparaît plus.  

    ![Capture d’écran représentant la page « Licenses - Self-service sign-up products » (« Licences - Produits d’inscription en libre-service ») avec le volet « Self-service sign-up product deleted » (« Produit d’inscription en libre-service supprimé ») à droite.](./media/directory-delete-howto/product-deleted.png)

6. Une fois que vous avez supprimé tous les produits, vous pouvez vous reconnecter au Centre d’administration Azure AD pour vérifier qu’aucune action n’est requise et qu’aucun produit ne bloque la suppression de votre organisation. Vous devriez alors pouvoir supprimer votre organisation Azure AD.

    ![le nom d’utilisateur est mal orthographié ou introuvable](./media/directory-delete-howto/delete-organization.png)

## <a name="next-steps"></a>Étapes suivantes

[Documentation Azure Active Directory](../index.yml)
