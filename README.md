# Netflix Shakti API

# Table of Contents
- [Overview/Getting Started](#overviewgetting-started)  
  - [Basic Request](#basic-request)  
  - [Building The Path](#building-the-path)  
- [Documentation](#documentation)  
  - [Types](#types)  
    - [Genres](#genres)
    - [Videos](#videos)
    - [Seasons](#seasons)
    - [LoLoMos](#lolomos)
  - [Quantifiers](#quantifiers)  

# Overview/Getting Started
*NOTE*: This writeup assumes you have a Netflix account, and have written/can write the code to send the requests yourself. This an overview of how to use Shakti, and documentation thereof. 

If you have questions, comments, or concerns feel free to drop me an email: howard@getcoffee.io

## Basic Request
Most, if not all, of Netflix's data is fetched through their API called "Shakti". The request path looks like so:
```
POST /api/shakti/$VALUE$/pathEvaluator?method=call&withSize=true&materialize=true&searchAPIV2=false
```
Where `$VALUE$` is the 'build identifier' (presumably the current Shakti build id).

Assuming you are sending this while logged in on Netflix page, either via a bookmarklet or simply the developer console, the headers are also simple: 
```
        'Accept': 'application/json, text/javascript, */*',
        'Accept-Language': 'en-US,en;q=0.8',
        'Content-Type': 'application/json',
        'Content-Size': PATH SIZE IN BYTES,
        'Cookies': ...
```
Since I couldn't determine any method for generating the cookies, I'm continuing with the assumption that you have the cookies, whether that be by sending the request in a browser that has the cookies or otherwise. I accomplish this through a headless browser. You may notice that the content size mentions the "path". This is the core of our request, and is the body. This is how we're going to actually specify what data we want. This comes later.

The body format is comprised of two components:
```
{
    'authURL': ...,
    'paths': [...],
}
```
The authURL is stored in a globally accessible object through the following: `netflix.reactContext.models.userInfo.data.authURL`. That's the easy part. The path is where we get into the thick of it.

## Building The Path
The paths component is an array of all the different pieces you want. Each piece is an array which represents one "type" and the attributes of that type you wish to retrieve. The attributes vary between types, however the path will *usually* follow the structure below:
```
...
    'paths': [
        [ // first piece
            "type",
            "ID",
            "attribute 1",
            "attribute 2",
        ],
        [ // second piece, etc.
            ...
        ]
    ]
...
```
Every path piece contains at least a type, an ID, and one or more attributes. When you send a query for a basic path, the server responds with the attributes of the type-instance that matches the ID.

We can use the show "Marvel's Agents of Shield" as an example, which has an ID of `70279852`. Shakti uses the type `video` for both movies and TV shows and distinguishes between them via attributes. To get the name of the show, the path would look like so:
```
...
    'paths': [
        [
            "video",
            "70279852",
            "name",
        ],
    ]
...
```

More complicated paths will use selectors and selector ranges. 
```
...
    'paths': [
        [ // first piece
            "type",
            "ID",
            "selector",     // Not always required, Application dependent 
            {               // Selector Range
                "from": -1,
                "to":   25,
            },
            "attribute 1",    // Can also look like ["attribute 1", "attribute 2", ...],
            "attribute 2",
        ],
        [ // second piece, etc.
        ...
        ]
    ]
...
```

# Documentation

## Types

### Genres

#### Basic Attributes
| Name      | Description             | Response Type | Example |
| --------- | ----------------------- | ------------- | ------- |
| id        | ID                      | int           | 26065   |
| length    | # of *Videos*           | int           |         |
| name      | Name                    | string        |         |
| menuName  | Name in Menus           | string        |         |
| subgenres | Subgenres               | Object        |         |
| summary   | Basic Genre Information | Object        |         |
| trackIds  | *N/A*                   | Object        |         |
| requestId | *N/A*                   | string        |         |
| rw        | *N/A*                   | Array         |         |

#### Relative Attributes

| Name | Description | Response Type |

| Attribute | Description | Notes |
| --------- | ----------- | ----- |
| id        | Genre ID | - |
| length    | Number of shows | - |
| name      | Name of the genre, i.e. "Exciting TV Shows" | - |
| trackIds  | N/A | - |
| requestId | N/A | It's used with `su` frequently. |
| su        | Must include a quantifier afterwards. Any attributes following the quantifier must be `videos` attributes | See genres#requestId |

### Videos
| Attribute               | Description | Notes |
| ----------------------- | ----------- | ----- |
| availability            | Can it be played | - |
| availabilityEndDateNear | Is the show leaving netflix soon? | - |
| bookmarkPosition        | N/A | - |
| cast                    | Cast persons | - |
| commonSense             | What does the maturity rating mean? | - |
| copyright               | Copyright | - |
| delivery                | Playback features such as 4k | - |
| episodeCount            | Number of episodes the show has | - |
| festivals               | N/A | - |
| genres                  | N/A | - |
| info                    | N/A | - |
| maturity                | Rating information | - |
| numSeasonLabel          | Plaintext for how many seasons there are | - |
| queue                   | Is it in the user's queue? | - |
| recentInterestingMoment | N/A | - |
| regularSynopsis         | More useable, readable synopsis | - |
| releaseYear             | Release year | - |
| runtime                 | N/A | - |
| seasonCount             | Integer for how many seasons there are | - |
| seasonList              | List of seasons | Must be called with a  |
| similars                | Shows that are similar | - |
| summary                 | Cursory show information | - |
| synopsys                | Synopsis | - |
| title                   | Title | - |
| trailers                | N/A | - |
| userRating              | Information on the rating for the user and by the user | - |
| watched                 | Has user watched? | - |
| writers                 | List of writers | - |

### Seasons

| Attribute | Type |Description | Notes |
| --------- | ---- | ----------- | ----- |
| id | int | Season ID | - |
| name | string | Full Season Name, i.e. Season 1 | - |
| shortName | string | Season Name Shorthand, i.e. S1 | - |
| hiddenEpisodeNumbers | bool | N/A | - |
| length | int | # of episodes | - |
| episodes | map | List of episodes | - | 

### LoLoMos

This is the most unintuitive type by name. LoLoMos is short for a "List of List of Movies" *(https://twitter.com/arungupta/status/624402051116568576)*

### Person
#### Basic Attributes
| Name         | Description                | Response Type | Example       |
| ------------ | -------------------------- | ------------- | ------------- |
| id           | Person ID                  | int           | 20055888      |
| headshot     | Photo resource description | Object        | N/A           |
| length       | # of *Videos* involved in  | int           | 4             |
| name         | Name                       | string        | "Terry Crews" |
| requestId    | N/A                        | string        | N/A           |
| trackIds     | N/A                        | Object        | N/A           |
| (0...length) | Video ID                   | Array         | N/A           |


## Quantifiers



