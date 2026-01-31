# Depression & Motor Activity — Time Series Analysis (Actigraph)

Projekt zaliczeniowy: analiza szeregów czasowych z sensorów (actigraf) w kontekście depresji/zaburzeń nastroju.  
Dane pochodzą z Kaggle: *The Depression Dataset (Depression and Motor Activity)*.

## Cel
1. Porównać rytm dobowy aktywności między grupami **control** vs **condition**.
2. Zbudować krótkoterminową prognozę aktywności (horyzont 48h) i ocenić modele metrykami błędu.
3. Wykryć anomalie (nagłe skoki aktywności) na podstawie reszt najlepszego modelu.

## Dane
Źródło: Kaggle — *The Depression Dataset*  
Pliki:
- `control/*.csv` – actigraph dla grupy kontrolnej
- `condition/*.csv` – actigraph dla grupy condition
- `scores.csv` – metadane i wyniki MADRS (`madrs1`, `madrs2`) + informacje demograficzne

Pomiar actigrafu: interwał minutowy (`timestamp`, `date`, `activity`).

## Metoda
### Przygotowanie danych
- Dane minutowe zagregowano do skali **1h** (średnia aktywność na godzinę), aby:
  - ograniczyć szum,
  - ułatwić modelowanie sezonowości dobowej (24h),
  - przyspieszyć obliczenia.
- Stabilizacja wariancji w modelowaniu: transformacja `log1p(activity)`.

### EDA (Exploratory Data Analysis)
- Wykresy szeregu czasowego (przykładowe osoby z najdłuższym pomiarem).
- Profil dobowy (średnia aktywność w godzinach 0–23) dla:
  - 1 osoby z każdej grupy,
  - średniej grupowej.

### Wskaźniki rytmu dobowego (na osobę)
Wyliczono metryki z profilu dobowego:
- `day_mean` — średnia aktywność 9–21
- `night_mean` — średnia aktywność 0–6
- `amp` — amplituda dzień–noc (`day_mean - night_mean`)
- `day_night_ratio` — (day+1)/(night+1)
- `rel_amp` — (max−min)/(max+min)

Porównanie grup: test **Welcha t-test** + **Cohen’s d**.

### Modelowanie i walidacja
Prognoza 48h dla wybranego uczestnika (najdłuższy zapis), ocena na zbiorze testowym:
- **seasonal naive** (baseline): `t = t-24h`
- **ETS Holt-Winters** (sezonowość 24h)
- **SARIMAX(1,0,1)x(1,0,1,24)** (lekka konfiguracja)

Metryki: **MAE**, **MSE** w skali `log1p(activity)`.  
AIC podano informacyjnie dla modeli statystycznych.

### Detekcja anomalii
- Najlepszy model wg MAE/MSE: **seasonal naive**.
- Anomalie wykryto regułą **3σ** na resztach (skala `log1p`).
- W oknie testowym wykryto **2 punkty odstające**, odpowiadające nagłym pikom aktywności.

## Najważniejsze wyniki (skrót)
- Grupa *condition* ma **niższą aktywność dzienną** oraz **inny profil dobowy (24h)** niż grupa kontrolna.
- Różnice w rytmie dobowym są widoczne głównie jako zmiana **kształtu profilu godzinowego**, a nie zawsze jako istotność pojedynczej metryki (np. amplitudy).
- W prognozie 48h kluczowa jest **sezonowość 24h** — model *seasonal naive* jest bardzo mocnym baseline i zwykle wygrywa z ETS.
- Anomalie (skrajne piki) można wykrywać na **resztach modelu**, ale detekcja zależy od ustalonego progu i wymaga jego konsekwentnego stosowania.

## Uruchomienie
### Wymagania
- Python 3.10+
- biblioteki: `pandas`, `numpy`, `matplotlib`, `scipy`, `statsmodels`, `scikit-learn`
