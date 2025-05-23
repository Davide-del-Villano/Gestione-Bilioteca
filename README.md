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

### Crea libro
- **Funzione:** createBook()
- **Input:** BookDTO bookDTO
- **Output:** ResponseEntity<BookDTO>
- **Endpoint:** POST /api/v1/books

### Aggiorna libro
- **Funzione:** updateBook()
- **Input:** Long id, BookDTO bookDTO
- **Output:** ResponseEntity<BookDTO>
- **Endpoint:** PUT /api/v1/books/{id}

### Elimina libro
- **Funzione:** deleteBook()
- **Input:** Long id
- **Output:** ResponseEntity<Void>
- **Endpoint:** DELETE /api/v1/books/{id}

### Elenco libri
- **Funzione:** getBooks()
- **Input:** Filtro (opzionale)
- **Output:** List<BookDTO>
- **Endpoint:** GET /api/v1/books

### Dettaglio libro
- **Funzione:** getBookById()
- **Input:** Long id
- **Output:** BookDTO
- **Endpoint:** GET /api/v1/books/{id}

### Ricerca nel catalogo
- **Funzione:** searchCatalog()
- **Input:** QueryParam (titolo, autore, ISBN)
- **Output:** List<BookDTO>
- **Endpoint:** GET /api/v1/catalog/search

### Libri disponibili
- **Funzione:** getAvailableBooks()
- **Input:** Nessuno
- **Output:** List<BookDTO>
- **Endpoint:** GET /api/v1/catalog/available

### Crea prenotazione
- **Funzione:** createReservation()
- **Input:** Long bookId
- **Output:** ReservationDTO
- **Endpoint:** POST /api/v1/reservations
- **Nota:** Richiede ID utente dal microservizio utenti

### Elenco prenotazioni
- **Funzione:** getReservations()
- **Input:** Nessuno
- **Output:** List<ReservationDTO>
- **Endpoint:** GET /api/v1/reservations
- **Nota:** Verifica ruolo utente tramite microservizio utenti

### Dettaglio prenotazione
- **Funzione:** getReservationById()
- **Input:** Long id
- **Output:** ReservationDTO
- **Endpoint:** GET /api/v1/reservations/{id}

### Annulla prenotazione
- **Funzione:** cancelReservation()
- **Input:** Long id
- **Output:** ResponseEntity<Void>
- **Endpoint:** DELETE /api/v1/reservations/{id}

### Crea prestito
- **Funzione:** createLoan()
- **Input:** LoanRequestDTO (bookId, userId)
- **Output:** LoanDTO
- **Endpoint:** POST /api/v1/loans
- **Nota:** Verifica ID utente tramite microservizio utenti

### Registra restituzione libro
- **Funzione:** returnLoan()
- **Input:** Long loanId
- **Output:** LoanDTO
- **Endpoint:** PUT /api/v1/loans/{id}/return

### Elenco prestiti
- **Funzione:** getLoans()
- **Input:** Nessuno
- **Output:** List<LoanDTO>
- **Endpoint:** GET /api/v1/loans

### Dettaglio prestito
- **Funzione:** getLoanById()
- **Input:** Long id
- **Output:** LoanDTO
- **Endpoint:** GET /api/v1/loans/{id}

### Prestiti in ritardo
- **Funzione:** getOverdueLoans()
- **Input:** Nessuno
- **Output:** List<LoanDTO>
- **Endpoint:** GET /api/v1/loans/overdue

### Richiesta prestito
- **Funzione:** requestLoan()
- **Input:** Long bookId
- **Output:** LoanRequestDTO con stato PENDING
- **Endpoint:** POST /api/v1/loans/request

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
