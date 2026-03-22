# Modeling EV/EBITDA with Financial Ratios

Projekt laczy ekonometrie, machine learning i klasteryzacje, aby odpowiedziec na pytanie:

**co najbardziej tlumaczy roznice w mnozniku EV/EBITDA miedzy spolkami?**

Analiza zostala przeprowadzona etapowo w notebookach `01-04` i konczy sie porownaniem podejsc modelowych oraz interpretacja wynikow w jezyku biznesowym.

## Executive Summary

- Po solidnym czyszczeniu danych jakosc modeli rosnie wyraznie: model OLS na probce minimalnie czyszczonej daje ok. `R^2=0.028`, a po pelnym czyszczeniu ok. `R^2=0.188`.
- Korekta heteroskedastycznosci (WLS/FGLS-like) poprawia dopasowanie do ok. `R^2=0.220`.
- Najlepszy model predykcyjny out-of-sample to **tuned XGBoost**: `R^2=0.3313`, `MAE=3.5877`, `RMSE=4.6996`, `MAPE=47.93%`.
- Klasteryzacja firm (`k=4`) pokazuje istotne roznice wycen miedzy archetypami biznesowymi (`Kruskal H=69.23`, `p<0.001`).
- Najwyzsza mediana EV/EBITDA wystepuje w klastrze o profilu **innovation + growth** (`14.079`), najnizsza w klastrze **market core** (`9.730`).

## Zakres i Dane

Wejsciem jest zbior surowy `data/raw/dane_finansowe.csv`, a nastepnie kolejne wersje danych przetworzonych.

Glowna grupa zmiennych obejmuje m.in.:

- wzrost (`Revenue_Growth`),
- rentownosc (`Profit_Margin`, `Gross_Profit_Margin`, `Return_on_Equity`),
- efektywnosc (`Asset_Turnover`, `Fixed_Asset_Turnover`),
- zadluzenie (`Debt_to_Equity`),
- inwestycje (`CAPEX_to_Revenue`, `R&D_to_Revenue`),
- plynnosc (`Current_Ratio`),
- wyplate dla akcjonariuszy (`Dividend_Yield`).

## Co Zostalo Zrobione (Notebooki 01-04)

### 1) Data and Problem Setup (`01_data_and_problem_setup.ipynb`)

Wykonano pipeline przygotowania danych do modelowania:

- kontrola brakow i wartosci skrajnych,
- filtrowanie EV/EBITDA oraz wskaznikow finansowych (progi ekonomiczne + percentyle),
- budowa wersji danych od "minimalnie czyszczonej" do "modeling-ready".

Najwazniejszy efekt etapu: **stabilniejsza, bardziej wiarygodna baza** do estymacji.

### 2) Econometric Analysis (`02_econometric_analysis.ipynb`)

Przeprowadzono pelna analize regresyjna:

- OLS (bazowy),
- testy diagnostyczne (Breusch-Pagan, White, Jarque-Bera, RESET),
- analiza punktow wplywowych (Cook's distance),
- WLS/FGLS-like jako odpowiedz na heteroskedastycznosc,
- testy roli sektorow i stabilnosci struktury.

Wyniki kluczowe:

- `R^2=0.188` (OLS, cleaned benchmark),
- `R^2=0.220` (WLS/FGLS-like),
- heteroskedastycznosc i napiecie formy funkcyjnej sa statystycznie istotne,
- efekt sektorowy jest istotny i nie powinien byc pomijany.

### 3) Machine Learning Models (`03_machine_learning_models.ipynb`)

Porownano kilka klas modeli i skupiono sie na metryce out-of-sample `R^2`:

- WLS benchmark na tym samym podziale train/test,
- XGBoost baseline,
- XGBoost tuned (RandomizedSearchCV),
- LightGBM, CatBoost, Random Forest, Ridge,
- warianty ensemble/stacking/log-target oraz analiza wyjasnialnosci (SHAP).

Wyniki kluczowe:

- WLS (test): `R^2=0.2472` (weighted test `R^2=0.2734`),
- XGBoost baseline: `R^2=0.3081`,
- **XGBoost tuned: `R^2=0.3313` (najlepszy wynik),**
- LightGBM: ok. `R^2=0.3178`,
- rozwiazania ensemble poprawialy profil bledow, ale nie przebily tuned XGBoost w top-line `R^2`.

Interpretacja: zaleznosci w danych sa czesciowo nieliniowe i interakcyjne, dlatego modele drzewiaste wypadaja lepiej od prostych modeli liniowych.

### 4) Clustering Companies (`04_clustering_comapnies.ipynb`)

Wykonano segmentacje firm metoda K-means (`k=4`) oraz profilowanie klastrow:

- standaryzacja cech,
- dobor i walidacja liczby klastrow,
- wizualizacje (PCA, silhouette),
- testy roznic miedzy grupami,
- eksport list firm i statystyk klastrow do CSV.

Wyniki kluczowe:

- mean silhouette: `0.3175` (umiarkowanie czytelny podzial),
- istotne roznice EV/EBITDA miedzy klastrami (`H=69.23`, `p<0.001`),
- mediany EV/EBITDA:
  - Cluster 4: `14.079` (najwyzsza premia),
  - Cluster 1: `11.726`,
  - Cluster 2: `9.785`,
  - Cluster 3: `9.730` (najnizsza).

## Najwazniejsze Wyniki Przekrojowe

1. **Jakosc danych jest krytyczna.**
Lepsze czyszczenie radykalnie poprawia sygnal modelowy i stabilnosc wnioskow.

2. **ML wygrywa z klasycznym modelem liniowym w predykcji out-of-sample.**
Najlepszy wynik daje tuned XGBoost (`R^2=0.3313`) vs WLS (`R^2=0.2472`).

3. **Heteroskedastycznosc nie jest detalem technicznym, tylko realna cecha danych.**
Wazenie obserwacji (WLS) poprawia dopasowanie i daje bardziej realistyczne wnioskowanie.

4. **Segmentacja odslania archetypy biznesowe, ktorych nie widac w jednej sredniej regresji.**
Profil innovation + growth ma wyrazna premie wyceny wzgledem profilu neutralnego.

## Wnioski Biznesowe

- Rynek premiuje spolki laczace wzrost i inwestycje rozwojowe (szczegolnie R&D), przy relatywnie kontrolowanym ryzyku finansowania.
- Sama wysoka rentownosc nie zawsze przeklada sie na wyzszy mnoznik, jezeli towarzyszy jej podwyzszone ryzyko bilansowe.
- Modelowanie wieloklasowe (ekonometria + ML + clustering) daje pelniejszy obraz niz jedna technika uzyta w izolacji.

## Ograniczenia i Ryzyka Interpretacji

- Analiza ma charakter przekrojowy, wiec nie jest to model przyczynowy.
- Wyniki zaleza od przyjetej logiki czyszczenia danych i filtrow outlierow.
- Istnieje ryzyko pominietych zmiennych (np. czynniki makro, jakosc zarzadzania, zdarzenia jednorazowe).
- Separacja klastrow jest umiarkowana, wiec granice miedzy profilami nie sa idealnie ostre.

## Artefakty Projektu

Kluczowe pliki wyjsciowe:

- `data/processed/df_cleaned.csv`
- `data/processed/df_sectors_cook2.csv`
- `data/processed/ml_train_dataset_used_for_models.csv`
- `data/processed/ml_test_dataset_used_for_models.csv`
- `data/processed/clusters/companies/cluster_company_list_*.csv`
- `data/processed/clusters/statistics/cluster_statistics_*.csv`

Notebooki:

- `notebooks_new/01_data_and_problem_setup.ipynb`
- `notebooks_new/02_econometric_analysis.ipynb`
- `notebooks_new/03_machine_learning_models.ipynb`
- `notebooks_new/04_clustering_comapnies.ipynb`

## Podsumowanie

Projekt dostarcza spojny workflow od przygotowania danych, przez rygorystyczna diagnostyke ekonometryczna, po nowoczesne modele ML i segmentacje firm.

Najmocniejszy wniosek praktyczny: **premia wyceny jest powiazana nie tylko z biezaca rentownoscia, ale rowniez z profilem wzrostu, innowacyjnosci i struktura ryzyka.**