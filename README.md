-- ==========================================
-- ZADANIE 3: Tworzenie tabel
-- ==========================================

-- Tworzenie tabeli producent
CREATE TABLE producent (
    id_producenta INT AUTO_INCREMENT PRIMARY KEY,
    nazwa VARCHAR(50) NOT NULL,
    miasto VARCHAR(100) NOT NULL,
    kod_pocztowy VARCHAR(6) NOT NULL,
    ulica VARCHAR(255) NOT NULL
);

-- Tworzenie tabeli pojazd
CREATE TABLE pojazd (
    id_pojazdu INT NOT NULL,
    model VARCHAR(100) NOT NULL,
    typ_pojazdu ENUM('osobowy', 'ciężarowy', 'autobus') DEFAULT 'osobowy',
    nadwozie VARCHAR(50),
    producent INT,
    cena_producenta DECIMAL(8, 2), -- 8 cyfr łącznie, 2 po przecinku
    PRIMARY KEY (id_pojazdu),
    FOREIGN KEY (producent) REFERENCES producent(id_producenta) ON DELETE CASCADE
);

-- ==========================================
-- ZADANIE 4: Wstawianie danych
-- ==========================================

-- Wstawienie 2 producentów
INSERT INTO producent (nazwa, miasto, kod_pocztowy, ulica) VALUES 
('Toyota', 'Tokio', '00-001', 'Główna 1'),
('BMW', 'Monachium', '80-809', 'Olimpijska 5');

-- Wstawienie pojazdów (po 2 dla każdego producenta, łącznie 4)
-- Zakładamy, że Toyota ma id=1, a BMW id=2
INSERT INTO pojazd (id_pojazdu, model, typ_pojazdu, nadwozie, producent, cena_producenta) VALUES 
(1, 'Corolla', 'osobowy', 'sedan', 1, 95000.00),
(2, 'Yaris', 'osobowy', 'hatchback', 1, 75000.00),
(3, 'X5', 'osobowy', 'SUV', 2, 350000.00),
(4, 'Seria 3', 'osobowy', 'sedan', 2, 200000.00);

-- ==========================================
-- ZADANIE 5: Zmiana klucza głównego
-- ==========================================

-- Najpierw usuwamy stary klucz główny
ALTER TABLE pojazd DROP PRIMARY KEY;

-- Dodajemy nowy klucz złożony (model + producent)
ALTER TABLE pojazd ADD PRIMARY KEY (model, producent);

-- ==========================================
-- ZADANIE 6: Wartość domyślna
-- ==========================================

ALTER TABLE pojazd ALTER COLUMN cena_producenta SET DEFAULT 100000.00;


-- ==========================================
-- ZADANIE 7: Zapytania do bazy "zawody"
-- (Zakładam nazwy tabel: zawodnicy, zawody, starty/wyniki)
-- ==========================================

-- 7a. Wyświetl 3 najlepszych zawodników (łączny czas, maj)
SELECT z.imie, z.nazwisko, SUM(w.czas) AS laczny_czas
FROM zawodnicy z
JOIN wyniki w ON z.id_zawodnika = w.id_zawodnika
JOIN zawody za ON w.id_zawodow = za.id_zawodow
WHERE MONTH(za.data) = 5
GROUP BY z.id_zawodnika, z.imie, z.nazwisko
ORDER BY laczny_czas ASC
LIMIT 3;

-- 7b. Nazwy miejsc, miast, krajów dla zawodów, które się nie odbyły
-- (Zakładam, że "nie odbyły się" oznacza brak wyników lub flagę statusu. Używam LEFT JOIN sprawdzając NULL)
SELECT za.nazwa, za.miasto, za.kraj
FROM zawody za
LEFT JOIN wyniki w ON za.id_zawodow = w.id_zawodow
WHERE w.id_zawodnika IS NULL;

-- 7c. Liczba zawodników w poszczególnych zawodach (0,5 pkt)
SELECT za.nazwa, COUNT(DISTINCT w.id_zawodnika) AS liczba_zawodnikow
FROM zawody za
JOIN wyniki w ON za.id_zawodow = w.id_zawodow
GROUP BY za.nazwa;

-- 7d. Pierwsza litera kraju i lista krajów (GROUP_CONCAT)
SELECT 
    SUBSTR(kraj, 1, 1) AS litera, 
    GROUP_CONCAT(DISTINCT kraj ORDER BY kraj SEPARATOR ', ') AS lista_krajow
FROM zawody -- lub tabela 'kraje' lub 'zawodnicy', zależnie gdzie jest kolumna 'kraj'
GROUP BY SUBSTR(kraj, 1, 1);

-- 7e. Nazwa klubu i średni czas z pływania, biegu i roweru
SELECT 
    z.klub, 
    w.konkurencja, -- np. 'plywanie', 'bieg', 'rower'
    AVG(w.czas) AS sredni_czas
FROM zawodnicy z
JOIN wyniki w ON z.id_zawodnika = w.id_zawodnika
WHERE w.konkurencja IN ('plywanie', 'bieg', 'rower')
GROUP BY z.klub, w.konkurencja;

-- 7f. Imię i nazwisko zawodnika oraz ilość startów (> 5)
SELECT z.imie, z.nazwisko, COUNT(*) AS ilosc_startow
FROM zawodnicy z
JOIN wyniki w ON z.id_zawodnika = w.id_zawodnika
GROUP BY z.id_zawodnika, z.imie, z.nazwisko
HAVING ilosc_startow > 5;
