# Self-Supervised Anomaly Detection in GPS Taxi Trajectories

Porównanie nienadzorowanego uczenia reprezentacji szeregów czasowych (**TS2Vec**) z klasyczną inżynierią cech (**Baseline**) w zadaniu detekcji anomalii na trajektoriach taksówek w Porto (ECML-PKDD 2015).

Projekt zrealizowany w ramach przedmiotu **Advanced Data Mining (ADM 2026)**.

---

## 👥 Autorzy
* **Hubert Stolarz**
* **Karol Kulig**

---

## 📌 Opis Projektu

Głównym wyzwaniem w detekcji anomalii w trajektoriach miejskich jest **całkowity brak etykiet** w rzeczywistych danych (brak ground truth). Celem projektu było zbadanie, jak nowoczesne, samonadzorowane uczenie reprezentacji (Self-Supervised Learning) radzi sobie w porównaniu z klasycznymi, interpretowalnymi statystykami trajektorii.

Oba podejścia zostały zestawione z wykorzystaniem tego samego algorytmu scoringowego — **Isolation Forest** (200 drzew, `contamination='auto'`). Ewaluację ilościową przeprowadzono poprzez kontrolowane wstrzykiwanie 4 typów syntetycznych anomalii do rzeczywistych tras.

---

## 📊 Zbiór Danych

* **Źródło:** [Porto Taxi Service Trajectory Prediction (ECML-PKDD 2015 / Kaggle)](https://www.kaggle.com/c/pkdd-15-predict-taxi-service-trajectory-i/data)
* **Wolumen:** ~1.7 miliona zarejestrowanych kursów (442 taksówki, lipiec 2013 – czerwiec 2014)
* **Częstotliwość próbkowania:** punkty GPS co 15 sekund
* **Preprocessing:** Usunięcie rekordów z flagą `MISSING_DATA` oraz pustych współrzędnych, odrzucenie tras poniżej 5 i powyżej 360 punktów (np. zapomniane taksometry). Zastosowano próbkowanie systematyczne (co 30. rekord), uzyskując zbalansowaną i odszumioną próbkę **54 573 przejazdów**.

---

## ⚙️ Architektura i Podejścia

| Cecha | Baseline (Inżynieria cech) | TS2Vec (Uczenie samonadzorowane) |
| :--- | :--- | :--- |
| **Reprezentacja** | Wektor 8 cech skalarnych | Wektor gęsty (320-dim embedding) |
| **Dane wejściowe** | Długość, liczba punktów, dystans w linii prostej, detour ratio, prędkości (mean/max/std), przekątna bounding boxa | 3 kanały: $\Delta\text{lon}$, $\Delta\text{lat}$, prędkość (w m i m/s); padding do 200 kroków czasowych |
| **Model** | Standaryzacja cech | TS2Vec trenowany przez 5 epok na 10k trasach + mean-pooling |
| **Detektor anomalii** | Isolation Forest | Isolation Forest |

---

## 🧪 Metodologia Ewaluacji: Wstrzykiwanie Anomalii

Do 800 losowych tras wprowadzono po 200 syntetycznych modyfikacji w 4 kategoriach:
1. **Detour:** Wklejenie pętli (~2 km) w środek trasy (symulacja naciągania kursu).
2. **GPS Jump:** Przesunięcie jednego punktu o ~3 km (symulacja usterki pomiarowej / skoku prędkości).
3. **Heavy Noise:** Rozproszony szum gaussowski (~150 m) na każdym punkcie (symulacja słabej jakości sygnału GPS).
4. **Reversal:** Odwrócenie sekwencji czasowej punktów trasy (`pts[::-1]`).

---

## 📈 Wyniki

### ROC-AUC wg typu anomalii:

| Typ anomalii | Baseline | TS2Vec | Wygrana metoda |
| :--- | :---: | :---: | :---: |
| **Detour** | **0.819** | 0.349 | Baseline |
| **GPS Jump** | **0.970** | 0.769 | Baseline |
| **Noise** | 0.934 | **0.988** | **TS2Vec** |
| **Reversal** | 0.512 | 0.552 | Losowe (~0.50) |

### Główne wnioski:
* **Inne definicje anomalii:** Baseline i TS2Vec wyłapują fundamentalnie odmienne nieprawidłowości. 
* **Baseline** doskonale radzi sobie z ekstremami skalarnymi i anomaliami geometrycznymi (nagłe skoki prędkości, nienaturalnie wysokie detour ratio).
* **TS2Vec** przewyższa podejście klasyczne w wykrywaniu rozproszonych zaburzeń strukturalnych (poszarpany szum), analizując płynność i lokalny kształt trajektorii.
* **Problem mean-poolingu:** Agregacja sekwencji do pojedynczego wektora za pomocą średniej całkowicie usuwa informację o kierunku upływu czasu, uniemożliwiając detekcję odwróconych tras (*Reversal*).
* **Niski PR-AUC:** Wynika z obecności wielu realnych, nieoznaczonych anomalii w zbiorze z Porto, które modele poprawnie umieszczały na czele rankingu, obniżając precyzję względem sztucznie wstrzykniętych próbek.

---

## 📁 Struktura Repozytorium

```plaintext
.
├── 01_load_and_clean.ipynb       # Pobieranie, filtrowanie i czyszczenie zbioru Porto Taxi
├── 02_baseline.ipynb             # Ekstrakcja 8 cech, wstrzykiwanie anomalii, ewaluacja Baseline
├── 03_ts2vec.ipynb               # Przygotowanie kanałów, trening TS2Vec i generowanie embeddingów
├── 04_maps_and_analysis.ipynb    # Wizualizacja tras na mapach, analiza rozbieżności i wykresy
├── ADM Project Presentation.pdf  # Slajdy z prezentacji projektu
├── ADM Project Raport.pdf        # Pełny raport techniczny z eksperymentów
└── README.md
