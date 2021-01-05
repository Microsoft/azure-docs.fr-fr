---
title: Considérations relatives au déploiement de la réinitialisation de mot de passe en libre-service Azure Active Directory
description: Découvrez les points à prendre en compte pour le déploiement et la bonne stratégie d’implémentation de la réinitialisation de mot de passe en libre-service Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: baselden
author: barbaraselden
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 765bfe0f805ae4219110d689e8f7f8fd54a26cf3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173934"
---
# <a name="plan-an-azure-active-directory-self-service-password-reset-deployment"></a>Planification du déploiement de la réinitialisation de mot de passe en libre-service Azure Active Directory

> [!IMPORTANT]
> Ce plan de déploiement fournit des conseils et présente les meilleures pratiques à suivre pour déployer la réinitialisation de mot de passe en libre-service (SSPR) Azure AD.
>
> **Si vous êtes un utilisateur final et que vous devez revenir à votre compte, accédez à [https://aka.ms/sspr](https://aka.ms/sspr)** .

La [réinitialisation de mot de passe en libre-service (SSPR)](https://www.youtube.com/watch?v=tnb2Qf4hTP8) est une fonctionnalité d'Azure Active Directory qui permet aux utilisateurs de réinitialiser leurs mots de passe sans solliciter l'aide du personnel informatique. Les utilisateurs peuvent rapidement débloquer leur compte et continuer à travailler, quels que soient l'heure ou l'endroit où ils se trouvent. En permettant aux employés de débloquer leur compte eux-mêmes, votre organisation peut réduire les pertes de productivité et les coûts de support élevés liés aux problèmes de mot de passe les plus courants.

La SSPR comprend les fonctionnalités suivantes :

* Le libre-service permet aux utilisateurs finaux de réinitialiser leurs mots de passe expirés ou non, sans solliciter l'aide d'un administrateur ou du support technique.
* [La réécriture du mot de passe](./concept-sspr-writeback.md) permet de gérer les mots de passe locaux et de résoudre les problèmes de verrouillage des comptes via le cloud.
* Les rapports d'activité sur la gestion des mots de passe donnent aux administrateurs un aperçu de l'activité d'inscription et de réinitialisation des mots de passe au sein de leur organisation.

Ce guide de déploiement explique comment planifier, puis tester un déploiement de la SSPR.

Pour voir rapidement le fonctionnement de la SSPR, puis revenir en arrière afin d’examiner d’autres considérations relatives au déploiement :

> [!div class="nextstepaction"]
> [Activer la réinitialisation de mot de passe en libre-service (SSPR)](tutorial-enable-sspr.md)

## <a name="learn-about-sspr"></a>En savoir plus sur la SSPR

Apprenez-en davantage sur la SSPR. Consultez [Fonctionnement : Réinitialisation de mot de passe en libre-service Azure AD](./concept-sspr-howitworks.md).

### <a name="key-benefits"></a>Principaux avantages

Les principaux avantages de la SSPR sont les suivants :

* **Gestion des coûts**. La SSPR réduit les coûts de support informatique en permettant aux utilisateurs de réinitialiser eux-mêmes leurs mots de passe. Elle réduit également les frais liés au temps perdu suite à l'oubli des mots de passe et aux verrouillages. 

* **Expérience utilisateur intuitive**. La SSPR fournit un processus d'inscription utilisateur unique et intuitif qui permet aux utilisateurs de réinitialiser leurs mots de passe et de débloquer leurs comptes à la demande sur tout type d'appareil et en tout lieu. Elle permet aux utilisateurs de reprendre plus rapidement leur travail et de gagner en productivité.

* **Flexibilité et sécurité**. La SSPR permet aux entreprises de bénéficier de la sécurité et de la flexibilité d'une plateforme cloud. Les administrateurs peuvent modifier les paramètres pour répondre à de nouvelles exigences de sécurité et transmettre ces modifications aux utilisateurs sans interrompre leur connexion.

* **Robustesse de l'audit et du suivi de l'utilisation**. Une organisation peut garantir aux utilisateurs que ses systèmes resteront sécurisés pendant qu'ils réinitialisent leurs mots de passe. Particulièrement robustes, les journaux d'audit contiennent des informations sur chacune des étapes du processus de réinitialisation de mot de passe. Ces journaux sont disponibles à partir d'une API et permettent à l'utilisateur d'importer les données dans le système SIEM (Security Incident and Event Monitoring) de son choix.

### <a name="licensing"></a>Licence

Azure Active Directory fonctionne sur la base d'une licence unique par utilisateur, ce qui signifie que chaque utilisateur doit disposer d'une licence appropriée pour les fonctionnalités qu'il utilise. Pour la SSPR, nous recommandons une licence de groupe. 

Pour comparer les éditions et les fonctionnalités et en savoir plus sur l'activation des licences utilisateur ou de groupe, consultez [Conditions de licence pour la réinitialisation de mot de passe en libre-service Azure AD](./concept-sspr-licensing.md).

Pour plus d'informations sur les prix, consultez [Tarifs Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="prerequisites"></a>Prérequis

* Un locataire Azure AD fonctionnel avec au moins une licence d’essai active. Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Un compte avec des privilèges d’administrateur général.


### <a name="training-resources"></a>Ressources de formation

| Ressources| Lien et description |
| - | - |
| Vidéos| [Offrez une meilleure extensibilité informatique à vos utilisateurs](https://youtu.be/g9RpRnylxS8) 
| |[Qu'est-ce que la réinitialisation de mot de passe en libre-service ?](https://youtu.be/hc97Yx5PJiM)|
| |[Déployer la réinitialisation de mot de passe en libre-service](https://www.youtube.com/watch?v=Pa0eyqjEjvQ&index=18&list=PLLasX02E8BPBm1xNMRdvP6GtA6otQUqp0)|
| |[Configurer la réinitialisation de mot de passe en libre-service pour les utilisateurs dans Azure AD](https://azure.microsoft.com/resources/videos/self-service-password-reset-azure-ad/) |
| |[[Préparer les utilisateurs à] inscrire [leurs] informations de sécurité pour Azure Active Directory](https://youtu.be/gXuh0XS18wA) |
| Cours en ligne|[Gestion des identités dans Microsoft Azure Active Directory](https://www.pluralsight.com/courses/microsoft-azure-active-directory-managing-identities) Utilisez la SSPR pour offrir à vos utilisateurs une expérience moderne et protégée. Consultez notamment le module « [Gestion des utilisateurs et des groupes Azure Active Directory](https://app.pluralsight.com/library/courses/microsoft-azure-active-directory-managing-identities/table-of-contents) ». |
|Cours payants Pluralsight |[Les enjeux de la gestion des identités et des accès (IAM)](https://www.pluralsight.com/courses/identity-access-management-issues) Découvrez les enjeux auxquels votre organisation est confrontée en matière d'IAM et de sécurité. Consultez notamment le module « Autres méthodes d’authentification ».|
| |[Prise en main de la suite Microsoft Enterprise Mobility](https://www.pluralsight.com/courses/microsoft-enterprise-mobility-suite-getting-started) Découvrez les bonnes pratiques à suivre pour étendre les ressources locales au cloud afin de bénéficier de fonctionnalités d'authentification, d'autorisation et de chiffrement, ainsi que d'une expérience mobile sécurisée. Consultez notamment le module « Configuration des fonctionnalités avancées de Microsoft Azure Active Directory Premium ».
|Tutoriels |[Effectuer un déploiement pilote de la réinitialisation de mot de passe en libre-service pour Azure AD](./tutorial-enable-sspr.md) |
| |[Activation de la réécriture du mot de passe](./tutorial-enable-sspr-writeback.md) |
| |[Réinitialisation de mot de passe Azure AD à partir de l'écran de connexion de Windows 10](./howto-sspr-windows.md) |
| Questions fréquentes (FAQ)|[Forum Aux Questions sur la gestion des mots de passe](./active-directory-passwords-faq.md) |


### <a name="solution-architecture"></a>Architecture de solution

L'exemple suivant décrit l'architecture de la solution de réinitialisation de mot de passe pour les environnements hybrides courants.

![Diagramme de l'architecture de la solution](./media/howto-sspr-deployment//solutions-architecture.png)

Description du workflow

Pour réinitialiser le mot de passe, les utilisateurs doivent accéder au [portail de réinitialisation de mot de passe](https://aka.ms/sspr). Ils doivent utiliser la ou les méthodes d'authentification auxquels ils se sont précédemment inscrits pour prouver leur identité. S'ils réussissent à réinitialiser le mot de passe, ils entament le processus de réinitialisation.

* Utilisateurs du cloud uniquement : la SSPR stocke le nouveau mot de passe dans Azure AD. 

* Utilisateurs hybrides : la SSPR réécrit le mot de passe sur l'instance locale d'Active Directory via le service Azure AD Connect. 

Remarque : pour les utilisateurs qui ont désactivé la [Synchronisation de hachage du mot de passe (PHS)](../hybrid/whatis-phs.md), la SSPR stocke uniquement les mots de passe sur l'instance locale d'Active Directory.

### <a name="best-practices"></a>Meilleures pratiques

Vous pouvez aider les utilisateurs à s'inscrire rapidement en déployant la SSPR en parallèle d'une autre application ou d'un autre service populaire de l'organisation. Cette action génère un grand nombre de connexions et booste les inscriptions.

Avant de déployer la SSPR, vous pouvez déterminer le nombre et le coût moyen de chaque appel de réinitialisation de mot de passe. Vous pouvez utiliser ces données après le déploiement afin de connaître la valeur ajoutée de la SSPR pour l’organisation.

#### <a name="enable-combined-registration-for-sspr-and-mfa"></a>Activer une inscription combinée pour SSPR et MFA

Microsoft recommande aux organisations d’activer l’expérience combinée d’inscription à SSPR et MFA (authentification multifacteur). Lorsque vous activez cette expérience d’inscription combinée, les utilisateurs ne doivent sélectionner qu’une fois leurs informations d’inscription pour activer ces deux fonctionnalités.

L'expérience d'inscription combinée n'oblige pas les organisations à activer à la fois la SSPR et Azure AD MFA. L'inscription combinée permet aux organisations de bénéficier d'une meilleure expérience utilisateur. Pour plus d'informations, consultez [Inscription combinée d'informations de sécurité](concept-registration-mfa-sspr-combined.md).

## <a name="plan-the-deployment-project"></a>Planifier le projet de déploiement

Tenez compte des besoins de votre organisation lorsque vous déterminez la stratégie de ce déploiement dans votre environnement.

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Lorsque des projets technologiques échouent, cela est généralement dû à des attentes qui ne correspondent pas à l’impact, aux résultats et aux responsabilités réels. Pour éviter ces pièges, [veillez à impliquer les parties prenantes appropriées](../fundamentals/active-directory-deployment-plans.md) et à ce qu’elles comprennent bien leurs rôles dans le projet. Pour ce faire, dressez une liste de leurs contributions et de leurs responsabilités.

#### <a name="required-administrator-roles"></a>Rôles d’administrateur nécessaires


| Rôle métier/personnage| Rôle Azure AD (si nécessaire) |
| - | - |
| Support technique de niveau 1| Administrateur de mots de passe |
| Support technique de niveau 2| Administrateur d’utilisateurs |
| Administrateur SSPR| Administrateur général |


### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau service. Vous devez communiquer de manière proactive avec vos utilisateurs sur ce qui va changer, à quel moment les changements seront appliqués et comment ils peuvent obtenir de l’aide en cas de problème. Consultez les [documents sur le déploiement de la réinitialisation de mot de passe en libre-service dans le centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=56768). Vous y trouverez des suggestions pour planifier votre stratégie de communication avec les utilisateurs finaux.

### <a name="plan-a-pilot"></a>Prévoir un pilote

Nous vous recommandons d’effectuer la configuration initiale de la SSPR dans un environnement de test. Commencez avec un groupe pilote en activant la SSPR pour un sous-ensemble d'utilisateurs de votre organisation. Consultez [Meilleures pratiques pour un pilote](../fundamentals/active-directory-deployment-plans.md).

Pour créer un groupe, consultez [Créer un groupe et ajouter des membres dans Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md). 

## <a name="plan-configuration"></a>Planifier la configuration

Les paramètres suivants sont requis pour activer SSPR avec les valeurs recommandées.

| Domaine | Paramètre | Valeur |
| --- | --- | --- |
| **Propriétés SSPR** | Réinitialisation de mot de passe en libre-service activée | Groupe **sélectionné** pour le projet pilote / **Tous** pour la production |
| **Méthodes d’authentification** | Méthodes d’authentification requises pour l’inscription | Toujours 1 de plus que nécessaire pour la réinitialisation |
|   | Méthodes d’authentification requises pour la réinitialisation | Une ou deux |
| **Inscription** | Obliger les utilisateurs à s’inscrire durant la connexion ? | Oui |
|   | Nombre de jours avant que les utilisateurs ne soient invités à reconfirmer leurs informations d’authentification | 90 à 180 jours |
| **Notifications** | Notifier les utilisateurs lors des réinitialisations de mot de passe ? | Oui |
|   | Notifier tous les administrateurs quand d’autres administrateurs réinitialisent leur mot de passe ? | Oui |
| **Personnalisation** | Personnaliser le lien du support technique | Oui |
|   | URL ou adresse e-mail personnalisée du support technique | Adresse e-mail ou site du support technique |
| **Intégration locale** | Écriture différée des mots de passe dans AD en local | Oui |
|   | Autoriser les utilisateurs à déverrouiller leur compte sans réinitialiser leur mot de passe | Oui |

### <a name="sspr-properties"></a>Propriétés SSPR

Lors de l'activation de la SSPR, choisissez un groupe de sécurité approprié dans l'environnement pilote.

* Pour appliquer l'inscription SSPR à tout le monde, nous vous recommandons d'utiliser l'option **Tous**.
* Sinon, sélectionnez l'instance d'Azure AD ou le groupe de sécurité AD approprié.

### <a name="authentication-methods"></a>Méthodes d’authentification

Une fois la SSPR activée, les utilisateurs ne peuvent réinitialiser leur mot de passe que s'ils disposent de données dans les méthodes d'authentification activées par l'administrateur. Les méthodes disponibles incluent notamment le téléphone, la notification de l'application Authenticator et les questions de sécurité. Pour plus d'informations, consultez [Quelles sont les méthodes d'authentification ?](./concept-authentication-methods.md).

Nous recommandons les paramètres de méthode d'authentification suivants :

* Définissez les **méthodes d'authentification requises pour l'inscription** ; il doit y en avoir au moins une de plus que le nombre requis pour la réinitialisation. Plus vous en définissez, plus la flexibilité est grande pour les utilisateurs lorsqu'ils doivent réinitialiser leur mot de passe.

* Définissez le **nombre de méthodes requises pour la réinitialisation** sur un niveau approprié pour votre organisation. Le plus simple est d’en choisir une seule, tandis que deux peuvent améliorer l’état de la sécurité. 

Remarque : les méthodes d'authentification de l'utilisateur doivent être configurées dans [Stratégies et restrictions de mot de passe dans Azure Active Directory](./concept-sspr-policy.md).

### <a name="registration-settings"></a>Paramètres d’inscription

Définissez l’option **Obliger les utilisateurs à s’inscrire durant la connexion** sur **Oui**. Ce paramètre oblige les utilisateurs à s'inscrire au moment de la connexion, ce qui garantit leur protection.

Définissez le **nombre de jours avant que les utilisateurs soient invités à reconfirmer leurs informations d'authentification** sur une valeur comprise entre **90** et **180** jours, sauf si votre organisation a besoin d'un délai plus court.

### <a name="notifications-settings"></a>Paramètres de notifications

Configurer les paramètres **Notifier les utilisateurs lors des réinitialisations de mot de passe** et **Notifier tous les administrateurs quand d’autres administrateurs réinitialisent leur mot de passe** sur **Oui**. Sélectionnez **Oui** pour les deux paramètres afin de renforcer la sécurité en veillant à ce que les utilisateurs soient informés lorsque leur mot de passe est réinitialisé. Tous les administrateurs seront également informés lorsque l'un d'eux modifiera un mot de passe. Si les utilisateurs ou les administrateurs reçoivent une notification et qu'ils ne sont pas à l'origine de la modification, ils peuvent immédiatement signaler un problème de sécurité potentiel.

### <a name="customization-settings"></a>Paramètres de personnalisation

Il est essentiel de personnaliser l’adresse e-mail ou l’URL du support technique afin de permettre aux utilisateurs qui rencontrent des problèmes de bénéficier d’une aide immédiate. Définissez cette option sur une page web ou une adresse e-mail de support technique courante que vos utilisateurs connaissent. 

Pour plus d'informations, consultez [Personnaliser les fonctionnalités d'Azure AD pour la réinitialisation de mot passe en libre-service](./howto-sspr-customization.md).

### <a name="password-writeback"></a>Réécriture du mot de passe

La **réécriture du mot de passe** est activée avec [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) et elle inscrit en temps réel les réinitialisations de mot de passe effectuées dans le cloud dans un répertoire local existant. Pour plus d'informations, consultez [Qu'est-ce que la réécriture du mot de passe ?](./concept-sspr-writeback.md).

Nous recommandons les paramètres suivants :

* Vérifiez que l'option **Réécriture des mots sur l'instance locale d'AD** est définie sur **Oui**. 
* Définissez **Autoriser les utilisateurs à déverrouiller leur compte sans réinitialiser leur mot de passe** sur **Oui**.

Par défaut, Azure AD déverrouille les comptes quand il effectue une réinitialisation de mot de passe.

### <a name="administrator-password-setting"></a>Paramètre de mot de passe administrateur

Les comptes administrateur disposent d'autorisations élevées. Les administrateurs d'entreprise en local ou les administrateurs de domaine ne peuvent pas réinitialiser leurs mots de passe via la SSPR. Les comptes administrateur locaux présentent les restrictions suivantes :

* Leur mot de passe ne peut être modifié qu'à partir de leur environnement local.
* Il est impossible d'utiliser les questions et réponses secrètes comme méthode de réinitialisation de leur mot de passe.

Nous vous recommandons de ne pas synchroniser vos comptes administrateur Active Directory locaux avec Azure AD.

### <a name="environments-with-multiple-identity-management-systems"></a>Environnements avec plusieurs systèmes de gestion de l’identité

Certains environnements disposent de plusieurs systèmes de gestion des identités. Les gestionnaires d’identités locaux comme Oracle AM et SiteMinder exigent une synchronisation avec AD pour les mots de passe. Pour ce faire, vous pouvez utiliser un outil tel que le service de notification de modification de mot de passe (PCNS) avec Microsoft Identity Manager (MIM). Pour plus d’informations sur ce scénario plus complexe, consultez l’article [Déployer le service de notification de modification de mot de passe MIM sur un contrôleur de domaine](/microsoft-identity-manager/deploying-mim-password-change-notification-service-on-domain-controller).

## <a name="plan-testing-and-support"></a>Planifier les tests et le support

À chaque étape de votre déploiement, depuis les groupes pilotes initiaux jusqu'à l'organisation dans son ensemble, assurez-vous que les résultats sont conformes aux attentes.

### <a name="plan-testing"></a>Planifier les tests

Pour vous assurer que votre déploiement fonctionne comme prévu, planifiez un ensemble de cas de test pour valider l'implémentation. Pour évaluer les cas de test, il vous faut un utilisateur test non administrateur doté d'un mot de passe. Pour créer un utilisateur, consultez [Ajouter de nouveaux utilisateurs dans Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).

Le tableau suivant inclut des scénarios de test utiles que vous pouvez utiliser pour documenter les résultats attendus par votre organisation en fonction de vos stratégies.
<br>


| Cas métier| Résultats attendus |
| - | - |
| Le portail SSPR est accessible à partir du réseau d’entreprise| Déterminé par votre organisation |
| Le portail SSPR est accessible en dehors du réseau d’entreprise| Déterminé par votre organisation |
| Réinitialiser le mot de passe utilisateur à partir du navigateur lorsque l’utilisateur n’est pas activé pour la réinitialisation de mot de passe| L’utilisateur n’est pas en mesure d’accéder aux flux de réinitialisation de mot de passe |
| Réinitialiser le mot de passe utilisateur à partir du navigateur lorsque l’utilisateur n’est pas inscrit pour la réinitialisation de mot de passe| L’utilisateur n’est pas en mesure d’accéder aux flux de réinitialisation de mot de passe |
| L'utilisateur se connecte lorsqu'il est invité à procéder à l'inscription de la réinitialisation de mot de passe| Invite l'utilisateur à inscrire ses informations de sécurité |
| L'utilisateur se connecte lorsque l'inscription à la réinitialisation de mot de passe est terminée| Invite l'utilisateur à inscrire ses informations de sécurité |
| Le portail SSPR est accessible lorsque l’utilisateur ne dispose pas d’une licence| Accessible |
| Réinitialiser le mot de passe utilisateur à partir de l'écran de verrouillage d'appareil Windows 10 Azure AD joint ou Azure AD hybride joint| L’utilisateur peut réinitialiser le mot de passe |
| Les données d’utilisation et d’inscription SSPR sont disponibles pour les administrateurs quasiment en temps réel| Disponible via les journaux d’audit |

Vous pouvez également consulter [Effectuer un déploiement pilote de réinitialisation de mot de passe en libre-service pour Azure AD](./tutorial-enable-sspr.md). Dans ce tutoriel, vous activerez un déploiement pilote de la SSPR au sein de votre organisation et testerez celui-ci à l'aide d'un compte non administrateur.

### <a name="plan-support"></a>Planifier le support

Bien que la SSPR ne pose généralement aucun problème aux utilisateurs, il est important de préparer le personnel de support à gérer les éventuels soucis. Même si un administrateur peut réinitialiser le mot de passe des utilisateurs finaux via le portail Azure AD, il est préférable de résoudre le problème via un processus de support en libre-service.

Pour faciliter le travail de votre équipe de support, vous pouvez créer une FAQ à partir des questions que vous envoient les utilisateurs. Voici quelques exemples :

| Scénarios| Description |
| - | - |
| Aucune des méthodes d'authentification auxquelles l'utilisateur s'est inscrit n'est disponible| Un utilisateur essaie de réinitialiser son mot de passe, mais aucune des méthodes d’authentification qu’il a enregistrées n’est disponible (par exemple, il a laissé son téléphone portable à la maison et n’a pas accès à ses e-mails). |
| L'utilisateur ne reçoit ni SMS ni appel sur son téléphone mobile ou professionnel| Un utilisateur tente de confirmer son identité par SMS ou via un appel, mais ne reçoit pas de SMS/appel. |
| L'utilisateur n'a pas accès au portail de réinitialisation de mot de passe| Un utilisateur souhaite réinitialiser son mot de passe, mais il n'est pas activé pour la réinitialisation de mot de passe et n'a donc pas accès à la page de mise à jour des mots de passe. |
| L'utilisateur ne peut pas définir de nouveau mot de passe| Un utilisateur termine la vérification pendant le flux de réinitialisation de mot de passe, mais ne peut pas définir de nouveau mot de passe. |
| L'utilisateur ne voit pas de lien Réinitialiser le mot de passe sur un appareil Windows 10| Un utilisateur tente de réinitialiser son mot de passe à partir de l'écran de verrouillage de Windows 10, mais l'appareil n'est pas joint à Azure AD, ou la stratégie d'appareil Intune n'est pas activée. |

### <a name="plan-rollback"></a>Planifier la restauration

Pour restaurer le déploiement :

* Utilisateur individuel : supprimez l'utilisateur du groupe de sécurité 

* Groupe : supprimez le groupe de la configuration SSPR

* Tous : désactivez la SSPR pour le locataire Azure AD

## <a name="deploy-sspr"></a>Déployer la SSPR

Avant le déploiement, assurez-vous d'avoir effectué les opérations suivantes :

1. Création et lancement de l'exécution de votre [plan de communication](#plan-communications)

1. Définition des [paramètres de configuration](#plan-configuration) appropriés

1. Identification des utilisateurs et des groupes pour l'environnement [pilote](#plan-a-pilot) et l'environnement de production

1. [Définition des paramètres de configuration](#plan-configuration) pour l'inscription et le libre-service

1. [Réécriture du mot de passe configuré](#password-writeback) si vous disposez d'un environnement hybride


**Vous êtes maintenant prêt à déployer la SSPR !**

Consultez [Activer la réinitialisation de mot de passe en libre-service](./tutorial-enable-sspr.md#enable-self-service-password-reset) pour obtenir des instructions pas à pas sur la configuration des domaines suivants.

1. [Méthodes d’authentification](./concept-authentication-methods.md)

1. [Paramètres d'inscription](./concept-registration-mfa-sspr-combined.md)

1. [Paramètres des notifications](#notifications-settings)

1. [Paramètres de personnalisation](./howto-sspr-customization.md)

1. [Intégration locale](./tutorial-enable-sspr-writeback.md)

### <a name="enable-sspr-in-windows"></a>Activer SSPR sous Windows
Dans le cas des ordinateurs Windows 7, 8, 8.1 et 10, vous pouvez [autoriser les utilisateurs à réinitialiser leur mot de passe sur l’écran de connexion Windows](./howto-sspr-windows.md).

## <a name="manage-sspr"></a>Gérer SSPR

Azure AD peut fournir des informations supplémentaires sur vos performances SSPR par le biais d'audits et de rapports.

### <a name="password-management-activity-reports"></a>Rapports d'activité sur la gestion des mots de passe 

Vous pouvez utiliser les rapports prédéfinis du portail Azure pour mesurer les performances SSPR. Si vous disposez d’une licence appropriée, vous pouvez également créer des requêtes personnalisées. Pour plus d'informations, consultez [Options de création de rapports pour la gestion des mots de passe Azure AD](./howto-sspr-reporting.md).

> [!NOTE]
>  Vous devez être [administrateur général](../roles/permissions-reference.md), et devez accepter que ces données soient collectées pour votre organisation. Pour cela, vous devez consulter l'onglet Rapports ou les journaux d'audit du portail Azure au moins une fois. Tant que cela ne sera pas fait, les données ne seront pas collectées.

Les journaux d’audit pour l’inscription et la réinitialisation de mot de passe sont disponibles pendant 30 jours. Si l'audit de sécurité de votre entreprise requiert une conservation plus longue, les journaux doivent être exportés et utilisés dans un outil SIEM comme [Azure Sentinel](../../sentinel/connect-azure-active-directory.md), Splunk ou ArcSight.

![Capture d'écran de rapport SSPR](./media/howto-sspr-deployment/sspr-reporting.png)

### <a name="authentication-methods--usage-and-insights"></a>Méthodes d'authentification - Utilisation et insights

[Utilisation et insights](./howto-authentication-methods-usage-insights.md) vous permet de comprendre comment les méthodes d'authentification des fonctionnalités comme Azure AD MFA et SSPR fonctionnent au sein de votre organisation. Cette fonctionnalité de création de rapports permet à votre organisation d'identifier les méthodes inscrites et d'en savoir plus sur leur utilisation.

### <a name="troubleshoot"></a>Dépanner

* Consultez [Résoudre les problèmes liés à la réinitialisation de mot de passe en libre-service](./troubleshoot-sspr.md) 

* Suivez [Forum Aux Questions sur la gestion des mots de passe](./active-directory-passwords-faq.md) 

### <a name="helpful-documentation"></a>Documentation utile

* [Que sont les méthodes d’authentification ?](./concept-authentication-methods.md)

* [Fonctionnement : Réinitialisation de mot de passe en libre-service Azure AD](./concept-sspr-howitworks.md).

* [Personnaliser les fonctionnalités d'Azure AD pour la réinitialisation de mot de passe en libre-service](./howto-sspr-customization.md)

* [Stratégies et restrictions de mot de passe dans Azure Active Directory](./concept-sspr-policy.md)

* [Qu’est-ce que la réécriture du mot de passe ?](./concept-sspr-writeback.md)

## <a name="next-steps"></a>Étapes suivantes

* Pour entamer le déploiement de la SSPR, consultez [Activation de la réinitialisation de mot de passe en libre-service Azure AD](tutorial-enable-sspr.md)

* [Envisager d’implémenter la protection par mot de passe Azure AD](./concept-password-ban-bad.md)

* [Envisager d’implémenter le verrouillage intelligent Azure AD](./howto-password-smart-lockout.md)