---
title: 'Azure AD Connect : Historique de publication des versions | Microsoft Docs'
description: Cet article répertorie toutes les versions d’Azure AD Connect et d’Azure AD Sync.
services: active-directory
author: billmath
manager: daveba
ms.assetid: ef2797d7-d440-4a9a-a648-db32ad137494
ms.service: active-directory
ms.topic: reference
ms.workload: identity
ms.date: 08/07/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 73318d1ee14894f5d22f7c4d2e61418e3b1038c1
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636875"
---
# <a name="azure-ad-connect-version-release-history"></a>Azure AD Connect : Historique de publication des versions
L’équipe Azure Active Directory (Azure AD) met régulièrement à jour Azure AD Connect avec de nouvelles fonctions et fonctionnalités. Tous les ajouts ne sont pas applicables à toutes les configurations.

Cet article est conçu pour vous aider à conserver la trace des versions publiées, et à comprendre les modifications apportées par la dernière version.



Ce tableau présente la liste des rubriques connexes :

Rubrique |  Détails
--------- | --------- |
Étapes de mise à niveau à partir d’Azure AD Connect | Différentes méthodes pour [effectuer une mise à niveau à partir d’une version précédente vers la dernière](how-to-upgrade-previous-version.md) version Azure AD Connect.
Autorisations requises | Pour plus d’informations sur les autorisations requises afin d’appliquer une mise à jour, consultez [Comptes et autorisations](reference-connect-accounts-permissions.md#upgrade).
Téléchargement| [Téléchargez Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771).

>[!NOTE]
>La publication d’une nouvelle version d’Azure AD Connect est un processus qui nécessite plusieurs étapes de contrôle qualité pour garantir la fonctionnalité de l’opération du service. Pendant que nous effectuons ce processus, le numéro d’une nouvelle version ainsi que l’état de la version sont mis à jour pour refléter l’état le plus récent.
Pendant que nous effectuons ce processus, le numéro de la version s’affiche avec un « X » en position de numéro de version mineure, comme dans « 1.3.X.0 ». Cela indique que les notes de publication de ce document sont valides pour toutes les versions commençant par « 1.3. ». Dès que nous aurons finalisé le processus de publication, le numéro de version sera mis à jour vers la version la plus récente, et l’état de lancement sera mis à jour vers « Lancé pour le téléchargement et la mise à niveau automatique ».
La mise à niveau automatique ne concernera pas toutes les versions d’Azure AD Connect. L’état de la version indique si une version est disponible en mise à niveau automatique ou en téléchargement uniquement. Si la mise à niveau automatique a été activée sur votre serveur Azure AD Connect, celui-ci sera automatiquement mis à niveau vers la dernière version d’Azure AD Connect qui est disponible en mise à niveau automatique. Certaines configurations Azure AD Connect seulement sont éligibles à la mise à niveau automatique. 

Pour clarifier l’utilisation de la mise à niveau automatique, il est prévu de vous transmettre toutes les mises à jour importantes et tous les correctifs critiques. Il ne s’agit pas nécessairement de la dernière version, car toutes les versions ne nécessitent pas/incluent un correctif d’un problème de sécurité critique (juste un exemple parmi tant d’autres). Un problème semblable à celui-ci serait résolu avec une nouvelle version fournie via la Mise à niveau automatique. S’il n’y a pas de problèmes de ce type, aucune mise à jour n’est effectuée à l’aide de la Mise à niveau automatique et, en général, vous devriez être satisfait si vous utilisez la dernière version de la Mise à niveau automatique.
Toutefois, si vous souhaitez bénéficier des dernières fonctionnalités et mises à jour, le meilleur moyen de savoir si elles existent est de consulter cette page et de les installer comme bon vous semble. 

Veuillez suivre ce lien pour en savoir plus sur la [mise à niveau automatique](how-to-connect-install-automatic-upgrade.md)

>[!IMPORTANT]
> À partir du 1er novembre 2020, nous entamerons un processus de dépréciation dans le cadre duquel les versions d'Azure AD Connect publiées depuis plus de 18 mois seront dépréciées. À ce moment-là, nous commencerons à déprécier Azure AD Connect 1.3.20.0 (publié le 24/04/2019) et ses versions antérieures, et nous procéderons à l’évaluation de la dépréciation des anciennes versions d’Azure AD Connect chaque fois qu’une nouvelle version sera publiée.
>
> Vous devez vous assurer que vous exécutez une version récente d'Azure AD Connect pour bénéficier d'une expérience de support optimale. 
>
>Si vous exécutez une version obsolète d'Azure AD Connect, vous risquez de ne pas bénéficier des correctifs de sécurité, améliorations des performances, outils de diagnostic et de résolution des problèmes et améliorations des services les plus récents, et si vous avez besoin d'assistance, nous ne serons peut-être pas en mesure de vous fournir le niveau de service dont votre organisation a besoin.
>
>Si vous avez activé Azure AD Connect pour la synchronisation, vous commencerez bientôt à recevoir automatiquement des notifications d'intégrité qui vous avertiront des dépréciations à venir lorsque vous exécuterez une version ancienne.
>
>Veuillez consulter [cet article](./how-to-upgrade-previous-version.md) pour en savoir plus sur la mise à niveau d'Azure AD Connect vers la version la plus récente.
>
>Pour obtenir des informations sur l’historique des versions déconseillées, consultez [Archive de l’historique des versions Azure AD Connect](reference-connect-version-history-archive.md).

## <a name="15450"></a>1.5.45.0

### <a name="release-status"></a>État de la version
29/07/2020 : publiée pour téléchargement

### <a name="functional-changes"></a>Modifications fonctionnelles
Il s’agit d’une version du correctif de bogue. Elle ne fait l’objet d’aucune modification fonctionnelle.

### <a name="fixed-issues"></a>Problèmes résolus

- Résolution d’un problème empêchant l’administrateur d’activer l’« authentification unique transparente » quand le compte d’ordinateur AZUREADSSOACC est déjà présent dans « Active Directory ».
- Résolution d’un problème qui provoquait une erreur de mise en lots pendant l’importation différentielle de l’API V2 quand un objet en conflit était réparé dans le portail Health.
- Résolution d’un problème de configuration d’importation/exportation provoquant l’importation d’une règle personnalisée désactivée comme activée.

## <a name="15420"></a>1.5.42.0

### <a name="release-status"></a>État de la version
10/07/2020 : publiée pour téléchargement

### <a name="functional-changes"></a>Modifications fonctionnelles
Cette version comprend une préversion publique de la fonctionnalité permettant d’exporter la configuration d’un serveur de Azure AD Connect existant dans un fichier .JSON qui peut ensuite être utilisé lors de l’installation d’un nouveau serveur de Azure AD Connect pour créer une copie du serveur d’origine.

Une description détaillée de cette nouvelle fonctionnalité est disponible dans [cet article](./how-to-connect-import-export-config.md)

### <a name="fixed-issues"></a>Problèmes résolus
- Correction d’un bogue où de faux avertissement pendant la mise à niveau concernant la taille de la base de données locale sur les builds localisés.
- Correction d’un bogue où il y aurait une fausse erreur dans les événements de l’application pour l’échange de nom de compte/nom de domaine.
- Correction d’une erreur qui entraînait l’échec de l’installation de Azure AD Connect sur un contrôleur de domaine, avec l’erreur « membre introuvable ».


## <a name="15300"></a>1.5.30.0

### <a name="release-status"></a>État de la version
07/05/2020 : publiée pour téléchargement

### <a name="fixed-issues"></a>Problèmes résolus
Ce correctif corrige un problème où des domaines non sélectionnés étaient incorrectement sélectionnés à partir de l’interface utilisateur de l’Assistant si seuls des conteneurs petits-enfants étaient sélectionnés.


>[!NOTE]
>Cette version comprend la nouvelle API de point de terminaison v2 pour la synchronisation d’Azure AD Connect.  Ce nouveau point de terminaison v2 est actuellement en préversion publique.  Cette version ou une version ultérieure est requise pour utiliser la nouvelle API de point de terminaison v2.  Toutefois, l’installation seule de cette version n’active pas le point de terminaison v2. Vous continuerez d’utiliser le point de terminaison v1, sauf si vous activez le point de terminaison v2.  Vous devez suivre les étapes décrites dans [API de point de terminaison v2 pour la synchronisation d’Azure AD Connect (préversion publique)](how-to-connect-sync-endpoint-api-v2.md) pour l’activer et vous abonner à la préversion publique.  

## <a name="15290"></a>1.5.29.0

### <a name="release-status"></a>État de la version
23/04/2020 : publiée pour téléchargement

### <a name="fixed-issues"></a>Problèmes résolus
Ce correctif corrige un problème introduit dans la version 1.5.20.0, dans lequel un administrateur de locataire avec MFA ne pouvait pas activer DSSO.

## <a name="15220"></a>1.5.22.0

### <a name="release-status"></a>État de la version
20/04/2020 : publiée pour téléchargement

### <a name="fixed-issues"></a>Problèmes résolus
Ce correctif logiciel résout un problème dans la build 1.5.20.0 si vous avez cloné la règle **In from AD - Group Join** et que vous n’avez pas cloné la règle **In from AD - Group Common**.

## <a name="15200"></a>1.5.20.0

### <a name="release-status"></a>État de la version
04/09/2020 : publiée pour téléchargement

### <a name="fixed-issues"></a>Problèmes résolus
- Cette build de correctif logiciel résout un problème lié au build 1.5.18.0 si la fonctionnalité de filtrage de groupe est activée et utilise mS-DS-ConsistencyGuid comme point d’ancrage source.
- Correction d’un problème dans le module PowerShell ADSyncConfig, où l’appel de la commande DSACLS utilisée dans toutes les cmdlets Set-ADSync* provoquait l’une des erreurs suivantes :
     - `GrantAclsNoInheritance : The parameter is incorrect.   The command failed to complete successfully.`
     - `GrantAcls : No GUID Found for computer …`

> [!IMPORTANT]
> Si vous avez cloné la règle de synchronisation **In from AD - Group Join**, que vous n’avez pas cloné la règle de synchronisation **In from AD - Group Common** et que vous envisagez une mise à niveau, effectuez les étapes suivantes dans le cadre de la mise à niveau :
> 1. Pendant la mise à niveau, désactivez l’option **Démarrez le processus de synchronisation une fois la configuration terminée**.
> 2. Modifiez la règle de synchronisation de jointure clonée et ajoutez les deux transformations suivantes :
>     - Définissez le flux direct `objectGUID` sur `sourceAnchorBinary`.
>     - Définissez le flux d’expression `ConvertToBase64([objectGUID])` sur `sourceAnchor`.     
> 3. Activez le planificateur à l’aide de `Set-ADSyncScheduler -SyncCycleEnabled $true`.



## <a name="15180"></a>1.5.18.0

### <a name="release-status"></a>État de la version
02/04/2020 : publiée pour téléchargement

### <a name="functional-changes-adsyncautoupgrade"></a>Modifications fonctionnelles ADSyncAutoUpgrade 

- Ajout de la prise en charge de la fonctionnalité mS-DS-ConsistencyGuid pour les objets groupe. Cela vous permet de déplacer des groupes entre les forêts et de reconnecter des groupes dans AD à Azure AD lorsque le paramètre objectID du groupe AD a changé, par exemple lorsqu’un serveur AD est reconstruit après un désastre. Pour plus d’informations, consultez [Déplacement de groupes entre des forêts](how-to-connect-migrate-groups.md).
- L’attribut mS-DS-ConsistencyGuid est défini automatiquement sur tous les groupes synchronisés et vous n’avez rien à faire pour activer cette fonctionnalité. 
- Suppression de l’applet de commande Get-ADSyncRunProfile, car elle n’est plus utilisée. 
- Modification de l’avertissement qui s’affiche lorsque vous tentez d’utiliser un compte d’administrateur d’entreprise ou d’administrateur de domaine pour le compte de connecteur AD DS pour fournir davantage de contexte. 
- Ajout d’une nouvelle applet de commande pour supprimer des objets de l’espace de connecteur. L’ancien outil CSDelete.exe est supprimé et il est remplacé par la nouvelle applet de commande Remove-ADSyncCSObject. L’applet de commande Remove-ADSyncCSObject prend un objet CsObject en entrée. Cet objet peut être récupéré à l’aide de l’applet de commande Get-ADSyncCSObject.

>[!NOTE]
>L’ancien outil CSDelete.exe a été supprimé et remplacé par la nouvelle applet de commande Remove-ADSyncCSObject. 

### <a name="fixed-issues"></a>Problèmes résolus

- Correction d’un bogue dans le sélecteur de forêt/UO d’écriture différée de groupe lors de la réexécution de l’Assistant Azure AD Connect après la désactivation de la fonctionnalité. 
- Introduction d’une nouvelle page d’erreur qui s’affiche si les valeurs de Registre DCOM requises sont manquantes avec un nouveau lien d’aide. Les informations sont également écrites dans des fichiers journaux. 
- Résolution d’un problème lié à la création du compte de synchronisation Azure Active Directory, à cause duquel l’activation des extensions d’annuaire ou de PHS peut échouer parce que le compte n’a pas été propagé sur tous les réplicas de service avant la tentative d’utilisation. 
- Correction d’un bogue dans l’utilitaire de compression des erreurs de synchronisation qui ne gérait pas correctement les caractères de substitution. 
- Correction d’un bogue dans la mise à niveau automatique qui a laissé le serveur dans l’état suspendu du planificateur. 

## <a name="14380"></a>1.4.38.0
### <a name="release-status"></a>État de la version
9/12/2019 : Publication pour le téléchargement. Non disponible par le biais de la mise à niveau automatique.
### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités
- Nous avons mis à jour la synchronisation du hachage de mot de passe pour Azure AD Domain Services afin de prendre en compte le remplissage dans les hachages Kerberos.  Cela permet d’améliorer les performances pendant la synchronisation du mot de passe d’Azure AD avec Azure AD Domain Services.
- Nous avons ajouté la prise en charge de sessions fiables entre l’agent d’authentification et Service Bus.
- Cette version applique TLS 1.2 pour la communication entre l’agent d’authentification et les services cloud.
- Nous avons ajouté un cache DNS pour les connexions WebSocket entre l’agent d’authentification et les services cloud.
- Nous avons ajouté la possibilité de cibler un agent spécifique à partir du cloud pour tester la connectivité de l’agent.

### <a name="fixed-issues"></a>Problèmes résolus
- La version 1.4.18.0 comportait un bogue dans lequel la cmdlet PowerShell concernant DSSO utilisait les informations d'identification Windows de connexion au lieu des informations d'identification d'administrateur fournies lors de l'exécution de PowerShell. Par conséquent, il n’était pas possible d’activer DSSO dans plusieurs forêts par le biais de l’interface utilisateur Azure AD Connect. 
- Correctif permettant d’activer DSSO simultanément dans toutes les forêts par le biais de l’interface utilisateur Azure AD Connect

## <a name="14320"></a>1.4.32.0
### <a name="release-status"></a>État de la version
08/11/2019 : publiée pour téléchargement. Non disponible par le biais de la mise à niveau automatique.

>[!IMPORTANT]
>En raison d’une modification de schéma interne dans cette version d’Azure AD Connect, si vous gérez les paramètres de configuration d’une relation d’approbation AD FS à l’aide de MSOnline PowerShell, vous devez mettre à jour votre module MSOnline PowerShell vers la version 1.1.183.57 ou ultérieure

### <a name="fixed-issues"></a>Problèmes résolus

Cette version résout un problème avec les appareils Azure AD Hybride joints existants. Cette version contient une nouvelle règle de synchronisation d’appareil qui corrige ce problème.
Notez que cette modification de règle peut entraîner la suppression d’appareils obsolètes d’Azure AD. Cela n’est pas préoccupant car Azure AD n’utilise pas ces objets d’appareil durant l’autorisation d’accès conditionnel. Pour certains clients, le nombre d’appareils qui seront supprimés par le biais de cette modification de règle peut dépasser le seuil de suppression. Si vous constatez que les suppressions d’objets d’appareil dans Azure AD dépassent le seuil de suppression des exportations, il est conseillé d’autoriser la poursuite de ces suppressions. [Guide pratique pour autoriser le déroulement des suppressions quand elles dépassent le seuil de suppression](how-to-connect-sync-feature-prevent-accidental-deletes.md)

## <a name="14250"></a>1.4.25.0

### <a name="release-status"></a>État de la version
28/09/2019 : Mise en production pour la mise à niveau automatique afin de sélectionner des locataires. Non disponible pour téléchargement.

Cette version corrige un bogue qui avait pour effet que certains serveurs étaient mis à niveau automatiquement vers la version 1.4.18.0 et rencontraient des problèmes en lien avec la réinitialisation de mot de passe en libre-service (SSPR) et la réécriture du mot de passe.

### <a name="fixed-issues"></a>Problèmes résolus

Dans certaines circonstances, les serveurs mis à niveau automatiquement vers la version 1.4.18.0 ne réactivaient pas la réinitialisation de mot de passe en libre-service et la réécriture du mot de passe une fois la mise à niveau terminée. Cette mise en production de la mise à niveau automatique corrige ce problème et réactive la réinitialisation de mot de passe en libre-service et la réécriture du mot de passe.

Nous avons résolu un bogue dans l’utilitaire de compression des erreurs de synchronisation qui ne gérait pas correctement les caractères de substitution.

## <a name="14180"></a>1.4.18.0

>[!WARNING]
>Nous sommes en train de faire des recherches sur un incident rencontrés par certains clients avec des appareils joints à Azure AD Hybride existants après avoir effectué la mise à niveau vers cette version d’Azure AD Connect. Nous conseillons aux clients qui ont déployé Azure AD Hybride de différer la mise à niveau vers cette version jusqu’à ce que la cause racine de ce problème soit entièrement comprise et atténuée. Des informations supplémentaires seront fournies dès que possible.

>[!IMPORTANT]
>Avec cette version d’Azure AD Connect, certains clients peuvent voir une partie ou l’ensemble de leurs appareils Windows disparaître d’Azure AD. Cela n’est pas préoccupant car Azure AD n’utilise pas ces identités d’appareil durant l’autorisation d’accès conditionnel. Pour plus d’informations, voir [Comprendre la disparition des appareils Azure AD Connect 1.4.xx.x](reference-connect-device-disappearance.md)


### <a name="release-status"></a>État de la version
25/9/2019 : Publiée pour la mise à niveau automatique uniquement.

### <a name="new-features-and-improvements"></a>Améliorations et nouvelles fonctionnalités
- De nouveaux outils de résolution des problèmes permettent de corriger les scénarios « utilisateur non synchronisé », « groupe non synchronisé » ou « membre de groupe non synchronisé ».
- Ajout d’une prise en charge des clouds nationaux dans le script de résolution des problèmes d’Azure AD Connect.
- Les clients doivent être informés que les points de terminaison WMI dépréciés pour MIIS_Service sont désormais supprimés. Toutes les opérations WMI doivent à présent être effectuées par le biais des applets de commande PS.
- Amélioration de la sécurité en réinitialisant la délégation contrainte sur l’objet AZUREADSSOACC.
- Lors de l’ajout ou de la modification d’une règle de synchronisation, si des attributs sont utilisés dans la règle et qu’ils se trouvent dans le schéma du connecteur, mais ne sont pas ajoutés au connecteur, ces attributs sont automatiquement ajoutés au connecteur. Il en va de même pour le type d’objet affecté par la règle. Si quelque chose est ajouté au connecteur, le connecteur sera marqué pour une importation complète au cours du prochain cycle de synchronisation.
- L’utilisation d’un administrateur d’entreprise ou de domaine en tant que compte de connecteur n’est plus prise en charge dans les nouveaux déploiements d’Azure AD Connect. Cette version n’aura pas d’impact sur les déploiements en cours d’Azure AD Connect à l’aide d’un administrateur d’entreprise ou de domaine en tant que compte de connecteur.
- Dans le gestionnaire de synchronisation, une synchronisation complète est exécutée lors de la création, de la modification ou de la suppression d’une règle. Une fenêtre contextuelle s’affiche pour toute modification de règle, notifiant l’utilisateur que l’importation complète ou la synchronisation totale va être exécutée.
- Ajout d’étapes d’atténuation pour les erreurs de mot de passe à la page Connecteurs > Propriétés > Connectivité.
- Ajout, dans la page des propriétés du connecteur, d’un avertissement de dépréciation du gestionnaire du service de synchronisation. Cet avertissement prévient l’utilisateur que des modifications doivent être apportées via l’Assistant Azure AD Connect.
- Ajout d’une nouvelle erreur pour les problèmes liés à la stratégie de mot de passe d’un utilisateur.
- Prévention d’une erreur de configuration du filtrage de groupe par les filtres de domaine et d’UO. Le filtrage de groupe affiche une erreur lorsque le domaine ou l’unité d’organisation du groupe entré est déjà filtré et empêche l’utilisateur de poursuivre tant que le problème n’est pas résolu.
- Les utilisateurs ne peuvent plus créer de connecteur pour Active Directory Domain Services ou Windows Azure Active Directory dans l’interface utilisateur Synchronization Service Manager.
- Résolution du problème d’accessibilité des contrôles d’interface utilisateur personnalisés dans Synchronization Service Manager.
- Activation de six tâches de gestion de la fédération pour toutes les méthodes de connexion dans Azure AD Connect.  (Auparavant, seule la tâche « Mettre à jour le certificat TLS/SSL AD FS » était disponible pour l'ensemble des connexions.)
- Ajout d’un avertissement lors de la modification de la méthode de connexion, qui passe de la fédération à la synchronisation de hachage de mot de passe (PHS) ou à l’authentification directe (PTA), convertissant tous les utilisateurs et domaines Azure AD en authentification managée.
- Suppression des certificats de signature de jeton dans la tâche « Réinitialiser Azure AD et l’approbation AD FS » et ajout d’une sous-tâche distincte pour mettre à jour ces certificats.
- Ajout d'une nouvelle tâche de gestion de fédération appelée « Gérer les certificats » qui comporte des sous-tâches permettant de mettre à jour les certificats de signature de jeton ou les certificats TLS de la batterie de serveurs AD FS.
- Ajout d’une nouvelle sous-tâche de gestion de la fédération appelée « Spécifier le serveur principal », qui permet aux administrateurs de préciser un nouveau serveur principal pour la batterie de serveurs AD FS.
- Ajout d’une nouvelle tâche de gestion de la fédération appelée « Gérer les serveurs », qui comporte des sous-tâches permettant de déployer un serveur AD FS, un serveur proxy d’application web et de spécifier le serveur principal.
- Ajout d’une nouvelle tâche de gestion de la fédération appelée « Afficher la configuration de la fédération », qui affiche les paramètres AD FS actuels.  (En raison de cet ajout, les paramètres AD FS ont été supprimés de la page « Vérification de votre solution ».)

### <a name="fixed-issues"></a>Problèmes résolus
- Résolution du problème d’erreur de synchronisation dans le scénario suivant lequel un objet utilisateur qui prend le contrôle de son objet contact correspondant dispose d’une autoréférence (par exemple, l’utilisateur est son propre manager).
- Les fenêtres contextuelles d’aide s’affichent désormais sur le focus clavier.
- Pour la mise à niveau automatique, si une application en conflit s’exécute pendant 6 heures, il y a possibilité de la supprimer et de poursuivre la mise à niveau.
- Limitation du nombre d’attributs qu’un client peut sélectionner à 100 par objet lors de la sélection des extensions d’annuaire. De cette façon, l’erreur ne peut pas se produire pendant l’exportation, car Azure présente un maximum de 100 attributs d’extension par objet.
- Correction d’un bogue permettant de renforcer le script de connectivité AD.
- Correction d’un bogue permettant de consolider l’installation d’Azure AD Connect sur une machine qui utilise un service WCF de canaux nommés.
- Amélioration des diagnostics et de la résolution des problèmes liés aux stratégies de groupe qui n’autorisent pas le démarrage du service ADSync lors de l’installation initiale.
- Correction d’un bogue responsable de l’écriture incorrecte du nom d’affichage d’un ordinateur Windows.
- Correction d’un bogue responsable de l’écriture incorrecte du type de système d’exploitation d’un ordinateur Windows.
- Correction d’un bogue responsable de la synchronisation de manière inattendue d’ordinateurs non-Windows 10. Notez que l’effet de cette modification se traduit désormais par la suppression des ordinateurs non-Windows 10 qui, auparavant, étaient synchronisés. Cela n’a pas d’incidence sur les fonctionnalités, car la synchronisation des ordinateurs Windows est uniquement utilisée pour la jonction de domaine Azure AD Hybride, qui ne fonctionne que pour les appareils Windows 10.
- Ajout de plusieurs nouvelles applets de commande (internes) au module PowerShell ADSync.


## <a name="13210"></a>1.3.21.0
>[!IMPORTANT]
>Il existe un problème connu lié à la mise à niveau d’Azure AD Connect d’une version antérieure vers 1.3.21.0, où le portail Microsoft 365 ne reflète pas la version mise à jour même si Azure AD Connect a été mis à niveau avec succès.
>
> Pour résoudre ce problème, vous devez importer le module **AdSync**, puis exécuter l’applet de commande PowerShell `Set-ADSyncDirSyncConfiguration` sur le serveur Azure AD Connect.  Vous pouvez procéder comme suit :
>
>1. Ouvrez PowerShell en mode Administrateur.
>2. Exécutez `Import-Module "ADSync"`.
>3. Exécutez `Set-ADSyncDirSyncConfiguration -AnchorAttribute ""`.
 
### <a name="release-status"></a>État de la version 

14/05/2019 : publiée pour téléchargement

### <a name="fixed-issues"></a>Problèmes résolus 

- Correction d’une vulnérabilité avec élévation de privilège qui existe dans Microsoft Azure Active Directory Connect build 1.3.20.0.  Sous certaines conditions, cette vulnérabilité peut permettre à un attaquant d’exécuter deux applets de commande PowerShell dans le contexte d’un compte privilégié et d’effectuer des actions privilégiées.  Cette mise à jour de sécurité résout le problème en désactivant ces cmdlets. Pour plus d’informations, reportez-vous à la [mise à jour de sécurité](https://portal.msrc.microsoft.com/security-guidance/advisory/CVE-2019-1000).


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
