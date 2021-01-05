---
title: Résoudre les problèmes du runtime d’intégration auto-hébergé dans Azure Data Factory
description: Découvrez comment résoudre les problèmes liés au runtime d’intégration auto-hébergé dans Azure Data Factory.
services: data-factory
author: lrtoyou1223
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 11/17/2020
ms.author: lle
ms.openlocfilehash: ccebdbf428180f8ff4ab10dc6007c3ec35a66362
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503571"
---
# <a name="troubleshoot-self-hosted-integration-runtime"></a>Résoudre les problèmes liés au runtime d’intégration auto-hébergé

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article présente des méthodes couramment employées pour résoudre les problèmes liés au runtime d’intégration (IR) auto-hébergé dans Azure Data Factory.

## <a name="gather-self-hosted-ir-logs-from-azure-data-factory"></a>Collecter les journaux de l’IR auto-hébergé auprès d’Azure Data Factory

Azure Data Factory prend en charge l’affichage et le chargement des journaux d’erreurs pour les activités ayant échoué qui sont exécutées sur un IR auto-hébergé ou partagé. Pour obtenir l’ID de rapport d’erreurs, suivez les instructions fournies ici, puis entrez l’ID de rapport pour rechercher les problèmes connus associés.

1. Dans Data Factory, sélectionnez **Exécutions de pipeline**.

1. Sous **Exécutions de l’activité**, dans la colonne **Erreur**, sélectionnez le bouton en surbrillance pour afficher les journaux d’activité, comme illustré dans la capture d’écran suivante :

    ![Capture d’écran de la section « Exécutions de l’activité » du volet « Toutes les exécutions de pipelines ».](media/self-hosted-integration-runtime-troubleshoot-guide/activity-runs-page.png)

    Les journaux d’activité s’affichent pour l’exécution de l’activité qui a échoué.

    ![Capture d’écran des journaux d’activité pour l’activité ayant échoué.](media/self-hosted-integration-runtime-troubleshoot-guide/send-logs.png) 
    
1. Pour obtenir de l’aide, sélectionnez **Envoyer des journaux**.
 
   La fenêtre **Share the self-hosted integration runtime (IR) logs with Microsoft (Partager les journaux du runtime d’intégration (IR) auto-hébergé avec Microsoft)** s’ouvre.

    ![Capture d’écran de la fenêtre « Partager les journaux du runtime d’intégration (IR) auto-hébergé avec Microsoft ».](media/self-hosted-integration-runtime-troubleshoot-guide/choose-logs.png)

1. Sélectionnez les journaux que vous souhaitez envoyer. 
    * Pour un *IR auto-hébergé*, vous pouvez charger les journaux relatifs à l’activité ayant échoué ou tous les journaux sur le nœud de l’IR auto-hébergé. 
    * Pour un *IR partagé*, vous pouvez charger uniquement les journaux liés à l’activité ayant échoué.

1. Lorsque les journaux sont chargés, conservez une trace de l’ID du rapport au cas où vous auriez besoin d’une aide supplémentaire ultérieurement pour résoudre le problème.

    ![Capture d’écran de l’ID de rapport affiché dans la fenêtre de progression du chargement pour les journaux IR.](media/self-hosted-integration-runtime-troubleshoot-guide/upload-logs.png)

> [!NOTE]
> Les requêtes de consultation et de chargement des journaux sont exécutées sur toutes les instances de l’IR auto-hébergé en ligne. S’il manque des journaux, assurez-vous que toutes les instances de l’IR auto-hébergé sont en ligne. 


## <a name="self-hosted-ir-general-failure-or-error"></a>Erreur générale ou échec général de l’IR auto-hébergé

### <a name="out-of-memory-issue"></a>Problème de mémoire insuffisante

#### <a name="symptoms"></a>Symptômes

Une erreur OutOfMemoryException (Mémoire insuffisante) se produit lorsque vous essayez d’exécuter une activité de recherche avec un IR lié ou un IR auto-hébergé.

#### <a name="cause"></a>Cause

Une nouvelle activité peut générer une erreur Mémoire insuffisante si l’ordinateur de l’IR subit une utilisation de mémoire élevée momentanément. Le problème peut être dû à l’exécution de nombreuses activités simultanées et l’erreur est due à la conception.

#### <a name="resolution"></a>Résolution

Vérifiez l’utilisation des ressources et l’exécution simultanée d’activités sur le nœud IR. Ajustez l’heure interne et l’heure de déclenchement des exécutions d’activité pour éviter un trop grand nombre d’exécutions sur un même nœud IR en même temps.


### <a name="ssltls-certificate-issue"></a>Problème de certificat TLS/SSL

#### <a name="symptoms"></a>Symptômes

Lorsque vous essayez d’activer un certificat SSL (Secure Sockets Layer)/TLS (Transport Layer Security) (avancé) en choisissant le certificat (après avoir sélectionné **Self-hosted IR Configuration Manager (Gestionnaire de configuration de l’IR auto-hébergé)**  > **Accès à distance à partir de l’intranet**), vous recevez l’erreur suivante :

« Les paramètres d’accès distant ne sont pas valides. Échec de la vérification d’identité pour le message sortant. L’identité DNS attendue sur le point de terminaison distant était « abc.microsoft.com », mais le DNS fourni pour le point de terminaison distant revendique « microsoft.com ». S’il s’agit d’un point de terminaison distant valable, vous pouvez résoudre le problème en spécifiant explicitement l’identité DNS « microsoft.com » pour la propriété d’identité de l’élément EndpointAddress lors de la création du proxy du canal. »

Dans l’exemple précédent, « microsoft.com » est ajouté au certificat choisi.

#### <a name="cause"></a>Cause

Il s’agit d’un problème connu dans Windows Communication Foundation (WCF). La validation SSL/TLS WCF vérifie uniquement le dernier DNSName dans le champ de l’**autre nom de l’objet** (SAN). 

#### <a name="resolution"></a>Résolution

Un certificat générique est pris en charge dans l’IR auto-hébergé Azure Data Factory v2. Ce problème se produit généralement parce que le certificat SSL n’est pas correct. Le dernier nom DNS dans le SAN doit être valide. 

Pour vérifier et corriger le nom DNS, procédez comme suit : 

1. Ouvrez la console de gestion.
1. Sous **Détails du certificat**, vérifiez la valeur dans les zones **Objet** et **Autre nom de l’objet**. Par exemple, « DNS Name = microsoft.com.com » n’est pas un nom valide.
1. Contactez la société émettrice du certificat pour que les noms DNS incorrects soient supprimés.

### <a name="concurrent-jobs-limit-issue"></a>Problème lié à la limite de tâches simultanées

#### <a name="symptoms"></a>Symptômes

Lorsque vous essayez d’augmenter la limite de travaux simultanés à partir de l’interface Azure Data Factory, le processus se bloque avec l’état *Mise à jour*.

Exemple de scénario : la valeur maximale de travaux simultanés est actuellement définie sur 24 et vous souhaitez augmenter le nombre afin que les travaux puissent s’exécuter plus rapidement. La valeur minimale que vous pouvez entrer est 3 et la valeur maximale est 32. Vous augmentez la valeur de 24 à 32, puis vous sélectionnez le bouton **Mettre à jour**. Le processus est bloqué avec l’état *Mise à jour*, comme indiqué dans la capture d’écran suivante. Vous actualisez la page et la valeur 24 est toujours affichée. Elle n’a pas été mise à jour sur 32 comme prévu.

![Capture d’écran du volet Nœuds du runtime d’intégration, affichant le processus bloqué sur l’état « Mise à jour ».](media/self-hosted-integration-runtime-troubleshoot-guide/updating-status.png)

#### <a name="cause"></a>Cause

La limite du nombre de travaux simultanés dépend de la mémoire et du cœur logique de l’ordinateur. Essayez d’ajuster la valeur vers le bas jusqu’à une valeur telle que 24, puis affichez le résultat.

> [!TIP] 
> - Pour en savoir plus sur le nombre de cœurs logiques et pour déterminer le nombre de cœurs logiques de votre ordinateur, consultez [Quatre façons de trouver le nombre de cœurs dans votre UC sur Windows 10](https://www.top-password.com/blog/find-number-of-cores-in-your-cpu-on-windows-10/).
> - Pour savoir comment calculer le fichier math.log, accédez à la [calculatrice de logarithme](https://www.rapidtables.com/calc/math/Log_Calculator.html).


### <a name="self-hosted-ir-high-availability-ha-ssl-certificate-issue"></a>Problème de certificat SSL à haute disponibilité de l’IR auto-hébergé

#### <a name="symptoms"></a>Symptômes

Le nœud Worker de l’IR auto-hébergé a signalé l’erreur suivante :

«Échec de l’extraction des états partagés du nœud principal net.tcp://abc.cloud.corp.Microsoft.com:8060/ExternalService.svc/. ID d’activité : XXXXX Échec de la génération de la chaîne de certificat X.509 CN=abc.cloud.corp.Microsoft.com, OU=test, O=Microsoft. Le certificat qui a été utilisé est doté d'une chaîne d'approbation impossible à vérifier. Remplacez ce certificat ou modifiez l'élément certificateValidationMode. « La fonction de révocation n’a pas pu vérifier la révocation car le serveur de révocation était déconnecté ».

#### <a name="cause"></a>Cause

Lorsque vous traitez des cas liés à une négociation SSL/TLS, vous pouvez rencontrer certains problèmes liés à la vérification de la chaîne de certificats. 

#### <a name="resolution"></a>Résolution

- Voici un moyen rapide et intuitif de résoudre les échecs de génération de chaîne de certificats X.509 :
 
    1. Exportez le certificat, qui doit être vérifié. Pour ce faire, procédez comme suit :
    
       a. Dans Windows, sélectionnez **Démarrer**, commencez à taper **Certificats**, puis sélectionnez **Gérer les certificats d’ordinateur**.
       
       b. Dans l’Explorateur de fichiers, dans le volet gauche, recherchez le certificat que vous souhaitez vérifier, cliquez dessus avec le bouton droit, puis sélectionnez **Toutes les tâches** > **Exporter**.
    
        ![Capture d’écran du contrôle « Toutes les tâches » > « Exporter » pour un certificat dans le volet « Gérer les certificats d’ordinateur ».](media/self-hosted-integration-runtime-troubleshoot-guide/export-tasks.png)

    2. Copiez le certificat exporté sur la machine cliente. 
    3. Côté client, dans une fenêtre d’invite de commandes, exécutez la commande suivante. Veillez à remplacer *\<certificate path>* et *\<output txt file path>* par les chemins réels.
    
        ```
        Certutil -verify -urlfetch    <certificate path>   >     <output txt file path> 
        ```

        Par exemple :

        ```
        Certutil -verify -urlfetch c:\users\test\desktop\servercert02.cer > c:\users\test\desktop\Certinfo.txt
        ```
    4. Recherchez les erreurs dans le fichier TXT de sortie. Le résumé des erreurs se trouve à la fin du fichier TXT.

        Par exemple : 

        ![Capture d’écran d’un résumé des erreurs à la fin du fichier TXT.](media/self-hosted-integration-runtime-troubleshoot-guide/error-summary.png)

        Si aucune erreur ne s’affiche à la fin du fichier journal, comme indiqué dans la capture d’écran suivante, vous pouvez considérer que la chaîne de certificats a été générée correctement sur l’ordinateur client.
        
        ![Capture d’écran d’un fichier journal qui n’indique aucune erreur.](media/self-hosted-integration-runtime-troubleshoot-guide/log-file.png)      

- Si une extension de nom de fichier AIA (Authority Information Access), CDP (CRL Distribution Point) ou OCSP (Online Certificate Status Protocol) est configurée dans le fichier de certificat, vous pouvez la vérifier de manière plus intuitive :
 
    1. Pour plus d’informations, consultez les détails du certificat, comme indiqué dans la capture d’écran suivante :
    
        ![Capture d’écran des détails du certificat.](media/self-hosted-integration-runtime-troubleshoot-guide/certificate-detail.png)
    
    1. Exécutez la commande suivante. Veillez à remplacer *\<certificate path>* par le chemin d’accès réel du certificat.
    
        ```
          Certutil   -URL    <certificate path> 
        ```
    
        L’outil de récupération d’URL s’ouvre. 
        
    1. Pour vérifier les certificats avec les extensions de nom de fichier AIA, CDP et OCSP, sélectionnez **Récupérer**.

        ![Capture d’écran de l’outil de récupération d’URL et du bouton Récupérer.](media/self-hosted-integration-runtime-troubleshoot-guide/retrieval-button.png)
 
        Vous avez généré la chaîne de certificats correctement si l’état du certificat AIA est *Vérifié* et que l’état du certificat CDP ou OCSP est *Vérifié*.

        Si vous échouez lorsque vous tentez de récupérer AIA ou CDP, contactez l’équipe réseau pour préparer la machine cliente à se connecter à l’URL cible. Cela suffit si le chemin d’accès HTTP ou le chemin d’accès LDAP (Lightweight Directory Access Protocol) peuvent être vérifiés.

### <a name="self-hosted-ir-could-not-load-file-or-assembly"></a>L’IR auto-hébergé n’a pas pu télécharger le fichier ou l’assembly

#### <a name="symptoms"></a>Symptômes

Vous obtenez le message d'erreur suivant :

« Impossible de charger le fichier ou l’assembly 'XXXXXXXXXXXXXXXX, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' ou une de ses dépendances. Le système ne peut pas trouver le fichier spécifié. ID d’activité : 92693b45-b4bf-4fc8-89da-2d3dc56f27c3"
 
Voici un message d’erreur plus spécifique : 

« Impossible de charger le fichier ou l’assembly 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' ou une de ses dépendances. Le système ne peut pas trouver le fichier spécifié. ID d’activité : 92693b45-b4bf-4fc8-89da-2d3dc56f27c3"

#### <a name="cause"></a>Cause

Dans Process Monitor, vous pouvez afficher les résultats suivants :

[![Capture d’écran de la liste de chemins dans Process Monitor.](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png)](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png#lightbox)

> [!TIP] 
> Dans Process Monitor, vous pouvez définir des filtres comme indiqué dans la capture d’écran suivante.
>
> Le message d’erreur précédent indique que la DLL System.ValueTuple ne se trouve pas dans le dossier associé *GAC* ou dans *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway*, ou dans le dossier *C:\Program Files\Microsoft Integration Runtime\4.0\Shared*.
>
> Fondamentalement, le processus charge la DLL d’abord à partir du dossier *GAC*, puis à partir du dossier *Partagé* et enfin à partir du dossier *Passerelle*. Par conséquent, vous pouvez charger la DLL à partir de n’importe quel chemin d’accès utile.

<br>

![Capture d’écran de la page « Filtre Process Monitor » répertoriant les filtres pour la DLL.](media/self-hosted-integration-runtime-troubleshoot-guide/set-filters.png)

#### <a name="resolution"></a>Résolution

Le fichier *System.ValueTuple.dll* se trouve dans le dossier *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway\DataScan*. Pour résoudre le problème, copiez le fichier *System.ValueTuple.dll* dans le dossier *C:\Program Files\Microsoft Integration Runtime\4.0\Gateway*.

Vous pouvez utiliser la même méthode pour résoudre d’autres problèmes de fichier ou d’assembly manquant.

#### <a name="more-information-about-this-issue"></a>Plus d'informations sur ce problème

La raison pour laquelle vous voyez le fichier *System.ValueTuple.dll* sous *%windir%\Microsoft.NET\assembly* et *%windir%\assembly* est qu’il s’agit d’un comportement .NET. 

Dans l’erreur suivante, vous pouvez voir clairement que l’assembly *System.ValueTuple* est manquant. Ce problème se produit lorsque l’application tente de vérifier l’assembly *System.ValueTuple.dll*.
 
"\<LogProperties>\<ErrorInfo>[{"Code":0,"Message":"L’initialiseur de type pour 'Npgsql.PoolManager' a généré une exception.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.TypeInitializationException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[{"Code":0,"Message":"Impossible de charger le fichier ou l’assembly 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' ou une de ses dépendances. Le système ne trouve pas le fichier spécifié.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.IO.FileNotFoundException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[]}]}]\</ErrorInfo>\</LogProperties>"
 
Pour plus d’informations sur le GAC, consultez [Global Assembly Cache](https://docs.microsoft.com/dotnet/framework/app-domains/gac).


### <a name="self-hosted-integration-runtime-authentication-key-is-missing"></a>La clé d’authentification du runtime d’intégration auto-hébergé est manquante

#### <a name="symptoms"></a>Symptômes

Le runtime d’intégration auto-hébergé se met soudainement hors connexion sans clé d’authentification et le journal des événements affiche le message d’erreur suivant : 

« La clé d’authentification n’est pas encore affectée »

![Capture d’écran du volet d’événement du runtime d’intégration indiquant que la clé d’authentification n’est pas encore affectée.](media/self-hosted-integration-runtime-troubleshoot-guide/key-missing.png)

#### <a name="cause"></a>Cause

- Le nœud IR auto-hébergé ou l’IR auto-hébergé logique dans le portail Azure a été supprimé.
- Une désinstallation propre a été effectuée.

#### <a name="resolution"></a>Résolution

Si aucune des causes ci-dessus ne s’applique, vous pouvez accéder au dossier *%programdata%\Microsoft\Data Transfer\DataManagementGateway* et vérifier si le fichier *Configurations* a été supprimé. S’il a été supprimé, suivez les instructions de l’article NetWrix [Détecter qui a supprimé un fichier de vos serveurs de fichiers Windows](https://www.netwrix.com/how_to_detect_who_deleted_file.html).

![Capture d’écran du volet Détails du journal des événements pour la vérification du fichier de configuration.](media/self-hosted-integration-runtime-troubleshoot-guide/configurations-file.png)


### <a name="cant-use-self-hosted-ir-to-bridge-two-on-premises-datastores"></a>Impossible d’utiliser l’IR auto-hébergé pour relier deux magasins de données locaux

#### <a name="symptoms"></a>Symptômes

Une fois que vous avez créé les IR auto-hébergés pour les magasins de données source et de destination, connectez-les pour terminer une activité de copie. Si les magasins de données sont configurés dans différents réseaux virtuels, ou si les magasins de données ne peuvent pas comprendre le mécanisme de passerelle, vous recevez l’une des erreurs suivantes : 

* « Le pilote de la source est introuvable dans l’IR de destination »
* « La source n’est pas accessible par l’IR de destination »
 
#### <a name="cause"></a>Cause

L’IR auto-hébergé est conçu comme un nœud central d’une activité de copie, et non un agent client qui doit être installé pour chaque magasin de données.
 
Dans le cas ci-dessus, vous devez créer le service lié pour chaque magasin de données avec le même IR et l’IR doit être en mesure d’accéder aux deux magasins de données via le réseau. Cela n’a pas d’importance si le runtime d’intégration (IR) est installé sur le magasin de données source ou de destination, ou sur un troisième ordinateur. Si deux services liés sont créés avec différents IR mais qu’ils sont utilisés dans la même activité de copie, le runtime d’intégration (IR) de destination est utilisé et vous devez installer les pilotes pour les deux magasins de données sur l’ordinateur IR de destination.

#### <a name="resolution"></a>Résolution

Installez les pilotes pour les magasins de données source et de destination sur le runtime d’intégration de destination et assurez-vous qu’il peut accéder au magasin de données source.
 
Si le trafic ne peut pas traverser le réseau entre deux magasins de données (par exemple, s’ils sont configurés en deux réseaux virtuels), vous risquez de ne pas terminer la copie en une seule activité, même si l’IR est installé. Si vous ne pouvez pas terminer la copie en une seule activité, vous pouvez créer deux activités de copie avec deux IR, chacun dans un VENT : 
* Copiez un IR du magasin de données 1 vers le Stockage Blob Azure.
* Copiez l’autre IR à partir du Stockage Blob Azure vers le magasin de données ddatastore 2. 

Cette solution peut simuler la nécessité d’utiliser le runtime d’intégration pour créer un pont qui connecte deux magasins de données déconnectés.


### <a name="credential-sync-issue-causes-credential-loss-from-ha"></a>Un problème de synchronisation des informations d’identification provoque la perte des informations d’identification de haute disponibilité

#### <a name="symptoms"></a>Symptômes

Si les informations d’identification de la source de données « XXXXXXXXXX » sont supprimées du nœud de runtime d’intégration actuel avec la charge utile, vous recevez le message d’erreur suivant :

« Quand vous avez supprimé le service de lien sur le portail Azure, ou la tâche n’a pas la bonne charge utile. Recréez un service de lien avec vos informations d’identification. »

#### <a name="cause"></a>Cause

Votre IR auto-hébergé est intégré en mode Haute disponibilité avec deux nœuds, mais les nœuds ne sont pas dans l’état de synchronisation des informations d’identification. Cela signifie que les informations d’identification stockées dans le nœud de répartiteur ne sont pas synchronisées avec d’autres nœuds Worker. Si un basculement se produit du nœud répartiteur vers le nœud Worker et que les informations d’identification existent uniquement dans le nœud répartiteur précédent, la tâche échoue lorsque vous tentez d’accéder aux informations d’identification et vous pouvez recevoir l’erreur ci-dessus.

#### <a name="resolution"></a>Résolution

La seule façon d’éviter ce problème consiste à s’assurer que les deux nœuds sont dans l’état de synchronisation des informations d’identification. Si elles ne sont pas synchronisées, vous devez réentrer les informations d’identification du nouveau répartiteur.


### <a name="cant-choose-the-certificate-because-the-private-key-is-missing"></a>Impossible de choisir le certificat, car la clé privée est manquante

#### <a name="symptoms"></a>Symptômes

* Vous avez importé un fichier PFX dans le magasin de certificats.
* Lorsque vous avez sélectionné le certificat via l’interface utilisateur du gestionnaire de configuration d’IR, vous avez reçu le message d’erreur suivant :

   « Échec du changement du mode de chiffrement de la communication intranet. Il est probable que le certificat « \<*certificate name*> » n’a pas de clé privée qui prend en charge l’échange de clés ou le processus ne dispose peut-être pas de droits d’accès pour la clé privée. Pour plus d’informations, consultez l’exception interne. »

    ![Capture d’écran du volet Paramètres du gestionnaire de configuration du runtime d’intégration, affichant le message d’erreur « Clé privée manquante ».](media/self-hosted-integration-runtime-troubleshoot-guide/private-key-missing.png)

#### <a name="cause"></a>Cause

- Le compte d’utilisateur dispose d’un niveau de privilège faible et ne peut pas accéder à la clé privée.
- Le certificat a été généré en tant que signature, mais pas en tant qu’échange de clés.

#### <a name="resolution"></a>Résolution

* Pour utiliser l’interface utilisateur, utilisez un compte disposant des privilèges appropriés pour accéder à la clé privée.  
* Importez le certificat en exécutant la commande suivante :
    
    ```
    certutil -importpfx FILENAME.pfx AT_KEYEXCHANGE
    ```

## <a name="self-hosted-ir-setup"></a>Installation du runtime d’intégration IR auto-hébergé

### <a name="integration-runtime-registration-error"></a>Erreur d’inscription du runtime d’intégration 

#### <a name="symptoms"></a>Symptômes

Vous pouvez vouloir exécuter un IR auto-hébergé dans un autre compte pour l’une des raisons suivantes :
- La stratégie d’entreprise interdit le compte de service.
- L’authentification est obligatoire.

Après avoir modifié le compte de service dans le volet Service, vous pouvez constater que le runtime d’intégration s’arrête de fonctionner et que vous obtenez le message d’erreur suivant :

« Le nœud Integration Runtime (auto-hébergé) a rencontré une erreur durant l'inscription. Impossible de se connecter au service hôte Integration Runtime (auto-hébergé). »

![Capture d’écran de la fenêtre de gestionnaire de configuration du runtime d’intégration, indiquant une erreur d’inscription de l’IR.](media/self-hosted-integration-runtime-troubleshoot-guide/ir-registration-error.png)

#### <a name="cause"></a>Cause

De nombreuses ressources sont accordées uniquement au compte de service. Lorsque vous remplacez le compte de service par un autre compte, les autorisations de toutes les ressources dépendantes restent les mêmes.

#### <a name="resolution"></a>Résolution

Accédez au journal des événements du runtime d’intégration pour vérifier l’erreur.

![Capture d’écran du journal des événements IR, indiquant qu’une erreur d’exécution s’est produite.](media/self-hosted-integration-runtime-troubleshoot-guide/ir-event-log.png)

* Si l’erreur dans le journal des événements est « UnauthorizedAccessException », procédez comme suit :

    1. Vérifiez le compte de service d’ouverture de session *DIAHostService* dans le panneau de service Windows.

        ![Capture d’écran du volet des propriétés du compte de service d’ouverture de session.](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

    1. Vérifiez si le compte de service d’ouverture de session dispose de l’autorisation de lecture/écriture sur le dossier  *%programdata%\Microsoft\DataTransfer\DataManagementGateway*.

        - Par défaut, si le compte d’ouverture de session du service n’a pas été modifié, il doit avoir des autorisations de lecture/écriture.

            ![Capture d’écran du panneau Autorisations du service.](media/self-hosted-integration-runtime-troubleshoot-guide/service-permission.png)

        - Si vous avez modifié le compte d’ouverture de session du service, atténuez le problème en procédant comme suit :
 
            a. Effectuez une désinstallation propre du runtime d’intégration auto-hébergé actuel.   
            b. Installez le runtime d’intégration auto-hébergé.  
            c. Modifiez le compte de service en procédant comme suit :  

             i. Accédez au dossier d’installation du runtime d’intégration auto-hébergé, puis accédez au dossier *Microsoft Integration Runtime\4.0\Shared*.  
             ii. Ouvrez une fenêtre Invite de commandes en utilisant des privilèges élevés. Remplacez *\<user>* et *\<password>* par vos propres nom d’utilisateur et mot de passe, puis exécutez la commande suivante :   
                `dmgcmd.exe -SwitchServiceAccount "<user>" "<password>"`  
             iii. Si vous souhaitez passer au compte LocalSystem, veillez à utiliser un format correct pour ce compte : `dmgcmd.exe -SwitchServiceAccount "NT Authority\System" ""`  
                N’utilisez *pas* ce format : `dmgcmd.exe -SwitchServiceAccount "LocalSystem" ""`     
             iv. À l’inverse, étant donné que le système local dispose de privilèges plus élevés que l’administrateur, vous pouvez également le modifier directement en « Services ».  
             v. Vous pouvez utiliser un utilisateur local/de domaine pour le compte d’ouverture de session du service IR.            

            d. Inscrivez le runtime d’intégration.

* Si l’erreur « Échec du démarrage du service Integration Runtime Service (DIAHostService). Vérifiez que vous disposez de privilèges suffisants pour démarrer les services système. » s’affiche, procédez comme suit :

    1. Vérifiez le compte de service d’ouverture de session *DIAHostService* dans le panneau de service Windows.
    
        ![Capture d’écran du volet « Ouvrir une session » pour le compte de service.](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

    1. Vérifiez si le compte de service d’ouverture de session dispose de l’autorisation **Ouvrir une session en tant que service** pour démarrer le service Windows :

        ![Capture d’écran du volet des propriétés « Ouvrir une session en tant que service ».](media/self-hosted-integration-runtime-troubleshoot-guide/logon-as-service.png)

#### <a name="more-information"></a>Informations complémentaires

Si aucun des deux modèles de résolution précédents ne s’applique dans votre cas, essayez de collecter les journaux des événements Windows suivants : 
- Journaux des applications et des services > Runtime d’intégration
- Journaux d’activité Windows > Application

### <a name="cant-find-the-register-button-to-register-a-self-hosted-ir"></a>Impossible de trouver le bouton Inscrire pour inscrire un IR auto-hébergé    

#### <a name="symptoms"></a>Symptômes

Lorsque vous inscrivez un IR auto-hébergé, le bouton **Inscrire** ne s’affiche pas dans le volet du gestionnaire de configuration.

![Capture d’écran du volet du gestionnaire de configuration, affichant un message indiquant que le nœud du runtime d’intégration n’est pas inscrit.](media/self-hosted-integration-runtime-troubleshoot-guide/no-register-button.png)

#### <a name="cause"></a>Cause

Depuis la sortie d’Integration Runtime 3.0, le bouton **Inscrire** des nœuds de runtime d’intégration existants a été supprimé pour proposer un environnement plus clair et plus sécurisé. Si un nœud a été inscrit sur un IR (qu’il soit en ligne ou non), réinscrivez-le dans un autre IR en désinstallant le nœud précédent, puis installez et inscrivez le nœud.

#### <a name="resolution"></a>Résolution

1. Dans le Panneau de configuration, désinstallez l’IR existant.

    > [!IMPORTANT] 
    > Dans le processus suivant, sélectionnez **Oui**. Ne conservez pas les données pendant le processus de désinstallation.

    ![Capture d’écran du bouton « Oui » pour la suppression de toutes les données utilisateur du runtime d’intégration.](media/self-hosted-integration-runtime-troubleshoot-guide/delete-data.png)

1. Si vous n’avez pas le fichier MSI du programme d’installation du runtime d’intégration, accédez au [Centre de téléchargement](https://www.microsoft.com/en-sg/download/details.aspx?id=39717) pour télécharger l’IR le plus récent.
1. Installez le fichier MSI et inscrivez le runtime d’intégration.


### <a name="unable-to-register-the-self-hosted-ir-because-of-localhost"></a>Impossible d’inscrire l’IR auto-hébergé en raison du localhost    

#### <a name="symptoms"></a>Symptômes

Vous ne pouvez pas inscrire l’IR auto-hébergé sur un nouvel ordinateur lorsque vous utilisez get_LoopbackIpOrName.

**Débogage :** Une erreur d'exécution s'est produite.
L’initialiseur de type pour « Microsoft. DataTransfer. DIAgentHost. DataSourceCache » a levé une exception.
Une erreur non récupérable s’est produite lors d’une recherche de base de données.
 
**Détail de l’exception :** System.TypeInitializationException : L’initialiseur de type pour « Microsoft. DataTransfer. DIAgentHost. DataSourceCache » a levé une exception. ---> System.Net.Sockets.SocketException: Une erreur non récupérable s’est produite lors d’une recherche de base de données sur System net.DNS.GetAddrInfo (String name).

#### <a name="cause"></a>Cause

Le problème se produit généralement lors de la résolution de l’hôte local.

#### <a name="resolution"></a>Résolution

Utilisez l’adresse IP localhost 127.0.0.1 pour héberger le fichier et résoudre le problème.

### <a name="self-hosted-setup-failed"></a>Échec de l’installation auto-hébergée    

#### <a name="symptoms"></a>Symptômes

Vous ne pouvez pas désinstaller un runtime d’intégration ni installer un nouveau runtime d’intégration existant ou mettre à niveau un IR existant vers un nouveau runtime d’intégration.

#### <a name="cause"></a>Cause

L’installation du runtime d’intégration dépend du service Windows Installer. Vous pouvez rencontrer des problèmes d’installation pour les raisons suivantes :
- Espace disque disponible insuffisant.
- Autorisations manquantes.
- Le service Windows NT est verrouillé.
- Utilisation du processeur trop élevée.
- Le fichier MSI est hébergé dans un emplacement réseau lent.
- Certains registres ou fichiers système ont été touchés par inadvertance.

### <a name="the-ir-service-account-failed-to-fetch-certificate-access"></a>Le compte de service IR n’a pas pu récupérer l’accès au certificat

#### <a name="symptoms"></a>Symptômes

Lorsque vous installez l’IR auto-hébergé via le gestionnaire de configuration Microsoft Integration Runtime, un certificat avec une autorité de certification approuvée est généré. Le certificat n’a pas pu être appliqué pour chiffrer les communications entre deux nœuds et le message d’erreur suivant s’affiche : 

« Échec du changement du mode de chiffrement de la communication intranet : Impossible d’accorder au compte du service Integration Runtime l’accès au certificat « \<*certificate name*> ». Code d’erreur 103 »

![Capture d’écran affichant le message d’erreur «... Impossible d’accorder au compte du service Integration Runtime l’accès au certificat ».](media/self-hosted-integration-runtime-troubleshoot-guide/integration-runtime-service-account-certificate-error.png)

#### <a name="cause"></a>Cause

Le certificat utilise le fournisseur de stockage de clés (KSP), qui n’est pas encore pris en charge. À ce jour, l’IR auto-hébergé ne prend en charge que le stockage du fournisseur de services de chiffrement (CSP).

#### <a name="resolution"></a>Résolution

Nous vous recommandons d’utiliser des certificats CSP dans ce cas.

**Solution 1** 

Pour importer le certificat, exécutez la commande suivante :

`Certutil.exe -CSP "CSP or KSP" -ImportPFX FILENAME.pfx`

![Capture d’écran de la commande certutil pour l’importation du certificat.](media/self-hosted-integration-runtime-troubleshoot-guide/use-certutil.png)

**Solution 2** 

Pour convertir le certificat, exécutez les commandes suivantes :

`openssl pkcs12 -in .\xxxx.pfx -out .\xxxx_new.pem -password pass: <EnterPassword>`
`openssl pkcs12 -export -in .\xxxx_new.pem -out xxxx_new.pfx`

Conversion avant et après :

![Capture d’écran du résultat avant la conversion du certificat.](media/self-hosted-integration-runtime-troubleshoot-guide/before-certificate-change.png)

![Capture d’écran du résultat après la conversion du certificat.](media/self-hosted-integration-runtime-troubleshoot-guide/after-certificate-change.png)

### <a name="self-hosted-integration-runtime-version-5x"></a>Version 5.x du runtime d’intégration auto-hébergé
Pour la mise à niveau vers la version 5.x du runtime d’intégration auto-hébergé Azure Data Factory, nous avons besoin du **runtime .NET Framework 4.7.2** ou version ultérieure. Sur la page de téléchargement, vous trouverez des liens de téléchargement pour la dernière version 4.x et les deux dernières versions 5.x. 

Pour les clients Azure Data Factory v2 :
- Si la mise à jour automatique est activée et que vous avez déjà mis à niveau le runtime .NET Framework vers la version 4.7.2 ou ultérieure, le runtime d’intégration auto-hébergé sera automatiquement mis à niveau vers la version 5.x la plus récente.
- Si la mise à jour automatique est activée et que vous n’avez pas mis à niveau le runtime .NET Framework vers la version 4.7.2 ou ultérieure, le runtime d’intégration auto-hébergé ne sera pas automatiquement mis à niveau vers la version 5.x la plus récente. Le runtime d’intégration auto-hébergé conservera la version 4.x actuelle. Vous pouvez voir un avertissement pour la mise à niveau du runtime .NET Framework dans le portail et le client du runtime d’intégration auto-hébergé.
- Si la mise à jour automatique est désactivée et que vous avez déjà mis à niveau le runtime .NET Framework vers la version 4.7.2 ou ultérieure, vous pouvez télécharger manuellement la version 5.x la plus récente et l’installer sur votre ordinateur.
- Si la mise à jour automatique est désactivée et que vous n’avez pas mis à niveau le runtime .NET Framework vers la version 4.7.2 ou ultérieure. Lorsque vous essayez d’installer manuellement le runtime d’intégration auto-hébergé 5.x et d’inscrire la clé, vous devez d’abord mettre à niveau votre runtime .NET Framework.


Pour les clients Azure Data Factory v1 :
- Le runtime d’intégration auto-hébergé 5.X ne prend pas en charge Azure Data Factory  v1.
- Le runtime d’intégration auto-hébergé sera automatiquement mis à niveau vers la version 4.x la plus récente. Et la version la plus récente de 4.x n’expire pas. 
- Si vous essayez d’installer manuellement le runtime d’intégration auto-hébergé 5.x et d’inscrire la clé, vous êtes informé que le runtime d’intégration auto-hébergé 5.x ne prend pas en charge Azure Data Factory v1.


## <a name="self-hosted-ir-connectivity-issues"></a>Problème de connexion de l’IR auto-hébergé

### <a name="self-hosted-integration-runtime-cant-connect-to-the-cloud-service"></a>Le runtime d’intégration auto-hébergé ne peut pas se connecter au service cloud

#### <a name="symptoms"></a>Symptômes

Lorsque vous tentez d’inscrire le runtime d’intégration auto-hébergé, le gestionnaire de configuration affiche le message d’erreur suivant :

« Le nœud Integration Runtime (auto-hébergé) a rencontré une erreur durant l’inscription. »

![Capture d’écran du message « Le nœud Integration Runtime (auto-hébergé) a rencontré une erreur durant l’inscription. »](media/self-hosted-integration-runtime-troubleshoot-guide/unable-to-connect-to-cloud-service.png)

#### <a name="cause"></a>Cause 

L’IR auto-hébergé ne peut pas se connecter au backend du service Azure Data Factory. Ce problème est généralement dû à des paramètres réseau dans le pare-feu.

#### <a name="resolution"></a>Résolution

1. Vérifiez si le service du runtime d’intégration est en cours d’exécution. Si tel est le cas, passez à l’étape 2.
    
   ![Capture d’écran montrant que le service IR auto-hébergé est en cours d’exécution.](media/self-hosted-integration-runtime-troubleshoot-guide/integration-runtime-service-running-status.png)
    
1. Si aucun proxy n’est configuré sur l’IR auto-hébergé (qui est le paramètre par défaut), exécutez la commande PowerShell suivante sur l’ordinateur sur lequel est installé le runtime d’intégration auto-hébergé :

    ```powershell
    (New-Object System.Net.WebClient).DownloadString("https://wu2.frontend.clouddatahub.net/")
    ```
        
   > [!NOTE]     
   > L’URL du service peut varier en fonction de l’emplacement de votre instance Data Factory. L’URL du service se trouve sous **UI ADF** > **Connexions** > **Runtimes d’intégration** >  **Modifier l’IR auto-hébergé** > **Nœuds** > **Afficher les URL du service**.
            
    Voici la réponse attendue :
            
    ![Capture d’écran de la réponse de la commande PowerShell.](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)
            
1. Si vous ne recevez pas la réponse que vous attendiez, utilisez l’une des méthodes suivantes, selon le cas :
            
    * Si vous recevez le message « le nom distant n’a pas pu être résolu », il existe un problème DNS (Domain Name System). Contactez votre équipe réseau pour résoudre ce problème.
    * Si vous recevez un message « Certificat SSL/TLS non approuvé », [vérifiez si le certificat](https://wu2.frontend.clouddatahub.net/) est approuvé sur l’ordinateur, puis installez le certificat public à l’aide du gestionnaire de certificats. Cette action doit atténuer le problème.
    * Accédez à **Windows** > **Observateur d’événements (journaux)**  > **Journaux des applications et des services** > **Runtime d’intégration** et recherchez les défaillances provoquées par le DNS, une règle de pare-feu ou des paramètres réseau d’entreprise. Si vous trouvez une de ces défaillances, forcez la fermeture de la connexion. Étant donné que chaque entreprise a personnalisé ses propres paramètres réseau, contactez votre équipe réseau pour résoudre ces problèmes.

1. Si « proxy » a été configuré sur le runtime d’intégration auto-hébergé, vérifiez que votre serveur proxy est en mesure d’accéder à notre point de terminaison de service. Pour obtenir un exemple de commande, consultez [PowerShell, les requêtes Web et les proxys](https://stackoverflow.com/questions/571429/powershell-web-requests-and-proxies).    
                
    ```powershell
    $user = $env:username
    $webproxy = (get-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet
    Settings').ProxyServer
    $pwd = Read-Host "Password?" -assecurestring
    $proxy = new-object System.Net.WebProxy
    $proxy.Address = $webproxy
    $account = new-object System.Net.NetworkCredential($user,[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pwd)), "")
    $proxy.credentials = $account
    $url = "https://wu2.frontend.clouddatahub.net/"
    $wc = new-object system.net.WebClient
    $wc.proxy = $proxy
    $webpage = $wc.DownloadData($url)
    $string = [System.Text.Encoding]::ASCII.GetString($webpage)
    $string
    ```

Voici la réponse attendue :
            
![Capture d’écran de la réponse attendue de la commande PowerShell.](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)

> [!NOTE] 
> Considérations liées au proxy :
> * Vérifiez si le serveur proxy doit être placé dans la liste des destinataires approuvés. Dans ce cas, assurez-vous que [ces domaines](./data-movement-security-considerations.md#firewall-requirements-for-on-premisesprivate-network) figurent dans la liste des destinataires approuvés.
> * Vérifiez si le certificat TLS/SSL « wu2.frontend.clouddatahub.net/ » est approuvé sur le serveur proxy.
> * Si vous utilisez l’authentification Active Directory sur le proxy, remplacez le compte de service par le compte d’utilisateur qui peut accéder au proxy en tant que « Service Integration Runtime ».

### <a name="error-message-self-hosted-integration-runtime-nodelogical-self-hosted-ir-is-in-inactive-running-limited-state"></a>Message d’erreur : Le nœud du runtime d’intégration auto-hébergé /l’IR auto-hébergé logique présente l’état Inactif/ « En cours d’exécution (limité) »

#### <a name="cause"></a>Cause 

Le nœud Runtime intégré auto-hébergé peut avoir un état **inactif**, comme indiqué dans la capture d’écran suivante :

![Capture d’écran du nœud Runtime intégré auto-hébergé avec état inactif](media/self-hosted-integration-runtime-troubleshoot-guide/inactive-self-hosted-ir-node.png)

Ce comportement se produit lorsque les nœuds ne peuvent pas communiquer entre eux.

#### <a name="resolution"></a>Résolution

1. Connectez-vous à la machine virtuelle hébergée sur un nœud. Sous **Journaux des applications et des services** > **Runtime d’intégration**, ouvrez l’Observateur d’événements, puis filtrez tous les journaux d’erreurs.

1. Vérifiez si un journal des erreurs contient l’erreur suivante : 
    
    ```
    System.ServiceModel.EndpointNotFoundException: Could not connect to net.tcp://xxxxxxx.bwld.com:8060/ExternalService.svc/WorkerManager. The connection attempt lasted for a time span of 00:00:00.9940994. TCP error code 10061: No connection could be made because the target machine actively refused it 10.2.4.10:8060. 
    System.Net.Sockets.SocketException: No connection could be made because the target machine actively refused it. 
    10.2.4.10:8060
    at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
    at System.Net.Sockets.Socket.Connect(EndPoint remoteEP)
    at System.ServiceModel.Channels.SocketConnectionInitiator.Connect(Uri uri, TimeSpan timeout)
    ```
       
1. Si vous voyez cette erreur, tapez la commande suivante dans une fenêtre d’invite de commandes. 

   ```
   telnet 10.2.4.10 8060
   ```
   
1. Si vous recevez l’erreur de ligne de commande « Impossible d’ouvrir la connexion à l’hôte » présentée dans la capture d’écran suivante, contactez votre service informatique pour obtenir de l’aide afin de résoudre ce problème. Lorsque vous parvenez à utiliser Telnet, contactez le support Microsoft si vous rencontrez toujours des problèmes pour l’état du nœud IR.
        
   ![Capture d’écran de l’erreur de ligne de commande « Impossible d’ouvrir la connexion à l’hôte ».](media/self-hosted-integration-runtime-troubleshoot-guide/command-line-error.png)
        
1. Vérifiez si le journal des erreurs contient l’entrée suivante :

    ```
    Error log: Cannot connect to worker manager: net.tcp://xxxxxx:8060/ExternalService.svc/ No DNS entries exist for host azranlcir01r1. No such host is known Exception detail: System.ServiceModel.EndpointNotFoundException: No DNS entries exist for host xxxxx. ---> System.Net.Sockets.SocketException: No such host is known at System.Net.Dns.GetAddrInfo(String name) at System.Net.Dns.InternalGetHostByName(String hostName, Boolean includeIPv6) at System.Net.Dns.GetHostEntry(String hostNameOrAddress) at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri) --- End of inner exception stack trace --- Server stack trace: at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri)
    ```
    
1. Pour résoudre le problème, essayez l’une des méthodes suivantes, voire les deux :
    - Placez tous les nœuds dans le même domaine.
    - Ajoutez l’adresse IP au mappage de l’hôte dans tous les fichiers hôtes de la machine virtuelle hébergée.

### <a name="connectivity-issue-between-the-self-hosted-ir-and-your-data-factory-instance-or-the-self-hosted-ir-and-the-data-source-or-sink"></a>Problème de connectivité entre l’IR auto-hébergé et votre instance Data Factory ou l’IR auto-hébergé et la source de données ou le récepteur

Pour résoudre le problème de connectivité réseau, vous devez savoir comment collecter la trace réseau, comprendre comment l’utiliser et [analyser la trace Microsoft Network Monitor (Netmon)](#analyze-the-netmon-trace) avant d’appliquer les outils Netmon à de vrais cas à partir de l’IR auto-hébergé.

#### <a name="symptoms"></a>Symptômes

Vous devrez peut-être parfois résoudre certains problèmes de connectivité entre le runtime d’intégration auto-hébergé et votre instance Data Factory, comme indiqué dans la capture d’écran suivante, ou entre l’IR auto-hébergé et la source de données ou le récepteur. 

![Capture d’écran d’un message « Échec de la requête HTTP traitée »](media/self-hosted-integration-runtime-troubleshoot-guide/http-request-error.png)

Dans les deux cas, vous pouvez rencontrer les erreurs suivantes :

* « Échec de la copie avec l’erreur : error:Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Impossible de se connecter à SQL Server : « Adresse IP »

* « Une ou plusieurs erreurs se sont produites. An error occurred while sending the request. Le serveur a clos la connexion sous-jacente : Une erreur inattendue s’est produite lors de la réception. Impossible de lire les données de la connexion de transport : une connexion existante a dû être fermée par l’hôte distant. Une connexion existante a été fermée de force par l’ID d’activité de l’hôte distant ».

#### <a name="resolution"></a>Résolution

Lorsque vous rencontrez les erreurs précédentes, corrigez-les en suivant les instructions de cette section.

- Collectez une trace Netmon pour analyse : 

    1. Vous pouvez définir le filtre pour voir toute réinitialisation à partir du serveur vers le côté client. Dans l’exemple de capture d’écran ci-dessous, vous pouvez voir que le côté serveur est le serveur Data Factory.

        ![Capture d’écran du serveur Data Factory.](media/self-hosted-integration-runtime-troubleshoot-guide/data-factory-server.png)

    1. Lorsque vous obtenez le package de réinitialisation, vous pouvez trouver la conversation en suivant le protocole TCP (Transmission Control Protocol).

        ![Capture d’écran de la conversation TCP.](media/self-hosted-integration-runtime-troubleshoot-guide/find-conversation.png)

    1. Obtenez la conversation entre le client et le serveur Data Factory ci-dessous en supprimant le filtre.

        ![Capture d’écran des détails de la conversation.](media/self-hosted-integration-runtime-troubleshoot-guide/get-conversation.png)

- Une analyse de la trace Netmon que vous avez collectée montre que la durée de vie (TTL) est 64. Selon les valeurs mentionnées dans l’article [Principes de base de la durée de vie (TTL) et des limites de tronçons](https://packetpushers.net/ip-time-to-live-and-hop-limit-basics/), extraites dans la liste suivante, vous pouvez voir qu’il s’agit du système Linux qui réinitialise le package et provoque la déconnexion.

    Les valeurs de durée de vie et de limite de tronçon par défaut varient entre différents systèmes d’exploitation, comme indiqué ici :
    - Noyau Linux 2.4 (circa 2001) : 255 pour TCP, UDP (User Datagram Protocol) et ICMP (Internet Control Message Protocol)
    - Noyau Linux 4.10 (2015) : 64 pour TCP, UDP et ICMP
    - Windows XP (2001) : 128 pour TCP, UDP et ICMP
    - Windows 10 (2015) : 128 pour TCP, UDP et ICMP
    - Windows Server 2008 : 128 pour TCP, UDP et ICMP
    - Windows Server 2019 (2018) : 128 pour TCP, UDP et ICMP
    - macOS (2001) : 64 pour TCP, UDP et ICMP

    ![Capture d’écran montrant une valeur TTL de 61.](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-61.png)
    
    Dans l’exemple précédent, la durée de vie est indiquée sous la forme 61 au lieu de 64, car lorsque le package réseau atteint sa destination, il doit traverser différents tronçons, tels que des routeurs ou des périphériques réseau. Le nombre de routeurs ou périphériques réseau est déduit pour produire la durée de vie (TTL) finale.
    
    Dans ce cas, vous pouvez voir qu’une réinitialisation peut être envoyée à partir du système Linux avec TTL 64.

-  Pour confirmer l’origine de la réinitialisation de l’appareil, vérifiez le quatrième tronçon de l’IR auto-hébergé.
 
    *Package réseau provenant du système Linux A avec TTL 64 -> B TTL 64 moins 1 = 63 -> C TTL 63 moins 1 = 62 -> TTL 62 moins 1 = 61 Runtime d’intégration auto-hébergé*

- Dans une situation idéale, le nombre de tronçons TTL est de 128, ce qui signifie que le système d’exploitation Windows exécute votre instance Data Factory. Comme indiqué dans l’exemple suivant, *128 moins 107 = 21 tronçons*, ce qui signifie que 21 tronçons pour le package ont été envoyés de l’instance Data Factory vers l’IR auto-hébergé pendant la négociation TCP 3.
 
    ![Capture d’écran montrant une valeur TTL de 107.](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-107.png)

    Par conséquent, vous devez faire appel à l’équipe réseau pour vérifier à quoi correspond le quatrième tronçon provenant de l’IR auto-hébergé. S’il s’agit du pare-feu, comme avec le système Linux, consultez tous les journaux pour déterminer la raison pour laquelle ce périphérique réinitialise le package après la négociation TCP 3. 
    
    Si vous n’êtes pas sûr de l’emplacement à examiner, essayez d’accéder à la trace Netmon à partir de l’IR auto-hébergé et du pare-feu pendant la période problématique. Cette approche vous permet de déterminer quel appareil a pu réinitialiser le package et a provoqué la déconnexion. Dans ce cas, vous devez également encourager votre équipe réseau à avancer.

### <a name="analyze-the-netmon-trace"></a>Analyser la trace NetMon

> [!NOTE] 
> Les instructions suivantes s’appliquent à la trace Netmon. Étant donné que la trace Netmon n’est pas prise en charge, vous pouvez utiliser Wireshark à cet effet.

Lorsque vous essayez d’utiliser Telnet **8.8.8.8 888** avec la trace Netmon collectée, vous devez voir la trace dans les captures d’écran suivantes :

![Capture d’écran montrant le message d’erreur « Impossible d’ouvrir la connexion au serveur hôte sur le port 888 ».](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-1.png)

![Capture d’écran montrant une description de la trace Netmon.](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-2.png)
 

L’image précédente montre que vous n’avez pas pu établir la connexion TCP côté serveur **8.8.8.8** sur le port **888**, de sorte que vous y voyez deux packages **SynReTransmit** supplémentaires. Étant donné que la source **SELF-HOST2** ne parvient pas à se connecter à **8.8.8.8** avec le premier package, elle continue d’effectuer des tentatives pour établir la connexion.

> [!TIP]
> Pour établir cette connexion, essayez la solution suivante :
> 1. Sélectionnez **Charger le filtre** > **Filtre standard** > **Adresses** > **Adresses IPv4**.
> 1. Pour appliquer le filtre, entrez **IPv4.Address == 8.8.8.8**, puis sélectionnez **Appliquer**. Vous devez ensuite voir la communication entre l’ordinateur local et la destination **8.8.8.8**.

![Capture d’écran montrant les adresses de filtre.](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-1.png)
        
![Capture d’écran montrant plus d’adresses de filtre.](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-2.png)

Les scénarios réussis sont illustrés dans les exemples suivants : 

- Si vous pouvez utiliser Telnet **8.8.8.8 53** sans aucun problème, il y a une négociation TCP 3 réussie et la session se termine avec une négociation TCP 4.

    ![Capture d’écran montrant un scénario de connexion réussi.](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-1.png)
     
    ![Capture d’écran montrant les détails d’un scénario de connexion réussi.](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-2.png)

- La négociation TCP 3 précédente produit le workflow suivant :

    ![Schéma du workflow de négociation TCP 3.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-3-handshake-workflow.png)
 
- La négociation TCP 4 pour terminer la session est illustrée par les workflows suivants :

    ![Capture d’écran des détails de la négociation TCP 4.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake.png)

    ![Schéma d’un workflow de négociation TCP 4.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake-workflow.png) 

### <a name="microsoft-email-notification-about-updating-your-network-configuration"></a>Notification par e-mail de Microsoft sur la mise à jour de la configuration de votre réseau

Vous pouvez recevoir la notification par e-mail ci-dessous, qui vous recommande de mettre à jour la configuration du réseau afin de permettre la communication avec les nouvelles adresses IP pour Azure Data Factory d’ici le 8 novembre 2020 :

   ![Capture d’écran de la notification par e-mail de Microsoft demandant la mise à jour de la configuration réseau.](media/self-hosted-integration-runtime-troubleshoot-guide/email-notification.png)

#### <a name="determine-whether-this-notification-affects-you"></a>Déterminer si cette notification vous concerne

Cette notification s’applique aux scénarios suivants :

##### <a name="scenario-1-outbound-communication-from-a-self-hosted-integration-runtime-thats-running-on-premises-behind-a-corporate-firewall"></a>Scénario 1 : Communications sortantes à partir du runtime d’intégration auto-hébergé exécuté localement derrière le pare-feu d’entreprise

Comment déterminer si vous êtes concerné :

- Vous *n’êtes pas* concerné si vous définissez des règles de pare-feu basées sur des noms de domaine complets qui utilisent l’approche décrite dans le document [Configurations de pare-feu et configuration de la liste d’autorisation pour les adresses IP](data-movement-security-considerations.md#firewall-configurations-and-allow-list-setting-up-for-ip-address-of-gateway).

- Vous êtes *concerné* si vous activez explicitement la liste d’autorisation pour les adresses IP sortantes sur votre pare-feu d’entreprise.

   Si vous êtes concerné, procédez comme suit : avant le 8 novembre 2020, demandez à votre équipe d’infrastructure réseau de mettre à jour votre configuration réseau pour utiliser les adresses IP Data Factory les plus récentes. Pour télécharger les dernières adresses IP, accédez à [Découvrir les étiquettes de service à l’aide de fichiers JSON téléchargeables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

##### <a name="scenario-2-outbound-communication-from-a-self-hosted-integration-runtime-thats-running-on-an-azure-vm-inside-a-customer-managed-azure-virtual-network"></a>Scénario 2 : Communications sortantes à partir du runtime d’intégration auto-hébergé exécuté sur une machine virtuelle Azure au sein d’un réseau virtuel Azure géré par le client

Comment déterminer si vous êtes concerné :

- Vérifiez si vous avez des règles de groupe de sécurité réseau sortantes dans un réseau privé qui contient l’IR auto-hébergé. S’il n’existe aucune restriction sur les communications sortantes, vous n’êtes pas concerné.

- Si vous avez des restrictions sur les règles de trafic sortant, vérifiez si vous utilisez ou non des étiquettes de service. Si vous utilisez des étiquettes de service, vous n’êtes pas concerné. Vous n’avez pas besoin de modifier ni d’ajouter quoi que ce soit, car les nouvelles plages d’adresses IP se trouvent sous les étiquettes de service actuelles. 

  ![Capture d’écran d’un contrôle de destination montrant DataFactory comme destination.](media/self-hosted-integration-runtime-troubleshoot-guide/destination-check.png)

- Vous *êtes* concerné si vous activez explicitement la liste d’autorisation pour les adresses IP sortantes sur votre paramètre de règles de groupe de sécurité réseau sur le réseau virtuel Azure.

   Si vous êtes concerné, procédez comme suit : avant le 8 novembre 2020, demandez à votre équipe d’infrastructure réseau de mettre à jour les règles de groupe de sécurité réseau sur votre configuration de réseau virtuel Azure pour utiliser les adresses IP les plus récentes de Data Factory. Pour télécharger les dernières adresses IP, accédez à [Découvrir les étiquettes de service à l’aide de fichiers JSON téléchargeables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

##### <a name="scenario-3-outbound-communication-from-ssis-integration-runtime-in-a-customer-managed-azure-virtual-network"></a>Scénario 3 : Communications sortantes à partir de SSIS Integration Runtime dans le réseau virtuel Azure géré par le client

Comment déterminer si vous êtes concerné :

- Vérifiez si vous avez des règles de groupe de sécurité réseau sortantes dans un réseau privé qui contient des runtimes d’intégration SQL Server Integration Services (SSIS). S’il n’existe aucune restriction sur les communications sortantes, vous n’êtes pas concerné.

- Si vous avez des restrictions sur les règles de trafic sortant, vérifiez si vous utilisez ou non des étiquettes de service. Si vous utilisez des étiquettes de service, vous n’êtes pas concerné. Vous n’avez pas besoin de modifier ni d’ajouter quoi que ce soit, car les nouvelles plages d’adresses IP se trouvent sous les étiquettes de service actuelles.

- Vous *êtes* concerné si vous activez explicitement la liste d’autorisation pour les adresses IP sortantes sur votre paramètre de règles de groupe de sécurité réseau sur le réseau virtuel Azure.

  Si vous êtes concerné, procédez comme suit : avant le 8 novembre 2020, demandez à votre équipe d’infrastructure réseau de mettre à jour les règles de groupe de sécurité réseau sur votre configuration de réseau virtuel Azure pour utiliser les adresses IP les plus récentes de Data Factory. Pour télécharger les dernières adresses IP, accédez à [Découvrir les étiquettes de service à l’aide de fichiers JSON téléchargeables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

### <a name="couldnt-establish-a-trust-relationship-for-the-ssltls-secure-channel"></a>Impossible d’établir une relation de confiance pour le canal sécurisé SSL/TLS 

#### <a name="symptoms"></a>Symptômes

L’IR auto-hébergé ne peut pas se connecter au service Azure Data Factory.

Lorsque vous vérifiez le journal des événements de l’IR auto-hébergé ou les journaux de notification du client dans la table CustomLogEvent, le message d’erreur suivant est disponible :

« Le serveur a clos la connexion sous-jacente : Impossible d’établir une relation de confiance pour le canal sécurisé SSL/TLS. Le certificat distant n’est pas valide selon la procédure de validation. »

La façon la plus simple de vérifier le certificat de serveur du service Data Factory consiste à ouvrir l’URL du service Data Factory dans votre navigateur. Par exemple, ouvrez le lien [Vérifier le certificat du serveur](https://eu.frontend.clouddatahub.net/) sur l’ordinateur où est installé l’IR auto-hébergé, puis affichez les informations du certificat du serveur.

  ![Capture d’écran du volet Vérifier le certificat de serveur du service Azure Data Factory.](media/self-hosted-integration-runtime-troubleshoot-guide/server-certificate.png)

  ![Capture d’écran de la fenêtre de vérification du chemin d’accès de certification du serveur.](media/self-hosted-integration-runtime-troubleshoot-guide/certificate-path.png)

#### <a name="cause"></a>Cause

Il existe deux raisons possibles à ce problème :

- Raison 1 : L’autorité de certification racine du certificat de serveur de service Azure Data Factory n’est pas approuvée sur l’ordinateur sur lequel l’IR auto-hébergé est installé. 
- Raison 2 : Vous utilisez un proxy dans votre environnement et le certificat de serveur du service Azure Data Factory est remplacé par le proxy, tandis que le certificat de serveur remplacé n’est pas approuvé par l’ordinateur sur lequel l’IR auto-hébergé est installé.

#### <a name="resolution"></a>Résolution

- Pour la raison 1 : assurez-vous que le certificat de serveur Azure Data Factory et sa chaîne de certificats sont approuvés par l’ordinateur sur lequel l’IR auto-hébergé est installé.
- Pour la raison 2 : approuvez l’autorité de certification racine remplacée sur l’ordinateur de l’IR auto-hébergé ou configurez le proxy pour qu’il ne remplace pas le certificat de serveur Data Factory.

Pour plus d’informations sur l’approbation des certificats sur Windows, consultez [Installation du certificat racine approuvé](/skype-sdk/sdn/articles/installing-the-trusted-root-certificate).

#### <a name="additional-information"></a>Informations supplémentaires
Nous avons déployé un nouveau certificat SSL, qui est signé à partir de DigiCert. Vérifiez si la racine globale DigiCert G2 se trouve dans l’autorité de certification racine de confiance.

  ![Capture d’écran montrant le dossier G2 de racine globale DigiCert dans le répertoire des autorités de certification racines de confiance.](media/self-hosted-integration-runtime-troubleshoot-guide/trusted-root-ca-check.png)

S’il ne se trouve pas dans l’autorité de certification racine de confiance, [téléchargez-le ici](http://cacerts.digicert.com/DigiCertGlobalRootG2.crt ). 


## <a name="self-hosted-ir-sharing"></a>Partage du runtime d’intégration auto-hébergé

### <a name="sharing-a-self-hosted-ir-from-a-different-tenant-is-not-supported"></a>Le partage de l’IR auto-hébergé à partir d’un autre locataire n’est pas pris en charge 

#### <a name="symptoms"></a>Symptômes

Vous pouvez remarquer d’autres fabriques de données (sur différents locataires) lors de la tentative de partage de l’IR auto-hébergé à partir de l’interface utilisateur Azure Data Factory, mais vous ne pouvez pas le partager entre les fabriques de données qui se trouvent sur des locataires différents.

#### <a name="cause"></a>Cause

L’IR auto-hébergé ne peut pas être partagé entre plusieurs locataires.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la résolution des problèmes, essayez les ressources suivantes :

*  [Blog Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Demandes de fonctionnalités Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vidéos Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Page Microsoft Q&A](/answers/topics/azure-data-factory.html)
*  [Forum Stack Overflow pour Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Informations Twitter sur Data Factory](https://twitter.com/hashtag/DataFactory)
*  [Guide de performances des flux de données de mappage](concepts-data-flow-performance.md)
