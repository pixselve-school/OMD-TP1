```plantuml
@startuml
left to right direction

actor "Client" as c

actor "Membre du personnel" as p

rectangle "Système de réservation du cinéma" {
  usecase "Réservation d'une séance de cinéma" as UC1
  usecase "Gérer la répartition des séances dans les différentes salles" as UC2
  usecase "Créer un compte de fidelité" as UC3
  usecase "Souscrire à un abonnement Cinépass" as UC4
  usecase "Réserver 10 séances à un tarif avantageux" as UC5
  usecase "Changer la grille tarifaire" as UC6
  usecase "Verification des pièces justificatifs des clients" as UC7
}
c --> UC1
p --> UC2
c --> UC3
c --> UC4
c --> UC5
p --> UC6
p --> UC7

UC3 -.-> UC4
UC3 -.-> UC5
@enduml
```

# Diagrammes de séquence système


## Créer un compte de fidelité
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

## Souscrire à un abonnement Cinépass
```mermaid
sequenceDiagram
actor Client
actor Membre du personnel
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

Site internet->>Client: Affichage de la page de souscription à un abonnement Cinépass contenant un formulaire avec les champs suivants : addresse de facturation et IBAN
Client->>Site internet: Remplissage du formulaire
Site internet->>Client: Affichage de la page de confirmation de souscription à un abonnement Cinépass
```

## Réserver 10 séances à un tarif avantageux (cinécarte)
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

## Changer la grille tarifaire
```mermaid
sequenceDiagram
actor Membre du personnel
participant Site internet de gestion interne

Membre du personnel->>Site internet de gestion interne: Ouverture du site
Site internet de gestion interne->>Membre du personnel: Affichage de la page d'accueil
Membre du personnel->>Site internet de gestion interne: Clic sur "Changer la grille tarifaire"
Site internet de gestion interne->>Membre du personnel: Affichage de la page de changement de la grille tarifaire contenant un formulaire avec les champs suivants : tarif normal, tarif réduit, tarif enfant...
Membre du personnel->>Site internet de gestion interne: Remplissage du formulaire
Site internet de gestion interne->>Membre du personnel: Affichage de la page de confirmation de changement de la grille tarifaire

```

## Vérification des pièces justificatives des clients
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

## Gérer la répartition des séances dans les salles
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

## Réservation sur le site internet 

### Avec un compte

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
    critical Verification du numéro de la Cinécarte
        Borne ->> Borne: Vérification du numéro de la Cinécarte
    option Numéro de la Cinécarte incorrect
        Borne ->> Client: Affichage d'un message d'erreur
    end
    Borne ->> Client: Indique le nombre de place restante sur la Cinécarte et lui propose de les utiliser
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

### Sans compte

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
    critical Verification du numéro de la Cinécarte
        Borne ->> Borne: Vérification du numéro de la Cinécarte
    option Numéro de la Cinécarte incorrect
        Borne ->> Client: Affichage d'un message d'erreur
    end
    Borne ->> Client: Indique le nombre de place restante sur la Cinécarte et lui propose de les utiliser
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



## Réservation sur la borne
### Sans compte

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
    critical Verification du numéro de la Cinécarte
        Borne ->> Borne: Vérification du numéro de la Cinécarte
    option Numéro de la Cinécarte incorrect
        Borne ->> Client: Affichage d'un message d'erreur
    end
    Borne ->> Client: Indique le nombre de place restante sur la Cinécarte et lui propose de les utiliser
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

### Avec compte

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
    critical Verification du numéro de la Cinécarte
        Borne ->> Borne: Vérification du numéro de la Cinécarte
    option Numéro de la Cinécarte incorrect
        Borne ->> Client: Affichage d'un message d'erreur
    end
    Borne ->> Client: Indique le nombre de place restante sur la Cinécarte et lui propose de les utiliser
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