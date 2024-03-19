# OMDB
To aid creating [*Openmusic-Compatible* servers](https://github.com/OatsCG/Openmusic-Server-Specs), and for educational purposes.

OMDB (Openmusic Database) is the worlds largest openly downloadable database with metadata for over 154 million songs, 28 million albums, and 5 million artists. (**4x larger** than MusicBrainz, ~same as Discogs [<sup>[1]</sup>](https://en.wikipedia.org/wiki/List_of_online_music_databases))

### [Download the OMDB from OneDrive here.](https://utoronto-my.sharepoint.com/:u:/g/personal/charlie_giannis_mail_utoronto_ca/Ecw5drWAQYlCuJoYwM-XULsBm-7JJK4FwaxdSMum6I9fWQ?e=lFEWFO)
> fulldb.tar 72.1 GB -> 175 GB, Created Jan 24 2024, Expires Apr 18 2024
>> 154,331,239 tracks, 28,046,579 albums, 5,779,932 artists, 154,494,422 playbacks
#### [Download the Lite OMDB (Album.Tracks[0].Views > 1000) from OneDrive here.](https://utoronto-my.sharepoint.com/:u:/g/personal/charlie_giannis_mail_utoronto_ca/ETUyM6_IZ-xIhXwLtL1o0f0BAsLEQzrIP3nAtpzQ_xO0bg?e=vEuean)
> litedb.tar 12.0 GB -> 31 GB, Created Mar 19 2024, Expires Apr 18 2024
>> 21,221,507 tracks, 4,463,915 albums, 6,301,744 artists, 21,275,551 playbacks

### Copyright Notice
The OMDB is for research purposes only. Public use of the database is permitted, however it is your responsibility to ensure that copyright laws for your country are followed. Failure to do so may result in legal pursuit by the owners of the content you are infringing on. Research your country's copyright laws before proceeding.


## How To Use
`fulldb.tar` is a dump of a [PostgreSQL 15](https://www.postgresql.org/download/) database. To use it, you must first create an empty PostgreSQL 15 database, and then restore using the tar file.

DO NOT unzip the tar file. For that matter, do not unzip/open any file downloaded from the internet, unless you're using the proper tools as shown below.

`fulldb.tar` restores into a database that is **175 GB**, and `litedb.tar` into **27 GB**. Make sure you have sufficient storage before proceeding.

### Creating a Database
I recommend using a GUI such as [pgAdmin](https://www.pgadmin.org/), or you can use the terminal.

This will create an empty database called fulldb, assuming PostgreSQL is installed in `/Library`:
```
/Library/PostgreSQL/15/bin/createdb -W fulldb
```

### Restoring via fulldb.tar
Next, use this command to write fulldb.tar to the database fulldb:
```
/Library/PostgreSQL/15/bin/pg_restore -W -d fulldb < fulldb.tar
```
On my Macbook Pro (M1 Pro 2021), this step takes about 4 hours with this file. Once restored, you can query the database via pgAdmin, or a library such as [pg (Node JS)](https://www.npmjs.com/package/pg) or [psycopg2 (Python)](https://pypi.org/project/psycopg2/). Your first step should be to run "ANALYZE" to speed up full-text search.


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
