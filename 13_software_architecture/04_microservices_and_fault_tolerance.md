# Microservices and Fault Tolerance

Fault tolerance becomes a non-trivial problem when we have hundreds of microservices interacting with each other. What happens when a critical service with lots of upstream dependencies fails? A typical way to recover from failures is to _retry_ the requests, and exponentially back off (similar to the way [TCP performs congestion control](/topic/11_networking?subtopic=02_tcp)). However, if the service is down for an extended period of time, we could end up with retried requests piling up in these upstream dependencies, leading to resource exhaustion and _cascading failures_.

## Circuit Breakers

The **circuit breaker** patern aims to solve the problem of _cascading failures_ as a result of service downtime. As the name implies, the circuit breaker pattern is based on electrical circuit breakers, which automatically interrupt current flow during power surges or faults to protect cascading failures in your electronic devices and appliances.

### Circuit Breaker States

Software circuit breakers are practically implemented as request interceptors and have 3 states: open, half open, and closed:

- When the circuit breaker is **closed**, it allows requests to execute normally. While doing so, it measures faults and successes, and if the number of faults exceed a certain threshold, the circuit will break and enter the open state.
- When the circuit breaker is **open**, it prevents the execution of all requests passing through it. The circuit breaker will remain in this state for some configured timespan, at which point it will enter the half-open state.
- Finally, when the circuit breaker is **half-open**, it will retry _some_ but not all of the failed requests. If these requests continue to fail, the circuit breaker will re-enter the open state and remain that way for another break period. Otherwise, it will close and start handling requests normally.

One additional optimization we can make is to configure a _fallback_ policy. When a downstream API fails, we can specify an alternative backup service or cache that the circuit breaker can redirect requests to. This will give us higher availability in our overall system, as we can continue to satisfy client queries when a service breaks.

## Rate Limiting

Another way to provide fault tolerance is to pre-emptively **rate-limit** upstream dependencies from flooding our service with requests. This is especially important when we're dealing with malicious threats to fault tolerance external to our system, such as Distributed Denial of Service (DDOS) attacks.

There are a few ways of rate-limiting and API throttling in software systems:

1. **Fixed Window**: Specifiy a limit on how many requests a client can send in a fixed window of time. After the fixed window passes, that limit resets.
2. **Token Bucket**: We specify a "bucket" with a capacity to hold N tokens. Whenever a request comes in, it "redeems" a token and is fulfilled. When all the tokens are gone, requests will be dropped / fail. The bucket will refill tokens at some speficied maximum refill rate.

   - The maximum refill rate in this scenario is the maximum rate that users will have on average.
   - The max capacity can be **larger** than the max refill rate. That means that for a given point in time, we could have a large number of requests coming in at once which consumes all available tokens (capacity). However subsequent requests will need to wait for the tokens to refill. Thus we are able to accommodate bursty traffic while still guaranteeing a maximum requests per second rate.

## Microservice Fault Tolerance in the Wild

- [Hystrix](https://github.com/Netflix/Hystrix) was a latency and fault tolerance library designed by Netflix to isolate points of access to remote systems, services and 3rd party libraries. It entered maintenance mode in 2020.
- [Resilience4J](https://resilience4j.readme.io/docs/getting-started) has been the replacement for Hystrix ever since, and is a lightweight fault tolerance library with a functional programming model for Java applications. It implements the circuit breaker, rate-limiter, and retry patterns
