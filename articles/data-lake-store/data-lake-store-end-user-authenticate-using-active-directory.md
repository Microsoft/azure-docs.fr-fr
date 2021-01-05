---
title: Authentification des utilisateurs finaux - Data Lake Storage Gen1 avec Azure AD
description: Découvrir comment authentifier les utilisateurs finaux auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref
ms.openlocfilehash: a5272f7d580a3f8a68afda9150a1e95c1807eba6
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103779"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>Authentification de l’utilisateur final auprès d’Azure Data Lake Storage Gen1 à l’aide d’Azure Active Directory
> [!div class="op_single_selector"]
> * [Authentification de l’utilisateur final](data-lake-store-end-user-authenticate-using-active-directory.md)
> * [Authentification de service à service](data-lake-store-service-to-service-authenticate-using-active-directory.md)
>
>

Azure Data Lake Storage Gen1 utilise Azure Active Directory pour l’authentification. Avant de créer une application qui fonctionne avec Data Lake Storage Gen1 ou Azure Data Lake Analytics, vous devez déterminer la manière dont vous voulez authentifier votre application dans Azure Active Directory (Azure AD). Les deux options principales disponibles sont :

* Authentification de l’utilisateur final (cet article)
* Authentification de service à service (choisissez cette option dans la liste déroulante ci-dessus)

En raison de ces deux options, votre application est fournie avec un jeton OAuth 2.0, qui est attaché à chaque requête adressée à Data Lake Storage Gen1 ou Azure Data Lake Analytics.

Cet article traite de la création d’une **application native Azure AD pour l’authentification de l’utilisateur final**. Pour obtenir des instructions sur la configuration de l’application Azure AD pour l’authentification de service à service, consultez [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory](./data-lake-store-service-to-service-authenticate-using-active-directory.md).

## <a name="prerequisites"></a>Prérequis
* Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* Votre ID d’abonnement. Vous pouvez le récupérer à partir du portail Azure. Par exemple, il est disponible à partir du panneau de compte Data Lake Storage Gen1.

    ![Obtenir l’ID d’abonnement](./media/data-lake-store-end-user-authenticate-using-active-directory/get-subscription-id.png)

* Votre nom de domaine Azure AD. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du Portail Azure. Dans la capture d’écran ci-dessous, le nom de domaine est **contoso.onmicrosoft.com**, et le GUID entre crochets est l’ID client.

    ![Obtenir le domaine AAD](./media/data-lake-store-end-user-authenticate-using-active-directory/get-aad-domain.png)

* Votre ID de client Azure. Pour obtenir des instructions pour récupérer l’ID de client, consultez [Obtenir l’ID de client](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).

## <a name="end-user-authentication"></a>Authentification des utilisateurs finaux
Ce mécanisme d’authentification est l’approche recommandée si vous souhaitez qu’un utilisateur final se connecte à votre application via Azure AD. Votre application est alors en mesure d’accéder aux ressources Azure avec le même niveau d’accès que l’utilisateur final qui s’est connecté. Votre utilisateur final doit fournir ses informations d’identification régulièrement pour que votre application maintienne l’accès.

Conséquence de la connexion de l’utilisateur final : votre application reçoit un jeton d’accès et un jeton d’actualisation. Le jeton d’accès est lié à chaque requête adressée au Data Lake Storage Gen1 ou à Data Lake Analytics et, par défaut, il est valide pendant une heure. Le jeton d’actualisation peut être utilisé pour obtenir un nouveau jeton d’accès et, par défaut, il est valide pendant deux semaines au maximum. Vous pouvez utiliser deux approches différentes pour la connexion de l’utilisateur final.

### <a name="using-the-oauth-20-pop-up"></a>Utilisation de la fenêtre contextuelle OAuth 2.0
Votre application peut déclencher une fenêtre contextuelle d’autorisation OAuth 2.0 dans laquelle l’utilisateur final peut entrer ses informations d’identification. Cette fenêtre contextuelle fonctionne également avec le processus d’authentification à 2 facteurs Azure AD (TFA), si nécessaire.

> [!NOTE]
> Cette méthode n’est pas encore prise en charge dans la bibliothèque d’authentification Azure AD (ADAL) pour Python ou Java.
>
>

### <a name="directly-passing-in-user-credentials"></a>Transmission directe des informations d’identification de l’utilisateur
Votre application peut fournir directement des informations d’identification de l’utilisateur à Azure AD. Cette méthode fonctionne uniquement avec les comptes d’utilisateur dotés d’ID d’organisation. Elle n’est pas compatible avec les comptes d’utilisateur personnels ou « live ID », notamment ceux se terminant par @outlook.com ou @live.com. En outre, cette méthode n’est pas compatible avec les comptes d’utilisateur qui nécessitent l’authentification à 2 facteurs Azure AD (TFA).

### <a name="what-do-i-need-for-this-approach"></a>De quoi ai-je besoin pour cette approche ?
* Votre nom de domaine Azure AD. Celui-ci est déjà répertorié dans les prérequis de cet article.
* ID de locataire Azure AD. Celui-ci est déjà répertorié dans les prérequis de cet article.
* Application native **Azure AD**
* ID de l’application native Azure AD
* URI de redirection de l’application native Azure AD
* Définir des autorisations déléguées


## <a name="step-1-create-an-active-directory-native-application"></a>Étape 1 : Créer une application native Active Directory

Créez et configurez une application native Azure AD pour l’authentification de l’utilisateur final auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory. Pour obtenir des instructions, consultez la page [Créer une application Azure AD](../active-directory/develop/howto-create-service-principal-portal.md).

Si vous suivez les instructions du lien, veillez à sélectionner le type d’application **Native**, comme l’illustre la capture d’écran suivante :

![Création d’une application web](./media/data-lake-store-end-user-authenticate-using-active-directory/azure-active-directory-create-native-app.png "Création d’une application native")

## <a name="step-2-get-application-id-and-redirect-uri"></a>Étape 2 : Obtenir l’ID et l’URI de redirection de l’application

Consultez [Obtenir l’ID d’application](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) pour récupérer l’ID d’application.

Pour récupérer l’URI de redirection, procédez comme suit.

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, cliquez sur **Inscriptions des applications**, puis cliquez sur l’application native Azure AD que vous venez de créer.

2. Dans le panneau **Paramètres** de l’application, cliquez sur **URI de redirection**.

    ![Obtenir un URI de redirection](./media/data-lake-store-end-user-authenticate-using-active-directory/azure-active-directory-redirect-uri.png)

3. Copiez la valeur affichée.


## <a name="step-3-set-permissions"></a>Étape 3 : Définir des autorisations

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, cliquez sur **Inscriptions des applications**, puis cliquez sur l’application native Azure AD que vous venez de créer.

2. Dans le panneau **Paramètres** de l’application, cliquez sur **Autorisations requises**, puis sur **Ajouter**.

    ![Capture d’écran du panneau Paramètres avec l’option URI de redirection entourée et le panneau URI de redirection avec l’option URI réel entourée.](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-1.png)

3. Dans le panneau **Ajouter l’accès des API**, cliquez sur **Sélectionner une API**, puis sur **Azure Data Lake** et enfin sur **Sélectionner**.

    ![Capture d’écran du panneau Ajouter un accès d’API avec l’option Sélectionner une API entourée et le panneau Sélectionner une API avec l’option Azure Data Lake et l’option Sélectionner entourées.](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-2.png)

4.  Dans le panneau **Ajouter l’accès des API**, cliquez sur **Sélectionner des autorisations**, cochez la case pour accorder un **Accès complet à Data Lake Store**, puis cliquez sur **Sélectionner**.

    ![Capture d’écran du panneau Ajouter un accès d’API avec l’option Sélectionner les autorisations entourée et le panneau Activer l’accès avec l’option Disposer d’un accès complet au service Azure Data Lake et l’option Sélectionner entourées.](./media/data-lake-store-end-user-authenticate-using-active-directory/aad-end-user-auth-set-permission-3.png)

    Cliquez sur **Done**.

5. Répétez les deux dernières étapes pour accorder également des autorisations à **l’API Gestion des services Microsoft Azure**.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez créé une application native Azure AD et regroupé les informations nécessaires à vos applications clientes créées à l’aide du SDK .NET, du SDK Java, de l’API REST, etc. Vous pouvez à présent passer aux articles suivants qui traitent de l’utilisation de l’application web Azure AD, d’abord pour vous authentifier auprès de Data Lake Storage Gen1, et ensuite pour effectuer d’autres opérations sur le magasin.

* [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec le Kit SDK Java](data-lake-store-end-user-authenticate-java-sdk.md)
* [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec le Kit SDK .NET](data-lake-store-end-user-authenticate-net-sdk.md)
* [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec Python](data-lake-store-end-user-authenticate-python.md)
* [Authentification des utilisateurs finaux auprès de Data Lake Storage Gen1 avec l’API REST](data-lake-store-end-user-authenticate-rest-api.md)