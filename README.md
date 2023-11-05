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
&emsp; WHERE masa BETWEEN 15 AND 24<br />
UNION <br />
SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki;<br />
### |EXCEPT| Identyfikatory pudełek, które nigdy nie zostały zamówione,
- SELECT idpudelka FROM pudelka p<br />
EXCEPT <br />
SELECT idpudelka FROM artykuly a<br />
&emsp; JOIN zamowienia z ON z.idzamowienia = a.idzamowienia; <br />
### |EXCEPT| Masa mieści się w przedziale od 25 do 35 g, ale koszt produkcji nie mieści się w przedziale od 25 do 35 gr
- SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
&emsp; WHERE masa BETWEEN 25 AND 35<br />
EXCEPT<br />
SELECT idczekoladki, nazwa, masa, koszt FROM czekoladki<br />
&emsp; WHERE koszt BETWEEN 0.25 AND 0.35;<br />
### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości,
- SELECT idmeczu, SUM(h) AS gospodarze, SUM(a) AS goscie FROM (<br />
&emsp; SELECT idmeczu, unnest(gospodarze) AS h, unnest(goscie) AS a FROM siatkowka.statystyki<br />
) GROUP BY idmeczu<br />
ORDER BY idmeczu;<br />
 ### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości, dla meczów, które skończyły się po 5 setach i zwycięzca ostatniego seta zdobył w nim ponad 15 punktów,
 - SELECT idmeczu, (SELECT SUM(c) FROM unnest(gospodarze) c) AS gospodarze, <br />
(SELECT SUM(a) FROM unnest(goscie) a) AS goscie FROM siatkowka.statystyki <br />
WHERE gospodarze[5] IS NOT null AND gospodarze[5]>15 OR goscie[5]>15; <br />
### Identyfikator i wynik meczu w formacie x:y, np. 3:1 (wynik jest pojedynczą kolumną – napisem),
- SELECT idmeczu, SUM(CASE WHEN gospodarze>goscie THEN 1 ELSE 0 END) || ':' || SUM(CASE WHEN goscie>gospodarze THEN 1 ELSE 0 END) AS "x:y" FROM ( <br />
&emsp; SELECT idmeczu, UNNEST(gospodarze) AS gospodarze, UNNEST(goscie) AS goscie FROM siatkowka.statystyki <br />
) AS dane <br />
GROUP BY idmeczu <br />
ORDER BY idmeczu; <br />
### Identyfikator meczu, sumę punktów zdobytych przez gospodarzy i sumę punktów zdobytych przez gości, dla meczów, w których gospodarze zdobyli ponad 100 punktów,
- SELECT idmeczu, gospodarze, goscie FROM ( <br />
&emsp; SELECT idmeczu, (SELECT(SUM(c)) FROM UNNEST(gospodarze) c) AS gospodarze, (SELECT(SUM(d)) FROM UNNEST(goscie) d) AS goscie FROM siatkowka.statystyki <br />
) AS dane <br />
WHERE gospodarze > 100; <br />
###  Identyfikator meczu, liczbę punktów zdobytych przez gospodarzy w pierwszym secie, sumę punktów zdobytych w meczu przez gospodarzy, dla meczów, w których pierwiastek kwadratowy z liczby punktów zdobytych przez gospodarzy w pierwszym secie jest mniejszy niż logarytm o podstawie 2 z sumy punktów zdobytych w meczu przez gospodarzy
- SELECT idmeczu, Iset, gospodarze FROM ( <br />
&emsp; SELECT idmeczu, gospodarze[1] AS Iset, (SELECT(SUM(c)) FROM UNNEST(gospodarze) c) AS gospodarze <br />
&emsp; FROM siatkowka.statystyki <br />
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
- SELECT p.nazwa, p.opis, cz.nazwa, cz.opis FROM pudelka p <br />
JOIN zawartosc z ON z.idpudelka = p.idpudelka <br />
JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki; <br />
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
OIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
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
&emsp; SELECT idczekoladki, nazwa, koszt FROM czekoladki<br />
&emsp; ORDER BY koszt DESC<br />
) WHERE koszt>(SELECT koszt FROM czekoladki WHERE idczekoladki='d08');<br />
### Kto (nazwa klienta) złożył zamówienia na takie same czekoladki (pudełka) jak zamawiała Górka Alicja.
- WITH dane AS(<br />
&emsp; SELECT DISTINCT k.nazwa as imie, p.nazwa FROM pudelka p <br />
&emsp; JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
&emsp; JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
&emsp; JOIN klienci k ON z.idklienta=k.idklienta<br />
&emsp; WHERE k.nazwa='Górka Alicja'<br />
	)<br />
SELECT DISTINCT k.nazwa FROM pudelka p<br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta<br />
WHERE p.nazwa IN (SELECT nazwa FROM dane) AND k.nazwa!='Górka Alicja';<br />
### Kto (nazwa klienta, adres) złożył zamówienia na takie same czekoladki (pudełka) jak zamawiali klienci z Katowic.
- WITH dane AS(<br />
&emsp; SELECT DISTINCT k.nazwa as imie, p.nazwa FROM pudelka p<br />
&emsp; JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
&emsp; JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
&emsp; JOIN klienci k ON z.idklienta=k.idklienta;<br />
&emsp; WHERE k.miejscowosc='Katowice'<br />
	)<br />
SELECT DISTINCT k.nazwa, k.ulica, k.miejscowosc FROM pudelka p<br />
JOIN artykuly a ON a.idpudelka=p.idpudelka<br />
JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
JOIN klienci k ON z.idklienta=k.idklienta<br />
WHERE p.nazwa IN (SELECT nazwa FROM dane) AND k.miejscowosc!='Katowice';<br />
# LAB5
### Pudełka, w którym jest najwięcej czekoladek (uwaga: konieczne jest użycie LIMIT)
- SELECT idpudelka, COUNT(idczekoladki) FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br /?
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka<br />
ORDER BY count DESC<br />
LIMIT 1;<br/>
### Łącznej liczby czekoladek w poszczególnych pudełkach,
- SELECT idpudelka, SUM(sztuk) AS ilosc FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, z.sztuk FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka<br />
ORDER BY idpudelka;<br />
### Łącznej liczby czekoladek bez orzechów w poszczególnych pudełkach,
- SELECT idpudelka, SUM(sztuk) AS ilosc FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, cz.orzechy, z.sztuk FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
&emsp; WHERE cz.orzechy IS NULL<br />
) GROUP BY idpudelka;<br />
### Łącznej liczby czekoladek w mlecznej czekoladzie w poszczególnych pudełkach.
- SELECT idpudelka, SUM(sztuk) AS ilosc FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, cz.czekolada, z.sztuk FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
&emsp; WHERE cz.czekolada = 'mleczna'<br />
) GROUP BY idpudelka;<br />
### Masy poszczególnych pudełek,
- SELECT idpudelka, SUM(sztuk*masa) FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, z.sztuk, cz.masa FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka <br />
ORDER BY idpudelka;<br />
### Pudełka o największej masie,
- SELECT idpudelka, SUM(sztuk*masa) AS masa FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, z.sztuk, cz.masa FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka <br />
ORDER BY masa DESC<br />
LIMIT 1;<br />
### Średniej masy pudełka w ofercie cukierni,
- SELECT AVG(masa) FROM (<br />
&emsp; SELECT idpudelka, SUM(sztuk*masa) AS masa FROM (<br />
&emsp; &emsp; SELECT p.idpudelka, cz.idczekoladki, z.sztuk, cz.masa FROM pudelka p<br />
&emsp; &emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; &emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka<br />
)<br />
### Średniej wagi pojedynczej czekoladki w poszczególnych pudełkach,
- SELECT idpudelka, SUM(sztuk*masa)/SUM(sztuk)::float AS masa FROM (<br />
&emsp; SELECT p.idpudelka, cz.idczekoladki, z.sztuk, cz.masa FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka;<br />
### Liczby zamówień na poszczególne dni,
- SELECT datarealizacji, COUNT(idzamowienia) FROM zamowienia<br />
GROUP BY datarealizacji<br />
ORDER BY datarealizacji;<br />
### Łącznej liczby wszystkich zamówień,
- SELECT COUNT(idzamowienia) FROM zamowienia <br />
### Łącznej wartości wszystkich zamówień,
- SELECT SUM(zakupy) FROM (<br />
&emsp;SELECT z.idzamowienia, p.idpudelka, a.sztuk, a.sztuk * p.cena AS zakupy FROM klienci k<br />
&emsp;JOIN zamowienia Z on z.idklienta=k.idklienta<br />
&emsp;JOIN artykuly a ON a.idzamowienia=z.idzamowienia<br />
&emsp;JOIN pudelka p ON p.idpudelka=a.idpudelka<br />
) <br />

### Klientów, liczby złożonych przez nich zamówień i łącznej wartości złożonych przez nich zamówień.
- SELECT nazwa, COUNT(nazwa) AS ilosc_zamowien, SUM(sztuk*cena) AS zakupy FROM (<br />
&emsp; SELECT k.nazwa, z.idzamowienia, a.sztuk, p.cena FROM klienci k<br />
&emsp; JOIN zamowienia z ON z.idklienta=k.idklienta<br />
&emsp; JOIN artykuly a ON a.idzamowienia=z.idzamowienia<br />
&emsp; JOIN pudelka p ON p.idpudelka=a.idpudelka<br />
) GROUP BY nazwa<br />
ORDER BY nazwa;<br />
### Czekoladki, która występuje w największej liczbie pudełek,
- SELECT idczekoladki, COUNT(idczekoladki) AS ilosc FROM(<br />
&emsp; SELECT DISTINCT a.idpudelka, cz.idczekoladki FROM czekoladki cz<br />
&emsp; JOIN zawartosc s ON s.idczekoladki=cz.idczekoladki<br />
&emsp; JOIN artykuly a ON a.idpudelka=s.idpudelka<br />
&emsp; JOIN zamowienia z ON z.idzamowienia=a.idzamowienia<br />
) GROUP BY idczekoladki<br />
ORDER BY ilosc DESC<br />
LIMIT 1;<br />
### Pudełka, które zawiera najwięcej czekoladek bez orzechów,
- WITH DANE as (<br />
&emsp; SELECT p.nazwa, SUM(z.sztuk) AS ilosc FROM pudelka p<br />
&emsp; JOIN zawartosc z ON z.idpudelka=p.idpudelka<br />
&emsp; JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
&emsp; WHERE cz.orzechy IS NULL<br />
&emsp; GROUP BY p.nazwa<br />
)<br />
SELECT nazwa, ilosc FROM dane <br />
WHERE ilosc=(SELECT MAX(ilosc) FROM dane);<br />
### Czekoladki, która występuje w najmniejszej liczbie pudełek,
- WITH dane AS (<br />
&emsp; SELECT s.idczekoladki AS nazwa, COUNT(s.idczekoladki) AS ilosc FROM zawartosc s<br />
&emsp; GROUP BY nazwa<br />
) <br />
SELECT nazwa FROM dane <br />
WHERE ilosc = (SELECT MIN(ilosc) FROM dane);<br />

LUB 

- SELECT cz.idczekoladki FROM czekoladki cz<br />
EXCEPT <br />
SELECT s.idczekoladki FROM zawartosc s;<br />
### Pudełko, które jest najczęściej zamawiane przez klientów.
- WITH DANE AS (<br />
&emsp; SELECT a.idpudelka AS nazwa, COUNT(a.idpudelka) AS ilosc FROM artykuly a<br />
&emsp; GROUP BY nazwa<br />
)<br />
SELECT nazwa FROM dane<br />
WHERE ilosc = (SELECT MAX(ilosc) FROM dane);<br />
### Liczby zamówień na poszczególne kwartały,
- SELECT kwartał, COUNT(idzamowienia) FROM (<br />
&emsp; SELECT DATE_PART('quarter', datarealizacji) AS kwartał, idzamowienia FROM zamowienia<br />
)GROUP BY kwartał;<br />
### Liczby zamówień na poszczególne miesiące,
- SELECT month, COUNT(idzamowienia) FROM (<br />
SELECT DATE_PART('month', datarealizacji) AS month , idzamowienia FROM zamowienia<br />
) GROUP BY month;<br />
### Liczby zamówień do realizacji w poszczególnych tygodniach,
- SELECT week, COUNT(idzamowienia) FROM (<br />
&emsp; SELECT DATE_PART('week', datarealizacji) AS week, idzamowienia FROM zamowienia<br />
&emsp; ORDER BY week<br />
) GROUP BY week;<br />
### Liczby zamówień do realizacji w poszczególnych miejscowościach.
- SELECT miejscowosc, COUNT(miejscowosc) FROM (<br />
&emsp;SELECT k.miejscowosc, z.idzamowienia FROM klienci k<br />
&emsp;JOIN zamowienia z ON z.idklienta=k.idklienta<br />
)GROUP BY miejscowosc<br />
ORDER BY miejscowosc;<br />
### Łącznej masy wszystkich pudełek czekoladek znajdujących się w cukierni,
- SELECT SUM(masa) FROM (<br />
&emsp;SELECT p.idpudelka, cz.idczekoladki, cz.masa * z.sztuk AS masa FROM zawartosc z<br />
&emsp;JOIN pudelka p ON p.idpudelka=z.idpudelka<br />
&emsp;JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
);<br />
### Łącznej wartości wszystkich pudełek czekoladek znajdujących się w cukierni.
- SELECT SUM(cena) FROM (<br />
&emsp;SELECT idpudelka, cena FROM pudelka<br />
);<br />
### Zysk ze sprzedaży jednej sztuki poszczególnych pudełek (różnica między ceną pudełka i kosztem jego wytworzenia),
- SELECT idpudelka, cena-SUM(koszt) AS zysk FROM (<br />
&emsp;SELECT p.idpudelka, p.cena, cz.idczekoladki, cz.koszt*z.sztuk AS koszt FROM zawartosc z<br />
&emsp;JOIN pudelka p ON p.idpudelka=z.idpudelka<br />
&emsp;JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka, cena<br />
ORDER BY idpudelka;<br />
### Zysk ze sprzedaży zamówionych pudełek,
- WITH dane AS (<br />
&emsp;SELECT idpudelka, cena-SUM(koszt) AS zysk FROM (<br />
&emsp;&emsp;SELECT p.idpudelka, p.cena, cz.koszt * z.sztuk AS koszt FROM zawartosc z<br />
&emsp;&emsp;JOIN pudelka p ON p.idpudelka=z.idpudelka<br />
&emsp;&emsp;JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka, cena<br />
ORDER BY idpudelka<br />
)<br />
SELECT idpudelka, SUM(zakup) AS zysk FROM (<br />
&emsp;SELECT idzamowienia, a.idpudelka, sztuk * zysk AS zakup FROM artykuly a<br />
&emsp;JOIN dane ON a.idpudelka=dane.idpudelka<br />
) GROUP BY idpudelka<br />
ORDER BY idpudelka;<br />
### Zysk ze sprzedaży wszystkich pudełek czekoladek w cukierni.
- WITH dane AS (<br />
&emsp;SELECT idpudelka, cena-SUM(koszt) AS zysk FROM (<br />
&emsp;&emsp;SELECT p.idpudelka, p.cena, cz.idczekoladki, cz.koszt*z.sztuk AS koszt FROM zawartosc z<br />
&emsp;&emsp;JOIN pudelka p ON p.idpudelka=z.idpudelka<br />
&emsp;&emsp;JOIN czekoladki cz ON cz.idczekoladki=z.idczekoladki<br />
) GROUP BY idpudelka, cena<br />
ORDER BY idpudelka<br />
)<br />
SELECT SUM(zakup) FROM (<br />
&emsp;SELECT a.idzamowienia, a.sztuk * d.zysk AS zakup FROM artykuly a<br />
&emsp;JOIN dane d ON d.idpudelka=a.idpudelka<br />
);<br />
### Liczbę porządkową i identyfikator pudełka czekoladek (idpudelka).
- SELECT SUM(sztuk), idpudelka FROM(<br />
&emsp;SELECT * FROM artykuly a<br />
)GROUP BY idpudelka<br />
ORDER BY idpudelka;<br />
