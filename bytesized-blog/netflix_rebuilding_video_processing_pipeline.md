# How Netflix Chopped Up Their Video Processing Monolith into a Bunch of Microservices

[Full article link](https://medium.com/netflix-techblog/rebuilding-netflix-video-processing-pipeline-with-microservices-4e5e6310e359)

## Background Info

Back in 2014, Netflix developed this nifty video processing platform called Reloaded, which was designed to convert high quality media files into compressed assets for streaming purposes.

Reloaded was initially built as a [monolith](/topic/13_software_architecture). And over the years, it began to grow (uh oh). As a wise philosopher (me) once said - _"Mo' nolith? Mo' features, mo' problems"_. Here are a few:

- **Tightly coupled functionality**. Video quality logic was entangled with video encoding, so it was super annoying to recalculate video quality without re-encoding

- **DRY Violations**: Had a whole bunch of gross, repeated code since everything was stewing in the same repository

- **Deployment Woes**: Modules all had to be deployed together since it was one big fat repo

- **Testing takes a long ass time**: All the modules had to be tested at the same time during deployments. And the testing had to be pretty rigorous so that they don't break prod. To make matters worse: rollbacks are super expensive if they _do_ bring it down. All this = testing takes **two weeks**.

They decided to build out a new platform called **Cosmos**.

## How those crazy sons of bitches did it

They took a look at the current video processing pipeline and broke it up into steps. Then they rolled out services that do each step:

- Video Inspection Service (VIS): Takes the raw media files extracts some metadata for downstream services. Flag issues if we see some invalid or sus metadata

- Complexity Analysis Service (CAS): Looks at the content itself to figure out the optimal encoding recipe for it (using some metric called "complexity"). Calls VES to do some pre-encoding and VQS on that output as well.

- Ladder Generation Service (LGS): Creates the actual encoding recipe based on CAS's results. Netflix has a bunch of little tricks for optimizing this.

- Video Encoding Service (VES): does all the encoding stuff by breaking all the files into little chunks and encoding each one in parallel and stitching together the output

- Video Validation Service (VVS): Takes an encoded video and compares the results with the expected results based on the encode. Flag any discrepancies.

- Video Quality Service (VQS): does all the video quality calculation (but does it do this for each chunk still or does it do it for the whole output now??)

## Orchestrator Services

Video processing has a couple of use cases. So they needed some orchestrator services to handle each one.

First, obviously, there's the member streaming use case where the encoded assets get yeeted onto a CDN. Also, it needs audio and text assets (subtitles). The Streaming Workflow Orchestrator handles all of this.

Next there's the studio operations use case, e.g. marketing clips, production reviews. These actually require some fast turnarounds since the production teams will look at the encoded video output and decide how they should shoot for the next day. So the Studio Workflow Orchestrator is optimized for speed. It also has some studio-specific features like [forensic watermarking](https://massive.io/content-security/what-is-forensic-watermarking/) (an anti-piracy thing)

## Microservices are Lowkey Bussin

The whole process of building and migrating everything took 5 years. But it's paid some dividends - the extensibility of microservices allows quick iteration and deployment of new features like Ads encoding optimization (for the Ad-supported plan).

So yeah the moral of the story is that if you got this big ass monolith and five spare years, maybe you 🫵 should try doing microservices
