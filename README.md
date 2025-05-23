# Microservizio Gestione Biblioteca

## Panoramica

Il microservizio Gestione Biblioteca si occupa della gestione delle operazioni relative ai libri, agli utenti e alle prenotazioni in una biblioteca universitaria. Le principali funzionalità offerte sono:

- Ricerca e consultazione del catalogo libri (studenti e docenti)
- Prenotazione, prestito e restituzione dei libri (studenti e docenti)
- Gestione dell'inventario librario (amministrativi)
- Visualizzazione dello stato delle prenotazioni o dei prestiti personali (tutti gli utenti)

## Operazioni Gestite

- Gestione dei dati bibliografici dei libri in biblioteca (amministrativi)
- Gestione dell'acquisizione e rimozione di copie di un determinato libro (amministrativi)
- Richiesta di prestito da parte degli studenti
- Gestione dei prestiti e delle restituzioni (amministrativi)

## Stack tecnologico

- Struttura: Spring Boot
- Database: MySQL
- Broker di messaggi: RabbitMQ
- Containerizzazione: Docker
- Orchestrazione: Kubernetes (non incluso in questa repository)
- Documentazione API: Swagger / OpenAPI 3.0

## Modello dei Dati

### Libro (Book)
- id
- isbn
- title
- author
- publisher
- publication_year
- copies_total
- copies_available

### Utente Biblioteca (LibraryUser)
- id
- external_id
- role
- registered_at

### Prenotazione (Reservation)
- id
- book_id
- user_id
- reservation_date
- status

### Prestito (Loan)
- id
- book_id
- user_id
- loan_date
- due_date
- return_date
- status

## API REST

#############################################
# Crea libro
# @fun: createBook()
# @param: BookDTO bookDTO
# @return: ResponseEntity<BookDTO>
#############################################
POST /api/v1/books

#############################################
# Aggiorna libro
# @fun: updateBook()
# @param: Long id, BookDTO bookDTO
# @return: ResponseEntity<BookDTO>
#############################################
PUT /api/v1/books/{id}

#############################################
# Elimina libro
# @fun: deleteBook()
# @param: Long id
# @return: ResponseEntity<Void>
#############################################
DELETE /api/v1/books/{id}

#############################################
# Elenco libri
# @fun: getBooks()
# @param: Filtro (opzionale)
# @return: List<BookDTO>
#############################################
GET /api/v1/books

#############################################
# Dettaglio libro
# @fun: getBookById()
# @param: Long id
# @return: BookDTO
#############################################
GET /api/v1/books/{id}

#############################################
# Ricerca nel catalogo
# @fun: searchCatalog()
# @param: QueryParam (titolo, autore, ISBN)
# @return: List<BookDTO>
#############################################
GET /api/v1/catalog/search

#############################################
# Libri disponibili
# @fun: getAvailableBooks()
# @param: Nessuno
# @return: List<BookDTO>
#############################################
GET /api/v1/catalog/available

#############################################
# Crea prenotazione
# @fun: createReservation()
# @param: Long bookId
# @return: ReservationDTO
# @note: Richiede ID utente dal microservizio utenti
#############################################
POST /api/v1/reservations

#############################################
# Elenco prenotazioni
# @fun: getReservations()
# @param: Nessuno
# @return: List<ReservationDTO>
# @note: Verifica ruolo utente tramite microservizio utenti
#############################################
GET /api/v1/reservations

#############################################
# Dettaglio prenotazione
# @fun: getReservationById()
# @param: Long id
# @return: ReservationDTO
#############################################
GET /api/v1/reservations/{id}

#############################################
# Annulla prenotazione
# @fun: cancelReservation()
# @param: Long id
# @return: ResponseEntity<Void>
#############################################
DELETE /api/v1/reservations/{id}

#############################################
# Crea prestito
# @fun: createLoan()
# @param: LoanRequestDTO (bookId, userId)
# @return: LoanDTO
# @note: Verifica ID utente tramite microservizio utenti
#############################################
POST /api/v1/loans

#############################################
# Registra restituzione libro
# @fun: returnLoan()
# @param: Long loanId
# @return: LoanDTO
#############################################
PUT /api/v1/loans/{id}/return

#############################################
# Elenco prestiti
# @fun: getLoans()
# @param: Nessuno
# @return: List<LoanDTO>
#############################################
GET /api/v1/loans

#############################################
# Dettaglio prestito
# @fun: getLoanById()
# @param: Long id
# @return: LoanDTO
#############################################
GET /api/v1/loans/{id}

#############################################
# Prestiti in ritardo
# @fun: getOverdueLoans()
# @param: Nessuno
# @return: List<LoanDTO>
#############################################
GET /api/v1/loans/overdue

#############################################
# Richiesta prestito
# @fun: requestLoan()
# @param: Long bookId
# @return: LoanRequestDTO con stato PENDING
#############################################
POST /api/v1/loans/request

## Integrazione con Altri Microservizi

### Gestione Utenti e Ruoli
- Recupero dell'ID esterno e ruolo utente per autorizzazioni
- Verifica permessi prima di prenotazioni/prestiti
- Consumo evento user.deleted per eliminare dati associati

### Comunicazioni e Notifiche
- Invio notifiche su prenotazioni/prestiti via eventi RabbitMQ

### Valutazione o Esami (opzionale)
- Collegamento libri-corsi tramite ID corso

## Eventi RabbitMQ

### Eventi pubblicati
- reservation.created
- loan.started
- loan.returned
- loan.overdue

### Eventi consumati
- user.deleted
- course.ended

## Sicurezza e Autorizzazioni

L'accesso alle API è regolato da ruoli utente forniti dal microservizio autenticazione:
- ADMIN
- TEACHER
- STUDENT

## Esempio di Flusso Utente

1. Uno studente accede al catalogo e prenota un libro.
2. Un amministratore approva e avvia il prestito.
3. Alla restituzione, l'amministratore aggiorna lo stato.
4. In caso di ritardo, viene generata una notifica automatica.
