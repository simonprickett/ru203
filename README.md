# RU203: Querying, Indexing, and Full-text Search with Redis

## Introduction

This repository contains example data and setup instructions for the Redis University course [RU203: Querying, Indexing, and Full-Text Search with Redis](https://university.redis.com/courses/ru203/).

To take this course, you'll first need do the following:

1. Clone this git repository
1. Get RediSearch and load the sample data
1. Build the RediSearch indexes by running the index creation commands

Need help getting started? Stop by the [Redis University Discord server](https://discord.gg/wYQJsk5c4A).

To get RediSearch, we recommend either launching it from Docker (for beginners) or building it from source and running the locally (more advanced).

> We are also currently trialling the use of [Gitpod](https://gitpod.io) hosted environments for this course.  To use Gitod, you'll need a GitHub account and a web browser: there's no software to install locally.  See Option 3 for details.

## Option 1: Run RediSearch locally with Docker (Best option for beginners)

First, make sure you have [Docker installed](https://docs.docker.com/get-docker/).

### 1. Start Redis

Next, run the following command from your terminal:

```
docker run -it --rm --name redis-search -p 6379:6379  redislabs/redisearch:2.0.5
```

This will launch a Redis instance with RediSearch installed. The instance will be listening on localhost port 6379.

### 2. Load the sample data

The commands that load the sample data are in the file `commands.redis`, which is part of this git repository. To load this data, run the following command:

```
docker exec -i redis-search redis-cli < commands.redis > output.txt
```

Check the "output" file for "Invalid" responses.

```
grep Invalid output.txt
```

If you have any "Invalid" responses, you might not have RediSearch installed properly. If you have
any problems, find us on [our Discord
channel](https://discord.gg/wYQJsk5c4A).

### 3. Create the indexes

To create the indexes, first start the Redis CLI:

```
docker exec -it redis-search redis-cli
```

Then paste in the index creation commands (see the "Building Indexes" section for details).

## Option 2: Build RediSearch from source

### 1. Build RediSearch and launch Redis

First, follow the instructions for [building and running RediSearch from source](https://oss.redis.com/redisearch/Quick_Start/#building_and_running_from_source).


### 2. Load the sample data

The commands that load the sample data are in the file `commands.redis`, which is part of this git repository. To load this data, run the following command from your terminal:

```
redis-cli < commands.redis > output.txt
```

This assumes that you have `redis-cli` in your path.

Check the "output" file for "Invalid" responses.

```
grep Invalid output.txt
```

If you have any "Invalid" responses, you might not have RediSearch installed properly. If you have
any problems, find us on [our Discord channel](https://discord.gg/wYQJsk5c4A).

### 3. Create the indexes

To create the indexes, first start the Redis CLI:

```
redis-cli
```

Then paste in the index creation commands (see the "Building Indexes" section for details).

## Option 3: Use a Gitpod Hosted Environment

Gitpod provides on demand hosted development environments in the cloud.  This is an option that requires no software installation on your machine, and which we are currently trialling for this course.  If you choose this option, we'd love to hear what you think of it - please find us in the `#ru203-querying-indexing-and-full-text-search` channel on the [Redis Discord](https://discord.gg/yEbfGhqhsz) chat server.

Begin by clicking the button below to launch a new cloud development environment with Gitpod:

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/simonprickett/ru203)

If you're using Gitpod for the first time, a popup will appear asking you to authorize Gitpod to work with your GitHub account.

You'll see the Gitpod workspace initialize in a new browser tab.  The workspace consists of the VSCode editor and its built in terminal.  A script runs automatically in the terminal to pull down a Redis Stack Docker container and start it, giving you a Redis instance with RediSearch enabled.

Wait until you see the following message in the terminal:

```
Container redis-stack  Started 
```

You're now ready to load the course data.  You'll also notice two dialog boxes in the bottom right of the workspace, informing you that services are available on port 8001 and port 6379.  Ignore these for the moment.

Load the course data by typing this command into the terminal:

```bash
load-data
```

When this command returns, click on the "x" icon to dismiss the dialog box in the bottom right of the workspace that says "A service is available on port 6379".

Click "Open Browser" in the dialog box that says "A service is available on port 8001".  This will open RedisInsight (a GUI interface to Redis) in a new tab.  Agree to the terms of service for RedisInsight and you should see the RedisInsight browser view showing the contents of your Redis database.

At the bottom of the RedisInsight tab, you'll see a toolbar with "CLI", "Command Helper" and "Profiler" options.  Click on "CLI" to bring up a Redis CLI window.  You'll use this throughout the course wherever `redis-cli` is needed.

If you close or forget your Gitpod workspace tabs, go to [gitpod.io](https://gitpod.io) to see a list of your most recently opened workspaces.

You're now ready to use your Redis CLI window to create the RediSearch indexes for this course.  See the "Building Indexes" section for details.

## Building Indexes

This data ships without RediSearch indexes, so you need to create them yourself.

We're going to give you all the commands you need to create these indexes, but
before you run these index commands, make sure you're in the redis CLI:

    $ redis-cli

Then run the following commands:

    FT.CREATE books-idx ON HASH PREFIX 1 ru203:book:details: SCHEMA isbn TAG SORTABLE title TEXT WEIGHT 2.0 SORTABLE subtitle TEXT SORTABLE thumbnail TAG NOINDEX description TEXT SORTABLE published_year NUMERIC SORTABLE average_rating NUMERIC SORTABLE authors TEXT SORTABLE categories TAG SEPARATOR ";" author_ids TAG SEPARATOR ";"

    FT.CREATE users-idx ON HASH PREFIX 1 ru203:user:details: SCHEMA first_name TEXT SORTABLE last_name TEXT SORTABLE email TAG SORTABLE escaped_email TEXT NOSTEM SORTABLE user_id TAG SORTABLE last_login NUMERIC SORTABLE

    FT.CREATE authors-idx ON HASH PREFIX 1 ru203:author:details: SCHEMA name TEXT SORTABLE author_id TAG SORTABLE

    FT.CREATE authors-books-idx ON HASH PREFIX 1 ru203:author:books: SCHEMA book_isbn TAG SORTABLE author_id TAG SORTABLE

    FT.CREATE checkouts-idx ON HASH PREFIX 1 ru203:book:checkout: SCHEMA user_id TAG SORTABLE book_isbn TAG SORTABLE checkout_date NUMERIC SORTABLE return_date NUMERIC SORTABLE checkout_period_days NUMERIC SORTABLE geopoint GEO

## You're Ready!

Now, you're ready to take the course.

If you aren't signed up yet, visit the [course signup page](https://university.redis.com/courses/ru203/) to register!

## Appendix: The Data Model

The queries in this repository rely on a data model composed of Redis
hashes. The entities in this data model include books, authors, library
checkouts, and users. The following diagram represents this data model:

```
    Authors
+--------------+
|  name        |               Author-Books
|              |            +----------------+
|  author_id   +------------+  author_id     |
|              |            |                |
+--------------+        +---+  book_isbn     |
                        |   |                |
    Users               |   +----------------+
+--------------+        |
|  first_name  |        |
|              |        |
|  last_name   |        |       Checkouts
|              |        |  +------------------------+
|  email       |   +----|--+  user_id               |
|              |   |    |  |                        |
|  user_id     +---+    +--+  book_isbn             |
|              |        |  |                        |
|  last_login  |        |  |  checkout_date         |
|              |        |  |                        |
+--------------+        |  |  checkout_length_days  |
                        |  |                        |
    Books               |  |  geopoint              |
+--------------+        |  |                        |
|  isbn        +--------+  +------------------------+
|              |
|  title       |
|              |
|  subtitle    |
|              |
|  thumbnail   |
|              |
|  description |
|              |
|  categories  |
|              |
|  authors     |
|              |
|  author_ids  |
|              |
+--------------+
```

### Subscribe to our YouTube Channel

We'd love for you to [check out our YouTube channel](https://youtube.com/redisinc), and subscribe if you want to see more Redis videos!
