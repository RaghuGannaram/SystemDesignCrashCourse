# Spotify System design

## 1. Functional requirements:

-   Application should serve substantial number of songs/music from various sources (Movies, TV shows, Private albums) accross the world.
-   User can search for any song from the home page. He can narrowdown the search results with filters.
    -   Based on the source of the music eg. Movie songs/music, TV show songs/music, private albums etc.
    -   Based on the artist of the song/music.
    -   Based on the language eg. English, Chinese, Spanish, Hindi etc.
    -   Based on the origin of the song/music eg. India, Germany, USA etc.
-   User can play any song/music he selects.
-   User can further register to the application, so that he can login to the application to customize the usage.
    -   He can save a particular song.
    -   He can like a song (or dislike).
    -   He can create his custom playlists.
    -   He can follow an artist (So that, each time an artist uploads a music number, the user can be notified).
    -   He can further turn himself into an artist and upload his music creations

## 2. Non functional requirements:

-   The application is read heavy, as there are lot of users listening to the songs/music than upload songs/music at any point of time.
-   The application should be highly available.
-   The application can be eventual consistent.

## 3. Back of the envelop estimations:

-   **Assumptions**

        -   Application DAU:                                                20 million
        -   Songs on the application:                                       100 million
        -   Songs listened by user per day:                                 10
        -   size of each song:                                              5MB
        -   size of each song's metadata:                                   1KB
        -   size of user profile (incl of user interaction/preferences):    10KB

-   **Calculations**

        -   Number of songs listened per day:                               20 million x 10 = 200 million/day
                                                                            2,320/second
        -   Total daily data transfer for songs:                            200 million x 5MB = 1PB/day
                                                                            11.5TB/second
        -   Total daily data transfer for metadata:                         200 million x 1KB = 200GB
                                                                            2.3MB/second
