# octo2influx - Fork
This is my fork of [yo8192's octo2influx](https://github.com/yo8192/octo2influx) project.

**Breaking changes**
> A bug Grafana version `11.1` breaks the Flux queries used for tariff pricing. 
> - I recommend using `grafana/grafana:11.0.1` until the following issue is fixed.
> - https://github.com/grafana/grafana/issues/89948

My reasons for forking the project are:
- I use Unraid as my Docker environment, which has its own peculiarities and does not run docker-compose natively.
- I like versioning images and making them available for distribution.
- It gives me another reason to play with GitHub actions.

The octo2influx tool enables users to download their Octopus Energy usage, including import and export data.
The latter is for solar equipped properties that export energy back to the grid.

[InfluxDB](https://www.influxdata.com/products/influxdb-overview/) v2 database and display in [Grafana](https://grafana.com/).

## About
octo2influx retrieves energy usage data and the tariffs using the [Octopus API](https://developer.octopus.energy/docs/api/).

The tool calculates the cost based on the tariff. You can switch between tariffs to compare costs.
>Additional tariffs need to be configured to compare prices.

In my case, I have only configured the current active tariff and will add more as needed or when I change to a new one.

For example, switching to the "Agile" tariff during the same time period on the dashboard above will provide us with 
a different cost readout.
![screenshot of the electricity cost with a different tariff](images/grafana-example-agile.png)

## Installation and Running
This fork focuses on running the application with Docker.
- docker-compose
- Docker run
- Unraid Docker template

```shell
docker run -d --name='octo2influx' -e 'FREQ'='1h' -v '/mnt/user/appdata/octo2influx':'/etc/octo2influx/':'rw' --restart=unless-stopped 'ghcr.io/echoblag/octo2influx:latest'
```

The image has been extended in a few small areas to make it easier to update and setting file permissions.

> The application can be run as a standalone Python process using `python3 ./octo2influx.py`

### Data Availability
Octopus typically makes your usage data available the next day.
> If you have recently had smart meters installed, it can take a few days for consumption data to start coming in.

## Configuration
First, create your own `config.yaml` file using the [example](src/config.example.yaml) as a starting point.
how to get the information you need.

The priority of configuration options from highest to lowest is:
1. Environment variables
2. Command line flags
3. Config file

> The help section provides all the CLI flags and their equivalent environment variables. 

> The settings can be declared in the config file `config.yaml` located in `/etc/octo2influx`.

The application has a flexible command line parameters with a detailed help section:
```
python3 ./octo2influx.py --help
usage: octo2influx [-h] [--from_max_days_ago FROM_MAX_DAYS_AGO] [--from_days_ago FROM_DAYS_AGO] [--to_days_ago TO_DAYS_AGO] [--loglevel LOGLEVEL] [--timezone TIMEZONE]
                   [--base_url BASE_URL] [--octopus_api_key OCTOPUS_API_KEY] [--price_types PRICE_TYPES] [--usage USAGE] [--tariffs TARIFFS] [--influx_org INFLUX_ORG]
                   [--influx_bucket INFLUX_BUCKET] [--influx_tariff_measurement INFLUX_TARIFF_MEASUREMENT] [--influx_usage_measurement INFLUX_USAGE_MEASUREMENT]
                   [--influx_url INFLUX_URL] [--influx_api_token INFLUX_API_TOKEN]

Download usage and pricing data from the Octopus API
(https://developer.octopus.energy/docs/api/) and store into Influxdb.

options:
  -h, --help            show this help message and exit
  --from_max_days_ago FROM_MAX_DAYS_AGO
                        Get Octopus data from the last retrieved timestamp, but no more than this many days ago.
  --from_days_ago FROM_DAYS_AGO
                        Get Octopus data from that many days ago (0 means today). If set, this overrides from_max_days_ago.
  --to_days_ago TO_DAYS_AGO
                        Get Octopus data until that many days ago (0 means today).
  --loglevel LOGLEVEL   Level of logs (INFO, DEBUG, WARNING, ERROR).
  --timezone TIMEZONE   Timezone of the Octopus account (e.g. where you live). Most likely always "Europe/London".
  --base_url BASE_URL   Base URL of the Octopus API (e.g. "https://api.octopus.energy/v1").
  --octopus_api_key OCTOPUS_API_KEY
                        (**Config file or environment only**) The API Token to connect to the Octopus API. Can be generated on
                        https://octopus.energy/dashboard/developer/.
  --price_types PRICE_TYPES
                        (**Config only**) List of price types to retrieve using the Octopus API, and their units.
  --usage USAGE         (**Config only**) List of Octopus usage (electricity/gas import consumption, or export) to retrieve using the Octopus API.
  --tariffs TARIFFS     (**Config only**) List of Octopus tariffs to retrieve using the Octopus API.
  --influx_org INFLUX_ORG
                        InfluxDB 2.X organization name to store the data into.
  --influx_bucket INFLUX_BUCKET
                        InfluxDB 2.X bucket name to store the data into (e.g. "mybucket/autogen").
  --influx_tariff_measurement INFLUX_TARIFF_MEASUREMENT
                        InfluxDB 2.X measurement name to store tariff data into.
  --influx_usage_measurement INFLUX_USAGE_MEASUREMENT
                        InfluxDB 2.X measurement name to store consumption data into.
  --influx_url INFLUX_URL
                        URL of the InfluxDB 2.X instance to store the data into (e.g. "http://localhost:8086")
  --influx_api_token INFLUX_API_TOKEN
                        (**Config file or environment only**) The API Token to connect to the InfluxDB 2.x instance.

IMPORTANT NOTE: you should *not* define secrets and API tokens on the command
line, as it is unsecure (e.g. it may stay in your shell history, appear in
system audit logs, etc): you can define in an access-restricted configuration
file instead.

The settings can also be set in the config file config.yaml (in
/etc/octo2influx, ~/.config/octo2influx, or the directory defined by the env var
octo2influxDIR), or via environment variable of the form
octo2influx_COMMAND_LINE_ARG.
The priority from highest to lowest is: environment, command line, config file.
```

## Acknowledgements
This is a fork of the https://github.com/yo8192/octo2influx project that integrates GitHub Actions to build images as well as 
dependency management.

### Pre-fork
The [yo8192's octo2influx](https://github.com/yo8192/octo2influx) project is originally based on https://github.com/stevenewey/octograph/.

The motivation for the rewrite came following a solar installation, the project was largely rewritten to use 
InfluxDB v2 and the Influx query language, factoring in electricity export and an updated Grafana dashboard.
