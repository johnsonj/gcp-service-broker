# Pivotal Cloud Foundry Service Broker for Google Cloud Platform

Depends on
[lager](https://github.com/pivotal-golang/lager) and
[gorilla/mux](https://github.com/gorilla/mux).

Requires go 1.6 and the associated buildpack

## Prerequisites

### GCP prereqs

1. go to console.cloud.google.com and sign up, walking through the setup wizard
1. next to the Google Cloud Platform logo in the upper left-hand corner, click the dropdown and select "Create Project"
1. give your project a name and click "Create"
1. when the project is created (a notification will show in the upper right), refresh the page.
1. in the left nav, go to API Manager
1. click "Library"
1. search "google cloud resource manager api", click the option with no other modifiers, and enable.
1. search "Google Identity and Access Management (IAM) API", click the option with no other modifiers, and enable.
1. if you wish to use CloudSQL, search "sqladmin", click the only option, and enable.
1. in the left nav, go to IAM and Admin
1. click Service Accounts
1. click create service account and set the role to Owner
1. check furnish new private key (leave json key type)
1. click create, save the downloaded file to a safe and accessible place.

### Db prereqs

1. create new MySQL instance
1. run `CREATE DATABASE servicebroker;`
1. run `CREATE USER '<username>'@'%' IDENTIFIED BY '<password>';`
1. run `GRANT ALL PRIVILEGES ON servicebroker.* TO '<username>'@'%' WITH GRANT OPTION;`
1. (optional) create ssl certs for the database and save them somewhere secure

### required env vars - if deploying as an app, add these to missing-properties.yml

* ROOT_SERVICE_ACCOUNT_JSON (the string version of the credentials file created for the Owner level Service Account)
* SB_USERNAME (a username to sign all service broker requests with - the same one used in cf create-service-broker)
* SB_PASSWORD (a password to sign all service broker requests with - the same one used in cf create-service-broker)
* DB_HOST (the host for the database to back the service broker)
* DB_USERNAME (the database username for the service broker to use)
* DB_PASSWORD (the database password for the service broker to use)

### optional env vars - if deploying as an app, optionally add these to missing-properties.yml

* DB_PORT (defaults to 3306)
* CA_CERT
* CLIENT_CERT 
* CLIENT_KEY 
* CLOUDSQL_CUSTOM_PLANS (A JSON array of objects with fields guid, name, description, tier, 
pricing_plan, max_disk_size, display_name, and service (CloudSQL's service id_)


## Usage

### As an App

#### Update the manifest with ENV vars
1. replace any blank variables that are in manifest.yml with your own ENV vars

#### Push the service broker to CF and enable services
1. cf push gcp-service-broker
1. cf create-service-broker <service broker name> <username> <password> <service broker url>
1. cf enable-service-access pubsub

### As a Tile

#### Import the product into Ops Manager
1. Click "Import a Product" and upload the .pivotal file from the product directory

#### Add the product to your Dashboard
1. Click the plus icon next to the uploaded product

#### Configure the Service Broker
1. Click on the tile and fill in any required fields (tabs will be orange if updates are needed)
1. Once the tile is green and updates are applied, review the service/plan access and
update if necessary using cf disable-service-access. By default, all services and plans
are enabled except CloudSQL (unless plans have been saved for it). If you wish to change this,
you'll need to use the cf cli's service-access commands.

### (Optional) Increase the default provision/bind timeout
It is advisable, if you want to use CloudSQL, to increase the default timeout for provision and
bind operations to 90 seconds. CloudFoundry does not, at this point in time, support asynchronous
binding, and CloudSQL bind operations may exceed 60 seconds. To change this setting, set
broker_client_timeout_seconds = 90

### Use!

e.g. cf create-service pubsub default foobar -c '{"topic_name": "foobar"}'
e.g. cf bind-service myapp foobar -c '{"role": "pubsub.admin"}'

Service calls take the following custom parameters, all as strings, (required where marked):

* [PubSub](https://cloud.google.com/pubsub/docs/)
    * Provison
        * topic_name (required)
        * subscription_name
        * is_push (defaults to false, to set use "true")
        * endpoint (for when is_push == "true", defaults to nil)
        * ack_deadline (in seconds, defaults to 10, max 600)
    * Bind
        * role without "roles/" prefix (see https://cloud.google.com/iam/docs/understanding-roles for available roles)
* [Cloud Storage](https://cloud.google.com/storage/docs/)
    * Provison
        * name (required)
        * location (for options, see https://cloud.google.com/storage/docs/bucket-locations. Defaults to us)
    * Bind
        * role without "roles/" prefix (see https://cloud.google.com/iam/docs/understanding-roles for available roles)
* [BigQuery](https://cloud.google.com/bigquery/docs/)
    * Provison
        * name (required)
    * Bind
        * role without "roles/" prefix (see https://cloud.google.com/iam/docs/understanding-roles for available roles), e.g. pubsub.admin
* [CloudSQL](https://cloud.google.com/sql/docs/)
    * Provison
        * instance_name (required)
        * database_name (required)
        * version (defaults to 5.6)
        * disk_size in GB (only for 2nd gen, defaults to 10)
        * region (defaults to us-central)
        * zone (for 2nd gen) 
        * disk_type (for 2nd gen, defaults to ssd)
        * failover_replica_name (only for 2nd gen, if specified creates a failover replica, defaults to "")
        * maintenance_window_day (for 2nd gen only, defaults to 1 (Sunday))
        * maintenance_window_hour (for 2nd gen only, defaults to 0)
        * backups_enabled (defaults to true, set to "false" to disable)
        * backup_start_time (defaults to 06:00)
        * binlog (defaults to false for 1st gen, true for 2nd gen, set to "true" to use)
        * activation_policy (defaults to on demand)
        * replication_type (defaults to synchronous)
        * auto_resize (2nd gen only, defaults to false, set to "true" to use)
    * Bind
        * username
        * password
* [ML APIs](https://cloud.google.com/ml/)
    * Bind
        * role without "roles/" prefix (see https://cloud.google.com/iam/docs/understanding-roles for available roles)


## Change Notes


# This is not an official Google product.