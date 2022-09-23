## Réserver 10 séances à un tarif avantageux (cinécarte)

### Analyse Fonctionnelle

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

## Analyse Comportementale

### Diagramme de Séquence :

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
ClientPerson -> Server ++ : buy10TicketsCard(creditCard, expirationdate, cryptonumber)
return error with payment
else account already has a card
ClientPerson -> Server ++ : buy10TicketsCard(creditCard, expirationdate, cryptonumber)
Server -> Cinema ++ : getCard(client)
return 10ticket card found
Server -> Server ++ : add 10 tickets to existing card
return succesfully added
return succesfully paid & added
else account ok
ClientPerson -> Server ++ : buy10TicketsCard(creditCard, expirationdate, cryptonumber)
Server -> Cinema ++ : getCard(client)
return no card found
Server -> Cinema ++ : addCard(client, 10ticketcard)
return succesfully added
return succesfully paid & added
end
@enduml
```

### Diagramme d'Etats :
```plantuml
@startuml
(*) --> "Tesstt etset"
"First Action" --> (*)
@enduml
```

Un client peut faire le choix d'acheter 10 places de cinéma en meme temps pour un prix plus avantageux. Ces places seront assignées à une carte lui permettant de réserver des séances cinémas en renseignant le numéro de carte.
Pour réserver 10 places, le client doit se rendre sur le site internet. Il peut alors ensuite cliquer sur un bouton "Achat d'une cinécarte". Il est alors invité à se connecter. Cette connexion est requise pour réserver les 10 places. Si il n'a pas de compte, il est invité à un en créer un. Sinon il peut renseigner ses informations de connexions. Le système reçoit ces informations et vérifie bien qu'elles correspondent bien à compte client. Si ce n'est pas le cas, elle renvoie un message d'erreur au client.
Sinon celui-ci est connecté et la page de paiement s'affiche devant lui. Il peut alors renseigner ses informations de paiements et valider. Les informations de paiements sont envoyés au serveur ainsi qu'un token (qui est généré lorsqu'un client se connecte). Ce token est utile pour reconnaitre quel client fait la requete au serveur.
Le système vérifie dans un premier temps que le client ne possède pas deja une carte contenant des places, si c'est le cas, il ajoutera 10 places à la carte si les informations de paiement sont corrects. Nous avons fait le choix de ne pas recréer de carte à chaque fois pour que le client puisse conserver toujours la meme carte et la recharger quand il le souhaite. 
Si le client n'a pas de carte, une carte est alors créer avec 10 places dessus. Un message de succès est ensuite envoyé au client pour lui confirmer l'achat. Néanmoins, si le paiement échoue, un message d'erreur lui sera retourner l'invitant à revérifier ses informations de paiements.

