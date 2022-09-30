## Analyse Fonctionnelle

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

## Analyse Comportementale

### Diagramme de Séquence :

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

### Diagramme d'Etats :
```plantuml
@startuml
start
-> Client Informations received;
if (Check if email already exist) then (yes)
    :Error creating account;
    :Error message to client;
    end
else (no)
    :Add Client information to the cinema system;
    :Send success message to client;
    stop
@enduml
```

L'utilisateur peut faire le choix de créer un compte client pour le cinéma. Pour cela, il doit se rendre sur le site du cinéma.
Sur celui-ci, il pourra trouver un bouton pour créer un compte qui l'amenera vers un formulaire de création de compte. Il pourra alors remplir les données utilisées pour la création d'un compte (nom, prénom, email, date de naissance, adresse et son mot de passe).
Une fois renseignées, ces informations sont envoyés au système et execute la fonction createAccount qui commencera par vérifier que la personne ne possède pas déja un compte. Si elle possède deja un compte, elle lui renverra un message d'erreur. Sinon elle stockera les informations du client comme compte client dans le système.
Le client recevra alors un message de succès.