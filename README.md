# OMDB
For use in LICENCED *openmusic-compatible* servers, and for educational purposes.

OMDB (OpenMusic Database) is the worlds largest openly downloadable database with metadata for over 145 million songs, 24 million albums, and 5 million artists.

[Download the OMDB from OneDrive here]().

## How To Use
`fulldb.tar` is a dump of a PostgreSQL 15 database. To use it, you must first create an empty PostgreSQL 15 database, and then restore using the file.

### Creating a Database
I recommend using a GUI such as [pgAdmin](https://www.pgadmin.org/), or you can use the terminal.

This will create a database called fulldb, assuming PostgreSQL is installed in `/Library`:
```
/Library/PostgreSQL/15/bin/createdb -W fulldb
```

### Restoring via fulldb.tar
Next, use this command to write fulldb.tar to the database fulldb:
```
/Library/PostgreSQL/15/bin/pg_restore -W -d fulldb < fulldb.tar
```
On my Macbook Pro (M1 Pro 2021), this step takes about 3 hours with this file. Once restored, you can query the database via pgAdmin, or a library such as [pg (Node JS)](https://www.npmjs.com/package/pg) or [psycopg2 (Python)](https://pypi.org/project/psycopg2/). Your first step should be to run "ANALYZE" to speed up full-text search.


## Database Schema

### Artists
- **Description**: Stores information about artists.
- **Columns**:
  - `ID`: TEXT, Primary Key
  - `Name`: TEXT
  - `Profile_Photo`: TEXT
  - `Subscribers`: INT

### Albums
- **Description**: Stores information about albums.
- **Columns**:
  - `ID`: TEXT, Primary Key
  - `Title`: TEXT
  - `Artwork`: TEXT
  - `Type`: TEXT
  - `Year`: INT

### Playbacks
- **Description**: Stores playback data, including the audio and video YouTube ID.
- **Columns**:
  - `ID`: SERIAL, Primary Key
  - `Audio_YTID`: TEXT
  - `Video_YTID`: TEXT
  - `Audio_Live`: TEXT
  - `Is_Explicit`: INT

### Tracks
- **Description**: Stores information about tracks, including indexed search vectors and trigram indexes.
- **Columns**:
  - `ID`: TEXT, Primary Key
  - `Playback_Clean`: INT, References `Playbacks(ID)`
  - `Playback_Explicit`: INT, References `Playbacks(ID)`
  - `MXID`: TEXT
  - `Title`: TEXT
  - `Runtime_Seconds`: INT
  - `Views`: INT
  - `Album_Index`: INT
  - `Album`: TEXT, References `Albums(ID)`
  - `TitleTrgm`: TEXT
  - `AlbumTrgm`: TEXT
  - `ArtistsTrgm`: TEXT
  - `title_vector`: tsvector
  - `album_vector`: tsvector
  - `artist_vector`: tsvector
  - `combined_vector`: tsvector

### Album_Relation
- **Description**: Defines an Album's base album. "Album 1" and "Album 1 (Deluxe)" have equal baseIDs.
- **Columns**:
  - `relatedID`: TEXT
  - `baseID`: TEXT

### Artist_Album
- **Description**: Relationship between artists and albums.
- **Columns**:
  - `Artist_ID`: TEXT, References `Artists(ID)`
  - `Album_ID`: TEXT, References `Albums(ID)`
  - Composite Primary Key: (`Artist_ID`, `Album_ID`)

### Artist_Track
- **Description**: Relationship between artists and tracks.
- **Columns**:
  - `Artist_ID`: TEXT, References `Artists(ID)`
  - `Track_ID`: TEXT, References `Tracks(ID)`
  - Composite Primary Key: (`Artist_ID`, `Track_ID`)

### Features_Track
- **Description**: Relationship between featured artists and tracks.
- **Columns**:
  - `Artist_ID`: TEXT, References `Artists(ID)`
  - `Track_ID`: TEXT, References `Tracks(ID)`
  - Composite Primary Key: (`Artist_ID`, `Track_ID`)
