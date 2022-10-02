# TP1 · Conception Orientée-Objet
## Mael KERICHARD - Romain BRIEND

L’objectif de ce TD/TP est de préparer un dossier complet d’analyse et de conception par Objet à partir d’un cahier des charges.
Ce cahier des charges concerne un système de réservation d'un cinéma.

## Diagramme de cas d'utilisation

Nous avons identifié 8 cas d'utilisation de notre système. Nous avons tout d'abord les cas d'utilisation d'un employé du cinéma. Celui-ci peut changer la grille tarifaire, vérifier la validité d'un ticket de cinéma ainsi que de gérer la répartition des séances dans les salles. D'un autre coté, nous avons le client qui peut réserver un ticket de cinéma et créer un compte de fidélité. Si il possède un compte de fidélité, il peut alors également souscrire à une carte comportant 10 tickets à prix réduits ou alors à un abonnement cinéma illimité. Si il possède un abonnement, il peut alors se désinscrire.

```plantuml
@startuml
actor "Client" as c

actor "Membre du personnel" as p

rectangle "Système de réservation du cinéma" {
  usecase "Réservation d'une séance de cinéma" as UC1
  usecase "Gérer la répartition des séances dans les différentes salles" as UC2
  usecase "Créer un compte de fidelité" as UC3
  usecase "Souscrire à un abonnement Cinépass" as UC4
  usecase "Réserver 10 séances à un tarif avantageux" as UC5
  usecase "Changer la grille tarifaire" as UC6
  usecase "Verification de la validité des billets" as UC7
  usecase "Désabonnement du Cinépass" as UC8
}
c --> UC1
p --> UC2
c --> UC3
c --> UC4
c --> UC5
p --> UC6
p --> UC7
c --> UC8

UC3 -.-> UC4
UC3 -.-> UC5
UC4 -.-> UC8
@enduml
```

## Diagramme de classe

Notre diagramme de classe est orienté autour d'une classe centrale "Cinema". Celle-ci contient la liste des clients, les salles et séances, ainsi que les tickets réservés. Nous avons deux autres classes qui nous servent à manipuler notre système à partir de requete des clients ou des employés. D'une part, il y a "EmployeeMonitor" qui possède toutes les méthodes utiles aux employés pour modifier les tarifs, gérer les séances et pour vérifier la validité d'un ticket. D'autre part,
nous avons le Server qui reçoit les requetes des clients et fait les vérifications et enregistrement des informations auprès du système.

```mermaid
classDiagram

    Card --|> CineCard
    Card --|> CinePass

    class Movie{
        +String title
        +String director
        +String[] actors
        +int duration
        +String target 
    }

    class Room{
        +int size
        +RoomType type
    }
    
    Screening "*" o-- "1" Room
    Screening "*" o-- "1" Movie

    class Screening{
        +Movie movie
        +DateTime datetime
        +Room  room
        +int amountReserved
    }
    
    Room "*" o-- "1" RoomType
    
    class RoomType{
        <<enumeration>>
        DOLBY
        3D
        STANDARD
    }
    
    class Client{
        +long id
        +ClientInfo info
        +int fidelityPoints
        +Card[] cards
    }
    
    Client "1" *-- "1" ClientInfo
    
    class ClientInfo{
        +String firstname
        +String lastname
        +String email
        +Date birthday
        +String address
        +Hash password
    }
    
    Client "1" -- "0..2" Card

    class Card{
        +String id
    }

    class CineCard{
        +int amountLeft
    }

    class CinePass{
        +String iban
        +Date expirationDate
    }
    
    class Server{
        +Cinema cinema
        +boolean createAccount(ClientInfo info)
        -boolean checkAccountExist(String email)
        +boolean login(String email, String password)
        +boolean buy10TicketsCard(PaymentInfo paymentinfo, String tokenClient)
        +Screening[] getScreenings(DateTime date, Movie movie)
        +Movie[] getMovies()
        +boolean bookTickets(String email, Screening screening, Booking[] bookings, paymentInfo paymentInfo)
        +HashMap~ReductionType, int~ getPrices(Screening screening)
        +boolean unsubrscribeCard(String idCard)
        +Pair~ClientInfo, Card[]~ getProfileInfo(long clientid)
    }
    
    Server "1" o-- "1" Cinema

    class Cinema{
        +HashMap~RoomType, int~ prices
        +HashMap~ReductionType, int~ reductions
        +Room[] rooms
        +Screening[] screenings
        +Card[] cards
        +Client[] clients
        +Ticket[] tickets
        +void addClient(Client client)
        +Card getCard(Client client)
        +Card getCard(String id)
        +void addCard(Card card)
    }
    
    Cinema "1" *-- "*" Room
    Cinema "1" *-- "*" Screening
    Cinema "1" *-- "*" Card
    Cinema "1" *-- "*" Client
    Cinema "1" *-- "*" Ticket
    
    class EmployeeMonitor{
        +Cinema cinema
        +boolean addScreening(Movie movie, DateTime datetime, Room  room)
        +boolean removeScreening(DateTime datetime, Room  room)
        +void changePrice(RoomType roomType, int price)
        +Ticket getTicket(long ticketId)
        +void changeReduction(ReductionType reductionType, int percentage)
    }
    
    EmployeeMonitor "1" o-- "1" Cinema

    class Ticket{
        +long id
        +ReductionType reductionType
        +Screening screening
    }
    
    Ticket "*" -- "1" Screening
    Ticket "*" -- "1" ReductionType

    class Booking{
        <<abstract>>
    }

    class StandardBooking{
        +ReductionType reductionType
    }
    
    StandardBooking "*" -- "1" ReductionType
    
    class CardBooking{
        +String id
    }
    
    Booking --|> StandardBooking
    Booking --|> CardBooking
    
    class PaymentInfo{
        +String creditCard
        +Date expirationdate
        +int cryptonumber
    }

    class ReductionType{
        <<enumeration>>
        CHILD
        STUDENT
        RETIRED
        STANDARD
        UNEMPLOYED
        BIRTHDAY
        FIDELITYPOINTS
    }
```

## Diagramme d'Etats

Nous avons donc un diagramme d'états centré autour des clients avec deux états : l'état connecté à un compte, et l'état sans compte. Chaque état permet au client de faire différentes actions, ou de mêmes actions réalisables différemments (réserver un ticket).

```plantuml
@startuml
[*] --> C2
state "Client avec Compte" as C1
state "Client sans Compte" as C2
C1 --> C2 : Deconnexion
C2 --> C1 : [Client->hasAccount] Connexion
C2 --> C2 : [Client->!hasAccount] Créer un compte \n - Réserver une place cinéma (cartes, paiements)
C1 --> C1 : Réserver une place cinéma (cartes, paiements, points de fidélité ou anniversaire) \n - [Client->!hasCinepass] Prendre un abonnement cinépass \n - Acheter une carte 10 tickets \n - [Client->hasCinepass] Se désabonner d'un cinépass
@enduml
```

## Analyse Fonctionnelle et Comportementale pour chaque cas d'utilisation

### Création d'un compte client

L'utilisateur peut faire le choix de créer un compte client pour le cinéma. Pour cela, il doit se rendre sur le site du cinéma.
Sur celui-ci, il pourra trouver un bouton pour créer un compte qui l'amenera vers un formulaire de création de compte. Il pourra alors remplir les données utilisées pour la création d'un compte (nom, prénom, email, date de naissance, adresse et son mot de passe).
Une fois renseignées, ces informations sont envoyés au système et execute la fonction createAccount qui commencera par vérifier que la personne ne possède pas déja un compte. Si elle possède deja un compte, elle lui renverra un message d'erreur. Sinon elle stockera les informations du client comme compte client dans le système.
Le client recevra alors un message de succès.

```mermaid
sequenceDiagram

actor Client
participant Site internet

Client->>Site internet: Ouverture du site
Site internet->>Client: Affichage de la page d'accueil
Client->>Site internet: Clic sur "Créer un compte"
Site internet->>Client: Affichage de la page de création de compte contenant un formulaire avec les champs suivants : nom, prénom, adresse, date de naissance, email, mot de passe, confirmation du mot de passe
Client->>Site internet: Remplissage du formulaire
critical Validité des données
option Email déjà utilisée
Site internet->>Client: Affichage d'un message d'erreur
Client->>Site internet: Modification des données
end

Site internet->>Client: Affichage de la page de confirmation de création de compte

Site internet->>Site internet: Envoi d'un email de confirmation à l'adresse email fournie par le client avec le numéro de fidélité
```

#### Diagramme de Séquence :

```plantuml
@startuml
alt account already exists
ClientPerson -> Server ++ : createAccount(clientInfo)
Server -> Server ++ : checkAccountExist(email)
return email already exist
return error email already exist
else email ok
ClientPerson -> Server ++ : createAccount(clientInfo)
Server -> Server ++ : checkAccountExist(email)
return email doesn't exist
Server -> Cinema ++ : addClient(client)
return success
return account created successfully
end
@enduml
```

### Souscrire à un abonnement Cinépass


Le client souhaite souscrire à un abonnement Cinépass. Pour ce faire, il doit disposer d'un compte de fidelité.
Cette souscription s'effectue sur le site web du cinéma. Avant de procéder, le client doit s'identifier.

Si ce n'est pas déjà le cas, il est invité à rentrer son email et son mot de passe. Ces informations seront envoyées et vérifiées par le serveur. Si les identifiants sont corrects, le client est connecté. Sinon, il est invité à réessayer avec un message d'erreur.

Si le serveur détecte que le compte client est déjà associé à un Cinépass, un message d'erreur est renvoyé au client. Il ne peut pas souscrire à un autre Cinépass sur un même compte.

Une fois connecté, le client peut démarrer le processus de souscription. Pour ce faire, il doit rentrer un IBAN (qui servira à prélever le paiement tous les mois) et une address de facturation. Ces informations seront stockées par le serveur pendant la durée de l'abonnement.
Une fois ces informations rentrées et traitées par le serveur, le client est redirigé vers une page de confirmation de souscription. Il peut alors accéder à son Cinépass.

```mermaid
sequenceDiagram
actor Client
participant Site internet

Client->>Site internet: Ouverture du site
Site internet->>Client: Affichage de la page d'accueil
Client->>Site internet: Clic sur "Souscrire à un abonnement Cinépass"
alt Le client n'est pas connecté
Site internet->>Client: Affichage de la page de connexion
Client->>Site internet: Remplissage du formulaire de connexion
critical Validité des données
    option Email ou mot de passe incorrect
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Modification des données
end
end

critical Déjà abonné
    option Oui
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Clic sur "Retour à l'accueil"
        Site internet->>Client: Affichage de la page d'accueil
    end

Site internet->>Client: Affichage de la page de souscription à un abonnement Cinépass contenant un formulaire avec les champs suivants : addresse de facturation et IBAN
Client->>Site internet: Remplissage du formulaire
Site internet->>Client: Affichage de la page de confirmation de souscription à un abonnement Cinépass
```

#### Digramme de séquence

```plantuml
@startuml
alt account error login
ClientPerson -> Server ++ : login(email, password)
return wrong email or password
else login ok
ClientPerson -> Server ++ : login(email, password)
return connected successfully
end
alt account already subscribed
ClientPerson -> Server ++ : createCinePass(iban)
Server -> Cinema ++ : getCard(client)
return cinepass
return error already subscribed
else account ok
ClientPerson -> Server ++ : createCinePass(iban)
Server -> Cinema ++ : getCard(client)
return no card
Server -> Cinema ++ : addCard(client, cinepass)
return success
return cinepass created successfully
end
@enduml
```

### Désabonnement du Cinépass

Une fois abonné, un client peut se désabonner de son Cinépass à tout moment. Il aura toujours accès à ses avantages jusqu'à la fin de son abonnement et il ne sera pas prélevé à nouveau.

Cette opération s'effectue sur le site web. Avant de procéder, le client doit s'identifier.

Si ce n'est pas déjà le cas, il est invité à rentrer son email et son mot de passe. Ces informations seront envoyées et vérifiées par le serveur. Si les identifiants sont corrects, le client est connecté. Sinon, il est invité à réessayer avec un message d'erreur.

Le serveur va ensuite vérifier si le client est abonné. Si ce n'est pas le cas, un message d'erreur est renvoyé au client et celui-ci peut retourner sur la page d'accueil.

Sinon, le serveur va vérifier si le client a déjà demandé un désabonnement. Si c'est le cas, un message d'erreur est renvoyé au client.

Sinon, le client sera invité à confirmer sa demande de désabonnement. Il devra alors rentrer son mot de passe pour confirmer. Si le mot de passe est correct, le serveur enregistre la demande de désabonnement et le client est redirigé vers la page d'accueil. Sinon, un message d'erreur est renvoyé au client et celui-ci peut réessayer.

```mermaid
sequenceDiagram
actor Client
participant Site internet

Client->>Site internet: Ouverture du site
Site internet->>Client: Affichage de la page d'accueil

alt Le client n'est pas connecté
Site internet->>Client: Affichage de la page de connexion
Client->>Site internet: Remplissage du formulaire de connexion
critical Validité des données
    option Email ou mot de passe incorrect
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Modification des données
end
end

Client->>Site internet: Clic sur "Gérer mon Cinépass"
Site internet->>Client: Affichage de la page de gestion du Cinépass
Client->>Site internet: Clic sur "Se désabonner"

critical Éligibilité au désabonnement
    option Non abonné
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Retour à la page d'accueil
    option Déjà demandé
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Retour à la page d'accueil
end


Site internet->>Client: Affichage de la page de confirmation de désabonnement contenant un formulaire qui demande le mot de passe
Client->>Site internet: Remplissage du formulaire
critical Validité des données
    option Mot de passe incorrect
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Modification des données
end
Site internet->>Site internet: Enregistrement de la demande de désabonnement
Site internet->>Site internet: Envoi d'un email de confirmation
Site internet->>Client: Affichage de la page de confirmation
```

#### Digramme de séquence

```plantuml
@startuml
alt account error login
ClientPerson -> Server ++ : login(email, password)
return wrong email or password
else login ok
ClientPerson -> Server ++ : login(email, password)
return connected successfully
end
ClientPerson -> Server ++ : getProfileInfo(clientid)
return profile info
alt card already unsubscribed
ClientPerson -> Server ++ : unsubrscribeCard(idCard)
Server -> Cinema ++ : getCard(idCard)
return expired cinepass
return error cinepass already unsubscribed
else unsubscribe successfully
ClientPerson -> Server ++ : unsubrscribeCard(idCard)
Server -> Cinema ++ : getCard(idCard)
return cinepass
return Successfully unsubscribed
end
@enduml
```

### Réserver 10 séances à un tarif avantageux (cinécarte)

Un client peut faire le choix d'acheter 10 places de cinéma en meme temps pour un prix plus avantageux. Ces places seront assignées à une carte lui permettant de réserver des séances cinémas en renseignant le numéro de carte.
Pour réserver 10 places, le client doit se rendre sur le site internet. Il peut alors ensuite cliquer sur un bouton "Achat d'une cinécarte". Il est alors invité à se connecter. Cette connexion est requise pour réserver les 10 places. Si il n'a pas de compte, il est invité à un en créer un. Sinon il peut renseigner ses informations de connexions. Le système reçoit ces informations et vérifie bien qu'elles correspondent bien à compte client. Si ce n'est pas le cas, elle renvoie un message d'erreur au client.
Sinon celui-ci est connecté et la page de paiement s'affiche devant lui. Il peut alors renseigner ses informations de paiements et valider. Les informations de paiements sont envoyés au serveur ainsi qu'un token (qui est généré lorsqu'un client se connecte). Ce token est utile pour reconnaitre quel client fait la requete au serveur.
Le système vérifie dans un premier temps que le client ne possède pas deja une carte contenant des places, si c'est le cas, il ajoutera 10 places à la carte si les informations de paiement sont corrects. Nous avons fait le choix de ne pas recréer de carte à chaque fois pour que le client puisse conserver toujours la meme carte et la recharger quand il le souhaite.
Si le client n'a pas de carte, une carte est alors créer avec 10 places dessus. Un message de succès est ensuite envoyé au client pour lui confirmer l'achat. Néanmoins, si le paiement échoue, un message d'erreur lui sera retourner l'invitant à revérifier ses informations de paiements.

```mermaid
sequenceDiagram
actor Client
participant Site internet
participant Banque

Client->>Site internet: Ouverture du site
Site internet->>Client: Affichage de la page d'accueil
Client->>Site internet: Clic sur "Achat d'une cinécarte"
alt Le client n'est pas connecté
Site internet->>Client: Affichage de la page de connexion
Client->>Site internet: Remplissage du formulaire de connexion
critical Validité des données
    option Email ou mot de passe incorrect
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Modification des données
end
end

Site internet->>Client: Affichage de la page d'achat d'une cinécarte contenant un formulaire avec les champs suivants : numéro de carte bancaire, date d'expiration, cryptogramme visuel
Client->>Site internet: Remplissage du formulaire
critical Validité des données
    Site internet->>Banque: Vérification de la validité de la carte bancaire
    option Numéro de carte bancaire invalide
        Site internet->>Client: Affichage d'un message d'erreur
        Client->>Site internet: Modification des données
end

Site internet->>Client: Affichage de la page de confirmation de l'achat d'une cinécarte avec le numéro de la cinécarte
Site internet->>Site internet: Envoi d'un email de confirmation à l'adresse email du client avec le numéro de la cinécarte

```

#### Diagramme de Séquence :

```plantuml
@startuml
alt account error login
ClientPerson -> Server ++ : login(email, password)
return wrong email or password
else login ok
ClientPerson -> Server ++ : login(email, password)
return connected successfully
end
alt error payment
ClientPerson -> Server ++ : buy10TicketsCard(paymentInfo)
return error with payment
else account already has a card
ClientPerson -> Server ++ : buy10TicketsCard(paymentInfo)
Server -> Cinema ++ : getCard(client)
return 10ticket card found
Server -> Server ++ : add 10 tickets to existing card
return succesfully added
return succesfully paid & added
else account ok
ClientPerson -> Server ++ : buy10TicketsCard(paymentInfo)
Server -> Cinema ++ : getCard(client)
return no card found
Server -> Cinema ++ : addCard(client, 10ticketcard)
return succesfully added
return succesfully paid & added
end
@enduml
```

### Réserver un ticket de cinéma

Pour réserver un ticket de cinéma, un client peut utiliser une borne du cinéma ou alors se rendre sur le site internet. Dans ces deux cas, il peut effectuer la réservation en se connectant à son compte ou pas.
Dans tous les cas de figures, le client navigue d'une part parmi les films proposés, et ensuite parmi les séances disponibles dans le cinéma. 

Une fois son choix réalisé, il peut alors se connecter, créer un compte si il n'en a pas ou simplement ignorer la connection.

Si le client fait le choix de se connecter, le système lui proposera d'utiliser une carte cinépass ou de tickets si il en possède. Si c'est son anniversaire et qu'il n'a pas encore utilisé sa place gratuite il pourra également le faire à cette séance.
Si son nombre de points de fidélité est assez élevé, il pourra également payer une ou plusieurs places avec ses points. 

Si il ne se connecte pas, il est invité à entrer un email pour pouvoir recevoir ses places sur celui-ci.

Une requete est ensuite effectué pour récupérer les prix de la séance (qui dépendent du type de salle).
Il peut ensuite séléctionner le nombre de place qu'il souhaite, et choisir pour chaque place le moyen de paiement : Carte (en indiquant le numéro de carte si ce n'est pas l'une des siennes), anniversaire, fidélité ou standard).

Dans le cas d'un paiement standard, il peut choisir une réduction du prix de la séance en fonction du client qui prendra la place (Enfant, Etudiant, Retraité, Sans emploi, Sans réduction). Cette réduction est un pourcentage qui s'applique sur le prix de la séance.

Une fois les informations renseignées, une requete est faite au système qui va vérifier la validité des cartes utilisés et/ou de l'utilisation des points de fidélités/place anniversaire. Si il y a un problème, le système renvoie au client les informations qui ne vont pas.
Le client peut alors modifier son choix puis revalider. Si toutes les informations sont bonnes et que le client doit payer un montant supérieur à 0€, il est invité à payer par carte bancaire (ou espèces sur bornes). Le paiement est ensuite validé (ou non si erreur) et les places sont alors réservés (et crédités des cartes si elles sont utilisées).

Les tickets sont ensuite envoyés par mail au client qui a passé la commande.

#### Réservation sur le site internet

##### Avec un compte

```mermaid
sequenceDiagram
actor Client
Client ->> Site internet: Ouverture
Site internet ->> Client: Liste des films
Client ->> Site internet: Choix du film
Site internet ->> Client: Liste des séances
Client ->> Site internet: Choix de la séance

Site internet ->> Client: Connexion, incription ou continuer sans compte

alt Le client n'a pas de compte
  Client ->> Site internet: Choix de créer un compte
    Site internet ->> Client: Formulaire d'inscription
    Client ->> Site internet: Remplissage du formulaire
    Site internet ->> Client: Confirmation de l'inscription
else Le client a un compte
    Client ->> Site internet: Choix de se connecter
    Site internet ->> Client: Demande de l'addresse mail et du mot de passe
    Client ->> Site internet: Renseignement des informations
    
    critical Verification des informations
        Site internet ->> Site internet: Vérification des informations
    option Informations incorrectes
        Site internet ->> Client: Affichage d'un message d'erreur
    end
end


alt Possède un Cinépass
    Site internet ->> Client: Indique que le client possède un Cinépass et lui propose de l'utiliser
end

alt Possède une place gratuite suite à un anniversaire ou autre offre promotionnelle
    Site internet ->> Client: Indique que le client possède une place gratuite et lui propose de l'utiliser
end

Borne ->> Client: Demande si le client souhaite utiliser une Cinécarte
alt Utilise une Cinécarte
    Borne ->> Client: Demande le numéro de la Cinécarte
    Client ->> Borne: Insertion du numéro de la Cinécarte
end

Site internet ->> Client: Liste des tarifs (tarif normal, tarif réduit, tarif enfant)

Client ->> Site internet: Choix du tarif et du nombre de places

alt Si le prix à payer est supérieur à 0 (i.e. si le client n'a pas utilisé son Cinépass ou sa Cinécarte)
    Site internet ->> Client: Demande des informations de paiement
    Client ->> Site internet: Renseignement des informations de paiement
    
    critical Verification des informations
        Site internet ->> Banque: Vérification des informations de paiement
    option Informations incorrectes
        Banque ->> Site internet: Signalement que le paiement a échoué
        Site internet ->> Client: Affichage d'un message d'erreur
    end
    Banque ->> Site internet: Paiement validé
end



Site internet ->> Site internet: Envoi d'un mail de confirmation
Site internet ->> Client: Affichage de la confirmation de la réservation
```

##### Sans compte

```mermaid
sequenceDiagram
actor Client
Client ->> Site internet: Ouverture
Site internet ->> Client: Liste des films
Client ->> Site internet: Choix du film
Site internet ->> Client: Liste des séances
Client ->> Site internet: Choix de la séance

Site internet ->> Client: Connexion, incription ou continuer sans compte
Client ->> Site internet: Choix de continuer sans compte
Site internet ->> Client: Demande de l'addresse mail
Client ->> Site internet: Renseignement de l'addresse mail

Borne ->> Client: Demande si le client souhaite utiliser une Cinécarte
alt Utilise une Cinécarte
    Borne ->> Client: Demande le numéro de la Cinécarte
    Client ->> Borne: Insertion du numéro de la Cinécarte
end


Site internet ->> Client: Liste des tarifs (tarif normal, tarif réduit, tarif enfant)

Client ->> Site internet: Choix du tarif et du nombre de places
alt Si le prix à payer est supérieur à 0 (i.e. si le client n'a pas utilisé sa Cinécarte)
    Site internet ->> Client: Demande des informations de paiement
    Client ->> Site internet: Renseignement des informations de paiement
    critical Verification des informations
        Site internet ->> Banque: Vérification des informations de paiement
    option Informations incorrectes
        Banque ->> Site internet: Signalement que le paiement a échoué
        Site internet ->> Client: Affichage d'un message d'erreur
    end
    Banque ->> Site internet: Paiement validé
end

Site internet ->> Site internet: Envoi d'un mail de confirmation
Site internet ->> Client: Affichage de la confirmation de la réservation
```



#### Réservation sur la borne
##### Sans compte

```mermaid
sequenceDiagram
actor Client
Borne ->> Client: Liste des films
Client ->> Borne: Choix du film
Borne ->> Client: Liste des séances
Client ->> Borne: Choix de la séance
Borne ->> Client: Connexion ou continuer sans compte
Client ->> Borne: Choix de continuer sans compte

Borne ->> Client: Demande si le client souhaite utiliser une Cinécarte
alt Utilise une Cinécarte
    Borne ->> Client: Demande le numéro de la Cinécarte
    Client ->> Borne: Insertion du numéro de la Cinécarte
end

Borne ->> Client: Liste des tarifs (tarif normal, tarif réduit, tarif enfant)
Client ->> Borne: Choix du tarif et du nombre de places


alt Si le prix à payer est supérieur à 0 (i.e. si le client n'a pas utilisé sa Cinécarte)

    Borne ->> Client: Demande le moyen de paiement
    
    alt Paiement par carte bancaire
        Client ->> Borne: Choix de payer par carte bancaire
        Borne ->> Client: Demande au client de continuer sur le terminal de paiement
        Client ->> Terminal de paiement: Insertion de la carte
        Terminal de paiement ->> Client: Demande le code PIN
        Client ->> Terminal de paiement: Insertion du code PIN
        critical Verification des informations
            Terminal de paiement ->> Banque: Vérification des informations de paiement
        option Informations incorrectes
            Banque ->> Terminal de paiement: Signalement que le paiement a échoué
            Terminal de paiement ->> Borne: Signalement que le paiement a échoué
            Borne ->> Client: Affichage d'un message d'erreur
        end
        Banque ->> Terminal de paiement: Paiement validé
        Terminal de paiement ->> Borne: Signale que le paiement est validé
        Terminal de paiement ->> Client: Affichage que le paiement est validé
        
    else Paiement par espèces
        Client ->> Borne: Choix de payer par espèces
        Borne ->> Client: Demande d'insérer les billets et pièces
        Client ->> Borne: Insertion des billets et pièces
        critical Verification des pièces et billets
            Borne ->> Borne: Vérification des pièces et billets
        option Montant insuffisant
            Borne ->> Client: Affichage du restant à payer
        option Montant supérieur
            Borne ->> Client: Rendu de la monnaie
        end
        Borne ->> Client: Affichage que le paiement est validé
        
        
    end
end

Borne ->> Client: Affichage de la confirmation de la réservation
Borne ->> Client: Impression du ticket
```

##### Avec compte

```mermaid
sequenceDiagram
actor Client
Borne ->> Client: Liste des films
Client ->> Borne: Choix du film
Borne ->> Client: Liste des séances
Client ->> Borne: Choix de la séance
Borne ->> Client: Connexion ou continuer sans compte
Client ->> Borne: Choix de se connecter
Borne ->> Client: Demande le numéro de carte de fidélité au clavier ou au scanner
Client ->> Borne: Insertion du numéro de carte de fidélité
critical Verification du numéro de carte de fidélité
    Borne ->> Borne: Vérification du numéro de carte de fidélité
option Numéro de carte de fidélité incorrect
    Borne ->> Client: Affichage d'un message d'erreur
end

alt Possède un Cinépass
    Borne ->> Client: Indique que le client possède un Cinépass et lui propose de l'utiliser
end

alt Possède une place gratuite suite à un anniversaire ou autre offre promotionnelle
    Site internet ->> Client: Indique que le client possède une place gratuite et lui propose de l'utiliser
end

Borne ->> Client: Demande si le client souhaite utiliser une Cinécarte
alt Utilise une Cinécarte
    Borne ->> Client: Demande le numéro de la Cinécarte
    Client ->> Borne: Insertion du numéro de la Cinécarte
end

Site internet ->> Client: Liste des tarifs (tarif normal, tarif réduit, tarif enfant)

Client ->> Borne: Choix du tarif et du nombre de places
alt Si le prix à payer est supérieur à 0 (i.e. si le client n'a pas utilisé son Cinépass ou sa Cinécarte)
    Borne ->> Client: Demande le moyen de paiement
    
    alt Paiement par carte bancaire
        Client ->> Borne: Choix de payer par carte bancaire
        Borne ->> Client: Demande au client de continuer sur le terminal de paiement
        Client ->> Terminal de paiement: Insertion de la carte
        Terminal de paiement ->> Client: Demande le code PIN
        Client ->> Terminal de paiement: Insertion du code PIN
        critical Verification des informations
            Terminal de paiement ->> Banque: Vérification des informations de paiement
        option Informations incorrectes
            Banque ->> Terminal de paiement: Signalement que le paiement a échoué
            Terminal de paiement ->> Borne: Signalement que le paiement a échoué
            Borne ->> Client: Affichage d'un message d'erreur
        end
        Banque ->> Terminal de paiement: Paiement validé
        Terminal de paiement ->> Borne: Signale que le paiement est validé
        Terminal de paiement ->> Client: Affichage que le paiement est validé
        
    else Paiement par espèces
        Client ->> Borne: Choix de payer par espèces
        Borne ->> Client: Demande d'insérer les billets et pièces
        Client ->> Borne: Insertion des billets et pièces
        critical Verification des pièces et billets
            Borne ->> Borne: Vérification des pièces et billets
        option Montant insuffisant
            Borne ->> Client: Affichage du restant à payer
        option Montant supérieur
            Borne ->> Client: Rendu de la monnaie
        end
        Borne ->> Client: Affichage que le paiement est validé
        
        
    end
end

Borne ->> Client: Affichage de la confirmation de la réservation
Borne ->> Client: Impression du ticket
```

#### Diagramme de Séquence

```plantuml
@startuml
ClientPerson -> Server ++ : getMovies()
return movies
ClientPerson -> Server ++ : getScreenings(date, movie)
return screenings
alt login (optional)
ClientPerson -> Server ++ : login(email, password)
Server -> Cinema ++ : getCard(client)
return cards found
return connected successfully, return cards found
end
ClientPerson -> Server ++ : getPrices(screening)
return prices
alt error with cards
ClientPerson -> Server ++ : bookTickets(email, screening, bookings, paymentInfo)
Server -> Cinema ++ : getCard(idCard)
return card with no tickets (or not valid/found)
return error with cards (not enough tickets or not valid etc.)
else error with payment
ClientPerson -> Server ++ : bookTickets(email, screening, bookings, paymentInfo)
return error message with payment
else booking ok
ClientPerson -> Server ++ : bookTickets(email, screening, bookings, paymentInfo)
return tickets numbers
end
@enduml
```

### Gérer la répartition des séances dans les salles

La gestion de la répartition des séances dans les salles est effectuée par un membre du personnel. Cette opération s'effectue sur un client accessible en interne, mais partageant la même base de données que le site web accessible au public.

Ce site n'est accessible que sur le réseau local du cinéma, il ne nécessite pas d'authentification supplémentaire.

Sur celui-ci, une rubrique "Répartition des séances" est disponible.
À partir de là, le membre du personnel peut former une association entre une salle, un film et un horaire.
Une fois cette association créée, le serveur va vérifier si la salle est disponible à l'horaire indiqué. Si c'est le cas, l'association est enregistrée et la salle est marquée comme occupée à l'horaire indiqué. Sinon, un message d'erreur est renvoyé au membre du personnel.

Ces modifications sont ensuite enregistrées par le serveur et appliquées sur le site web principal.
Les clients peuvent alors réserver des places pour cette séance.

```mermaid
sequenceDiagram
actor Membre du personnel
participant Site internet de gestion interne

Membre du personnel->>Site internet de gestion interne: Ouverture du site
Site internet de gestion interne->>Membre du personnel: Affichage de la page d'accueil
Membre du personnel->>Site internet de gestion interne: Clic sur "Gérer la répartition des séances dans les salles"
Site internet de gestion interne->>Membre du personnel: Affichage de la page de gestion de la répartition des séances dans les salles contenant un formulaire avec les champs suivants : salle, film, horaire...
Membre du personnel->>Site internet de gestion interne: Remplissage du formulaire
critical Validité des données
    option Salle déjà occupée à l'horaire indiqué
        Site internet de gestion interne->>Membre du personnel: Affichage d'un message d'erreur
        Membre du personnel->>Site internet de gestion interne: Modification des données
end
Site internet de gestion interne->>Membre du personnel: Affichage de la page de confirmation
```
#### Digramme de séquence

```plantuml
@startuml
alt screening already exist
CinemaEmployee -> EmployeeMonitor ++ : addScreening(movie, datetime, room)
return room already used at this time
else screening ok
CinemaEmployee -> EmployeeMonitor ++ : addScreening(movie, datetime, room)
return screening added successfully
end
@enduml
```

### Changer la grille tarifaire

Le changement de grille tarifaire est effectué par un membre du personnel. Cette opération s'effectue sur un client accessible en interne, mais partageant la même base de données que le site web accessible au public.

Ce site n'est accessible que sur le réseau local du cinéma, il ne nécessite pas d'authentification supplémentaire.

Sur celui-ci, une rubrique "Grille tarifaire" est disponible.
Elle permet de modifier les tarifs en fonction du type de salle (standard, 3D et Dolby) et également modifier les réductions à apporter aux étudiants, aux enfants et aux personnes âgées etc...
Ces modifications sont ensuite enregistrées par le serveur et appliquées sur le site web principal.

```mermaid
sequenceDiagram
actor Membre du personnel
participant Site internet de gestion interne

Membre du personnel->>Site internet de gestion interne: Ouverture du site
Site internet de gestion interne->>Membre du personnel: Affichage de la page d'accueil
Membre du personnel->>Site internet de gestion interne: Clic sur "Changer la grille tarifaire"
Site internet de gestion interne->>Membre du personnel: Affichage de la page de changement de la grille tarifaire contenant un formulaire avec les champs suivants : type de salle, tarif normal et réductions apportées aux tarifs réduits
Membre du personnel->>Site internet de gestion interne: Remplissage du formulaire
Site internet de gestion interne->>Membre du personnel: Affichage de la page de confirmation de changement de la grille tarifaire
```

#### Digramme de séquence
```plantuml
@startuml
CinemaEmployee -> EmployeeMonitor ++ : changePrice(roomType, price)
return changed successfully
@enduml
```

```plantuml
@startuml
CinemaEmployee -> EmployeeMonitor ++ : changeReduction(reductionType, percentage)
return changed successfully
@enduml
```

### Vérification des tickets des clients

Avant de pouvoir rentrer dans les salles, les clients doivent passer par un couloir d'accès où un employé du cinéma vérifie leur ticket.
Pour ce faire, il scan le code barre du ticket et une requete est effectué au système pour récupérer les informations du ticket.

Si le système ne renvoit aucune information, alors le ticket n'existe pas dans le système et n'est donc pas valide.
Le ticket peut également être invalide si il est associé à une séance passé ou trop éloigné de la date et heure actuelle.

Après un scan de ticket, l'employé peut voir quelle réduction a été appliqué lors de l'achat du ticket et donc demandé une pièce justificative si nécessaire.

```mermaid
sequenceDiagram
actor Membre du personnel
actor Client
participant Ordinateur de contrôle des entrées

Client->>Membre du personnel: Présentation du ticket pour la séance
Membre du personnel->>Client: Scan du ticket
Membre du personnel->>Ordinateur de contrôle des entrées: Vérification de la validité du ticket
critical Validité du ticket
    option Ticket invalide
        Ordinateur de contrôle des entrées->>Membre du personnel: Affichage d'un message d'erreur
        Client->>Membre du personnel: Demande d'un nouveau ticket
end
alt Le ticket nécessite un contrôle des pièces justificatives
    Membre du personnel->>Client: Demande des pièces justificatives (carte d'itentité...)
    Client->>Membre du personnel: Présentation des pièces justificatives
    critical Validité des pièces justificatives
        Membre du personnel->>Membre du personnel: Vérification des pièces justificatives
        option Pièces justificatives invalides
            Membre du personnel->>Client: Demande d'une nouvelle pièce justificative
        end
end
Ordinateur de contrôle des entrées->>Membre du personnel: Affichage du nom du film et de la salle
Membre du personnel->>Client: Indique la salle à rejoindre
```

#### Diagramme de séquence

```plantuml
@startuml
alt ticket id doesn't exist
CinemaEmployee -> EmployeeMonitor ++ : getTicket(ticketID)
return null
else ticket exist
CinemaEmployee -> EmployeeMonitor ++ : getTicket(ticketID)
return ticket (reductionType, Screening, id)
end
@enduml
```