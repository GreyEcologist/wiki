# Livre Blanc Livepeer Streamflow

**Évolutivité de Livepeer sur Ethereum via orchestration, micropaiements probabilistes et négociation de tâches hors chaîne**

**Auteurs**   
Doug Petkanics <doug@livepeer.org>    
Yondon Fu <yondon@livepeer.org>   

**Chercheurs**    
Eric Tang <eric@livepeer.org>   
Philipp Angele <philipp@livepeer.org>   
Josh Allmann <josh@livepeer.org>    

**STATUS: PROPOSITION - Des commentaires et des revues sont demandés sur cette première proposition.**


## Abstrait #####################################

La proposition Streamflow introduit des mises à jour du protocole Livepeer et des implémentations hors chaîne qui permettront à Livepeer de s’adapter au-delà des limites actuelles du protocole alpha déployé dans la blockchain Ethereum. Il suggère des mises à jour qui abordent les questions d’abordabilité, de fiabilité, de performance et d’évolutivité du réseau. Les éléments clés sont introduits, notamment un registre de services, un mécanisme de négociation et de paiement hors connexion, une séparation entre les nodes d’orchestration et de transcodage, l’élimination de la solution du problème de disponibilité des données en tant que dépendance de la vérification sans confiance et l’ouverture du nombre de nodes qui peuvent rivaliser pour effectuer des travaux sur le réseau à partir des limites arbitraires basses au cours de l’alpha. L’architecture résultante permettra aux utilisateurs du réseau d’effectuer des travaux de transcodage à grande échelle sur le réseau auprès de nombreux fournisseurs de travaux simultanés, tout en réduisant considérablement l’impact de la demande et de la volatilité des prix de la blockchain sous-jacente sur la viabilité économique de l’utilisation du réseau.


## Table des Matières #####################################

* [Introduction et contexte]
* [Proposition de protocole de flux]
    * [Orchestrateurs et transcodeurs]
    * [Assouplissement de la sécurité du transcodeur et de la sécurité renforcée]
    * [Registre de service]
    * [Négociation des emplois hors chaîne]
    * [Micropaiements probabilistes]
    * [Vérification de la chaîne basée sur l'erreur]
* [Analyse économique]
    * [Jeton de Livepeer]
    * [La délégation en tant que signal de sécurité et de réputation]
    * [Inflation dans les états liés et délégués apathiques]
    * [Considérations d'ingénierie hors chaîne]
* [Les attaques]
    * [Pressant délégué]
    * [Délégation de vol]
* [Zones de recherche ouvertes]
    * [Vérification non déterministe]
    * [Protocoles de reservoir de transcodeur publics]
    * [Diffuseur Doublespend Mitigation]
    * [Paiements VOD]
* [Voie de migration]
* [Annexe]
    * [Annexe A: Flux de travail probabiliste sur les micro-paiements]
* [Références]

## Introduction et Contexte #####################################

Le protocole Livepeer encourage et sécurise un réseau décentralisé de nodes de transcodage vidéo. Les utilisateurs qui souhaitent transcoder une vidéo peuvent soumettre une tâche au réseau à un prix qu’ils jugent acceptable, se voir attribuer un transcodeur, le faire effectuer et effectuer le transcodage avec des garanties de précision économiquement sûres. Le protocole en direct utilise un mécanisme basé sur la participation déléguée pour élire les nodes qui sont jugés suffisamment fiables et de haute qualité pour effectuer l'encodage vidéo en direct de manière opportune et performante.

La version alpha du protocole actuellement déployé sur la blockchain Ethereum a mis en œuvre de nombreuses conceptions spécifiées à l'origine dans [le livre blanc Livepeer](https://github.com/livepeer/wiki/blob/master/WHITEPAPER.md). Le système basé sur la participation déléguée, avec ses incitations inflationnistes, s'est avéré efficace pour inciter à la participation et pour créer un réseau précoce de transcodeurs et de délégants engagé pour effectuer le travail de transcodage et l'assurance qualité en conséquence. Le réseau est utilisable et, pour un certain nombre de cas d'utilisation tels que le transcodage à durée de vie longue ou le prototypage d'applications décentralisées, est une option viable aujourd'hui à ses débuts. Cependant, pour l'utilisation à plus grande échelle des services d'infrastructure vidéo, la version alpha présente les faiblesses suivantes:

1. Le coût d'utilisation du réseau est trop corrélé aux fluctuations de la tarification du gaz Ethereum et, par conséquent, en cas de prix élevés du gaz ou de scénarios de codage nécessitant de nombreuses transactions, le réseau devient trop coûteux pour être viable par rapport aux solutions centralisées.
1. L'affectation de tâches et la négociation de transcodeur sur chaîne basées sur des enjeux créent des scénarios non fiables pour le diffuseur. Si leur transcodeur attribué est mis hors ligne, ils encourent des coûts et des délais supplémentaires pour la négociation d'un deuxième transcodeur, ce qui peut être extrêmement perturbant dans un contexte de diffusion en direct.
1. Le problème de disponibilité des données reste non résolu (en production), et par conséquent, la vérification du travail ne peut pas être totalement sans confiance et non interactive.
1. Les transcodeurs n'ont aucun moyen de gérer leur disponibilité pour effectuer ou non des tâches en fonction de la capacité et de la charge de travail qui dépassent l'enjeu.
1. Bien que le réseau encourage la concurrence par les prix, il n'encourage pas directement la concurrence en matière de performances et la responsabilité.
1. Les limites actuelles des limites de gaz Ethereum et la mise en œuvre pratique du protocole limitent à un nombre très bas le nombre de transcodeurs actifs pouvant être actifs à tout moment, créant ainsi un obstacle important à l’entrée sur le marché pour les travaux sur le réseau.

La suite de ce livre blance propose des solutions qui abordent chacune de ces faiblesses. Il débute par une description des propositions de modification d’architecture et de protocole. Il analyse ensuite les impacts économiques de ces modifications sur le réseau, avant d’aborder les attaques éventuelles. Il poursuit en reconnaissant les domaines de recherche ouverts pouvant contribuer à faire passer cette proposition d’une sécurité économique et sociale / basée sur la réputation à une sécurité fortement, cryptographiquement assurée. Et enfin, nous conclurons avec quelques réflexions sur une voie de migration du protocole alpha vers Streamflow dans le réseau en direct, si la communauté souhaite accepter ces modifications.

Ce document décrit les modifications apportées au protocole conceptuel et analyse leur impact, mais laisse les spécifications spécifiques pour leur mise en œuvre à un document SPEC associé qui sera fourni dans le cadre du processus de proposition d'amélioration Livepeer (LIP).

_Remarque: pour absorber correctement les mises à jour de protocole, il est important de comprendre le fonctionnement du protocole Livepeer actuel, comme décrit dans le livre blanc [1].


## Proposition Streamflow #################################

Cette proposition introduit un certain nombre de modifications et de nouveaux concepts dans l'écosystème Livepeer. Chacune de ces solutions a des répercussions sur un ou plusieurs des domaines suivants: abordabilité, performance, fiabilité ou extensibilité. Ils comprennent:

* Introduction d'un nouveau rôle d'orchestrateur, aux rôles existants de diffuseurs et de transcodeurs.
* Détente sur la limitation du nombre de transcodeurs, permettant un accès libre pour concourir au travail entre tous les orchestrateurs détenteurs de jetons en herbe répondant aux exigences minimales de sécurité et de mise.
* Registre de services dans lequel les orchestrateurs annoncent leurs capacités et services disponibles en chaîne.
* Négociation des prix hors chaîne et attribution des tâches entre radiodiffuseurs et orchestrateurs.
* Paiements hors chaîne utilisant des micropaiements probabilistes, avec règlement en chaîne et dépôts de garantie.
* Flux de vérification mis à jour, dans lequel la vérification sur chaîne ne doit être effectuée que dans le cas d'une défaillance constatée.


## Orchestrateurs et transcodeurs #################################

Actuellement, un transcodeur sur le réseau Livepeer est un node prenant en charge le protocole qui surveille et interagit avec le protocole Blockchain et effectue un travail de transcodage vidéo. En bref, il orchestre et travaille sur le réseau et transcode la vidéo. Cela peut créer des problèmes de performances et de fiabilité, et empêcher les nodes de faire évoluer leurs opérations. Streamflow propose une architecture à deux niveaux, qui contient une division entre:

* Un orchestrateur prenant en charge le protocole, négocie les travaux avec les diffuseurs, est chargé de fournir aux diffuseurs des segments transcodés vérifiés et coordonne l’exécution de cette tâche au sein d’un vaste reservoir de transcodeurs.
* Un transcodeur qui n’est pas nécessairement au courant du protocole de jalonnement Livepeer ou de la blockchain, mais qui n’est qu’un matériel concurrentiel et rentable, qui se contente de courir pour transcoder la vidéo de la manière la plus économique et la plus rapide possible, coordonnée par les orchestrateurs.

<img src="https://livepeer-dev.s3.amazonaws.com/docs/otsplit.jpg" alt="Orchestrator Transcoder Split" style="width:750px">

Le premier niveau de cette architecture est similaire au protocole Livepeer actuel, dans lequel les transcodeurs actuels sont renommés en orchestrateurs. Ces orchestrateurs demandent à LPT de fournir des cautionnements contre le travail qu’ils effectuent, de sorte que s’ils endommagent le réseau, ils encourent une pénalité économique. Les radiodiffuseurs sont conscients de ces orchestrateurs, négocient des tâches avec eux et reçoivent en retour des segments transcodés, avec la possibilité de réduire les orchestrateurs s’ils effectuent un travail malhonnête.

Le deuxième niveau de cette architecture est un nouveau concept appelé reservoir de transcodeur. La tâche qui consiste à faire le transcodage vidéo le plus rapidement et le moins cher possible peut toutefois être effectuée par des GPU disposant de la capacité disponible, tels que ceux décrits dans la proposition Video Miner[[2](#Références)], pour lesquels NVENC asics est inactif. Ce matériel devrait juste être en mesure de rivaliser pour effectuer le travail de codage proprement dit, et les forces concurrentielles et les résultats économiques devraient entraîner des prix beaucoup moins élevés que si les orchestrateurs eux-mêmes devaient effectuer le travail sur les mêmes configurations basées sur le processeur, nécessaires pour exécuter une orchestration à base de blocs et de protocoles. sur le réseau.

Ceci est analogue aux reservoirs d’extraction de crypto-monnaie actuels, dans lesquels la mise en œuvre des reservoirs eux-mêmes peut être centralisée ou décentralisée; ils peuvent être gérés par l'opérateur de reservoir central eux-mêmes ou peuvent être ouverts pour exploiter les millions d'ordinateurs disponibles à travers le monde, tous en concurrence pour exploiter le bloc suivant. Un orchestrateur peut s’assurer le transcodage de son propre reservoir, ce qui donne la même configuration que celle utilisée dans le protocole Livepeer actuel, même si, ce faisant, il doit être expert dans deux emplois très distincts avec les orchestrateurs qui ouvrent leurs reservoirs à des sources de transcodage potentiellement plus rapides ou moins chères. D'autre part, si un orchestrateur exécute son propre reservoir, il peut faire confiance aux encodages vidéo sans avoir à vérifier le travail des transcodeurs publics non fiables.

Les avantages de cette configuration à deux niveaux sont les suivants:

* Quiconque veut gagner des frais peut le faire simplement en allumant son matériel et en le faisant courir pour effectuer les tâches de transcodage disponibles. Aucune connaissance de la chaîne, crypto-monnaie, jalonnement, dépôts, n'est nécessairement nécessaire. Cela ressemble beaucoup à la façon dont tout le monde peut gagner du bitcoin en exploitant une réserve. Cependant, si l'accès à la concurrence est ouvert à tous, les transcodeurs offrant des avantages en termes d'électricité, de bande passante et de géolocalisation seront probablement plus performants que ceux fonctionnant sans ces installations concurrentielles.
* Les radiodiffuseurs bénéficient de la sécurité groupée d’un orchestrateur en chaîne, mais les implémentations de dimensionnement sous-jacentes et la sécurité des reservoirs publics et privés peuvent être laissées à l’expérimentation en dehors du protocole Livepeer.
* Les orchestrateurs peuvent se concentrer sur les opérations appropriées pour les interactions de protocole et la sécurité, plutôt que sur le dimensionnement du matériel. Un orchestrateur pourrait orchestrer des centaines de flux simultanément sans avoir à transcoder la vidéo elle-même.
* D'autres implémentations de reservoir de transcodeurs peuvent être disponibles, tirant parti des GPU, des configurations distribuées pour rechercher des emplois dans différentes régions et encourageant la concurrence, ce qui se traduira par des prix disponibles moins chers pour les diffuseurs.

Ce document décrit le protocole de communication et de sécurité du diffuseur / orchestrateur, mais il laisse le deuxième niveau du protocole orchestrateur / transcodeur à différentes implémentations. Dans le cas simple où un orchestrateur est son propre reservoir de transcodeurs, la sécurité du protocole est maintenue, tandis que d'autres compromis en termes de confiance / performance peuvent être apportés dans des implémentations alternatives pour les reservoirs de coordination, allant de centralisé et sécurisé à décentralisé, sécurisé par des enjeux et des dépôts dans la couche deux. Il est théorisé que, puisque la vérification du travail effectué par des acteurs aléatoires dans un reservoir public implique un coût supplémentaire pour un orchestrateur, les reservoirs privés peuvent surperformer les reservoirs publics, mais cela peut éventuellement être surmonté par de solides protocoles de dépôt, de réduction et de vérification cryptoéconomiques.

### Assouplissement de la sécurité du transcodeur et de la sécurité renforcée

Le deuxième changement majeur proposé par Streamflow consiste à assouplir la limite artificielle du nombre de transcodeurs (orchestrateurs actifs dans Streamflow) actifs. À la genèse, ce paramètre a été défini sur 10 et a depuis été étendu à 15, ce qui constitue néanmoins un obstacle majeur à l’entrée, dans la mesure où un node a besoin de plus en plus de LPT pour accéder au reservoir actif. Dans Streamflow, l'objectif est de supprimer cette limite arbitraire et de permettre à tout nœud offrant une sécurité suffisante sous la forme d'un accès de pieu (ou de pôle délégué) de se faire concurrence sur le réseau.

Les raisons de la limite en premier lieu étaient:

Avec le travail attribué en chaîne aux transcodeurs actifs, il était essentiel qu'ils soient en ligne et disponibles pour effectuer le travail. Une contrainte sur la disponibilité de ce poste et une visibilité facile de leurs statistiques et de leurs performances ont contribué à assurer un réseau de haute qualité.
Les limitations de gaz Ethereum sur le calcul et la comptabilité autour de cet ensemble actif ont créé une limite artificielle, qui pourrait toujours dépasser le point actuel, mais pas d'un ordre de grandeur.
Au cours de l’alpha, il était important d’être en contact étroit et en coordination avec le groupe actif afin qu’il puisse mettre à jour fréquemment le logiciel, réagir aux bugs et aider au développement et au contrôle de qualité du réseau.
Les transcodeurs actifs avaient besoin de suffisamment d’enjeu risqués pour sécuriser le réseau, de sorte que s’ils trichaient, ils se verraient infliger une lourde pénalité économique.

Les effets de la réduction de cette limite artificielle seront réalisés par:

* La négociation et le basculement des travaux hors chaîne, ce qui signifie que les orchestrateurs qui ne sont pas disponibles ou qui n’effectuent pas de travail perdront tout travail à venir, mais ne nuiront pas à l’expérience du télédiffuseur.
* Le set actif ne doit pas être calculé par tour, mais peut simplement être maintenu en place lorsque les orchestrateurs se lient, se désolidarisent ou se font couper.
* Les orchestrateurs actifs et compétitifs voudront toujours porter une attention particulière aux mises à niveau, aux bugs et au développement du réseau - mais les orchestrateurs inactifs qui ne réussissent pas ne pourront tout simplement pas attirer du travail sur le réseau sans nuire à l'expérience du diffuseur.
* Maintenant qu'un pourcentage important de la participation initiale est activement actif, les exigences peuvent être définies de manière à ce qu'il y ait suffisamment de participation et de sécurité pour sécuriser un plus grand nombre de nœuds en concurrence pour des travaux sur le réseau.

Le nombre exact d'orchestrateurs cibles et la méthode de mise en œuvre restent un problème de recherche ouvert. Au départ, il devrait y avoir une augmentation de l'ordre de grandeur - telle que des centaines d'orchestrateurs actifs au lieu de 15 - avec l'objectif de se développer à terme pour atteindre les 1000 afin de pouvoir proposer chaque service dans toutes les régions du monde avec des licenciements redondants. Voici quelques mécanismes envisagés, avec une brève description de certains de leurs compromis:

1. **Augmentez le nombre N (nombre d'emplacements d'orchestrateur) de 15 à quelque chose de beaucoup plus grand, tel que 200**: les choses fonctionneraient essentiellement de la même manière qu'aujourd'hui, avec un obstacle beaucoup moins important à l'activation d'un node. Mais cela rendrait les actions liées au collage plus coûteuses. Des problèmes d'échelle et de gaz dans Ethereum peuvent entrer en jeu.
2. **Définir un enjeu minimum requis pour devenir un orchestrateur**: cela établirait un maximum possible de N tout en permettant à quiconque de savoir exactement ce qu'il faut pour atteindre cette barre de sécurité et rester dans l'ensemble actif. Cela permettrait également à un réseau croissant d'orchestrateurs de générer un LPT inflationniste et encouragerait les délégués à rechercher activement de nouveaux orchestrateurs offrant des actions à frais, cherchant à dépasser le minimum pour devenir actifs afin de concourir pour le travail.
3. **Définissez un montant d’enjeu fixe pour tout orchestrateur**: cela obligerait les orchestrateurs à exécuter des nodes supplémentaires et les mandants à reconfigurer en permanence afin de pouvoir utiliser leur inflation de LPT. Cependant, l'expérience utilisateur qui en résulte pour les orchestrateurs et les mandataires présente certaines faiblesses, ainsi que des détails de mise en œuvre complexes.
4. **Éliminez toute exigence de mise minimale du protocole et laissez les clients configurer le montant de la mise nécessaire pour sécuriser un travail**: cela crée l'accès le plus ouvert possible et est le plus décentralisé au départ, mais offre le moins de coordination possible entre les délégués et les orchestrateurs détenteurs de jetons. créer un réseau de haute qualité - essentiellement la réputation joue un rôle plus important et pourrait donc conduire à une plus grande centralisation du travail effectué à long terme, car les délégants ont une capacité collective moins grande à acheminer le travail.

Bien que les avantages et les inconvénients des approches susmentionnées soient pris en compte, il est important de noter que le résultat de la mise en œuvre de l’un de ces objectifs sera un réseau d’orchestrateurs élargi, davantage de licenciements et une concurrence accrue au profit des diffuseurs, orientez les mises vers des nodes capables de fournir des services supplémentaires de manière fiable et rentable au réseau, moyennant des frais.

L'un des avantages des modèles de mise minimale est que, lorsque les frais circulent dans le réseau, il n'y a aucune raison de faire fonctionner un nœud qui ne fait pas concurrence pour le travail sur le réseau. Le nombre de créneaux horaires est limité et cette mise est mieux utilisée pour déléguer vers un nœud qui fournirait un partage des frais, plutôt que de rester assis sur un node inactif ne recueillant que des récompenses.

### Registre de Service

Streamflow étend le rôle du registre de service dans le protocole on chain. Les orchestrateurs continueront à publier leurs informations de récompense, de réduction, de redevance et de connexion, mais ils publieront également les services proposés par leur node et les régions desservies par leur node. Cela aura des répercussions sur les performances et les radiodiffuseurs pourront rechercher les services spécifiques qu’ils souhaitent, desservis par un node à proximité. Les orchestrateurs ne feront plus la publicité du prix qu'ils facturent, car la négociation des prix et de la disponibilité est en train de sortir de la chaîne. En ce qui concerne les services considérés, il existe probablement deux abstractions:


1. **Un service**
    1. Identificateur de service - l'identifiant qui représente ce service particulier, tel que «CPUTranscoding», “GPUTranscoding“ ou “SegmentVerification“. Il reste encore du travail à faire sur la définition exacte, et il est possible que les services soient plus granulaires, comme l'entrée / paires de codage de sortie telles que “H264 1080p -> 720p“.
    1. Fonction de vérification - le pointeur d'adresse sur la fonction de vérification qui sera exécuté pour invoquer la vérification en chaîne de l'exactitude de ce service (peut être nulle si aucune vérification n'est disponible).
1. **Emplacements**
    1. L'implémentation est à déterminer, mais il s'agit probablement d'une abstraction qui spécifie un tableau des régions que ce node veut ou peut servir.

La combinaison de ces publicités permettra aux radiodiffuseurs de filtrer le registre de services pour les nodes qui annoncent les services et les emplacements qu’ils souhaitent desservir afin d’engager efficacement des négociations hors chaîne avec les fournisseurs de services appropriés. L'emplacement était un facteur précédemment ignoré dans la version alpha de Livepeer. Toutefois, il peut être essentiel pour l'acquisition de vidéo en direct que les nodes recevant la vidéo soient situés à proximité de la source vidéo, en raison de divers problèmes de réseau pouvant se produire et créer une instabilité accrue. connexions plus longues avec plus de sauts.

Bien entendu, un emplacement annoncé peut être falsifié. Cependant, à l'instar de nombreux aspects de Streamflow, les implémentations client vont rapidement détecter et filtrer les orchestrateurs peu performants, ce qui leur coûtera la possibilité d'effectuer des travaux futurs et de gagner des frais. L'exécution honnête de nodes, en maximisant leurs relations de succès de télédiffuseur calculées par le client, leurs frais et leurs statistiques sur la réputation, permettra probablement de diffuser des informations de localisation utiles menant à des emplois durables, négociés, attribués et durables.

Lorsque les nodes gagnent du LPT inflationniste, ils peuvent, pour optimiser leur utilisation, ajouter un nouveau node au registre de services, qui sert une capacité ou un emplacement pour lequel il existe une demande, mais pas assez fiable ou économique l'offre - élargissant ainsi l'empreinte du réseau et la capacité de servir divers clients et cas d'utilisation.

### Négociation des travailles hors chaîne

Le passage d'une tâche en chaîne à une négociation en dehors de la chaîne est peut-être le changement le plus important proposé par Streamflow. Cela change l’hypothèse selon laquelle les emplois sont acheminés strictement en fonction de l’enjeu, et cela sera analysé ci-dessous dans la section Analyse, mais cela apporte également d’énormes avantages. À savoir:

* **Disponibilité** - Les radiodiffuseurs seront en mesure de s'assurer que les orchestrateurs sont disponibles pour effectuer du travail avant de passer un contrat avec eux.
* **Redondance** - Si un orchestrateur n'est pas disponible avant ou pendant le travail, passez simplement à un autre orchestrateur. Ou commencez par travailler avec plusieurs orchestrateurs en premier lieu pour la redondance.
* **Vitesse** - Commencez le travail immédiatement. Il n'est pas nécessaire d'attendre une confirmation sur la chaîne.
* **Rentable** - Il n'y a pas de coûts liés au travail sur la chaîne ou au gaz associés à la demande de service sur le réseau.

Pour mener une négociation, un radiodiffuseur interagira avec le protocole suivant:

1. Lisez le registre de service et analysez tous les orchestrateurs disponibles qui correspondent aux paramètres de service et d'emplacement demandés, avec la participation minimale requise.
1. Ils utiliseront ensuite les informations de connectivité fournies pour envoyer une requête de travail à chacun d'entre eux.
    1. Une demande de travail contient le service demandé et l'emplacement demandé (facultatif).
1. Les orchestrateurs répondent le plus rapidement possible avec un devis pour effectuer le travail, s’ils souhaitent le concurrencer et qu’ils sont disponibles.
    1. Les orchestrateurs incluent également des paramètres de micropaiements probabilistes dans leur offre de prix (décrite ci-dessous).
1. Les radiodiffuseurs collectent ces données de réponse, ainsi que les temps de réponse des orchestrateurs.
1. Ils utilisent leur propre algorithme interne en prenant en compte les préférences en matière de temps de réponse, de prix, d’historique de travail, de paramètres de gestion de projet, de redondance, de sécurité sous forme de mise, afin de choisir le ou les orchestrateurs avec lesquels travailler.
1. Ils commencent à envoyer des segments vidéo et des tickets PM au (x) orchestrateur (s) sélectionné (s).
1. L'orchestrateur vérifie le dépôt de chaîne du télédiffuseur sur la chaîne et, si le niveau de dépôt est suffisant, il renvoie le segment encodé au télédiffuseur.

Il est clair que l'étape 5 de ce protocole laisse beaucoup à la mise en œuvre. En résumé, les radiodiffuseurs peuvent choisir leurs propres orchestrateurs et n’ont pas besoin de suivre la chaîne pour annoncer le travail ou d’en obtenir un.

Ils peuvent travailler avec leur propre orchestrateur s’ils le souhaitent, puis ne commencer à envoyer des segments qu’à un autre candidat lorsqu’ils atteignent leur propre capacité de calcul. Ils peuvent travailler avec le même node avec lequel ils entretiennent une relation de longue date et ne basculer vers un autre lorsque ce node tombe en panne ou devient indisponible. Ils peuvent commencer avec un encodage de processeur redondant 5x dès le début pour un flux en direct premium très important, ou utiliser l'encodage GPU le moins cher possible dans le monde pour un travail à la demande très peu fiable afin de réduire les coûts.

<img src="https://livepeer-dev.s3.amazonaws.com/docs/pricenegotiation.jpg" alt="Offchain Job Negotiation" style="width: 750px">

La commutation et l'ajout de redondance n'introduisent aucun surcoût de coût de transaction en chaîne pour le radiodiffuseur, alors que dans la version alpha du protocole, la commutation nécessite une transaction en chaîne supplémentaire et des temps de confirmation supérieurs à 15-30 secondes.

Notez que les étapes 1 à 4 peuvent éventuellement être effectuées en arrière-plan sur une base continue, plutôt que lors de la création du flux. Si un radiodiffuseur gère de nombreux flux simultanés, il peut s'avérer intéressant de conserver une table de prix / service à jour pour tous les orchestrateurs disponibles, de sorte qu'ils puissent simplement commencer à en utiliser un à tout moment, sur n'importe quel flux.


### Micropaiements Probabilistes
L’effet le plus important sur les économies de coûts générées par Streamflow proviendra de cette proposition de micropaiements probabilistes. Auparavant, le protocole utilisait un flux de transaction deposit () -> job () -> claim () -> verify () -> applyFees () pour libérer des paiements pour le travail effectué. Les trois dernières de ces transactions devaient être effectuées pour 1 000 segments de vidéo en moyenne (ou plus), et effectuer cinq transactions pour un travail court serait totalement prohibitif pour les transcodeurs.

Pour des informations générales sur les micropaiements probabilistes, il est suggéré de passer en revue une publication de l’équipe du protocole d’orchidée sur son utilisation dans un réseau VPN décentralisé, ainsi que la recherche universitaire précédente[[3, 4, 5](#Références)]. En résumé, le diffuseur émet des tickets signés avec chaque segment de travail destiné à l'orchestrateur. Le ticket a une valeur faciale élevée s'il “gagne“, ce qui permet à l'orchestrateur de l'encaisser dans la chaîne pour ce montant élevé. Cependant, la probabilité de gain étant très faible, la valeur attendue de chaque ticket correspond au prix / segment sur lequel le radiodiffuseur et l'orchestrateur sont convenus. À long terme, les radiodiffuseurs paieront presque exactement ce qu'ils conviennent par segment aux orchestrateurs, et les orchestrateurs recevront le montant exact pour le travail qu'ils ont effectué, en raison des probabilités au travail.

En utilisant les micropaiements probabilistes, le coût de la collecte des paiements peut correspondre au coût d’une transaction simple et légère, et le montant du paiement collecté peut effectivement être groupé avec le montant que l’orchestrateur est disposé à encaisser. Par exemple, l'orchestrateur peut toujours encaisser des paiements d'un montant de 10 dollars en ETH, tandis que le coût de l'encaissement du ticket en raison du prix de l'essence peut être de 0,10 dollar, soit 1% de frais généraux. Si le prix de l'essence augmente de 10 fois, l'orchestrateur peut au lieu de cela encaisser des paiements de 100 dollar tout en maintenant les mêmes frais généraux de 1%, ou absorber davantage de frais généraux s'il le souhaite pour rester compétitif. Conformément à la philosophie qui sous-tend de nombreuses mises à jour de la proposition Streamflow, celle-ci sera axée sur le marché et configurable par le client plutôt que par le protocole.

La capacité de paiement d’un télédiffuseur lorsqu’un orchestrateur gagne de l’argent est garantie par un dépôt en chaîne, un dépôt bloqué et des pénalités.

En raison de la négociation de travail hors chaîne et des redondances potentielles dont un radiodiffuseur peut avoir besoin, il peut envoyer des tickets micropaiements probabilistes à plusieurs orchestrateurs à la fois, démarrer et arrêter de travailler avec un orchestrateur à tout moment, et de la même manière, orchestrateur peut cesser de travailler à tout moment pour un radiodiffuseur s’ils déterminent qu’ils ne paient pas correctement ou souhaitent passer hors ligne. Cela modifie énormément le modèle mental d'un «travail» dans Livepeer en tant que flux continu complet, à un travail en tant que segment vidéo unique associé à un seul ticket micropaiements probabilistes.

Le processus complet de gestion de la performance est laissé pour une [Annexe](#appendix), car il aborde la vérification, la négociation hors chaîne et de nombreux autres domaines tels que le risque de double dépense et les mesures d'atténuation.

### Vérification de la Chaîne Basée sur la Faute
Le dernier changement majeur proposé par Streamflow consiste à ajuster le protocole de vérification afin de réduire les coûts et d’éviter les problèmes de disponibilité des données. Auparavant, les transcodeurs devaient invoquer la vérification Truebit pour un segment `VerificationRate` sur 1, défini à l'origine pour un segment sur 1000. Cela coûte très cher et est nécessaire que le transcodeur ait fonctionné correctement ou incorrectement. La nouvelle proposition est la suivante:

* Les radiodiffuseurs sont responsables de vérifier les segments transcodés reçus et de les contester à la chaîne Truebit uniquement s'ils estiment que la vérification du segment a échoué.
* Si Truebit (ou une autre fonction appropriée de vérification de la chaîne) convient, la part de l’orchestrateur est réduite et le diffuseur reçoit une prime importante.

<img src="https://livepeer-dev.s3.amazonaws.com/docs/faultverification.jpg" alt="Fault Based Verificaiton">

Une partie de l’argument contre cette méthode est que le diffuseur ne dispose pas de ressources de calcul importantes pour réencoder la vidéo afin de vérifier si le travail a été effectué correctement ou non. En utilisant la même approche aléatoire que le protocole d'origine, le télédiffuseur peut vérifier 1 segment de VerificationRate s'il le souhaite. Il pourrait vérifier davantage si cela nécessite davantage de fiabilité ou externaliser la vérification vers un autre node du réseau et payer ce node pour effectuer une vérification efficace en son nom, ce qui revient à embaucher un deuxième orchestrateur pour un seul des segments `VerificationRate`. Ils pourraient utiliser un orchestrateur bon marché pour le travail principal, mais compter sur l'orchestrateur, qui jouit d'une grande réputation et de coûts élevés, en tant que vérificateur plus fiable. Il existe également des vérifications beaucoup moins coûteuses qui peuvent être effectuées en analysant les images de la vidéo en sortie plutôt que par un réencodage complet, tel qu'une vérification basée sur des métriques. Ces vérifications peu coûteuses peuvent être utilisées pour vérifier s’il existe une faute probable et, dans ce cas uniquement, il est ensuite procédé à un nouveau codage avant de lancer le défi à Truebit.

Cependant, l’essentiel est que l’orchestrateur ne sache pas quels segments seront contestés, et si un segment échouait à la vérification, il risquait de perdre énormément d’enjeu. Les avantages de la triche devraient dépasser la valeur d'un dépôt à participation fixe entièrement réduit, ce qui est peu probable pour de nombreux cas d'utilisation. De plus, dans la mesure où les radiodiffuseurs peuvent utiliser des redondances, s’ils détectent une incohérence ou s’ils soupçonnent de tricher à partir d’une opération bon marché sans réencodage, elle pourrait simplement choisir de travailler avec un autre orchestrateur sur ce segment afin d’obtenir un codage approprié et en conséquence. insérez-le dans sa playlist.

L’un des effets de cela est que le coût de Truebit n’est pas nécessaire, sauf en cas de fraude évidente - et donc presque jamais, puisqu’il ne devrait jamais en valoir la peine pour un orchestrateur de tricher intentionnellement. Cela rend le réseau beaucoup moins cher à utiliser que le coût d'invocation de Truebit sur chaque segment vidéo de `verificationRate`.


## Analyse Economique #################################

Les modifications proposées par Streamflow entraînent des incitations et des comportements légèrement différents pour les orchestrateurs et les délégués, ce qui se traduira par un réseau plus évolutif, fiable et rentable. Cette section commence par une analyse d’impact économique de ces modifications proposées, y compris un examen du rôle du jeton de compte rendu, du rôle de la délégation, des effets de l’inflation sur le réseau et de certaines considérations économiques hors chaîne.

### Livepeer Token

Le token Livepeer (LPT) peut toujours être décrit comme un token de travail. Ceux qui l'ont implanté ont eu la possibilité d'effectuer un travail sur le réseau et donc de gagner les futurs frais (en ETH) pour ce travail. Les travaux ont été acheminés en proportion directe de la mise, si les prix proposés par tous les nodes étaient constants. Dès le début, il y avait des mécanismes conçus pour une “exigence de travail“, en ce sens que si un node n'effectuait pas suffisamment de travail dans une certaine proportion de son enjeu, il pourrait alors être réduit. Il s'agissait d'une tentative visant à garantir que les nodes contribueraient réellement à la valeur (ou imposeraient des frais généraux pour ne pas le faire ou le simuler), plutôt que de simplement rester les bras croisés et générer de l'inflation. De plus, rien n'obligeait les travaux à être effectués de manière rentable ou performante. La concurrence pourrait être encouragée par la société, mais pas imposée au niveau du protocole.

Les mises à jour du protocole pour assouplir le nombre d'orchestrateurs contraint artificiellement, ainsi que la négociation de travail hors chaîne, semblent modifier cette connexion directe entre le token et le droit de travailler à la surface, mais après analyse, la même valeur est appliquée à l'état d'équilibre. Examinons la fonction qu’un détenteur de token tente de maximiser:

`Valeur accumulée en un seul tour = LPT gagné par l'inflation + frais gagnés.`

Le TPL inflationniste est prévisible, basé sur la récompense d'un orchestrateur. Un mandant peut choisir exactement combien de LPT il souhaite gagner en échange du travail d’assurance qualité qu’il accomplit.

Les frais gagnés en revanche sont moins sous le contrôle du détenteur du token. C'est parce que cela dépend de:

1. Combien de travail leur orchestrateur effectue
2. Les frais de l'orchestrateur
3. Quelle part totale est déléguée à l'orchestrateur et, par conséquent, à quel pourcentage des frais ils ont droit.

À la fin d'un tour, un mandant sera en mesure de calculer la capacité de gain de son LPT implanté. C’est ce ratio de frais:

`ETH en frais / unité de LPT misé`

Quelle sera la statistique visible que les délégués pourront utiliser pour comparer les orchestrateurs, et comme il fallait s'y attendre, la délégation devrait passer d'un tour à l'autre en nodes où il serait possible d'optimiser ce rapport. En bref, pourquoi rester avec un node qui partage un enjeu 1gwei / LPT alors qu’un autre node vers lequel vous pouvez basculer est celui qui partage 2 enjeu gwei / LPT?

<img src="https://livepeer-dev.s3.amazonaws.com/docs/feeratio.jpg" alt="Fee Ratio">

Cependant, il convient de noter que l'acte consistant à transférer davantage de participations sur ce node opportuniste signifie que les frais seront partagés entre davantage de participations et que le ratio de frais diminuera. L'état d'équilibre est que les nodes qui effectuent plus de travail (gagnent plus) ont plus d'enjeu, et que les nodes effectuant moins de travail avec le même partage de redevance ont moins d'enjeu. Tous les nodes concurrentiels doivent en définitive se retrouver avec les mêmes ratios de frais d’équilibre, avec une répartition des pouvoirs intelligemment déléguée à un délégant - et donc un LPT implicitement appliqué intelligemment donne accès à des travaux permettant de gagner des frais sur le réseau, quel que soit le mode d’emploi attribué.

### Délégation en tant que Signal de Sécurité et de Réputation

Un résultat négatif que les gens pourraient prévoir est que les nodes qui gagnent beaucoup de travail pourraient fournir 0% de redevance, et donc ne pas attirer de délégation. Ce n’est pas un problème; ils utilisent du matériel, encourent des coûts et fournissent un excellent service au réseau; ils n’auront peut-être pas besoin de délégation. Mais la délégation, d’autre part, offre une sécurité supplémentaire - c’est un enjeu supplémentaire qui peut être réduit si le node triche - un signal de réputation accru. Les clients utilisent ce signal pour sélectionner les nodes avec lesquels travailler. Ainsi, un node concurrentiel annonçant une part de frais> 0% serait plus susceptible d'attirer une participation, et donc un travail % node de partage des frais. Encore une fois, cela contribue aux configurations flexibles et aux cas d'utilisation du réseau. Cela augmente les possibilités de concurrence, de décentralisation, de diversité et de résilience du réseau.

Alors que les nouveaux nodes cherchent à concurrencer pour travailler sur le réseau, ils peuvent avoir besoin d'attirer suffisamment de participation pour offrir la sécurité requise par les diffuseurs. Dans ces cas, il est probable que ces nodes définiraient un partage des frais plus important. Les délégants actifs auront la possibilité de rechercher et de s’intéresser aux nodes qui gagnent des portions démesurées de travaux, avec des parts de redevance plus importantes, ce qui entraîne des ratios de redevance plus élevés. En bref, la participation déléguée peut assurer la sécurité et acheminer le travail en échange de frais partagés lorsque le travail est bien exécuté. Une délégation active peut conduire à donner aux nodes plus opportunistes la possibilité d'étendre de manière compétitive l'empreinte et les capacités du réseau.

### Inflation dans L'état de Servitude et Délégués Apathiques
L'une des critiques du modèle de jeu d'enjeu sans plafond, sans enjeu minimum, est qu'il permet un comportement paresseux de la part des délégants. Le LPT inflationniste continue de s'accumuler dans l'état lié, continue de s'aggraver et permet au délégant de l'installer et de l'oublier tout en collectant le LPT sans ajouter de valeur significative au réseau.

Cela peut être le cas aux tout premiers jours du réseau, avant que les frais ne constituent une incitation supplémentaire pour les délégués, mais il est peu probable que les orchestrateurs soient en concurrence pour travailler, gagner de l'argent et distribuer des frais. À ce stade, le comportement du pilote automatique peut toujours générer une augmentation du LPT, mais renoncerait aux frais potentiels pouvant être générés par le passage à des orchestrateurs générant un rapport ETH / LPT plus élevé.

Alors que le taux d'inflation est susceptible de diminuer avec une utilisation réduite, lorsque les détenteurs de jetons cherchent à se faire concurrence pour gagner les frais, la part de la fonction de récompense qui est imputable à l'inflation du LPT continue également à diminuer, une grande partie provenant des frais. Et comme indiqué ci-dessus dans la section LPT, la nécessité de contrôler en permanence le réseau et d'acheminer le travail vers les nodes qui surpassent les autres nodes est motivée par les rendements opportunistes. En bref, un délégant apathique est moins récompensé qu'un délégant actif.

En outre, les orchestrateurs qui ont déjà eu besoin d'attirer des délégations extérieures pour atteindre la mise minimale, accumulent suffisamment de participations pour sécuriser leur propre node, peuvent donc réduire leur part des frais. À ce stade, un délégant optimisateur serait mieux servi en recherchant un nouveau nœud en devenir - essentiellement un node pouvant étendre l'empreinte du réseau - qui pourrait offrir une quote-part de redevance plus élevée afin d'attirer une participation. C’est cette assurance qualité constante effectuée par le mandant qui optimise ses efforts, ainsi que le compromis entre la rémunération et les frais qui créeront une concurrence constante et favoriseront la décentralisation du réseau.

### Considérations D'ingénierie Hors Chaîne
Comme mentionné précédemment, l'une des principales philosophies de Streamflow consiste à déplacer de nombreux avis sur les valeurs de paramètre valides et les interactions p2p hors du protocole principal et dans les implémentations et les configurations client. La mise en œuvre et la configuration multiples de ces paramètres conduiront à un réseau robuste, résistant aux attaques et aux acteurs malveillants. Cependant, étant donné que le protocole lui-même a moins d'opinion, beaucoup reste à mettre en œuvre par le client. Voici quelques-unes des principales considérations à prendre en compte du point de vue de l'ingénierie pour que Streamflow fonctionne vraiment efficacement:

* Stratégies de gestion des risques liés à la gestion de projet - lorsqu'un orchestrateur doit travailler avec ou non avec un diffuseur en fonction de sa réputation et de son historique, et inversement.
* Génération sécurisée de nombres aléatoires pour le protocole PM.
* Résistance DDoS pour les orchestrateurs.
* Algorithmes de redondance et de basculement pour les diffuseurs dans différents scénarios et cas d'utilisation.
* Stratégie de découverte des prix pour les radiodiffuseurs.
* Protocoles de diffusion en continu à faible latence et vérification de la signature / du paiement lorsque le segment final n'est pas disponible avant le début des travaux.

Chacune de ces solutions peut affecter l'efficacité du réseau du point de vue d'un radiodiffuseur - et donc les redondances nécessaires, voire les coûts. La bonne nouvelle est qu’une grande partie de ce qui précède peut être gérée via des stratégies hors chaîne et qu’elle peut être constamment expérimentée avec différentes implémentations ou configurations en concurrence. Un réseau dont les agents agissent de manières différentes et imprévisibles est plus difficile à optimiser pour un attaquant qui, autrement, se tournerait vers une seule implémentation.


## Les Attaques ###############################
Certains sous-protocoles spécifiques, tels que les vérifications basées sur PM et Truebit, sont sujets à leurs propres attaques, que nous laissons pour analyse dans ces domaines de recherche. Voici une brève discussion sur les attaques potentielles et les contre-mesures dans le contexte économique des modifications apportées au protocole par Streamflow.

### Presser les Délégués

Lorsqu'un orchestrateur candidat souhaite exploiter un node et exprimer sa candidature, il peut être nécessaire d'attirer une délégation afin d'atteindre le montant de dépôt minimum spécifié par le client pour attirer les travaux classiques. Pour ce faire, ils peuvent représenter un RewardCut et un FeeShare attrayants. Cependant, au moment où leur node commence à effectuer des tâches et à gagner du LPT inflationniste, ils peuvent souhaiter utiliser ce LPT pour investir davantage dans leur node unique afin de réduire le montant de l'inflation et les frais qu'ils doivent partager avec leurs mandataires. Pour ce faire, ils peuvent chasser les délégants actuels en manipulant leur `RewardCut` et `FeeShare` à un point peu attrayant, puis en comblant le vide avec leur propre participation.

Ceci est théoriquement acceptable, car les délégués peuvent passer à des nodes plus attrayants et s’ajuster au mieux de leurs intérêts. Malheureusement, cela crée une UX agaçante, dans la mesure où les mandataires doivent être constamment vigilants et actifs pour agir dans leur meilleur intérêt. À chaque tour, les actions peuvent passer de dessous.

Selon une idée reçue, les orchestrateurs qui souhaitent créer des nodes supplémentaires, conserver une réputation positive pour attirer une délégation importante et se faire concurrence pour obtenir des frais, se verront lésés par cette approche et n'attireront pas de délégation ultérieure.

### Vol des Frais de Délégation

Comme indiqué dans la section intitulée Presser les Délégués du mandant ci-dessus, il est possible que l'orchestrateur chasse ses délégués. Cela pourrait devenir une technique particulièrement malveillante si l'Orchestrateur conservait également ses tickets PM gagnants jusqu'au moment du départ des délégants, puis les encaissait lorsqu'il contenait tous les enjeux de son node. Essentiellement, les frais et récompenses auxquels les délégués ont droit seraient remis à l'orchestrateur.

Cela peut être contrebalancé par la date d'expiration indiquée sur les billets PM, qui est antérieure à la date de retrait des dépôts bloqués dans le temps des diffuseurs. En tant que tels, les tickets devront être encaissés rapidement et contiendront potentiellement la part d'honoraires engagée de l'orchestrateur à ce moment-là, de telle sorte que, lors de l'encaissement d'un ticket gagnant, les fractionnements appropriés puissent être effectués entre les mandataires et l'orchestrateur.


## Zones de Recherche Ouvertes ############################

Comme pour tous les travaux effectués dans le domaine des protocoles crypto-économiques basés sur les chaînes de blocs, de nombreux problèmes de recherche doivent encore être résolus avant que les systèmes puissent atteindre une décentralisation complète, une absence de confiance et une rentabilité. Le projet mène activement des recherches dans quelques domaines. La participation de la communauté est également la bienvenue pour faire progresser ces domaines.

### Vérification Non Déterministe
Les recherches se poursuivent pour vérifier la probabilité qu’un segment codé par un processeur graphique représente le même contenu que le segment pré-codé. Cette approche probabiliste et fondée sur des métriques a été démontrée lors d'expériences et de recherches antérieures pour produire des scores précis. Toutefois, la possibilité de réduire réellement les dépôts en fonction de résultats probabilistes est certainement discutable et nécessite des recherches supplémentaires.

En revanche, le codage déterministe peut continuer à être vérifié par divers systèmes de vérification, notamment Truebit, la vérification du matériel basée sur SGX, Oracles ou même des vérificateurs de confiance.

### Protocoles de Pool de Transcodeurs

La scission des responsabilités d'orchestration et de transcodeur devrait permettre de redimensionner considérablement les opérations des nodes sur Livepeer, en exploitant le matériel inactif pour transcoder la vidéo, sans obliger nécessairement toutes les machines à être reconnues par Livepeer. On pense que les pools privés, dans lesquels l'orchestrateur contient également ce matériel de transcodage, seront les plus rentables, car l'orchestrateur peut avoir la certitude que le résultat généré par les transcodeurs est correct et non malveillant.

Les pools publics, dans lesquels l'orchestrateur ne fait pas confiance aux transcodeurs, mais permettent à n'importe qui de participer à la course pour transcoder des segments, pourraient s'avérer très puissants pour tirer parti des calculs inactifs, sans avoir à disposer d'une infrastructure dédiée. Cependant, étant donné que ces nodes de transcodage distants ne sont pas fiables, l'orchestrateur devrait vérifier leur travail, sinon il risquerait d'être réduit. Cela entraîne des coûts supplémentaires et par conséquent, il est peu probable que la concurrence avec les gisements privés soit mise en concurrence - à moins que des protocoles économiques ne puissent être créés pour sécuriser ces gisements publics sous la forme de gisements. S'il peut être démontré qu'un transcodeur inconnu particulier est le résultat d'une condition de slashing invoquée et qu'il dispose d'un dépôt / enjeu suffisant pour couvrir le coût du slash, alors les pools publics peuvent être viables.

La recherche et la conception sont ici un sujet ouvert.

### Diffuseur Pour la Mitigation des “Doublespend”

Dans un système de micropaiements probabilistes, il est toujours possible qu'un radiodiffuseur ait émis plus de tickets gagnants qu'il ne lui en reste pour payer (accidentellement). Et comme les orchestrateurs ne peuvent pas informer immédiatement le télédiffuseur du billet gagnant, il est difficile d'obtenir une comptabilité exacte du solde du télédiffuseur. Nous poursuivons nos recherches sur les paramètres requis et la gestion des dépôts afin d'éviter une double dépense accidentelle dans le cadre de divers modèles d'utilisation du réseau. Voir l’analyse plus détaillée dans l’annexe sur les micropaiements probabilistes.

### Paiements Vidéo à la Demande

Une des propriétés intéressantes que les radiodiffuseurs peuvent rechercher en matière de transcodage vidéo à la demande est la possibilité de rendre le contenu disponible, de demander un travail et de le faire disparaître - de sorte que l’orchestrateur puisse effectuer le travail de manière asynchrone, le distribuer sur plusieurs nœuds ou planifiez-le lorsque des ressources inactives sont disponibles.

Toutefois, dans le schéma de MP décrit dans Streamflow, le radiodiffuseur doit être en ligne pour pouvoir envoyer en continu des paiements sous forme de flux de contenu. Une partie de la sécurité réside dans la reconnaissance du fait que si un orchestrateur ne continue pas à faire le travail, c'est bon, car le télédiffuseur va tout simplement arrêter d'envoyer des paiements futurs.

Toutefois, pour les travaux de vidéo à la demande (VOD), si un télédiffuseur paie à l’avance tous les segments de la vidéo puis disparaît hors connexion, aucune sécurité ne garantit que l’orchestrateur effectuera le transcodage ou rendra les segments transcodés disponibles au diffuseur. Pour le moment, le transcodage VOD est possible, mais le téléchargement-disparition ne l'est pas. La recherche se poursuivra sur de meilleurs mécanismes permettant les paiements en VOD.

## Voie de Migration ############################

La proposition Streamflow en est au tout début de son cycle de recherche, de conception, de retour d’information et de mise en œuvre. Il mérite certainement une critique, des tests, des audits et une acceptation approfondis de la part de la communauté avant d’être mis en ligne sur le réseau principal Ethereum comme prochaine itération sur le protocole alpha de Livepeer. Cette section vise à énumérer quelques considérations préliminaires sur la manière dont une migration de protocole pourrait avoir lieu:

* Une nouvelle logique de contrat intelligent serait déployée dans Ethereum, mais il est prévu que très peu, voire aucune migration de données ne sera nécessaire. Le mécanisme de mise à jour existant du proxy-delegatecall de Livepeer pourrait être utilisé.
* L'état existant dans le protocole Livepeer, y compris les soldes de mise, les frais, les récompenses, la délégation, etc., serait maintenu.
* Les transcodeurs, les diffuseurs, les orchestrateurs et les mandataires mettraient à jour leur logiciel client, qui contiendrait une logique pour la négociation des tâches, la redondance, les paiements et la vérification mise à jour.
* Les orchestrateurs enregistrent tous les nouveaux paramètres requis dans le registre de services, y compris les services pris en charge et éventuellement les emplacements.
* Les radiodiffuseurs établiraient des contrats et des dépôts de particules. Les dépôts existants peuvent migrer du contrat * Minter vers le contrat PM en passant par une action des utilisateurs, à la demande.
* Il est prévu que cela pourrait être accompli avec peu de temps d'arrêt du protocole.
* Les clients tiers, tels que les explorateurs de protocole et les outils d'analyse, auraient probablement besoin de mises à jour pour refléter les nouvelles interactions de protocole.

Un chemin de migration formel, une liste de contrôle et plusieurs exécutions de testnet observées seront disponibles au fil du temps, à l'approche de la date de publication de Streamflow.

## Résumé #################################

En conclusion, les propositions contenues dans ce document visent à mettre en lumière un chemin évolutif pour le réseau d'infrastructure vidéo de Livepeer - un réseau qui dissocie le coût d'utilisation de la blockchain Ethereum du coût d'utilisation du réseau lui-même et qui fournit aux développeurs de vidéos redimensionnés existants avec la fiabilité et la performance dont ils ont besoin de leur infrastructure.

Tous les commentaires, idées et contributions sont les bienvenues. N'hésitez donc pas à vous rendre sur le [forum Livepeer](https://forum.livepeer.org) ou sur [Discord Chat](https://discord.gg/RR4kFAh) pour participer.


## Annexe ################################

### Annexe A: Flux de travail des micro-paiements probabilistes

* Un dépôt de garantie d'orchestrateur est leur enjeu. Cela peut être réduit s'ils trichent et échouent à une vérification.
* Un diffuseur (utilisant ce terme pour un utilisateur général, qui peut être davantage un développeur que un diffuseur) dépose un dépôt bloqué pour couvrir le travail futur pour lequel il paiera sur le réseau.
* Un diffuseur veut une vidéo transcodée. Ils examinent le registre en ligne des orchestrateurs faisant la publicité de leurs services et négocient hors chaîne avec ceux qui répondent à leurs besoins:
    * Les orchestrateurs leur fournissent un devis.
    * Les orchestrateurs fournissent des paramètres de micropaiement probabilistes (MP), qui peuvent varier en fonction des conditions du réseau Ethereum. Par exemple, ils peuvent définir le montant du ticket gagnant de telle sorte que le coût de l’encaissement soit inférieur à 1% de la valeur reçue.
* Le diffuseur envoie des segments de vidéo aux orchestrateurs avec lesquels il souhaite travailler, ainsi qu’un ticket PM.
    * Le ticket PM est un protocole interactif destiné à empêcher toute partie de biaiser la source du hasard utilisée pour déterminer si un ticket est gagnant ou non. Après chaque ticket gagnant, l'orchestrateur doit générer un nouveau # aléatoire et envoyer l'engagement au diffuseur. Ceci est probablement acceptable, car le diffuseur et l'orchestrateur enverront déjà des données dans les deux sens - cela impliquerait simplement un message supplémentaire envoyé par l'orchestrateur à chaque fois qu'un nouvel engagement est requis. Une optimisation ultérieure éventuellement possible consiste à utiliser une fonction aléatoire vérifiable (VRF) implémentée dans un contrat intelligent: l'orchestrateur attribue une clé de publication au diffuseur et l'orchestrateur signe les tickets reçus avec la clé privée correspondante; le contrat de VRF vérifie que le sig est correct, qui est ensuite utilisé comme source de pseudo-hasard. En conséquence, le protocole devient non interactif. Aurait besoin d'évaluer la faisabilité de la mise en œuvre du VRF dans un contrat intelligent et le coût de la vérification de ce type de signature - ok, ne vous concentrez pas sur cela maintenant, mais sur une possibilité pour l'avenir.
* Le diffuseur reçoit une sortie transcodée de l'orchestrateur.
    * Le radiodiffuseur peut vérifier tous les segments qu’il souhaite vérifier.
    * Si le travail ne se vérifie pas, ils peuvent fournir cette preuve à Truebit sur chaîne pour réduire l’orchestrateur et gagner une énorme récompense.
* Si le diffuseur ne reçoit pas de travail de la part de l'orchestrateur, arrêtez simplement de leur envoyer de futurs segments et travaillez avec un autre orchestrateur.
* Si l’orchestrateur ne reçoit pas de ticket PM valide, ne faites pas le travail et ne renvoyez aucune sortie.
    * Le protocole contiendra des messages pour certaines conditions d'erreur, telles que `LowPMBalance` ou `SegmentFormatDidntMatchJobInputParams` afin que les radiodiffuseurs puissent recevoir des informations utiles à déboguer ou que leur node puisse prendre des décisions telles que le remplissage de leur solde.
* Orchestrator surveille le dépôt du radiodiffuseur et évalue le risque de défaillance.
    * Algorithme simple pour commencer. Si leur solde est trop bas, arrêtez simplement de travailler.
    * Orchestrateur encaisse les tickets gagnants au fur et à mesure de leur réception (ou attend que le prix de l'essence soit moins cher, en évaluant le risque de défaillance)

Pour une analyse complète et la spécification des structures de données du ticket, la prévention de la double dépense et d'autres considérations relatives à la conception, voir ce [document externe](https://hackmd.io/uHMFeNSyS_GyzwnO3Ld74A?view).

## Références ###########################################

1. Livepeer Whitepaper - Doug Petkanics, Eric Tang - <https://github.com/livepeer/wiki/blob/master/WHITEPAPER.md>
2. The Video Miner, A Path To Scaling Video Transcoding -Philipp Angele - <https://medium.com/livepeer-blog/the-video-miner-a-path-to-scaling-video-transcoding-a3487d232a1>
3. Ethereum Probabilistic Micropayments - Gustav Simonson - <https://medium.com/@gustav.simonsson/ethereum-probabilistic-micropayments-ae6e6cd85a06>
4. Electronic Lottery Tickets as Micropayments - Ron Rivest - MIT Lab for Computer Science - <https://people.csail.mit.edu/rivest/pubs/Riv97b.pdf>
5. Decentralized Anonymous Micropayments - A. Chiesa, M. Green, J. Liu, P. Miao, I. Miers and P. Mishra - <https://eprint.iacr.org/2016/1033.pdf>

