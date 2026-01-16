---
layout: post
title:  "The Next.js Myth"
date:   2026-01-15 22:53:00 -0600
---

[Next.js](https://nextjs.org/) is a popular web framework built on top of [React](https://react.dev/) that provides sophisticated features for building modern and global web applications. It offers **server-side rendering**, **static site generation**, **API routes**, **incremental static regeneration**, **image optimization**, and more.

This framework was built by [Vercel](https://vercel.com/), a company that provides hosting and deployment services for web applications. Through the Vercel platform, you can deploy your Next.js applications with ease and take advantage of one of their best features in the market: *edge computing* and good CDN services. This means that when users try to access your application, the content is delivered from servers that are geographically closer to them, resulting in faster load times and improved performance. Beyond that, you get great SEO out of the box, which is crucial for improving your website's visibility.

But try the following: Run Next.js's [bundler analyzer](https://nextjs.org/docs/app/guides/package-bundling#nextjs-bundle-analyzer-experimental) on any production app, and you'll discover an uncomfortable truth. A simple landing page often ships **500KB+ of JavaScript**—before adding any business logic. Check your Network tab. Run [Lighthouse](https://developers.google.com/web/tools/lighthouse). The numbers don't lie.

Next.js ships a entire runtime to browsers, meaning the client side gets react runtime, next.js runtime, and also your code. Meanwhile, simpler tools like [Jekyll](https://jekyllrb.com/) or [HTMX](https://htmx.org/) achieve perfect Lighthouse scores with 10% of the payload.

We can actually go through Next's most popular features and see when they might actually be useful:

- **SSR/SSG/ISR**: The technical SEO trinity. But if your app is behind authentication or serves internal users, you're shipping complexity without getting much benefit. A [Vite](https://vite.dev/) SPA would be faster to build and debug.

- **Edge computing**: Great for global purposes. But in practice, most apps serve speciifc regional users. A simple us-east-1 webapp with CloudFlare CDN will outperform edge functions for 90% of use cases if you are targeting US users or specific regions.

- **File-based routing**: This souldn't even be discussed but I've seen junior developers choose Next just because of the clear file based routing. Every modern framework has this already or can be achieved through a library or a plugin.

### The Real Beneficiaries

I don't think Next.js is an engineering tool, I think it's a **marketing platform disguised as a framework**. Marketing teams gain the most from its capabilities: edge-rendered landing pages for A/B tests, ISR for content updates without deploys, and built-in SEO for organic growth. They get to ship campaigns at scale without waiting for engineering sprints.

**Engineering teams** pay the price. The "great developer experience" evaporates once you debug your first hydration mismatch or trace through three layers of caching to understand why content won't update. Vercel makes deployment simple—if you use Vercel. Self-hosting means wrestling with edge functions, middleware, and Node.js server configurations that nobody warned you about.


### When Next.js Actually Makes Sense

- **Content-heavy marketing sites** requiring frequent updates and SEO juice
- **E-commerce platforms** with global audiences and personalization needs  
- **Media companies** balancing paywalls, ads, and content delivery
- **Well-funded startups** with dedicated DevOps who can manage the complexity

### The Myth

Next.js has been marketed as a general-purpose framework for a long time, but it's mostly a **specialized tool for content-driven businesses**. It trades most of the engineering simplicity for great marketing capabilities, which I think it's a great choice for businesses, where marketing brings a lot of revenue but not for solo developers or small teams building internal tools or simple applications.

The pendulum is swinging back. Teams are rediscovering that most web apps don't need universal rendering, edge functions, or 500KB of JavaScript. Sometimes a boring Rails app or a simple static site generator delivers more value with less drama.

Choose Next.js when its complexity serves a clear business purpose. Otherwise, you're buying a bulldozer to plant a garden.
