---
title: The new static web stack made easy with Terraform and Jekyll over TLS
---

## Motivation

Until recently, my old web property languished on a past end-of-life Rails Heroku stack. It was slow to build and served over plain HTTP. Time for an upgrade.

I taught myself Terraform and Jekyll by migrating peterkong.com to this new stack. I'm convinced it's an ideal configuration for modern static sites: S3-backed CloudFront offers blazing fast page loads, SSL/TLS configuration is one-and-done, and your cloud architecture is expressed in code, so you never need click through tedious AWS interfaces trying to remember how you configured a firewall setting or bucket rule. And you have complete control over every part of your stack.

I've captured my learnings in two public repos: [terraform-static-site](https://github.com/happythenewsad/terraform-static-site) and [jekyll-pkcom](https://github.com/happythenewsad/jekyll_pkcom)

Feel free to jump right in. The rest of this post will cover some more detailed thoughts and observations on working with Terraform. Code snippets reference [terraform-static-site](https://github.com/happythenewsad/terraform-static-site).

## Terraform's tactical merits
Terraform expresses cloud configurations (i.e. servers, load balancers, datastores, permissions) in code. This is super great because non-trivial cloud configurations become complex very quickly. Depending on your cloud provider\*, cloud configurations may not be versioned, or not versioned in an easily readable way. A team member may change a configuration and you may not realize it until something breaks. If you adhere to Terraform best practices, every single aspect of your cloud configuration will be versioned and controlled, making infrastructure debugging and rollbacks straightforward. 

Terraform deployments are idempotent, so existing architecture is preserved if reflects what's in your Terraform specification.

## Notes and learnings

The official Terraform [documentation](https://www.terraform.io/docs/configuration/index.html) is indispensable and generally quite good, but it can be overwhelming when starting out.

I found it helpful to start with a minimal working Terraform configuration (say, a single `.tf` file that deploys a bare EC2 instance), and extend iteratively.

Let’s look at [terraform-static-site](https://github.com/happythenewsad/terraform-static-site): it has a single Terraform directory, `s3`, with just 3 files inside. Yep, that's all you need. Like Git, Terraform only cares about the directory you tell it to care about. If you run `cd s3 && terraform init .`, Terraform will start managing artifacts within `s3/`, and only `s3/`. Terraform can manage multiple directories through Terraform modules, but those are out of the scope of this minimal example.

Let's take a quick look `s3.tf`:

	terraform {
		...
	} 
	provider "aws" {
		...
	}

	resource "aws_s3_bucket" "pkcom" {
		...
	}

	resource "aws_cloudfront_distribution" "pkcomCFDistro"{
		...
	}

There's some boilerplate, specifying AWS as our cloud provider, then just two resources: an S3 bucket and a CloudFront distribution. Sadly, Terraform [does not currently support](https://github.com/hashicorp/terraform/issues/571) dynamic names, so you'll have to change "pkcom" and "pkcomCFDistro" to something relevant to you.

That's it: get used to resource blocks; they're Terraform's bread and butter. Resources are tied to a specific cloud provider, so if you're not using AWS, you'll need to use Azure resource blocks, for example, which will have configuration options that differ from AWS.

You could populate `s3.tf` with hard-coded variables, like S3 bucket names, but it's best to think of your main `.tf` file as a template, and store variables separately. Let's check out `variables.tf`:

	variable "region" {
	  default = "us-west-2"
	}

	variable "property" {
	}

	variable "bucketName" {
	}
	...

This is how you declare variables in Terraform. I've abstracted the *values* of the variables into `terraform.tfvars`. This is a way to keep secret values out of version control, but it's not required.

When you run `terraform apply`, the Terraform CLI will automatically inject variables into your template file(s).

That's it! Once you've populated `terraform.tfvars` with your specific values, you can deploy with `terraform apply`.

### http or https?

It's easy to misconfigure the connection between S3 and and CloudFront. After trial and error I found that it's best to have CloudFront fetch S3 objects over http, but accept only https requests:

	origin_protocol_policy = "http-only"
	...
	viewer_protocol_policy = "redirect-to-https"

Some recipes will specify 

	origin_protocol_policy = "match-viewer"`

which causes CloudFront to raise HTTP 504 errors in my experience. In general, it's handy to enable an S3 logging bucket when debugging S3 <-> CloudFront issues.

### custom domain / cert
[terraform-static-site](https://github.com/happythenewsad/terraform-static-site) is configured to use a custom domain, so your web property can be accessed via a normal domain instead of xyz.cloudfront.net. Doing this in AWS requires a custom SSL/TLS cert. You need out-of-band approval for a cert. Log into the [AWS Certificate Manager](https://console.aws.amazon.com/acm) to create one, then add it's ARN in `terraform.tvars`. If you prefer to host domains outside of AWS's Route53 service like I do, you'll also need to follow [this CNAMEing procedure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/CNAMEs.html). Verifying ownership of your domain is easy, but manual and out-of-band. It took about 10 minutes for AWS to recognize ownership for my domain after I updated CNAME records through my 3rd party DNS service.

### 503 gotcha
If you're getting 503's from CloudFront, it may be because you need a `custom_origin_config` within your `aws_cloudfront_distribution` resource block. Luckily, [terraform-static-site](https://github.com/happythenewsad/terraform-static-site)'s configuration should work out of the box.



And if you have any feedback or suggestions, please add an issue or open a PR on the [Github repo!](https://github.com/happythenewsad/terraform-static-site)


\* Terraform supports most cloud providers; this post uses AWS. 
