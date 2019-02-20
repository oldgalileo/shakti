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
  - [Selectors](#selectors)  

# Overview/Getting Started
*NOTE*: This writeup assumes you have a Netflix account, and have written/can write the code to send the requests yourself. This an overview of how to use Shakti, and documentation thereof. 

If you have questions, comments, or concerns feel free to drop me an email: howard@getcoffee.io

## Basic Request
Most, if not all, of Netflix's data is fetched through their API called "Shakti". The request path looks like so:
```
POST /api/shakti/$VALUE$/pathEvaluator
```
Where `$VALUE$` is the 'build identifier' (presumably the current Shakti build id). This value can be found in a globally accessible object via the following: `netflix.appContext.state.model.models.serverDefs.data.BUILD_IDENTIFIER`

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

We can use the show "Marvel's Agents of Shield" as an example, which has an ID of `70279852`. Shakti uses the type `videos` for both movies and TV shows and distinguishes between them via attributes. To get the name of the show, the path would look like so:
```
...
    'paths': [
        [
            "videos",
            "70279852",
            "title",
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

#### Attributes
| Name      | Description             | Response Type | Example             |
| --------- | ----------------------- | ------------- | ------------------- |
| id        | ID                      | int           | 26065               |
| length    | # of *Videos*           | int           | 75                  |
| name      | Name                    | string        | "Exciting TV Shows" |
| menuName  | Name in menus           | string        | "Exciting TV Shows" |
| subgenres | Subgenres               | Object        | N/A                 |
| summary   | Basic genre information | Object        | ...                 |
| su        | Video ID                | Array         | ...                 |
| trackIds  | *N/A*                   | Object        | ...                 |
| requestId | *N/A*                   | string        | N/A                 |
| rw        | *N/A*                   | Array         | ...                 |

### Videos

This is the most dynamic and complex of the broader types. Unlike the rest, the `videos` type has **three *implicit* types** which can alter the response: movies, shows, and episodes.  

As of right now, the subtypes are not properly reflected below and their attributes poorly differentiated. It's unclear the best way to format this, and will be revised in the future.

#### Attributes
|Name                   |Description            |Response Type|Example        |
|-----------------------|-----------------------|-------------|---------------|
|id                     |ID                     |int          |70136120       |
|availability           |Playable now           |Object       |...            |
|availabilityEndDateNear|Leaving soon           |N/A          |N/A            |
|bookmarkPosition       |Where user left off    |int          |-1             |
|boxArts                |URL to the box art (usually in landscape). This is just a key required in the request. It's then followed by one of the following keys: (_342x192, _550x124, _665x375, _1280x720). It also requires a key that defines the type. The current keys found are: "webp" and "jpg".|Object|...|
|cast                   |Cast                   |Array        |...            |
|copyright              |N/A                    |N/A          |N/A            |
|creditsOffset          |When credits start     |int          |1353           |
|current                |Current episode        |Array        |...            |
|delivery               |Viewing Options (4k...)|Object       |...            |
|directors              |Directors              |Array        |...            |
|dpSupplementalMessage  |N/A                    |string       |""             |
|episodeBadges          |N/A                    |Array        |...            |
|episodeCount           |# of episodes          |int          |192            |
|evidence               |Reason to watch show   |Object       |...            |
|hasSensitiveMetadata   |N/A                    |boolean      |false          |
|isNSRE                 |N/A                    |boolean      |false          |
|isOriginal             |Netflix Original Series|boolean      |false          |
|maturity               |Maturity info          |Object       |...            |
|numSeasonsLabel        |Readable # of seasons  |string       |"9 Seasons"    |
|queue                  |In user queue          |Object       |...            |
|regularSynopsis        |Meta synopsis          |string       |"This hit c..."|
|releaseYear            |Year released          |int          |2012           |
|requestId              |N/A                    |string       |N/A            |
|runtime                |Movie/Show runtime     |int          |1382           |
|seasonCount            |# of seasons           |int          |9              |
|seasonList             |Seasons                |Array        |...            |
|summary                |Basic video information|Object       |...            |
|synopsis               |Synopsis               |string       |"The boss i..."|
|tallBoxarts            |URL to the Box art in the same aspect ratio as the DVD cover art|Object|...|
|title                  |Title                  |string       |"The Office..."|
|userRating             |User rating info       |Object       |N/A            |
|watched                |Already watched        |bool         |false          |
|writers                |Writers                |Array        |...            |

### Seasons

| Name                 | Description    | Response Type | Example    |
| -------------------- | -------------- | ------------- | ---------- |
| id                   | ID             | int           | 70023522   |
| hiddenEpisodeNumbers | N/A            | bool          | false      |
| length               | # of Episodes  | int           | 6          |
| name                 | Readable name  | string        | "Season 1" |
| shortName            | Shortened name | string        | "S1"       |
| episodes             | Episodes       | Object        | N/A        |

### LoLoMos

This is the most unintuitive type by name. LoLoMos is short for a "List of List of Movies" *(https://twitter.com/arungupta/status/624402051116568576)*

### Person
#### Attributes
| Name         | Description                | Response Type | Example       |
| ------------ | -------------------------- | ------------- | ------------- |
| id           | Person ID                  | int           | 20055888      |
| headshot     | Photo resource description | Object        | N/A           |
| length       | # of *Videos* involved in  | int           | 4             |
| name         | Name                       | string        | "Terry Crews" |
| requestId    | N/A                        | string        | N/A           |
| trackIds     | N/A                        | Object        | N/A           |
| (0...length) | Video ID                   | Array         | N/A           |

## Selectors



