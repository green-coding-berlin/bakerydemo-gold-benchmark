# bakerydemo GOLD benchmark

Reference Wagtail site implementation for energy usage benchmarking.

## Project setup

This project is based upon Wagtail’s official [bakerydemo](https://github.com/wagtail/bakerydemo), with tweaks to make it more suitable for local [energy consumption benchmarking](https://github.com/wagtail/wagtail/discussions/8843).

### Differences with vanilla bakerydemo

With the overall goal of making the project more representative of a well-maintained real-world production site:

- Full Debian base image rather than `-slim`
- Gunicorn instead of uWSGI
- Python 3.11 instead of 3.9
- Latest WhiteNoise 6.2.0 for Python 3.11 compatibility
- Up-to-date production-ready Django configuration

### Differences with real-world production sites

With the overall goal of making it simpler to get the project running locally with Docker as the only requirement:

- Running locally rather than in a data center
- No reverse-proxy CDN caching requests
- Serving media files with Django (bad practice) rather than dedicated object storage service (S3, Google Cloud Storage, etc)
- Running over HTTP rather than HTTPS

## Getting started

Requirement: Docker.

```bash
# Download a copy of https://github.com/thibaudcolas/bakerydemo-gold-benchmark.
cd bakerydemo-gold-benchmark
docker compose build app
docker compose up app
# The site is up and running but still needs its database initialised.
docker compose exec app ./manage.py migrate
docker compose exec app ./manage.py load_initial_data
```

The site is now up and running with its demo content, but still needs cache warming to be representative of a real-world production site. In particular, Wagtail generates optimised images on the fly _the first time an image is requested_.

Here is a sample `wget` command to warm up the cache:

```bash
wget --recursive --spider --no-directories http://localhost:8005/ -o warmup.log
```

From there, the site can be accessed at <http://localhost:8005/>.

## Benchmark scenarios

Our scenarios cover different aspects of a site, representing real-world user journeys, all spanning multiple pages:

- `homepage-landing.js`: The simplest user journey. Landing on the homepage, navigating to the bread listing page, and finally to a bread detail page.
- `blog-filtering.js`: Another simple user journey. Landing on the blogs listing, filtering it by tag, and arriving on a blog page.
- `contact-us.js`: Successful submission of a simple Wagtail form.
- `search.js`: Usage of a search form. From the homepage, arriving on the search results for "bread", and opening one result.
- `admin.js`: A simple journey through the Wagtail admin. Logging in, editing a blog page, and checking the results in the live preview.

Note although the navigation through the sequence of pages represents a real-world journey, the scenarios’ duration isn’t representative of real usage. Average times spent across different page types are:

- Homepage: 2min
- Listing page: 1min
- Search results: 30s
- Blog post: 1min30s to 8min

### Puppeteer scenarios

Puppeteer test scripts are in `benchmark/puppeteer`. To run those scenarios (requirement: Node 18),

```bash
cd benchmark/puppeteer
npm install
# Then run each scenario with `node`:
node homepage-landing.js
```



## OpenEnergyBadge

These badges show the cost of running certain scenarios in this repository:

#### All Routes
<a href="https://metrics.green-coding.berlin/stats.html?id=37e0ca9c-b38e-4833-8316-59802d8ef1da"><img src="https://api.green-coding.berlin/v1/badge/single/37e0ca9c-b38e-4833-8316-59802d8ef1da?metric=RAPL"></a>


#### Migrations
<a href="https://metrics.green-coding.berlin/stats.html?id=dfb58eb7-7100-4ec6-80ee-7653e1329190"><img src="https://api.green-coding.berlin/v1/badge/single/dfb58eb7-7100-4ec6-80ee-7653e1329190?metric=RAPL"></a>

#### Cache Warmups
<a href="https://metrics.green-coding.berlin/stats.html?id=2821c396-98f0-4210-8aad-a9fc5a37f01e"><img src="https://api.green-coding.berlin/v1/badge/single/2821c396-98f0-4210-8aad-a9fc5a37f01e?metric=RAPL"></a>

#### Admin route only
<a href="https://metrics.green-coding.berlin/stats.html?id=ac6b2e5e-7b02-4002-a864-2a5b9e5bc3de"><img src="https://api.green-coding.berlin/v1/badge/single/ac6b2e5e-7b02-4002-a864-2a5b9e5bc3de?metric=RAPL"></a>

