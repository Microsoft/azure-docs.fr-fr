---
title: Guide de résolution des problèmes
titleSuffix: Microsoft Genomics
description: En savoir plus sur la résolution des problèmes liés à l’utilisation de Microsoft Genomics, notamment sur les messages d’erreur et leur résolution.
keywords: dépannage, erreur, débogage
services: genomics
author: ruchir
ms.author: ruchir
ms.service: genomics
ms.workload: genomics
ms.topic: troubleshooting
ms.date: 10/29/2018
ms.openlocfilehash: c508c10d619cde1a16d89b446c5cfd1a3ce81daf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023821"
---
# <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

Voici quelques conseils de dépannage pour certains des problèmes courants que vous pouvez rencontrer lors de l’utilisation du service Microsoft Genomics, ou MSGEN.

 Pour accéder aux questions fréquentes (FAQ), qui ne sont pas liées à la résolution des problèmes, consultez [Questions courantes](frequently-asked-questions-genomics.md).
## <a name="step-1-locate-error-codes-associated-with-the-workflow"></a>Étape 1 : Rechercher les codes d’erreur associés au flux de travail

Vous pouvez localiser les messages d’erreur associés au flux de travail en :

1. Utilisant la ligne de commande et en tapant `msgen status`
2. Examinant le contenu de standardoutput.txt.

### <a name="1-using-the-command-line-msgen-status"></a>1. Utilisation de la ligne de commande et saisie de `msgen status`

```bash
msgen status -u URL -k KEY -w ID 
```




Il existe trois arguments obligatoires :

* URL : URL de base de l’API
* KEY : clé d’accès de votre compte Genomics
    * Pour trouver votre URL et votre clé, accédez au portail Azure et ouvrez la page de votre compte Microsoft Genomics. Sous le titre **Gestion**, choisissez **Clés d’accès**. Vous y trouvez à la fois l’URL de l’API et vos clés d’accès.

  
* ID : ID du flux de travail
    * Pour trouver votre ID de flux de travail, tapez la commande `msgen list`. En supposant que votre fichier config contient l’URL et vos clés d’accès, et qu’il se trouve au même emplacement que votre exécutable msgen, la commande doit se présenter comme suit : 
        
        ```bash
        msgen list -f "config.txt"
        ```
        La sortie de cette commande doit ressembler à ceci :
        
        ```bash
            Microsoft Genomics command-line client v0.7.4
                Copyright (c) 2018 Microsoft. All rights reserved.
                
                Workflow List
                -------------
                Total Count  : 1
                
                Workflow ID     : 10001
                Status          : Completed successfully
                Message         :
                Process         : snapgatk-20180730_1
                Description     :
                Created Date    : Mon, 27 Aug 2018 20:27:24 GMT
                End Date        : Mon, 27 Aug 2018 20:55:12 GMT
                Wall Clock Time : 0h 27m 48s
                Bases Processed : 1,348,613,600 (1 GBase)
        ```

  > [!NOTE]
  >  Vous pouvez également inclure le chemin du fichier config au lieu d’entrer directement l’URL et la clé. Si vous incluez ces arguments dans la ligne de commande en même temps que le fichier config, les arguments de la ligne de commande sont prioritaires.  

Pour l’ID de flux de travail 1001 et le fichier config.txt placé dans le même chemin que l’exécutable msgen, la commande doit ressembler à ceci :

```bash
msgen status -w 1001 -f "config.txt"
```

### <a name="2--examine-the-contents-of-standardoutputtxt"></a>2.  Examen du contenu de standardoutput.txt 
Recherchez le conteneur de sortie pour le flux de travail en question. MSGEN crée un dossier `[workflowfilename].logs.zip` après chaque exécution de flux de travail. Décompressez le dossier pour afficher son contenu :

* outputFileList.txt : liste des fichiers de sortie générés pendant le flux de travail
* standarderror.txt : ce fichier est vide.
* standardoutput.txt : consigne tous les messages d’état de niveau supérieur, notamment les erreurs, qui se sont affichés lors de l’exécution du flux de travail.
* Fichiers journaux GATK : tous les autres fichiers du dossier `logs`

Pour la résolution des problèmes, examinez le contenu de standardoutput.txt et notez les messages d’erreur qui s’affichent.


## <a name="step-2-try-recommended-steps-for-common-errors"></a>Étape 2 : Essayer les étapes recommandées pour les erreurs courantes

Cette section présente brièvement les résultats d’erreurs courantes obtenus par le service Microsoft Genomics (msgen) et les stratégies que vous pouvez utiliser pour les résoudre. 

Le service Microsoft Genomics (msgen) peut lever les deux types d’erreurs suivants :

1. Erreurs internes de service : erreurs qui sont internes au service et qui peuvent ne pas être résolues en corrigeant des paramètres ou des fichiers d’entrée. Un renvoi du flux de travail peut parfois corriger ces erreurs.
2. Erreurs d’entrée : erreurs qui peuvent être résolues en utilisant les arguments corrects ou en corrigeant des formats de fichiers.

### <a name="1-internal-service-errors"></a>1. Erreurs internes de service

Une erreur interne de service n’est pas actionnable par l’utilisateur. Vous pouvez renvoyer le flux de travail mais, si cela ne fonctionne pas, contactez le support Microsoft Genomics

| Message d’erreur                                                                                                                            | Procédure de résolution des problèmes recommandée                                                                                                                                   |
|------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Une erreur interne s’est produite. Essayez de renvoyer le flux de travail. Si vous voyez à nouveau cette erreur, contactez le support Microsoft Genomics pour obtenir de l’aide | Renvoyez le flux de travail. Contactez le support Microsoft Genomics pour obtenir de l’aide si le problème persiste en créant un [ticket](file-support-ticket-genomics.md ) de support. |

### <a name="2-input-errors"></a>2. Erreurs d’entrée

Ces erreurs sont actionnables par l’utilisateur. Selon le type de fichier et le code d’erreur, le service Microsoft Genomics génère des codes d’erreur distincts. Suivez les étapes de dépannage recommandées ci-dessous.

| Type de fichier | Code d'erreur | Message d’erreur                                                                           | Procédure de résolution des problèmes recommandée                                                                                         |
|--------------|------------|-----------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Quelconque          | 701        | La lecture [readId] a [numberOfBases] bases, mais la limite est de [maxReadLength]           | Cette erreur indique le plus souvent un endommagement des fichiers qui aboutit à la concaténation de deux lectures. Vérifiez vos fichiers d’entrée. |
| BAM          | 200        |   Impossible de lire le fichier « [NomVotreFichier] ».                                                                                       | Vérifiez le format du fichier BAM. Renvoyez le workflow avec un fichier correctement mis en forme.                                                                           |
| BAM          | 201        |  Impossible de lire le fichier BAM [nom_fichier].                                                                                      |Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                            |
| BAM          | 202        | Impossible de lire le fichier BAM [nom_fichier]. Fichier trop petit et en-tête manquant.                                                                                        | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                            |
| BAM          | 203        |   Impossible de lire le fichier BAM [nom_fichier]. L’en-tête du fichier a été endommagé.                                                                                      |Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                           |
| BAM          | 204        |    Impossible de lire le fichier BAM [nom_fichier]. L’en-tête du fichier a été endommagé.                                                                                     | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                           |
| BAM          | 205        |    Impossible de lire le fichier BAM [nom_fichier]. L’en-tête du fichier a été endommagé.                                                                                     | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                            |
| BAM          | 206        |   Impossible de lire le fichier BAM [nom_fichier]. L’en-tête du fichier a été endommagé.                                                                                      | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                            |
| BAM          | 207        |  Impossible de lire le fichier BAM [nom_fichier]. Fichier tronqué près de l’offset [offset].                                                                                       | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                            |
| BAM          | 208        |   Fichier BAM non valide. ReadID [Read_Id] n’a aucune séquence dans le fichier [nom_fichier].                                                                                      | Vérifiez le format du fichier BAM.  Envoyez le workflow avec un fichier correctement mis en forme.                                                                             |
| FASTQ        | 300        |  Impossible de lire le fichier FASTQ. [Nom_fichier] ne se termine pas par un saut de ligne.                                                                                     | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                           |
| FASTQ        | 301        |   Impossible de lire le fichier FASTQ [nom_fichier]. L’enregistrement FASTQ est supérieur à la taille de mémoire tampon à l’offset : [_offset]                                                                                      | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                         |
| FASTQ        | 302        |     Erreur de syntaxe FASTQ. Le fichier [nom_fichier] comporte une ligne vide.                                                                                    | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                         |
| FASTQ        | 303        |       Erreur de syntaxe FASTQ. Le fichier [nom_fichier] a un caractère de départ non valide à l’offset : [_offset], type de ligne : [type_ligne], caractère : [_char]                                                                                  | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                         |
| FASTQ        | 304      |  Erreur de syntaxe FASTQ au niveau de readID [_ReadID].  La première lecture du lot n’a pas readID qui se termine par /1 dans le fichier [nom_fichier]                                                                                       | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                         |
| FASTQ        | 305        |  Erreur de syntaxe FASTQ au niveau de readID [_readID]. La deuxième lecture du lot n’a pas readID qui se termine par /2 dans le fichier [nom_fichier]                                                                                      | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                          |
| FASTQ        | 306        |  Erreur de syntaxe FASTQ au niveau de readID [_ReadID]. La première lecture de la paire n’a pas d’ID qui se termine par /1 dans le fichier [nom_fichier]                                                                                       | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                          |
| FASTQ        | 307        |   Erreur de syntaxe FASTQ au niveau de readID [_ReadID]. ReadID ne se termine pas par /1 ni /2. Le fichier [nom_fichier] ne peut pas être utilisé comme fichier FASTQ associé.                                                                                      |Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                          |
| FASTQ        | 308        |  Erreur de lecture FASTQ. Les lectures des deux extrémités ont généré des réponses différentes. Avez-vous choisi les fichiers FASTQ corrects ?                                                                                       | Corrigez le format du fichier FASTQ et renvoyez le workflow.                                                                         |
|        |       |                                                                                        |                                                                           |

## <a name="step-3-contact-microsoft-genomics-support"></a>Étape 3 : Contacter le support Microsoft Genomics

Si vos tâches continuent d’échouer, ou si vous avez d’autres questions, contactez le support Microsoft Genomics sur le portail Azure. Pour en savoir plus sur la procédure d’envoi d’une demande de support, cliquez [ici](file-support-ticket-genomics.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à résoudre les problèmes courants liés au service Microsoft Genomics. Pour plus d’informations et pour consulter les questions fréquentes (FAQ) plus générales, consultez [Questions courantes](frequently-asked-questions-genomics.md). 
