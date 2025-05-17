-- DROP & CREATE DATABASE
DROP DATABASE IF EXISTS MTBMS;
CREATE DATABASE MTBMS;
USE MTBMS;

-- USERS TABLE
CREATE TABLE Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- MOVIES TABLE
CREATE TABLE Movies (
    movie_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(100),
    genre VARCHAR(50),
    duration INT
);

-- THEATERS TABLE
CREATE TABLE Theaters (
    theater_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    location VARCHAR(100)
);

-- SHOWS TABLE
CREATE TABLE Shows (
    show_id INT AUTO_INCREMENT PRIMARY KEY,
    movie_id INT,
    theater_id INT,
    show_time DATETIME,
    FOREIGN KEY (movie_id) REFERENCES Movies(movie_id),
    FOREIGN KEY (theater_id) REFERENCES Theaters(theater_id)
);

-- SEATS TABLE
CREATE TABLE Seats (
    seat_id INT AUTO_INCREMENT PRIMARY KEY,
    show_id INT,
    seat_number VARCHAR(10),
    status VARCHAR(20),
    FOREIGN KEY (show_id) REFERENCES Shows(show_id)
);

-- BOOKINGS TABLE
CREATE TABLE Bookings (
    booking_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    show_id INT,
    seats_booked INT,
    booking_time DATETIME,
    FOREIGN KEY (user_id) REFERENCES Users(user_id),
    FOREIGN KEY (show_id) REFERENCES Shows(show_id)
);

-- PAYMENTS TABLE
CREATE TABLE Payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    booking_id INT,
    payment_method VARCHAR(50),
    amount DECIMAL(10,2),
    payment_time DATETIME,
    FOREIGN KEY (booking_id) REFERENCES Bookings(booking_id)
);

-- SAMPLE INSERTS
INSERT INTO Users (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

INSERT INTO Movies (title, genre, duration) VALUES
('Inception', 'Sci-Fi', 148),
('Titanic', 'Romance', 195);

INSERT INTO Theaters (name, location) VALUES
('PVR Cinemas', 'Chennai'),
('INOX', 'Bangalore');

INSERT INTO Shows (movie_id, theater_id, show_time) VALUES
(1, 1, '2025-04-21 18:00:00'),
(2, 2, '2025-04-21 20:00:00');

-- 10 seats per show
INSERT INTO Seats (show_id, seat_number, status)
SELECT 1, CONCAT('A', LPAD(num, 2, '0')), 'Available'
FROM (SELECT @row := @row + 1 AS num FROM
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION
     SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
     (SELECT @row := 0) r) seat_numbers;

INSERT INTO Seats (show_id, seat_number, status)
SELECT 2, CONCAT('B', LPAD(num, 2, '0')), 'Available'
FROM (SELECT @row := @row + 1 AS num FROM
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION
     SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
     (SELECT @row := 0) r) seat_numbers;

-- VIEW: Available_Seats
CREATE OR REPLACE VIEW Available_Seats AS
SELECT s.show_id, m.title AS movie, t.name AS theater, COUNT(*) AS seats_left
FROM Seats s
JOIN Shows sh ON s.show_id = sh.show_id
JOIN Movies m ON sh.movie_id = m.movie_id
JOIN Theaters t ON sh.theater_id = t.theater_id
WHERE s.status = 'Available'
GROUP BY s.show_id;

-- VIEW: Booking_Summary
CREATE OR REPLACE VIEW Booking_Summary AS
SELECT b.booking_id, u.name AS user, m.title AS movie, b.seats_booked, p.amount
FROM Bookings b
JOIN Users u ON b.user_id = u.user_id
JOIN Shows s ON b.show_id = s.show_id
JOIN Movies m ON s.movie_id = m.movie_id
JOIN Payments p ON b.booking_id = p.booking_id;

-- TRIGGER: Prevent Overbooking
DELIMITER //
CREATE TRIGGER PreventOverbooking
BEFORE INSERT ON Bookings
FOR EACH ROW
BEGIN
    DECLARE available INT;
    SET available = (
        SELECT COUNT(*) FROM Seats
        WHERE show_id = NEW.show_id AND status = 'Available'
    );
    IF NEW.seats_booked > available THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Not enough available seats for this show.';
    END IF;
END;
//
DELIMITER ;

-- TRIGGER: Update Seat Status After Booking
DELIMITER //
CREATE TRIGGER UpdateSeatStatusAfterBooking
AFTER INSERT ON Bookings
FOR EACH ROW
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE seat_to_book INT;
    WHILE i < NEW.seats_booked DO
        SET seat_to_book = (
            SELECT seat_id FROM Seats
            WHERE show_id = NEW.show_id AND status = 'Available'
            LIMIT 1
        );
        IF seat_to_book IS NOT NULL THEN
            UPDATE Seats SET status = 'Booked' WHERE seat_id = seat_to_book;
        END IF;
        SET i = i + 1;
    END WHILE;
END;
//
DELIMITER ;

-- STORED PROCEDURE: GetUserBookings
DELIMITER //
CREATE PROCEDURE GetUserBookings(IN uid INT)
BEGIN
    SELECT u.name, m.title AS movie, s.show_time, b.seats_booked
    FROM Bookings b
    JOIN Users u ON b.user_id = u.user_id
    JOIN Shows s ON b.show_id = s.show_id
    JOIN Movies m ON s.movie_id = m.movie_id
    WHERE u.user_id = uid;
END;
//
DELIMITER ;

-- STORED PROCEDURE: GetAvailableSeatsForShow
DELIMITER //
CREATE PROCEDURE GetAvailableSeatsForShow(IN sid INT)
BEGIN
    SELECT seat_number
    FROM Seats
    WHERE show_id = sid AND status = 'Available';
END;
//
DELIMITER ;

-- Sample Booking
INSERT INTO Bookings (user_id, show_id, seats_booked, booking_time) VALUES
(1, 1, 3, NOW());

INSERT INTO Payments (booking_id, payment_method, amount, payment_time) VALUES
(1, 'Credit Card', 450.00, NOW());
