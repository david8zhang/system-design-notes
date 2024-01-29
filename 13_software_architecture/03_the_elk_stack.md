# ELK - Elasticsearch, Logstash and Kibana

In previous sections, we looked a the microservices architecture and the advantages it gives us in terms of independent scalability, flexibility, and reliability in a large software system. If you'll recall, one of the drawbacks was the inability to enforce standardized monitoring and logging across these hundreds or potentially thousands of services.

Enter the **ELK stack** - an acronym used to describe a technology stack comprised of three popular technologies, Elasticsearch, Logstash, and Kibana. The ELK stack aims to provide an easy way to aggregate, analyze and vizualize logs across many different services, and has become very popular for troubleshooting and monitoring large microservices-based systems.

Let's take a look at each component in further detail

## Elasticsearch

We briefly mentioned Elasticsearch we we talked about [search indexes](/topic/10_specialized_data_stores_indexes?subtopic=01_search_indexes). As a refresher, it's a distributed search index built on top of Apache Lucene, and it provides some nice features for analytics and querying. The key thing to note in the context of the ELK stack is that often times, Elasticsearch is used more as a datastore for logs than as a search index

## Logstash

Logstash is an open source data-ingestion tool that transforms and aggregates data from multiple sources to be republished elswhere. It functions as the entrypoint for your microservices to send their logs to. Traditionally it was used to just parse and forward log data to Elasticsearch to be analyzed. In more recent years, Logstash has started to become a more general-purpose [stream processing consumer](/topic/09_stream_processing), and has a rich plugin ecosystem that lets you customize how you ingest, parse, and publish your data, with support for multiple downstream destinations (which it refers to as "stashes").

## Kibana

Kibana is an analytics and visualization platform which serves as the view layer for the ELK stack. It provides rich functionality for creating various realtime data visualizations like charts, maps and tables. It's basically a web platform that queries data from Elasticsearch via its REST API and provides configurations for how to display the results.

## From ELK to Elastic Stack

Today, the ELK Stack has evolved to incorporate two new components (X-Pack and Beats) and is now more commonly referred to as the "Elastic stack". X-Pack is a pack of features which provide additional functionality on top of ElasticSearch, such as graph visualizations and machine learning based anomaly detection. Beats is a event publishing agent that can be installed on your microservice hosts to standardize how you publish your log files (FileBeat) and metrics (MetricBeat).

**Note:** The ELK stack started off as a fully open source project, but has recently become a proprietary, managed offering provided by the Elastic NV company after transitioning from the Apache License v2 to the dual Server Side Public License and Elastic License. AWS provides alternatives to Elasticsearch and Kibana under the old Apache License in the form of OpenSearch and OpenSearch Dashboards.
