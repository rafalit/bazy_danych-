# LAB2
	
### Masa mieści się w przedziale od 25 do 35 g, ale koszt produkcji nie mieści się w przedziale od 25 do 35 gr,
- SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
WHERE masa BETWEEN 25 AND 35 AND koszt NOT BETWEEN 0.25 AND 0.35;
### Masa mieści się w przedziale od 25 do 35 g, ale koszt produkcji nie mieści się ani w przedziale od 15 do 24 gr, ani w przedziale od 25 do 35 gr.
- SELECT idczekoladki, nazwa, masa, koszt*100 || ' gr' AS koszt FROM czekoladki <br />
WHERE masa BETWEEN 25 AND 35 AND koszt NOT BETWEEN 0.15 AND 0.24 AND koszt NOT BETWEEN 0.25 AND 0.35;

# LAB3
### W listopadzie 2013 (w tym i kolejnych zapytaniach użyj funkcji date_part lub extract),
- SELECT idzamowienia, datarealizacji FROM zamowienia<br />
WHERE DATE_PART('month', datarealizacji) ='11' AND DATE_PART('year', datarealizacji)='2013';
### 17, 18 lub 19 dnia miesiąca,
- SELECT idzamowienia, datarealizacji FROM zamowienia <br />
WHERE DATE_PART('day', datarealizacji) BETWEEN 17 AND 19;
### Nazwa nie rozpoczyna się żadną z liter: 'D'-'K', 'S' i 'T',
- SELECT idczekoladki, nazwa, czekolada, orzechy, nadzienie FROM czekoladki<br />
WHERE nazwa > 'E' AND nazwa< 'K%' AND nazwa NOT LIKE 'S%' AND nazwa NOT LIKE 'T%';
### Wyświetla nazwy i numery telefonów klientów, którzy podali numer stacjonarny telefonu (np. 012 222 22 00),
- SELECT nazwa, telefon FROM klienci <br />
WHERE LENGTH(telefon::text)>'11';
### |UNION| Masa mieści się w przedziale od 15 do 24 g lub koszt produkcji mieści się w przedziale od 15 do 24 gr,
- SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
WHERE masa BETWEEN 15 AND 24<br />
UNION <br />
SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki;<br />
### |EXCEPT| Identyfikatory pudełek, które nigdy nie zostały zamówione,
- SELECT idpudelka FROM pudelka p<br />
EXCEPT <br />
SELECT idpudelka FROM artykuly a<br />
JOIN zamowienia z ON z.idzamowienia = a.idzamowienia; <br />
### |EXCEPT| Masa mieści się w przedziale od 25 do 35 g, ale koszt produkcji nie mieści się w przedziale od 25 do 35 gr
- SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
WHERE masa BETWEEN 25 AND 35<br />
EXCEPT<br />
SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
WHERE koszt BETWEEN 0.25 AND 0.35;<br />
### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości,
- SELECT idmeczu, SUM(h) AS gospodarze, SUM(a) AS goscie FROM (<br />
SELECT idmeczu, unnest(gospodarze) AS h, unnest(goscie) AS a FROM siatkowka.statystyki<br />
	) GROUP BY idmeczu<br />
	ORDER BY idmeczu;<br />
 ### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości, dla meczów, które skończyły się po 5 setach i zwycięzca ostatniego seta zdobył w nim ponad 15 punktów,
 - SELECT idmeczu, (SELECT SUM(c) FROM unnest(gospodarze) c) AS gospodarze, <br />
(SELECT SUM(a) FROM unnest(goscie) a) AS goscie FROM siatkowka.statystyki <br />
WHERE gospodarze[5] IS NOT null AND gospodarze[5]>15 OR goscie[5]>15; <br />
### Identyfikator i wynik meczu w formacie x:y, np. 3:1 (wynik jest pojedynczą kolumną – napisem),
- SELECT idmeczu, SUM(CASE WHEN gospodarze>goscie THEN 1 ELSE 0 END) || ':' || <br />
SUM(CASE WHEN goscie>gospodarze THEN 1 ELSE 0 END) AS "x:y" FROM ( <br />
SELECT idmeczu, UNNEST(gospodarze) AS gospodarze, UNNEST(goscie) AS goscie <br /> 
FROM siatkowka.statystyki <br />
	) AS dane <br />
	GROUP BY idmeczu <br />
ORDER BY idmeczu; <br />
### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości, dla meczów, w których gospodarze zdobyli ponad 100 punktów,
- SELECT idmeczu, gospodarze, goscie FROM ( <br />
SELECT idmeczu, (SELECT(SUM(c)) FROM UNNEST(gospodarze) c) AS gospodarze, <br /> 
(SELECT(SUM(d)) FROM UNNEST(goscie) d) AS goscie FROM siatkowka.statystyki <br />
) AS dane <br />
WHERE gospodarze > 100; <br />
###  Identyfikator meczu, liczbę punktów zdobytych przez gospodarzy w pierwszym secie, sumę punktów zdobytych w meczu przez gospodarzy, dla meczów, w których pierwiastek kwadratowy z liczby punktów zdobytych przez gospodarzy w pierwszym secie jest mniejszy niż logarytm o podstawie 2 z sumy punktów zdobytych w meczu przez gospodarzy
- SELECT idmeczu, Iset, gospodarze FROM ( <br />
SELECT idmeczu, gospodarze[1] AS Iset, (SELECT(SUM(c)) FROM UNNEST(gospodarze) c) AS gospodarze <br />
FROM siatkowka.statystyki <br />
) AS dane <br />
WHERE SQRT(Iset)<LOG(2, gospodarze); <br />
# LAB4
### Zamówili pudełko Kremowa fantazja lub Kolekcja jesienna,
- SELECT DISTINCT k.nazwa, k.ulica, k.miejscowosc FROM klienci k <br />
JOIN zamowienia z ON z.idklienta=k.idklienta <br />
JOIN artykuly a ON a.idzamowienia=z.idzamowienia <br />
JOIN pudelka p ON a.idpudelka=p.idpudelka <br />
WHERE p.nazwa IN ('Kremowa fantazja', 'Kolekcja jesienna'); <br />
### Zamówili co najmniej 2 sztuki pudełek Kremowa fantazja lub Kolekcja jesienna w ramach jednego zamówienia,
- SELECT DISTINCT k.nazwa, k.ulica, k.miejscowosc FROM klienci k<br />
JOIN zamowienia z ON z.idklienta=k.idklienta<br />
JOIN artykuly a ON a.idzamowienia=z.idzamowienia<br />
JOIN pudelka p ON p.idpudelka=a.idpudelka<br />
WHERE p.nazwa IN ('Kremowa fantazja', 'Kolekcja jesienna') AND sztuk>1;<br />
### Zamówili pudełka, które zawierają czekoladki z migdałami.
- SELECT DISTINCT k.nazwa, k.ulica, k.miejscowosc FROM klienci k,br />
JOIN zamowienia z ON z.idklienta=k.idklienta<br />
JOIN artykuly a ON a.idzamowienia=z.idzamowienia<br />
JOIN zawartosc s ON s.idpudelka=a.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=s.idczekoladki<br />
WHERE cz.orzechy LIKE 'migdały';<br />
### Informacje na temat wszystkich pudełek,
- SELECT p.nazwa, p.opis, cz.nazwa, cz.opis FROM pudelka p
JOIN zawartosc z ON z.idpudelka = p.idpudelka
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki;
### Informacje na temat pudełek, których nazwa zawiera słowo Kolekcja.
- SELECT p.nazwa, p.opis, cz.nazwa, cz.opis FROM pudelka p <br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki <br />
WHERE p.nazwa LIKE 'Kolekcja %';<br />
### Zawierają czekoladki o wartości klucza głównego d09
- SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.idczekoladki = 'd09';<br />
### Zawierają przynajmniej jedną czekoladkę, której nazwa zaczyna się na S,
- SELECT DISTINCT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.nazwa LIKE 'S%';<br />
### Zawierają przynajmniej 4 sztuki czekoladek jednego gatunku (o takim samym kluczu głównym),
- SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE z.sztuk>=4;<br />
### Zawierają czekoladki z nadzieniem truskawkowym,
- SELECT DISTINCT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.nadzienie='truskawki';<br />
### Nie zawierają czekoladek w gorzkiej czekoladzie,
- SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
EXCEPT <br />
SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.czekolada = 'gorzka';<br />
### Zawierają co najmniej 3 sztuki czekoladki Gorzka truskawkowa,
- SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.nazwa='Gorzka truskawkowa' AND z.sztuk>=3;<br />
### Nie zawierają czekoladek z orzechami,
- SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
EXCEPT <br />
SELECT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.orzechy IS NOT NULL; <br />
### Zawierają przynajmniej jedną czekoladkę bez nadzienia.
- SELECT DISTINCT p.nazwa, p.opis, p.cena FROM pudelka p<br />
JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
WHERE cz.nadzienie IS NULL;<br />
### Wyświetl wartości kluczy głównych oraz nazwy czekoladek, których koszt jest większy od kosztu czekoladki o wartości klucza głównego równej d08.
- SELECT idczekoladki, nazwa FROM (<br />
SELECT idczekoladki, nazwa, koszt FROM czekoladki<br />
ORDER BY koszt DESC<br />
) WHERE koszt>(SELECT koszt FROM czekoladki WHERE idczekoladki='d08');<br />
### Kto (nazwa klienta) złożył zamówienia na takie same czekoladki (pudełka) jak zamawiała Górka Alicja.
- WITH dane AS(<br />
SELECT DISTINCT k.nazwa as imie, p.nazwa FROM pudelka p <br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta<br />
WHERE k.nazwa='Górka Alicja'<br />
	)<br />
SELECT DISTINCT k.nazwa FROM pudelka p<br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta<br />
WHERE p.nazwa IN (SELECT nazwa FROM dane) AND k.nazwa!='Górka Alicja';<br />
### Kto (nazwa klienta, adres) złożył zamówienia na takie same czekoladki (pudełka) jak zamawiali klienci z Katowic.
WITH dane AS(<br />
SELECT DISTINCT k.nazwa as imie, p.nazwa FROM pudelka p<br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta;<br />
WHERE k.miejscowosc='Katowice'<br />
	)<br />
SELECT DISTINCT k.nazwa, k.ulica, k.miejscowosc FROM pudelka p<br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta<br />
WHERE p.nazwa IN (SELECT nazwa FROM dane) AND k.miejscowosc!='Katowice'<br />
