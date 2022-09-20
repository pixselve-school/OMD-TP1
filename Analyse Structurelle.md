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
        +Client client
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
        +boolean buy10TicketsCard(String creditCard, Date expirationdate, int cryptonumber)
        +Screening[] getScreenings(DateTime date, Movie movie)
        +Movie[] getMovies()
        +boolean bookTickets(String email, Screening screening, Booking[] bookings, paymentInfo paymentInfo)
        +HashMap~ReductionType, int~ getPrices(Screening screening)
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
        +boolean checkValidityTicket(long ticketId)
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
    
    class paymentInfo{
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
    }
```

# Diagrammes de séquence

## Créer un compte de fidelité
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

## Souscrire à un abonnement Cinépass
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

## Réserver 10 séances à un tarif avantageux (cinécarte)
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

## Changer la grille tarifaire
```plantuml
@startuml
CinemaEmployee -> EmployeeMonitor ++ : changePrice(roomType, price)
return changed successfully
@enduml
```

## Vérification de validité d'un ticket
```plantuml
@startuml
alt ticket not valid
CinemaEmployee -> EmployeeMonitor ++ : checkValidityTicket(ticketId)
return ticket not valid
else screening date not corresponding to current date and/or time
CinemaEmployee -> EmployeeMonitor ++ : checkValidityTicket(ticketId)
return ticket datetime screening not corresponding to current date and/or time
else ticket valid
CinemaEmployee -> EmployeeMonitor ++ : checkValidityTicket(ticketId)
return ticket valid
end
@enduml
```

## Gérer la répartition des séances dans les salles
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

```plantuml
@startuml
alt screening doesn't exist
CinemaEmployee -> EmployeeMonitor ++ : removeScreening(datetime, room)
return screening doesn't exist
else screening ok
CinemaEmployee -> EmployeeMonitor ++ : removeScreening(datetime, room)
return screening removed successfully
end
@enduml
```

## Réservation sur le site internet ou sur la borne

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