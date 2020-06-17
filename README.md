# Sample project with wiremock-standalone

See <http://wiremock.org/docs/running-standalone/>

## Requisites

1. Install Java 8 or newer for your Operating System
2. Download the [wiremock-standalone .jar](https://repo1.maven.org/maven2/com/github/tomakehurst/wiremock-standalone/2.26.3/wiremock-standalone-2.26.3.jar)
3. You can test the xkcd's API and wiremock with your Browser itself since they are all GET requests.

## Mocked API is Xkcd's API

Api URL: <https://xkcd.com/json.html>

Current comic metadata: `GET https://xkcd.com/info.0.json`
Search comic metadata by comic number: `GET https://xkcd.com/2319/info.0.json`

Sample requests and the responses:

`GET https://xkcd.com/info.0.json`

```json
{
    "alt": "The hard part about opening a hole in the proof of the Poincaré conjecture is that Grigori Perelman will come out of retirement to try to fix it by drawing a loop around the hole and contracting it to a point.",
    "day": "15",
    "img": "https://imgs.xkcd.com/comics/millennium_problems.png",
    "link": "",
    "month": "6",
    "news": "",
    "num": 2320,
    "safe_title": "Millennium Problems",
    "title": "Millennium Problems",
    "transcript": "",
    "year": "2020"
}
```

`GET https://xkcd.com/2319/info.0.json`

```json
{
    "alt": "10^13.4024: A person who has come back to numbers after a journey deep into some random theoretical field",
    "day": "12",
    "img": "https://imgs.xkcd.com/comics/large_number_formats.png",
    "link": "",
    "month": "6",
    "news": "",
    "num": 2319,
    "safe_title": "Large Number Formats",
    "title": "Large Number Formats",
    "transcript": "",
    "year": "2020"
}
```

## Accesing the xkcd's API through WireMock Standalone

You can use the local wiremock instance as a proxy:

```bash
java -jar wiremock-standalone-2.26.3.jar --port 8765 --proxy-all="https://xkcd.com"
```

`GET http://localhost:8765/info.0.json` will return the same as `GET https://xkcd.com/info.0.json`:

```json
{
    "alt": "The hard part about opening a hole in the proof of the Poincaré conjecture is that Grigori Perelman will come out of retirement to try to fix it by drawing a loop around the hole and contracting it to a point.",
    "day": "15",
    "img": "https://imgs.xkcd.com/comics/millennium_problems.png",
    "link": "",
    "month": "6",
    "news": "",
    "num": 2320,
    "safe_title": "Millennium Problems",
    "title": "Millennium Problems",
    "transcript": "",
    "year": "2020"
}
```
