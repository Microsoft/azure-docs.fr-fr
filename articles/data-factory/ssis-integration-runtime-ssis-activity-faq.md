---
title: Résoudre les problèmes d’exécution de package dans le runtime d’intégration SSIS
description: Cet article fournit des instructions pour la résolution des problèmes d’exécution de package SSIS dans le runtime d’intégration SSIS
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: wenjiefu
author: wenjiefu
ms.reviewer: sawinark
manager: shwang
ms.custom: seo-lt-2019
ms.date: 04/15/2019
ms.openlocfilehash: b4902e1fb7a2a181d3d5b2ce2ac6d1d458500fce
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844180"
---
# <a name="troubleshoot-package-execution-in-the-ssis-integration-runtime"></a>Résoudre les problèmes d’exécution de package dans le runtime d’intégration SSIS

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article contient des erreurs les plus courantes que vous pouvez rencontrer quand vous exécutez des packages SQL Server Integration Services (SSIS) dans le runtime d’intégration SSIS. Il décrit les causes possibles et les actions permettant de résoudre les erreurs.

## <a name="where-to-find-logs-for-troubleshooting"></a>Où trouver les journaux pour la résolution des problèmes

Utilisez le portail Azure Data Factory pour consulter la sortie de l’activité d’exécution de package SSIS. Cette sortie inclut le résultat de l’exécution, les messages d’erreur et l’ID d’opération. Pour plus d’informations, consultez [Superviser le pipeline](how-to-invoke-ssis-package-ssis-activity.md#monitor-the-pipeline).

Utilisez le catalogue SSIS (SSISDB) pour consulter les journaux des détails concernant l’exécution. Pour plus d’informations, consultez [Superviser les packages en cours d’exécution et autres opérations](/sql/integration-services/performance/monitor-running-packages-and-other-operations?view=sql-server-2017).

## <a name="common-errors-causes-and-solutions"></a>Erreurs courantes, causes et solutions

### <a name="error-message-connection-timeout-expired-or-the-service-has-encountered-an-error-processing-your-request-please-try-again"></a>Message d’erreur : « Délai d'expiration de la connexion dépassé. » ou « Le service a rencontré une erreur lors du traitement de votre demande. Réessayez. »

Voici des causes possibles et les actions recommandées :
* La source ou la destination de données est surchargée. Vérifiez la charge sur votre source ou votre destination de données et si elle dispose d’une capacité suffisante. Par exemple, si vous avez utilisé Azure SQL Database, envisagez d’effectuer un scale-up si la base de données est susceptible d’expirer.
* Le réseau entre le runtime d’intégration SSIS et la source ou la destination de données est instable, en particulier quand la connexion est inter-régions ou entre un emplacement local et Azure. Appliquez le modèle de nouvelle tentative dans le package SSIS en effectuant les étapes suivantes :
  * Vérifiez que vos packages SSIS peuvent se réexécuter en cas d’échec sans effets secondaires (par exemple, une perte de données ou une duplication des données).
  * Configurez **Nouvelle tentative** et **Intervalle avant nouvelle tentative** de l’activité **Exécuter le package SSIS** sous l’onglet **Général**. ![Définir des propriétés sous l’onglet Général](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png)
  * Pour un composant source ou de destination ADO.NET et OLE DB, définissez **ConnectRetryCount** et **ConnectRetryInterval** dans le Gestionnaire de connexions dans le package SSIS ou l’activité SSIS.

### <a name="error-message-ado-net-source-has-failed-to-acquire-the-connection--with-a-network-related-or-instance-specific-error-occurred-while-establishing-a-connection-to-sql-server-the-server-was-not-found-or-was-not-accessible"></a>Message d’erreur : « La source ADO NET n’a pas pu acquérir la connexion '...' » avec « Une erreur liée au réseau ou spécifique à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible. »

Ce problème signifie généralement que la source ou la destination de données n’est pas accessible à partir du runtime d’intégration SSIS. Les raisons peuvent varier. Essayez les actions suivantes :
* Vérifiez que vous passez correctement le nom/l’adresse IP de la source ou de la destination de données.
* Vérifiez que le pare-feu est correctement défini.
* Vérifiez que votre réseau virtuel est correctement configuré si votre source ou destination de données est locale :
  * Vous pouvez vérifier si le problème provient de la configuration du réseau virtuel en provisionnant une machine virtuelle Azure dans le même réseau virtuel. Vérifiez ensuite si la source ou la destination de données est accessible à partir de la machine virtuelle Azure.
  * Vous trouverez plus d’informations sur l’utilisation d’un réseau virtuel avec un runtime d’intégration SSIS dans [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md).

### <a name="error-message-ado-net-source-has-failed-to-acquire-the-connection--with-could-not-create-a-managed-connection-manager"></a>Message d’erreur : « La source ADO NET n’a pas pu acquérir la connexion '...' » avec « Impossible de créer un gestionnaire de connexions managées. »

La cause possible est que le fournisseur ADO.NET utilisé dans le package n’est pas installé dans le runtime d’intégration SSIS. Vous pouvez installer le fournisseur en utilisant une configuration personnalisée. Vous trouverez plus d’informations sur la configuration personnalisée dans [Personnaliser la configuration du runtime d’intégration Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md).

### <a name="error-message-the-connection--is-not-found"></a>Message d’erreur : « La connexion '...' est introuvable »

Un problème connu dans les versions antérieures de SQL Server Management Studio (SSMS) peut provoquer cette erreur. Si le package contient un composant personnalisé (par exemple, le Feature Pack SSIS-Azure ou des composants partenaires) qui n’est pas installé sur l’ordinateur où SSMS est utilisé pour effectuer le déploiement, SSMS supprime ce composant et provoque l’erreur. Mettez à niveau [SSMS](/sql/ssms/download-sql-server-management-studio-ssms) vers la dernière version dans laquelle le problème a été résolu.

### <a name="error-messagessis-executor-exit-code--1073741819"></a>Message d’erreur : « Code de sortie de SSIS Executor : -1073741819 ».

* Cause possible et action recommandée :
  * Cette erreur peut être due à la limitation de la source et de la destination Excel quand plusieurs sources ou destinations Excel s’exécutent en parallèle en multithread. Vous pouvez contourner cette limitation en configurant vos composants Excel pour qu’ils s’exécutent en séquence, ou les séparer en différents packages et déclencher via « Tâche d’exécution de package » avec la propriété ExecuteOutOfProcess définie sur True.

### <a name="error-message-there-is-not-enough-space-on-the-disk"></a>Message d’erreur : « Espace insuffisant sur le disque »

Cette erreur signifie que le disque local est entièrement utilisé dans le nœud de runtime d’intégration SSIS. Vérifiez si votre package ou configuration personnalisée consomme beaucoup d’espace disque :
* Si le disque est consommé par votre package, il sera libéré une fois l’exécution du package terminée.
* Si le disque est consommé par votre configuration personnalisée, vous devez arrêter le runtime d’intégration SSIS, modifier votre script et redémarrer le runtime d’intégration. L’ensemble du conteneur d’objets blob Azure que vous avez spécifié pour la configuration personnalisée est copié dans le nœud de runtime d’intégration SSIS. Par conséquent, vérifiez du contenu inutile se trouve sous ce conteneur.

### <a name="error-message-failed-to-retrieve-resource-from-master-microsoftsqlserverintegrationservicesscalescaleoutcontractcommonmasterresponsefailedexception-code300004-descriptionload-file--failed"></a>Message d’erreur : « Échec de la récupération de la ressource du maître. Microsoft.SqlServer.IntegrationServices.Scale.ScaleoutContract.Common.MasterResponseFailedException : Code : 300004. Description : Échec du chargement du fichier « *** ».

* Cause possible et action recommandée :
  * Si l’activité SSIS exécute un package à partir du système de fichiers (fichier de package ou fichier projet), cette erreur se produit si le fichier projet, de package ou de configuration n’est pas accessible avec les informations d’identification d’accès au package que vous avez fournies dans l’activité SSIS.
    * Si vous utilisez Azure Files :
      * Le chemin d’accès au fichier doit commencer par \\\\\<storage account name\>. file.core.windows.net\\\<file share path\>
      * Le domaine doit être « Azure »
      * Le nom d’utilisateur doit être \<storage account name\>
      * Le mot de passe doit être \<storage access key\>
    * Si vous utilisez un fichier local, veillez vérifier si les informations d'identification et les autorisations d’accès au réseau virtuel et au package sont correctement configurées pour que votre runtime d'intégration SSIS Azure puisse accéder à votre partage de fichiers locaux

### <a name="error-message-the-file-name--specified-in-the-connection-was-not-valid"></a>Message d’erreur : « Le nom de fichier '...' spécifié dans la connexion n’est pas valide »

* Cause possible et action recommandée :
  * Le nom de fichier spécifié n’est pas valide
  * Vérifiez que vous utilisez le nom de domaine complet (FQDN) au lieu du nom court dans votre gestionnaire de connexions

### <a name="error-message-cannot-open-file-"></a>Message d’erreur : « Impossible d’ouvrir le fichier '...' »

Cette erreur se produit quand une exécution de package ne parvient pas à trouver un fichier sur le disque local dans le runtime d’intégration SSIS. Essayez les actions suivantes :
* N’utilisez pas le chemin absolu dans le package qui est en cours d’exécution dans le runtime d’intégration SSIS. Utilisez le répertoire de travail (.) ou le dossier temporaire (%TEMP%) de l’exécution actuelle à la place.
* Si vous avez besoin de conserver des fichiers sur des nœuds de runtime d’intégration SSIS, préparez les fichiers comme décrit dans [Personnaliser la configuration](how-to-configure-azure-ssis-ir-custom-setup.md). Tous les fichiers se trouvant dans le répertoire de travail sont nettoyés une fois l’exécution terminée.
* Utilisez Azure Files au lieu de stocker le fichier dans le nœud de runtime d’intégration SSIS. Pour plus d’informations, consultez [Utiliser des partages de fichiers Azure](/sql/integration-services/lift-shift/ssis-azure-files-file-shares?view=sql-server-2017#use-azure-file-shares).

### <a name="error-message-the-database-ssisdb-has-reached-its-size-quota"></a>Message d’erreur : « La base de données « SSISDB » a atteint son quota de taille.»

Le fait que la base de données SSISDB créée dans Azure SQL Database ou SQL Managed Instance a atteint son quota constitue une cause possible. Essayez les actions suivantes :
* Envisagez d’augmenter le nombre de DTU de votre base de données. Vous trouverez plus d’informations dans les [limites SQL Database pour un serveur logique](../azure-sql/database/resource-limits-logical-server.md).
* Vérifiez si votre package génère de nombreux journaux. Si c’est le cas, vous pouvez configurer un travail élastique pour nettoyer ces journaux. Pour plus d’informations, consultez [Nettoyer les journaux SSISDB avec les travaux de base de données élastique Azure](how-to-clean-up-ssisdb-logs-with-elastic-jobs.md).

### <a name="error-message-the-request-limit-for-the-database-is--and-has-been-reached"></a>Message d’erreur : « La limite de requêtes pour la base de données est ... et elle a été atteinte. »

Si de nombreux packages s’exécutent en parallèle dans le runtime d’intégration SSIS, cette erreur peut se produire car SSISDB a atteint sa limite de requêtes. Envisagez d’augmenter le nombre de DTC de SSISDB pour résoudre ce problème. Vous trouverez plus d’informations dans les [limites SQL Database pour un serveur logique](../azure-sql/database/resource-limits-logical-server.md).

### <a name="error-message-ssis-operation-failed-with-unexpected-operation-status-"></a>Message d’erreur : « L’opération SSIS a échoué avec un état d’opération inattendu : ... »

L’erreur est principalement due à un problème temporaire. Par conséquent, tentez de réexécuter le package. Appliquez le modèle de nouvelle tentative dans le package SSIS en effectuant les étapes suivantes :

* Vérifiez que vos packages SSIS peuvent se réexécuter en cas d’échec sans effets secondaires (par exemple, une perte de données ou une duplication des données).
* Configurez **Nouvelle tentative** et **Intervalle avant nouvelle tentative** de l’activité **Exécuter le package SSIS** sous l’onglet **Général**. ![Définir des propriétés sous l’onglet Général](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png)
* Pour un composant source ou de destination ADO.NET et OLE DB, définissez **ConnectRetryCount** et **ConnectRetryInterval** dans le Gestionnaire de connexions dans le package SSIS ou l’activité SSIS.

### <a name="error-message-there-is-no-active-worker"></a>Message d’erreur : « Il n’y a aucun Worker Agent actif. »

Cette erreur signifie généralement que le runtime d’intégration SSIS a un état défectueux. Consultez le portail Azure pour connaître l’état et le détail des erreurs. Pour plus d’informations, consultez [Runtime d’intégration Azure-SSIS](./monitor-integration-runtime.md#azure-ssis-integration-runtime).

### <a name="error-message-your-integration-runtime-cannot-be-upgraded-and-will-eventually-stop-working-since-we-cannot-access-the-azure-blob-container-you-provided-for-custom-setup"></a>Message d’erreur : « Votre runtime d’intégration ne peut pas être mis à niveau et cessera finalement de fonctionner, car nous ne pouvons pas accéder au conteneur d’objets blob Azure que vous avez fourni pour la configuration personnalisée. »

Cette erreur se produit quand le runtime d’intégration SSIS ne peut pas accéder au stockage défini pour une configuration personnalisée. Vérifiez si l’URI de la signature d’accès partagé (SAP) que vous avez fourni est valide et n’a pas expiré.

### <a name="error-message-microsoft-ole-db-provider-for-analysis-services-hresult-0x80004005-description-com-error-com-error-mscorlib-exception-has-been-thrown-by-the-target-of-an-invocation"></a>Message d’erreur : « Fournisseur Microsoft OLE DB pour Analysis Services. 'Hresult : 0x80004005 Description :' Erreur COM : Erreur COM : mscorlib ; Une exception a été levée par la cible d’un appel »

Le fait que le nom d'utilisateur ou le mot de passe soit configuré pour l'authentification Azure Analysis Services alors que la fonctionnalité Azure AD Multi-Factor Authentication est activée constitue une cause possible. Cette authentification n’est pas prise en charge dans le runtime d’intégration SSIS. Essayez d’utiliser un principal de service pour l’authentification Azure Analysis Services :

1. Préparez un principal de service comme décrit dans [Automatisation à l’aide de principaux de service](../analysis-services/analysis-services-service-principal.md).
2. Dans le Gestionnaire de connexions, configurez **Utiliser un nom d'utilisateur et un mot de passe spécifiques** : définissez **AppID** comme nom d’utilisateur et **clientSecret** comme mot de passe.

### <a name="error-message-adonet-source-has-failed-to-acquire-the-connection-guid-with-the-following-error-message-login-failed-for-user-nt-authorityanonymous-logon-when-using-a-managed-identity"></a>Message d’erreur : « La source ADO NET Source n’a pas pu acquérir la connexion la connexion {GUID}. Message d’erreur : Échec de la connexion pour l’utilisateur 'NT AUTHORITY\ANONYMOUS LOGON' » lors de l’utilisation d’une identité managée

Vérifiez que vous ne configurez pas **Authentification par mot de passe Active Directory** comme méthode d’authentification du Gestionnaire de connexions quand le paramètre *ConnectUsingManagedIdentity* a la valeur **True** . Vous pouvez le configurer plutôt sur **Authentification SQL**, qui est ignoré si *ConnectUsingManagedIdentity* est défini.

### <a name="error-message-0xc020801f-at--odata-source--cannot-acquire-a-managed-connection-from-the-run-time-connection-manager"></a>Message d’erreur : « 0xC020801F at ..., OData Source [...] : Impossible d’acquérir une connexion gérée depuis le gestionnaire de connexions en cours d’exécution »

Le fait que le protocole TLS (Transport Layer Security) ne soit pas activé dans le runtime d’intégration SSIS qui est exigé par votre source OData constitue une cause possible. Vous pouvez activer TLS dans le runtime d’intégration SSIS à l’aide de la configuration personnalisée. Vous trouverez des informations supplémentaires dans [Vous ne pouvez pas vous connecter à Project Online OData à partir d’SSIS](/office365/troubleshoot/cant-connect-project-online-odata-from-ssis) et [Personnaliser la configuration du runtime d’intégration Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md).

### <a name="error-message-request-staging-task-with-operation-guid--fail-since-error-failed-to-dispatch-staging-operation-with-error-message-microsoftsqlserverintegrationservicesaisagentcoreaisagentexception-failed-to-load-data-proxy"></a>Message d’erreur : « Échec de la tâche de mise en lots des requêtes avec le GUID d’opération ... en raison de l’erreur : Échec de la distribution de l’opération de mise en lots avec le message d’erreur : Microsoft.SqlServer.IntegrationServices.AisAgentCore.AisAgentException : Échec du chargement du proxy de données. »

Assurez-vous que votre runtime d’intégration Azure-SSIS est configuré avec un runtime d’intégration auto-hébergé. Pour plus d’informations, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS IR dans ADF](self-hosted-integration-runtime-proxy-ssis.md).

### <a name="error-message-staging-task-status-failed-staging-task-error-errorcode-2010-errormessage-the-self-hosted-integration-runtime--is-offline"></a>Message d’erreur : « État de la tâche de mise en lots : Échec. Erreur de tâche de mise en lots : ErrorCode : 2010, message d’erreur : Le runtime d’intégration auto-hébergé ... est hors connexion »

Assurez-vous que votre runtime d’intégration auto-hébergé est installé et démarré. Pour plus d’informations, consultez [Créer et configurer un runtime d’intégration auto-hébergé](create-self-hosted-integration-runtime.md).

### <a name="error-message-staging-task-error-errorcode-2906-errormessage-package-execution-failed-output-operationerrormessages-error-the-requested-ole-db-provider--is-not-registered-if-the-64-bit-driver-is-not-installed-run-the-package-in-32-bit-mode"></a>Message d’erreur : « Erreur de tâche de mise en lots : ErrorCode : 2906, message d’erreur : Échec de l’exécution du package., Sortie : {"OperationErrorMessages": « Erreur : Le fournisseur OLE DB ... demandé n’est pas inscrit. Si le pilote 64 bits n’est pas installé, exécutez le package en mode 32 bits... »

Assurez-vous que le fournisseur correspondant utilisé par les connecteurs OLE DB dans votre package est installé correctement sur l’ordinateur du runtime d’intégration auto-hébergé. Pour plus d’informations, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS IR dans ADF](self-hosted-integration-runtime-proxy-ssis.md#prepare-the-self-hosted-ir).

### <a name="error-message-staging-task-error-errorcode-2906-errormessage-package-execution-failed-output-operationerrormessages-error-systemiofileloadexception-could-not-load-file-or-assembly-microsoftwindowsazurestorage-version-cultureneutral-publickeytoken31bf3856ad364e35-or-one-of-its-dependencies-the-located-assemblys-manifest-definition-does-not-match-the-assembly-reference"></a>Message d’erreur : « Erreur de tâche de mise en lots : ErrorCode : 2906, message d’erreur : Échec de l’exécution du package., Sortie : {"OperationErrorMessages": « Erreur : System.IO.FileLoadException : Impossible de charger le fichier ou l’assembly « Microsoft.WindowsAzure.Storage, Version=..., Culture=neutral, PublicKeyToken=31bf3856ad364e35 » ou l’une de ses dépendances. La définition du manifeste de l’assembly trouvé ne correspond pas à la référence de l’assembly.'... »

Une installation ou une mise à niveau incorrecte de votre runtime d’intégration auto-hébergé peut expliquer cette erreur. Suggérez de télécharger et de réinstaller le dernier runtime d’intégration auto-hébergé. Pour plus d’informations, consultez [Créer et configurer un runtime d’intégration auto-hébergé](create-self-hosted-integration-runtime.md#installation-best-practices).

### <a name="error-message-a-connection-is-required-when-requesting-metadata-if-you-are-working-offline-uncheck-work-offline-on-the-ssis-menu-to-enable-the-connection"></a>Message d’erreur : « Une connexion est nécessaire lors d’une demande de métadonnées. Si vous travaillez hors connexion, désactivez l’option Travailler hors connexion dans le menu SSIS pour activer la connexion. »

* Cause possible et action recommandée :
  * Si le journal d’exécution contient également le message d’avertissement « Le composant ne prend pas en charge l’utilisation du gestionnaire de connexions avec la valeur ConnectByProxy définie sur true », cela signifie qu’un gestionnaire de connexions est utilisé sur un composant qui ne prend pas encore en charge « ConnectByProxy ». Pour connaître les composants pris en charge, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS IR dans ADF](self-hosted-integration-runtime-proxy-ssis.md#enable-ssis-packages-to-connect-by-proxy).
  * Le journal d’exécution est disponible dans le [rapport SSMS](/sql/integration-services/performance/monitor-running-packages-and-other-operations?view=sql-server-2017#reports) ou dans le dossier du journal que vous avez spécifié dans l’activité d’exécution du package SSIS.
  * Il est également possible d’utiliser un réseau virtuel pour accéder aux données locales. Pour plus d’informations, consultez [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md).

### <a name="error-message-staging-task-status-failed-staging-task-error-errorcode-2906-errormessage-package-execution-failed-output-operationerrormessages-ssis-executor-exit-code--1n-loglocation-ssistelemetryexecutionlog-effectiveintegrationruntime--executionduration--durationinqueue--integrationruntimequeue--"></a>Message d’erreur : « État de la tâche de mise en lots : Échec. Erreur de tâche de mise en lots : ErrorCode : 2906, message d’erreur : Échec de l’exécution du package., Sortie : {"OperationErrorMessages": "Code de sortie de SSIS Executor : -1.\n", "LogLocation": "...\\SSISTelemetry\\ExecutionLog\\...", "effectiveIntegrationRuntime": "...", "executionDuration": ..., "durationInQueue": { "integrationRuntimeQueue": ... }}"

Assurez-vous que le runtime Visual C++ est installé sur la machine du runtime d’intégration auto-hébergé. Pour plus d’informations, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS IR dans ADF](self-hosted-integration-runtime-proxy-ssis.md#prepare-the-self-hosted-ir).

### <a name="multiple-package-executions-are-triggered-unexpectedly"></a>Plusieurs exécutions de package sont déclenchées de manière inattendue

* Cause possible et action recommandée :
  * L’activité de procédure stockée ADF ou l’activité Lookup est utilisée pour déclencher l’exécution du package SSIS. La commande t-sql peut rencontrer un problème temporaire et déclencher la réexécution, ce qui entraînerait plusieurs exécutions de package.
  * Utilisez l’activité ExecuteSSISPackage à la place, qui garantit que l’exécution du package ne sera pas renouvelée sauf si l’utilisateur a défini un nombre de nouvelles tentatives dans l’activité. Vous trouverez des détails sur [https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity](./how-to-invoke-ssis-package-ssis-activity.md)
  * Affinez votre commande T-SQL pour pouvoir la réexécuter en vérifiant si une exécution a déjà été déclenchée.

### <a name="package-execution-takes-too-long"></a>L’exécution du package prend trop de temps

Voici des causes possibles et les actions recommandées :

* Un trop grand nombre d’exécutions de package ont été planifiées sur le runtime d’intégration SSIS. Toutes ces exécutions attendront leur tour dans une file d’attente.
  * Déterminer le nombre maximal à l’aide de cette formule :

    Nombre max d’exécutions en parallèle par runtime d’intégration = Nombre de nœuds * nombre max d’exécutions en parallèle par nœud
  * Pour savoir comment définir le nombre de nœuds et le nombre maximal d’exécutions en parallèle par nœud, consultez [Créer un runtime d’intégration Azure-SSIS dans Azure Data Factory](create-azure-ssis-integration-runtime.md).
* Le runtime d’intégration SSIS est arrêté ou a un état défectueux. Pour savoir comment vérifier l’état du runtime d’intégration SSIS et les erreurs, consultez [Runtime d’intégration Azure-SSIS](monitor-integration-runtime.md#azure-ssis-integration-runtime).

Nous vous recommandons également de définir un délai d’expiration sous l’onglet **Général** : ![Définir des propriétés sous l’onglet Général](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png).

### <a name="poor-performance-in-package-execution"></a>Problèmes de performances dans l’exécution des packages

Essayez les actions suivantes :

* Vérifiez que le runtime d’intégration SSIS se trouve dans la même région que la source et la destination de données.

* Définissez le niveau de journalisation de l’exécution de package sur **Performances** pour collecter des informations de durée pour chaque composant de l’exécution. Pour plus d’informations, consultez [Journalisation d’Integration Services (SSIS)](/sql/integration-services/performance/integration-services-ssis-logging).

* Vérifier les performances de nœuds de runtime d’intégration dans le portail Azure :
  * Pour plus d’informations sur la façon de superviser le runtime d’intégration SSIS, consultez [Runtime d’intégration Azure-SSIS](monitor-integration-runtime.md#azure-ssis-integration-runtime).
  * Vous trouverez l’historique du processeur/de la mémoire pour le runtime d’intégration SSIS en consultant les métriques de la fabrique de données dans le portail Azure.
    ![Superviser les métriques du runtime d’intégration SSIS](media/ssis-integration-runtime-ssis-activity-faq/monitor-metrics-ssis-integration-runtime.png)