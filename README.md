# enterprise-scripts-python

Sample Python code that makes it easy to get started with the Twitter Enterprise APIs. These are intended to be simple, easy-to-use, command line scripts for anyone familiar with Python to begin making requests to the APIs.

## Features

- Customize requests by passing any of the supported flags (parameters) when invoking the scripts
- The `search.py` script supports both 'data' and 'counts' (`-c`) requests as well as auto pagination (`-n` flag)
- The PowerTrack `get_stream.py` script supports a custom chunksize (`-c` flag) – useful when testing low volume streams
- Generate a bearer token for use with the /totals endpoint of the Engagement API (`/Engagement-API/generate_bearer_token.py`)
- Generate user access tokens to authorize requests to the Engagement API and access user-owned private  metrics (`/Engagement-API/generate_user_access_tokens.py`)
- Supports pretty printing the results either by default or through an optional flag (`-p`)

## Dependencies

- Python (recommended >= 3.6 for f-string support)
- Twitter Enterprise API Account (e.g., console.gnip.com access)
- Third-party libraries (see 'requirements.txt' for all library dependencies):
  - requests
  - requests_oauthlib
  - python-dotenv

Run `pip install -r requirements.txt` to install the third-party library dependenices.

## Supported APIs

Below are the API that are currently supported by this collection of scripts (click product name to jump to that section of README):

1. [Engagement API](#engagement-api) - Get detailed impression and engagement metrics by Tweet ID ([link to docs](https://developer.twitter.com/en/docs/metrics/get-tweet-engagement/overview))
2. [Historical PowerTrack](#historical-powertrack) - Get batch historical Tweets ([link to docs](https://developer.twitter.com/en/docs/tweets/batch-historical/overview))
3. [PowerTrack](#powertrack) - Filter realtime Tweets ([link to docs](https://developer.twitter.com/en/docs/tweets/filter-realtime/overview/powertrack-api))
4. [Search API](#search-api-30-day-and-full-archive) - 30-Day and Full-Archive APIs to search for Tweets ([link to docs](https://developer.twitter.com/en/docs/tweets/search/overview/enterprise))

## Authorization and Authentication

These scripts are built to work with the paid, Enterprise tier of Twitter APIs. To use the scripts, you must have authorization to access the Enterprise APIs as part of a "trial" or ongoing contract. If you have access to the [Gnip console](console.gnip.com) and one or more of the APIs listed above, then you should be set. Otherwise, you can [apply for access here](https://developer.twitter.com/en/products/twitter-api/enterprise/application).

_Note:_ PowerTrack, Historical PowerTrack, and Search use basic authentication (username, password). The Engagement API requires your Twitter app to be whitelisted and uses a mix of 3-legged Oauth (user-context auth) and app-only auth / bearer token.

## Configuration

Create a file named ".env" at the root of the repository directory with the relevant credentials (see '.env.example'). Here's the example '.env' for reference:

```shell
# Account creds for PowerTrack, Historical PowerTrack, and Search API
USERNAME=""
PASSWORD=""
ACCOUNT_NAME=""
POWERTRACK_LABEL=""
SEARCH_LABEL=""
SEARCH_ARCHIVE="" # 'fullarchive' or '30day'

# Twitter app creds for Engagement API
TWITTER_CONSUMER_KEY=""
TWITTER_CONSUMER_SECRET=""
TWITTER_ACCESS_TOKEN=""
TWITTER_ACCESS_TOKEN_SECRET=""
TWITTER_BEARER_TOKEN=""
```

### Authenticating with the Engagement API 

Two authentication methods are available with the Engagement API: [OAuth 1.0a](https://developer.twitter.com/en/docs/tutorials/authenticating-with-twitter-api-for-enterprise/authentication-method-overview#oauth1.0a) and [OAuth 2.0 Bearer Token](https://developer.twitter.com/en/docs/tutorials/authenticating-with-twitter-api-for-enterprise/authentication-method-overview#oauth2.0).

**OAuth 2.0 Bearer Token** allows you to access publicly available engagement metrics. This authentication method can be used to get total counts for Favorites (aka Likes), Retweets, Quote Tweets, Replies, and video views for any publicly available Tweets when making requests to the /totals endpoint. 

→ Use the `/Engagement-API/generate_bearer_token.py` script to generate the required bearer token. You can then store that returned value in the '.env' file as shown above.

**OAuth 1.0a** allows you to make requests on behalf of a user and access private engagement metrics that are owned by the user in question. 

This authentication method is required for:
* All requests sent to the /28hr endpoint and /historical endpoint
* Accessing any of the following private metrics: Impressions, Engagements, Media Views, Media Engagements, URL Clicks, Hashtag Clicks, Detail Expands, Permalink Clicks, App Install Attempts, App Opens, Email Tweet, User Follows, and User Profile Clicks

→ Use the `/Engagement-API/generate_user_access_tokens.py` script to get a user to authorize your application and generate the required Access Tokens (user's access token and access token secret). You can then store the returned Access Tokens, along with your developer App's consumer key and secret in the '.env' file as shown above.

Note: If you are sending a request on behalf of your own Twitter account (in other words, the account that owns your developer App), you can generate the required Access Tokens directly from within the [developer portal](https://developer.twitter.com/en/portal/projects-and-apps).

[Find out more](https://developer.twitter.com/en/docs/twitter-api/enterprise/engagement-api/guides/authenticating-with-the-engagement-api) about authenticating with the Engagement API.

## Engagement API

Pass one or more Tweet IDs to get a variety of impression and engagement metrics.

### Totals endpoint

Get publicly available metrics (favorites, replies, retweets, video_views) using bearer-token auth and/or those metrics plus impressions and engagements for owned Tweets (user-context auth). Supports up to 250 Tweets per request.

Run `$ python engagement_totals.py --help` to see the list of arguments available when running the script. Here they are for reference:

```shell
-t TWEET_IDS (required): Pass one or more Tweet IDs (space delimited)
-m METRICS (optional): Pass one or more metrics (space delimited). Overrides the default metrics (supports all avail metrics by default).
-o --owned (optional): Boolean flag that uses user-context auth to get owned metrics (impressions and engagements)
```

#### Example commands

Unowned Tweets (default metrics):

```shell
$ python engagement_totals.py -t 20 21
```
Owned Tweets (default metrics):
```
$ python engagement_totals.py -t <TweetID_1> <TweetID_2> -o -m favorites retweets
```

### 28hr endpoint

Get up to 17 engagement metrics for a collection of up to 25 Tweets for the past 28 hours.

Run `$ python engagement_28hr.py --help` to see the list of arguments available when running the script. Here they are for reference:

```shell
-t TWEET_IDS (required): Pass one or more Tweet IDs (space delimited)
-m METRICS (optional): Pass one or more metrics (space delimited). Overrides the default metrics (supports all avail metrics by default).
```

#### Example commands

Owned Tweets (default metrics):

```shell
$ python engagement_28hr.py -t <TweetID_1> <TweetID_2>
```

Owned Tweets (specific metrics):

```shell
$ python engagement_28hr.py -t <TweetID_1> <TweetID_2> -m impressions engagements user_follows
```

### Historical endpoint

Get up to 17 engagement metrics for a collection of up to 25 Tweets for any time period after September 1, 2014 (up to 4 weeks at a time).

Run `$ python engagement_historical.py --help` to see the list of arguments available when running the script. Here they are for reference:

```shell
-t TWEET_IDS (required): Pass one or more Tweet IDs (space delimited)
-s START_DATE (optional): Specify a start date for the query (e.g., 2019-01-01T12:00:00Z)
-e END_DATE (optional): Specify an end date for the query (e.g., 2019-01-28T12:00:00Z)
-m METRICS (optional): Pass one or more metrics (space delimited). Overrides the default metrics (supports all avail metrics by default).
```

#### Example commands

Owned Tweets (defaults):
```shell
$ python engagement_historical.py -t <TweetID_1> <TweetID_2>
```

Owned Tweets (specific metrics, start, and end date):

```shell
$ python engagement_historical.py -t <TweetID_1> <TweetID_2> -s 2019-01-01T12:00:00Z -e 2019-01-28T12:00:00Z -m impressions engagements
```

## Historical PowerTrack

Create and run jobs against entire archive of publicly available Tweets.

### Create a job

To create a job, you must first go in and define your job by editing the `historical_job.json` file. Below is a breakdown of each element in the JSON file:

1. `publisher` - leave as-is.
2. `dataFormat` - leave as-is.
3. `fromDate` - UTC timestamp indicating the start time of the period of interest (minute granularity)
4. `toDate` - UTC timestamp indicating the end time of the period of interest (minute granularity)
5. `title` - the title of your job, must be unique to your account.
6. `rules` - An array of PowerTrack rules (up to 1,000) that determines which data is returning by your job.

### Accept or reject a job

Choose to accept or reject a job that is in the 'quoted' stage.

To accept a job, run the command:

```shell
$ python accept_or_reject_job.py -j <jobURL> -a
```

To reject a job, run the command:

```shell
$ python accept_or_reject_job.py -j <jobURL> -r
```

### Monitor a job

After a job is created, you can monitor the current status of a specific job.

Run the command:

```shell
$ python monitor_job -j <jobURL>
```

### Download Twitter data files

When job status equals 'delivered', the files are available for download. The `download_job.py` script will retrieve the list of URLs and begin downloading each file to a `./downloads` folder. If the job is interrupted at any point, simply restart the download script and it will only download new files (skipping over previously downloaded files). (_Note:_ the required 'dataURL' argument is returned by the response of a completed job from `monitor_job.py` request.)

Run the command:

```shell
$ python download_job.py -d <dataURL>
```

### Get job results (list of S3 URLs)

Retrieves info about a completed Historical PowerTrack job, including a list of URLs that correspond to the data files generated for a completed job. (_Note:_ the required 'dataURL' is returned by the response of a completed job from `monitor_job.py` request.)

Run the command:

```shell
$ python job_results.py -d <dataURL>
```

### Get jobs (list all jobs)

Retrieves details for all historical PowerTrack jobs which are not expired for the given account.

Run the command:

```shell
$ python list_jobs.py
```

## PowerTrack

This streaming API enables you to filter the full Twitter firehose in realtime.

### Add a Rule

Create a rule (filter) that will be applied to your PowerTrack stream.

Run the command:

```shell
$ python add_rules.py -r 'python OR ruby OR javascript'
```

_Note:_ You must quote the full rule value if it contains spaces or more than one clause (the above is a good example of this).

### Delete a Rule

Delete one or more rules from your stream by referencing the rule ID.

Run the command:

```shell
$ python delete_rules.py -i 1154088735153123328
```

### List Rules

Get all existing rules for a stream.

Run the command:

```shell
$ python get_rules.py
```

### Connect to PowerTrack Stream

Connect to your PowerTrack stream and start streaming Tweets to your terminal.

Run the command:

```shell
$ python get_stream.py
```

The code reads the stream in "chunks", with a default chunksize of 10000. This is optimized for consistent volume flowing through the stream. If you're simply testing by using a unique hashtag or Tweeting from your own account (e.g., `from:your-handle`), you can specify a smaller chunksize with the `-c` flag. A value of 500 or 1000 would work better in this case.

## Search API (30-day and Full-Archive)

Query for data or counts against the past 30 days or the full archive public Tweet data. This script allows you to specify a product 'archive' (30day or fullarchive), get Tweet data (by default) or counts (`-c`), and specify the query and date range directly on the command line.

Run `$ python search.py --help` to see the full list of arguments for this script. Here they are for reference:

```shell
  -r, --request_file    Use json file for request body
  -q QUERY, --query QUERY
                        A valid query up to 2,048 characters
  -c, --counts          Make request to 'counts' endpoint
  -f FROM_DATE, --from_date FROM_DATE
                        Oldest date from which results will be provided
  -t TO_DATE, --to_date TO_DATE
                        Most recent date to which results will be provided
  -m MAX_RESULTS, --max_results MAX_RESULTS
                        Maximum number of results returned by a single
                        request/response cycle (range: 10-500, default: 100)
  -b {day,hour,minute}, --bucket {day,hour,minute}
                        The unit of time for which count data will be
                        provided.
  -n, --next            Auto paginate through next tokens
  -p, --pretty_print    Pretty print the results
```

At a minimum, you must specify pass the `-r` flag for the request file or the query (`-q`) argument. One of the two arguments is required to run the script.

### Search API Tweet type (additional script)

There's a secondary Python script available, `search_tweet_type.py`, that contains a function to classify the type of Tweets returned by the search results. This script only supports a data request (not counts), parses the Tweet payload for select fields, and adds the derived "Tweet type" classification to the response output. It also checks for the presense of an "extended Tweet" and will read the `full_text` object if necessary (to avoid truncated Tweet text).

Run the command:

```shell
$ python search_tweet_type.py -r
```

### Example commands

30-Day data request (no optional args passed):

```shell
$ python search.py -q from:jack
```

30-Day counts request specified by request file (request.json):

```shell
$ python search.py -r -c
```

Full-archive data request with specifc dates and maxResults of '500':

```shell
$ python search.py -q 'python OR ruby' -f 201907010000 -t 201907072359 -m 500
```

Full-archive counts requests that paginates full data set:

```shell
$ python search.py -q 'python OR ruby' -f 201904010000 -t 201907010000 -c -b day -n
```