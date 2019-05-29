# Livre Blanc Livepeer

**Protocole et Motivations Economiques Pour un Réseau Decentralisé De Streaming Vidéo**

Doug Petkanics <doug@livepeer.org>    
Eric Tang <eric@livepeer.org>   

## Abstrait ###########################################

Le projet Livepeer vise à fournir un protocole réseau de streaming vidéo en direct entièrement décentralisé, hautement évolutif, utilisant des jetons cryptographiques, et donnant lieu à une solution pouvant servir de couche média live dans la pile de développement décentralisé (web3). En outre, Livepeer est destiné à offrir une alternative économique aux solutions de diffusion centralisées pour tous les diffuseurs existants. Dans ce document, nous décrivons le protocole Livepeer, un protocole basé sur la participation pour inciter les participants à un réseau de diffusion vidéo en direct d’une manière sécurisée du point de vue du jeu. Nous présentons des solutions pour la vérification évolutive du travail décentralisé, ainsi que pour la prévention du travail inutile, dans le but de jouer les attributions de jetons dans un système inflationniste.


## Table des Matières ###########################################

* [Introduction et contexte]
    * [La pile de vidéos en direct]
* [Protocole Livepeer]
    * [Segments vidéo](#segments-vidéo)
    * [Le Livepeer Token]
    * [Rôles de protocole]
    * [Consensus]
    * [Collage + Délégation]
    * [Transcoder() Transaction]
    * [Diffusion + transcodage]
        * [Prétraitement]
        * [Le travail]
        * [Fin du travail]
    * [Vérification du travail]
        * [Une note sur Truebit]
    * [Génération de tokens]
    * [Slashing]
    * [Distribution de tokens](#distribution-de-tokens)
    * [La gouvernance]
* [Les attaques]
    * [Attaques de consensus]
    * [DDoS]
    * [Transcodeur inutile ou autorégulant]
    * [Griefing du transcodeur]
    * [Chain Reorg]
* [Distribution vidéo en direct]
* [Cas d'utilisation]
    * [Consommation de contenu à la carte]
    * [Services de vidéo sociale à mise à l'échelle automatique]
    * [Journalisme en direct sans censure]
    * [DApps vidéo activés]
* [Résumé]
* [Appendice]
    * [Référence de paramètre de protocole Livepeer]
    * [Types de transaction de protocole Livepeer]
* [Références](#Références)

*Remarque: Ce document a été publié pour la première fois en avril 2017. Une proposition d’augmentation de capacité appelée «Streamflow» a été proposée en décembre 2018. Elle propose certaines itérations et améliorations concernant certaines des idées présentées ci-dessous. Lisez la proposition pour [Streamflow ici](https://github.com/livepeer/wiki/blob/master/STREAMFLOW.md).*


## Introduction et Contexte ###########################################

La vision du Web décentralisé a commencé à se concrétiser au cours des dernières années avec l’émergence de réseaux tels que [Ethereum](http://ethereum.org) pour permettre l’informatique “trustless”, [Swarm](http://swarm-gateways.net/bzz:/theswarm.eth/) et [IPFS/Filecoin](http://ipfs.io) pour permettre le stockage décentralisé et la distribution de contenu, Bitcoin et divers projets de tokens pour faciliter le transfert p2p de valeur, et des registres d’identification numerique décentralisés comme [Blockstack](http://blockstack.org) et [ENS](http://ens.readthedocs.io/en/latest/introduction.html) pour fournir des noms  accessibles au contenu et aux identités. Ces éléments constituent la base des applications décentralisées (DApps) qui doivent être construites sous forme de contenu Web ou mobile en grande partie statique ou rarement mis à jour, mais pour le moment, les DApps n'ont toujours pas la possibilité d'inclure le streaming de données et de médias de manière ouverte et décentralisée. Le projet Livepeer a pour objectif de décentraliser la diffusion de vidéos en direct sur Internet.


[La présentation du projet Livepeer](https://github.com/livepeer/wiki/wiki/Project-Overview) constitue une introduction intéressante à l’état actuel de la vidéo en direct sur Internet. Ce livre blanc se concentrera principalement sur les détails du protocole crypto-économique de Livepeer, plutôt que sur l'analyse de rentabilisation, mais en résumé, la vue d'ensemble décrit l'état actuel de la diffusion en direct comme se développant à un rythme rapide, centralisé et coûteux. D'autre part, une solution P2P entièrement décentralisée, dans laquelle les nodes apportaient leur propre calcul et où la bande passante en service de diffusion en direct de vidéo en direct serait plus ouverte et évolutive, le nombre de connexions pouvant être desservies n'étant pas limitée.

Cette technologie est certes disponible dans une certaine mesure, mais jusqu’à présent, rien n’indiquait que les utilisateurs gèrent des nodes offrant cette fonctionnalité. De plus, aucun financement adéquat n’a été prévu pour le développement d’un protocole ouvert susceptible de faciliter cela, et qui profite à l'ensemble de l'internet plutôt qu'à une seule entreprise centralisée. Cependant, avec l'émergence récente de protocoles utilisant des clés cryptographiques[[2, 3](#Références)], il est maintenant possible d'inciter simultanément les utilisateurs à contribuer au calcul et à la bande passante vers la diffusion vidéo en direct, de manière à financer le développement d'un serveur multimédia ouvert, et d’une solution capable de diffuser des vidéos en streaming en direct conformément aux normes et formats les plus récents nécessaires pour atteindre la gamme complète de périphériques. De plus, les mesures économiques traditionnellement considérées comme le résultat de protocoles utilisant des tokens indiquent que le coût supporté par le radiodiffuseur pour utiliser le réseau Livepeer pourrait être inférieur à celui de toute solution centralisée.

À mesure que la technologie et le protocole Livepeer seront livrés, les utilisateurs pourront participer au flux suivant:

1. Capturez une vidéo sur votre caméra, votre téléphone, votre écran ou votre webcam et envoyez-la sur le réseau Livepeer.
2. Les nodes fonctionnant sur le réseau l'encoderont dans tous les formats nécessaires pour atteindre chaque périphérique pris en charge. Les utilisateurs de ces nodes seront encouragés par les redevances versées par le radiodiffuseur en ETH et par la possibilité de renforcer la réputation via le jeton de protocole pour obtenir le droit d'effectuer davantage de travaux à l'avenir.
3. Tout utilisateur du réseau peut demander à voir le flux, qui lui sera automatiquement distribué en temps quasi réel.

<img src="https://s3.amazonaws.com/livepeerorg/LPExample.png" alt="Livepeer Network Example" style="width: 750px">

### Pile de Vidéos en Direct

La pile de technologies pour la diffusion de vidéos en direct a évolué au cours de nombreuses années et contient de nombreuses couches. Les radiodiffuseurs doivent capturer la vidéo à la source, établir une interface avec un serveur multimédia pour traiter et transcoder la vidéo dans de nombreux formats, répartir la vidéo sur un réseau, puis permettre au consommateur final de lire la vidéo avec une qualité élevée. Il y a aussi des questions économiques qui sont introduites lorsque l'on réfléchit à cette pile, par exemple si le diffuseur ou le consommateur devrait payer pour la bande passante nécessaire au transfert de la vidéo.

De nos jours, une plate-forme de diffusion en direct typique doit prendre en charge les formats vidéo RTMP, HLS et Mpeg-Dash dans les codecs H.264 et VP8. De nouveaux codecs tels que H.265 / HEVC, VP9 et AV1 deviendront de plus en plus populaires dans un proche avenir, à mesure que les consommateurs s'habitueront à une qualité vidéo supérieure. Pour le seul système HLS, [Apple suggère](https://developer.apple.com/library/content/documentation/General/Reference/HLSAuthoringSpec/Requirements.html#//apple_ref/doc/uid/TP40016596-CH2-SW1) des débits allant de 145 kb/s à 7800 kb/s, afin de desservir les différents types d'appareils dans différentes conditions. Tout cela ajoute une complexité et des coûts importants à la diffusion vidéo en direct.

La pile de développement décentralisée existante (web3) contient des solutions pour certaines des couches requises pour une plate-forme vidéo en direct, telles que le transfert de fichiers et les paiements, mais il n’existe actuellement aucune solution pour la capture et l’interface, le transcodage et le traitement, ainsi que pour le traitement des couches de vidéo en direct. Pour ce faire, Livepeer introduit le serveur [LPMS (Livepeer Media Server)](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server), une implémentation open source d’un serveur multimédia qui fournit toutes les fonctionnalités spécifiques à la vidéo en direct nécessaires aux développeurs de DApp et aux diffuseurs existants pour les intégrer à leurs applications. Pour en savoir plus à ce sujet [cliquez ici](https://github.com/livepeer/wiki/wiki/Livepeer-Media-Server).

En tant qu'application autonome, tout développeur peut créer une application en temps réel au-dessus du système de gestion de la chaîne logistique, mais cette dernière serait toujours centralisée et devrait être mise à l'échelle de manière traditionnelle. Cependant, lorsque chaque nde du réseau Livepeer exécute le système LPMS, et que les incitations économiques offertes par le protocole garantissent que ces nodes apporteront leur puissance de traitement et leur bande passante au service du transcodage et de la distribution de vidéo en direct, **une solution évolutive à la consommation Le réseau est mis à la disposition des développeurs, qui peuvent simplement envoyer leur flux en direct sur le réseau et obtenir les détails de mise en œuvre de la mise à l'échelle, du paiement et de l'hébergement des médias**.

## Protocole Livepeer ###########################################

Le protocole Livepeer définit la manière dont les différents acteurs d'un écosystème de diffusion en direct participent de manière sécurisée et économiquement rationnelle. Les deux principaux domaines sur lesquels le protocole doit porter sont la distribution réelle et évolutive de la vidéo en direct de la source à un grand nombre de consommateurs, ainsi que les incitations économiques à encourager la participation au réseau de manière sécurisée et basée sur la théorie du jeu. Bien que ce livre blanc traite de la distribution vidéo en direct proprement dite, où il se superpose au protocole économique, il se concentrera principalement sur ce dernier afin de démontrer la sécurité et l’alignement économique. Au plus haut niveau, le protocole est conçu pour:

- Autorisez n'importe quel nde à envoyer une vidéo en direct sur le réseau et payez éventuellement pour qu'il soit transcodé dans différents formats et débits binaires.
- Autorisez n'importe quel node à demander la vidéo du réseau.
- Permettre aux participants d’apporter leur puissance de traitement et leur bande passante en service de transcodage et de distribution de vidéo, et d’être rémunérés en conséquence.

Dans un réseau décentralisé où les participants sont récompensés proportionnellement à la quantité de travail qu’ils ont apportée, les deux grands défis à relever pour assurer la sécurité sont:

- Peut-on vérifier que le travail effectué par les nodes a été effectué correctement?
- Les nodes sont-ils récompensés pour un travail réel qui a apporté de la valeur au réseau, par opposition à un travail factice effectué dans le but d'obtenir injustement des attributions de tokens?

Le protocole Livepeer est conçu pour traiter à la fois la vérification du travail et la prévention du faux travail, tout en offrant des solutions pour l’évolutivité automatique du réseau et une gouvernance pour l’évolution du protocole dans le temps.

### Segments Vidéo

L'unité de base des médias au sein de Livepeer est ce que nous appellerons un `segment`. Un segment est un bloc découpé dans le temps d'audio multiplexé et de vidéo de durée `t`. Chaque segment du réseau Livepeer est unique et contient les preuves cryptographiques permettant de vérifier que le diffuseur a prévu ces données spécifiques pour ce segment spécifique. Chaque flux est composé de nombreux segments consécutifs, chacun contenant un numéro de séquence identifiant leur ordre approprié. Un segment contient les champs suivants:

| Champ de segment vidéo | La description |
|--------|--------|
| **StreamID** | Identifie le node d'origine et le flux auxquels ce segment appartient. |
| **SequenceNumber** | L'ordre séquentiel auquel ce segment appartient dans le flux d'origine. |
| **DataPayload** | Les métadonnées binaires et les données représentant l'audio / vidéo dans ce segment. |
| **DataHash** | Le hachage de la charge de données. |
| **BroadcasterSignature** | Une signature du diffuseur de `Priv(StreamID, SequenceNumber, hash(StreamID, SequenceNumber, DataHash))` qui peut être utilisé pour attester et vérifier que le radiodiffuseur prétend qu'il s'agit des vraies données pour ce segment unique. |

Le protocole Livepeer utilise généralement des segments comme unité de travail pour le transcodage, la distribution et les paiements.

### Le Livepeer Token

Le Livepeer Token (LPT) est le token de protocole du réseau Livepeer. Mais ce n'est pas le moyen d'échange. Les radiodiffuseurs utilisent l'éther Ethereum (ETH) pour diffuser des vidéos sur le réseau. Les nodes qui contribuent au traitement et à la bande passante gagnent en ETH sous la forme de redevances des diffuseurs. LPT est un signe que les participants qui souhaitent travailler sur l’enjeu du réseau afin de coordonner la répartition du travail sur le réseau et de garantir que le travail sera effectué honnêtement et correctement. LPT a les buts suivants:

- Il sert de mécanisme de liaison dans un système de preuve de participation déléguée, dans lequel la participation est déléguée à des transcodeurs (ou validateurs) qui participent au protocole pour transcoder une vidéo et valider le travail. Le token et la réduction potentielle résultant d'une violation de protocole sont nécessaires pour sécuriser le réseau contre un certain nombre d'attaques. Plus ci-dessous.
- Il achemine le travail à travers le réseau proportionnellement à la quantité de tokens implicites et délégués, servant essentiellement de mécanisme de coordination.
- Il s’agit d’une unité de compte spécifique à l’écosystème Livepeer, qui constitue la base du concept SectorCoin, applicable aux fonctionnalités supplémentaires devant être introduites à l’avenir [[4](#Références)]. Des services tels que les enregistreurs numériques, les sous-titres codés, l’insertion / la monétisation d’annonces et les outils d’analyse peuvent tous s’intégrer à l’écosystème Livepeer et potentiellement utiliser la sécurité fournie par l’implémentation LPT.


Une allocation initiale de token Livepeer sera distribuée de sorte que les parties prenantes puissent remplir différents rôles dans le réseau et l’utiliser, puis un token supplémentaire sera émis en fonction de l’émission programmée par algorithme dans le temps. Voir la section [distribution de tokens](#distribution-de-tokens).

Suivant les conventions d'Ethereum et de nombreux tokens populaires ERC20 [[16](#Références)], LPT sera divisible par 10 ^ 18, avec des dénominations plus grandes, telles que le LPT lui-même destiné à être utilisé pour des transactions au niveau utilisateur telles que le jalonnement, et des dénominations plus petites destinées à être utilisées pour la comptabilité de protocole.

### Rôles de protocole

Avant de poursuivre, définissons les rôles dans le réseau afin qu’il existe un vocabulaire commun pour la discussion du protocole. Un nœud Livepeer est un ordinateur exécutant le logiciel Livepeer.

| Rôle du node | La description | 
|--------|-----------|
| **Diffuseur** | Node Livepeer publiant le flux d'origine. |
| **Transcodeur** | Node Livepeer effectuant le travail de transcodage du flux dans un autre format de codec, de débit binaire ou de conditionnement. |
| **Node de relais** | Node Livepeer participant à la distribution de vidéo en direct et à la transmission de messages de protocole, mais n’effectuant pas nécessairement de transcodage. |
| **Consommateur** | Node Livepeer demandant le flux, susceptible de le visualiser ou de le servir via une passerelle vers leur application ou les utilisateurs de DApp. |

En plus des rôles ci-dessus joués par les utilisateurs exécutant des nodes Livepeer, le protocole fera également référence aux systèmes suivants. Bien que nous utilisions certains systèmes spécifiques pour faire référence à une implémentation possible, d'autres systèmes peuvent également être basculés s'ils offrent des fonctionnalités et des garanties crypto-économiques similaires:

| Rôle du système | La description | 
|--------|-----------|
| **Swarm** | Contenu adressé plate-forme de stockage. Il est possible de garantir la disponibilité temporaire des données pendant le processus de vérification via le protocole SWEAR [[7, 12](#Références)]. (Remarque: dans ce document, nous faisons référence à Swarm, mais d'autres plates-formes de stockage à contenu adressé peuvent être remplacées si la disponibilité des données peut être garantie avec une probabilité élevée)
| **Livepeer Smart Contract** | Contrat intelligent fonctionnant sur le réseau Ethereum [[1](#Références)]. |
| **Truebit** | Protocole de vérification Blackbox qui garantit l'exactitude du calcul placé sur la chaîne (à un coût élevé) [[6](#Références)]. (<http://truebit.io>) |

Voici un aperçu visuel des rôles et de la manière dont ils communiquent dans le processus de vérification du travail décrit ci-dessous.

<img src="https://livepeer-dev.s3.amazonaws.com/docs/lpprotocol.png" alt="Protocol Visual Overview" style="width: 750px"> 

*Segments allant du diffuseur au transcodeur et finalement au consommateur. Le transcodeur s'assure qu'il dispose de signatures et d'une preuve de travail pour participer à la procédure de vérification du travail.*

**Remarque sur les transcodeurs**: Les transcodeurs jouent le rôle le plus important dans l'écosystème Livepeer. Ce sont eux qui prennent un flux d’entrée et le convertissent dans de nombreux formats différents de manière opportune pour une distribution à faible temps de latence. En tant que tels, ils bénéficient d'une haute disponibilité, d'un matériel efficace et puissant (potentiellement avec le transcodage accéléré par GPU), de connexions à bande passante élevée et de pratiques éprouvées de DevOps. Les transcodeurs doivent beaucoup moins fonctionner que les autres participants du réseau, car lorsqu’ils s’occupent de transcoder un flux, il n’est pas idéal de laisser tomber le réseau. Bien que le réseau puisse prendre en charge de nombreux participants jouant le rôle de transcodeur (et obtenant les attributions de jetons requises), il s'agit d'un rôle spécial délégué de la plupart des participants au réseau, afin de garantir le maintien d'un réseau fiable offrant de la valeur aux diffuseurs. Plus ci-dessous sur cette délégation.

### Consensus

Livepeer a un système de consensus à deux couches. Le grand livre et les transactions LPT sont sécurisés par la blockchain sous-jacente, telle que Ethereum. Tout transfert du token LPT ou toute transaction dans le système peut être considéré comme ayant été confirmé avec la même sécurité que la preuve de travail sous-jacente ou la preuve de blocage de la mise. La deuxième couche, cependant, dicte la distribution du LPT nouvellement généré. Ceci est régi par le smart contrat Livepeer et la participation au protocole de divers acteurs. Bien qu'aucun consensus ne soit requis à proprement parler, en termes d'acceptation et de validation des blocs précédents, le protocole définit des règles de participation et les conditions dans lesquelles les acteurs seront sanctionnés pour ne pas s'être acquittés de leur rôle.

Ce deuxième niveau de consensus régissant le token nouvellement généré est basé sur la délégation de preuve (DPOS), inspirée par des systèmes tels que Bitshares, Steem, Tendermint et Casper [[5, 9, 10, 11](#Références)]. Les transcodeurs jouent le rôle de validateurs dans le réseau. Tout utilisateur peut déléguer sa participation à un transcodeur, qui doit ensuite effectuer des tâches de transcodage sur le réseau, participer au protocole de vérification du travail et appeler des fonctions sur une chaîne à des intervalles spécifiques pour valider ce travail. Le protocole distribuera des frais et des jetons nouvellement générés, et réduira les enjeux d'acteurs mal comportés. Le résultat de la validation sera enregistré sur la chaîne via Truebit une fois la validation effectuée. Il n'y aura donc plus de place pour des litiges entre le diffuseur et le transcodeur.

### Collage + Délégation

Dans Livepeer, pour indiquer une participation dans le réseau, les nodes doivent lier une partie de leur LPT. Ils le font par le biais de la transaction `Bond()`, qui lie leur participation au contrat intelligent jusqu’à ce qu’ils `Unbond()` se lient (), auquel cas ils entrent dans un état non lié qui durera une `UnbondingPeriod`. À la fin de la `UnbondingPeriod`, ils peuvent alors retirer leur LPT.

Le montant cautionné est utilisé pour déléguer la participation à un transcodeur. Le réseau prend en charge simultanément `N` transcodeurs actifs, ce qui est un paramètre de réseau mobile. Tout node peut indiquer qu’il souhaite devenir un transcodeur avec une transaction `Transcoder()` et le protocole sélectionnera les `N` transcodeurs dont l’enjeu est le plus cumulatif (leur propre + délégués d’autres nodes) au début de chaque tour, avec un transcodeur aléatoire de la liste d'attente.

Le token nouvellement généré dans Livepeer est distribué aux nodes liés en proportion relative par rapport à la quantité de travail qu'ils ont liée (moins les frais), tant qu'ils ont été délégués vers des nodes de transcodage se comportant conformément au protocole. Les obligations peuvent être réduites (réduites d’un certain pourcentage) si les nodes auxquels ils ont délégué ne se comportent pas et ne violent pas l’une des conditions de réduction. Les nodes qui se sont liés et ont délégué à un transcodeur reçoivent également une partie des frais générés par le transcodeur par le biais de travaux de transcodage sur le réseau. En substance, les nodes qui effectuent un travail gagnent les droits que les radiodiffuseurs ont payés pour ce travail.

Lorsque ce document utilise le terme "délégant", il fait référence aux nodes liés qui ont délégué leur participation à un candidat transcodeur, au lieu de le déléguer à eux-mêmes en tant que transcodeur.

En résumé, les participants choisissent de lier leur mise pour les raisons suivantes:

- Participez à la délégation vers des transcodeurs efficaces qui fourniront un excellent service au réseau, assurant sa valeur aux radiodiffuseurs.
- Construire une réputation et une allocation de travail futur sous forme de jeton alloué proportionnellement à la mise.
- Gagnez des frais générés par les transcodeurs.
- Ils souhaiteront peut-être devenir un transcodeur.

### Transcoder() Transaction

Un node indique sa volonté de devenir un transcodeur en soumettant une transaction `Transcoder()`, qui publie les trois propriétés suivantes:

- `PricePerSegment`: le prix le plus bas qu'ils sont disposés à accepter pour transcoder un segment de vidéo.
- `BlockRewardCut`: Le % de la récompense de bloc que les nodes liés les paieront pour le service de transcodage. (Example 2%. Si un node lié devait recevoir 100 LPT en récompense de bloc, 2 LPT au transcodeur).
- `FeeShare`: pourcentage des frais de diffusion des travaux que le transcodeur est disposé à partager avec les nodes liés qui délèguent leurs tâches. (Exemple 25%. Si un transcodeur recevait 100 ETH en frais, il devrait payer 25 ETH aux nodes liés).

Le transcodeur peut mettre à jour leur disponibilité et ses informations jusqu’à `RoundLockAmount`, avant le prochain cycle de transcodage. Ceci est offert en % du tour. (Exemple 10% == 2,4 heures. Ils peuvent modifier cette information jusqu'à 2,4 heures avant le prochain tour qui dure `RoundLength` 1 jour). Cela donne aux nodes cautionnés la possibilité de réviser le partage des frais et le partage des récompenses par tokens par rapport aux autres transcodeurs, ainsi que les frais anticipés basés sur le tarif qu'ils facturent et la demande du réseau, et de transférer leur participation déléguée s'ils le souhaitent. Au début d’un tour de transcodage (déclenché par un appel à la transaction `InitializeRound()`), les transcodeurs actifs pour ce tour sont déterminés en fonction de la mise totale déléguée à chaque transcodeur, et les enjeux et les taux sont bloqués pendant la durée du tour.

Un changement est autorisé pendant `RoundLockPeriod`: le prix/segment proposé le plus bas pour l'un des transcodeurs candidats est verrouillé et ne peut pas être déplacé, mais d'autres candidats au transcodeur peuvent ajuster leur prix / segment à la baisse. Cela leur permet de faire correspondre le prix proposé le plus bas sur le réseau s'ils le souhaitent afin de garantir leur part du travail pondérée en fonction de la mise sur le réseau. Ils ne sont pas autorisés à augmenter le prix offert pendant cette période.

Voici un exemple d'état des options de transcodeur qu'un délégant peut examiner lorsqu'il décide à qui déléguer.

Transcoder ID | PricePerSegment | BlockRewardCut | FeeShare |
|----|----|----|----|
| 1 | 22 wei | 1% | 25% |
| 2 | 30 wei | 2% | 40% |
| 3 | 10 wei | 4% | 1% |
| ... | ... | ... | ... |
| N | 14 wei | 0% | 2% |

*Note sur le prix: Dans ce document, nous listons le prix/segment. En réalité, Livepeer envisage d’utiliser un modèle inspiré de la comptabilité gaz où il existe une notion d’unités de gaz requises pour certains paramètres de tâche d’un segment, tels que le débit, le codage, la taille de trame, etc. Le prix / segment est un les incitations sont les mêmes, mais en réalité, ils communiqueront probablement le prix / l’essence.*

### Diffusion + transcodage

Les transcodeurs qui sont ouverts au commerce sur le réseau jettent leur chapeau pour le transcodage en soumettant une transaction `TranscodeAvailability()`. Cela indique leur disponibilité et les place dans un pool de transcodeurs disponibles pour accepter un travail récemment soumis.

Lorsqu'un radiodiffuseur soumet son flux sur le réseau Livepeer, il reçoit un `StreamID`. Cela sert à la fois d'identifiant unique et contient également l'adresse du node d'origine, de sorte que les nodes sachent comment demander et acheminer des demandes pour utiliser ce flux vers l'origine. Le flux contient de nombreux segments consécutifs, comme décrit dans la section [Segments vidéo](#segments-vidéo). Si le radiodiffuseur souhaite que le réseau se charge de transcoder son flux dans tous les formats et débits nécessaires pour atteindre chaque utilisateur sur chaque appareil, la première étape consiste à soumettre une transaction de travail de transcodage sur une chaîne. Les travaux reçoivent également un identifiant unique et les données d'entrée dans le travail consistent en:

`Job(StreamID, TranscodingOptions, PricePerSegment)`

Les `TranscodingOptions` définissent les débits en sortie, les formats, les codages, etc., et le `PricePerSegment` indique le prix que le diffuseur proposera.

Dès que cette transaction est exploitée, le hachage de bloc suivant sera utilisé pour déterminer de manière pseudo-aléatoire le transcodeur sélectionné pour ce travail. Tous les transcodeurs dont le prix est inférieur ou égal au prix offert seront pris en compte et le module de hachage de bloc, le nombre de transcodeurs candidats (pondérés par leurs mises) détermineront l’indice du transcodeur sélectionné.

À ce stade, le radiodiffuseur peut commencer à diffuser des segments vidéo vers le transcodeur et il participera au protocole suivant. Le protocole utilise également une solution de stockage persistant, par exemple Swarm, dans le cadre du processus de vérification du travail.

#### Prétraitement

1.  **Diffuseur** -> **Livepeer Smart Contract**: soumet un dépôt sur la chaîne pour couvrir le coût du travail de transcodage complet. Celui-ci peut être rechargé ultérieurement à tout moment, mais le transcodeur peut cesser de fonctionner si le dépôt est épuisé au fur et à mesure de leur encaissement progressif.

#### Le travail

2. **Diffuseur** -> **Livepeer Smart Contract**: Job (ID de flux, options, prix / segment)
    - Crée la demande de travail sur la chaîne et place un dépôt ETH en dépôt bloqué pour payer le travail.
3. Le protocole peut utiliser le hachage de bloc suivant pour sélectionner de manière déterministe le transcodeur approprié pour ce travail.
4. **Transcodeur** -> **Diffuseur**: envoie le streamID de sortie et confirme que le travail est accepté.
5. **Diffuseur** -> **Transcoder**: envoyez des segments de flux contenant des signatures vérifiant les données d'entrée.
7. **Le transcodeur** effectue le transcodage et rend le nouveau flux de sortie disponible sur le réseau
9. **Transcodeur**: Stockez un reçu de transcodage pour chaque segment de travail de transcodage. Un reçu de transcodage comporte les champs suivants.

| Transcoder le champ de réception | La Description |
|-------|------------|
| **StreamID** | Identifie le node d'origine et le flux auxquels ce segment appartient. |
| **Sequence Number** | L'ordre séquentiel auquel ce segment appartient dans le flux d'origine. |
| **Input Data hash** | Le hachage de la charge de données du segment d'entrée. |
| **Transcoded Data hash** | Le hachage des données de sortie après le transcodage de ce segment. |
| **Broadcaster segment signature** | Une signature du diffuseur de Priv (StreamID, Seq #, Dhash) pouvant être utilisée pour attester et vérifier que le diffuseur prétend qu'il s'agit des données vraies pour ce segment unique. |
| **Transcoder segment signature** | Une signature de tous les champs ci-dessus du transcodeur attestant de l'affirmation selon laquelle ce transcodage de sortie spécifique a été effectué sur cette entrée spécifique. |

Chaque fois que le transcodeur constate qu'il ne reçoit plus de segments, il peut appeler `ClaimWork()` pour réclamer son travail.

#### Fin du Travail

10. **Transcodeur** -> **Livepeer Smart Contract**: Appelez `ClaimWork (JobID, StartSegmentSeq #, EndSegmentSeq #, MerkleRoot)`. Transcoder affirme sur la chaîne qu'il a effectué un travail sur la plage de segment revendiquée, avec une racine centrale de toutes les données de réception de transcodage afin de valider le contenu de ces segments codés.
11. Attendez que cette transaction soit minée et observez le prochain blockhash. Le protocole peut ensuite déterminer quels segments seront vérifiés sur la base du `VerificationRate`.
12. **Transcodeur** -> **Swarm**: écrivez les charges utiles de données d'entrée pour les segments qui seront contestés via la vérification, en utilisant les paramètres SWEAR pour vous assurer que les données seront là assez longtemps pour la vérification (heure de `VerificationPeriod`).
13. **Transcodeur** -> **Contrat Livepeer Smart**: fournissez des revendications de transcodage sur la chaîne pour chaque segment à vérifier, ainsi que des épreuves de type Merkle pour les reçus de chaque segment des revendications de transcodage. Le contrat intelligent peut vérifier les signatures de Broadcaster et de **Transcodeur** pour s'assurer que toutes les données nécessaires sont disponibles pour effectuer la vérification, et peut également vérifier les preuves de fond de référence par rapport à la racine de fond de mot de passe validée de `ClaimWork()`.
14. **Transcodeur** -> **Truebit**: Verify(). Il s’agit d’un appel en chaîne du contrat intelligent Truebit, dans lequel Transcoder fournit le hachage d’entrée Swarm pour le segment en question. (Plus d'informations sur la vérification dans la section suivante)
15. **Truebit** -> **Livepeer Smart Contract**: le résultat du travail est écrit en chaîne. Ceci est comparé au résultat de la revendication de transcodage fourni par le transcodeur.
16.  **Contrat Livepeer Smart**: à ce stade, le contrat Livepeer Smart dispose de toutes les informations nécessaires pour déterminer si le travail du transcodeur est vérifié. 
    - Si la vérification est correcte, utilisez-le comme entrée dans l'algorithme d'allocation de tokens et libérez les frais bloqués. 
    - S'il est incorrect, Transcoder et ses opérateurs se verrouillent avec `FailedVerificationSlashAmount` et le diffuseur est remboursé.

Le radiodiffuseur peut arrêter d’envoyer des segments à n’importe quel moment, ce qui est effectivement un `EndJob()`.

À ce stade, le transcodage a été effectué, la preuve du travail a été revendiquée sur la chaîne et l'échec ou le succès de la vérification du travail a été signalé. Toutes les informations sont en chaîne pour déterminer l'allocation des frais et des tokens aux transcodeurs et aux mandataires, ou la réduction en cas d'échec de la vérification. Voyons comment le travail est réellement vérifié.

### Vérification du Travail

Pour pouvoir allouer des frais aux transcodeurs qui affirment avoir effectué un travail de transcodage, il est nécessaire que le protocole puisse déterminer que le travail a été exécuté correctement avec une probabilité élevée. Pour cela, Livepeer étend ses recherches et utilise le protocole [Truebit](http://truebit.io) [[6](#Références)].

Truebit fonctionne en laissant un participant (le solveur) effectuer le travail réel pour le prix, dans ce cas un transcodage, puis en demandant à des participants supplémentaires (vérificateurs) de vérifier le travail afin de détecter les erreurs, les erreurs ou la tricherie. La tâche est décomposée en très petites étapes et les vérificateurs vérifient le travail du solveur pour trouver la première étape qui diffère de ce à quoi ils s'attendaient. Ensuite, seul un très petit pas doit être joué en chaîne par un contrat intelligent (juge), qui peut dire quelle partie a fait le travail correctement. Les incitations économiques, y compris les erreurs forcées pour inciter les vérificateurs à effectuer des contrôles, garantissent qu’il n’est pas rentable de tricher ou de contester de manière incorrecte, mais qu’il est rentable de jouer le rôle de contrôle du travail.

L'inconvénient de ce protocole est qu'il en coûte 5 à 50 fois plus cher que le travail original pour pouvoir vérifier tout le travail. Livepeer utilise Truebit comme boîte noire pour vérifier les segments, mais il évite de payer cette taxe de vérification très élevée en ne vérifiant qu'un petit pourcentage de segments de manière aléatoire et en utilisant des barres obliques en cas d'échec des vérifications. Le paramètre `VerificationRate` défini dans Livepeer détermine la fréquence à laquelle un segment spécifique doit être contesté dans Truebit, ainsi que le caractère aléatoire d’un hachage de bloc ultérieur après que le travail a été engagé dans la chaîne de blocs, détermine les segments spécifiquement sélectionnés.

Si le travail est engagé via un appel `ClaimWork()` dans le bloc `N`, alors

Si `Sha3 (N, BlockHash (N), Seg #)% VerificationRate == 0`, le segment # doit être vérifié.

Le transcodeur fournit des revendications de transcodage sur la chaîne pour les segments candidats en appelant la transaction Verify(). Livepeer Smart Contract peut vérifier l'authenticité de ces revendications à l'aide des signatures internes et des preuves de validation fournies, puis appeler un appel à Truebit pour vérifier uniquement ces segments.

Les solveurs et les vérificateurs Truebit accèdent aux données d'entrée d'un segment à partir d'un système de stockage à contenu persistant, tel que Swarm. Le transcodeur est chargé de vérifier que les données du segment sont disponibles dans Swarm et peut éventuellement rechercher des reçus du protocole SWEAR [[5](#Références)] garantissant la persistance pendant un certain temps, suffisamment long pour que Truebit puisse le lire. De plus, ils peuvent prendre eux-mêmes en charge l’exécution d’un nœud Swarm en s'assurant que les données sont disponibles pour la vérification Truebit. S'ils ont des raisons de croire que les données ne sont pas disponibles dans Swarm, ils peuvent les fournir ou simplement appeler `ClaimWork()` avec les données précédemment disponibles.

Truebit écrit les résultats du calcul (réussi ou non) dans le contrat Livepeer Smart, qui peut ensuite être utilisé dans les calculs de récompense et de réduction dans le protocole. Un node de transcodage ne peut prédire à l'avance quels segments seront vérifiés, et les pénalités suivantes seront appliquées en cas de triche ou de non-transcodage correct:

- `FailedVerificationSlashAmount` sera réduit s'il échoue une vérification de Truebit.
- `MissedVerificationSlashAmount` sera réduite si elles ne parviennent pas à fournir les revendications de transcodage et invoquent Truebit sur les segments pour lesquels elles étaient tenues de le faire.
Frais perdus du diffuseur.
Non seulement le transcodeur sera-t-il réduit, mais tous leurs délégués seront également réduits. Ils prendront ce compte en compte dans la décision de qui déléguer, et Transcoder pourrait perdre le travail lucratif qu’ils occupent.

Il est important qu'il soit plus rentable de simplement miser LPT sur un transcodeur valide et honnêtement performant, plutôt que de tricher et de prendre des sanctions très sévères tout en percevant des frais et des allocations de jetons pour un travail malhonnête. Une sélection judicieuse des paramètres de réduction et du taux de vérification peut y contribuer.

#### Une note sur Truebit

*Bien que le protocole utilise Truebit pour permettre une vérification du travail totalement sans confiance, il peut s'avérer nécessaire en pratique d'utiliser les solutions disponibles qui fournissent une vérification sans le degré de confiance que Truebit peut offrir alors que Truebit est encore en développement et en test. Certaines options, classées par degré de confiance, incluent:*

*1. Oracle basé sur l'API Livepeer - Faites confiance à Livepeer pour vérifier le calcul. Très centralisé, pas idéal pour rien au-delà des tests.*  
*2. Service de calcul Oraclize - Faites confiance à une entreprise qui fournit des preuves de calcul et dont toute la réputation repose sur la mise en chaîne de données externes sur une chaîne avec des preuves que celles-ci n'ont pas été falsifiées.*  
*3. Enclaves matérielles sécurisées - Des services tels qu'Intel SGX ou TownCrier fournissent des environnements informatiques sécurisés. Confiance que leur implémentation matérielle est correcte et sécurisée. Cela peut être décentralisé et audité.*


### Génération des Tokens

Livepeer est inflationniste en ce sens que de nouveaux tokens seront générés et attribués dans le temps conformément au calendrier indiqué ci-dessous dans [Distribution de tokens](#distribution-de-tokens). Si tous les rôles dans Livepeer se comportent conformément au protocole, les tokens nouvellement générés seront attribués aux utilisateurs proportionnellement à leur mise en jeu (moins les frais). Les transcodeurs ont pour rôle d’appeler la fonction `Reward()` afin de déclencher la nouvelle attribution de token ou de nouvelles barres obliques pouvant être calculées à partir de toutes les données disponibles en chaîne.

Chaque transcodeur devra appeler `Reward()` une fois par tour.

- Assurez-vous qu'un transcodeur actif appelle `Reward()`.
- Assurez-vous que le transcodeur n'a pas encore appelé `Récompense()` dans ce tour.
- Calculez le nombre de tokens à generer en fonction du `InflationRate`. Generer ce nombre de tokens.
- Calculez la coupe du transcodeur en fonction de leur `BlockRewardCut`.
- Distribuez ceci dans la participation liée du transcodeur.
- Répartissez le reste dans le pool de récompenses des délégués.
- Mettez à jour le nombre de tokens liés à ce transcodeur.

Si vous n’appelez pas `Reward()`, vous perdez une partie des attributions de jetons, ce qui nuit à la réputation de Transcoder lorsqu’il s’agit d’être élu par les mandants pour le rôle.

### Slashing

Comme mentionné précédemment, les conditions de slashing sont:

- Échec d'une vérification
- Avoir omis d'invoquer la vérification lorsqu'il est requis de le faire
- Ne pas effectuer une part proportionnelle du travail requis sur la plate-forme en fonction de la participation déléguée

L'un des avantages de l'intégration dans l'écosystème Ethereum réside dans les effets réseau que vous pouvez tirer du fait de pouvoir vous baser sur d'autres protocoles tels que Truebit et Swarm / SWEAR. Malheureusement, avec la dépendance de ces systèmes externes, qui eux-mêmes ont des dépendances et des incitations externes, il est possible qu’une faille ou une faiblesse de l’un de ces protocoles entraîne une réduction des coûts dans Livepeer.

Par exemple, si un travail de vérification Truebit restait dans leur file d'attente pendant une longue période sans qu'aucun solveur ou vérificateur ne le réclame, Livepeer ne verrait pas le résultat de cette vérification à temps avant l'appel de `Reward()`. Ou si le réseau Swarm subissait une partition et ne pouvait pas propager le fichier au vérificateur Truebit à temps, cela pourrait également créer un problème.

Ces risques peuvent être atténués en incitant les participants au protocole Livepeer à jouer ces rôles en interne, qui pourraient juger dans leur intérêt de servir de vérificateurs Truebit ou de nodes Swarm. Mais il existe aussi une autre approche qui introduit le concept de seuils de probabilité sur les paramètres de réduction. Des variables de protocole facultatives telles que `VerificationFailureThreshold` peuvent être définies pour indiquer que, tant que le noeud réussit 99% des vérifications, elles ne seront pas réduites par exemple. Cela restera un autre domaine de recherche à élaborer avant le déploiement du réseau.

L'échec de l'appel de la condition de barre de vérification peut être vérifié et invoqué par tout participant au protocole Livepeer. Il existe un `FinderFee` qui spécifie le pourcentage du montant de la barre oblique que le chercheur recevra comme récompense pour avoir invoqué avec succès cette condition de réduction.

Le reste des fonds réduits entrera dans le `CommonPool`, qui peut être utilisé ou alloué à des utilisations communes telles que le développement ultérieur de l'écosystème, conformément au mécanisme de gouvernance du protocole.

### Distribution de Tokens

En tant que token qui représente la capacité de participer et d’effectuer des travaux sur le réseau via un algorithme d’implantation DPoS, la distribution initiale du token Livepeer suivra les modèles d’autres systèmes DPoS qui nécessitent un état de genèse largement distribué.

Une attribution initiale du jeton sera distribuée à la communauté à la genèse et aux premières étapes du réseau. Les destinataires peuvent l'utiliser pour jouer le rôle de transcodeur ou de délégant. Une partie sera allouée aux groupes ayant fourni du travail préalable et de l'argent pour le protocole avant la genèse, et une partie sera dotée pour le développement à long terme du projet principal.

Lors du lancement du réseau, l'émission de tokens se poursuivra selon un calendrier inflationniste, le token étant généré à `InflationRate` par tour par rapport au flottant de tokens en circulation. Comme le token est émis proportionnellement à la participation de tous les participants cautionnés au protocole, il sert à encourager la participation active. Les participants sont "protégés" de cette inflation, car ils gagnent leur part proportionnelle. Ce ne sont que les participants inactifs qui restent assis sur un token sans le céder à la participation, et qui verront la propriété de leur réseau proportionnelle réduite à néant par cette inflation.

L’objectif initial pour `InflationRate` sera fixé de manière à inciter environ ParticipationRate à être lié et à participer activement [[19](#Références)). Par exemple, si `ParticipationRate` est égal à 50%, des incitations existent pour que la moitié du token exceptionnel soit liée. Le taux d'inflation évoluera de manière algorithmique à chaque tour pour inciter la cible de participation. Un taux d'inflation plus élevé inciterait davantage de personnes à être liées, et un taux plus bas inciterait davantage de personnes à choisir des liquidités plutôt que de participer. C'est ce compromis entre la préférence de liquidité et le pourcentage de propriété du réseau qui devrait trouver un équilibre en raison d'un certain nombre de facteurs économiques du réseau.

### La gouvernance

Le rôle de la gouvernance dans le protocole Livepeer est destiné à être triple:

1. Déterminez si vous brûlez ou appropriez des fonds communs qui ont été éliminés des nœuds qui se comportent mal.
2. Ajustez les paramètres du réseau pour assurer un réseau sain et prospère, ce qui est précieux pour les diffuseurs.
3. Invoquez les mises à jour de protocole proposées de manière décentralisée.

De nombreux paramètres de réseau référencés dans ce document, tels que `UnbondingPeriod`, `RoundLength`, `ParticipationRate` et `VerificationRate`, sont ajustables. Des propositions d'ajustement de ces paramètres peuvent être soumises et le processus de gouvernance, y compris le vote par transcodeurs proportionnellement à leur participation déléguée, déterminera automatiquement l'adoption de ces modifications dans le protocole. La spécification détaillée pour la gouvernance est laissée pour un autre document. [Voir plus ici](https://github.com/livepeer/wiki/wiki/Governance).


## Les Attaques

Cette section décrit les différentes manières dont des acteurs malveillants peuvent tenter d’attaquer le réseau Livepeer. Nous utilisons un modèle d'attaquant rationnel dans lequel l'attaquant prend des décisions en fonction de son propre intérêt économique. Un certain nombre d'attaques sont atténuées du fait qu'il est non rentable de mener de telles attaques, mais nous nous efforçons également de faire en sorte qu'au pire, le réseau subisse une perte d'efficacité dans le cas d'une attaque non rentable prolongée et ne subisse aucune défaillance.

## Attaques de consensus

Comme mentionné précédemment, le consensus dans l'écosystème Livepeer est fourni par la plate-forme de blockchain sous-jacente (Ethereum par exemple). 51% d'attaques, le double-dépense de Livepeer token et les fourches du réseau nécessiteraient les mêmes ressources et le même coût d'attaque que Ethereum lui-même.

Livepeer is a protocole implicitement, et bien que les transcodeurs se sont déroulés dans le processus de vérification du processus et dans le processus de distribution des récompenses des tokens, ils n'ont pas encore été transformés en rôle ou en valeur. autres transcodeurs. Il n'y a pas de concept de chaîne ni de validation des blocs précédents. Il existe simplement des incitations économiques à vérifier son propre travail et à distribuer son propre part d'allocations de tokens lorsque c'est son tour. En tant que tel, que l'attaque s'est manifestée dans une preuve de protocole d'enjeu, que ce soit l'attaque à long terme, le problème il n'y a aucune possibilité de signer plusieurs blocs ou de créer. une chaîne plus longue d'un état antérieur. Cependant, vous êtes sûr que, lorsqu’il a fallu que la chaîne de télévision sous-jacente apparaisse, vous êtes menacé de mort.

Bien que la sécurité de la blockchain sous-jacente soit efficace pour la prévention des attaques consensuelles, il existe toujours une classe d’attaques de qualité et d’efficacité pouvant nuire au réseau Livepeer.

### DDoS

Le déni de service dans Livepeer peut aller de deux manières:

1. Un transcodeur peut essayer d'empêcher ou de ralentir un radiodiffuseur d'envoyer son flux codé sur le réseau en acceptant un travail mais en refusant le transcodage.
2. Un radiodiffuseur peut empêcher un transcodeur de faire le travail qu’il croit avoir été assigné en refusant de leur envoyer des segments.

Les deux attaques ont un coût et peuvent être atténuées, avec une légère contrariété.

Dans le premier cas, un transcodeur doit payer pour réclamer sa disponibilité en chaîne. S'ils ne reçoivent pas de frais parce qu'ils n'ont pas fait le travail, ils jettent l'ETH. Le radiodiffuseur peut simplement renvoyer le travail et se voir attribuer un nouveau transcodeur. Une option potentielle d’évolutivité est que le protocole puisse identifier un certain nombre de transcodeurs valides par ordre de priorité plutôt que par un seul, ce qui permet au radiodiffuseur de continuer sans autre transaction en chaîne. De plus, toutes les statistiques sur les travaux acceptés et le nombre moyen de segments transcodés / travail, etc., peuvent être calculées à partir de données sur la chaîne, et les mandataires l'utilisent pour entrer dans leur décision quant aux destinataires de la délégation. Tiens-toi mal et perds ton rôle.

Dans le cas d’un radiodiffuseur empêchant un transcodeur de fonctionner, il s’agit simplement d’un calcul de planification de la capacité. Un node de transcodage peut conserver des enregistrements de sa capacité pour des travaux simultanés, de la probabilité qu'un travail soit actif / inactif, et s'assurer qu'il a toujours la conviction qu'il aura la capacité nécessaire pour le travail qu'il réclame. Le simple fait d'ignorer ou d'appeler `EndJob()` sur un node qui refuse d'envoyer des segments ne nuit guère au transcodeur.

### Transcodeur Inutile ou Autorégulant

Si un transcodeur dispose de suffisamment de participation pour conserver sa position, il pourrait théoriquement lister 100% `BlockRewardCut`, 0% `FeeShare`, et facturer un `PricePerSegment` élevé de sorte qu’il n’ait jamais à faire de travail, tout en pouvant collecter son allocation de tokens. Ceci est empêché par la `CompétitivitéTolérance` qui les oblige à fournir une certaine quantité de travail valable. De plus, en raison des coûts de transaction liés à la participation au protocole supportés par Transcoders, il serait plus rentable pour eux de s’en remettre simplement à un Transcoder valide partageant les frais avec eux, que de se comporter comme un Transcoder inutile ne recevrait aucun frais à proprement parler.

Un Transcodeur qui se comporte mal et qui produit une sortie non valide se trouve rapidement réduit au point de voir son enjeu réduit trop bas pour pouvoir conserver son travail et recevoir du travail.

### Griefing du Transcodeur

Si un radiodiffuseur souhaite rendre le protocole très coûteux à exploiter pour un transcodeur, il peut envoyer aux transcodeurs des numéros de segment non consécutifs. En effet, les transcodeurs peuvent prétendre travailler pour une plage continue de numéros de segment dans une transaction unique, mais ils doivent effectuer plusieurs transactions pour revendiquer un travail sur des plages de numéros de segment aléatoires. Cela peut être défendu par les options suivantes:

1. Transcoder appelle `EndJob()` et ne se donne pas la peine de faire le travail ou d'essayer de percevoir les frais.
2. Le protocole met en œuvre l'analyse en chaîne ou un meilleur codage de revendication de segment afin de réduire les frais associés à la revendication de segments non consécutifs dans un seul appel.
3. Ignorez simplement les segments et ne réclamez jamais le travail.

Cette attaque a un coût élevé pour un radiodiffuseur car il doit disposer d’un dépôt et soumettre des travaux en chaîne afin d’être affecté à un transcodeur. Ils ont la capacité de rendre la vie agaçante pour un transcodeur et de perdre potentiellement en efficacité, mais sans endommager le réseau.

### Chain Reorg

Lorsqu'un diffuseur soumet une tâche au Livepeer Smart Contract, le protocole utilise le hachage de bloc actuel pour déterminer le transcodeur auquel la tâche sera affectée. Les réorganisations de la blockchain sous-jacente peuvent être source de confusion dans ce scénario. Bien que ce ne soit pas "une attaque" directement, un transcodeur sera valide une seconde, puis lors de la réorganisation, ne sera plus valide. Quand une réorg est détectée, le diffuseur peut soit rediriger le flux vers le nouveau transcodeur valide, soit le protocole peut détecter les blocs oncle inclus dans la chaîne principale et considérer qu'un transcodeur est valide si un bloc oncle compris dans un seuil donné aurait fait les valides.

## Distribution Vidéo en Direct ###########################################

Ce livre blanc s'est largement concentré sur les incitations économiques et le protocole permettant d'assurer un transcodage correct de la vidéo en direct, ce qui est nécessaire pour prendre en charge le streaming adaptatif au débit et atteindre chaque périphérique. La distribution de la vidéo sur l’ensemble du réseau est également importante, de sorte qu’elle puisse être consommée avec une qualité élevée et une latence faible. Les aspects économiques de la distribution reposent sur la comptabilité de bande passante tit-for-tat popularisée par Bittorrent et étendue via des protocoles tels que SWAP [[13](#Références)]. Pour simplifier, les nodes paient pour demander un segment de vidéo et les nodes sont payés pour desservir un segment de vidéo. Si un node a déjà un segment et peut le servir à plusieurs demandeurs, il est rentable. Nous appelons ce type de node, un node de relais.

Il existe différentes incitations en termes de bande passante pour les nodes jouant différents rôles dans le réseau.

* Les consommateurs peuvent être disposés à échanger de la bande passante en amont afin de servir le contenu à d'autres consommateurs en échange de la possibilité de consommer la vidéo eux-mêmes gratuitement. Voir des systèmes comme Webtorrent [[14](#Références)].
* Les diffuseurs servent de nodes d’origine et peuvent vouloir facturer la consommation de la vidéo ou subventionner le coût de la bande passante afin que tout le monde puisse accéder à leur vidéo gratuitement.
* Les nodes de transcodeurs et de relais sont disposés à fournir de la bande passante au service de la distribution de vidéos tant qu’elles sont rentables. Ceci est similaire au rôle des CDN traditionnels.

Les `Segments` étant l’unité de base du flux de données sur le réseau, il est possible de comptabiliser la bande passante en utilisant l’ETH comme base de règlement. Nous empruntons l'abstraction du contrat de carnet de chèques à Swarm [[6](#Références)] comme méthode de paiement hors chaîne avec le règlement en chaîne. Les développements futurs dans l'écosystème, y compris le réseau Raiden [[15](#Références)], pourraient également permettre d'utiliser des canaux de paiement à cette fin. Le transfert de jetons étant natif au protocole, il est également possible d'intégrer la tarification associée au contenu directement dans le protocole. Un radiodiffuseur peut facturer son temps ou son contenu directement, et les nodes opteront pour ce transfert de valeur en payant un prix / segment plus élevé qui sera ensuite redistribué au diffuseur.

Il est important de noter que bien que la comptabilité de bande passante puisse être utilisée pour rentabiliser les nodes relais qui transmettent simplement des segments vidéo sur le réseau pour augmenter la capacité, à la CDN, ces nodes sont uniquement motivés par la demande de contenu, et pas incité par de nouvelles attributions de tokens. En fait, la sortie de Livepeer peut être insérée dans un CDN traditionnel (comme Amazon S3, Cloudflare, etc.) ou décentralisé (comme IPFS ou Swarm). Le développement de ce protocole peer-to-peer pour la distribution de segment vidéo sera une opportunité permanente d'optimisation et d'amélioration des performances.

Il a été démontré que les réseaux CDN d'égal à égal réduisent de 80 à 98% les besoins en bande passante sur un serveur CDN d'origine [[17](#Références)], et la mécanique des jetons observée dans les réseaux décentralisés peut permettre aux parties prenantes de s'aligner sur le développement et la maintenance d'une version ouverte du système propriétaire P2N CDN qui existent aujourd'hui. Le protocole PPSPP [[18](#Références)] constitue un candidat viable pour une mise en œuvre ouverte axée sur la diffusion de contenu en direct.

En tant que facteur non critique pour la crypto-économie du protocole Livepeer, les détails sont épargnés par ce document, mais les personnes intéressées peuvent [suivre ici](https://github.com/livepeer/go-livepeer) le développement et rechercher un futur document traitant uniquement du protocole de distribution vidéo.

## Cas d'utilisation ###########################################

Le projet Livepeer concerne la décentralisation de la diffusion vidéo en direct un à plusieurs (multidiffusion). Il s’agit de la forme la plus authentique de diffusion dans les médias, car elle permet à un radiodiffuseur de se connecter directement avec son auditoire, de manière directe, sans altération, interprétation après coup ni effet. Cela donne à chacun une tribune pour s'exprimer. Les solutions centralisées existantes peuvent pâtir de la censure, du contrôle de tiers sur les données / relations / monétisation des utilisateurs et des structures de coûts inefficaces liées au paiement du service. Voici quelques cas d’utilisation logique pour les applications et les services à construire sur Livepeer.

### Consommation de Contenu à la Carte

Avec une transaction de transfert de valeur intégrée au protocole, il est désormais possible pour les radiodiffuseurs de facturer directement aux téléspectateurs la consommation de leur diffusion en direct, sans nécessiter de carte de crédit, de compte ni de contrôle de l'identité de l'utilisateur via une plate-forme centralisée. Cela a des applications dans l'éducation (payer pour assister à un cours en ligne), événements (payer pour voir un concert ou un événement sportif en direct), divertissement (payer pour regarder le flux en direct d'un joueur ou d'un artiste), et de nombreux autres cas d'utilisation - tout en préservant la la vie privée du spectateur et lui permettant de ne payer que ce qu’il consomme directement au radiodiffuseur.

### Mise à l'Echelle Automatique des Services de Vidéo Sociale

L'un des défis de la création de services vidéo grand public consiste aujourd'hui à faire évoluer l'infrastructure pour répondre à la demande liée au nombre croissant de flux et au nombre croissant de consommateurs à mesure que de nouveaux utilisateurs s'ajoutent. Une couche de services qui laisse facilement les développeurs commencer à construire leur solution vidéo sur le réseau Livepeer, qui évoluera automatiquement pour prendre en charge un nombre illimité de flux et de visionneuses au fur et à mesure, constituera une solution bienvenue pour les développeurs d’infrastructure qui, sinon, devraient continuer à provisionner serveurs, octroyer des licences aux serveurs de médias et gérer efficacement les ressources pour gérer les pointes.

### Journalisme en Direct Sans Censure

Les plates-formes actuelles telles que Twitter et Facebook offrent des solutions vidéo en direct incroyables pour atteindre un large public, mais elles sont également les premières à être bloquées ou censurées dans diverses situations de conflit politique. L'utilisation d'un réseau décentralisé tel que Livepeer rendrait presque impossible d'empêcher le mot de s'exprimer sur ce qui se passe réellement sur le terrain en temps réel.

### DApps vidéo activés

Les applications décentralisées (DApps) commencent à émerger, principalement grâce à l'écosystème Ethereum. Cependant, à ce jour, il n’existait pas de solution viable pour incorporer de la vidéo en direct dans un DApp sans utiliser de solution centralisée ni limiter le nombre de clients consommateurs en fonction des contraintes de WebRTC. En introduisant Livepeer dans la pile, une application peut être entièrement décentralisée, tout en conservant la vidéo en direct, à grande échelle, pour autant d'utilisateurs que de le souhaiter.

## Résumé ###########################################

En résumé, le protocole Livepeer incite les nodes à apporter leur traitement et leur bande passante au réseau au service du transcodage et de la distribution de vidéo en direct. La vérification du travail est résolue par une extension évolutive s'ajoutant au protocole Truebit, qui incite les nœuds à effectuer correctement les opérations de transcodage afin de gagner leurs frais et les allocations de jetons et de préserver leur rôle de transcodeur. La gamification du réseau et le problème de faux travail sont résolus via l’économie de la preuve déléguée de la comptabilisation des récompenses de bloc de mise. Il devient économiquement plus rationnel de simplement investir ses tokens vers un node à valeur ajoutée plutôt que de payer des frais sur le réseau pour les distribuer à d'autres délégants lors de l'exécution de travaux pour lesquels il n'existait pas vraiment de demande.

Le résultat final est un réseau évolutif, à la carte, pour la diffusion vidéo en direct décentralisée - une couche manquante dans la pile web3 que Livepeer cherche à remplir.

## Appendice ###########################################

### Référence de paramètre de protocole Livepeer

Nom du paramètre
Description
Exemple de valeur
T
Longueur du segment en secondes.
2 secondes
N
Nombre de transcodeurs actifs.
144
RoundLength
Délai entre l'élection d'un nouveau cycle de transcodeurs.
1 jour
InflationRate
Le taux d'inflation cible actuel par tour de LPT. (Se déplace algorithmiquement).
.04% (equivalent de 15%/année)
ParticipationRate
Le pourcentage cible de token lié ou liquide.
50%
RoundLockAmount
Les taux des transcodeurs se verrouillent pour ce pourcentage d'une partie à la fin d'une série afin que les mandataires puissent vérifier et déléguer en conséquence sans se soucier des modifications de taux de dernière minute.
10% == 2.4 heures
UnbondingPeriod
Temps entre l’état non lié et la capacité retirée des fonds.
1 mois
VerificationPeriod
Date limite de vérification d'une attestion de travaille après la soumission de cette attestation. Cette période sert également de délai minimum pour la réception d'une persistance des données dans la solution de stockage décentralisé.
6 heures
VerificationRate
Le% de segments qui seront vérifiés.
1/500
FailedVerificationSlashAmount
% à réduire en cas d'échec de la vérification (au-delà du seuil d'échec potentiel autorisé).
5%
MissedRewardSlashAmount
% à réduire dans le cas où il manque un tour de récompense de bloc (peut-être le faire uniquement dans le cas de n manquements consécutifs).
3%
MissedVerificationSlashAmount
% à réduire si le transcodeur n’appelle pas la vérification.
10%
CompetitivenessTolerance
Si tous les transcodeurs étaient toujours disponibles et fixaient le même prix et les mêmes frais, ils recevraient un travail proportionnel à leur participation. Ce paramètre définit le % qu'ils doivent être dans ce travail cible % pour pouvoir prétendre à une allocation de jetons. Cela empêche les transcodeurs de faire très peu de travail par rapport à leur enjeu.
90% (Exemple extrême. Avec 100 transcodeurs et 100 000 segments, cela signifie que je vais bien si je n'ai fait que 100 segments (10% des 1 000 que je devais faire)).


*SlashingThresholds (TBD)
Espace réservé pour indiquer que nous ne pouvons pas réduire toutes les pannes, uniquement si elles dépassent un certain seuil de % de taux de défaillance.


VerificationFailureThreshold
% des vérifications que vous pouvez échouer sans être coupé. Utile en raison de dépendances externes telles que Swarm / Truebit qui pourraient provoquer une défaillance sporadique.
1%
FinderFee
% du montant de la barre oblique que le viseur recevra à titre de compensation.
5%
SlashingPeriod
Le délai pour invoquer une condition de barre oblique après la VerificationPeriod a completé.
1 heure

Types de transaction de protocole Livepeer

Transaction
Description
Bond()
Obligation de liaison vers un transcodeur.
Unbond()
Entrez l'état non lié pour le fixe UnbondingPeriod.
Transcoder()
Déclarez vos intentions en tant que transcodeur.
ResignAsTranscoder()
Renoncez à vos intentions en tant que transcodeur.
TranscodeAvailability()
Ces transcodeurs sont actuellement ouvert à accepter un autre travail. Ils sont dans le pool pour être assignés au hasard sur de nouvelles offres d’emploi.
Job()
Soumettez un travail de transcodage sur une chaîne.
EndJob()
Terminez le travail pour abandonner la responsabilité de transcodage.
Deposit()
Soumettez un dépôt sur la chaîne qui sera utilisé et utilisé pour payer des emplois.


Withdraw()
Retrait du dépôt et de la mise non liée.
ClaimWork()
Terminez le travail de transcodage et indiquez quels segments vous pouvez prouver que vous avez transcodé via la plage de segments et la racine Merkle.
DistributeFees()
Transcoder réclame les frais pour une réclamation particulière après vérification.
Reward()
Toutes les vérifications sur la chaîne réduisent-elles ou distribuent-elles des allocations de tokens? Ne peut être invoqué que par un transcodeur actif dans le tour en cours, une fois par tour.
Verify()
Transcoder fournit les revendications de transcodage pour les segments qui seront vérifiées avec les preuves de moukle pour comparaison avec la racine de Merkle de
ClaimWork(). Appelez explicitement Truebit pour effectuer la vérification.
InitializeRound()
Cette transaction doit être appelée une fois après le bloc de début de la nouvelle série pour initialiser le nouveau pool de transcodeurs actif.
UpdateDelegatorStake()
Cela permet à un délégant de réclamer ses honoraires + l'allocation de tokens des tours précédents. Il est automatiquement appelé par dissociation et liaison, mais il constitue un élément de sécurité au cas où le délégant souhaite mettre à jour sans changer d'état.
*GovernanceTransactions()
À déterminer.


Références

Ethereum White Paper - Vitalik Buterin - Ethereum Wiki - https://github.com/ethereum/wiki/wiki/White-Paper
Fat Protocols - Joel Monegro - USV Blog - http://www.usv.com/blog/fat-protocols
Crypto Tokens and the Coming Age of Protocol Innovation - Albert Wenger - http://continuations.com/post/148098927445/crypto-tokens-and-the-coming-age-of-protocol
The Case For SectorCoins - Eric Tang - https://medium.com/@ericxtang/case-for-sectorcoins-b70a7c820c2d#.7892n4a57
Delegated Proof-of-Stake Consensus - Daniel Larimer - https://bitshares.org/technology/delegated-proof-of-stake-consensus/
Truebit - Jason Teutsch and Christian Reitweisner - https://people.cs.uchicago.edu/~teutsch/papers/truebit.pdf
swap, swear and swindle, incentive system for swarm - viktor trón, aron fischer, dániel a. nagy, zsolt felföldi, nick johnson - http://swarm-gateways.net/bzz:/theswarm.eth/ethersphere/orange-papers/1/sw%5E3.pdf
Kademlia: A Peer-to-peer Information System Based On The XOR Metric - Petar Maymounkov and David Mazieres https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf
Steem Whitepaper - Daniel Larimer, Ned Scott, Valentine Zavgorodnev, Benjamin Johnson, James Calfee, Michael Vandeberg - https://steem.io/SteemWhitePaper.pdf
Introducing Casper "the Friendly Ghost" - Vlad Zamfir - https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/
Tendermint Docs - Jae Kwon and Ethan Buchman - https://tendermint.com/docs
Swarm - http://swarm-gateways.net/bzz:/theswarm.eth/
Incentives Build Robustness in BitTorrent - Bram Cohen - http://bittorrent.org/bittorrentecon.pdf
WebTorrent - https://webtorrent.io/
Raiden Network - http://raiden.network/
ERC20 Token Standard - https://github.com/ethereum/EIPs/issues/20
Peer5 leverages viewers’ devices for a P2P approach to streaming video - https://techcrunch.com/2017/01/26/peer5-y-combinator/
Peer-to-Peer Streaming Peer Protocol - https://tools.ietf.org/html/rfc7574
Inflation and Participation in Stake Based Protocols - Doug Petkanics - https://medium.com/@petkanics/inflation-and-participation-in-stake-based-token-protocols-1593688612bf


