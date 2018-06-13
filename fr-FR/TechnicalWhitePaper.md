# EOS.IO Technical White Paper v2

**Publication du 16 Mars 2018**

**Résumé:** Le logiciel EOS.IO introduit une nouvelle architecture blockchain conçue pour permettre le scaling vertical et horizontal d'applications décentralisées. Ceci est réalisé en créant une plateforme de type système d'exploitation sur laquelle les applications peuvent être développées. Le logiciel fournit les comptes, l'authentification, les bases de données, la communication asynchrone et l'ordonnancement des tâches sur les unités de traitement (Coeurs/Threads CPU). La technologie qui en résulte est une architecture blockchain pouvant potentiellement atteindre des millions de transactions à la seconde, élimine les frais d'utilisation et permet un déploiement et une maintenance rapides et faciles des applications décentralisées, le tout sur une blockchain gouvernée.

**VEUILLEZ NOTER QUE: LES JETONS CRYPTOGRAPHIQUES MENTIONNÉS DANS CE WHITEPAPER FONT RÉFÉRENCE AUX JETONS CRYPTOGRAPHIQUES SUR UNE BLOCKCHAIN LANCÉE QUI ADOPTE LE LOGICIEL EOS.IO. ILS NE FONT PAS RÉFÉRENCE AUX JETONS COMPATIBLES ERC-20 QUI SONT DISTRIBUÉS SUR LA BLOCKCHAIN D'ÉTHÉRÉUM EMPLOYÉS DANS LE CADRE DE LA DISTRIBUTION DES JETONS EOS.**

Copyright © 2018 block.one

Sans autorisation, quiconque peut utiliser, reproduire ou distribuer tout matériel contenu dans le présent whitepaper à des fins non commerciales et éducatives (c.-à-d. à des fins autres que des lucratives ou  des fins commerciales), pourvu que la source originale et l'avis de droit d'auteur applicable soient cités.


**AVERTISSEMENT:** Le présent whitepaper technique EOS.IO v2 n'est fourni qu'à titre d'information. block.one ne garantit pas l'exactitude ou les conclusions de ce whitepaper, et ce whitepaper est fourni "tel quel". block.one ne fait pas et rejette expressément toutes les représentations et garanties, expresses, implicites, statutaires ou autres, de quelque nature que ce soit, y compris, mais sans s'y limiter: (i) les garanties de qualité marchande, d'adaptation à un usage particulier, de convenance, d'usage, de titre ou de non-violation; (ii) que le contenu de ce whitepaper est exempt d'erreur; et (iii) que ce contenu ne portera pas atteinte aux droits des tiers. block.one et ses affiliés ne peuvent être tenus responsables des dommages de toute sorte découlant de l'utilisation, de la référence ou de la confiance dans ce whitepaper ou dans le contenu de ce dernier, même s'ils sont conscients de la possibilité de tels dommages. En aucun cas Block.one ou ses affiliés ne seront responsables envers toute personne ou entité pour tous dommages, pertes, responsabilités, coûts ou dépenses de toute nature, qu'ils soient directs ou indirects, consécutifs, compensatoires, accessoires, réels, exemplaires, punitifs ou spéciaux pour l'utilisation, la référence à, ou la confiance dans ce whitepaper ou tout contenu contenu de ce whitepaper, y compris, sans limitation, toute perte d'affaires, revenus, profits, données, accès, avantages ou autres pertes intangibles.

<!-- MarkdownTOC depth=4 autolink=true bracket=round list_bullets="-*+" -->

- [Contexte](#contexte)
- [Pré-requis des applications Blockchain](#pré-requis-des-applications-blockchain)
  * [Le support de millions d'utilisateurs](#le-support-de-millions-dutilisateurs)
  * [Utilisation gratuite](#utilisation-gratuite)
  * [Mises à jour faciles et correction des bogues](#mises-à-jour-faciles-et-correction-des-bogues)
  * [Faible latence](#faible-latence)
  * [Performance séquentielle](#performance-séquentielle)
  * [Performance parallèle](#performance-parallèle)
- [Algorithme de consensus \(BFT-DPOS\)](#algorithme-de-consensus-bft-dpos)
  * [Confirmation de la transaction](#confirmation-de-la-transaction)
  * [Transaction comme preuve de participation \(TaPoS\)](#transaction-comme-preuve-de-participation-tapos)
- [Comptes](#comptes)
  * [Actions & Gestionnaires](#actions--gestionnaires)
  * [Gestion des permissions basée sur les rôles](#gestion-des-permissions-basée-sur-les-rôles)
    + [Niveaux de permission nommés](#niveaux-de-permission-nommés)
    + [Cartographie des permissions](#cartographie-des-permissions)
    + [Évaluation des permissions](#Évaluation-des-permissions)
      - [Groupes de permissions par défaut](#groupes-de-permissions-par-défaut)
      - [Évaluation parallèle des autorisations](#Évaluation-parallèle-des-autorisations)
  * [Actions avec délai obligatoire](#actions-avec-délai-obligatoire)
  * [Récupération des clés volées](#récupération-des-clés-volées)
- [Exécution parallèle déterministe des applications](#exécution-parallèle-déterministe-des-applications)
  * [Minimiser la latence de la communication](#minimiser-la-latence-de-la-communication)
  * [Manipulateurs d'action en lecture seule](#manipulateurs-daction-en-lecture-seule)
  * [Transactions atomiques avec plusieurs comptes](#transactions-atomiques-avec-plusieurs-comptes)
  * [Évaluation partielle de l'état de la Blockchain](#Évaluation-partielle-de-létat-de-la-blockchain)
  * [Ordonnancement subjectif](#ordonnancement-subjectif)
  * [Transactions différées](#transactions-différées)
  * [Actions exemptes de contexte](#actions-exemptes-de-contexte)
- [Modèle de jeton et utilisation des ressources](#modèle-de-jeton-et-utilisation-des-ressources)
  * [Mesures objectives et subjectives](#mesures-objectives-et-subjectives)
  * [Paiement par le destinataire](#paiement-par-le-destinataire)
  * [Capacité de délégation](#capacité-de-délégation)
  * [Indépendance des coûts de transaction et de la valeur des jetons](#indépendance-des-coûts-de-transaction-et-de-la-valeur-des-jetons)
  * [Coûts de stockage](#coûts-de-stockage)
  * [Récompenses des blocs](#récompenses-des-blocs)
  * [Système de proposition de travail](#système-de-proposition-de-travail)
- [Gouvernance](#gouvernance)
  * [Gel des comptes](#gel-des-comptes)
  * [Modification du code de compte](#modification-du-code-de-compte)
  * [Constitution](#constitution)
  * [Mise à jour du protocole et de la constitution](#mise-à-jour-du-protocole-et-de-la-constitution)
    + [Changements d'urgence](#changements-durgence)
- [Scripts et machines virtuelles](#scripts-et-machines-virtuelles)
  * [Actions définies par schéma](#actions-définies-par-schéma)
  * [Base de données définie par schéma](#base-de-données-définie-par-schéma)
  * [API générique de base de données multi-index](#api-générique-de-base-de-données-multi-index)
  * [Séparation de l'authentification et de l'application](#séparation-de-lauthentification-et-de-lapplication)
- [Communication Inter-Blockchains](#communication-inter-blockchains)
  * [Preuves de Merkle pour la validation des clients légers \(LCV\)](#preuves-de-merkle-pour-la-validation-des-clients-légers-lcv)
  * [Latence de la communication interchaînes](#latence-de-la-communication-interchaînes)
  * [Preuve de finalité](#preuve-de-finalité)
  * [Témoins séparés \(Segwit\)](#témoins-séparés-segwit)
- [Conclusion](#conclusion)

<!-- /MarkdownTOC -->

# Contexte 

La technologie Blockchain a été introduite en 2008 avec le lancement de la monnaie Bitcoin, et depuis lors, les entrepreneurs et les développeurs ont tenté de généraliser la technologie pour prendre en charge un plus large éventail d'applications sur une seule plateforme Blockchain.

Alors qu'un certain nombre de plateformes blockchain ont eu du mal à prendre en charge les applications fonctionnelles décentralisées, des blockchains spécifiques à des applications telles que l'échange décentralisé de BitShares (2014) et la plateforme sociale Steem (2016) sont devenues des blockchains très utilisées avec des dizaines de milliers d'utilisateurs actifs au quotidien. Ils y sont parvenus en augmentant la performance à des milliers de transactions par seconde, en réduisant la latence à 1,5 seconde, en éliminant les frais par transaction et en offrant une expérience utilisateur similaire à celle actuellement offerte par les services centralisés existants.

Les plateformes blockchain existantes sont grevées par des frais importants et une capacité de calcul limitée qui empêchent l'adoption généralisée de la blockchain.

# Pré-requis des applications Blockchain

Afin de voir leur utilisation se généraliser, les applications blockchain requièrent une plateforme suffisamment flexible pour répondre aux exigences suivantes:

## Le support de millions d'utilisateurs

Pour concurrencer des entreprises comme eBay, Uber, AirBnB et Facebook, il faut une technologie blockchain capable de supporter des dizaines de millions d'utilisateurs quotidiens. Dans certains cas, une application peut ne pas fonctionner tant qu'une masse critique d'utilisateurs n'est pas atteinte. Il est clair qu'une plateforme capable de traiter un très grand nombre d'utilisateurs est donc primordiale.

## Utilisation gratuite

Les développeurs d'applications ont besoin de flexibilité pour offrir aux utilisateurs des services gratuits; les utilisateurs ne devraient pas avoir à payer pour utiliser la plateforme ou bénéficier de ses services. Une plateforme blockchain dont l'utilisation est gratuite pour les utilisateurs sera probablement adoptée à plus grande échelle. Les promoteurs et les entreprises peuvent alors créer des stratégies de monétisation efficaces.

## Mises à jour faciles et correction des bogues

Les entreprises qui construisent des applications basées sur une blockchain ont besoin de flexibilité pour améliorer leurs applications avec de nouvelles fonctionnalités. La plateforme doit prendre en charge les mises à niveau logicielles et des smartcontracts.

Tous les logiciels non triviaux sont sujets à des bogues, même avec la plus rigoureuse des vérifications formelles. La plateforme doit être suffisamment robuste pour permettre la correction des bogues lorsqu'ils surviennent inévitablement.

## Faible latence

Une bonne expérience utilisateur exige un temps de réponse fiable avec un délai ne dépassant pas quelques secondes. Des délais plus longs frustrent les utilisateurs et rendent les applications construites sur blockchain moins compétitives par rapport aux alternatives existantes ne fonctionnant pas sur blockchain. La plateforme devrait supporter une faible latence des transactions.

## Performance séquentielle

Il y a des applications qui ne peuvent tout simplement pas être implémentées avec des algorithmes parallèles en raison d'étapes séquentiellement dépendantes. Les applications telles que les plateformes d'échanges ont besoin d'une performance séquentielle suffisante pour traiter des volumes élevés. Par conséquent, la plateforme doit offrir des performances séquentielles rapides.

## Performance parallèle

Les applications à grande échelle doivent répartir la charge de travail entre de multiples CPU et ordinateurs.

# Algorithme de consensus (BFT-DPOS)

Le logiciel EOS.IO utilise le seul algorithme de consensus décentralisé connu qui s'est avéré capable de répondre aux exigences de performance des applications blockchain, [la preuve d'enjeu déléguée (DPOS)](https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper). Selon cet algorithme, ceux qui détiennent des jetons sur une blockchain adoptant le logiciel EOS.IO peuvent sélectionner des producteurs de blocs par le biais d'un système de vote d'approbation continu. Toute personne peut choisir de participer à la production de blocs et aura la possibilité de produire des blocs, à condition qu'elle puisse persuader les détenteurs de jetons de voter pour eux.

Le logiciel EOS.IO permet de produire des blocs exactement toutes les 0,5 seconde et un seul producteur est autorisé à produire un bloc à un moment donné. Si le bloc n'est pas produit à l'heure prévue, le bloc pour cet interval de temps est ignoré. Lorsqu'un ou plusieurs blocs sont sautés, il y a un écart de 0,5 ou plus d'une seconde dans la blockchain.

En utilisant le logiciel EOS.IO, les blocs sont produits en séries de 126 (6 blocs chacun, fois 21 producteurs). Au début de chaque tour, 21 producteurs de blocs uniques sont choisis en fonction de la préférence des votes exprimés par les détenteurs de jetons. Les producteurs sélectionnés sont programmés selon un ordre convenu par 15 producteurs ou plus.

Si un producteur saute un bloc et n'en a produit aucun dans les dernières 24 heures, il est retiré de la liste des producteurs jusqu'à ce qu'il avise la chaîne de blocs de son intention de recommencer à produire des blocs. Cela permet d'assurer le bon fonctionnement du réseau en minimisant le nombre de blocs manqués par les producteurs absents, dont le manque de fiabilité est avéré.

Dans des conditions normales, une chaîne de blocs DPOS ne connaît pas de forks car, au lieu d'être en concurrence, les producteurs de blocs coopèrent pour produire des blocs. Dans le cas où il y a un fork, le consensus passera automatiquement à la chaîne la plus longue. Cette méthode fonctionne parce que le taux auquel les blocs sont ajoutés à un fork de la blockchain est directement corrélé au pourcentage de producteurs de blocs qui partagent le même consensus. En d'autres termes, un fork de blockchain avec plus de producteurs croîtra en longueur plus rapidement qu'un fork avec moins de producteurs, parce que le fork avec plus de producteurs connaîtra moins de blocs manqués.

En outre, aucun producteur de blocs ne devrait produire des blocs sur deux fourches en même temps. Un producteur de blocs pris en train de faire cela sera probablement éliminé. Les preuves cryptographiques de cette double production peuvent également être utilisées pour éliminer automatiquement les fraudeurs.

La tolérance de faute byzantine est ajoutée au DPOS traditionnel en permettant à tous les producteurs de signer tous les blocs à condition qu'aucun producteur ne signe deux blocs avec le même horodatage ou la même hauteur de bloc. Une fois que 15 producteurs ont signé un bloc, le bloc est considéré comme irréversible. Tout producteur byzantin devrait générer des preuves cryptographiques de sa trahison en signant deux blocs avec le même horodatage ou la même hauteur de bloc. Selon ce modèle, un consensus irréversible devrait pouvoir être atteint en moins d'une seconde.

## Confirmation de la transaction

Les blockchains DPOS classiques ont une participation de 100 % des producteurs de blocs. Une transaction peut être considérée comme confirmée avec une certitude de 99,9% après une moyenne de 0,25 secondes à partir de l'heure de diffusion.

En plus du DPOS, EOS.IO ajoute la tolérance aux fautes byzantines asynchrones (aBFT) pour une réalisation plus rapide de l'irréversibilité. L'algorithme aBFT confirme à 100% l'irréversibilité en 1 seconde.

## Transaction comme preuve de participation (TaPoS)

Le logiciel EOS.IO exige que chaque transaction comprenne une partie du hashage d'un en-tête de bloc récent. Ce hashage sert à deux fins:

1. empêche de rejouer une transaction sur des fourches qui n'incluent pas le bloc référencé; et
2. signale au réseau qu'un utilisateur particulier et sa mise sont sur un fork spécifique.

Au fil du temps, tous les utilisateurs finissent par confirmer directement la blockchain, ce qui rend difficile la falsification des chaînes de contrefaçon, car la contrefaçon ne serait pas en mesure de migrer les transactions de la chaîne légitime.

# Comptes

Le logiciel EOS.IO permet à tous les comptes d'être référencés par un nom unique lisible par l'homme d'une longueur maximale de 12 caractères. Le nom est choisi par le créateur du compte. Le créateur du compte doit réserver la RAM nécessaire pour stocker le nouveau compte jusqu'à ce que le nouveau compte engage des jetons pour réserver sa propre RAM.

Dans un contexte décentralisé, les développeurs d'applications paieront le coût nominal de la création d'un compte pour inscrire un nouvel utilisateur. Les entreprises traditionnelles dépensent déjà des sommes importantes par client qu'elles acquièrent sous forme de publicité, de services gratuits, etc. Le coût de financement d'un nouveau compte en bloc devrait être insignifiant en comparaison. Heureusement, il n'est pas nécessaire de créer des comptes pour les utilisateurs déjà inscrits par une autre application.

## Actions & Gestionnaires

Chaque compte peut envoyer des Actions structurées à d'autres comptes et peut définir des scripts pour gérer les Actions lorsqu'elles sont reçues. Le logiciel EOS.IO donne à chaque compte sa propre base de données privée qui n'est accessible que par ses propres gestionnaires d'actions. Les scripts de gestion des actions peuvent également envoyer des actions à d'autres comptes. La combinaison d'Actions et de gestionnaires d'actions automatisés permet à EOS.IO de définir des smartcontracts.

Pour prendre en charge l'exécution parallèle, chaque compte peut également définir un nombre quelconque de portées dans leur base de données. Les producteurs de blocs ordonnanceront la transaction de telle sorte qu'il n'y ait pas de conflit sur l'accès à la mémoire des champs et qu'ils puissent donc être exécutés en parallèle.

## Gestion des permissions basée sur les rôles

La gestion des permissions implique de déterminer si une Action est bien autorisée ou non. La forme la plus simple de gestion des permissions consiste à vérifier qu'une transaction possède les signatures requises, mais cela implique que les signatures requises soient déjà connues. En général, l'autorité est liée à des individus ou à des groupes d'individus et est souvent compartimentée. Le logiciel EOS.IO fournit un système de gestion des autorisations déclaratives qui donne aux comptes un contrôle de haut niveau sur qui peut faire quoi et quand.

Il est essentiel que l'authentification et la gestion des permissions soient normalisées et distinguées de l'aspect (la logique) "business" de l'application. Cela permet de développer des outils pour gérer les permissions d'une manière générale et fournit également des opportunités significatives pour l'optimisation des performances.

Chaque compte peut être contrôlé par une combinaison pondérée d'autres comptes et de clés privées. Cela crée une structure d'autorité hiérarchique qui reflète la façon dont les permissions sont organisées en réalité et rend le contrôle multi-utilisateurs sur les comptes plus facile que jamais. Le contrôle multi-utilisateurs est le plus grand contributeur à la sécurité et, lorsqu'il est utilisé correctement, il peut réduire considérablement le risque de vol dû au piratage.

Le logiciel EOS.IO permet aux comptes de définir quelle combinaison de clés et/ou de comptes peut envoyer un type d'action particulier à un autre compte. Par exemple, il est possible d'avoir une clé pour le compte d'un réseau social d'un utilisateur et une autre pour l'accès à la plateforme d'échanges. Il est même possible de donner à d'autres comptes la permission d'agir au nom du compte d'un utilisateur sans leur attribuer de clés.

### Niveaux de permission nommés

<img align="right" src="https://github.com/EOSIO/Documentation/blob/images/images/diagram3.png" width="228.395px" height="300px" />

En utilisant le logiciel EOS.IO, les comptes peuvent définir des niveaux de permission nommés dont chacun peut être dérivé de permissions nommées de niveau supérieur. Chaque niveau d'autorisation nommé définit une autorité; une autorité est un contrôle multi-signature de seuil composé de clés et/ou de niveaux d'autorisation nommés d'autres comptes. Par exemple, le niveau d'autorisation "Ami" d'un compte peut être défini pour qu'une Action sur le compte soit contrôlée de manière égale par n'importe quel ami du compte.

Un autre exemple est la blockchain de Steem qui a trois niveaux de permission nommés et codés en dur: propriétaire, actif et affichage. La permission d'affichage ne peut effectuer que des actions sociales telles que le vote et l'affichage, alors que la permission active peut tout faire sauf changer le propriétaire. L'autorisation du propriétaire est destinée au "cold-storage" et permet toutes les options. Le logiciel EOS.IO généralise ce concept en permettant à chaque titulaire de compte de définir sa propre hiérarchie ainsi que le regroupement des actions.

### Cartographie des permissions

Le logiciel EOS.IO permet à chaque compte de définir un mappage entre un contrat/action ou un contrat de tout autre compte et leur propre niveau d'autorisation nommé. Par exemple, un titulaire de compte pourrait mettre en correspondance l'application de réseau social du titulaire de compte avec le groupe d'autorisation "Ami" du titulaire de compte. Avec ce mappage, n'importe quel ami pourrait poster en tant que titulaire du compte sur les réseaux sociaux du titulaire du compte. Même s'ils postent en tant que titulaire du compte, ils utiliseraient toujours leurs propres clés pour signer l'Action. Cela signifie qu'il est toujours possible d'identifier les amis qui ont utilisé le compte, ainsi que la manière employée pour y parvenir.

### Évaluation des permissions

Lors de l'exécution d'une action de type "**Action**", de **@alice** à **@bob**, le logiciel EOS.IO vérifie d'abord si **@alice** a défini un mappage de permissions pour **@bob.groupa.subgroup.Action**. Si rien n'est trouvé, alors un mappage pour **@bob.groupa.subgroup** puis **@bob.groupa**, et enfin **@bob** sera vérifié. Si aucune autre correspondance n'est trouvée, le mappage présumé se fera vers le groupe de permissions nommé @**alice.active**.

Une fois qu'un mappage est identifié, le pouvoir de signature est validé à l'aide du processus de signature multiple de seuil et de l'autorité associée à la permission nommée. Si cela échoue, alors il traverse jusqu'à la permission parent et finalement jusqu'à la permission propriétaire, **@alice.owner**.

<img align="center" src="https://github.com/EOSIO/Documentation/blob/images/images/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### Groupes de permissions par défaut

La technologie EOS.IO permet également à tous les comptes d'avoir un groupe "propriétaire" qui peut tout faire, et un groupe "actif" qui peut tout faire sauf changer le groupe propriétaire. Tous les autres groupes de permission sont dérivés de "active".

#### Évaluation parallèle des autorisations

Le processus d'évaluation des permissions est "en lecture seule" et les modifications apportées aux permissions par les transactions ne prennent effet qu'à la fin d'un bloc. Cela signifie que toutes les clés et l'évaluation des autorisations pour toutes les transactions peuvent être exécutées en parallèle. De plus, cela signifie qu'une validation rapide de l'autorisation est possible sans commencer une logique d'application coûteuse qui devrait être annulée. Enfin, cela signifie que les autorisations de transaction peuvent être évaluées au fur et à mesure que les transactions en attente sont reçues et n'ont pas besoin d'être réévaluées au fur et à mesure qu'elles sont appliquées.

Tout bien considéré, la vérification des autorisations représente un pourcentage important du calcul nécessaire pour valider les transactions. Faire de ce processus un processus en lecture seule et trivialement parallélisable permet une augmentation spectaculaire de la performance.

Lorsque l'on relit la chaîne de blocs pour régénérer l'état déterministe à partir du journal des Actions, il n'est pas nécessaire d'évaluer à nouveau les permissions. Le fait qu'une transaction soit incluse dans un bloc valide connu est suffisant pour sauter cette étape. Cela réduit considérablement la charge de calcul associée à la relecture d'une chaîne de blocs en constante expansion.

## Actions avec délai obligatoire

Le temps est un élément essentiel de la sécurité. Dans la plupart des cas, il n'est pas possible de savoir si une clé privée a été volée tant qu'elle n'a pas été utilisée. La sécurité basée sur le temps est encore plus critique lorsque les gens ont des applications qui exigent que les clés soient conservées sur des ordinateurs connectés à Internet pour une utilisation quotidienne. Le logiciel EOS.IO permet aux développeurs d'applications d'indiquer que certaines Actions doivent attendre un temps minimum après avoir été incluses dans un bloc avant de pouvoir être appliquées. Pendant ce temps, elles peuvent être annulés.

Les utilisateurs peuvent ensuite recevoir un avis par email ou par SMS lorsqu'une de ces Actions est diffusée. S'ils ne l'ont pas autorisée, ils peuvent utiliser le processus de recouvrement de compte pour récupérer leur compte et annuler l'Action.

Le délai requis dépend de la sensibilité d'une opération. Le paiement d'un café peut se faire sans délai et être irréversible en quelques secondes, tandis que l'achat d'une maison peut nécessiter une période de recours de 72 heures. Le transfert d'un compte entier à un nouveau contrôle peut prendre jusqu'à 30 jours. Les délais exacts sont choisis par les développeurs d'applications et les utilisateurs.

## Récupération des clés volées

Le logiciel EOS.IO permet aux utilisateurs de rétablir le contrôle de leur compte en cas de vol de clés. Un propriétaire de compte peut utiliser n'importe quelle clé de propriétaire qui était active au cours des 30 derniers jours avec l'approbation de son partenaire de recouvrement de compte désigné pour réinitialiser la clé de propriétaire de son compte. Le partenaire de recouvrement de compte ne peut pas réinitialiser le contrôle du compte sans l'aide du propriétaire.

Il n'y a rien à gagner pour le pirate informatique en essayant de passer par le processus de récupération parce qu'ils "contrôlent" déjà le compte. De plus, s'ils passent par le processus, le partenaire de récupération exigera probablement l'identification et l'authentification multifactorielle (téléphone et courriel). Cela compromettrait probablement le pirate ou ne lui rapporterait rien dans le processus.

Ce processus est également très différent d'un simple arrangement multi-signature. Dans le cas d'une transaction à signatures multiples, une autre entité devient partie à chaque transaction exécutée. En revanche, avec le processus de récupération, le partenaire de récupération n'est qu'une partie au processus de récupération et n'a aucun pouvoir sur les transactions quotidiennes. Cela réduit considérablement les coûts et les responsabilités légales pour toutes les personnes concernées.


# Exécution parallèle déterministe des applications

Le consensus sur la blockchain dépend d'un comportement déterministe (reproductible). Cela signifie que toute exécution parallèle doit être exempte d'utilisation d'exclusions mutuelles ("mutexes") ou d'autres primitives de verrouillage ("locking primitives"). Sans verrouillage, il doit y avoir un moyen de garantir que les transactions qui peuvent être exécutées en parallèle ne créent pas de résultats non déterministes.

La version de juin 2018 du logiciel EOS.IO s'exécutera sur un seul thread, mais elle contient les structures de données nécessaires à une future exécution parallèle et multithread.

Dans une blockchain basée sur le logiciel EOS.IO, une fois le fonctionnement en parallèle activé, il incombera au producteur du bloc d'organiser la livraison de l'action en fragments ("shards") indépendants afin qu'ils puissent être évalués en parallèle. L'ordre de traitement est la production d'un producteur de blocs et sera exécuté de façon déterministe, mais le processus de génération de cet ordre n'a pas besoin d'être déterministe. Cela signifie que les producteurs de blocs peuvent utiliser des algorithmes parallèles pour planifier les transactions.

Une partie de l'exécution parallèle signifie que lorsqu'un script génère une nouvelle action, il n'est pas livré immédiatement, mais qu'il est programmé pour être livré dans le cycle suivant. La raison pour laquelle il ne peut pas être livré immédiatement est que le récepteur peut modifier activement son propre état dans un autre fragment ("shard").


## Minimiser la latence de la communication

La latence est le temps qu'il faut pour qu'un compte envoie une Action à un autre compte et reçoive une réponse. Le but est de permettre à deux comptes d'échanger des Actions dans un même bloc sans avoir à attendre 0,5 seconde entre chaque Action. Pour ce faire, le logiciel EOS.IO divise chaque bloc en cycles. Chaque cycle est divisé en fragments ("shards") et chaque fragment contient une liste de transactions. Chaque transaction contient un ensemble d'actions à livrer. Cette structure peut être visualisée comme une arborescence où les couches alternées sont traitées séquentiellement et en parallèle.

      Bloc

        Région

          Cycles (séquentiels)

            Fragments (parallèles)

              Transactions (séquentielles)

                Actions (séquentielles)

                  Comptes récepteurs et comptes notifiés (parallèles)

Les transactions générées dans un cycle peuvent être livrées dans n'importe quel cycle ou bloc suivant. Les producteurs de blocs continueront d'ajouter des cycles à un bloc jusqu'à ce que le temps maximum alloué soit écoulé ou jusqu'à ce qu'il n'y ait plus de nouvelles transactions générées à livrer.

Il est possible d'utiliser l'analyse statique d'un bloc pour vérifier qu'à l'intérieur d'un cycle donné, il n'y a pas deux fragments contenant des transactions qui modifient le même compte. Tant que cet invariant est maintenu, un bloc peut être traité en exécutant tous les fragment en parallèle.


## Manipulateurs d'action en lecture seule

Certains comptes peuvent être en mesure de traiter une Action sur une base réussite/échec sans modifier leur état interne. Si tel est le cas, alors ces gestionnaires peuvent être exécutés en parallèle, à condition que seuls les gestionnaires d'actions en lecture seule pour un compte particulier soient inclus dans un ou plusieurs fragments d'un cycle particulier.

## Transactions atomiques avec plusieurs comptes

Il est parfois souhaitable de s'assurer que les Actions soient livrées et acceptées par plusieurs comptes atomiquement. Dans ce cas, les deux Actions sont placées dans une seule transaction et les deux comptes se verront attribuer le même fragment et les Actions traitées séquentiellement.

## Évaluation partielle de l'état de la Blockchain

L'expansion d'une blockchain nécessite que les composants soient modulaires. Tout le monde ne devrait pas avoir à tout exécuter, surtout si les composants n'ont besoin que d'un petit sous-ensemble d'applications.

Un développeur de plateforme d'échanges exécute des nœuds complets dans le but d'afficher l'état de l'échange à ses utilisateurs. Cette application d'échange n'a pas besoin de l'état associé aux applications de réseaux sociaux. Le logiciel EOS.IO permet à n'importe quel nœud complet de choisir n'importe quel sous-ensemble d'applications à exécuter. Les actions livrées à d'autres applications sont ignorées en toute sécurité si votre application ne dépend jamais de l'état d'un autre contrat.


## Ordonnancement subjectif 

Le logiciel EOS.IO ne peut pas obliger les producteurs de blocs à livrer une action à un autre compte. Chaque producteur de blocs fait sa propre mesure subjective de la complexité de calcul et du temps requis pour le traitement d'une transaction. Ceci s'applique, qu'une transaction soit générée par un utilisateur ou automatiquement par un smartcontract.

Sur une blockchain lancée adoptant le logiciel EOS.IO, au niveau du réseau, toutes les transactions sont facturées à un coût de bande passante informatique basé sur le nombre d'instructions WASM exécutées. Toutefois, chaque producteur de blocs utilisant le logiciel peut calculer l'utilisation des ressources à l'aide de son propre algorithme et de ses propres mesures. Lorsqu'un producteur de blocs conclut qu'une transaction ou un compte a consommé une quantité disproportionnée de la capacité de calcul, il rejette simplement la transaction lorsqu'il produit son propre bloc; toutefois, il continuera de traiter la transaction si d'autres producteurs de blocs la jugent valide.

En général, tant qu'un seul producteur de blocs considère une transaction comme valide et que les limites d'utilisation des ressources sont respectées, tous les autres producteurs de blocs l'accepteront également, mais la transaction peut prendre jusqu'à une minute pour trouver ce producteur.

Dans certains cas, un producteur peut créer un bloc qui comprend des transactions dont l'ordre de grandeur se situe à l'extérieur des limites acceptables. Dans ce cas, le producteur de blocs suivant peut choisir de rejeter le bloc et l'égalité sera brisée par le troisième producteur. Ce n'est pas différent de ce qui se produirait si un gros bloc causait des ralentissements du réseau. La communauté remarquerait un modèle d'abus et finirait par retirer les votes du producteur malhonnête.

Cette évaluation subjective du coût de calcul libère la blockchain de la nécessité de mesurer avec précision et de façon déterministe le temps de fonctionnement d'une opération. Avec cette conception, il n'est pas nécessaire de compter précisément les instructions, ce qui augmente considérablement les possibilités d'optimisation sans rompre le consensus.

## Transactions différées

Le logiciel EOS.IO prend en charge les transactions différées dont l'exécution est prévue à l'avenir. Cela permet au calcul de passer d'un fragment ("sharding") à l'autre et/ou de créer des processus à long terme qui planifient continuellement une transaction de continuation.

## Actions exemptes de contexte

Une action sans contexte implique des calculs qui dépendent uniquement des données de transaction, mais pas de l'état de la blockchain. La vérification de signature, par exemple, est un calcul qui ne requiert que les données de la transaction et une signature pour déterminer la clé publique qui a signé la transaction. C'est l'un des calculs individuels les plus coûteux qu'une blockchain doit effectuer, mais comme ce calcul est sans contexte, il peut être effectué en parallèle.

Les actions libres de contexte sont comme les autres actions de l'utilisateur, sauf qu'elles n'ont pas accès à l'état de la blockchain pour effectuer la validation. Non seulement cela permet à EOS.IO de traiter en parallèle toutes les actions sans contexte telles que la vérification des signatures, mais plus important encore, cela permet une vérification généralisée des signatures.

Avec la prise en charge des actions sans contexte, les techniques d'évolutivité telles que le Sharding, Raiden, Plasma, les States Channels et autres deviennent beaucoup plus parallélisables et pratiques. Ce développement permet une communication inter-blockchains efficace et une évolutivité potentiellement illimitée.

# Modèle de jeton et utilisation des ressources

**VEUILLEZ NOTER: LES JETONS CRYPTOGRAPHIQUES MENTIONNÉS DANS CE WHITEPAPER FONT RÉFÉRENCE AUX JETONS CRYPTOGRAPHIQUES SUR UNE BLOCKCHAIN LANCÉE QUI ADOPTE LE LOGICIEL EOS.IO. ILS NE FONT PAS RÉFÉRENCE AUX JETONS COMPATIBLES ERC-20 QUI SONT DISTRIBUÉS SUR LA BLOCKCHAIN D'ÉTHÉRÉUM EN RELATION AVEC LA DISTRIBUTION DES JETONS EOS.**

Toutes les blockchain sont limitées en ressources et nécessitent un système pour prévenir les abus. Avec une blockchain qui utilise le logiciel EOS.IO, il y a trois grandes catégories de ressources qui sont consommées par les applications:

1. Bande passante et stockage des logs (disque);
2. Calculs et historique de calcul (CPU); et
3. Stockage Actif (RAM).

La bande passante et le calcul ont deux composantes: l'utilisation instantanée et l'utilisation à long terme. Une blockchain tient un journal de toutes les actions et ce journal est finalement stocké et téléchargé par tous les nœuds complets. Avec le journal des Actions, il est possible de reconstruire l'état de toutes les applications.

La dette de calcul est un ensemble de calculs qui doivent être effectués pour régénérer l'état à partir du journal des actions. Si la dette de calcul devient trop importante, il devient nécessaire de prendre des captures de l'état de la blockchain et de se débarrasser de l'historique de cette dernière. Si la dette  de calcul augmente trop rapidement, il peut prendre jusqu'à 6 mois pour reproduire les transactions d'une année complète. Il est donc essentiel que la dette de calcul soit gérée avec rigueur.

Le stockage de la blockchain est une information accessible, grâce à l'application. Il comprend des informations telles que les carnets de commandes et les soldes de comptes. Si l'état n'est jamais lu par l'application, il ne doit pas être stocké. Par exemple, le contenu d'un blog et les commentaires ne sont pas lus par la logique de l'application, ils ne doivent donc pas être sauvegardés dans la mémoire dédiée de la blockchain. Entre-temps, l'existence d'un poste/commentaire, le nombre de votes et d'autres propriétés sont stockés la mémoire de la blockchain.

Les producteurs de blocs publient leur capacité disponible en terme de bande passante, de calcul et d'état. Le logiciel EOS.IO permet à chaque compte de consommer un pourcentage de la capacité disponible proportionnel à la quantité de jetons détenus dans un contrat de dépôt long de 3 jours. Par exemple, si une blockchain basée sur le logiciel EOS.IO est lancée et si un compte détient 1% du total des jetons distribuables en vertu de cette chaîne de blocs, alors ce compte a le potentiel d'utiliser 1% de la capacité de stockage disponible.

L'adoption du logiciel EOS.IO sur une blockchain lancée signifie que la bande passante et la capacité de calcul sont allouées sur la base d'une réserve fractionnaire parce qu'elles sont transitoires (la capacité inutilisée ne peut pas être conservée pour une utilisation future). L'algorithme utilisé par le logiciel EOS.IO est similaire à l'algorithme utilisé par Steem pour limiter l'utilisation de la bande passante.

## Mesures objectives et subjectives

Comme évoqué précédemment, l'instrumentalisation de la capacité de calcul a un impact significatif sur la performance et l'optimisation; par conséquent, toutes les contraintes d'utilisation des ressources sont en fin de compte subjectives et l'application de la loi est faite par les producteurs de blocs en fonction de leurs algorithmes et estimations individuels. Ceux-ci sont généralement implémentés par un producteur de blocs via l'élaboration d'un plugin personnalisé.

Cela dit, il y a certaines choses qui sont triviales à mesurer objectivement. Le nombre d'Actions livrées et la taille des données stockées dans la base de données interne sont bon marché à mesurer objectivement. Le logiciel EOS.IO permet aux producteurs de blocs d'appliquer le même algorithme sur ces mesures objectives mais peut choisir d'appliquer des algorithmes subjectifs plus stricts sur des mesures subjectives.

## Paiement par le destinataire

Traditionnellement, c'est l'entreprise qui paie pour les locaux, la puissance de calcul et les autres frais nécessaires au fonctionnement de l'entreprise. Le client achète des produits spécifiques à l'entreprise et les revenus provenant de la vente de ces produits sont utilisés pour couvrir les coûts d'exploitation de l'entreprise. De même, aucun site Web n'oblige ses visiteurs à effectuer des micropaiements pour visiter son site Web afin de couvrir les frais d'hébergement. Par conséquent, les applications décentralisées ne devraient pas forcer leurs clients à payer directement la blockchain pour l'utilisation de cette dernière.

Une blockchain lancée sur le logiciel EOS.IO n'exige pas que ses utilisateurs paient directement la blockchain pour son utilisation et n'empêche donc pas une entreprise de déterminer sa propre stratégie de monétisation pour ses produits.

Bien qu'il soit vrai que le récepteur peut payer, EOS.IO permet à l'expéditeur de payer pour la bande passante, le calcul et le stockage. Cela permet aux développeurs d'applications de choisir le mode convenant le mieux à leur application. Dans de nombreux cas, le paiement par l'expéditeur réduit considérablement la complexité pour les développeurs d'applications qui ne veulent pas mettre en œuvre leur propre système de tarification. Les développeurs d'applications peuvent déléguer la bande passante et le calcul à leurs utilisateurs, puis laisser le modèle " expéditeur-payeur " faire respecter l'usage. Du point de vue de l'utilisateur final, c'est gratuit, mais du point de vue de la blockchain, c'est l'expéditeur qui paie.

## Capacité de délégation

Un détenteur de jetons sur une blockchain lancée sur la plateforme EOS.IO qui n'a peut-être pas un besoin immédiat de consommer tout ou partie de la bande passante disponible, peut déléguer ou louer cette bande passante non consommée à d'autres; les producteurs de blocs qui utilisent le logiciel EOS.IO sur cette blockchain reconnaîtront cette délégation de capacité et alloueront la bande passante en conséquence.

## Indépendance des coûts de transaction et de la valeur des jetons

L'un des principaux avantages de la plateforme EOS.IO est que la quantité de bande passante disponible pour une application est entièrement indépendante du prix des jetons. Si un propriétaire d'application détient un nombre pertinent de jetons sur une blockchain adoptant le logiciel EOS.IO, alors l'application peut fonctionner indéfiniment dans le cadre d'un état et d'une bande passante dédiée. Dans ce cas, les développeurs et les utilisateurs ne sont pas affectés par la volatilité des prix sur le marché symbolique et ne dépendent donc pas d'un flux de prix. En d'autres termes, une blockchain qui adopte le logiciel EOS.IO permet aux producteurs de blocs d'augmenter naturellement la bande passante, le calcul et le stockage disponible par jeton indépendamment de la valeur du jeton.

Une blockchain utilisant le logiciel EOS.IO attribue également des jetons aux producteurs de blocs chaque fois qu'ils produisent un bloc. La valeur des jetons aura un impact sur la quantité de bande passante, de stockage et de calcul qu'un producteur peut se permettre d'acheter; ce modèle tire naturellement parti de l'augmentation de la valeur des jetons pour augmenter les performances du réseau.

## Coûts de stockage 

Alors que la bande passante et le calcul peuvent être délégués, le stockage de l'état de l'application exigera qu'un développeur d'application tienne les jetons jusqu'à ce que cet état soit supprimé. Si l'état n'est jamais effacé, les jetons sont effectivement retirés de la circulation.

## Récompenses des blocs

Une blockchain qui adopte le logiciel EOS.IO attribuera de nouveaux jetons à un producteur de blocs chaque fois qu'un bloc est produit. Dans ces circonstances, le nombre de jetons créés est déterminé par la médiane de la rémunération désirée publiée par tous les producteurs de blocs. Le logiciel EOS.IO peut être configuré pour imposer un plafond sur les primes à la production, de sorte que l'augmentation annuelle totale de l'approvisionnement en jetons ne dépasse pas 5 %.

## Système de proposition de travail

En plus d'élire les producteurs de blocs, conformément à une blockchain basée sur le logiciel EOS.IO, les détenteurs de jetons peuvent élire un certain nombre de propositions de réalisation conçues pour le bénéfice de la communauté. Les propositions gagnantes recevront des jetons allant jusqu'à un pourcentage configuré de l'inflation des jetons moins les jetons qui ont été payés aux producteurs de blocs. Ces propositions recevront des jetons proportionnels aux votes que chaque demande a reçus des détenteurs de jetons, jusqu'à concurrence du montant qu'ils demandent pour l'exécution de leur travail. Les propositions élues peuvent être remplacées par des propositions nouvellement élues par les détenteurs de jetons.

Le système de contrats implémentant les propositions de travail pourraient ne pas être en place lors du lancement initial en juin 2018, mais le mécanisme de financement le sera. Il commencera à accumuler des fonds en même temps que le début de la rétribution des producteurs. Étant donné que le système de propositions des travail sera mis en œuvre dans WASM, il peut être ajouté à une date ultérieure sans besoin de faire un "fork".


# Gouvernance

La gouvernance est le processus par lequel les gens d'une communauté:

1. Parvenir à un consensus sur des questions subjectives d'action collective qui ne peuvent pas être entièrement traitées par des algorithmes logiciels;
2. Exécuter les décisions qu'ils prennent; et
3. Modifier les règles de gouvernance elles-mêmes par le biais d'amendements constitutionnels. 

Une blockchain basée sur le logiciel EOS.IO met en œuvre un processus de gouvernance qui dirige efficacement l'influence existante des producteurs de blocs. En l'absence d'un processus de gouvernance défini, les blockchains antérieures reposaient sur des processus de gouvernance ad hoc, informels et souvent controversés aboutissant à des résultats imprévisibles.

Une blockchain basée sur le logiciel EOS.IO reconnaît que le pouvoir provient des détenteurs de jetons qui délèguent ce pouvoir aux producteurs de blocs. Les producteurs de blocs ont un pouvoir limité et vérifié de geler les comptes, de mettre à jour les applications défectueuses et de proposer des changements au protocole sous-jacent.

L'élection des producteurs de blocs est intégrée dans le logiciel EOS.IO. Avant qu'un changement puisse être apporté à la blockchain, les producteurs de blocs doivent l'approuver. Si les producteurs de blocs refusent d'apporter les changements souhaités par les détenteurs de jetons, ils peuvent être éliminés. Si les producteurs de blocs font des changements sans la permission des détenteurs de jetons, tous les autres validateurs de nœuds complets non producteurs (échanges, etc.) rejetteront le changement.

## Gel des comptes

Parfois, un smartcontract se comporte de manière aberrante ou imprévisible et ne fonctionne pas comme prévu; d'autres fois, une application ou un compte peut découvrir un exploit qui lui permet de consommer une quantité déraisonnable de ressources. Lorsque de tels problèmes surviennent inévitablement, les producteurs de blocs ont le pouvoir de rectifier de telles situations.

Les producteurs de blocs de toutes les blockchains ont le pouvoir de choisir quelles transactions sont incluses dans les blocs, ce qui leur donne la possibilité de geler les comptes. Une blockchain utilisant le logiciel EOS.IO formalise cette autorité en soumettant le processus de gel d'un compte à un vote de 15/21 des producteurs actifs. Si les producteurs abusent du pouvoir, ils peuvent être éliminés et un compte sera débloqué.

## Modification du code de compte

Quand tout échoue et qu'une "application imparable" agit d'une manière imprévisible, une blockchain utilisant le logiciel EOS.IO permet aux producteurs de blocs de remplacer le code du compte sans forcer la chaîne de blocs entière. Semblable au processus de gel d'un compte, ce remplacement du code exige un vote de 15/21 des producteurs de blocs élus.

## Constitution

Le logiciel EOS.IO permet aux Blockchains d'établir un accord de service peer-to-peer ou un contrat liant mutuellement les utilisateurs qui le signent, ce que l'on appelle une "constitution". Le contenu de cette constitution définit les obligations entre les utilisateurs qui ne peuvent pas être entièrement appliquées par le code et facilite la résolution des litiges en établissant la juridiction et le choix de la loi ainsi que d'autres règles mutuellement acceptées. Chaque transaction diffusée sur le réseau doit incorporer le hash de la constitution dans le cadre de la signature et lie ainsi explicitement le signataire au contrat.

La constitution définit également l'intention lisible par l'homme du protocole du code source. Cette intention est utilisée pour identifier la différence entre un bogue et une fonctionnalité lorsque des erreurs se produisent et guide la communauté sur les corrections correctes ou incorrectes.

## Mise à jour du protocole et de la constitution

Le logiciel EOS.IO définit le processus suivant par lequel le protocole, tel que défini par le code source canonique et sa constitution, peut être mis à jour:

1.  Les producteurs de blocs proposent un changement à la constitution et obtiennent l'approbation 15/21.
2.  Les producteurs de blocs maintiennent l'approbation 15/21 de la nouvelle **constitution** pendant 30 jours consécutifs.
3.  Tous les utilisateurs sont tenus d'indiquer qu'ils acceptent la nouvelle constitution comme condition du traitement des transactions futures.
4.  Les producteurs de blocs adoptent des changements au code source pour refléter le changement dans la constitution et le proposer à la blockchain en utilisant le hash de la nouvelle constitution.
5.  Les producteurs de blocs maintiennent l'approbation 15/21 du nouveau **code** pendant 30 jours consécutifs.
6.  Les changements au code entrent en vigueur 7 jours plus tard, donnant à tous les nœuds complets non producteurs 1 semaine pour la mise à niveau après la ratification du code source.
7.  Tous les nœuds qui ne passent pas au nouveau code s'éteignent automatiquement.

Par défaut, la configuration du logiciel EOS.IO, le processus de mise à jour de la blockchain pour ajouter de nouvelles fonctionnalités prend 2 à 3 mois, tandis que les mises à jour pour corriger les bogues non critiques qui ne nécessitent pas de modification de la constitution peuvent prendre 1 à 2 mois.

### Changements d'urgence

Les producteurs de blocs peuvent accélérer le processus si un changement de logiciel est nécessaire pour corriger un bogue nuisible ou une faille de sécurité qui nuit activement aux utilisateurs. En général, il pourrait être contre la constitution pour les mises à jour accélérées d'introduire de nouvelles fonctionnalités ou de corriger des bogues inoffensifs.

# Scripts et machines virtuelles

Le logiciel EOS.IO sera d'abord et avant tout une plateforme pour coordonner la livraison de messages authentifiés (appelés Actions) aux comptes. Les détails du langage de script et de la machine virtuelle sont des détails spécifiques à l'implémentation qui sont pour la plupart indépendants de la conception de la technologie EOS.IO. Tout langage ou machine virtuelle déterministe et correctement sandboxé avec des performances suffisantes peut être intégré avec l'API logicielle EOS.IO.

## Actions définies par schéma

Toutes les actions envoyées entre les comptes sont définies par un schéma qui fait partie de l'état de consensus de la blockchain. Ce schéma permet une conversion transparente entre la représentation binaire et JSON des Actions.

## Base de données définie par schéma

L'état de la base de données est également défini à l'aide d'un schéma similaire. Ceci garantit que toutes les données stockées par toutes les applications sont dans un format qui peut être interprété comme du JSON lisible par l'homme mais stocké et manipulé avec l'efficacité du binaire.

## API générique de base de données multi-index

L'élaboration de smartcontracts nécessite un schéma de base de données défini pour suivre, stocker et trouver des données. Les développeurs ont généralement besoin des mêmes données triées ou indexées par plusieurs champs et pour maintenir la cohérence entre tous les indices.

## Séparation de l'authentification et de l'application

Pour maximiser les possibilités de parallélisation et minimiser la dette de calcul associée à la régénération du statut de l'application à partir du journal des transactions, le logiciel EOS.IO sépare la logique de validation en trois sections:

1. Valider la cohérence interne d'une Action;
2. Valider que toutes les conditions préalables sont valides; et
3. Modifier l'état de l'application.

La validation de la cohérence interne d'une Action est en lecture seule et ne nécessite aucun accès à l'état de la blockchain. Cela signifie qu'il peut être réalisé avec un parallélisme maximal. La validation des conditions préalables, telles que l'équilibre requis, est en lecture seule et peut donc également bénéficier du parallélisme. Seule la modification de l'état de l'application nécessite un accès en écriture et doit être traitée séquentiellement pour chaque application.

L'authentification est le processus en lecture seule qui consiste à vérifier qu'une action peut être appliquée. L'application fait le travail. En temps réel, les deux calculs doivent être effectués, mais une fois qu'une transaction est incluse dans la blockchain, il n'est plus nécessaire d'effectuer les opérations d'authentification.

# Communication Inter-Blockchains

Le logiciel EOS.IO est conçu pour faciliter la communication entre les blocs. Pour ce faire, il est facile de générer la preuve de l'existence de l'Action et la preuve de la séquence d'Action. Ces preuves combinées à une architecture d'application conçue autour de "l'Action passing" permet de cacher aux développeurs d'applications les détails de la communication inter-blockchains et de la validation des preuves, ce qui permet de présenter des abstractions de haut niveau aux développeurs.

<img align="right" src="https://github.com/EOSIO/Documentation/blob/images/images/Diagram1.jpg" width="362.84px" height="500px" />

## Preuves de Merkle pour la validation des clients légers (LCV)

L'intégration avec d'autres blockchain est beaucoup plus facile si les clients n'ont pas besoin de traiter toutes les transactions. Après tout, un échange ne se soucie que des transferts à l'intérieur et à l'extérieur de l'échange et rien d'autre. Il serait également idéal que la chaîne d'échange puisse utiliser des preuves de dépôt de Merkle légères plutôt que d'avoir à faire entièrement confiance à ses propres producteurs de blocs. À tout le moins, les producteurs de blocs d'une chaîne aimeraient maintenir les frais généraux les plus faibles possibles lorsqu'ils se synchronisent avec une autre blockchain.

L'objectif du LCV est de permettre la génération de preuves d'existence relativement légères qui peuvent être validées par quiconque suivant un ensemble de données relativement légères. Dans ce cas, l'objectif est de prouver qu'une transaction particulière a été incluse dans un bloc particulier et que le bloc est inclus dans l'historique vérifié d'une blockchain particulière.

Bitcoin prend en charge la validation des transactions en supposant que tous les nœuds ont accès à l'historique complet des en-têtes de bloc qui s'élève à 4 Mo d'en-têtes de bloc par an. A 10 transactions par seconde, une preuve valide nécessite environ 512 octets. Ceci fonctionne bien pour une blockchain avec un intervalle de 10 minutes, mais n'est plus " léger " pour les blockchains avec un intervalle de 0,5 seconde.

Le logiciel EOS.IO permet des preuves légères pour tous ceux qui ont un en-tête de bloc irréversible après le point d'inclusion de la transaction. En utilisant la structure hash-linked montré, il est possible de prouver l'existence de toute transaction avec une preuve de moins de 1024 octets de taille.

A partir d'un identifiant de bloc pour un bloc dans la blockchain, et des en-têtes d'un bloc irréversible de confiance. Il est possible de prouver que le bloc est inclus dans la blockchain. Cette preuve prend les digests de ceil(log2(N)) pour son chemin, où N est le nombre de blocs dans la chaîne. Avec une méthode d'authentification de SHA256, vous pouvez prouver l'existence de n'importe quel bloc d'une chaîne qui contient 100 millions de blocs en 864 octets.

Il y a peu de frais généraux supplémentaires associés à la production de blocs avec le hash-linking approprié pour permettre ces preuves, ce qui signifie qu'il n'y a aucune raison de ne pas générer des blocs de cette façon.

Quand vient le temps de valider les preuves sur d'autres chaînes, il y a une grande variété d'optimisations temps/espace/bande passante qui peuvent être faites. Le suivi de tous les en-têtes de bloc (420 Mo/an) permet de conserver des preuves de petite taille. Seules les en-têtes récentes peuvent offrir un compromis entre le stockage à long terme minimal et la taille de la preuve. Alternativement, une blockchain peut utiliser une approche d'évaluation paresseuse où elle se souvient des hashs intermédiaires des preuves passées. Les nouvelles preuves n'ont qu'à inclure des liens vers les arbres clairsemés connus. L'approche exacte utilisée dépendra nécessairement du pourcentage de blocs étrangers qui incluent des transactions référencées par Merkle proof.

Après une certaine densité d'interconnexion, il devient plus efficace de simplement faire en sorte qu'une chaîne contienne l'historique complet du bloc d'une autre chaîne et élimine le besoin de preuves entièrement Pour des raisons de performance, il est idéal de minimiser la fréquence des preuves inter-chaînes.


## Latence de la communication interchaînes

Lorsqu'ils communiquent avec une autre blockchain externe, les producteurs de blocs doivent attendre qu'il y ait une certitude à 100 % qu'une transaction ait été confirmée de façon irréversible par l'autre blockchain avant de l'accepter comme intrant valide. En utilisant une blockchain basée sur le logiciel EOS.IO et DPoS avec des blocs de 0,5 seconde et l'ajout de l'irréversibilité Byzantine Fault Tolerant, cela prend environ 0,5 seconde. Si les producteurs de blocs d'une chaîne n'attendent pas l'irréversibilité, la situation serait semblable à celle d'un exchange créditant un dépôt qui a été inversé par la suite et  pourrait  donc avoir un impact sur la validité du consensus de la blockchain. Le logiciel EOS.IO utilise à la fois DPoS et aBFT pour assurer une irréversibilité rapide.

## Preuve de finalité

Lorsque l'on utilise des Preuves Merkle provenant de blockchains externes, il y a une différence significative entre savoir que toutes les transactions traitées sont valides et savoir qu'aucune transaction n'a été ignorée ou omise. Bien qu'il soit impossible de prouver que toutes les transactions les plus récentes sont connues, il est possible de prouver qu'il n'y a pas eu de trous dans l'historique des transactions. Le logiciel EOS.IO facilite cela en attribuant un numéro d'ordre à chaque Action livrée à chaque compte. Un utilisateur peut utiliser ces numéros de séquence pour prouver que toutes les actions destinées à un compte particulier ont été traitées et qu'elles ont été traitées dans l'ordre.

## Témoins séparés (Segwit)

Le concept de témoin distinct (SegWit) est que les signatures de transaction ne sont pas pertinentes après qu'une transaction est irréversiblement incluse dans la blockchain. Une fois qu'il est inaltérable, les données de signature peuvent être réduites et tout le monde peut encore dériver l'état actuel. Comme les signatures représentent un pourcentage important de la plupart des transactions, SegWit représente une économie significative en termes d'utilisation du disque et de temps de synchronisation.

Ce même concept peut s'appliquer aux Preuves Merkle utilisées pour la communication inter-blockchain. Une fois qu'une preuve est acceptée et irréversiblement enregistrée dans la blockchain, les 2KB de hashage SHA256 utilisés par la preuve ne sont plus nécessaires pour dériver le bon état de la blockchain. Dans le cas de la communication inter-blockchain, ces économies sont 32 fois plus importantes que les économies réalisées sur les signatures normales.

Un autre exemple de SegWit serait pour les posts de blog Steem. Selon ce modèle, un message ne contiendrait que le contenu du SHA256 (contenu du blog) et le contenu du blog se trouverait dans les données séparées sur les témoins. Le producteur de bloc vérifierait que le contenu existe et a le hashage donné, mais le contenu du blog n'aurait pas besoin d'être stocké afin de récupérer l'état actuel du journal de la blockchain. Cela permet de prouver que le contenu a déjà été connu sans avoir à stocker ce contenu pour toujours.

# Conclusion

Le logiciel EOS.IO est conçu à partir de l'expérience avec des concepts éprouvés et des meilleures pratiques, et représente un progrès fondamental dans la technologie blockchain. Le logiciel fait partie d'un plan holistique pour une société globale et évolutive dans laquelle les applications décentralisées peuvent être facilement déployées et gouvernées.
