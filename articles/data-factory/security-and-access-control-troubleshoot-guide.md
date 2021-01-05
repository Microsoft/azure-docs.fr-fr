---
title: Résoudre les problèmes de sécurité et de contrôle d’accès
description: Découvrez comment résoudre les problèmes de sécurité et de contrôle d’accès dans Azure Data Factory.
services: data-factory
author: lrtoyou1223
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 11/19/2020
ms.author: lle
ms.reviewer: craigg
ms.openlocfilehash: 51cb1a1a8151748fc9c6cd4c81da967424b52868
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505152"
---
# <a name="troubleshoot-azure-data-factory-security-and-access-control-issues"></a>Résoudre les problèmes de sécurité et de contrôle d’accès dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article présente des méthodes couramment employées pour résoudre les problèmes de sécurité et de contrôle d’accès dans Azure Data Factory.

## <a name="common-errors-and-messages"></a>Erreurs et messages courants

### <a name="connectivity-issue-in-the-copy-activity-of-the-cloud-datastore"></a>Problème de connectivité dans l’activité de copie du magasin de données cloud

#### <a name="symptoms"></a>Symptômes

Divers messages d’erreur peuvent être renvoyés en cas de problèmes de connectivité dans le magasin de données source ou récepteur.

#### <a name="cause"></a>Cause 

Le problème est-il généralement dû à l'un des facteurs suivants ?

* Paramètre de proxy dans le nœud Runtime d’intégration auto-hébergé (IR), si vous utilisez un IR auto-hébergé.

* Paramètre de pare-feu dans le nœud Runtime d’intégration auto-hébergé, si vous utilisez un IR auto-hébergé.

* Paramètre de pare-feu dans le magasin de données cloud.

#### <a name="resolution"></a>Résolution

* Pour vous assurer qu’il s’agit d’un problème de connectivité, vérifiez les points suivants :

   - L’erreur est générée à partir des connecteurs source ou récepteur.
   - L’échec se produit au début de l’activité de copie.
   - L’échec est cohérent pour Azure IR ou le runtime d’intégration auto-hébergé avec un nœud, car il peut s’agir d’un échec aléatoire dans un Runtime d’intégration auto-hébergé à plusieurs nœuds si seuls certains nœuds ont le problème.

* Si vous utilisez un **runtime d'intégration auto-hébergé**, vérifiez vos paramètres de proxy, de pare-feu et réseau, car la connexion au même magasin de données peut être réussie si vous utilisez un Azure IR. Pour résoudre les problèmes de ce scénario, consultez :

   * [Ports et pare-feu pour le runtime d’intégration auto-hébergé](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime#ports-and-firewalls)
   * [Connecteur Azure Data Lake Storage](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-store)
  
* Si vous utilisez un **Azure IR**, essayez de désactiver le paramètre de pare-feu du magasin de données. Cette approche peut résoudre les problèmes dans les deux situations suivantes :
  
   * Les [adresses IP Azure IR ](https://docs.microsoft.com/azure/data-factory/azure-integration-runtime-ip-addresses) ne figurent pas dans la liste verte.
   * La fonctionnalité *Autoriser les services Microsoft approuvés à accéder à ce compte de stockage* est désactivée pour [Stockage Blob Azure](https://docs.microsoft.com/azure/data-factory/connector-azure-blob-storage#supported-capabilities) et [Azure Data Lake Storage Gen 2](https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#supported-capabilities).
   * Le paramètre *Autoriser l’accès aux services Azure* n’est pas activé pour Azure Data Lake Storage Gen1.

Si aucune des méthodes précédentes ne fonctionne, contactez Microsoft pour obtenir de l’aide.


### <a name="invalid-or-empty-authentication-key-issue-after-public-network-access-is-disabled"></a>Problème de clé d’authentification vide ou non valide après la désactivation de l’accès au réseau public

#### <a name="symptoms"></a>Symptômes

Une fois que vous avez désactivé l’accès au réseau public pour Data Factory, le runtime d’intégration auto-hébergé génère l’erreur suivante : « La clé d'authentification n'est pas valide. »

#### <a name="cause"></a>Cause

Le problème est probablement dû à une erreur de résolution DNS (Domain Name System), car la désactivation de la connectivité publique et l’établissement d’un point de terminaison privé ne permettent pas la reconnexion.

Pour vérifier si le nom de domaine complet (FQDN) Data Factory est résolu en adresse IP publique, procédez comme suit :

1. Vérifiez que vous avez créé la machine virtuelle Azure dans le même réseau virtuel que le point de terminaison privé Data Factory.

2. Exécutez PsPing et Ping à partir de la machine virtuelle Azure dans le nom de domaine complet Data Factory :

   `psping.exe <dataFactoryName>.<region>.datafactory.azure.net:443`
   `ping <dataFactoryName>.<region>.datafactory.azure.net`

   > [!Note]
   > Vous devez spécifier un port pour la commande PsPing. Le port 443 est illustré ici, mais n’est pas obligatoire.

3. Vérifiez si les deux commandes correspondent à une adresse IP publique Azure Data Factory basée sur une région spécifiée. L’adresse IP doit respecter le format suivant : `xxx.xxx.xxx.0`

#### <a name="resolution"></a>Résolution

Pour résoudre ce problème, procédez comme suit :
- Consultez l’article [Azure Private Link pour Azure Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-private-link#dns-changes-for-private-endpoints). L’instruction consiste à configurer la zone ou le serveur DNS privé pour résoudre le nom de domaine complet Data Factory en adresse IP privée.

- Nous vous recommandons d’utiliser un DNS personnalisé comme solution à long terme. Toutefois, si vous ne voulez pas configurer la zone ou le serveur DNS privé, essayez la solution temporaire suivante :

  1. Modifiez le fichier hôte dans Windows et mappez l’adresse IP privée (le point de terminaison privé Azure Data Factory) au nom de domaine complet Azure Data Factory.
  
     Dans la machine virtuelle Azure, accédez à `C:\Windows\System32\drivers\etc`, puis ouvrez le fichier *hôte* dans le Bloc-notes. Ajoutez la ligne qui mappe l’adresse IP privée au nom de domaine complet à la fin du fichier, puis enregistrez la modification.
     
     ![Capture d’écran du mappage de l’adresse IP privée à l’hôte.](media/self-hosted-integration-runtime-troubleshoot-guide/add-mapping-to-host.png)

  1. Réexécutez les mêmes commandes que dans les étapes de vérification précédentes pour vérifier la réponse, qui doit contenir l’adresse IP privée.

  1. Réinscrivez le runtime d’intégration auto-hébergé et le problème devrait être résolu.

### <a name="unable-to-register-ir-authentication-key-on-self-hosted-vms-due-to-private-link"></a>Impossible d’inscrire la clé d’authentification de l’IR sur les machines virtuelles auto-hébergées en raison d’une liaison privée

#### <a name="symptoms"></a>Symptômes

Vous ne pouvez pas inscrire la clé d’authentification IR sur la machine virtuelle auto-hébergée, car le lien privé est activé. Vous recevez le message d’erreur suivant :

« Échec de l’obtention du jeton de service du service ADF avec la clé *************** et le coût du temps est : 0,1250079 seconde, le code d’erreur est : InvalidGatewayKey, activityId est : XXXXXXX et le message d’erreur détaillé est L’adresse IP du client n’est pas une adresse IP privée valide, car la fabrique de données n’a pas pu accéder au réseau public et ne peut donc pas atteindre le cloud pour établir la connexion. »

#### <a name="cause"></a>Cause

Le problème peut être dû à la machine virtuelle dans laquelle vous tentez d’installer le runtime d’intégration auto-hébergé. Pour vous connecter au cloud, assurez-vous que l’accès au réseau public est activé.

#### <a name="resolution"></a>Résolution

**Solution 1**
 
Pour résoudre ce problème, procédez comme suit :

1. Accédez à la page [Fabriques - Mettre à jour](https://docs.microsoft.com/rest/api/datafactory/Factories/Update).

1. En haut à droite, sélectionnez le bouton **Essayer**.
1. Sous **Paramètres**, renseignez les informations requises. 
1. Sous **Corps**, collez la propriété suivante :

    ```
    { "tags": { "publicNetworkAccess":"Enabled" } }
    ```
1. Sélectionnez **Exécuter** pour exécuter la fonction. 

1. Sous **Paramètres**, renseignez les informations requises. 

1. Sous **Corps**, collez la propriété suivante :
    ```
    { "tags": { "publicNetworkAccess":"Enabled" } }
    ``` 

1. Sélectionnez **Exécuter** pour exécuter la fonction. 
1. Vérifiez que **Code de réponse : 200** s'affiche. La propriété que vous avez collée doit également s’afficher dans la définition JSON.

1. Ajoutez de nouveau la clé d’authentification IR dans le runtime d’intégration.


**Solution 2**

Pour résoudre le problème, accédez à [Azure Private Link pour Azure Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-private-link).

Essayez d’activer l’accès au réseau public sur l’interface utilisateur, comme illustré dans la capture d’écran suivante :

![Capture d’écran du contrôle « Activé » pour « Autoriser l’accès au réseau public » dans le volet Mise en réseau.](media/self-hosted-integration-runtime-troubleshoot-guide/enable-public-network-access.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la résolution des problèmes, essayez les ressources suivantes :

*  [Private Link pour Data Factory](data-factory-private-link.md)
*  [Blog Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Demandes de fonctionnalités Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vidéos Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Page Microsoft Q&A](/answers/topics/azure-data-factory.html)
*  [Forum Stack Overflow pour Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Informations Twitter sur Data Factory](https://twitter.com/hashtag/DataFactory)
