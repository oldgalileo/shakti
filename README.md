# Netflix Shakti API

# Setting Up The Request
All of Netflix's data is fetched through this API they have developed called "Shakti". Almost all of these requests have a pretty straightforward form:
```
POST /api/shakti/VALUE/pathEvaluator?method=call&withSize=true&materialize=true&searchAPIV2=false
```
As of right now, I'm unclear as to how the value is generated. I merely watch for the first `pathEvaluator` request and grab the value. Assuming you are sending this while logged in on Netflix page, either via a bookmarklet or simply the developer console, the headers are also simple: 
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

# Building The Path
The paths component is an array of all the different pieces you want. Each piece itself is an array, and is broken down into a few pieces. These differ based on what particular information you want. This is what it looks like so far
```
...
    'paths': [
        [               // first piece
            "type",
            "ID",
            "quantifier",   // Situational, 
            "attribute",    // Can also look like ["attribute 1", "attribute 2", ...],
        ],
        [               // second piece, etc.
        ...
        ]
    ]
...
```
This is the basic form of the path. We have our type, an ID for that type, and then what attributes or information we would like to retrieve from the specific type#ID. There can also be a quantifier, which defines how many entries you want in return. This is situational, and isn't included with every request like the other 3 are. 

## Type
There are 4 types for a path piece:
- Genres
- Videos *(Synonymous with 'Show')*
- Seasons
- Person
- Lolomos *(List of List of Movies, https://twitter.com/arungupta/status/624402051116568576)*

Each of these has an ID, as well as some allowed attributes.

## Attributes
This is the good stuff. Each type has its own set of attributes. 

### Genres

| Attribute | Description | Notes |
| --------- | ----------- | ----- |
| id        | ID of the genre in question | Seems redundant. |
| length    | Number of shows | - |
| name      | Name of the genre, i.e. "Exciting TV Shows" | - |
| trackIds  | UNKNOWN | - |
| requestId | UNKNOWN | It's used with `su` frequently. |
| su        | Must include a quantifier afterwards. Any attributes following the quantifier must be `videos` attributes | Weird stuff with requestID? |

### Videos
| Attribute | Description | Notes |
| --------- | ----------- | ----- |
| availability | Can it be played | - |
| availabilityEndDateNear | Is the show leaving netflix soon? | - |
| bookmarkPosition | UNKNOWN | - |
| cast | Cast persons | - |
| commonSense | What does the maturity rating mean? | - |
| copyright | Copyright | - |
| delivery | Playback features such as 4k | - |
| episodeCount | Number of episodes the show has | - |
| festivals | UNKNOWN | - |
| genres | UNKNOWN | - |
| info | UNKNOWN | - |
| maturity | Rating information | - |
| numSeasonLabel | Plaintext for how many seasons there are | - |
| queue | Is it in the user's queue? | - |
| recentInterestingMoment | UNKNOWN | - |
| releaseYear | Release year | - |
| runtime | UNKNOWN | - |
| seasonCount | Integer for how many seasons there are | - |
| seasonList | UNKNOWN | - |
| similars | Shows that are similar | - |
| summary | Cursory show information | - |
| synopsys | Synopsis | - |
| title | Title | - |
| trailers | UNKNOWN | - |
| userRating | Information on the rating for the user and by the user | - |
| watched | Has user watched? | - |
| writers | List of writers | - |

