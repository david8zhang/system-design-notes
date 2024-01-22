# Content Delivery Networks

Content Delivery Networks (CDNs), are geographically distributed caches for static content, like HTML, image, video, and audio files. These files tend to be large, so we'd like to avoid having to download them repeatedly from our application servers or object stores (which we'll get into later).

There are a couple of types of CDNs:

A **push CDN** pre-emptively populates content that we know will be accessed in the CDN. For example, a streaming service like Netflix may have content that comes out every month that they'd anticipate their subscribers will consume, so they pre-load it onto their CDNs

A **pull CDN**, in contrast, only populates the CDN with content when a user requests it. These are useful for when we don’t know in advance what content will be popular.

### CDNs in the Wild

- [Akamai](https://www.akamai.com/solutions/content-delivery-network) provides CDN solutions among other edge computing services
- [Cloudflare](https://www.cloudflare.com/application-services/products/cdn/) provides a free CDN service
- [AWS CloudFront](https://aws.amazon.com/cloudfront/) is AWS's CDN offering
