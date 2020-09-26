{% include champions-league-article-list.md %}

## Gdzie trzymać dane?
Tematem dzisiejszego odcinka są bazy danych. Tak jak poprzednio, zamiast gotowego rozwiązania skupię się bardziej na możliwościach, a nie konkretnym wyborze.
Ponieważ aplikacja dla Ligii Mistrzów jest projektem typu greenfield to całe szczęście nie musimy nic migrować, a jedynie zastanowić się, gdzie chcielibyśmy mieć dane. Jest oczywiście jeden problem w [specyfikacji](https://poznajazure.pl/champions-league/).

Mamy tam 3 dość istotne punkty: 
- Ważnym wydarzeniem będzie losowanie na żywo i sprawdzanie wyników przez uczestników
- Mogą być różne sposoby zdobycia biletów: quizy, losowania, inne - to się jeszcze dogada 🤦‍♂️
- Aplikacja musi działać już za miesiąc, ma obsłużyć tegoroczne finały i może przyszłe

Czyli mówiąc wprost:
- Wydajność bazy musi być zmienna w czasie
- Nie do końca wiemy co w niej będzie :)
- Spora część danych będzie do odczytu i cache powinien sprawować się nieźle (są wyniki, albo nie ma wyników)
Jak do tego podejść? Od bardzo dawna mamy dwie szkoły baz danych SQL i NoSQL. Część z Was może zakrzyknąć: jak to? Od dawna? Przecież NoSQL to ten wiek. Niby tak, niby nie. Sam termin został użyty przez Carlo Strozziego w 1998 roku jako nazwa dla lekkiej relacyjnej bazy open source Strozzi NoSQL. Tutaj zaczynamy trochę wchodzić w filozofię i zastanawiać się, która baza jest jaka. I czy termin NoSQL, nie powinien zostać przekształcony w NoREL (od “not relational”), ale to na pewno już dyskusja nie na ten email. Jedyna uwaga na koniec: bazy nierelacyjne mają swoje korzenie już w latach 60 XX wieku, więc ciężko je uznać za coś nowego :)



## Opcje...
A jak to wygląda w Azure? Możemy zdecydować się na 4 opcje, acz sprowadzają się do dwóch powyższych:
- Postawić coś swojego na VM’kach. Da się? No ba!
- Użyć Cosmos DB. Dzięki temu mamy od razu bazę “global scale” z kilkoma różnymi API dostępowymi, czyli: SQL (format natywny), MongoDB, Cassanda, Gremlin, key-value czy etcd. To ostatnie jest jeszcze w private preview.
- Użyć SQL. I tu znowu cała gama rozwiązań, czyli: Azure SQL (prawie identyczny z SQL Server 2019), PostgreSQL, MySQL czy MariaDB
- Zwykły Azure Storage o ile wystarczy nam plikowa baza danych, lub prosty key-value.
No dobra to kilka przemyśleń na każdy z powyższych punktów

## Postawić coś swojego na VM’kach. Da się? No ba!
Własna baza to niekoniecznie jest zły pomysł, ale trzeba spełnić 2 warunki. Oba, a nie tylko jeden z nich💡. Po pierwsze wiedzieć, dlaczego wybrana usługa PaaS nie spełni naszych wymogów. Znam kilku specjalistów, którzy potrafią SQL Server wyciągnąć więcej na bardzo konkretnych VM. Robią wtedy bardzo konkretne zastosowania obliczeniowe.
Po drugie musimy umieć taką bazą zarządzać zarówno pod względem backup jak i security. I teraz bardzo ważny punkt: mieć specjalistów do tego na pokładzie. Jeżeli decydujemy się na taki krok, to poszukiwanie nagle na rynku specjalisty jest strzałem w kolano, bo dużo ich nie ma. Przy okazji jak doszliśmy tak daleko, to jak przeszliśmy przez punkt pierwszy 🤪.
Jeżeli chodzi o projekt “Liga Mistrzów”, to wątpię abyśmy spełniali pierwszy warunek, więc całe szczęście nad drugim nie musimy rozmyślać. Czas nagli, szkoda energii.



## Zwykły Azure Storage o ile wystarczy nam plikowa baza danych, lub prosty key-value
Zwykły Azure Storage też prawdopodobnie nie spełni naszych wymagań, bo wymaga dużo czasu na jej projektowanie, tak by nasza aplikacja potrafiła działać tylko na zapytaniach po kluczu. Jest też limit na maksymalną liczbę operacji, więc skalowanie jest utrudnione. Szczególnie, że po osiągnięciu maksa, jedynym sposobem jest migracja na Cosmos DB. Nie da się dorzucić po prostu dolarów do pieca.

W tym miejscu może zaświtać Ci pytanie: po co omawiamy tą opcję? Najlepszą odpowiedzią jest wpis na blogu Troy Hunt, który [https://haveibeenpwned.com/](https://haveibeenpwned.com/) oparł na Azure Storage. Dzięki temu jak sam pisze: In other words, if I want 100GB of storage and I want to hit it 10 million times, it’ll cost me $8 a month. Nic innego w Azure nie da Wam takiej ilości danych za tak małe pieniądze. Polecam zerknąć na artykuł: [Working with 154 million records on Azure Table Storage](https://www.troyhunt.com/working-with-154-million-records-on/), ale tak jak napisałem wyżej wymaga to bardzo dobrego zaprojektowania naszej aplikacji.

## Użyć Cosmos DB. Dzięki temu mamy od razu bazę “global scale” z kilkoma różnymi API dostępowymi, czyli: SQL (format natywny), MongoDB, Cassanda, Gremlin, key-value czy etcd. To ostatnie jest jeszcze w private preview.
Cosmos jest TĄ bazą, w którą większość programistów chce wejść, gdy tylko zaczynają zabawę z Azure. Kilka różnych API, szybkie skalowanie, wiele możliwości. Ochów i achów nie ma końca. Bądźmy jednak szczerzy, nie jest jednak ona pozbawiona wad.

Po pierwsze przy braku połączenia wiedzy z doświadczeniem, łatwo zapłacić naprawdę grube pieniądze. Mamy tu idealny przykład zależności na ile można wycenić doświadczenie. Konsekwencją jest brak używania Cosmos w wielu organizacjach, gdyż wszyscy boją się kosztów. (Uwaga prywatna: Ponieważ ja mam już to doświadczenie, a firmom zdarzyło się zapłacić za moją naukę to jestem w stanie pokazać, że potrafi być ona tańsza niż Azure SQL).

Drugą wadą Cosmosa jest brak “prostego” backup. Nie da się zrobić kopii bezpieczeństwa, bez własnego lub znalezionego na GitHub narzędzia. Niedługo całe szczęście ta wada powinna zniknąć, gdyż w tej chwili trwają pracę nad “point in time restore”, ale są one w statusie “in development” ([więcej](https://azure.microsoft.com/en-us/updates/pointintimerestore-pitr-for-azure-cosmos-db/)). Czy Twoja organizacja jest gotowa żyć bez opcji backup? Nie wiem 😁.

Ostatnią sprawą jest praca na lokalnej maszynie. Jedyny emulator da się uruchomić Windows, albo w Windows Container, co sprowadza się do tego samego. Czyli na Linux i Mac nie szans pracy przez połączenia sieciowego. Oczywiście gdy zdecydujemy się na na przykład na API MongoDB w wersji Cosmos to lokalnie możemy pracować na lokalnej wersji Mongo. Podobnie będzie z Casandra. Dla natywnego formatu nie ma już niestety takiej opcji.

## Użyć SQL. I tu znowu cała gama rozwiązań, czyli: Azure SQL (prawie identyczny z SQL Server 2019), PostgreSQL, MySQL czy MariaDB
Ostatnia opcja to Azure Database w różnych odmianach. Oczywiście najbardziej “wspieranym” jest Azure SQL, ale pozostałe też dają radę, więc zależy to od umiejętności w zespole. Jest to klasyczny SQL, więc mamy od razu ORMy i wsparcie narzędziowe. Do tego łatwo postawić sobie odpowiednik na laptopie i pracować nawet w pociągu. Jest też backup.

Dodatkowym plusem jest zarządzanie przez DTU, czyli dajemy maksymalną moc naszej bazie. Należy zastanowić się jak dużo nam tej mocy potrzeba. Na przykład portal [https://dotnetomaniak.pl](dotnetomaniak.pl) chodzi z dużym zapasem na 10 DTU (około €8) i gdyby nie wdrożenia to mógłby nawet na 5 DTU (około €4). Przy projektowaniu nowego systemu opcja idealna, ponieważ będziemy musieli tak projektować aplikację, żeby się zmieścić w limitach.

Są też oczywiście wady, które wynikają z niedoskonałości ORM i budowaniu czasami zapytań bardzo drogich. Trzeba więc uważać. Do tego skalowanie nie jest “instant”, potrafi ono trwać naprawdę długie minuty. Czasem szybciej, czasem wolniej, ale wiadomo że gorzej będzie wtedy kiedy najbardziej będziemy tego potrzebować. To nieoficjalne prawo Murphy'ego.

Jeszcze jedna uwaga o Azure SQL. Są dwa modele opłat: vCore i DTU. Jak wybierzesz vCore to możesz się przestraszyć. Wolę o tym uprzedzić z góry, bo już kilka razu się spotkałem z opinią, że SQL w Azure to kosztuje krocie 😁

## Na koniec...
Czas na podsumowanie. Jak zwykle opisałem analizę i nie przedstawiłem jasnej odpowiedzi. Decyzja należy do Ciebie, bo tabliczka na drzwiach “Azure Ninja” zobowiązuje.

Jeżeli temat Cię zainteresował i chcesz wiedzieć co będzie dalej to dołącz do nas na używając poniższego formularza:

{% include mail-banner.html %}
