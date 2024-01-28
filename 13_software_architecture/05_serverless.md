# Serverless Computing

Serverless computing is a method of providing compute resources on an as-used basis, enabling users to write and deploy code without needing to worry about the underlying infrastructure. The concept was first pioneered by Amazon in 2014 with the introduction of AWS Lambda.

The term serverless is somewhat of a misnomer, as servers are still being used behind the scenes. The main selling point is that the actual configuration and provisioning of those servers is entirely handled by the vendor, effectively hiding them from the developer.

Some examples of serverless providers are:

- [AWS Lambda](https://aws.amazon.com/pm/lambda)
- [Google Cloud Functions](https://cloud.google.com/functions#key-features)

## Advantages & Disadvantages

Serverless computing has several advantages:

- **Potentially lower cost**: If services have uneven traffic load, serverless may be a more cost-effective solution since we don't waste resources and money running idle servers
- **Simplified scalability**: Vendors automatically scale resource allocation based on usage, vastly simplifying the process for developers.
- **Simplified code**: Typically the developer just writes a function rather than needing to setup all the boilerplate for running a server.

However, it has some downsides:

- **Cold Start Problem**: When serverless functions aren't invoked for a long time, vendors will dynamically scale down resource usage to zero. Subsequent invocations to this "cold" function will require all of these resources to be spun back up, which could result in high latency
  - There are some ways of mitigating this, such as a minimum allocation setting to keep functions "warm"
- **Costly for multi-step workflows or long-running processes**: Long running processes with consistent load are typically better off using traditional servers. Reserved hardware is typically much more cost effective in these scenarios.

  Furthermore, multi-step workflows can be extremely inefficient and expensive. A famous example is when [Amazon Prime Video cut costs by 90%](https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90) in 2023 after switching from a serverless architecture to a monolithic one for their machine-learning based audio corruption detection workflow.
