---
title: Résoudre les problèmes d’authentification unique basée sur SAML dans Azure Active Directory
description: Résolvez les problèmes liés à une application Azure AD configurée pour l’authentification unique basée sur SAML.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.openlocfilehash: 2e8508d4fceb1e7fb580350c726dd70eeaea31c6
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658874"
---
# <a name="troubleshoot-saml-based-single-sign-on-in-azure-active-directory"></a>Résoudre les problèmes d’authentification unique basée sur SAML dans Azure Active Directory
Si vous rencontrez un problème lors de la configuration d’une application, vérifiez que vous avez suivi toutes les étapes du didacticiel dédié à l’application. Dans la configuration de l’application, une documentation en ligne sur la configuration de l’application vous est proposée. Pour une aide pas à pas détaillée, vous pouvez accéder à la [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](../saas-apps/tutorial-list.md).

## <a name="cant-add-another-instance-of-the-application"></a>Impossible d’ajouter une autre instance de l’application
Pour ajouter une deuxième instance d’une application, vous devez être en mesure de :
-   Configurer un identificateur unique pour la deuxième instance. Vous ne pourrez pas configurer le même identificateur que celui utilisé pour la première instance.
-   Configurer un certificat différent de celui utilisé pour la première instance.

Si l’application ne prend en charge aucune de ces configurations, vous ne pourrez pas configurer une deuxième instance.

## <a name="cant-add-the-identifier-or-the-reply-url"></a>Impossible d’ajouter l’identificateur ou l’URL de réponse
Si vous n’êtes pas en mesure de configurer l’identificateur ou l’URL de réponse, vérifiez que les valeurs de l’identificateur et de l’URL de réponse correspondent aux modèles préconfigurés pour l’application.

Pour connaître les modèles préconfigurés pour l’application :
1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général** ou que **Coadministrateur**. Passez à l’étape 7. Si vous êtes déjà dans le panneau de configuration de l’application sur Azure AD.
2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.
3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4. Cliquez sur **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
5. Cliquez sur **Toutes les applications** pour afficher la liste complète de vos applications.
   * Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications.**
6. Sélectionnez l’application pour laquelle vous souhaitez configurer l’authentification unique.
7. Une fois l’application chargée, cliquez sur **Authentification unique** dans le menu de navigation de gauche de l’application.
8. Sélectionnez **Authentification basée sur SAML** dans la liste déroulante **Mode**.
9. Accédez à la zone de texte **Identificateur** ou **URL de réponse** sous la section **Domaine et URL**.
10. Il existe trois façons de connaître les modèles pris en charge pour l’application :
    * Dans la zone de texte, les modèles pris en charge s’affichent sous forme d’espace réservé. *Exemple :* <https://contoso.com>.
    * Si le modèle n’est pas pris en charge, un point d’exclamation rouge s’affiche lorsque vous essayez d’entrer la valeur dans la zone de texte. Si vous placez le pointeur de la souris sur le point d’exclamation rouge, les modèles pris en charge s’affichent.
    * Vous pouvez également obtenir des informations sur les modèles pris en charge dans le didacticiel dédié à l’application. Sous la section **Configurer Azure AD pour l’authentification unique**, accédez à l’étape de configuration des valeurs sous la section **Domaine et URL**.

Si les valeurs ne correspondent aux modèles préconfigurés sur Azure AD. Vous pouvez :
-   Travailler avec le fournisseur de l’application pour obtenir des valeurs qui correspondent au modèle préconfiguré sur Azure AD
-   Contacter l’équipe Azure AD à l’adresse <aadapprequest@microsoft.com> ou laisser un commentaire dans le didacticiel pour demander la mise à jour des modèles pris en charge pour l’application

## <a name="where-do-i-set-the-entityid-user-identifier-format"></a>Où définir le format d’EntityID (identificateur d’utilisateur)
Vous ne pouvez pas sélectionner le format d’EntityID (identificateur d’utilisateur) qu’Azure AD envoie à l’application dans la réponse après l’authentification de l’utilisateur.

Azure AD sélectionne le format de l’attribut NameID (Identificateur d’utilisateur) en fonction de la valeur sélectionnée ou du format demandé par l’application dans la demande d’authentification SAML. Pour plus d’informations, consultez l’article [Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md#authnrequest) dans la section NameIDPolicy.

## <a name="cant-find-the-azure-ad-metadata-to-complete-the-configuration-with-the-application"></a>Impossible de trouver les métadonnées Azure AD pour terminer la configuration avec l’application
Pour télécharger les métadonnées ou le certificat de l’application à partir d’Azure AD, procédez comme suit :
1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général** ou que **Coadministrateur**.
2. Ouvrez **l’extension Azure Active Directory** en cliquant sur **Tous les services** en haut du menu de navigation principal de gauche.
3. Tapez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
4. Cliquez sur **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
5. Cliquez sur **Toutes les applications** pour afficher la liste complète de vos applications.
   * Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications.**
6. Sélectionnez l’application pour laquelle vous avez configuré l’authentification unique.
7. Une fois l’application chargée, cliquez sur **Authentification unique** dans le menu de navigation de gauche de l’application.
8. Accédez à la section **Certificat de signature SAML**, puis cliquez sur la valeur de colonne **Télécharger**. En fonction de ce que l’application nécessite pour configurer l’authentification unique, vous voyez soit l’option de téléchargement des métadonnées XML, soit le certificat.

Azure AD ne fournit pas d’URL permettant d’obtenir les métadonnées. Les métadonnées peuvent uniquement être récupérées sous forme de fichier XML.

## <a name="customize-saml-claims-sent-to-an-application"></a>Personnaliser les revendications SAML envoyées à une application
Pour savoir comment personnaliser les revendications d’attribut SAML envoyées à votre application, consultez [Claims mapping in Azure Active Directory (public preview)](../develop/active-directory-claims-mapping.md) (Mappage de revendications dans Azure Active Directory [préversion]) pour plus d’informations.

## <a name="next-steps"></a>Étapes suivantes
* [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)