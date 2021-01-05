---
title: Utiliser le rapport d’activité pour déplacer des applications AD FS vers Azure Active Directory | Microsoft Docs
description: Le rapport d’activité d’applications AD FS (Active Directory Federation Services) permet de migrer rapidement des applications d’AD FS vers Azure Active Directory (Azure AD). Cet outil de migration pour AD FS identifie la compatibilité avec Azure AD et fournit des instructions de migration.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 01/14/2019
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: 77a43d5bd5f2b228d5ed4384fc1efdca76f8ea0b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573882"
---
# <a name="use-the-ad-fs-application-activity-report-preview-to-migrate-applications-to-azure-ad"></a>Utiliser le rapport d’activité des applications AD FS (préversion) pour migrer des applications vers Azure AD

De nombreuses organisations utilisent les services de fédération Active Directory (AD FS) pour fournir une authentification unique aux applications clou. Il est avantageux de transférer vos applications AD FS vers Azure AD pour l’authentification, notamment en termes de gestion des coûts, de gestion des risques, de productivité, de conformité et de gouvernance. Toutefois, comprendre quelles applications sont compatibles avec Azure AD et identifier des étapes de migration spécifiques peut prendre beaucoup de temps.

Le rapport d’activité des applications AD FS (préversion) dans le portail Azure vous permet d’identifier rapidement les applications capables de migrer vers Azure AD. Il évalue la compatibilité de toutes les applications AD FS avec Azure AD, recherche tout problème éventuel et fournit des instructions sur la préparation d’applications individuelles pour la migration. Le rapport d’activité des applications AD FS vous offre les possibilités suivantes :

* **Découvrir des applications AD FS et définir l’ampleur de votre migration.** Le rapport d'activité des applications AD FS répertorie toutes les applications AD FS de votre organisation qui ont fait l'objet d'une connexion utilisateur active au cours des 30 derniers jours. Le rapport indique que les applications sont prêtes pour la migration vers Azure AD. Le rapport n'affiche pas dans AD FS les parties de confiance liées à Microsoft telles qu'Office 365. Par exemple, les parties de confiance portant le nom « urn:federation:MicrosoftOnline ».

* **Hiérarchiser les applications pour la migration.** Obtenez le nombre d’utilisateurs uniques qui se sont connectés à l’application au cours des 1, 7 ou 30 derniers jours afin de déterminer la criticité ou le risque de la migration de l’application.
* **Exécuter des tests de migration et résoudre des problèmes.** Le service de génération de rapports exécute automatiquement des tests pour déterminer si une application est prête pour la migration. Les résultats sont affichés dans le rapport d’activité des applications AD FS en tant qu’état de la migration. Si la configuration AD FS n'est pas compatible avec une configuration Azure AD, des conseils spécifiques vous sont donnés pour la configuration dans Azure AD.

Les données d’activité des applications AD FS sont disponibles pour les utilisateurs auxquels est attribué l’un des rôles d’administrateur suivants : administrateur général, lecteur de rapport, lecteur de sécurité, administrateur d’application ou administrateur d’application cloud.

## <a name="prerequisites"></a>Prérequis

* Votre entreprise doit utiliser actuellement AD FS pour accéder aux applications.
* Azure AD Connect Health doit être activé dans votre locataire Azure AD.
* L’agent Azure AD Connect Health pour AD FS doit être installé.
   * [Apprenez-en davantage sur l’intégrité d’Azure AD Connect Health](../hybrid/how-to-connect-health-adfs.md)
   * [Prise en main de la configuration d’Azure AD Connect Health et installation de l’agent AD FS](../hybrid/how-to-connect-health-agent-install.md)

>[!IMPORTANT] 
>Diverses raisons expliquent pourquoi toutes les applications attendues n'apparaissent pas après l'installation d'Azure AD Connect Health. Le rapport d'activité des applications AD FS affiche uniquement les parties de confiance AD ​​FS qui se sont connectées au cours des 30 derniers jours. De plus, le rapport n'affiche pas les parties de confiance liées à Microsoft telles qu'Office 365.

## <a name="discover-ad-fs-applications-that-can-be-migrated"></a>Découvrir les applications AD FS pouvant être migrées 

Le rapport d’activité des applications AD FS est disponible sur le portail Azure, sous les rapports **Utilisation et insights** d’Azure AD. Le rapport d’activité des applications AD FS analyse chaque application AD FS pour déterminer si elle peut être migrée telle quelle, ou si un examen supplémentaire est nécessaire. 

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un rôle d’administrateur ayant accès aux données d’activité des applications AD FS (administrateur général, lecteur de rapport, lecteur de sécurité, administrateur d’application ou administrateur d’application cloud).

2. Sélectionnez **Azure Active Directory**, puis **Applications d’entreprise**.

3. Sous **Activité**, sélectionnez **Utilisation et insights (préversion)** , puis **Activité des applications AD FS** pour ouvrir la liste de toutes les applications AD FS au sein de votre organisation.

   ![Activité des applications AD FS](media/migrate-adfs-application-activity/adfs-application-activity.png)

4. Pour chaque application figurant dans la liste d’activités des applications AD FS, affichez l’**État de la migration** :

   * **Prêt pour la migration** signifie que la configuration de l’application AD FS est entièrement prise en charge dans Azure AD et peut être migrée telle quelle.

   * **Révision nécessaire** signifie que certains paramètres de l’application peuvent être migrés vers Azure AD, mais que vous devez examiner les paramètres qui ne peuvent pas être migrés tels quels.

   * **Étapes supplémentaires nécessaires** signifie qu’Azure AD ne prend pas en charge certains paramètres de l’application, de sorte que l’ application ne peut pas être migrée dans son état actuel.

## <a name="evaluate-the-readiness-of-an-application-for-migration"></a>Évaluer l’état de préparation d’une application pour la migration 

1. Dans la liste d’activités des applications AD FS, cliquez sur l’état dans la colonne **État de la migration** pour ouvrir les détails de la migration. Vous voyez un résumé des tests de configuration réussis, ainsi que tout problème de migration potentiel.

   ![Détails de la migration](media/migrate-adfs-application-activity/migration-details.png)

2. Cliquez sur un message pour ouvrir les détails supplémentaires de la règle de migration. Pour obtenir la liste complète des propriétés testées, voir le tableau [ Tests de configuration d’application AD FS ](#ad-fs-application-configuration-tests) ci-dessous.

   ![Détails de la règle de migration](media/migrate-adfs-application-activity/migration-rule-details.png)

### <a name="ad-fs-application-configuration-tests"></a>Tests de configuration d’application AD FS

Le tableau suivant répertorie tous les tests de configuration effectués sur les applications AD FS.

|Résultats  |Réussite/Avertissement/Échec  |Description  |
|---------|---------|---------|
|Test-ADFSRPAdditionalAuthenticationRules <br> Au moins une règle non migrable a été détectée pour l’authentification supplémentaire.       | Réussite/Avertissement          | La partie de confiance a des règles pour demander une authentification MFA (authentification multifacteur). Pour passer à Azure AD, convertissez ces règles en stratégies d’accès conditionnel. Si vous utilisez une authentification MFA locale, nous vous recommandons de passer à Azure AD MFA. [En savoir plus sur l’accès conditionnel](../authentication/concept-mfa-howitworks.md).        |
|Test-ADFSRPAdditionalWSFedEndpoint <br> La partie de confiance a la valeur AdditionalWSFedEndpoint définie sur true.       | Réussite/Échec          | La partie de confiance dans AD FS autorise plusieurs points de terminaison d’assertion WS-Fed. Azure AD n’en prend en charge qu’un seul. Si vous êtes dans une situation où cela bloque la migration, [faites-le nous savoir](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/38695621-allow-multiple-ws-fed-assertion-endpoints).     |
|Test-ADFSRPAllowedAuthenticationClassReferences <br> La partie de confiance a défini AllowedAuthenticationClassReferences.       | Réussite/Échec          | Ce paramètre dans AD FS vous permet de spécifier si l’application est configurée pour autoriser uniquement certains types d’authentification. Nous vous recommandons d’utiliser l’accès conditionnel pour accéder à cette fonctionnalité.  Si vous êtes dans une situation où cela bloque la migration, [faites-le nous savoir](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/38695672-allow-in-azure-ad-to-specify-certain-authentication).  [En savoir plus sur l’accès conditionnel](../authentication/concept-mfa-howitworks.md).          |
|Test-ADFSRPAlwaysRequireAuthentication <br> AlwaysRequireAuthenticationCheckResult      | Réussite/Échec          | Ce paramètre dans AD FS vous permet de spécifier si l’application est configurée pour ignorer les cookies SSO et **Toujours demander l’authentification**. Dans Azure AD, vous pouvez gérer la session d’authentification à l’aide de stratégies d’accès conditionnel pour obtenir un comportement similaire. [En savoir plus sur la configuration de la gestion de session d’authentification avec l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md).          |
|Test-ADFSRPAutoUpdateEnabled <br> La partie de confiance a la valeur AutoUpdateEnabled définie sur true       | Réussite/Avertissement          | Ce paramètre dans AD FS vous permet de spécifier si AD FS est configuré pour mettre à jour automatiquement l’application en fonction des changements apportés aux métadonnées de fédération. Si Azure AD ne prend pas en charge ce paramètre actuellement, il ne devrait pas bloquer la migration de l’application vers Azure AD.           |
|Test-ADFSRPClaimsProviderName <br> La partie de confiance dispose de plusieurs ClaimsProviders activés       | Réussite/Échec          | Ce paramètre dans AD FS appelle les fournisseurs d’identité à partir desquels la partie de confiance accepte les revendications. Dans Azure AD, vous pouvez activer la collaboration externe à l’aide d’Azure AD B2B. [En savoir plus sur Azure AD B2B](../external-identities/what-is-b2b.md).          |
|Test-ADFSRPDelegationAuthorizationRules      | Réussite/Échec          | Des règles d’autorisation de délégation personnalisées sont définies pour l’application. Il s’agit d’un concept WS-Trust pris en charge par Azure AD à l’aide de protocoles d’authentification modernes, par exemple OpenID Connect et OAuth 2.0. [En savoir plus sur la plateforme d’identités Microsoft](../develop/v2-protocols-oidc.md).          |
|Test-ADFSRPImpersonationAuthorizationRules       | Réussite/Avertissement          | Des règles d’autorisation de délégation d’emprunt d’identité personnalisées sont définies pour l’application. Il s’agit d’un concept WS-Trust pris en charge par Azure AD à l’aide de protocoles d’authentification modernes, par exemple OpenID Connect et OAuth 2.0. [En savoir plus sur la plateforme d’identités Microsoft](../develop/v2-protocols-oidc.md).          |
|Test-ADFSRPIssuanceAuthorizationRules <br> Au moins une règle non migrable a été détectée pour IssuanceAuthorization.       | Réussite/Avertissement          | Des règles d’autorisation d’émission personnalisées sont définies pour l’application dans AD FS. Azure AD prend en charge cette fonctionnalité avec un accès conditionnel Azure AD. [En savoir plus sur l’accès conditionnel](../conditional-access/overview.md). <br> Vous pouvez également restreindre l’accès à une application en fonction des utilisateurs ou des groupes affectés à celle-ci. [En savoir plus sur l’affectation de l’accès aux applications à des utilisateurs et des groupes](./assign-user-or-group-access-portal.md).            |
|Test-ADFSRPIssuanceTransformRules <br> Au moins une règle non migrable a été détectée pour IssuanceTransform.       | Réussite/Avertissement          | Des règles de transformation d’émission personnalisées sont définies pour l’application dans AD FS. Azure AD prend en charge la personnalisation des revendications émises dans le jeton. Pour plus d’informations, consultez [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).           |
|Test-ADFSRPMonitoringEnabled <br> La partie de confiance a la valeur MonitoringEnabled définie sur true.       | Réussite/Avertissement          | Ce paramètre dans AD FS vous permet de spécifier si AD FS est configuré pour mettre à jour automatiquement l’application en fonction des changements apportés aux métadonnées de fédération. Si Azure AD ne prend pas en charge ce paramètre actuellement, il ne devrait pas bloquer la migration de l’application vers Azure AD.           |
|Test-ADFSRPNotBeforeSkew <br> NotBeforeSkewCheckResult      | Réussite/Avertissement          | AD FS autorise une asymétrie temporelle basé sur les heures NotBefore et NotOnOrAfter dans le jeton SAML. Azure AD le gère automatiquement par défaut.          |
|Test-ADFSRPRequestMFAFromClaimsProviders <br> La partie de confiance a la valeur RequestMFAFromClaimsProviders définie sur true.       | Réussite/Avertissement          | Ce paramètre dans AD FS détermine le comportement de l’authentification MFA quand l’utilisateur provient d’un autre fournisseur de revendications. Dans Azure AD, vous pouvez activer la collaboration externe à l’aide d’Azure AD B2B. Vous pouvez ensuite appliquer des stratégies d’accès conditionnel pour protéger l’accès invité. En savoir plus sur [Azure AD B2B](../external-identities/what-is-b2b.md) et l’[accès conditionnel](../conditional-access/overview.md).          |
|Test-ADFSRPSignedSamlRequestsRequired <br> La partie de confiance a la valeur SignedSamlRequestsRequired définie sur true       | Réussite/Échec          | L’application est configurée dans AD FS pour vérifier la signature de la demande SAML. Azure AD accepte une demande SAML signée. Toutefois, il ne vérifie pas la signature. Azure AD dispose de différentes méthodes de protection contre les appels malveillants. Par exemple, Azure AD utilise les URL de réponse configurées dans l’application pour valider la demande SAML. Azure AD envoie uniquement un jeton aux URL de réponse configurées pour l’application. Si vous êtes dans une situation où cela bloque la migration, [faites-le nous savoir](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/13394589-saml-signature).          |
|Test-ADFSRPTokenLifetime <br> TokenLifetimeCheckResult        | Réussite/Avertissement         | L’application est configurée pour une durée de vie de jeton personnalisée. La valeur par défaut pour AD FS est une heure. Azure AD prend en charge cette fonctionnalité avec un accès conditionnel. Pour plus d’informations, voir [Configurer la gestion de session d’authentification avec l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md).          |
|La partie de confiance est configurée pour chiffrer les revendications. Azure AD prend cela en charge       | Réussite          | Avec Azure AD, vous pouvez chiffrer le jeton envoyé à l’application. Pour en savoir plus, voir [Configurer le chiffrement des jetons SAML Azure AD](./howto-saml-token-encryption.md).          |
|EncryptedNameIdRequiredCheckResult      | Réussite/Échec          | L’application est configurée pour chiffrer la revendication nameID dans le jeton SAML. Avec Azure AD, vous pouvez chiffrer l’intégralité du jeton envoyé à l’application. Le chiffrement de revendications spécifiques n’est pas encore pris en charge. Pour en savoir plus, voir [Configurer le chiffrement des jetons SAML Azure AD](./howto-saml-token-encryption.md).         |

## <a name="check-the-results-of-claim-rule-tests"></a>Vérifier les résultats des tests de règle de revendication

Si vous avez configuré une règle de revendication pour l’application dans AD FS, l’expérience fournit une analyse précise pour toutes les règles de revendication. Vous verrez les règles de revendication qui peuvent être déplacées vers Azure AD et celles qui nécessitent un examen plus approfondi.

1. Dans la liste d’activités des applications AD FS, cliquez sur l’état dans la colonne **État de la migration** pour ouvrir les détails de la migration. Vous voyez un résumé des tests de configuration réussis, ainsi que tout problème de migration potentiel.

2. Sur la page **Détails de la règle de migration**, développez les résultats pour afficher des informations détaillées sur les problèmes de migration potentiels et obtenir des conseils supplémentaires. Pour obtenir une liste détaillée de toutes les règles de revendication testées, voir le tableau [Vérifier les résultats des tests de règle de revendication](#check-the-results-of-claim-rule-tests) ci-dessous.

   L’exemple ci-dessous montre les détails de la règle de migration IssuanceTransform. Il répertorie les parties spécifiques de la revendication qui doivent être examinées et traitées avant de pouvoir migrer l’application vers Azure AD.

   ![La règle de migration fournit des instructions supplémentaires](media/migrate-adfs-application-activity/migration-rule-details-guidance.png)

### <a name="claim-rule-tests"></a>Tests de règle de revendication

Le tableau suivant répertorie tous les tests de règle de revendication effectués sur les applications AD FS.

|Propriété  |Description  |
|---------|---------|
|UNSUPPORTED_CONDITION_PARAMETER      | L’instruction de condition utilise des expressions régulières pour évaluer si la revendication correspond à un modèle donné.  Pour obtenir une fonctionnalité similaire dans Azure AD, vous pouvez utiliser une transformation prédéfinie telle que IfEmpty(), StartWith(), Contains() ou autre. Pour plus d’informations, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).          |
|UNSUPPORTED_CONDITION_CLASS      | L’instruction de condition comprend plusieurs conditions qui doivent être évaluées avant d’exécuter l’instruction d’émission. Azure AD peut prendre en charge cette fonctionnalité avec les fonctions de transformation de la revendication dans lesquelles vous pouvez évaluer plusieurs valeurs de revendication.  Pour plus d’informations, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).          |
|UNSUPPORTED_RULE_TYPE      | La règle de revendication n’a pas pu être reconnue. Pour plus d’informations sur la configuration des revendications dans Azure AD, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).          |
|CONDITION_MATCHES_UNSUPPORTED_ISSUER      | L’instruction de condition utilise un émetteur qui n’est pas pris en charge dans Azure AD. Actuellement, Azure AD ne reçoit pas de revendications de magasins autres qu’Active Directory ou Azure AD. Si cela vous empêche de migrer des applications vers Azure AD, [faites-le nous savoir](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/38695717-allow-to-source-user-attributes-from-external-dire).         |
|UNSUPPORTED_CONDITION_FUNCTION      | L’instruction de condition utilise une fonction d’agrégation pour émettre ou ajouter une revendication, quel que soit le nombre de correspondances.  Dans Azure AD, vous pouvez évaluer l’attribut d’un utilisateur afin de déterminer la valeur à utiliser pour la revendication avec des fonctions telles que IfEmpty(), StartWith() et Contains() entre autres. Pour plus d’informations, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).          |
|RESTRICTED_CLAIM_ISSUED      | L’instruction de condition utilise une revendication restreinte dans Azure AD. Vous pouvez peut-être émettre une revendication restreinte, mais vous ne pouvez pas en modifier la source ou lui appliquer une transformation. Pour plus d’informations, voir [Personnaliser des revendications émises dans des jetons pour une application spécifique dans Azure AD](../develop/active-directory-claims-mapping.md).          |
|EXTERNAL_ATTRIBUTE_STORE      | La déclaration d’émission utilise un magasin d’attributs différent d’Active Directory. Actuellement, Azure AD ne reçoit pas de revendications de magasins autres qu’Active Directory ou Azure AD. Si cela vous empêche de migrer des applications vers Azure AD, [faites-le nous savoir](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/38695717-allow-to-source-user-attributes-from-external-dire).          |
|UNSUPPORTED_ISSUANCE_CLASS      | La déclaration d’émission utilise ADD pour ajouter des revendications à l’ensemble de revendications entrantes. Dans Azure AD, vous pouvez le configurer en tant que transformations de revendications multiples.  Pour plus d’informations, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-claims-mapping.md).         |
|UNSUPPORTED_ISSUANCE_TRANSFORMATION      | La déclaration d’émission utilise des expressions régulières pour transformer la valeur de la revendication à émettre. Pour obtenir une fonctionnalité similaire dans Azure AD, vous pouvez utiliser une transformation prédéfinie telle que Extract(), Trim(), ToLower ou autre. Pour plus d’informations, voir [Personnaliser des revendications émises dans le jeton SAML pour des applications d’entreprise](../develop/active-directory-saml-claims-customization.md).          |

## <a name="troubleshooting"></a>Dépannage

### <a name="cant-see-all-my-ad-fs-applications-in-the-report"></a>Le rapport n'affiche pas toutes mes applications AD FS

 Si vous avez installé le service Azure AD Connect Health mais que l'invite d'installation de celui-ci apparaît toujours ou que le rapport n'affiche pas toutes vos applications AD FS, il se peut que vous n'ayez pas d'applications AD FS actives ou que vos applications AD FS soient des applications Microsoft.
 
 Le rapport d'activité des applications AD FS répertorie toutes les applications AD FS de votre organisation qui ont fait l'objet d'une connexion utilisateur active au cours des 30 derniers jours. En outre, le rapport n'affiche pas dans AD FS les parties de confiance liées à Microsoft telles qu'Office 365. Par exemple, les parties de confiance portant le nom « urn:federation:MicrosoftOnline », « microsoftonline » ou « microsoft:winhello:cert:prov:server » n'apparaissent pas dans la liste.





## <a name="next-steps"></a>Étapes suivantes

- [Vidéo : Utilisation du rapport d’activité AD FS pour migrer une application](https://www.youtube.com/watch?v=OThlTA239lU)
- [Gestion des applications avec Azure Active Directory](what-is-application-management.md)
- [Gérer l’accès aux applications](what-is-access-management.md)
- [Fédération avec Azure AD Connect](../hybrid/how-to-connect-fed-whatis.md)
