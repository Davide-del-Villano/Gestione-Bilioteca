# Gestione-Bilioteca

## Panoramica
Questo microservizio si occupa della gestione delle operazioni relative ai libri, agli utenti e alle prenotazioni in una biblioteca universitaria:  
• (Studenti e Docenti) Ricerca e consultazione del catalogo libri  
• (Studenti e Docenti) Prenotazione, prestito e restituzione dei libri  
• (Amministrativi) Gestione dell’inventario librario  
• (Tutti) Visualizzazione dello stato delle proprie prenotazioni o prestiti  



## Stack tecnologico
• Struttura: Spring Boot  
• Database: MySQL  
• Broker di messaggi: RabbitMQ  
• Containerizzazione: Docker  
• Orchestrazione: Kubernetes (non incluso in questa repo)  
• Documentazione API: Swagger / OpenAPI 3.0  


## Modello dei Dati

### Libro (Book)
• id: ID del libro  
• isbn: Codice ISBN  
• title: Titolo  
• author: Autore  
• publisher: Casa editrice  
• publication_year: Anno di pubblicazione  
• copies_total: Numero totale di copie  
• copies_available: Numero di copie disponibili  

### Utente Biblioteca (LibraryUser)
• id: ID utente  
• external_id: ID dell’utente nel sistema esterno (es. gestione utenti)  
• role: Ruolo (enum: STUDENT, TEACHER, ADMIN)  
• registered_at: Data di registrazione nel sistema biblioteca  

### Prenotazione (Reservation)
• id: ID prenotazione  
• book_id: ID del libro  
• user_id: ID utente  
• reservation_date: Data di prenotazione  
• status: Stato (enum: ATTIVA, CANCELLATA, SCADUTA, EROGATA)  

### Prestito (Loan)
• id: ID prestito  
• book_id: ID libro  
• user_id: ID utente  
• loan_date: Data inizio prestito  
• due_date: Data di scadenza  
• return_date: Data di restituzione effettiva (nullable)  
• status: Stato (enum: IN_CORSO, RESTITUITO, IN_RITARDO)  



## API REST

### Gestione Libri (Admin)
• POST /api/v1/books - Aggiungi un nuovo libro  
• PUT /api/v1/books/{id} - Modifica un libro esistente  
• DELETE /api/v1/books/{id} - Rimuovi un libro  
• GET /api/v1/books - Elenco dei libri con filtri (titolo, autore, ISBN)  
• GET /api/v1/books/{id} - Dettagli di un libro  

### Catalogo e Ricerca (Studenti e Docenti)
• GET /api/v1/catalog/search - Ricerca libri per titolo, autore o ISBN  
• GET /api/v1/catalog/available - Libri disponibili  

### Prenotazioni (Studenti e Docenti)
• POST /api/v1/reservations - Prenota un libro  
• GET /api/v1/reservations - Lista delle proprie prenotazioni  
• GET /api/v1/reservations/{id} - Dettagli di una prenotazione  
• DELETE /api/v1/reservations/{id} - Cancella una prenotazione attiva  

### Prestiti (Admin e Utente)
• POST /api/v1/loans - Registra un nuovo prestito  
• PUT /api/v1/loans/{id}/return - Restituzione di un libro  
• GET /api/v1/loans - Lista prestiti per utente (visibili all’utente stesso o da admin)  
• GET /api/v1/loans/{id} - Dettaglio di un prestito  
• GET /api/v1/loans/overdue - Prestiti in ritardo (solo admin)  


## Integrazione con Altri Microservizi

Il microservizio **Gestione Biblioteca** comunica con i seguenti microservizi:

### Gestione Utenti e Ruoli
• Recupera l’**ID esterno** e il **ruolo dell’utente** (STUDENT, TEACHER, ADMIN) per autorizzare le azioni.  
• Verifica i permessi prima di consentire prenotazioni o prestiti.  
• Consuma l’evento `user.deleted` per eliminare dati associati a un utente rimosso.

### Comunicazioni e Notifiche
• Invia **notifiche automatiche** su prenotazioni effettuate, prestiti in scadenza o in ritardo.  
• Utilizza eventi RabbitMQ per segnalare quando avvisare l’utente.

### Valutazione/Esami (opzionale)
• Collega determinati libri a esami o corsi specifici.  
• Utile per suggerire letture legate ai corsi frequentati.


## Eventi (RabbitMQ)

**Eventi pubblicati:**  
• `reservation.created`: una nuova prenotazione è stata registrata  
• `loan.started`: un nuovo prestito è iniziato  
• `loan.returned`: un libro è stato restituito  
• `loan.overdue`: un prestito è in ritardo (trigger automatico)  

**Eventi consumati:**  
• `user.deleted`: rimuove prenotazioni e prestiti associati a un utente disattivato  
• `course.ended`: può generare eventi per svuotare o aggiornare disponibilità  


## Sicurezza e Autorizzazioni
L’accesso alle API è basato su **ruoli utente** provenienti dal microservizio di autenticazione/autorizzazione:  
• `ROLE_ADMIN`: può gestire libri, prestiti e visualizzare tutto  
• `ROLE_TEACHER` e `ROLE_STUDENT`: possono cercare, prenotare e consultare i propri prestiti  


## Esempio di Flusso Utente
1. Uno studente accede al catalogo → cerca un libro → prenota.  
2. Un bibliotecario (admin) vede la prenotazione e genera un prestito.  
3. Alla restituzione, lo stesso bibliotecario aggiorna lo stato del prestito.  
4. Eventuali ritardi generano eventi e notifiche automatiche.
