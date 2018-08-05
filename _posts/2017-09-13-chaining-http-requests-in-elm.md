---
layout: post
title: Chaining HTTP Requests In Elm
summary: Learn how to chain HTTP requests that depend on each other using Elm tasks
author: Tom Schaefer
author_url: http://teecom.com/people/tommy-schaefer/
tags: elm web http
---

In some situations, one HTTP request may be dependent on information from
another request. For example, you may need to get an API token and then fetch a
piece of information with that token. In cases like these, it can be useful to
model multiple requests as a single operation.

Thankfully, Elm has just the thing to help us out...

## Tasks

In Elm, [Tasks](http://package.elm-lang.org/packages/elm-lang/core/latest/Task)
model asynchronous operations that are not guaranteed to succeed. Tasks are a
part of Elm's core library and are an implementation of
["futures"](https://en.wikipedia.org/wiki/Futures_and_promises).

They are kind of interesting because they act less like functions that get
called and more like data that describes how an action should be performed.
Tasks are not called directly, but turned into
[commands](https://www.elm-tutorial.org/en/03-subs-cmds/02-commands.html) and
eventually executed by the Elm runtime.


### Chaining Tasks

Tasks can be chained using the
[andThen](http://package.elm-lang.org/packages/elm-lang/core/latest/Task#andThen)
function, which takes a callback and a task. The given task
will run and, if it is successful, its result will be given to the callback.

Take the following example:

```elm
succeed "hello"
    |> andThen (\h -> succeed (h ++ " world"))
```

If run, this task would succeed with the value `hello world`.

When the first task given to `andThen` fails, execution stops and the
callback is never fired.

In this example:

```elm
fail "error"
    |> andThen (\h -> succeed (h ++ " world"))
```

If run, the task would fail with the string `error` and the callback
would never fire.

## Taking HTTP Requests To Task

Elm's [HTTP package](http://package.elm-lang.org/packages/elm-lang/http/latest)
conveniently has a function,
[toTask](http://package.elm-lang.org/packages/elm-lang/http/1.0.0/Http#toTask),
which converts an `HTTP.Request` to a `Task.Task`.

In a typical use case, an HTTP request might look like this:

```elm
type Msg
    = RequestDone (Result Http.Error String)


update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        RequestDone (Ok _) ->
            model ! []

        RequestDone (Err _) ->
            model ! []


makeRequest : Cmd Msg
makeRequest =
    Http.getString "https://url.com"
        |> Http.send RequestDone
```

If we wanted to operate on the HTTP request as a task, we would only need to
change `makeRequest` like so:

```elm
...

makeRequest : Cmd Msg
makeRequest =
    Http.getString "https://url.com"
        |> Http.toTask
        |> Task.attempt RequestDone
```

## An Example Using The Elm Architecture

We have seen how `andThen` is used to chain tasks, and how `HTTP.Request`s
are converted to tasks with `toTask`. Let's put these ideas together by
chaining two HTTP requests.

In this example, we will use two APIs to display what country the
International Space Station is currently over. First, we will use the
[Open Notify ISS Current Location
API](http://open-notify.org/Open-Notify-API/ISS-Location-Now/) to get the
current position of the ISS. Next, we will use the [Nominatim Reverse Geocoding
API](http://wiki.openstreetmap.org/wiki/Nominatim#Reverse_Geocoding) to
determine what country the ISS is over.

### The Elm Architecture

[The Elm Architecture](https://guide.elm-lang.org/architecture/) is an Elm
pattern used to model and create applications that are easily extended and
refactored.

Using The Elm Architecture, the logic of every Elm program is broken up into
three distinct parts:

- The **model** is a representation of the application's current state.
- The **update** function responds to messages and modifies the application's
  state when necessary.
- The **view** defines how the application's state should be represented in
  HTML

### The Model

We will start by defining our model and a new type:

```elm
type alias Coordinates =
    { lat : String
    , lon : String
    }


type alias Model =
    { country : Maybe String
    }
```

The `Coordinates` type holds the positional data returned by the ISS API. The
`Model` type manages the application's internal state using information
returned by the reverse geocoding API.

### Messages And The Update Function

In this application, we have two messages:

```elm
type Msg
    = OnFetchISSLocation
    | FetchISSLocationDone (Result Http.Error String)
```

And the update function defines how our application responds to these
messages:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        OnFetchISSLocation ->
            model ! [ getISSLocation ]

        FetchISSLocationDone (Ok country) ->
            { model | country = Just country } ! []

        FetchISSLocationDone (Err _) ->
            { model | country = Just "the Earth (probably an Ocean)" } ! []
```

In the case of an `OnFetchISSLocation` message, the `getISSLocation` command
will be used to request the external APIs.

In the case of a successful `FetchISSLocationDone` message, the model will be
updated with the returned country name. If, however, the requests are
unsuccessful, the model will be updated to reflect this.

**Note:** One of the limitations of the reverse geocoding API is that it
returns an error when the provided corrdinates are not above a land mass.
For the sake of simplicity, it is assumed that any error encountered while
making these requests is due to this fact. We'll see this in action soon!

### Requesting The ISS's Location

The [Open Notify ISS Current Location
API](http://open-notify.org/Open-Notify-API/ISS-Location-Now/) responds with
data in the following format:

```json
{
    "message": "success",
    "timestamp": UNIX_TIME_STAMP,
    "iss_position": {
        "latitude": CURRENT_LATITUDE,
        "longitude": CURRENT_LONGITUDE
    }
}
```

We will start by creating a JSON decoder to extract the latitude and longitude
into our previously defined `Coordinates` type:

```elm
decodeCoordinates : Decode.Decoder Coordinates
decodeCoordinates =
    Decode.at [ "iss_position" ]
        (Decode.map2
            Coordinates
            (Decode.field "latitude" Decode.string)
            (Decode.field "longitude" Decode.string)
        )
```

Next, we define how the request is formed and convert the `HTTP.Request` to a
`Task.Task`:

```elm
getISSCoords : Task.Task Http.Error Coordinates
getISSCoords =
    Http.get "http://api.open-notify.org/iss-now.json" decodeCoordinates
        |> Http.toTask

```

### Reverse Geocoding The Coordinates

The [Nominatim Reverse Geocoding
API](http://wiki.openstreetmap.org/wiki/Nominatim#Reverse_Geocoding)
responds with data in the following form:

```json
{
    "place_id": "173409570",
    "licence": "Data © OpenStreetMap contributors, ODbL 1.0. http://www.openstreetmap.org/copyright",
    "osm_type": "relation",
    "osm_id": "339514",
    "lat": "38.9675925",
    "lon": "-0.1803423",
    "display_name": "Gandia, Safor, Valencia, Valencian Community, Spain",
    "address": {
        "city": "Gandia",
        "region": "Safor",
        "county": "Valencia",
        "state": "Valencian Community",
        "country": "Spain",
        "country_code": "es"
    },
    "boundingbox": [
        "38.94981",
        "39.0412816",
        "-0.2970744",
        "-0.1442421"
    ]
}
```

Like before, we will start by defining a decoder:

```elm
decodeCountry : Decode.Decoder String
decodeCountry =
    Decode.at [ "address" ]
        (Decode.field "country" Decode.string)
```

And then describe the request:

```elm
getCountry : Coordinates -> Task.Task Http.Error String
getCountry coords =
    Http.get
        ("http://nominatim.openstreetmap.org/reverse?format=json&lat="
            ++ coords.lat
            ++ "&lon="
            ++ coords.lon
            ++ "&zoom=18&addressdetails=1"
        )
        decodeCountry
        |> Http.toTask
```

### Chaining The Requests

We can now chain these two requests together with:

```elm
getISSLocation : Cmd Msg
getISSLocation =
    getISSCoords
        |> Task.andThen (\coords -> getCountry coords)
        |> Task.attempt FetchISSLocationDone
```

This command is run when `OnFetchISSLocation` messages are given to the
`update` function and it does the following:

1. Gets the current location of the ISS from an external service
2. Gets the country a set of coordinates belong to from an external service
3. Maps the result of this command to the `FetchISSLocationDone` message

## Putting It All Together

The full, working implementation of this example can be seen below:

```elm
module Main exposing (..)

import Html exposing (Html, a, div, h2, program, text)
import Html.Attributes exposing (style)
import Html.Events exposing (onClick)
import Http
import Json.Decode as Decode
import Task


main =
    Html.program
        { init = init
        , view = view
        , update = update
        , subscriptions = (\_ -> Sub.none)
        }



-- MODEL


type alias Coordinates =
    { lat : String
    , lon : String
    }


type alias Model =
    { country : Maybe String
    }


init : ( Model, Cmd Msg )
init =
    Model Nothing ! []



-- UPDATE


type Msg
    = OnFetchISSLocation
    | FetchISSLocationDone (Result Http.Error String)


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        OnFetchISSLocation ->
            model ! [ getISSLocation ]

        FetchISSLocationDone (Ok country) ->
            { model | country = Just country } ! []

        FetchISSLocationDone (Err _) ->
            { model | country = Just "the Earth (probably an Ocean)" } ! []



-- VIEW


view : Model -> Html Msg
view model =
    div
        [ style
            [ ( "align-items", "center" )
            , ( "display", "flex" )
            , ( "height", "100%" )
            , ( "justify-content", "center" )
            , ( "text-align", "center" )
            ]
        ]
        [ div []
            [ locationText model
            , a
                [ onClick OnFetchISSLocation
                , style
                    [ ( "background", "#d86c70" )
                    , ( "border-radius", "2px" )
                    , ( "color", "#fff" )
                    , ( "cursor", "pointer" )
                    , ( "display", "inline-block" )
                    , ( "margin-top", "40px" )
                    , ( "padding", "20px 30px" )
                    ]
                ]
                [ text "Get Location Of ISS" ]
            ]
        ]


locationText : Model -> Html Msg
locationText model =
    h2 []
        [ text
            ("The ISS is over: "
                ++ (Maybe.withDefault "the Earth" model.country)
            )
        ]



-- HTTP


getISSLocation : Cmd Msg
getISSLocation =
    getISSCoords
        |> Task.andThen (\coords -> getCountry coords)
        |> Task.attempt FetchISSLocationDone


getISSCoords : Task.Task Http.Error Coordinates
getISSCoords =
    Http.get "http://api.open-notify.org/iss-now.json" decodeCoordinates
        |> Http.toTask


getCountry : Coordinates -> Task.Task Http.Error String
getCountry coords =
    Http.get
        ("http://nominatim.openstreetmap.org/reverse?format=json&lat="
            ++ coords.lat
            ++ "&lon="
            ++ coords.lon
            ++ "&zoom=18&addressdetails=1"
        )
        decodeCountry
        |> Http.toTask


decodeCoordinates : Decode.Decoder Coordinates
decodeCoordinates =
    Decode.at [ "iss_position" ]
        (Decode.map2
            Coordinates
            (Decode.field "latitude" Decode.string)
            (Decode.field "longitude" Decode.string)
        )


decodeCountry : Decode.Decoder String
decodeCountry =
    Decode.at [ "address" ]
        (Decode.field "country" Decode.string)
```
