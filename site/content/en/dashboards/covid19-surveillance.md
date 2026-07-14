---
title: "COVID-19 Surveillance"
description: "Epidemiologická situace SARS-CoV-2 v České republice — denní případy, hospitalizace, testování a vakcinace."
image: "/images/dashboard-placeholder.svg"
highlight: true
tags: ["SARS-CoV-2", "hospitalizace", "testování", "vakcinace"]
data_source: '<a href="https://onemocneni-aktualne.mzcr.cz" target="_blank">MZČR — onemocnění aktuálně</a>'
update_freq: "Denně"
---

{{< stat-card src="/data/charts/covid_summary.json" >}}

{{< chart id="covidCases" src="/data/charts/covid_cases_weekly.json" title="Nové případy a úmrtí — týdenní přehled" height="380" >}}

{{< chart id="covidHosp" src="/data/charts/covid_hospitalization.json" title="Hospitalizace — stav pacientů (týdenní maximum)" height="380" >}}

{{< chart id="covidTest" src="/data/charts/covid_testing.json" title="Pozitivita PCR testů (%)" height="280" >}}

{{< chart id="covidInc" src="/data/charts/covid_incidence.json" title="7denní incidence na 100 000 obyvatel" height="280" >}}

<p class="stat-source">Data: <a href="https://onemocneni-aktualne.mzcr.cz/api/v2/covid-19" target="_blank">MZČR Open Data API v2</a> · Licence: otevřená data ČR</p>
