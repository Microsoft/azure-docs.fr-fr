---
title: Guide pratique pour utiliser l’accès aux applications en libre-service dans Azure AD
description: Activer le libre-service pour que les utilisateurs puissent trouver des applications dans Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: japere,asteen
ms.openlocfilehash: d99f0021ee53e972dacfdadb3e488498a3fc4ad0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372461"
---
# <a name="how-to-use-self-service-application-access"></a>Guide pratique pour utiliser l’accès aux applications en libre-service

Pour permettre à vos utilisateurs de découvrir eux-mêmes des applications depuis leur page Mes applications, vous devez activer l’ **accès aux applications en libre-service** pour toutes les applications pour lesquelles vous souhaitez autoriser les utilisateurs à les découvrir eux-mêmes et en demander l’accès.

Cette fonctionnalité est un excellent moyen pour un groupe informatique d’économiser du temps et de l’argent, et elle est recommandée dans le cadre d’un déploiement d’applications modernes avec Azure Active Directory.

Pour en savoir plus sur l’utilisation de Mes applications du point de vue de l’utilisateur final, consultez l’[aide du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

À l’aide de cette fonctionnalité, vous pouvez :

-   Autoriser les utilisateurs à découvrir eux-mêmes des applications à partir de [Mes applications](https://myapps.microsoft.com/) sans déranger le groupe informatique.
-   Ajouter ces utilisateurs à un groupe préconfiguré pour vous permettre de voir qui a demandé l’accès, de retirer l’accès et de gérer les rôles qui leur ont été attribués.
-   Éventuellement, autoriser une personne à approuver les demandes d’accès aux applications à la place du groupe informatique.
-   Éventuellement, configurer jusqu'à 10 personnes qui peuvent autoriser l’accès à cette application.
-   Éventuellement, autoriser une personne à définir les mots de passe que ces utilisateurs peuvent utiliser pour se connecter à l’application.
-   Éventuellement, attribuer automatiquement directement un rôle d’application à des utilisateurs affectés en libre-service.

## <a name="enable-self-service-application-access-to-allow-users-to-find-their-own-applications"></a>Activer l’accès aux applications en libre-service pour permettre aux utilisateurs de rechercher leurs propres applications

L’accès aux applications en libre-service est un excellent moyen pour permettre aux utilisateurs de découvrir eux-mêmes des applications, et éventuellement de permettre au groupe d’entreprise d’approuver l’accès à ces applications. Vous pouvez autoriser le groupe d’entreprise à gérer les informations d’identification attribuées à ces utilisateurs dans le cadre d’une authentification unique par mot de passe, directement depuis leur page Mes applications.

Pour activer l’accès en libre-service à une application, procédez comme suit :
1. Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous en tant **qu’Administrateur général** .
2. Ouvrez l’ **extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
3. Tapez «  **Azure Active Directory**  » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory** .
4. Sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
5. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.
   * Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications.**
6. Sélectionnez l’application pour laquelle vous souhaitez activer l’accès en libre-service à partir de la liste.
7. Une fois l’application chargée, sélectionnez **Libre-service** dans le menu de navigation gauche de l’application.
8. Pour activer l’accès en libre-service à cette application, définissez l’option **Autoriser les utilisateurs à demander l’accès à cette application ?** sur **Oui.**
9. Ensuite, pour sélectionner le groupe auquel doivent être ajoutés les utilisateurs qui demandent l’accès à cette application, sélectionnez le sélecteur à côté de l’étiquette **À quel groupe les utilisateurs attribués doivent-ils être ajoutés ?** et sélectionnez un groupe.
10. **Facultatif :** Si vous souhaitez exiger une approbation d’entreprise avant d’accorder l’accès aux utilisateurs, définissez l’option **Demander une approbation avant d’accorder l’accès à cette application ?** sur **Oui** .
11. **Facultatif : Pour les applications utilisant uniquement l’authentification unique par mot de passe,** si vous souhaitez autoriser ces approbateurs d’entreprise à spécifier les mots de passe envoyés à cette application pour les utilisateurs approuvés, définissez l’option **Autoriser les approbateurs à définir les mots de passe de l’utilisateur pour cette application ?** sur **Oui** .
12. **Facultatif :** Spécifiez les approbateurs d’entreprise autorisés à approuver l’accès à cette application. Sélectionnez **Qui est autorisé à approuver l’accès à cette application ?** Sélectionnez ensuite jusqu’à 10 approbateurs d’entreprise individuels.
    * Les groupes ne sont pas pris en charge.
13. **Facultatif :** **Pour les applications qui exposent des rôles** , si vous souhaitez attribuer un rôle à des utilisateurs approuvés en libre-service, sélectionnez le sélecteur à côté de l’option **À quel rôle attribuer des utilisateurs dans cette application ?** pour sélectionner le rôle auquel ces utilisateurs doivent être affectés.
14. Sélectionnez le bouton **Enregistrer** en haut de l’écran pour terminer.

Lorsque vous avez terminé la configuration d’applications en libre-service, les utilisateurs peuvent accéder à [Mes applications](https://myapps.microsoft.com/) et sélectionner le bouton **+Ajouter** pour rechercher les applications pour lesquelles vous avez activé l’accès en libre-service. Les approbateurs d’entreprise reçoivent également une notification dans leur page [Mes applications](https://myapps.microsoft.com/). Vous pouvez leur envoyer une notification par e-mail les avertissant qu’un utilisateur a demandé l’accès à une application nécessitant leur approbation. 

Ces approbations prennent en charge les flux de travail avec approbation unique uniquement, ce qui signifie que si vous spécifiez plusieurs approbateurs, chaque approbateur peut approuver l’accès à l’application.

## <a name="things-to-check-if-self-service-isnt-working"></a>Éléments à vérifier si le libre-service ne fonctionne pas
-   Vérifiez que l’utilisateur ou le groupe a été activé pour demander l’accès aux applications en libre-service.
-   Vérifiez que l’utilisateur visite l’emplacement adéquat pour demander l’accès aux applications en libre-service. Les utilisateurs peuvent accéder à leur page [Mes applications](https://myapps.microsoft.com/) et sélectionner le bouton **+Ajouter** pour rechercher les applications pour lesquelles vous avez activé l’accès en libre-service.
-   Si l’accès aux applications en libre-service a été récemment configuré, essayez de vous reconnecter à la page Mes applications de l’utilisateur après quelques minutes pour voir si les modifications de l’accès en libre-service sont apparues.

## <a name="next-steps"></a>Étapes suivantes
[Configuration d’Azure Active Directory pour la gestion de groupe en libre-service](../enterprise-users/groups-self-service-management.md)
