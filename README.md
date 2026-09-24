# Project_screentime_analysis
This R Shiny app was created for the Data Visualization Techniques course at the Faculty of Mathematics and Information Science, Warsaw University of Technology.
 
The app presents the results of an analysis of our screen time in the form of four charts:
 
- **Screen time by day.**
![](plots/wykres1.png)
- **Screen time in a selected week by hour of the day.** The total and average number of hours spent on screen are also calculated.
![](plots/wykres2.png)
- **Most frequently used apps, grouped by category.** The treemap shows each app's and category's share of total screen time, represented by the color scale and the size of the tiles.
![](plots/wykres3.png)
- **Sleep duration.** Estimated based on the phone's last activity of the night and the last morning alarm. The chart also shows the average and latest wake-up time.
![](plots/wykres4.png)
## Data source
 
The data was collected using [ActivityWatch](https://activitywatch.net) over the following periods:
 
- 29 Nov 2025 – 26 Jan 2026 for phones,
- 8 Dec 2025 – 26 Jan 2026 for computers.
## Libraries
 
- shiny
- jsonlite
- dplyr
- tidyr
- tibble
- plotly
- lubridate
- hms
- fontawesome
- treemapify
## My contribution 
 
This was a team project. My part was the **treemap of the most frequently used apps by category** (chart no. 3), including all the data cleaning and categorisation behind it. The other charts and the app's UI layout were prepared by my teammates.
 
My focus was on turning raw, event-level activity logs into a clean, meaningful summary that a viewer can read at a glance, and on making that pipeline respond to the user's choices in the app (person, device and date range).
 
### Data cleaning (`cleaning_data_ania.R`)
 
The input is a raw CSV export from ActivityWatch, one file per person and device (phone or computer). Each row is a single activity event with a timestamp, a duration and an app name. The function `cleaning_data_ania(path, start_date, end_date)` prepares this data in the following steps:
 
1. **Loading and date filtering.** I read the file, convert timestamps to dates and keep only the events from the range selected in the app. If the selected range contains no data, the app shows a clear message instead of an error (`shiny::validate`).
2. **Standardising app names.** App names contained Polish diacritics (e.g. "Zdjęcia", "Tłumacz"). I transliterated them to plain ASCII with `stringi`, so that they could be matched reliably against my category lists.
3. **Manual categorisation.** I built lookup lists of app names for nine categories: entertainment, communication, learning and work, shopping and services, travel, photos, browsers, system tools, and sport and health. Assigning apps to categories required judgement calls, for example RStudio, MATLAB, Teams and Excel count as "learning and work", while banking and food-delivery apps count as "shopping and services". The category labels in the app are in Polish.
4. **Filtering out uncategorised apps.** Apps that are not on any list are labelled "other" and removed, so that the treemap shows recognisable apps instead of a long tail of rarely used ones. As a result, the treemap does not cover the total screen time, only the categorised part of it.
5. **Combining devices.** When the user selects "all devices", the function is run separately on the phone and the computer file and the results are combined (`bind_rows`) in the Shiny server.
### Building the treemap (`plot_ania.R`)
 
The function `make_treemap(df)` takes the cleaned data and returns an interactive `plotly` treemap:
 
1. **Aggregation.** With `dplyr`, I sum the time spent per category and per app, and calculate each app's share of its category's time.
2. **Readability.** To avoid dozens of tiny, unreadable tiles, only apps that make up at least 10% of their category's time are shown as separate tiles.
3. **Hierarchy.** Categories are the parent nodes and apps are their children, so the user sees both the split between categories and the leading apps inside each one.
4. **Encoding.** The tile size and the colour both represent time spent (in hours). I designed a custom blue colour scale, from dark navy for small values to light cyan for large ones, with a colour bar labelled in hours.
5. **Interactivity.** Hovering over a tile shows the exact time in hours and the app's percentage share of its category. Clicking a category zooms into it (a built-in `plotly` treemap feature).
6. **Styling.** I used a dark background and white text to match the look of the whole app.
### Integration with the app
 
In the Shiny server, the treemap is built from reactive expressions. The selected person, device (phone, laptop or all) and date range determine which files are loaded and cleaned, so the chart updates automatically whenever the user changes any of these inputs.
 
### What I learned
 
- cleaning and structuring real-world log data for visualisation,
- designing a readable hierarchical chart and choosing what to leave out,
- building reusable R functions and connecting them to a reactive Shiny app,
- working on a shared codebase in a team.
