---
jupyter:
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.12.0
  nbformat: 4
  nbformat_minor: 2
---

::: {.cell .markdown}
### Importing Libraries
:::

::: {.cell .code execution_count="139"}
``` python
import numpy as np
import pandas as pd
```
:::

::: {.cell .markdown}
### Data Import
:::

::: {.cell .code execution_count="140"}
``` python
df0 = pd.read_json("data/MyData - 24 Oct 2023/StreamingHistory0.json")
df1 = pd.read_json("data/MyData - 24 Oct 2023/StreamingHistory1.json")
df2 = pd.read_json("data/MyData - 24 Oct 2023/StreamingHistory2.json")

df = pd.concat([df0, df1, df2], ignore_index = True)
del df0, df1, df2

print(f"Data import successful: {len(df)} rows, {len(df.columns)} columns")
```

::: {.output .stream .stdout}
    Data import successful: 21254 rows, 4 columns
:::
:::

::: {.cell .markdown}
### Data Timeframe
:::

::: {.cell .code execution_count="141"}
``` python
from datetime import datetime

def better_date(date):
    date_obj = datetime.strptime(date, "%Y-%m-%d")
    return date_obj.strftime("%d %B %Y")

print(f"Oldest date: {better_date(min(df["endTime"].tolist())[0:10])}\nMost recent date: {better_date(max(df["endTime"].tolist())[0:10])}")
```

::: {.output .stream .stdout}
    Oldest date: 01 October 2022
    Most recent date: 22 October 2023
:::
:::

::: {.cell .markdown}
### Top 10 Tracks
:::

::: {.cell .code execution_count="142"}
``` python
top_tracks = df.groupby(["artistName", "trackName"]).agg(time_played_min = ("msPlayed", "sum")).reset_index().sort_values("time_played_min", ascending = False)

to_omit = ["Heavy Thunder and Rain Sounds 8 Hours | Thunderstorm Sounds for Study and Sleep",
           "Sleep Sounds Rain & Thunderstorm White Noise 8 Hours | Fall Asleep with Rainstorm Sound Masking",
           "Long Rumbling Thunder - 10 hours",
           "Tropical Beach | Relaxing Ocean Sounds 8 Hours",
           "#1847 - Theo Von", "#1836 - Ryan Holiday"]

top_tracks = top_tracks[~ top_tracks["trackName"].isin(to_omit)]
top_tracks["time_played_min"] = round(top_tracks["time_played_min"] / 1000 / 60, 2)

top_tracks.head(n = 10).reset_index(drop = True).rename(columns = {"artistName": "Artist Name", "trackName": "Track Name", "time_played_min": "Minutes Played"})
```

::: {.output .execute_result execution_count="142"}
```{=html}
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Artist Name</th>
      <th>Track Name</th>
      <th>Minutes Played</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Radiohead</td>
      <td>Let Down - Remastered</td>
      <td>283.21</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ethel Cain</td>
      <td>American Teenager</td>
      <td>229.07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Carly Rae Jepsen</td>
      <td>The Loneliest Time (feat. Rufus Wainwright)</td>
      <td>219.92</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Julian Casablancas</td>
      <td>River of Brakelights</td>
      <td>211.01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Strokes</td>
      <td>Life Is Simple in the Moonlight</td>
      <td>194.55</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Radiohead</td>
      <td>Jigsaw Falling Into Place</td>
      <td>189.38</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Weyes Blood</td>
      <td>Children of the Empire</td>
      <td>181.84</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Ethel Cain</td>
      <td>A House In Nebraska</td>
      <td>179.80</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Cigarettes After Sex</td>
      <td>Apocalypse</td>
      <td>176.09</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ethel Cain</td>
      <td>Thoroughfare</td>
      <td>165.90</td>
    </tr>
  </tbody>
</table>
</div>
```
:::
:::

::: {.cell .markdown}
### Top 10 Artists
:::

::: {.cell .code execution_count="143"}
``` python
top_artists = df.groupby("artistName").agg(min_played = ("msPlayed", "sum")).reset_index().sort_values("min_played", ascending = False).reset_index(drop = True)

top_artists["min_played"] = round(top_artists["min_played"] / 1000 / 60, 2)

to_omit = ["Wet Jeans", "No Laying Up - Golf Podcast", "Binchtopia", "History That Doesn't Suck",
           "HBO's The Last of Us Podcast", "The Analytics Power Hour", "S-Town",
           "Last Podcast On The Left", "Two Hot Takes", "The Shotgun Start", "DataFramed", "SmartLess",
           "Relaxing White Noise"]

top_artists = top_artists[(~ top_artists["artistName"].isin(to_omit)) & (top_artists["min_played"] >= 10)]

top_artists.head(n = 10).reset_index(drop = True).rename(columns = {"artistName": "Artist Name", "min_played": "Minutes Played"})
```

::: {.output .execute_result execution_count="143"}
```{=html}
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Artist Name</th>
      <th>Minutes Played</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>The Strokes</td>
      <td>3221.39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Radiohead</td>
      <td>3175.52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Taylor Swift</td>
      <td>2234.47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ethel Cain</td>
      <td>1720.60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Weyes Blood</td>
      <td>1675.30</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cigarettes After Sex</td>
      <td>1369.78</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Travis Scott</td>
      <td>1107.80</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Carly Rae Jepsen</td>
      <td>729.77</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Lana Del Rey</td>
      <td>677.91</td>
    </tr>
    <tr>
      <th>9</th>
      <td>The Voidz</td>
      <td>665.84</td>
    </tr>
  </tbody>
</table>
</div>
```
:::
:::

::: {.cell .markdown}
### Which months did I listen to the most music?
:::

::: {.cell .code execution_count="144"}
``` python
def get_month(ts):
    dt_obj = datetime.strptime(ts, "%Y-%m-%d %H:%M")
    return dt_obj.strftime("%B")

df["month"] = [get_month(x) for x in df["endTime"].tolist()]
df["month"] = pd.Categorical(df["month"], categories = ["January", "February", "March", "April", "May", "June",
                                                        "July", "August", "September", "October", "November", "December"])

df.groupby("month", observed = False).agg(min_played = ("msPlayed", "sum")).reset_index().assign(min_played = lambda x: round(x["min_played"] / 1000 / 60, 2)).sort_values("min_played", ascending = False).reset_index(drop = True)
```

::: {.output .execute_result execution_count="144"}
```{=html}
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>min_played</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>August</td>
      <td>5075.66</td>
    </tr>
    <tr>
      <th>1</th>
      <td>May</td>
      <td>4683.30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>September</td>
      <td>4494.58</td>
    </tr>
    <tr>
      <th>3</th>
      <td>July</td>
      <td>4291.78</td>
    </tr>
    <tr>
      <th>4</th>
      <td>January</td>
      <td>4201.04</td>
    </tr>
    <tr>
      <th>5</th>
      <td>April</td>
      <td>4189.81</td>
    </tr>
    <tr>
      <th>6</th>
      <td>December</td>
      <td>4027.46</td>
    </tr>
    <tr>
      <th>7</th>
      <td>October</td>
      <td>3752.71</td>
    </tr>
    <tr>
      <th>8</th>
      <td>June</td>
      <td>3731.13</td>
    </tr>
    <tr>
      <th>9</th>
      <td>March</td>
      <td>3611.68</td>
    </tr>
    <tr>
      <th>10</th>
      <td>November</td>
      <td>3130.24</td>
    </tr>
    <tr>
      <th>11</th>
      <td>February</td>
      <td>2788.81</td>
    </tr>
  </tbody>
</table>
</div>
```
:::
:::
