---
title: FAQ Azure Active Directory Connect | Microsoft Docs
description: Cette page répond aux questions fréquentes à propos d’Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 4e47a087-ebcd-4b63-9574-0c31907a39a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/23/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ff2651d2fed9c307bbb90623572b4a74d750d3a4
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836068"
---
# <a name="azure-active-directory-connect-faq"></a>FAQ Azure Active Directory Connect

## <a name="general-installation"></a>Installation générale

**Q : Comment puis-je renforcer mon serveur Azure AD Connect pour réduire la surface d’attaque de sécurité ?**

Microsoft vous recommande de renforcer votre serveur Azure AD Connect afin de réduire la surface d’attaque de sécurité de ce composant essentiel de votre environnement informatique.  L’application des recommandations ci-après vous permettra de limiter les risques pour la sécurité dans votre organisation.

* Déployez Azure AD Connect sur un serveur joint au domaine et restreignez l’accès administratif aux administrateurs de domaine ou à d’autres groupes de sécurité étroitement contrôlés.

Pour plus d'informations, consultez les rubriques suivantes : 

* [Securing administrators groups](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-g--securing-administrators-groups-in-active-directory) (Sécurisation des groupes d’administrateurs)

* [Securing built-in administrator accounts](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-d--securing-built-in-administrator-accounts-in-active-directory) (Sécurisation des comptes Administrateur intégrés)

* [Security improvement and sustainment by reducing attack surfaces](/windows-server/identity/securing-privileged-access/securing-privileged-access#2-reduce-attack-surfaces ) (Amélioration et maintien en état de la sécurité par la réduction des surfaces d’attaque)

* [Reducing the Active Directory attack surface](/windows-server/identity/ad-ds/plan/security-best-practices/reducing-the-active-directory-attack-surface) (Réduction de la surface d’attaque Active Directory)

**Q : L’installation fonctionnera-t-elle si l’authentification à deux facteurs est activée pour l’administrateur général Azure Active Directory (Azure AD) ?**  
Pour les versions publiées à compter de février 2016, ce scénario est pris en charge.

**Q : Existe-t-il une manière d’installer Azure AD Connect sans assistance ?**  
L’installation d’Azure AD Connect est prise en charge uniquement lorsque vous utilisez l’assistant d’installation. L’installation automatique et silencieuse n’est pas prise en charge.

**Q : Je possède une forêt dans laquelle un domaine ne peut pas être contacté. Comment installer Azure AD Connect ?**  
Pour les versions publiées à compter de février 2016, ce scénario est pris en charge.

**Q : L’agent de contrôle d’intégrité d’Azure Active Directory Domain Services (Azure AD DS) fonctionne-t-il sur le noyau du serveur ?**  
Oui. Après avoir installé l’agent, vous pouvez terminer le processus d’inscription à l’aide du cmdlet PowerShell suivant : 

`Register-AzureADConnectHealthADDSAgent -Credentials $cred`

**Q : Azure AD Connect prend-il en charge la synchronisation à partir de deux domaines vers Azure AD ?**  
Oui, ce scénario est pris en charge. Reportez-vous à [Plusieurs domaines](how-to-connect-install-multiple-domains.md).
 
**Q : Est-il possible d’avoir plusieurs connecteurs pour le même domaine Active Directory dans Azure AD Connect ?**  
Non, il n’est pas possible d’avoir plusieurs connecteurs pour le même domaine AD. 

**Q : Est-il possible de déplacer la base de données Azure AD Connect d’une base de données locale vers une instance SQL Server distante ?**    
Oui. Les étapes suivantes expliquent la procédure générale à suivre. Nous travaillons actuellement sur un document plus détaillé.
1. Sauvegardez la base de données LocalDB ADSync.
La façon la plus simple de le faire consiste à utiliser SQL Server Management Studio, installé sur le même ordinateur qu’Azure AD Connect. Connectez-vous à *(localdb)\ADSync*, puis sauvegardez la base de données ADSync.

2. Restaurez la base de données ADSync sur votre instance SQL Server à distance.

3. Installez Azure AD Connect sur la [base de données SQL distante](how-to-connect-install-existing-database.md) existante.
   L’article montre comment effectuer la migration à l’aide d’une base de données SQL locale. Si vous passez à une base de données SQL à distance, vous devez également entrer un compte de service existant à l’étape 5 de ce processus ; le Service de synchronisation Windows s’exécutera sous ce compte de service du moteur de synchronisation, décrit ici :
   
      **Utiliser un compte de service existant** : Par défaut, Azure AD Connect utilise un compte de service virtuel, que les services de synchronisation doivent utiliser. Si vous utilisez une instance SQL Server distante ou un proxy qui exige une authentification, utilisez un compte de service administré ou un compte de service dans le domaine dont vous connaissez le mot de passe. Dans ce cas, entrez le compte à utiliser. Assurez-vous que les utilisateurs qui exécutent l’installation sont des administrateurs système dans SQL afin que les informations d’identification du compte de service puissent être créées. Pour en savoir plus, consultez la page [Autorisations et comptes Azure AD Connect](reference-connect-accounts-permissions.md#adsync-service-account). 
   
      Avec la version la plus récente, l’approvisionnement de la base de données peut désormais être exécuté hors bande par l’administrateur SQL. L’installation s’effectue ensuite par l’administrateur Azure AD Connect disposant des droits du propriétaire de la base de données. Pour plus d’informations, consultez la page [Installer Azure AD Connect à l’aide d’autorisations administrateur déléguées SQL](how-to-connect-install-sql-delegation.md).

Pour simplifier les choses, nous recommandons que les utilisateurs qui installent Azure AD Connect soient des administrateurs système dans SQL. Toutefois, les récentes versions vous permettent maintenant d’utiliser des administrateurs SQL délégués, comme le décrit la page [Installer Azure AD Connect à l’aide d’autorisations administrateur déléguées SQL](how-to-connect-install-sql-delegation.md).

**Q : Quelles sont les meilleures pratiques applicables sur le terrain ?**  

Les informations ci-dessous présentent quelques-unes des meilleures pratiques élaborées au fil des années par nos équipes d’ingénierie, de support et de conseil.  Ces informations se présentent sous la forme d’une liste à puces pour en faciliter la consultation.  Bien que cette liste soit la plus exhaustive possible, il est possible que certaines meilleures pratiques n’y figurent pas encore.

- Si vous utilisez un serveur SQL complet, utilisez un serveur local plutôt que distant :
    - Diminution du nombre de tronçons
    - Simplification de la détection des problèmes
    - Réduction de la complexité
    - Nécessité de désignation des ressources à SQL et autorisation de surcharge d’Azure AD Connect et du système d’exploitation
- Contournez le proxy dans la mesure du possible. Si vous n’y parvenez pas, assurez-vous que la valeur de délai d’expiration est supérieure à 5 minutes.
- Si un proxy est requis, vous devez l’ajouter au fichier machine.config.
- Tenez compte des travaux SQL locaux et des tâches de maintenance, ainsi que de leur impact sur Azure AD Connect, notamment la réindexation.
- Vérifiez la possibilité de résolution de DNS en externe.
- Assurez-vous que les [spécifications de serveur](how-to-connect-install-prerequisites.md#hardware-requirements-for-azure-ad-connect) sont conformes aux recommandations, que vous utilisiez des serveurs physiques ou virtuels.
- Vérifiez que les ressources requises sont dédiées si vous utilisez un serveur virtuel.
- Assurez-vous que le disque et la configuration de disque sont conformes aux meilleures pratiques pour SQL Server.
- Installez et configurez Azure AD Connect Health à des fins de surveillance.
- Utilisez le seuil de suppression intégré à Azure AD Connect.
- Examinez soigneusement les mises à jour de version pour vous préparer à toutes les modifications et aux nouveaux attributs qui peuvent être ajoutés.
- Sauvegardez tous les éléments :
    - Clés
    - Règles de synchronisation
    - Configuration du serveur
    - SQL Database
- Vérifiez qu’aucun agent de sauvegarde tiers ne sauvegarde SQL sans l’enregistreur VSS SQL (courant sur les serveurs virtuels avec des captures instantanées tierces).
- Limitez le nombre de règles de synchronisation personnalisées utilisées, car elles ajoutent de la complexité.
- Traitez les serveurs Azure AD Connect comme des serveurs de niveau 0.
- Évitez de modifier les règles de synchronisation cloud si vous n’en comprenez pas parfaitement l’impact et les axes stratégiques adéquats.
- Vérifiez l’ouverture des ports d’URL et de pare-feu corrects pour la prise en charge d’Azure AD Connect et d’Azure AD Connect Health.
- Tirez parti de l’attribut filtré de cloud pour détecter les problèmes d’objets fantômes et éviter ces objets.
- Avec le serveur de préproduction, vérifiez que vous utilisez la documentation de configuration d’Azure AD Connect pour garantir la cohérence entre les serveurs.
- Les serveurs de préproduction doivent se trouver dans des centres de données (emplacements physiques) distincts.
- Les serveurs de préproduction ne sont pas destinés à constituer une solution de haute disponibilité, mais vous pouvez en utiliser plusieurs.
- L’introduction d’un « décalage » dans les serveurs de préproduction peut atténuer certains risques de temps d’arrêt en cas d’erreur.
- Commencez par tester et valider toutes les mises à niveau sur le serveur de préproduction.
- Validez toujours les exportations avant de basculer vers le serveur de préproduction.  Afin de réduire l’impact sur l’activité, utilisez le serveur de préproduction pour effectuer des importations complètes et des synchronisations complètes.
- Dans la mesure du possible, assurez la cohérence des versions entre les serveurs Azure AD Connect. 

**Q : Puis-je autoriser Azure AD Connect à créer le compte de connecteur Azure AD sur une machine de groupe de travail ?**
Non.  Si vous souhaitez autoriser Azure AD Connect à créer automatiquement le compte de connecteur Azure AD, la machine doit être jointe au domaine.  

## <a name="network"></a>Réseau
**Q : Je dispose d’un pare-feu, d’un périphérique réseau ou d’une autre méthode qui limite la durée pendant laquelle les connexions peuvent rester ouvertes sur mon réseau. Quel doit être le seuil du délai d’expiration côté client lorsque j’utilise Azure AD Connect ?**  
Tous les logiciels réseau, périphériques physiques ou autres méthodes limitant la durée maximale pendant laquelle les connexions peuvent rester ouvertes doivent utiliser un seuil d’au moins cinq minutes (300 secondes) pour la connectivité entre le serveur où est installé le client Azure AD Connect et Azure Active Directory. Cette recommandation s’applique également à tous les outils de synchronisation des identités Microsoft précédemment publiés.

**Q : Les domaines avec un nom en une seule partie sont-ils pris en charge ?**  
Même si cette configuration réseau est fortement déconseillée ([consultez cet article](https://support.microsoft.com/help/2269810/microsoft-support-for-single-label-domains)), l’utilisation de la synchronisation Azure AD Connect avec un nom de domaine en une seule partie est prise en charge, tant que la configuration réseau du domaine à un seul niveau fonctionne correctement.

**Q : Les forêts comportant des domaines AD disjoints sont-elles prises en charge ?**  
Non, Azure AD Connect ne prend pas en charge les forêts locales qui contiennent des espaces de noms disjoints.

**Q : Les noms NetBIOS comportant un point sont-ils pris en charge ?**  
Non, Azure AD Connect ne prend pas en charge les forêts ni les domaines locaux dont le nom NetBIOS contient un point (.).

**Q : L'environnement IPv6 pur est-il pris en charge ?**  
Non, Azure AD Connect ne prend pas en charge les environnements IPv6 purs.

**Q : j’ai un environnement à forêts multiples et le réseau entre les deux forêts utilise la traduction d’adresses réseau (Network Address Translation). Est-il possible d’utiliser Azure AD Connect entre ces deux forêts ?**</br>
Non, l’utilisation d’Azure AD Connect sur NAT n’est pas prise en charge. 

## <a name="federation"></a>Fédération
**Q : Que faire si je reçois un e-mail me demandant de renouveler mon certificat Microsoft 365 ?**  
Pour obtenir des conseils sur le renouvellement du certificat, consultez la page [Renouveler les certificats](how-to-connect-fed-o365-certs.md).

**Q : « Mettre à jour automatiquement la partie de confiance » est défini pour la partie de confiance Microsoft 365. Dois-je effectuer une action lorsque mon certificat de signature de jetons bascule automatiquement ?**  
Utilisez les instructions décrites dans l’article [Renouveler les certificats](how-to-connect-fed-o365-certs.md).

## <a name="environment"></a>Environnement
**Q : Est-il possible de renommer le serveur une fois qu’Azure AD Connect est installé ?**  
Non. Lorsque le nom du serveur est modifié, le moteur de synchronisation n’est pas en mesure de se connecter à l’instance de base de données SQL et le service ne peut pas démarrer.

**Q : Les règles de synchronisation de chiffrement nouvelle génération (NGC, Next Generation Cryptographic) sont-elles prises en charge sur une machine FIPS ?**  
Non.  Elles ne sont pas prises en charge.

**Q. Si j’ai désactivé un appareil synchronisé (par exemple : HAADJ) dans le portail Azure, pourquoi est-il réactivé ?**<br>
Les appareils synchronisés peuvent être créés ou contrôlés localement. Si un appareil synchronisé est activé localement, il peut être réactivé dans le portail Azure même s’il a été précédemment désactivé par un administrateur. Pour désactiver un appareil synchronisé, utilisez le répertoire Active Directory local pour désactiver le compte d’ordinateur.

**Q. Si je bloque la connexion de l’utilisateur sur le portail Microsoft 365 ou Azure AD pour les utilisateurs synchronisés, pourquoi est-elle débloquée lors de la connexion ?**<br>
Les utilisateurs synchronisés peuvent être créés ou contrôlés localement. Si le compte est activé localement, il peut débloquer le blocage de connexion placé par l’administrateur.

## <a name="identity-data"></a>Données d’identité
**Q : Pourquoi l’attribut userPrincipalName (UPN) dans Azure AD ne correspond-t-il à l’UPN local ?**  
Pour plus d’informations, consultez les articles suivants :

* [Les noms d’utilisateur dans Microsoft 365, Azure ou Intune ne correspondent pas à l’UPN local ou à l’ID de connexion secondaire.](https://support.microsoft.com/kb/2523192)
* [Les modifications ne sont pas synchronisées par l’outil de synchronisation Azure Active Directory une fois que vous avez modifié l’UPN d’un compte utilisateur afin d’utiliser un autre domaine fédéré](https://support.microsoft.com/kb/2669550)

Vous pouvez également configurer Azure AD pour permettre au moteur de synchronisation de mettre à jour l’UPN comme décrit dans [Fonctionnalités du service de synchronisation Azure AD Connect](how-to-connect-syncservice-features.md).

**Q : La correspondance approximative entre un objet de groupe ou contact Azure AD local et un objet de groupe ou contact Azure AD existant est-elle prise en charge ?**  
Oui, cette correspondance approximative est basée sur l’adresse proxy. La correspondance approximative n’est pas prise en charge pour les groupes qui ne sont pas à extension messagerie.

**Q : Est-il possible de définir manuellement l’attribut ImmutableId sur un objet de groupe ou contact Azure AD existant pour la mise en correspondance exacte avec un objet de groupe ou contact local ?**  
Non, la définition manuelle de l’attribut ImmutableId sur un objet de groupe ou contact Azure AD existant pour la correspondance exacte n’est pas prise en charge à l’heure actuelle.

## <a name="custom-configuration"></a>Configuration personnalisée
**Q : Où réside la documentation sur les applets de commande PowerShell pour Azure Active Directory Connect ?**  
À l’exception des applets de commande qui sont décrites sur ce site, les autres applets de commande PowerShell disponibles dans Azure AD Connect ne sont pas prises en charge par le client.

**Q : Puis-je utiliser l’option « exportation serveur/importation serveur » dans Synchronization Service Manager pour déplacer la configuration entre des serveurs ?**  
Non. Cette option ne récupère pas tous les paramètres de configuration et ne doit pas être utilisée. Utilisez plutôt l’assistant pour créer la configuration de base sur le deuxième serveur et utilisez l’éditeur de règles de synchronisation pour générer des scripts PowerShell afin de déplacer une règle personnalisée entre les serveurs. Pour plus d’informations, consultez la page [Migration Swing](how-to-upgrade-previous-version.md#swing-migration).

**Q : Les mots de passe peuvent-ils être mis en cache pour la page de connexion Azure ? Est-il possible d’empêcher cette mise en cache, étant donné que la page contient un élément d’entrée de mot de passe avec l’attribut *autocomplete = "false"* ?**  
Pour l’instant, la modification des attributs HTML du champ **Mot de passe**, et notamment de la balise autocomplete, n’est pas prise en charge. Nous travaillons actuellement à l’élaboration d’une fonctionnalité autorisant le code Javascript personnalisé, afin de vous permettre d’ajouter n’importe quel attribut au champ **Mot de passe**.

**Q : La page de connexion Azure affiche les noms des utilisateurs qui se sont déjà connectés avec succès. Est-il possible de désactiver ce comportement ?**  
Pour l’instant, la modification des attributs HTML du champ d’entrée **Mot de passe**, et notamment de la balise autocomplete, n’est pas prise en charge. Nous travaillons actuellement à l’élaboration d’une fonctionnalité autorisant le code Javascript personnalisé, afin de vous permettre d’ajouter n’importe quel attribut au champ **Mot de passe**.

**Q : Existe-t-il un moyen d’empêcher les sessions simultanées ?**  
Non.

## <a name="auto-upgrade"></a>Mise à niveau automatique

**Q : Quels sont les avantages et les conséquences de l’utilisation de la mise à niveau automatique ?**  
Nous conseillons à tous les clients d’activer la mise à niveau automatique pour leur installation d’Azure AD Connect. Ainsi, ils recevront toujours les derniers correctifs, y compris les mises à jour de sécurité pour les vulnérabilités identifiées dans Azure AD Connect. Le processus de mise à niveau est simple et se fait automatiquement dès qu’une nouvelle version est disponible. Plusieurs milliers de clients Azure AD Connect utilisent la mise à niveau automatique à chaque nouvelle version.

Dans tous les cas, le processus de mise à niveau automatique établit d’abord si une installation est éligible à la mise à niveau automatique. Si tel est le cas, la mise à niveau est effectuée et testée. Le processus inclut également la recherche de modifications personnalisées apportées aux règles et aux facteurs environnementaux. Si les tests indiquent l’échec d’une mise à niveau, la version précédente est automatiquement restaurée.

Selon la taille de l’environnement, le processus peut prendre quelques heures. Lorsque la mise à niveau est en cours, il n’y a aucune synchronisation entre Windows Server Active Directory et Azure AD.

**Q : J’ai reçu un e-mail indiquant que ma mise à niveau automatique ne fonctionne plus et que je dois installer la nouvelle version. Pourquoi dois-je effectuer cette opération ?**  
L’année dernière, nous avons publié une version d’Azure AD Connect qui, dans certaines circonstances, désactivait la fonctionnalité de mise à niveau automatique sur votre serveur. Nous avons résolu le problème dans Azure AD Connect version 1.1.750.0. Si vous êtes touché par le problème, vous pouvez résoudre l’erreur en exécutant un script PowerShell afin de réparer la mise à niveau automatique ou passer manuellement à la dernière version d’Azure AD Connect. 

Pour exécuter le script PowerShell, [téléchargez le script PowerShell](https://aka.ms/repairaadconnect) et exécutez-le sur votre serveur Azure AD Connect dans une fenêtre PowerShell d’administration. Pour savoir comment exécuter le script, [regardez cette courte vidéo](https://aka.ms/repairaadcau).

Pour effectuer manuellement la mise à niveau, vous devez télécharger et exécuter la dernière version du fichier AADConnect.msi.
 
-  Si votre version actuelle est antérieure à la version 1.1.750.0, [téléchargez la dernière version pour la mise à niveau](https://www.microsoft.com/download/details.aspx?id=47594).
- Si vous utilisez la version 1.1.750.0 (ou ultérieure) d’Azure AD Connect, aucune action supplémentaire n’est requise. Vous utilisez déjà la version qui contient le correctif pour la mise à niveau automatique. 

**Q : J’ai reçu un e-mail me demandant de mettre à niveau vers la version la plus récente pour réactiver la mise à niveau automatique. J’utilise la version 1.1.654.0. Dois-je effectuer la mise à niveau ?**  
Oui, vous devez mettre à niveau vers la version 1.1.750.0 ou ultérieure pour réactiver la mise à niveau automatique. [Téléchargez la dernière version pour la mise à niveau](https://www.microsoft.com/download/details.aspx?id=47594).

**Q : J’ai reçu un e-mail me demandant de mettre à niveau vers la version la plus récente pour réactiver la mise à niveau automatique. Si j’ai utilisé PowerShell pour activer la mise à niveau automatique, dois-je quand même installer la dernière version ?**  
Oui, vous devez tout de même mettre à niveau vers la version 1.1.750.0 ou ultérieure. L’activation du service de mise à niveau automatique avec PowerShell ne résout pas le problème de mise à niveau automatique repéré dans les versions antérieures à la version 1.1.750.0.

**Q : Je souhaite mettre à niveau vers une version plus récente, mais je ne sais pas qui a installé Azure AD Connect et nous n’avons pas le nom d’utilisateur et le mot de passe. En avons-nous besoin ?**
Vous n’avez pas besoin de connaître le nom d’utilisateur et le mot de passe qui ont été initialement utilisés pour mettre à niveau Azure AD Connect. Utilisez n’importe quel compte Azure AD disposant du rôle d’administrateur général.

**Q : Comment puis-je connaître la version d’Azure AD Connect dont je dispose ?**  
Pour vérifier quelle version d’Azure AD Connect est installée sur votre serveur, accédez au Panneau de configuration et recherchez la version de Microsoft Azure AD Connect qui est installée en sélectionnant **Programmes** > **Programmes et fonctionnalités**, comme illustré ici :

![Version d’Azure AD Connect dans le Panneau de configuration](./media/reference-connect-faq/faq1.png)

**Q : Comment effectuer la mise à niveau vers la dernière version d’Azure AD Connect ?**  
Pour savoir comment effectuer la mise à niveau d’Azure AD Connect vers la dernière version, consultez [Azure AD Connect : effectuer une mise à niveau vers la dernière version](how-to-upgrade-previous-version.md). 

**Q : Nous avons déjà effectué une mise à niveau vers la dernière version d’Azure AD Connect l’année dernière. Devons-nous procéder à une nouvelle mise à niveau ?**  
L’équipe Azure AD Connect apporte fréquemment des mises à jour au service. Pour bénéficier des correctifs de bogues, des mises à jour de sécurité et des nouvelles fonctionnalités, il est important d’utiliser la dernière version sur votre serveur. Si vous activez la mise à niveau automatique, votre version logicielle sera automatiquement mise à jour. Pour rechercher l’historique des versions d’Azure AD Connect, consultez la page [Azure AD Connect : historique de publication des versions](reference-connect-version-history.md).

**Q : Combien de temps faut-il pour effectuer la mise à niveau et quel est l’impact sur mes utilisateurs ?**  
Le temps requis pour la mise à niveau dépend de la taille de votre locataire. Pour les grandes organisations, il peut être préférable d’effectuer la mise à niveau le soir ou le week-end. Lors de la mise à niveau, aucune activité de synchronisation n’a lieu.

**Q : Je pense que j’ai effectué la mise à niveau vers Azure AD Connect, mais dans le portail Office, il est toujours fait mention de DirSync. Pourquoi ?**  
L’équipe Office travaille actuellement pour que les mises à jour du portail Office reflètent le nom du produit actuel. Il ne reflète pas l’outil de synchronisation que vous utilisez.

**Q : Mon statut de mise à niveau automatique indique « Suspendu ». Pourquoi ? Dois-je l’activer ?**  
Un bogue a été introduit dans une version précédente : dans certains cas, l’état de la mise à niveau automatique a pris la valeur « Suspendu ». Techniquement, l’activation manuelle est possible, mais elle requiert plusieurs étapes complexes. Par conséquent, il est préférable d’installer la dernière version d’Azure AD Connect.

**Q : Mon entreprise a des exigences strictes en matière de gestion des changements et je souhaite contrôler à quel moment les mises à niveau ont lieu. Puis-je contrôler le lancement des mises à niveau automatiques ?**  
Non, il n’existe encore aucune fonctionnalité de ce type. La fonction est évaluée afin d’être publiée dans une prochaine version.

**Q : Recevrai-je un e-mail en cas d’échec de la mise à niveau automatique ? Comment savoir si elle a réussi ?**  
Vous ne serez pas informé du résultat de la mise à niveau. La fonction est évaluée afin d’être publiée dans une prochaine version.

**Q : Publiez-vous un calendrier des mises à niveau automatiques prévues ?**  
La mise à niveau automatique constitue la première étape du processus de publication d’une version plus récente. Chaque fois qu’une nouvelle version est publiée, les mises à niveau sont envoyées automatiquement. Les nouvelles versions d’Azure AD Connect sont préalablement annoncées dans la [feuille de route Azure AD](../fundamentals/whats-new.md).

**Q : La mise à niveau automatique met-elle également à jour Azure AD Connect Health ?**  
Oui, la mise à niveau automatique met également à jour Azure AD Connect Health.

**Q : Les serveurs Azure Active Directory Connect en mode de préproduction sont-ils également concernés par la mise à niveau automatique ?**  
Oui, vous pouvez mettre automatiquement à niveau un serveur Azure AD Connect en mode de préproduction.

**Q : Si la mise à niveau automatique échoue et que mon serveur Azure AD Connect ne démarre pas, que dois-je faire ?**  
Dans de rares cas, le service Azure AD Connect ne démarre pas après la mise à niveau. Dans ces cas, redémarrez le serveur. Cela permet généralement de résoudre le problème. Si le service Azure AD Connect ne démarre toujours pas, ouvrez un ticket de support. Pour plus d’informations, consultez la page décrivant la procédure de [création d’une demande de service pour contacter le support Microsoft 365](/archive/blogs/praveenkumar/how-to-create-service-requests-to-contact-office-365-support). 

**Q : Je ne sais pas quels sont les risques liés à la mise à niveau vers une version plus récente d’Azure AD Connect. Pouvez-vous m’appeler pour m’aider à procéder à la mise à niveau ?**  
Si vous avez besoin d’aide pour passer à une version plus récente d’Azure AD Connect, ouvrez un ticket de support en suivant les instructions permettant de [créer une demande de service pour contacter le support Microsoft 365](/archive/blogs/praveenkumar/how-to-create-service-requests-to-contact-office-365-support).

## <a name="operational-best-practice"></a>Bonnes pratiques opérationnelles    
Vous trouverez ci-dessous quelques bonnes pratiques à appliquer lors de la synchronisation entre Windows Server Active Directory et Azure Active Directory.

**Appliquez Multi-Factor Authentication sur tous les comptes synchronisés** Azure AD Multi-Factor Authentication permet de sécuriser l'accès aux données et aux applications tout en offrant une simplicité de gestion aux utilisateurs. MFA fournit une sécurité supplémentaire en exigeant une deuxième forme d'authentification, et procure une authentification renforcée par le biais d'un éventail de méthodes d'authentification faciles à utiliser. Les utilisateurs peuvent devoir s'authentifier via MFA selon les choix de configuration de l'administrateur. Pour plus d'informations sur MFA, cliquez ici : https://www.microsoft.com/security/business/identity/mfa?rtc=1

**Suivez les consignes de sécurité relatives au serveur Azure AD Connect** Le serveur Azure AD Connect contient des données d'identité critiques et doit être traité comme un composant de niveau 0, comme expliqué dans le [modèle de niveau administratif Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material). Consultez également les [consignes de sécurisation de votre serveur AADConnect](./how-to-connect-install-prerequisites.md#azure-ad-connect-server).

**Activez la synchronisation du hachage de mot de passe pour la détection des fuites d'informations d'identification** La synchronisation du hachage de mot de passe permet également la [détection des fuites d'informations d'identification](../identity-protection/concept-identity-protection-risks.md) sur vos comptes hybrides. Microsoft travaille en collaboration avec les chercheurs sur le «dark web » et les autorités policières pour trouver les paires nom d’utilisateur/mot de passe disponibles publiquement. Si l'une de ces paires correspond à celles de vos utilisateurs, le compte associé passe au niveau de risque élevé. 


## <a name="troubleshooting"></a>Dépannage
**Q : Comment puis-je obtenir de l’aide avec Azure AD Connect ?**

[Recherche dans la Base de connaissances Microsoft (KB)](https://www.microsoft.com/en-us/search/result.aspx?q=azure+active+directory+connect)

* Dans la base de connaissances, recherchez des solutions techniques aux problèmes courants couverts par la garantie de réparation et d’assistance, relatifs à la prise en charge d’Azure AD Connect.

[Page de questions Microsoft Q&R sur Azure Active Directory](/answers/topics/azure-active-directory.html)

* Recherchez des questions techniques et obtenez des réponses, ou posez vos questions en parcourant la [communauté Azure AD](/answers/topics/azure-active-directory.html).

[Bénéficier du support pour Azure AD](../fundamentals/active-directory-troubleshooting-support-howto.md)

**Q : Pourquoi les événements 6311 et 6401 se produisent-ils après des erreurs d’étape de synchronisation?**

Les événements 6311 **Le serveur a rencontré une erreur inattendue lors de l’exécution d’un rappel** et 6401 **Le contrôleur de l’agent de gestion a rencontré une erreur inattendue** sont toujours consignés après une erreur d’étape de synchronisation. Pour résoudre ces erreurs, vous devez nettoyer les erreurs de l’étape de synchronisation.  Pour plus d’informations, consultez [Résolution des erreurs lors de la synchronisation](tshoot-connect-sync-errors.md) et [Résolution des problèmes de synchronisation d’objet avec la synchronisation Azure AD Connect](tshoot-connect-objectsync.md)