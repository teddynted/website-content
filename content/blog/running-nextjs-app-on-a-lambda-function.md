### So what's Next.js?

Next.js it's a server-side rendering ReactJS framework. SSR it's a process of rendering a javascript client-side website into a static html and css on the server. Poor SEO is major disadvantage of a traditional ReactJS applications, that's where SSR comes in.

### Benefits of SSR

* Fast rendering websites.
* Consistent SEO performance.

### Why SEO?

* It Increases traffic to your website.
* It's a cost-effective marketing strategy.
* It increases sales and leads.

### Prerequisites

* Node 12
* AWS CLI
* AWS account

I will set it off by creating a <a class="markdown-link" href="/blog/aws-lambda-layers">lambda layer</a>, run theses commands in your development folder:

```bash
mkdir nextjs-aws-on-lambda-layer && cd nextjs-aws-on-lambda-layer 
mkdir nodejs
cd ..
touch command.sh
```

Add a bash script with the following commands:

![alt text](https://nextjs-portfolio.s3.amazonaws.com/layer-shell-script.png "AWS Lambda Layers")
