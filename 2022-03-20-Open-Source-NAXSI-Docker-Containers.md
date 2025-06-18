---
layout: post
title: nginx-naxsi has been open sourced!
subtitle: A modular, easy to use, poc NGINX + Naxsi waf.
tags: [opensource, naxsi]
comments: false
---

I was tasked with creating a proof-of-concept web application firewall for our on-prem infrastructure. This is what I have come up with. It's a dead simple implementation, 
written in Bash, that runs inside of a [Phusion BaseImage](https://hub.docker.com/r/phusion/baseimage/) container.

GitHub Link: coming soon.

DockerHub Link: coming soon.

# nginx-naxsi

##### Crude ASCII Diagrams

```bash
# nginx-naxsi waf - Learning Mode
#                                                       //===(log all)===>>[Elasticsearch]
#                                                       ||
#                                                 //= [WAF] (yy8080) --> F/E Container (80)
# Ingress (VIP:443) ------> LB/SSL (xx443) ------> == [WAF] (yy8080) --> F/E Container (80)
#                                                 \\= [WAF] (yy8080) --> F/E Container (80)
#
```

```bash
# nginx-naxsi waf - Blocking Mode
#                                                       //===(violations)===>> [Elasticsearch]
#                                                       ||
#                                                 //= [WAF] (yy8080) --> F/E Container (80)
# Ingress (VIP:443) -------> LB/SSL (xx443) -----> == [WAF] (yy8080) --> F/E Container (80)
#                                                 \\= [WAF] (yy8080) --> F/E Container (80)
#                                                       ||
#                                                       \\<<====(update rules)====[S3_BUCKET]
#
```

```bash
# nginx-naxsi-controller
#
#                //<<=========(ingest data)=========[Elasticsearch]
#                ||
#  [nginx-naxsi-controller]
#                ||
#                \\=========(update auto-gen rules)=======>>[S3_BUCKET]
#
#
```

### Requirements
* All environment variables need to be in place or it will refuse to run.
* Requires a single S3 bucket with read/write access.

### The Pieces

#### *nginx-naxsi*
* The actual WAF. (Internet) --> Load Balancer/SSL Term --> WAF --> F/E Endpoint
* http only -- terminate SSL / absorb that overhead elsewhere.
* Runs "pseudo-stateless" and slings data out for centralized reporting and management.
* Requires a container restart to move between learning and blocking modes.
* Run 1 WAF, or 100, it doesn't matter.

##### Learning Mode
* Streams training data to Elasticsearch in near-realtime.
* Runs the most basic of rule sets - allow all but flag everything.

##### Blocking Mode
* Pulls rule sets from S3 based on site name specified.
* If the container can't configure at run-time, it will refuse to run.
* Updates run every 2 minutes.
* If no update is found, no message will be displayed.
* If updates are found, it will reload NGINX gracefully and log the event.
* Error logs (read: waf shut them down) are sent to Elasticsearch for evaluation.

#### *nginx-naxsi-controller*
* Performs data analysis and sample rule set generation.
* Uses common elasticsearch endpoint, referencing a common site.
* Auto-generated rule sets are dumped to disk.
* Auto-generated rule sets are also uploaded to S3 for automation purposes.
* Designed for 1 controller per web site/collection of WAFs (read: fleet common $es_site_name).

#### *elasticsearch cluster* (wip/tbd)
* A demo ES 1.7 container is recommended.
* When live in production, backing code will likely reside elsewhere.
* Used for data aggregation across the entire fleet of WAFs.
* **tbd.**

## docker-compose.yml

```bash
version: '3'

services:
  nginx-naxsi:
    container_name: nginx-naxsi
    image: ekolp/nginx-naxsi:latest
    platform: linux/amd64
    environment:
      LEARNING_MODE: 'true'
      WAF_LISTEN_PORT: 80
      WAF_REDIRECT_HOST: 'slashdot.org'
      WAF_REDIRECT_PORT: 80
      ES_ENDPOINT: 'endpoint.amazonaws.com'
      ES_SITE_NAME: 'demowebapp'
      ES_PORT: 9200
      ES_LOCAL: 'false'
      S3_ACCESS_KEY: 'REDACTED'
      S3_SECRET_KEY: 'REDACTED'
      S3_REGION: 'us-west-2'
      S3_BUCKET: 'naxsi'
    ports:
      - '8080:80'
    build:
      context: ./
      dockerfile: Dockerfile-nginx-naxsi

  naxsi-controller:
    container_name: naxsi-controller
    image: ekolp/nginx-naxsi-controller:latest
    platform: linux/amd64
    environment:
      ES_ENDPOINT: 'endpoint.amazonaws.com'
      ES_SITE_NAME: 'demowebapp'
      ES_PORT: 9200
      S3_ENABLE: 'true'
      S3_ACCESS_KEY: 'REDACTED'
      S3_SECRET_KEY: 'REDACTED'
      SUMMARY_OUTPUT: 'false'
      S3_REGION: 'us-west-2'
      S3_BUCKET: 'naxsi'
    build:
      context: ./
      dockerfile: Dockerfile-nginx-naxsi-controller
```

### License and Authors

Author:: KickBack Rewards Systems (ekolp@krs.io)

