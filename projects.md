# Projects

This section documents my data science projects, research questions, and data stories I create throughout the semester.

---

<div class="card">

## Project 1: Does Explicit Content Affect Song Popularity?

**Research Question:**
Does having explicit content help or hurt a song's popularity on Spotify, and does that relationship differ across genres?

**Data Source:**
Data pulled from the [Spotify Web API](https://developer.spotify.com/documentation/web-api), using the Client Credentials authentication flow.

**Key Variables:**
- Explicit Content Flag — whether a track is marked explicit or clean (main independent variable)
- Search Term — used as a genre proxy (see note below)
- Duration — converted from milliseconds to seconds
- Release Date — used to examine trends over time

### My Analytical Process

When I started this project, I wanted to compare explicit vs. clean songs by genre using Spotify's own genre metadata. Early on, I discovered that Spotify's API restricts genre-classification and popularity-score access for developer apps, a change made in late 2024, so I had to pivot to using genre-related search terms as a proxy instead. This meant rethinking my approach. Rather than pulling a verified genre field, I treated each search term, such as pop or rock, as its own labeled subset, which is a weaker signal but still lets me compare explicit vs. clean tracks within each group.

I also ran into an API limit I didn't expect. Spotify's search endpoint caps results at 10 per request, which shaped my final sample size, 50 tracks across 5 search terms, more than I originally planned. Rather than forcing a workaround that might violate API terms, I treated this as a real constraint to document rather than hide. This also meant Spotify's popularity score was not available through this endpoint for my app's access level, so popularity itself could not be used as a variable in this analysis.

For cleaning, I converted track duration from milliseconds to seconds for readability, and parsed release dates into proper datetime format so I could plot duration trends over time. I chose a boxplot for the explicit vs. clean comparison specifically because it shows the spread and outliers within each genre group, not just an average, which matters here since a few very long or short tracks could otherwise skew a simple bar chart.

If I continued this project, I would want to find a data source that still exposes true genre tags and popularity scores, possibly by combining Spotify's public track data with an external dataset like Kaggle's historical Spotify charts, to get a more reliable signal for the original research question.

### Visualizations

![Track Duration by Genre Search Term and Explicit Content](viz1.png)

![Track Duration Over Time by Genre Search Term](viz2.png)

</div>

