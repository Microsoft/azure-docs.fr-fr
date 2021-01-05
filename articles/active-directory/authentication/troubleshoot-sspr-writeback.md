---
title: Résoudre les problèmes relatifs à l’écriture différée de réinitialisation de mot de passe libre-service - Azure Active Directory
description: Découvrez comment résoudre les problèmes courants et les étapes de résolution pour l’écriture différée de réinitialisation de mot de passe libre-service dans Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 08/26/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4d6bf4df1499d919cead0a184054e5ba0db9c06e
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346598"
---
# <a name="troubleshoot-self-service-password-reset-writeback-in-azure-active-directory"></a>Résoudre les problèmes relatifs à l’écriture différée de réinitialisation de mot de passe libre-service dans Azure Active Directory

La réinitialisation de mot de passe en libre-service (SSPR) Azure Active Directory (Azure AD) permet aux utilisateurs de réinitialiser leur mot de passe dans le cloud. La réécriture du mot de passe est une fonctionnalité activée avec [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) qui permet aux modifications de mot de passe dans le cloud d’être réécrites sur un annuaire local existant en temps réel.

Si vous rencontrez des problèmes avec l’écriture différée SSPR, les étapes de résolution des problèmes et les erreurs courantes suivantes peuvent vous aider. Si vous ne trouvez pas la réponse à votre problème, [nos équipes de support technique sont toujours disponibles](#contact-microsoft-support) pour vous aider davantage.

## <a name="troubleshoot-connectivity"></a>Résoudre les problèmes de connectivité

Si vous rencontrez des problèmes de réécriture du mot de passe pour Azure AD Connect, passez en revue les étapes suivantes qui pourraient vous aider à résoudre le problème. Pour récupérer votre service, nous vous recommandons de suivre les étapes ci-dessous dans l’ordre :

* [Vérifier la connectivité réseau](#confirm-network-connectivity)
* [Redémarrer le service de synchronisation Azure AD Connect](#restart-the-azure-ad-connect-sync-service)
* [Désactiver et réactiver la fonctionnalité de réécriture du mot de passe](#disable-and-re-enable-the-password-writeback-feature)
* [Installer la dernière version d’Azure AD Connect](#install-the-latest-azure-ad-connect-release)
* [Résoudre les problèmes d’écriture différée du mot de passe](#common-password-writeback-errors)

### <a name="confirm-network-connectivity"></a>Vérifier la connectivité réseau

Le point de défaillance le plus courant est que le pare-feu, les ports de proxy, ou les délais d’inactivité sont mal configurés.

Pour Azure AD Connect version *1.1.443.0* et versions ultérieures, vous avez besoin d’un accès *HTTPS sortant* pour les adresses suivantes :

* *\*.passwordreset.microsoftonline.com*
* *\*.servicebus.windows.net*

[Points de terminaison GOV](https://docs.microsoft.com/azure/azure-government/compare-azure-government-global-azure#guidance-for-developers) Azure :

* *\*.passwordreset.microsoftonline.us*
* *\*.servicebus.usgovcloudapi.net*

Si vous avez besoin de plus de granularité, consultez la [liste des plages d’adresses IP des centres de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). Cette liste est mise à jour tous les mercredis et entre en vigueur le lundi suivant.

Pour plus d’informations, consultez les [conditions préalables à la connectivité pour Azure AD Connect](../hybrid/how-to-connect-install-prerequisites.md).

### <a name="restart-the-azure-ad-connect-sync-service"></a>Redémarrer le service de synchronisation Azure AD Connect

Pour résoudre les problèmes de connectivité ou d’autres problèmes temporaires rencontrés avec le service, procédez comme suit pour redémarrer le service de synchronisation d’Azure AD Connect :

1. En tant qu’administrateur sur le serveur qui exécute Azure AD Connect, sélectionnez **Démarrer**.
1. Entrez *services.msc* dans le champ de recherche, puis sélectionnez **Entrée**.
1. Recherchez l’entrée *Microsoft Azure AD Sync*.
1. Cliquez avec le bouton droit sur l’entrée du service, sélectionnez **Redémarrer**, puis attendez que l’opération soit terminée.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/service-restart.png" alt-text="Redémarrer le service de synchronisation Azure AD à l’aide de l’interface graphique utilisateur" border="false":::

Ces étapes rétablissent votre connexion avec Azure AD et devraient résoudre vos problèmes de connectivité.

Si le redémarrage du service de synchronisation Azure AD Connect ne résout pas votre problème, essayez de désactiver, puis de réactiver la fonctionnalité de réécriture du mot de passe dans la section suivante.

### <a name="disable-and-re-enable-the-password-writeback-feature"></a>Désactiver et réactiver la fonctionnalité de réécriture du mot de passe

Pour continuer à résoudre les problèmes, procédez comme suit pour désactiver, puis réactiver la fonctionnalité de réécriture du mot de passe :

1. En tant qu’administrateur sur le serveur qui exécute Azure AD Connect, ouvrez **l’assistant Configuration d’Azure AD Connect**.
1. Dans **Connexion à Azure AD**, entrez vos informations d’identification d’administrateur général d’Azure AD.
1. Dans **Se connecter à AD DS**, entrez vos informations d’identification d’administrateur locales pour Active Directory Domain Services.
1. Dans **Identification de vos utilisateurs uniquement**, sélectionnez le bouton **Suivant**.
1. Dans **Fonctionnalités facultatives**, décochez la case **Réécriture du mot de passe**.
1. Sélectionnez **Suivant** dans les pages restantes de la boîte de dialogue sans apporter de modification jusqu’à ce que vous atteigniez la page **Prêt pour la configuration**.
1. Vérifiez que la page **Prêt à configurer** affiche l’option *Réécriture du mot de passe* comme *Désactivée*. Sélectionnez le bouton vert **Configurer** pour valider vos modifications.
1. Dans **Terminé**, désactivez l’option **Synchroniser maintenant**, puis sélectionnez **Terminer** pour fermer l’Assistant.
1. Rouvrez **l’Assistant Configuration d’Azure AD Connect**.
1. Répétez les étapes 2 à 8, en sélectionnant cette fois l’option *Réécriture du mot de passe* dans la page **Fonctionnalités facultatives** pour réactiver le service.

Ces étapes rétablissent votre connexion avec Azure AD et devraient résoudre vos problèmes de connectivité.

Si la désactivation, puis la réactivation de la fonctionnalité de réécriture du mot de passe ne résout pas votre problème, réinstallez Azure AD Connect dans la section suivante.

### <a name="install-the-latest-azure-ad-connect-release"></a>Installer la dernière version d’Azure AD Connect

Réinstaller Azure AD Connect peut résoudre les problèmes de configuration et de connectivité entre Azure AD et votre environnement Active Directory Domain Services local. Nous vous recommandons d’effectuer cette étape uniquement après avoir essayé les étapes précédentes pour vérifier et résoudre les problèmes de connectivité.

> [!WARNING]
> Si vous avez personnalisé les règles de synchronisation par défaut, *sauvegardez-les avant de procéder à la mise à niveau, puis redéployez-les manuellement après avoir terminé*.

1. Téléchargez la dernière version d’Azure AD Connect sur le [Centre de téléchargement de Microsoft](https://go.microsoft.com/fwlink/?LinkId=615771).
1. Puisque vous avez déjà installé Azure AD Connect, effectuez une mise à niveau sur place pour mettre à jour votre installation d’Azure AD Connect vers la dernière version.

    Exécutez le package téléchargé et suivez les instructions à l’écran pour mettre à jour Azure AD Connect.

Ces étapes devraient rétablir votre connexion avec Azure AD et résoudre vos problèmes de connectivité.

Si l’installation de la dernière version du serveur Azure AD Connect ne résout pas votre problème, la dernière étape consiste à essayer la désactivation, puis la réactivation de la réécriture du mot de passe après avoir installé la dernière version.

## <a name="verify-that-azure-ad-connect-has-the-required-permissions"></a>Vérifier qu’Azure AD Connect a les autorisations nécessaires

Azure AD Connect a besoin de l’autorisation AD DS **Réinitialiser le mot de passe** pour effectuer une écriture différée du mot de passe. Pour savoir si Azure AD Connect a l’autorisation pour un compte d’utilisateur AD DS local donné, utilisez la fonctionnalité **d’autorisation effective de Windows** :

1. Connectez-vous au serveur Azure AD Connect et démarrez **Synchronization Service Manager** en sélectionnant **Démarrer** > **Service de synchronisation**.
1. Sous l’onglet **Connecteurs**, sélectionnez le connecteur **Active Directory Domain Services**, puis sélectionnez **Propriétés**.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/synchronization-service-manager.png" alt-text="Gestionnaire de service de synchronisation montrant comment modifier des propriétés" border="false":::
  
1. Dans la fenêtre indépendante, sélectionnez l’onglet **Se connecter à la forêt Active Directory** et prenez note de la propriété **Nom d’utilisateur**. Cette propriété est le compte AD DS utilisé par Azure AD Connect pour effectuer la synchronisation d’annuaire.

    Pour qu’Azure AD Connect effectue une écriture différée du mot de passe, il faut que le compte des services AD DS dispose de l’autorisation de réinitialiser le mot de passe. Pour vérifier les autorisations sur ce compte d’utilisateur, procédez comme suit.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/synchronization-service-manager-properties.png" alt-text="Rechercher le compte d’utilisateur Active Directory du service synchronisation" border="false":::
  
1. Connectez-vous à un contrôleur de domaine local et démarrez l’application **Utilisateurs et ordinateurs Active Directory**.
1. Sélectionnez **Affichage** et vérifiez que l’option **Fonctionnalités avancées** est activée.  

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-advanced-features.png" alt-text="Utilisateurs et ordinateurs Active Directory montrant les fonctionnalités avancées" border="false":::
  
1. Recherchez le compte d’utilisateur AD DS à vérifier. Cliquez avec le bouton droit sur le nom du compte et sélectionnez **Propriétés**.  
1. Dans la fenêtre indépendante, accédez à l’onglet **Sécurité** et sélectionnez **Avancé**.  
1. Dans la fenêtre indépendante **Paramètres de sécurité avancés pour Administrateur**, accédez à l’onglet **Accès effectif**.
1. Choisissez **Sélectionner un utilisateur**, sélectionnez le compte AD DS utilisé par Azure AD Connect, puis sélectionnez **Afficher l’accès effectif**.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-effective-access.png" alt-text="Onglet Accès effectif montrant le compte de synchronisation" border="false":::
  
1. Faites défiler vers le bas et recherchez **Réinitialiser le mot de passe**. Si l’entrée a une coche, le compte AD DS est autorisé à réinitialiser le mot de passe du compte d’utilisateur Active Directory sélectionné.  

    :::image type="content" source="./media/troubleshoot-sspr-writeback/check-permissions.png" alt-text="Validation du fait que le compte de synchronisation possède l’autorisation de réinitialisation du mot de passe" border="false":::

## <a name="common-password-writeback-errors"></a>Erreurs courantes de réécriture du mot de passe

Les problèmes suivants peuvent se produire avec la réécriture du mot de passe. Si vous avez l’une de ces erreurs, passez en revue la solution proposée et vérifiez si la réécriture du mot de passe fonctionne alors correctement.

| Error | Solution |
| --- | --- |
| Le service de réinitialisation de mot de passe ne démarre pas localement. L’erreur 6800 apparaît dans le journal des événements de l’application de la machine Azure AD Connect. <br> <br> Après l’intégration, les utilisateurs fédérés ou qui ont recours à la synchronisation du hachage de mot de passe ou à l’authentification directe ne parviennent pas à réinitialiser leurs mots de passe. | Quand la réécriture du mot de passe est activée, le moteur de synchronisation appelle la bibliothèque de réécriture pour effectuer la configuration (intégration) en communiquant avec le service d’intégration cloud. Toutes les erreurs rencontrées au cours de l’intégration ou pendant le démarrage du point de terminaison WCF (Windows Communication Foundation) pour la réécriture du mot de passe entraînent une consignation dans le journal des événements de votre machine Azure AD Connect. <br> <br> Pendant le redémarrage du service Azure AD Sync (ADSync), si l’écriture différée a été configurée, le point de terminaison WCF démarre. Mais, si le démarrage du point de terminaison échoue, l’événement 6800 est journalisé et le service de synchronisation peut démarrer. La présence de cet événement signifie que le point de terminaison de la réécriture du mot de passe n’a pas démarré. Les détails du journal des événements pour cet événement 6800, ainsi que les entrées du journal des événements générées par le composant PasswordResetService, indiquent pour quelles raisons vous ne pouvez pas démarrer le point de terminaison. Passez en revue les erreurs du journal des événements et essayez de redémarrer Azure AD Connect si la réécriture du mot de passe ne fonctionne toujours pas. Si le problème persiste, essayez de désactiver, puis de réactiver la réécriture du mot de passe.
| Lorsqu’un utilisateur tente de réinitialiser un mot de passe ou de déverrouiller un compte avec l’écriture différée du mot de passe activée, l’opération échoue. <br> <br> De plus, un événement est consigné dans le journal des événements d’Azure AD Connect : « Synchronization Engine returned an error (Le moteur de synchronisation a retourné une erreur) hr=800700CE, message=L’extension ou le nom de fichier est trop long » après l’opération de déverrouillage. | Parcourez le compte Active Directory pour trouver Azure AD Connect et réinitialisez le mot de passe de sorte qu’il ne contienne pas plus de 256 caractères. Ensuite, ouvrez le **Service de synchronisation** dans le menu **Démarrer**. Accédez à **Connecteurs** et recherchez le **Connecteur Active Directory**. Sélectionnez-le, puis sélectionnez **Propriétés**. Accédez à la page **Informations d’identification** et entrez le nouveau mot de passe. Cliquez sur **OK** pour fermer la page. |
| À la dernière étape du processus d’installation d’Azure AD Connect, une erreur indique que la réécriture du mot de passe n’a pas pu être configurée. <br> <br> Le journal des événements de l’application Azure AD Connect contient l’erreur 32009 avec le texte « Erreur lors de l’obtention du jeton d’authentification ». | Cette erreur se produit dans les deux cas suivants : <br><ul><li>Vous avez spécifié un mot de passe incorrect pour le compte d’administrateur général spécifié au début du processus d’installation d’Azure AD Connect.</li><li>Vous avez tenté d’utiliser un utilisateur fédéré pour le compte d’administrateur général spécifié au début du processus d’installation d’Azure AD Connect.</li></ul> Pour corriger ce problème, assurez-vous que vous n’utilisez pas un compte fédéré pour l’administrateur général que vous avez spécifié au début du processus d’installation et que le mot de passe spécifié est correct. |
| Le journal des événements de la machine Azure AD Connect contient l’erreur 32002 qui est levée en exécutant PasswordResetService. <br> <br> Le message d’erreur : « Erreur de connexion à ServiceBus. Le fournisseur de jetons n’a pas pu fournir de jeton de sécurité. » | Votre environnement local n’est pas en mesure de se connecter au point de terminaison Service Bus dans le cloud. Cette erreur est due à une règle de pare-feu qui bloque une connexion sortante vers une adresse web ou un port spécifique. Consultez [Prérequis de connectivité](../hybrid/how-to-connect-install-prerequisites.md) pour plus d’informations. Une fois que vous avez mis à jour ces règles, redémarrez le serveur Azure AD Connect. La réécriture du mot de passe doit fonctionner de nouveau. |
| Après un certain temps, les utilisateurs fédérés ou qui ont recours à la synchronisation du hachage de mot de passe ou à l’authentification directe ne parviennent pas à réinitialiser leurs mots de passe. | Dans certains cas rares, le service de réécriture du mot de passe peut ne pas réussir à redémarrer quand Azure AD Connect a redémarré. Dans ces cas, commencez par vérifier si la réécriture du mot de passe est activée localement. Vous pouvez effectuer cette vérification à l’aide de l’Assistant Azure AD Connect ou de PowerShell. Si la fonctionnalité semble activée, essayez de l’activer ou de la désactiver à nouveau. Si cette étape de résolution des problèmes ne fonctionne pas, essayez une désinstallation et une réinstallation complètes d’Azure AD Connect. |
| Les utilisateurs fédérés ou qui ont recours à la synchronisation du hachage de mot de passe ou à l’authentification directe qui tentent de réinitialiser leurs mots de passe voient une erreur quand ils essaient de soumettre ces derniers. L’erreur indique un problème de service. <br ><br> En plus de ce problème, pendant les opérations de réinitialisation de mot de passe, une erreur peut figurer dans vos journaux des événements locaux, indiquant que l’agent de gestion s’est vu refuser l’accès. | Si vous voyez ces erreurs dans votre journal des événements, vérifiez que le compte de l’agent de gestion Active Directory (ADMA) qui a été spécifié dans l’Assistant au moment de la configuration dispose des autorisations nécessaires pour la réécriture du mot de passe. <br> <br> Une fois cette autorisation accordée, cela peut prendre jusqu’à 1 heure pour que les autorisations arrivent via la tâche en arrière-plan `sdprop` sur le contrôleur de domaine. <br> <br> Pour que la réinitialisation du mot de passe fonctionne, l’autorisation doit être marquée sur le descripteur de sécurité de l’objet utilisateur dont le mot de passe est réinitialisé. Tant que cette autorisation n’apparaît pas sur l’objet utilisateur, la réinitialisation du mot de passe continue d’échouer avec un message indiquant que l’accès a été refusé. |
| Les utilisateurs fédérés ou qui ont recours à la synchronisation du hachage de mot de passe ou à l’authentification directe qui tentent de réinitialiser leurs mots de passe voient une erreur lorsqu’ils soumettent ces derniers. L’erreur indique un problème de service. <br> <br> En plus de ce problème, durant les opérations de réinitialisation de mot de passe, une erreur « Objet introuvable »peut apparaître dans vos journaux des événements du service Azure AD Connect. | Cette erreur indique généralement que le moteur de synchronisation est incapable de trouver l’objet utilisateur dans l’espace du connecteur Azure AD ou l’objet espace du connecteur métaverse (MV) ou Azure AD lié. <br> <br> Pour résoudre ce problème, assurez-vous que l’utilisateur est véritablement synchronisé localement avec Azure AD via l’instance actuelle d’Azure AD Connect et examinez l’état des objets dans les espaces du connecteur et MV. Vérifiez que l’objet AD CS (Active Directory Certificate Services) est connecté à l’objet MV via la règle « Microsoft.InfromADUserAccountEnabled.xxx ».|
| Les utilisateurs fédérés ou qui ont recours à la synchronisation du hachage de mot de passe ou à l’authentification directe qui tentent de réinitialiser leurs mots de passe voient une erreur lorsqu’ils soumettent ces derniers. L’erreur indique un problème de service. <br> <br> En plus de ce problème, durant les opérations de réinitialisation de mot de passe, une erreur indiquant « Plusieurs correspondances trouvées » peut apparaître dans vos journaux des événements du service Azure AD Connect. | Cette erreur indique que le moteur de synchronisation a détecté que l’objet MV est connecté à plusieurs objets AD CS via « Microsoft.InfromADUserAccountEnabled.xxx ». Cela signifie que l’utilisateur dispose d’un compte activé dans plusieurs forêts. Ce scénario n’est pas pris en charge pour la réécriture du mot de passe. |
| Les opérations nécessitant un mot de passe échouent en renvoyant un message d’erreur de configuration. Le journal des événements d’application contient le message d’erreur Azure AD Connect 6329 dont le texte est libellé comme suit : « 0x8023061f (L’opération a échoué, car la synchronisation de mot de passe n’est pas activée pour cet agent de gestion) ». | Cette erreur se produit si la configuration d’Azure AD Connect est changée pour ajouter une nouvelle forêt Active Directory (ou pour supprimer et rajouter une forêt existante) après l’activation de la fonctionnalité de réécriture du mot de passe. Les opérations nécessitant un mot de passe pour les utilisateurs dans ces forêts récemment ajoutées échouent. Pour résoudre le problème, désactivez, puis réactivez la fonctionnalité de réécriture du mot de passe une fois les modifications de configuration de la forêt effectuées. |

## <a name="password-writeback-event-log-error-codes"></a>Codes d’erreur du journal des événements Écriture différée de mot de passe

Pour résoudre les problèmes liés à la réécriture du mot de passe, nous vous recommandons d’examiner le journal des événements de l’application sur votre machine Azure AD Connect. Ce journal des événements contient des événements issus de deux sources pour la réécriture du mot de passe. La source *PasswordResetService* décrit les opérations et les problèmes liés au fonctionnement de la réécriture du mot de passe. La source *ADSync* décrit les opérations et les problèmes liés à la définition des mots de passe dans votre environnement Active Directory Domain Services.

### <a name="if-the-source-of-the-event-is-adsync"></a>Si la source de l’événement est ADSync

| Code | Nom ou message | Description |
| --- | --- | --- |
| 6329 | BAIL : MMS(4924) 0x80230619 : « Une restriction empêche le mot de passe d’être remplacé par le mot de passe actuel spécifié. » | Cet événement se produit quand le service de réécriture du mot de passe tente de définir un mot de passe sur votre annuaire local qui ne respecte pas les critères d’ancienneté, d’historique, de complexité ou de filtrage du domaine. <br> <br> S’il existe une ancienneté minimale pour un mot de passe alors que vous l’avez changé avant le terme de cette ancienneté, vous ne pouvez pas le remodifier tant qu’il n’a pas atteint l’ancienneté spécifiée dans votre domaine. À des fins de test, l’ancienneté minimale doit être définie sur 0. <br> <br> Si des critères d’historique de mot de passe sont activés, vous devez sélectionner un mot de passe qui n’a pas été utilisé au cours des *N* dernières fois, où *N* est le paramètre d’historique du mot de passe. Si vous sélectionnez un mot de passe qui a été utilisé au cours des *N* dernières fois, un échec se produit. À des fins de test, l’historique de mot de passe doit être défini sur 0. <br> <br> S’il existe des critères de complexité des mots de passe, ils sont tous appliqués quand l’utilisateur tente de modifier ou réinitialiser un mot de passe. <br> <br> Si des filtres de mots de passe sont activés et qu’un utilisateur sélectionne un mot de passe qui ne répond pas aux critères de filtrage, l’opération de réinitialisation ou de modification échoue. |
| 6329 | MMS(3040) : admaexport.cpp(2837) : le serveur ne contient pas le contrôle de stratégie de mot de passe LDAP. | Ce problème se produit si le contrôle LDAP_SERVER_POLICY_HINTS_OID (1.2.840.113556.1.4.2066) n’est pas activé sur les contrôleurs de domaine. Pour utiliser la fonctionnalité d’écriture différée du mot de passe, vous devez activer le contrôle. Pour cela, les contrôleurs de domaine doivent se trouver sur Windows Server 2008 R2 ou version ultérieure. |
| HR 8023042 | Le moteur de synchronisation a renvoyé une erreur hr=80230402, message = Une tentative visant à obtenir un objet a échoué, car il existe des entrées en double avec le même point d’ancrage. | Cette erreur se produit quand le même identifiant utilisateur est activé dans plusieurs domaines. C’est, par exemple, le cas si vous synchronisez des forêts de comptes et de ressources et que le même identifiant utilisateur est présent et activé pour chacune des forêts. <br> <br> Cette erreur peut également se produire si vous utilisez un attribut d’ancrage non unique (comme un alias ou un UPN) et que deux utilisateurs partagent ce même attribut d’ancrage. <br> <br> Pour résoudre ce problème, assurez-vous de ne pas avoir d’utilisateurs en double dans vos domaines et d’utiliser un attribut d’ancrage unique pour chaque utilisateur. |

### <a name="if-the-source-of-the-event-is-passwordresetservice"></a>Si la source de l’événement est PasswordResetService

| Code | Nom ou message | Description |
| --- | --- | --- |
| 31001 | PasswordResetStart | Cet événement indique que le service local a détecté une demande de réinitialisation de mot de passe provenant du cloud pour un utilisateur fédéré ou qui a recours à la synchronisation du hachage de mot de passe ou à l’authentification directe. Cet événement est le premier de chaque opération d’écriture différée de réinitialisation de mot de passe. |
| 31002 | PasswordResetSuccess | Cet événement indique qu’un utilisateur a sélectionné un nouveau mot de passe pendant une opération de réinitialisation de mot de passe. Nous avons déterminé que ce mot de passe répond aux exigences de mot de passe d’entreprise. Le mot de passe a été correctement écrit en différé dans l’environnement Active Directory local. |
| 31003 | PasswordResetFail | Cet événement indique qu’un utilisateur a sélectionné un mot de passe et que le mot de passe est arrivé avec succès à l’environnement local. Toutefois, quand nous avons essayé de définir le mot de passe dans l’environnement Active Directory local, une erreur s’est produite. Cet échec peut se produire pour plusieurs raisons : <br><ul><li>Le mot de passe de l’utilisateur ne répond pas aux critères d’ancienneté, d’historique, de complexité ou de filtrage du domaine. Pour résoudre ce problème, créez un mot de passe.</li><li>Le compte de service ADMA n’a pas les autorisations appropriées pour définir le nouveau mot de passe du compte d’utilisateur en question.</li><li>Le compte d’utilisateur est dans un groupe protégé, comme celui des administrateurs de domaine ou d’entreprise, ce qui n’autorise pas les opérations de définition de mots de passe.</li></ul>|
| 31004 | OnboardingEventStart | Cet événement se produit si vous activez la réécriture du mot de passe avec Azure AD Connect et que nous avons commencé à intégrer votre organisation au service web de réécriture du mot de passe. |
| 31005 | OnboardingEventSuccess | Cet événement indique que le processus d’intégration a réussi et que la fonctionnalité de réécriture du mot de passe est prête à être utilisée. |
| 31006 | ChangePasswordStart | Cet événement indique que le service local a détecté une demande de modification de mot de passe provenant du cloud pour un utilisateur fédéré ou qui a recours à la synchronisation du hachage de mot de passe ou à l’authentification directe. Cet événement est le premier de chaque opération d’écriture différée de modification de mot de passe. |
| 31007 | ChangePasswordSuccess | Cet événement indique qu’un utilisateur a sélectionné un nouveau mot de passe pendant une opération de modification de mot de passe. Nous avons déterminé que le mot de passe répond aux critères définis pour les mots de passe de l’entreprise et que ce mot de passe a été correctement écrit dans l’environnement Active Directory local. |
| 31008 | ChangePasswordFail | Cet événement indique qu’un utilisateur a sélectionné un mot de passe et que le mot de passe est arrivé correctement dans l’environnement local, mais quand nous avons essayé de définir le mot de passe dans l’environnement Active Directory local, une défaillance s’est produite. Cet échec peut se produire pour plusieurs raisons : <br><ul><li>Le mot de passe de l’utilisateur ne répond pas aux critères d’ancienneté, d’historique, de complexité ou de filtrage du domaine. Pour résoudre ce problème, créez un mot de passe.</li><li>Le compte de service ADMA n’a pas les autorisations appropriées pour définir le nouveau mot de passe du compte d’utilisateur en question.</li><li>Le compte d’utilisateur est dans un groupe protégé, comme celui des administrateurs de domaine ou d’entreprise, qui n’autorisent pas les opérations de définition de mots de passe.</li></ul> |
| 31009 | ResetUserPasswordByAdminStart | Le service local a détecté une demande de réinitialisation de mot de passe pour un utilisateur fédéré ou qui a recours à la synchronisation du hachage de mot de passe ou à l’authentification directe provenant de l’administrateur au nom d’un utilisateur. Cet événement est le premier de chaque opération d’écriture différée de réinitialisation de mot de passe qui est lancée par un administrateur. |
| 31010 | ResetUserPasswordByAdminSuccess | L’administrateur a sélectionné un nouveau mot de passe durant une opération de réinitialisation de mot de passe lancée par l’administrateur. Nous avons déterminé que ce mot de passe répond aux exigences de mot de passe d’entreprise. Le mot de passe a été correctement écrit en différé dans l’environnement Active Directory local. |
| 31011 | ResetUserPasswordByAdminFail | L’administrateur a sélectionné un mot de passe au nom d’un utilisateur. Le mot de passe est arrivé avec succès à l’environnement local. Toutefois, quand nous avons essayé de définir le mot de passe dans l’environnement Active Directory local, une erreur s’est produite. Cet échec peut se produire pour plusieurs raisons : <br><ul><li>Le mot de passe de l’utilisateur ne répond pas aux critères d’ancienneté, d’historique, de complexité ou de filtrage du domaine. Pour résoudre ce problème, essayez un nouveau mot de passe.</li><li>Le compte de service ADMA n’a pas les autorisations appropriées pour définir le nouveau mot de passe du compte d’utilisateur en question.</li><li>Le compte d’utilisateur est dans un groupe protégé, comme celui des administrateurs de domaine ou d’entreprise, qui n’autorisent pas les opérations de définition de mots de passe.</li></ul>  |
| 31012 | OffboardingEventStart | Cet événement se produit si vous désactivez la réécriture du mot de passe avec Azure AD Connect et indique que nous avons commencé à désintégrer votre organisation du service web de réécriture du mot de passe. |
| 31013| OffboardingEventSuccess| Cet événement indique que le processus de désintégration a réussi et que la fonctionnalité de réécriture du mot de passe a correctement été désactivée. |
| 31014| OffboardingEventFail| Cet événement indique que le processus de désintégration n’a pas réussi. Cela peut être dû à une erreur d’autorisations sur le compte d’administrateur cloud ou local spécifié durant la configuration. L’erreur peut également se produire si vous tentez d’utiliser un administrateur général cloud fédéré quand vous désactivez la réécriture du mot de passe. Pour résoudre ce problème, vérifiez vos autorisations administratives et vérifiez que vous n’utilisez pas un compte fédéré pour configurer la fonctionnalité de réécriture du mot de passe.|
| 31015| WriteBackServiceStarted| Cet événement indique que le service de réécriture du mot de passe a démarré avec succès. Il est prêt à accepter les demandes de gestion de mot de passe provenant du cloud.|
| 31016| WriteBackServiceStopped| Cet événement indique que le service de réécriture du mot de passe s’est arrêté. Toute demande de gestion de mot de passe provenant du cloud est vouée à l’échec.|
| 31017| AuthTokenSuccess| Cet événement indique que nous avons correctement récupéré un jeton d’autorisation pour l’administrateur général spécifié pendant l’installation d’Azure AD Connect afin de démarrer le processus d’intégration ou de désintégration.|
| 31018| KeyPairCreationSuccess| Cet événement indique que nous avons correctement créé la clé de chiffrement de mot de passe. Cette clé est utilisée pour chiffrer les mots de passe à partir du cloud à envoyer à votre environnement local.|
| 31034| ServiceBusListenerError| Cet événement indique une erreur de connexion à l’écouteur Service Bus de votre locataire. Si le message d’erreur indique que le certificat distant n’est pas valide, vérifiez que votre serveur Azure AD Connect dispose de toutes les autorités de certification racines requises, comme décrit dans [Changements des certificats Azure TLS](../../security/fundamentals/tls-certificate-changes.md). |
| 32000| UnknownError| Cet événement indique qu’une erreur inconnue s’est produite durant une opération de gestion de mot de passe. Examinez le texte de l’exception dans l’événement pour plus d’informations. Si vous rencontrez des problèmes, essayez de désactiver, puis de réactiver la réécriture du mot de passe. Si cela ne change rien, incluez une copie de votre journal des événements avec l’ID de suivi spécifié lorsque vous ouvrez une demande de support.|
| 32001| ServiceError| Cet événement indique une erreur de connexion au service de réinitialisation de mot de passe à partir du cloud. Cette erreur se produit généralement quand le service local n’a pas pu se connecter au service web de réinitialisation de mot de passe.|
| 32002| ServiceBusError| Cet événement indique une erreur de connexion à une instance Service Bus de votre locataire. Cela peut se produire si vous bloquez les connexions sortantes de votre environnement local. Vérifiez que votre pare-feu autorise les connexions sur TCP 443 et à https://ssprdedicatedsbprodncu.servicebus.windows.net, puis réessayez. Si vous rencontrez encore des problèmes, essayez de désactiver, puis de réactiver la réécriture du mot de passe.|
| 32003| InPutValidationError| Cet événement indique que l’entrée passée à l’API du service web n’est pas valide. Recommencez l’opération.|
| 32004| DecryptionError| Cet événement indique qu’une erreur de déchiffrement du mot de passe arrivé du cloud s’est produite. Cette erreur peut être due à une incompatibilité de la clé de déchiffrement entre le service cloud et votre environnement local. Pour résoudre ce problème, désactivez, puis réactivez la réécriture du mot de passe dans votre environnement local.|
| 32005| ConfigurationError| Au cours de l’intégration, nous enregistrons des informations propres au locataire dans un fichier de configuration dans votre environnement local. Cet événement indique qu’une erreur s’est produite durant l’enregistrement de ce fichier ou, quand le service a démarré, durant la lecture de ce fichier. Pour résoudre ce problème, essayez de désactiver, puis réactiver la réécriture du mot de passe pour forcer une réécriture du fichier de configuration.|
| 32007| OnBoardingConfigUpdateError| Pendant l’intégration, nous envoyons des données du cloud au service de réinitialisation du mot de passe local. Ces données sont ensuite écrites dans un fichier en mémoire avant d’être envoyées au service de synchronisation pour être stockées de manière sécurisée sur le disque. Cet événement indique un problème d’écriture ou de mise à jour de ces données en mémoire. Pour résoudre ce problème, essayez de désactiver, puis réactiver la réécriture du mot de passe pour forcer une réécriture de ce fichier de configuration.|
| 32008| ValidationError| Cet événement indique que nous avons reçu une réponse non valide du service web de réinitialisation du mot de passe. Pour résoudre ce problème, essayez de désactiver, puis réactiver la réécriture du mot de passe.|
| 32009| AuthTokenError| Cet événement indique que nous n’avons pas réussi à obtenir un jeton d’autorisation pour le compte d’administrateur général spécifié pendant l’installation d’Azure AD Connect. Cette erreur peut être due à un mauvais nom d’utilisateur ou mot de passe spécifié pour le compte d’administrateur général. Elle peut également se produire si le compte d’administrateur général spécifié est fédéré. Pour résoudre ce problème, réexécutez la configuration avec le nom d’utilisateur et le mot de passe corrects, puis assurez-vous que l’administrateur correspond à un compte managé (sur le cloud uniquement ou avec la synchronisation de mot de passe).|
| 32010| CryptoError| Cet événement indique une erreur liée à la génération de la clé de chiffrement du mot de passe ou au déchiffrement d’un mot de passe arrivé du service cloud. Cette erreur indique probablement un problème lié à votre environnement. Examinez les détails de votre journal des événements pour en savoir plus sur la résolution de ce problème. Vous pouvez également essayer de désactiver, puis réactiver le service de réécriture du mot de passe.|
| 32011| OnBoardingServiceError| Cet événement indique que le service local n’a pas pu communiquer correctement avec le service web de réinitialisation du mot de passe pour lancer le processus d’intégration. Cela peut être lié à une règle de pare-feu ou à un problème d’obtention d’un jeton d’authentification pour votre locataire. Pour résoudre ce problème, veillez à ne pas bloquer les connexions sortantes via TCP 443 et TCP 9350 à 9354 ou à https://ssprdedicatedsbprodncu.servicebus.windows.net. Vérifiez également que le compte d’administrateur Azure AD que vous utilisez pour l’intégration n’est pas fédéré.|
| 32013| OffBoardingError| Cet événement indique que le service local n’a pas pu communiquer correctement avec le service web de réinitialisation du mot de passe pour lancer le processus de désintégration. Cette erreur peut être due à une règle de pare-feu ou à un problème d’obtention d’un jeton d’autorisation pour votre locataire. Pour résoudre ce problème, veillez à ne pas bloquer les connexions sortantes via 443 ou à https://ssprdedicatedsbprodncu.servicebus.windows.net et que le compte d’administrateur Azure Active Directory que vous utilisez pour la désintégration n’est pas fédéré.|
| 32014| ServiceBusWarning| Cet événement indique que nous avons dû réessayer de vous connecter à une instance Service Bus de votre locataire. Dans des conditions normales, cet événement n’est pas préoccupant, mais s’il se produit à de nombreuses reprises, envisagez de vérifier la connexion réseau à Service Bus, surtout s’il s’agit d’une connexion à latence élevée ou à faible bande passante.|
| 32015| ReportServiceHealthError| Pour analyser l’intégrité de votre service de réécriture du mot de passe, nous envoyons des données de pulsation à notre service web de réinitialisation du mot de passe toutes les cinq minutes. Cet événement indique qu’une erreur s’est produite lors du renvoi de ces informations d’intégrité au service web cloud. Ces informations d’intégrité n’incluent pas de données personnelles. Il s’agit uniquement de statistiques de pulsations et de services de base qui nous permettent de fournir des informations sur l’état du service dans le cloud.|
| 33001| ADUnKnownError| Cet événement indique qu’une erreur inconnue a été retournée par Active Directory. Pour plus d’informations, recherchez les événements issus de la source ADSync dans le journal des événements du serveur Azure AD Connect.|
| 33002| ADUserNotFoundError| Cet événement indique que l’utilisateur qui essaie de réinitialiser ou modifier un mot de passe est introuvable dans l’annuaire local. Cette erreur peut se produire quand l’utilisateur a été supprimé localement, mais pas sur le cloud. Cette erreur peut également se produire s’il existe un problème de synchronisation. Vérifiez vos journaux d’activité de synchronisation, ainsi que les détails de la dernière synchronisation pour plus d’informations.|
| 33003| ADMutliMatchError| Quand une demande de réinitialisation ou modification de mot de passe vient du cloud, nous utilisons l’ancrage cloud spécifiée pendant le processus d’installation d’Azure AD Connect pour déterminer comment lier cette demande à un utilisateur dans votre environnement local. Cet événement indique que nous avons trouvé deux utilisateurs dans votre annuaire local avec le même attribut d’ancrage cloud. Vérifiez vos journaux d’activité de synchronisation, ainsi que les détails de la dernière synchronisation pour plus d’informations.|
| 33004| ADPermissionsError| Cet événement indique que le compte de service de l’agent de gestion Active Directory (ADMA) n’a pas les autorisations appropriées sur le compte en question pour définir un nouveau mot de passe. Assurez-vous que le compte ADMA dans la forêt de l’utilisateur possède des autorisations de réinitialisation de mot de passe sur tous les objets de la forêt. Pour obtenir plus d’informations sur la définition des autorisations, reportez-vous à l’étape 4 : Configuration des autorisations Active Directory adéquates. Cette erreur peut également se produire lorsque l’attribut AdminCount de l’utilisateur a la valeur 1.|
| 33005| ADUserAccountDisabled| Cet événement indique que nous avons tenté de réinitialiser ou changer un mot de passe pour un compte qui a été désactivé localement. Activez le compte et recommencez l’opération.|
| 33006| ADUserAccountLockedOut| Cet événement indique que nous avons tenté de réinitialiser ou changer un mot de passe pour un compte qui a été verrouillé localement. Des verrouillages peuvent se produire quand un utilisateur a tenté une opération de modification ou réinitialisation de mot de passe un nombre de fois trop élevé sur un court laps de temps. Déverrouillez le compte et recommencez l’opération.|
| 33007| ADUserIncorrectPassword| Cet événement indique que l’utilisateur a spécifié un mot de passe actuel incorrect lors de l’opération de modification du mot de passe. Spécifiez le mot de passe correct actuel et réessayez.|
| 33008| ADPasswordPolicyError| Cet événement se produit quand le service de réécriture du mot de passe tente de définir un mot de passe sur votre annuaire local qui ne respecte pas les critères d’ancienneté, d’historique, de complexité ou de filtrage du domaine. <br> <br> S’il existe une ancienneté minimale pour un mot de passe alors que vous l’avez changé avant le terme de cette ancienneté, vous ne pouvez pas le remodifier tant qu’il n’a pas atteint l’ancienneté spécifiée dans votre domaine. À des fins de test, l’ancienneté minimale doit être définie sur 0. <br> <br> Si des critères d’historique de mot de passe sont activés, vous devez sélectionner un mot de passe qui n’a pas été utilisé au cours des *N* dernières fois, où *N* est le paramètre d’historique du mot de passe. Si vous sélectionnez un mot de passe qui a été utilisé au cours des *N* dernières fois, un échec se produit. À des fins de test, l’historique de mot de passe doit être défini sur 0. <br> <br> S’il existe des critères de complexité des mots de passe, ils sont tous appliqués quand l’utilisateur tente de modifier ou réinitialiser un mot de passe. <br> <br> Si des filtres de mots de passe sont activés et qu’un utilisateur sélectionne un mot de passe qui ne répond pas aux critères de filtrage, l’opération de réinitialisation ou de modification échoue.|
| 33009| ADConfigurationError| Cet événement indique qu’un problème s’est produit durant l’écriture d’un mot de passe dans votre annuaire local en raison d’un problème de configuration avec Active Directory. Consultez le journal des événements de l’application de la machine Azure AD Connect pour identifier les messages provenant du service ADSync et obtenir plus d’informations sur l’erreur qui s’est produite.|

## <a name="azure-ad-forums"></a>Forums d’Azure AD

Si vous avez des questions générales sur Azure AD et la réinitialisation du mot de passe en libre-service, vous pouvez demander de l’aide à la communauté via la [Page des Q&R Microsoft pour Azure Active Directory](/answers/topics/azure-active-directory.html). Les membres de la communauté comprennent des ingénieurs, des chefs de produit, MVP et autres professionnels de l’informatique.

## <a name="contact-microsoft-support"></a>Contact Microsoft support

Si vous ne trouvez pas la réponse à un problème, nos équipes de support technique sont toujours disponibles pour vous aider davantage.

Pour que nous puissions mieux vous aider, nous vous demandons de fournir autant de détails que possible au moment de l’ouverture d’un dossier d’incident. Ces détails comprennent ce qui suit :

* **Description générale de l’erreur**. Quelle est l’erreur ? Quel était le comportement que vous avez remarqué ? Comment pouvons-nous reproduire l’erreur ? Fournissez autant de détails que possible.
* **Page**. Sur quelle page étiez-vous quand vous avez remarqué l’erreur ? Indiquez l’URL si possible, ainsi qu’une capture d’écran de la page.
* **Code de support**. Quel était le code de support généré quand l’utilisateur a vu l’erreur ?
   * Pour trouver ce code, reproduisez l’erreur, puis sélectionnez le lien **Code de support** en bas de l’écran et envoyez à l’ingénieur du support technique le GUID obtenu.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-support-code.png" alt-text="Le code de support se trouve en bas à droite de la fenêtre du navigateur web.":::

  * Si vous êtes dans une page sans code de support dans la partie inférieure, appuyez sur F12 et recherchez le SID et le CID et envoyez ces deux résultats à l’ingénieur de support.
* **Date, heure et fuseau horaire**. Incluez la date et l’heure précises, *avec le fuseau horaire*, d’occurrence de l’erreur.
* **ID d’utilisateur**. Quel était l’utilisateur qui a vu l’erreur ? *user\@contoso.com* en est un exemple.
   * S’agit-il d’un utilisateur fédéré ?
   * S’agit-il d’un utilisateur de l’authentification directe ?
   * S’agit-il d’un utilisateur disposant de la synchronisation du hachage de mot de passe ?
   * S’agit-il d’un utilisateur cloud uniquement ?
* **Licences**. L’utilisateur a-t-il une licence Azure AD ?
* **Journal des événements de l’application**. Si vous utilisez la réécriture de mot de passe et que l’erreur se produit dans votre infrastructure locale, veuillez inclure une copie compressée du journal des événements de l’application provenant de votre serveur Azure AD Connect.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur SSPR, consultez [Fonctionnement : Réinitialisation de mot de passe en libre-service Azure AD](concept-sspr-howitworks.md) ou [Comment fonctionne la réécriture de la réinitialisation de mot de passe en libre-service dans Azure AD ?](concept-sspr-writeback.md).
