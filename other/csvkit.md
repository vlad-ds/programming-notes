# csvkit

https://csvkit.readthedocs.io/en/latest/

Python-written CLI tool to manipulate CSV data. 

```bash
pip install --upgrade csvkit
```

Convert Excel to CSV: 

```bash
in2csv SpotifyData.xlsx --sheet "Worksheet1_Popularity" > Spotify_Popularity.csv
```

Preview in console:

```bash
csvlook SpotifyData.csv
```

High level summary statistics:

```bash
csvstat Spotify_Popularity.csv
```

Filter by column: 

```bash
#prints columns
csvcut -n Spotify_MusicAttributes.csv
#select first and third column
csvcut -c 1,3 Spotify_MusicAttributes.csv
#select first and third column
csvcut -c "track_id","duration_ms" Spotify_MusicAttributes.csv 	
```

Filter by row:

```bash
#row(s) which matches "SOME_ID" on the track_id column
csvgrep -c "track_id" -m SOME_ID Spotify_Popularity.csv
```

`-m` is for exact matching. We can use `-r`  for fuzzy matching. 

**Caution**: `csvkit` indexes columns starting from 1! 

`csvstack` stacks rows from multiple CSV files with the same schema. 

```bash
csvstack Spotify_Rank6.csv Spotify_Rank7.csv > Spotify_AllRanks.csv
```

The `-g "Rank6","Rank7"` flag will add a new column (by default `group`) specifying the original source file for the row. Add `-n "source"` to name the group column. 

`sql2csv` allows us to query a variety of SQL databases. 

```bash
sql2csv --db "sqlite:///SpotifyDatabase.db" \ 
		--query "SELECT * FROM Spotify_Popularity" \
		> Spotify_Popularity.csv
```

`csvsql` allows us to apply SQL statements to one or more CSV files. It creates an in-memory SQL database that temporarily hosts the file being processed. It is suitable for small to medium files only.

```bash
csvsql --query "SELECT * FROM Spotify_MusicAttributes LIMIT 1" \
	Spotify_MusicAttributes.csv \
	| csvlook
	> output.csv
```

Remember that the SQL query has to be written in one line, with no breaks. If the query is long, you can first save it as a shell variable. 

Here's a workflow specifying a query, executing it on 2 CSV files and exporting the results to a new CSV. 

```bash
# Store SQL query as shell variable
sql_query="SELECT ma.*, p.popularity FROM Spotify_MusicAttributes ma INNER JOIN Spotify_Popularity p ON ma.track_id = p.track_id"

# Join 2 local csvs into a new csv using the saved SQL
csvsql --query "$sql_query" Spotify_MusicAttributes.csv Spotify_Popularity.csv \
	> Spotify_FullData.csv

# Preview newly created file
csvstat Spotify_FullData.csv
```

`csvsql` can also be used to update databases. 

```bash
#update database with the contents of CSV file
csvsql --db "sqlite:///SpotifyDatabase.db" \
	   --insert Spotify_MusicAttributes.csv
```

The `--no-inference-` flag will disable type inference. `--no-constraints` will generate a schema without length limits or null checks.

```bash
# Preview file
ls

# Upload Spotify_MusicAttributes.csv to database
csvsql --db "sqlite:///SpotifyDatabase.db" --insert Spotify_MusicAttributes.csv

# Store SQL query as shell variable
sqlquery="SELECT * FROM Spotify_MusicAttributes"

# Apply SQL query to re-pull new table in database
sql2csv --db "sqlite:///SpotifyDatabase.db" --query "$sqlquery" 
```





