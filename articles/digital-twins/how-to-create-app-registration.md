---
title: Créer une inscription d’application
titleSuffix: Azure Digital Twins
description: Découvrez comment créer une inscription d’application Azure AD en tant qu’option d’authentification pour les applications clientes.
author: baanders
ms.author: baanders
ms.date: 10/13/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: f252471cd3cd7e3a950bf2cfe324e580da129041
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209074"
---
# <a name="create-an-app-registration-to-use-with-azure-digital-twins"></a>Créer une inscription d’application à utiliser avec Azure Digital Twins

Lors de l’utilisation d’une instance Azure Digital Twins, il est courant d’interagir avec cette instance par le biais d’applications clientes, comme une application cliente personnalisée ou un exemple de type [ADT Explorer](quickstart-adt-explorer.md). Ces applications doivent s’authentifier auprès d’Azure Digital Twins pour interagir avec, et certains des [mécanismes d’authentification](how-to-authenticate-client.md) que les applications peuvent utiliser impliquent une **inscription d’application** [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md).

Cela n’est pas obligatoire pour tous les scénarios d’authentification. Toutefois, si vous utilisez une stratégie d’authentification ou un exemple de code qui nécessite une inscription d’application, y compris un **ID de client** et un **ID de locataire** , cet article vous montre comment en définir une.

## <a name="using-azure-ad-app-registrations"></a>Utilisation des inscriptions d’application Azure AD

[Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) est le service Microsoft basé sur le cloud qui gère les identités et les accès. La configuration d’une **inscription d’application** dans Azure AD est un moyen d’accorder à une application cliente l’accès à Azure Digital Twins.

Cette inscription d’application est l’emplacement où vous configurez les autorisations d’accès aux [API d’Azure Digital Twins](how-to-use-apis-sdks.md). Plus tard, les applications clientes peuvent s’authentifier auprès de l’inscription d’application à l’aide des **valeurs d’ID client et de locataire** de l’inscription, et ainsi recevoir les autorisations d’accès configurées aux API.

>[!TIP]
> Vous préférerez peut-être configurer une nouvelle inscription d’application chaque fois que vous en aurez besoin, *ou* pour effectuer cette opération une seule fois, en établissant une inscription d’application unique qui sera partagée entre tous les scénarios qui l’exigent.

## <a name="create-the-registration"></a>Création de l’inscription

Commencez par accéder à [Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) dans le portail Azure (vous pouvez utiliser ce lien ou le trouver à l’aide de la barre de recherche du portail). Sélectionnez *Inscriptions d’applications* dans le menu service, puis *+ Nouvelle inscription* .

:::image type="content" source="media/how-to-create-app-registration/new-registration.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Dans la page *Inscrire une application* qui suit, renseignez les valeurs demandées :
* **Name**  : Nom d’affichage d’application Azure AD à associer à l’inscription.
* **Types de comptes pris en charge**  : Select *Comptes dans cet annuaire organisationnel uniquement (Annuaire par défaut uniquement – Locataire unique)*
* **URI de redirection**  : *URL de réponse de l’application Azure AD* pour l’application Azure AD. Ajoutez un URI *Client public/natif (mobile et bureau)* pour `http://localhost`.

Lorsque vous avez terminé, cliquez sur le bouton *S’inscrire* .

:::image type="content" source="media/how-to-create-app-registration/register-an-application.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Une fois la configuration de l’inscription terminée, le portail vous redirige vers la page de détails correspondante.

## <a name="collect-client-id-and-tenant-id"></a>Collecter l’ID de client et l’ID de locataire

Ensuite, recueillez certaines valeurs importantes sur l’inscription d’application à partir de sa page de détails :

:::image type="content" source="media/how-to-create-app-registration/app-important-values.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Prenez note de _**l’ID d’application (client)**_ et de _**l’ID de répertoire (locataire)**_ affichés sur votre **page** . Il s’agit des valeurs dont une application cliente aura besoin pour s’authentifier auprès d’Azure Digital Twins.

## <a name="provide-azure-digital-twins-api-permission"></a>Fournir une autorisation d’API Azure Digital Twins

Ensuite, configurez l’inscription d’application que vous avez créée avec les autorisations de base pour les API Azure Digital Twins.

À partir de la page du portail pour l’inscription de votre application, sélectionnez *Autorisations des API* dans le menu. Sur la page d’autorisations suivantes, cliquez sur le bouton *+ Ajouter une autorisation* .

:::image type="content" source="media/how-to-create-app-registration/add-permission.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Dans la page *Demander des autorisations d’API* qui suit, basculez vers l’onglet *API utilisées par mon organisation* et recherchez *Azure Digital Twins* . Sélectionnez _**Azure Digital Twins**_ dans les résultats de la recherche pour continuer à attribuer des autorisations pour les API Azure Digital Twins.

:::image type="content" source="media/how-to-create-app-registration/request-api-permissions-1.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

>[!NOTE]
> Si votre abonnement dispose encore d’une instance Azure Digital Twins de la préversion publique précédente du service (avant juillet 2020), vous devez rechercher et sélectionner _**Azure Smart Spaces Services**_ à la place. Il s’agit d’un ancien nom pour le même ensemble d’API (notez que l’ *ID d’application (client)* est le même que dans la capture d’écran ci-dessus), et votre expérience ne sera pas modifiée au-delà de cette étape.
> :::image type="content" source="media/how-to-create-app-registration/request-api-permissions-1-smart-spaces.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Ensuite, vous allez sélectionner les autorisations à accorder pour ces API. Développez l’autorisation **Lecture (1)** , et activez la case *lecture.Écriture* pour accorder cette inscription d’application et les autorisations de lecture et d’écriture.

:::image type="content" source="media/how-to-create-app-registration/request-api-permissions-2.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Lorsque vous avez terminé, appuyez sur *Ajouter des autorisations* .

### <a name="verify-success"></a>Vérifier la réussite de l’exécution

Sur la page *Autorisations des API* , vérifiez qu’il existe désormais une entrée pour Azure Digital Twins reflétant les autorisations Lecture/Écriture :

:::image type="content" source="media/how-to-create-app-registration/verify-api-permissions.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Vous pouvez également vérifier la connexion à Azure Digital Twins dans le fichier *manifest.json* de l’inscription de l’application, qui a été automatiquement mise à jour avec les informations Azure Digital Twins lorsque vous avez ajouté les autorisations des API.

Pour ce faire, sélectionnez *Manifeste* dans le menu pour afficher le code du manifeste de l’inscription de l’application. Faites défiler la fenêtre de code vers le bas et recherchez ces champs sous `requiredResourceAccess`. Les valeurs doivent correspondre à celles de la capture d’écran ci-dessous :

:::image type="content" source="media/how-to-create-app-registration/verify-manifest.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::

Si ces valeurs sont manquantes, recommencez les étapes de la [section relative à l’ajout d’autorisations d’API](#provide-azure-digital-twins-api-permission).

## <a name="other-possible-steps-for-your-organization"></a>Autres étapes possibles pour votre organisation

Il est possible que votre organisation exige des actions supplémentaires de la part des propriétaires/administrateurs d’abonnement pour configurer correctement une inscription d’application. Les étapes requises peuvent varier en fonction des paramètres spécifiques de votre organisation.

Voici quelques activités courantes qu’un propriétaire/administrateur pour l’abonnement peut devoir effectuer. Ces opérations et d’autres peuvent être effectuées à partir de la page [*Inscriptions d’applications Azure AD*](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) dans le portail Azure.
* Accordez le consentement administrateur pour l’inscription d’application. Votre organisation peut avoir activé globalement l’option *Consentement administrateur requis* dans Azure AD pour toutes les inscriptions d’applications au sein de votre abonnement. Si c’est le cas, le propriétaire/l’administrateur devra sélectionner ce bouton pour votre société dans la page *Autorisations de l’API* de l’inscription d’application pour que celle-ci soit valide :

    :::image type="content" source="media/how-to-create-app-registration/grant-admin-consent.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::
  - Si le consentement a été accordé avec succès, l’entrée pour Azure Digital Twins doit alors indiquer une valeur d’ *État* _Accordé pour **(votre société)**_
   
    :::image type="content" source="media/how-to-create-app-registration/granted-admin-consent-done.png" alt-text="Vue de la page du service Azure AD dans le portail Azure, mettant en surbrillance l’option de menu « inscriptions d’applications » et le bouton « + Nouvelle inscription »":::
* Activer l’accès client public
* Définir des URL de réponse spécifiques pour l’accès web et au bureau
* Autoriser les flux d’authentification OAuth2 implicites

Pour plus d’informations sur l’inscription d’applications et ses différentes options d’installation, consultez [*Inscrire une application avec la plateforme d’identités Microsoft*](/graph/auth-register-app-v2).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous configurez une inscription d’application Azure AD qui peut être utilisée pour authentifier des applications clientes avec les API Azure Digital Twins.

Vous lirez ensuite des informations sur les mécanismes d’authentification, y compris ceux qui utilisent les inscriptions d’applications et d’autres qui ne le font pas :
* [*Guide pratique : Écrire le code d’authentification de l’application*](how-to-authenticate-client.md)