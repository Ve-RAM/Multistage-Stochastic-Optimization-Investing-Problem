# Assignment 2 — Stochastic Optimization

## Multistage investment using market data

**EAFIT University · Semester 2026-2**  
**Instructor:** Diego Fonseca  
**Team members:** Jaime Andres Jaramillo and Santiago Vera Ramirez

## 1. Overview

This project builds and evaluates a multistage investment policy for a foundation with **USD 100,000** at the end of December 2025 that aims to reach **USD 106,000** by September 2026. The portfolio consists of:

- **SPY:** ETF tracking the U.S. equity market.
- **LQD:** Investment-grade corporate bond ETF.
- **GLD:** Gold-backed ETF.

The model uses **thousands of U.S. dollars** as its monetary unit, allows rebalancing at the end of March and June, and minimizes the expected cost defined as four times the terminal shortfall minus the terminal surplus. It is calibrated using quarterly historical data from 2009–2025, while the 2026 trajectory is held out for out-of-sample evaluation.

The main file is `Practica_2_Optimizacion_Estocastica.ipynb`. It contains the model formulation, implementation, checks, analyses, and export of results.

## 2. Methods and workflow

The notebook performs the following steps:

1. Installs/imports dependencies and defines numerical tolerances and problem parameters.
2. Downloads quarterly adjusted closing prices for SPY, LQD, and GLD from Yahoo Finance, using the NYSE trading calendar to check the dates.
3. Computes returns and growth factors, then groups returns into eight joint states based on the signs of the three assets (`A`: nonnegative return; `B`: negative return).
4. Builds a three-period scenario tree. Joint-state probabilities are estimated from historical frequencies, and independence is assumed across different periods; independence between assets within the same quarter is **not** assumed.
5. Formulates and solves the optimization problem as a linear program using **nested L-shaped decomposition** and SciPy/HiGHS dual multipliers to generate cuts.
6. Checks the implementation on a reduced instance and compares the L-shaped result with the deterministic equivalent.
7. Solves the full scenario tree, records lower and upper bounds, and checks the relative convergence gap.
8. Builds an expected-value (EV) policy and compares its expected cost with the stochastic policy within the scenario tree; it also computes the value of the stochastic solution (VSS).
9. Fixes the policy before downloading the 2026 validation data, then evaluates both policies using closing prices for December 2025 and March, June, and September 2026.
10. Examines dominance between growth factors, variability removed by state aggregation, and the actual trajectory versus the tree's representative trajectory.
11. Exports tables, figures, metadata, package versions, and a ZIP archive containing the run results.

The solution is a policy conditioned on the history of observed states. For out-of-sample evaluation, the policy's investment proportions are applied to the actual available wealth. If a state occurs that is not part of the calibrated tree, the notebook allocates equal proportions to the assets for the remaining periods.

## 3. Requirements

- **Python 3** in a Jupyter Notebook-compatible environment.
- Internet access to retrieve data from Yahoo Finance.
- The Python dependencies listed below.
- **Google Colab is recommended**, as it is the notebook's intended environment and simplifies downloading the final ZIP archive.

Dependencies:

- `numpy`
- `pandas`
- `scipy` (HiGHS linear programming solver)
- `matplotlib`
- `yfinance`
- `pandas_market_calendars`

### Installation

In Google Colab, run the first code cell, which installs the dependencies using `%pip`:

```python
%pip -q install yfinance pandas_market_calendars scipy pandas numpy matplotlib
```

For a local environment, install the packages with:

```bash
python -m pip install yfinance pandas_market_calendars scipy pandas numpy matplotlib jupyter
```

The specific package versions are recorded when the notebook runs and are written to `requirements.txt` in the results folder. To reproduce a particular run, retain that file and use the recorded versions.

## 4. How to run

### Recommended: Google Colab

1. Upload or open `Practica_2_Optimizacion_Estocastica.ipynb` in Google Colab.
2. Make sure the runtime has internet access.
3. From the **Runtime** menu, select **Run all**. Alternatively, execute the cells manually in order, from first to last.
4. Wait for the data downloads, linear programming solves, and checks to finish. Solving the full scenario tree may take some time.
5. The notebook creates `/content/practica2_resultados`, saves the results, and generates `/content/practica2_resultados.zip`.
6. The final cell downloads the ZIP archive through the Colab interface.
7. Also save a copy of the notebook with its outputs. In Colab, select **File → Download → Download .ipynb**.

### Local execution

The notebook's final cell imports `google.colab.files`, so the download step is intended for Colab and will not work unchanged in a local environment. In local Jupyter, run the preceding cells in order and, if necessary, replace the final cell with a manual download or copy of the generated ZIP archive. Output paths are configured for `/content/practica2_resultados`; adjust `CARPETA` when running outside Colab.

## 5. Data and reproducibility

- **Data source:** Yahoo Finance, `Adj Close` field for SPY, LQD, and GLD.
- **Calibration period:** quarterly closing prices from December 2009 through December 2025, providing 64 returns per asset.
- **Out-of-sample validation:** closing prices for December 2025 and March, June, and September 2026.
- **Calendar:** NYSE trading calendar, used to verify closing dates.
- **Missing data:** the notebook does not fill in missing prices or replace a quarterly close with an earlier quote. If the dates or data fail the checks, execution stops.
- **Randomness:** the notebook **does not use random components** to generate scenarios or solve the model. Therefore, **no random seed is required**. Scenarios are enumerated from observed states and their empirical probabilities.
- **Tolerances:** the relative convergence gap and reduced-instance comparison use a tolerance of `1e-6`; the notebook also defines tolerances for numerical feasibility and target attainment.
- **External dependency:** Yahoo Finance may update or revise historical data, including adjusted prices. Consequently, runs performed on different dates may produce different results even when the code is unchanged.

To preserve evidence of a run, save the notebook with its outputs and the generated CSV files, especially `registro_descargas.csv` (download log). The notebook also exports `metadatos.json`, containing the run date, Python version, package versions, confirmation that no random components are used, and key solution metrics.

To repeat the analysis without querying Yahoo Finance again, the notebook notes that the saved closing prices can be loaded with `pandas.read_csv`. Retain the original query log and verify that the dates and dimensions match the expected values.

## 6. Generated files

The `practica2_resultados` folder includes, among other outputs:

- **CSV data and results:** returns, growth factors, states, scenario tree, investment policy, terminal outcomes, convergence, cuts, EV comparison, chronological validation, dominance, variability, and summary tables.
- **PNG figures:** price history, bound convergence, policy composition, wealth distribution, and validation trajectories.
- **`politica_fija.json`:** parameters, growth factors, and policy proportions fixed before downloading the validation data.
- **`requirements.txt`:** installed versions of the main dependencies.
- **`metadatos.json`:** execution environment and key metrics.
- **`README.txt`:** brief instructions also included in the results package.
- **`practica2_resultados.zip`:** compressed archive containing the exported results.

The results ZIP **does not automatically include the executed notebook or replace the PDF report**. Save and submit those separately if required by the assignment.

## 7. Checks and interpretation

The run includes assertions to check investment balances, nonnegativity, reduced-instance dimensions, agreement between methods within tolerance, and consistency of the L-shaped bounds. If the algorithm does not reach the tolerance within the iteration limit, it raises an error instead of reporting convergence.

The scenario-tree result is an optimal solution for the calibrated model, not a guarantee of future performance. The 2026 out-of-sample validation represents a single observed trajectory and, by itself, cannot establish general superiority or accurately estimate a probability of success. Expected cost within the scenario tree and realized cost in 2026 are different measures.

## 8. References

- Yahoo Finance historical data: [SPY](https://finance.yahoo.com/quote/SPY/history/), [LQD](https://finance.yahoo.com/quote/LQD/history/), and [GLD](https://finance.yahoo.com/quote/GLD/history/).
- `yfinance`: [documentation](https://ranaroussi.github.io/yfinance/).
- SciPy: [`linprog` with HiGHS](https://docs.scipy.org/doc/scipy/reference/optimize.linprog-highs.html).
- `pandas_market_calendars`: [documentation](https://pandas-market-calendars.readthedocs.io/).
