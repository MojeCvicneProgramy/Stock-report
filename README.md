# Stock-report

**Výsledný report:**
<img width="1138" alt="image" src="https://github.com/user-attachments/assets/c6156ec3-d88d-4df2-9736-417e0d7ca05b">

**Postup:**

--------
**Dáta získané pomocou API, nahrané do Power BI ako zdroj pomocou M-language funkcie:**

API zdroj: https://www.alphavantage.co/support/

dokumentácia: https://www.alphavantage.co/documentation/




let Source = (symbol) =>

let
    Zdroj = Json.Document(Web.Contents(
        "https://www.alphavantage.co/",
        [
        RelativePath="query",
        Query=[
            function="TIME_SERIES_DAILY",
            symbol=symbol,
            outputsize="full",
            apikey="HZNGD0465F7KXXOX"
        ]
        ]

        
        )),
          /// "https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=AMZN&outputsize=full&apikey=HZNGD0465F7KXXOX"
    #"Time Series (Daily)" = Zdroj[#"Time Series (Daily)"],
    #"Konvertované na tabuľku" = Record.ToTable(#"Time Series (Daily)"),
    #"Rozbalené Value" = Table.ExpandRecordColumn(#"Konvertované na tabuľku", "Value", {"1. open", "2. high", "3. low", "4. close", "5. volume"}, {"1. open", "2. high", "3. low", "4. close", "5. volume"})
in
    #"Rozbalené Value"

in Source

Výsledná tabuľka v Power Query, následne bola upravená
![image](https://github.com/user-attachments/assets/111b3266-2de9-4523-8bb8-4e426651fee2)

toto je tabuľka akcií, ktoré číta funkcia a ktoré potom naiahne pomocou API
![image](https://github.com/user-attachments/assets/d10535f1-ef2e-4d28-922a-5f830bf441c1)

Ručne vložená tabuľka s dátami ohľadne nákupu akcií, dátumy a počet
![image](https://github.com/user-attachments/assets/03c9be4d-139c-4561-8734-6ca414cb4694)


------
**Pre výpočet nákupnej ceny použitý DAX kód:**

Nakupna Cena = 
SUMX(
    'Nákup',
    'Nákup'[Počet] *
    CALCULATE(
        SUM('Stocks'[close]),
        FILTER(
            'Stocks',
            'Stocks'[Date] = 'Nákup'[Dátum nákupu].[Date] &&
            'Stocks'[stock] = 'Nákup'[Stock]
        )
    )
)

----------
**Pre výpočet predajnej ceny použitý DAX kód:**

Predaj Cena = 
SUMX(
    'Nákup',
    'Nákup'[Počet po splite] *
    CALCULATE(
        SUM('Stocks'[close]),
        FILTER(
            'Stocks',
            'Stocks'[Date] = 'Nákup'[Včera].[Date] &&
            'Stocks'[stock] = 'Nákup'[Stock]
        )
    )
)

--------
**API poskytuje dáta s oneskorením jedného dňa, takže aktuálnu dnešnú cenu sa nedá zistiť, len včerajšiu.**

Použitý kód DAX:

Včera = TODAY() - 1


-------
**Pre akcie Googlu a Amazonu použitý kód DAX, ktorý zohľadnuje split akcií v roku 2022 a prepočítava reálny počet akcií za reálnu cenu**


![image](https://github.com/user-attachments/assets/5b52d8be-36ec-4acc-abb9-0a120c81ff07)


Počet po splite = 
VAR AkciaAMZN = 
    [stock] = "AMZN" && [Dátum nákupu] < DATE(2022, 6, 6)
VAR AkciaGOOGL = 
    [stock] = "GOOGL" && [Dátum nákupu] < DATE(2022, 7, 15)
RETURN
    IF(AkciaAMZN || AkciaGOOGL, [Počet] * 20, [Počet])

![image](https://github.com/user-attachments/assets/9da900f9-0bd6-477a-8559-501d22b162f0)

-----
**Pre výpočet Celkového zisku a percentuálneho nárastu vytvorené nové Measure:**

Celkový zisk/strata = [Predaj Cena] - [Nakupna Cena]

Nárast/pokles = ([Predaj Cena] - [Nakupna Cena])/[Nakupna Cena]






    




------
