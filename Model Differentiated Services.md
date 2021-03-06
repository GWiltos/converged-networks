Model Differentiated Services
===

Spis treści:
1. [Wstęp](#wstep)
2. [Model DiffServ ](#model)
3. [Archtektura DiffServ](#architektura)
4. [Klasyfikowanie i oznaczanie pakietów](#klasyfikacja)
5. [PHB (Per hop behaviour)](#phb)
6. [Korzyści płynące z wdrożenia Differentiated Services](#zalety)
7. [Źródła](#zrodla)


## Wstęp  <a name="wstep"></a>
W sieci, w której wiele aplikacji rywalizuje o dostęp do pasma, zachodzi potrzeba priorytetyzacji pakietów w sieci.
Usługi potrzebują stałego i wysokiego poziomu jakości połączenia.
Dotyczy to szczególnie usług działających w trybie czasu rzeczywistego, które są nieodporne na opóźnienia
i wymagają stałej szerokości pasma. Takie wysokie wymagania doprowadziły do powstania mechanizmów, które będą zapewniały
transmisję danych z zachowaniem pewnych gwarancji. Obecnie najpopularniejszą architekturą sieci, która pozwala na zapewnienie
ścisłych gwarancji QoS w sieciach IP jest architektura Diffserv.
### Model DiffServ <a name="model"></a>
Jest to model oparty o architekturę usług zróżnicowanych, którego  głównym założeniem podczas tworzenia był wzrost 
skalowalności  w porównaniu do modelu [IntServ](Model%20Integrated%20Services.md). Architektura DiffServ nie wymaga sygnalizowania dla zestawienia połączenia.
Zapewnienie jakości usług polega na mechanizmie rozróżniania typu ruchu w sieci i odbywa się na poziomie klasy (grupy) połączeń,
a nie na poziomie pojedynczego połączenia. Architektura ta pozwala na gwarantowanie jakości usług przesyłania danych (QoS).
### Archtektura DiffServ <a name="architektura"></a>


  ![alt text](DiffService/architektura.png)
* **Agent Pasma (Bandwidth Broker)-** zarządza zasobami sieci, odpowiada za funkcje związane z konfiguracją routerów brzegowych 
oraz wykonywanie funkcji AC (Admision Control), czyli przyjmowanie albo odrzucanie nowych wywołań. 
Funkcja AC zapobiega stanom przeciążenia sieci oraz jest niezbędna w sieciach IP oferujących ścisłe gwarancje QoS.
* **Router brzegowy-** przydzielenie pakietu do klasy usług odbywa się w wejściowym ruterze brzegowym 
w chwili wejścia pakietu do domeny DiffServ.
* **Domena DiffServ-** zestaw ściśle związanych ze sobą węzłów(routerów). Domeny mogą ze sobą współpracować,
ale są również od siebie niezależne. Zbiór współpracujących ze sobą domen nazywa się region.
### Klasyfikowanie i oznaczanie pakietów <a name="klasyfikacja"></a>
**Pole DSCP [RFC2475](https://tools.ietf.org/html/rfc2474)** ma wartość 6 bitów i jest częścią jednobajtowego pola DS. W wersji 4 protokołu IP
wykorzystuje się do tego bajt ToS (Type of Service), natomiast w wersji 6 protokołu IP jest to bajt Traffic Class. Wartość pola DSCP określa przynależność danego pakietu do klasy i używane jest do wskazania sposobu przekazywania pakietu w węźle sieciowym.
Trzy najstarsze bity przydzielają dany pakiet do klasy i tym samym definiują jego priorytet, kolejne 2 bity przechowują dane dotyczące prawdopodobieństwa odrzucenia pakietu, ostatnie 2 bity zostały zarezerwowane dla celów eksperymentalnych i zostają one ignorowane przez urządzenia sieciowe podczas określania reguł PHB.

![alt text](DiffService/DSCP.png)

**Klasyfikacja pakietów** jest przeprowadzana na podstawie wartości jednego lub wielu pól nagłówka 
pakietu IP (np. adresu i numeru portu nadawcy czy identyfikatora protokołu transportowego) lub na podstawie wartości
punktu kodowego DSCP (Differantied Service Code Point) w nagłówku pakietu.

***BA (Behaviour Aggregate)-*** zespół pakietów z tym samym DSCP, pakiety te są tak samo traktowane w ramach domeny.

**Oznaczanie pakietów w routerach brzegowych** polega na ustawieniu odpowiedniej wartości pola DSCP w nagłówku pakietu. 
Oznaczanie może odbyć się bezpośrednio po przejściu przez mechanizm klasyfikacji bądź też po przejściu 
przez mechanizm monitorowania ruchu. W drugim przypadku wygląda to tak, że wartość pola zależy 
od przyporządkowania do odpowiedniego poziomu zgodności. Dzięki oznaczeniu pakietów mogą one w procesie 
kolejkowania być obsłużone z wyższym bądź niższym priorytetem. 
Oznaczenie wpływa również na obsługę pakietów w węzłach szkieletu sieci. 
### PHB (Per hop behaviour) <a name="phb"></a>
Sposób przesyłania pakietów przez routery szkieletowe określają reguły PHB (Per hop behaviour). 
Reguły te zdefiniowane są w każdym węźle dla każdej z obsługiwanych klas i określają sposób traktowania danej klasy 
ruchu rozpoznawanej na podstawie odpowiedniego pola DSCP.
#### Klasy usług 
Odpowiednie dokumenty RFC definiują przypisanie odpowiednich wartości kodowych do poszczególnych klas.
* **Przekazywanie niesklasyfikowane (bez opcji klasyfikowania)**
* **Klasa EX(Expedited Forwarding) [RFC3246](https://tools.ietf.org/html/rfc3246)** Cechuje się małymi opóźnieniami i małą zmiennością opóźnienia, 
ma również zagwarantowane pasmo.Tą klasę najczęściej przydziela się aplikacjom takim jak: VoIP,video,e-commerce. Wartość kodowa 101110.
* **Klasa AF(Assured Forwarding) [RFC2597](https://tools.ietf.org/html/rfc2597)** Charakteryzuje się brakiem gwarancji wielkości opóźnienia. 
Gwarantuje tylko, że wielkość opóźnienia będzie zgodna z ustaleniami z prawdopodobieństwem nie niższym, 
niż zostało uzgodnione. Zdefiniowane są 4 klasy usług AF. Pakiety, które zostaną przypisane do danej klasy AF, 
nie mogą zmieniać kolejności. W ramach każdej klasy określono 3 poziomy prawdopodobieństwa odrzucenia pakietów. 
Poziomy te służą do priorytetyzacji pakietów w kolejce. W sytuacji przeciążenia sieci w pierwszej kolejności 
odrzucane zostają pakiety z najwyższą wartością prawdopodobieństwa odrzucenia. Wartość kodowa:

Prawdopodobieństwo odrzucenia pakietu | Klasa 1 | Klasa 2 | Klasa 3 | Klasa 4
------------ | ------------- | ----------- | ------------- | -------------
Niskie  | 001010 | 010010 | 011010 | 100010 
Średnie  | 001100 | 010100 | 011100 | 100100 
Wysokie  | 001110 | 010110 | 011110 | 100110 

Przykładowo ruch pakietów w sieci można podzielić na 3 następujące klasy:
* **Pierwsza klasa** ruch w tej klasie ma przydzielone 50% dostępnej przepustowości łącza. Do tej klasy można zaliczyć ruch sesji TACACS. Pakiety, które znajdują się w tej klasie mają największy priorytet.
* **Druga klasa** ruch w tej klasie ma przydzielone 30% dostępnej przepustowości łącza (SMTP,FTP,SSH)
* **Trzecia klasa** ruch w tej klasie ma przydzielone 20% dostępnej przepustowości łącza. Klasa ta przeznaczona jest dla ruchu najmniej ważnego dla firmy z perspektywy gwarancji jakości. Może znaleźć się w niej ruch, który jest generowany przez przeglądarkę internetową(HTTP)
### Korzyści płynące z wdrożenia Differentiated Services <a name="zalety"></a>
* Zmniejsza obciążenie urządzeń sieciowych i jest łatwo skalowalny wraz ze rozwojem sieci
* Umożliwia klientom zachowanie i używanie istniejącego schematu priorytetyzacji ToS w warstwie 3
* Łagodzi wąskie gardła
---
### Źródła <a name="zrodla"></a>
* [Sposoby zapewnienia gwarantowanej
jakości usług w sieciach IP](http://www.przegladtelekomunikacyjny.pl/archive/WWW/artrec/stankiewicz2%272002.pdf) R.Stankiewicz, A.Jajszczyk
* [Architektura DiffServ](http://aai.tele.pw.edu.pl/data/SWUS/swus_diffserv_n.pdf) Politechnika Warszawska
* [Wprowadzenie do sieci z gwarancją obsługi](https://old.kt.agh.edu.pl/~jdomzal/QoS/lab_4/lab_diffserv.pdf) dr inż. Jerzy Domżał
* [Mechanizmy QOS w kontekście interowania sieci IPV4 I IPV6](http://zeszyty-naukowe.wwsi.edu.pl/zeszyty/zeszyt4/Mechanizmy_QOS_W_Kontekscie_Integrowania_Sieci_IPV4_i_IPV6.pdf) Tomasz Malinowski
* [Overview of DiffServ for Quality of Service](https://www.cisco.com/c/en/us/td/docs/ios/12_2/qos/configuration/guide/fqos_c/qcfdfsrv.pdf) Cisco Systems
* [RFC2597](https://tools.ietf.org/html/rfc2597)
* [RFC3246](https://tools.ietf.org/html/rfc3246)
* [RFC2475](https://tools.ietf.org/html/rfc2474)

