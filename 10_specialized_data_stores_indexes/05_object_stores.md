# Object Stores

Big Data systems like Hadoop aren't great for storing static content because we’re not planning to run any compute over it, we generally just want to store and retrieve it later. In other words, we're wasting CPU resources when we want more disk space

An **object store** is a data storage architecture for storing unstructured data. As the name implies, it's mainly used for storing **objects**, which can be any kind of file (video, image, text, etc.). As a result, they are _schemaless_.

Every object in the object store has 4 components:

1. A unique identifier used for identification and retrieval
2. The data itself
3. Metadata, such as file type, size, creation date, creator, etc.
4. Attributes, which are related to metadata. These encompass things like permissions for the file, e.g. which users are allowed to download it, delete it, etc.

In practice, object stores are typically are managed services that handle scaling and replication automatically. Users who interact with object stores don't need to concern themselves with the details of the physical storage devices underneath the hood - they simply interact with an API to upload, download, edit, and delete files.

Cloud platforms will generally offer object store pricing plans around retrieval performance. For example, the cheapest tier of object storage won't be very performant, and is thus suited for rarely accessed data, like historical records. Meanwhile, more heavily accessed application data like static assets (images, videos, audio files), will occupy higher-priced, higher-performance storage.

Data lakes, which are centralized repositories designed to store, process, and secure large amounts of structured, semi-structured, and unstructured data, are built on top of object stores since they typically stores data in its native format (blobs and files). Performing batch processing over this data would require us to export it to Hadoop for batch processing, which can be slow.
