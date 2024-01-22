# Object Stores

Big Data systems like Hadoop aren't great for storing static content because we’re not planning to run any compute over it, we generally just want to store and retrieve it later. In other words, we're wasting CPU resources when we want more disk space

An **object store** is a service offered by a cloud provider to store static content. They typically are managed services that handle scaling and replication and are schemaless; they're able to store any arbitrary file types

Data lakes, which are centralized repositories designed to store, process, and secure large amounts of structured, semi-structured, and unstructured data, are built on top of object stores since they typically stores data in its native format (blobs and files). Performing batch processing over this data would require us to export it to Hadoop for batch processing, which can be slow.
