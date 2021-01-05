---
title: Impossible d’accéder à cette erreur d’application d’entreprise avec l’application de proxy d’application
description: Comment résoudre les problèmes d’accès courants avec les applications Proxy d’application Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/21/2019
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0339e9f7688555b4d99c2d3255461b5675f642ff
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94649549"
---
# <a name="cant-access-this-corporate-application-error-when-using-an-application-proxy-application"></a>Erreur « Impossible d’accéder à cette application d’entreprise » lors de l’utilisation d’une application Proxy d’application

Cet article vous aide à résoudre les problèmes couramment associés à l’erreur « Impossible d’accéder à cette application d’entreprise » sur une application Proxy d’application Azure AD.

## <a name="overview"></a>Vue d’ensemble

Lorsque vous voyez cette erreur, recherchez le code d’état indiqué dans la page. Ce code est généralement l’un des suivants :

- **Dépassement du délai de la passerelle** : le service Proxy d’application ne peut pas atteindre le connecteur. Cette erreur indique généralement un problème lié à l’attribution du connecteur, au connecteur proprement dit ou aux règles de mise en réseau régissant le connecteur.
- **Passerelle incorrecte** : le connecteur ne peut pas atteindre l’application back-end. Cette erreur peut indiquer une configuration incorrecte de l’application.
- **Interdit** : l’utilisateur n’est pas autorisé à accéder à l’application. Cette erreur peut se produire si l’utilisateur n’est pas affecté à l’application dans Azure Active Directory ou s’il n’est pas autorisé à accéder à l’application sur le backend.

Pour trouver le code, examinez le champ « Code d’état » situé en bas à gauche du message d’erreur. Recherchez également d’éventuels conseils supplémentaires au bas de la page.

![Exemple : Erreur liée au dépassement du délai de la passerelle](./media/application-proxy-sign-in-bad-gateway-timeout-error/connection-problem.png)

Pour plus d’informations sur la résolution de la cause racine de ces erreurs et sur les suggestions de correction, consultez la section appropriée ci-dessous.

## <a name="gateway-timeout-errors"></a>Erreurs liées au dépassement du délai de la passerelle

Un dépassement du délai de la passerelle se produit quand le service ne peut pas atteindre le connecteur dans le délai imparti. Cette erreur se produit généralement quand une application est affectée à un groupe de connecteurs qui ne compte aucun connecteur opérationnel, ou quand certains ports nécessaires par le connecteur ne sont pas ouverts.

## <a name="bad-gateway-errors"></a>Erreurs liées à une passerelle incorrecte

Une passerelle incorrecte indique que le connecteur ne peut pas atteindre l’application back-end. Vérifiez que vous avez publié la bonne l’application. Les erreurs courantes à l’origine de ce problème sont :

- Faute de frappe ou erreur dans l’URL interne
- Non-publication de la racine de l’application Par exemple, publication de `http://expenses/reimbursement` lors d’une tentative d’accès à `http://expenses`
- Problèmes liés à la configuration de la délégation Kerberos contrainte
- Problèmes liés à l’application back-end

## <a name="forbidden-errors"></a>Erreurs liées à une interdiction

Si vous constatez une erreur liée à une interdiction, l’utilisateur n’a pas été affecté à l’application (soit dans Azure Active Directory, soit sur l’application backend).

Pour savoir comment affecter des utilisateurs à l’application dans Azure, consultez la [documentation sur la configuration](application-proxy-add-on-premises-application.md#test-the-application).

Si vous confirmez que l’utilisateur est affecté à l’application dans Azure, vérifiez la configuration de l’utilisateur dans l’application back-end. Si vous utilisez la délégation Kerberos contrainte ou l’authentification Windows intégrée, consultez notre page de dépannage consacrée à la délégation Kerberos contrainte pour obtenir des instructions.

## <a name="check-the-applications-internal-url"></a>Vérifier l’URL interne de l’application

La première chose à faire consiste à vérifier l’URL interne et à la corriger si nécessaire. Pour cela, ouvrez l’application par le biais d’**Applications d’entreprise**, puis sélectionnez le menu **Proxy d’application**. Vérifiez qu’il s’agit bien de l’URL interne utilisée sur votre réseau local pour accéder à l’application.

## <a name="check-the-application-is-assigned-to-a-working-connector-group"></a>Vérifier que l’application est affectée à un groupe de connecteurs opérationnel

Pour vérifier que l’application est affectée à un groupe de connecteurs opérationnel :

1. Ouvrez l’application dans le portail. Pour cela, accédez à **Azure Active Directory**, cliquez sur **Applications d’entreprise**, puis sur **Toutes les applications**. Ouvrez l’application, puis sélectionnez **Proxy d’application** dans le menu gauche.
1. Examinez le champ Groupe de connecteurs. Si le groupe ne comprend aucun connecteur actif, un avertissement s’affiche. Si vous ne voyez aucun avertissement, passez à la vérification de l’autorisation de tous les [ports nécessaires](application-proxy-add-on-premises-application.md).
1. Si le mauvais groupe de connecteurs apparaît, utilisez la liste déroulante pour sélectionner le bon groupe et vérifiez l’absence d’avertissements. Si le groupe de connecteurs souhaité apparaît, cliquez sur le message de l’avertissement pour ouvrir la page dans l’outil de gestion des connecteurs.
1. À partir de là, plusieurs options s’offrent à vous :

   - Déplacer un connecteur actif dans le groupe : si vous avez un connecteur actif qui doit appartenir à ce groupe et qui dispose d’une visibilité directe sur l’application back-end cible, vous pouvez déplacer le connecteur dans le groupe affecté. Pour cela, cliquez sur le connecteur. Dans le champ « Groupe de connecteurs », utilisez la liste déroulante pour sélectionner le groupe approprié, puis cliquez sur Enregistrer.
   - Télécharger un nouveau connecteur pour ce groupe : cette page contient un lien pour [télécharger un nouveau connecteur](https://download.msappproxy.net/Subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/Connector/Download). Installez le connecteur sur une machine avec une ligne de vue directe sur l’application backend. En règle générale, le connecteur est installé sur le même serveur que l’application. Utilisez le lien Télécharger le connecteur pour télécharger un connecteur sur l’ordinateur cible. Cliquez ensuite sur le connecteur, puis utilisez la liste déroulante « Groupe de connecteurs » pour vérifier qu’il appartient au bon groupe.
   - Résoudre le problème d’un connecteur inactif : si un connecteur est répertorié comme étant inactif, il ne peut pas atteindre le service. Cette erreur vient généralement du fait que certains ports nécessaires sont bloqués. Pour résoudre ce problème, passez à la vérification de l’autorisation de tous les ports nécessaires.

Après avoir suivi ces étapes pour vérifier que l’application est affectée à un groupe comprenant des connecteurs opérationnels, retestez l’application. Si elle ne fonctionne toujours pas, passez à la section suivante.

## <a name="check-all-required-ports-are-open"></a>Vérifier que tous les ports nécessaires sont ouverts

Vérifiez que tous les ports nécessaires sont ouverts. Pour connaître les ports obligatoires, consultez la section ports ouverts du [Didacticiel : Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md). Si tous les ports nécessaires sont ouverts, passez à la section suivante.

## <a name="check-for-other-connector-errors"></a>Rechercher d’autres erreurs liées aux connecteurs

Si aucune des procédures précédentes ne résout le problème, l’étape suivante consiste à rechercher des problèmes ou des erreurs liés au connecteur proprement dit. Certaines erreurs courantes sont répertoriées dans le [document de dépannage](./application-proxy-troubleshoot.md#connector-errors).

Vous pouvez également consulter directement les journaux d’activité du connecteur pour identifier d’éventuelles erreurs. De nombreux messages d’erreur partagent des recommandations spécifiques pour y remédier. Pour voir les journaux d’activité, consultez la [documentation sur les connecteurs](application-proxy-connectors.md#under-the-hood).

## <a name="additional-resolutions"></a>Résolutions supplémentaires

Si les procédures précédentes ne vous ont pas permis de résoudre le problème, il existe d’autres causes possibles. Pour identifier le problème :

Si votre application est configurée pour utiliser l’authentification Windows intégrée, testez l’application sans l’authentification unique. Dans le cas contraire, passez au paragraphe suivant. Pour tester l’application sans l’authentification unique, ouvrez votre application par le biais d’**Applications d’entreprise** et accédez au menu **Authentification unique**. Remplacez, dans la liste déroulante, « Authentification Windows intégrée » par « Authentification unique Azure AD désactivée ».

À présent, ouvrez un navigateur et réessayez d’accéder à l’application. Une fois authentifié, vous devriez pouvoir accéder à l’application. Si l’authentification ne fonctionne pas, le problème provient de la délégation Kerberos contrainte qui prend en charge l’authentification unique. Pour plus d’informations, consultez la page de dépannage consacrée à la délégation Kerberos contrainte.

Si l’erreur persiste, accédez à l’ordinateur sur lequel le connecteur est installé, ouvrez un navigateur et essayez d’atteindre l’URL interne utilisée pour l’application. Le connecteur agit comme un autre client à partir du même ordinateur. Si vous ne pouvez pas atteindre l’application, cherchez à savoir pourquoi cet ordinateur vous en empêche ou utilisez un connecteur sur un serveur capable d’accéder à l’application.

Si vous pouvez atteindre l’application à partir de cet ordinateur, recherchez des problèmes ou des erreurs liés au connecteur proprement dit. Certaines erreurs courantes sont répertoriées dans le [document de dépannage](application-proxy-troubleshoot.md#connector-errors). Vous pouvez également consulter directement les journaux d’activité du connecteur pour identifier d’éventuelles erreurs. La plupart des messages d’erreur indiquent des recommandations plus spécifiques sur les corrections à apporter. Pour savoir comment afficher les journaux d’activité, consultez [notre documentation sur les connecteurs](application-proxy-connectors.md#under-the-hood).

## <a name="next-steps"></a>Étapes suivantes

[Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md)