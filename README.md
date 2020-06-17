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

Being a proxy alone is not that useful, however when combining it with the record function, the incoming requests are saved as stubs and can be accessed later if, for instance, the xkcd API becomes unavailable. Another use case would be accessing the saved request/responses in a Integration Test environment to make the tests run faster and more reliable since an outside network connectin is not used.

For that rerun the application with `--record-mappings`:

```bash
java -jar wiremock-standalone-2.26.3.jar --port 8765 --proxy-all="https://xkcd.com" --record-mappings
```

Now execute

`GET http://localhost:8765/2319/info.0.json`

You should get his answer:

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

However in this time this request/response tuple was saved by Wiremock under `./mappings` and `__files`. Two files were created:

`__files/body-2319-info.0.json-YX439.json`

```json
{
    "month": "6",
    "num": 2319,
    "link": "",
    "year": "2020",
    "news": "",
    "safe_title": "Large Number Formats",
    "transcript": "",
    "alt": "10^13.4024: A person who has come back to numbers after a journey deep into some random theoretical field",
    "img": "https://imgs.xkcd.com/comics/large_number_formats.png",
    "title": "Large Number Formats",
    "day": "12"
}
```

and `mappings/mapping-2319-info.0.json-YX439.json`:

```json
{
  "id" : "2c7eb2e8-6837-3db3-bad3-2fb0a0a80d3c",
  "request" : {
    "url" : "/2319/info.0.json",
    "method" : "GET"
  },
  "response" : {
    "status" : 200,
    "bodyFileName" : "body-2319-info.0.json-YX439.json",
    "headers" : {
      "Server" : "nginx",
      "Content-Type" : "application/json",
      "Last-Modified" : "Wed, 17 Jun 2020 04:00:08 GMT",
      "ETag" : "W/\"5ee99548-15d\"",
      "Expires" : "Wed, 17 Jun 2020 05:49:06 GMT",
      "Cache-Control" : "max-age=300",
      "Accept-Ranges" : "bytes",
      "Date" : "Wed, 17 Jun 2020 07:29:21 GMT",
      "Via" : "1.1 varnish",
      "Age" : "0",
      "X-Served-By" : "cache-hhn4083-HHN",
      "X-Cache" : "HIT",
      "X-Cache-Hits" : "1",
      "X-Timer" : "S1592378961.372495,VS0,VE111",
      "Vary" : "Accept-Encoding"
    }
  },
  "uuid" : "2c7eb2e8-6837-3db3-bad3-2fb0a0a80d3c"
}
```

> The name of the files will vary, and the body files might not be formatted.

Now let's simulate an integration test environment where no network connection to the outside should be made. For that we can edit our `/etc/hosts` file to point the xkcd.com domain to somewhere else.

```txt
127.0.0.1 xkcd.com
```

Then restart the wiremock application: `java -jar wiremock-standalone-2.26.3.jar --port 8765 --proxy-all="https://xkcd.com" --record-mappings`

The xkcd API should now be unavaiable in the local machine:

`GET https://xkcd.com/2319/info.0.json` should return "Connection Refused"

However through the WireMock we can still access the recorded request/response map:

`GET http://localhost:8765/2319/info.0.json`

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

Any other request to Wiremock will fail with HTTP Code 500 since the underlying API is unreachable.
