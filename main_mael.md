# TP1 · Conception Orientée-Objet
## Mael KERICHARD - Romain BRIEND

L’objectif de ce TD/TP est de préparer un dossier complet d’analyse et de conception par Objet à partir d’un cahier des charges. 
Ce cahier des charges concerne un système de réservation d'un cinéma Pathé Gaumont. 

## Digramme de cas d'utilisation

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

## Analyse des cas d'utilisation

### Souscrire à un abonnement Cinépass


Le client souhaite souscrire à un abonnement Cinépass. Pour ce faire, il doit disposer d'un compte de fidelité.
Cette souscription s'effectue sur le site web du cinéma. Avant de procéder, le client doit s'identifier.

Si ce n'est pas déjà le cas, il est invité à rentrer son email et son mot de passe. Ces informations seront envoyées et vérifiées par le serveur. Si les identifiants sont corrects, le client est connecté. Sinon, il est invité à réessayer avec un message d'erreur.

Si le serveur détecte que le compte client est déjà associé à un Cinépass, un message d'erreur est renvoyé au client. Il ne peut pas souscrire à un autre Cinépass sur un même compte.

Une fois connecté, le client peut démarrer le processus de souscription. Pour ce faire, il doit rentrer un IBAN (qui servira à prélever le paiement tous les mois) et une address de facturation. Ces informations seront stockées par le serveur pendant la durée de l'abonnement.
Une fois ces informations rentrées et traitées par le serveur, le client est redirigé vers une page de confirmation de souscription. Il peut alors accéder à son Cinépass.

#### Digramme fonctionnel
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

#### Digramme structurel

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

#### Analyse comportementale

```mermaid
stateDiagram-v2

CheckUserConnected: Vérification de la connexion de l'utilisateur
VerifyCredentials: Vérification des identifiants
Connected: Verification de l'abonnement
SaveIban: Sauvegarde de l'IBAN et création de la carte
state check_subscribed <<choice>>

[*] --> CheckUserConnected: L'utilisateur clique sur "Souscrire à un abonnement Cinépass"
state check_connected <<choice>>
CheckUserConnected --> check_connected
check_connected --> Connexion : Non connecté

state Connexion {
    state check_credentials <<choice>>
    [*] --> VerifyCredentials
    VerifyCredentials --> check_credentials
    check_credentials --> VerifyCredentials : Incorrects
    check_credentials --> [*]: Corrects
}

check_connected --> Connected: Connecté
Connexion --> Connected
Connected --> check_subscribed



check_subscribed --> [*] : L'utilisateur est déjà abonné

check_subscribed --> SaveIban : L'utilisateur n'est pas abonné
SaveIban --> [*]
```


### Changer la grille tarifaire

Le changement de grille tarifaire est effectué par un membre du personnel. Cette opération s'effectue sur un client accessible en interne, mais partageant la même base de données que le site web accessible au public.

Ce site n'est accessible que sur le réseau local du cinéma, il ne nécessite pas d'authentification supplémentaire. 

Sur celui-ci, une rubrique "Grille tarifaire" est disponible. 
Elle permet de modifier les tarifs en fonction du type de salle (standard, 3D et Dolby) et spécifier les réductions à apporter aux étudiants, aux enfants et aux personnes âgées.
Ces modifications sont ensuite enregistrées par le serveur et appliquées sur le site web principal.

#### Digramme fonctionnel
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

#### Digramme structurel
```plantuml
@startuml
CinemaEmployee -> EmployeeMonitor ++ : changePrice(roomType, price)
return changed successfully
@enduml
```

#### Analyse comportementale
```mermaid
stateDiagram-v2

ChangePrice: Sauvegarde des nouveaux tarifs

[*] --> ChangePrice: Le membre du personnel change la grille tarifaire
ChangePrice --> [*]
```

### Gérer la répartition des séances dans les salles

La gestion de la répartition des séances dans les salles est effectuée par un membre du personnel. Cette opération s'effectue sur un client accessible en interne, mais partageant la même base de données que le site web accessible au public.

Ce site n'est accessible que sur le réseau local du cinéma, il ne nécessite pas d'authentification supplémentaire.

Sur celui-ci, une rubrique "Répartition des séances" est disponible.
À partir de là, le membre du personnel peut former une association entre une salle, un film et un horaire.
Une fois cette association créée, le serveur va vérifier si la salle est disponible à l'horaire indiqué. Si c'est le cas, l'association est enregistrée et la salle est marquée comme occupée à l'horaire indiqué. Sinon, un message d'erreur est renvoyé au membre du personnel.

Ces modifications sont ensuite enregistrées par le serveur et appliquées sur le site web principal.
Les clients peuvent alors réserver des places pour cette séance.

#### Digramme fonctionnel
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
#### Digramme structurel
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

#### Analyse comportementale
```mermaid
stateDiagram-v2

AddScreening: Verifie si la salle est disponible à l'horaire indiqué
SaveScreening: Enregistre la nouvelle séance

[*] --> AddScreening: Le membre du personnel ajoute une séance

state check_room <<choice>>
AddScreening --> check_room
check_room --> [*] : Occupée

check_room --> SaveScreening : Disponible
SaveScreening --> [*]

```

### Désabonnement du Cinépass

Une fois abonné, un client peut se désabonner de son Cinépass à tout moment. Il aura toujours accès à ses avantages jusqu'à la fin de son abonnement et il ne sera pas prélevé à nouveau.

Cette opération s'effectue sur le site web. Avant de procéder, le client doit s'identifier.

Si ce n'est pas déjà le cas, il est invité à rentrer son email et son mot de passe. Ces informations seront envoyées et vérifiées par le serveur. Si les identifiants sont corrects, le client est connecté. Sinon, il est invité à réessayer avec un message d'erreur.

Le serveur va ensuite vérifier si le client est abonné. Si ce n'est pas le cas, un message d'erreur est renvoyé au client et celui-ci peut retourner sur la page d'accueil.

Sinon, le serveur va vérifier si le client a déjà demandé un désabonnement. Si c'est le cas, un message d'erreur est renvoyé au client.

Sinon, le client sera invité à confirmer sa demande de désabonnement. Il devra alors rentrer son mot de passe pour confirmer. Si le mot de passe est correct, le serveur enregistre la demande de désabonnement et le client est redirigé vers la page d'accueil. Sinon, un message d'erreur est renvoyé au client et celui-ci peut réessayer.

#### Digramme fonctionnel
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

#### Digramme comportemental
```mermaid

```
