# A2RB Count Proxy

Forked and built from the [Alfa Jango Count
Proxy](https://github.com/alfajango/count-proxy).

This application grabs Meetup info from the Meetup API for the Ann Arbor
Ruby Brigade group, and stores it in a static JSONP file on Amazon S3 to
be embedded directly in a2rb.org.

## Process

This script runs a few times a day to update the Meetup group info.

It will loop through each event, with a slight delay to ease load on
the vendor servers. For each event, it will grab the event name,
description, date, location, and number of people registered.

The script will then write the info for all events to a JavaScript JSONP
string and store it as a file on Amazon S3.

Writing to jsonp will eliminate cross-domain concerns inherent in
loading JSON data from S3.

## File format

Example file format for the json file stored on Amazon S3:

```js
setJSON({
  Ann-Arbor-Ruby: {
    events: [
      {
        name: "Event Name",
        description: "<p>Event description</p>",
        time: 1412118000000,
        status: "upcoming",
        venue: {
          name: "Venue Name",
          address_1: "Address 1",
          address_2: "Address 2",
          city: "Ann Arbor",
          state: "MI",
          zip: "48105"
        }
        yes_rsvp_count: 56,
        maybe_rsvp_count: 2,
        headcount: 0,
        event_url: "http://www.meetup.com/Ann-Arbor-Ruby/events/203156012/",
        ...
      }
    ]
  }
});
```

## Running script

First, update the top of the script with the settings for the Meetup
group and and Meetup API credentials to import, as well as with your own
S3 bucket name and desired filenames.

To run the script, make sure it's executable:

```
sudo chmod u+x bin/get_counts
```

Then run it, be sure that your `MEETUP_API_KEY`, `AWS_ACCESS_KEY`, and `AWS_SECRET_KEY`
variables are present in your environment:

```
MEETUP_API_KEY=YOUR_MEETUP_API_KEY_HERE AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_HERE AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY_HERE bin/get_events
```

## Setting up on Heroku

You'll want to put the script somewhere on a server (or just set it up
locally), so that it can run on a scheduled cron job to update the
download counts and rewrite the file on S3.

The easiest place to deploy this script to is probably Heroku.

Just clone this repo from Github, and create a new app on Heroku's Cedar
stack with the heroku gem:

```
heroku apps:create my-meetup-proxy
git push heroku
```

Then add your Amazon Web Services access key and secret key to Heroku's
environment config for the app:

```
heroku config:add MEETUP_API_KEY=YOUR_MEETUP_API_KEY_HERE AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_HERE AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY_HERE
```

To test the script on heroku:

```
heroku run get_events
```

Now, let's setup Heroku's "scheduler" addon (previously called the "cron" addon):

```
heroku addons:add scheduler:standard
```

And set it up to run however often we'd like:

```
heroku addons:open scheduler
```

The above will open the Heroku scheduler dashboard in your browser,
where you can click "Add job...", type in `get_events`, and select your
frequency (probably daily, or at most hourly, if possible, to minimize
impact on venders' available API resources).
