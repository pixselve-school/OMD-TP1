![Inage](http://www.plantuml.com/plantuml/png/RL4zpjim3Drr2i9BfnJz_hSYW8TENHgqEoFJZWfPDYWgQ57qBXtZS-XYbHn5S81lUoRVpnEzzu8iORJAKYyW4x0PJWATOT9Y9gSKQfaOcjOQSj80Uj1r-9N68nDq0MPYFpb7TcNCIcKzki4ID8VVNk9bJ3m2ZeidFcFspZfjN5n6RE2F0WYUDFga-7R7-X0SWOyBEfqzgYOdVMyVji8lSM5Yi9Y3PcGnLmlonI213fq7cu7f-yntajTeqwnuFcuzssH53C61diPPrWgzwSWQYKiHF6q5XobupUc24Fzb6HxpPSQK0sskFgE_1Js_ysfYUFXKwnOFGJOzu1cTu43XLz6_t1KzeHkkhn_OM4jN5QOcXVwwfV-Wj3GQwuLc4oyvyMVmKZQzNpTw_Vctqc_ghzAmstrUhpyNz5XdJnKzL_HIUQyLlIcLoB3xMBZr8zFtvBemsly0)


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
## Réservation sur le site internet 
```mermaid
sequenceDiagram
actor Client
Client ->> Site internet: Ouverture
Site internet ->> Client: Liste des films
Client ->> Site internet: Choix du film
Site internet ->> Client: Liste des séances
Client ->> Site internet: Choix de la séance
Site internet ->> Client: Connexion, incription ou continuer sans compte
Client ->> Site internet: Choix de ne pas créer de compte

Client ->> Site internet: Choix de ne pas créer de compte
```
