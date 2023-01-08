---
date: "2020-02-10"
title: "Preparing Your S3 Bucket to Host a Static Website Using AWS CLI"
tags: ["aws"]
author: "masoudkf"
description: "Your S3 bucket is more versatile than it actually looks. For one, it can host a static website with an actuall domain name. There are, however, a couple of steps to take before your static site is up and running. Let's find out!"
---

<a target="_blank" rel="noopener noreferrer" href="https://aws.amazon.com/s3/?sc_channel=PS&sc_campaign=acquisition_CA&sc_publisher=google&sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CStorage%7CS3%7CCA%7CEN%7CText&sc_content=s3_e&sc_detail=amazon%20s3&sc_category=Storage&sc_segment=293634539894&sc_matchtype=e&sc_country=CA&s_kwcid=AL!4422!3!293634539894!e!!g!!amazon%20s3&ef_id=Cj0KCQiAm4TyBRDgARIsAOU75sp1nzvOImz6Fe3irQ3EtEudRvZlzuymfaK2OehnuhkXV0Ic5YF82RQaAjh5EALw_wcB:G:s">Amazon S3</a> is a great storage for your files. You can upload your files to what is known as a `bucket` and then access later. You can also have folders in your bucket to organize your files. Just think of a `bucket` as a root folder for your files in the cloud. It's more complicated than that, but I think it will suffice for now. And of course, you can have multiple buckets. 

One of the great things about S3 is that it can host your static website. You'll put your files in a bucket, and with a few steps, bam! You've got yourself a static website with an actuall address publicly accessible on the Internet. And it's very cheap; especially if you're eligible for AWS free usage tier which you'll get once you open an account for a year. You can check the pricing <a target="_blank" rel="noopener noreferrer" href="https://aws.amazon.com/s3/pricing/">here</a>. Just remember, we'll be using S3 Standard since we want high availability for our website. Enough with this. Let's talk about how to host a static website using AWS S3. Before we start, there are some prerequisites:

- An AWS account. In case you don't have it already, start <a target="_blank" rel="noopener noreferrer" href="https://aws.amazon.com/console/">here</a>
- An IAM user with AdministratorAccess. Follow these <a target="_blank" rel="noopener noreferrer" href="https://docs.aws.amazon.com/mediapackage/latest/ug/setting-up-create-iam-user.html">instructions</a> for creating one (the user doesn't actually have to be an admin user; but specifying fine-grained policies for a user is not something this tutorial is about. So, admin it is!)
- AWS CLI installed and configured on your machine. This is the AWS Command Line Interface which we'll be using throughout this tutorial. Follow these <a target="_blank" rel="noopener noreferrer" href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html">instructions</a> for your operating system if you don't already have it


All right. Let's do this. But before we start, I should point out that this tutorial is all about the command line. You can definitely do all of these steps using the AWS web console; however, we're not going to do that here. All right, let's begin now. First we want to create an `S3 bucket`. Open up your terminal and type this command. We use `mb` for make bucket:

```bash
aws s3 mb s3://myfantasticwebsite
```

Pay attention to the `s3://` prefix; it's important that you use it, otherwise the AWS CLI will yell at you. `myfantasticwebsite` is the name of your bucket and it will appear as part of your website address later on. Another thing, once you've created a bucket you can't change its name, so choose it wisely. Also, the name of your bucket **must** be unique through the entire AWS system, so you will get an error if your chosen name already exists. You can now go to the <a target="_blank" rel="noopener noreferrer" href="https://s3.console.aws.amazon.com/s3/">S3 console</a> and see your bucket sitting there. Well done!


However, this bucket is not yet ready to host a static website. For one, this bucket, by default, is not publicly accessible. That's against the core principle of a website that should be accessible by everybody. You can test this by first creating an HTML file and then upload it to your bucket. Let's do it:

```HTML
<p>Hello from AWS S3</p>
```

This is like the most simple HTML file ever; but it doesn't matter for now. Let's upload it to our bucket:

```bash
aws s3 cp index.html s3://myfantasticwebsite
```

Remember that the `index.html` file should be in your current directory. Otherwise, you'll have to address it differently. Now head back to the console and go to your bucket. You'll see that your `index.html` file is there. Click on the file and on the next page, you'll find a URL under _Object URL_. That's the URL to your file. Click on it, and ... Yes, you'll be presented by an AccessDenied page. Why? Because the file is not publicly available yet. It's private and just you, as the bucket owner, can access it. Let's fix this.

In order to make your objects (your files in the bucket) publicly accessible, you'll need something known as the **Bucket Policy**. It's a <a target="_blank" rel="noopener noreferrer" href="https://json.org">JSON</a> file in which you'll specify how you want your objects to be accessed. Let's create this file for our static website:

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForMyStaticWebsite",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::myfantasticwebsite/*"
        }
    ]
}
```

Here, `Version` is not the version of your website or file or anything. It's the version of this JSON structure provided by Amazon. In the future, Amazon might present a new structure for bucket policies and we'll have to use that. So, don't worry about it. The `Sid` part is optional and you can name it whatever you want. The `Principal` part indicates who this policy will be applied to--in this case, it's everyone. The `Action` part is the action you want to give to your principals, and the `Resource` is every object (file) in our bucket. Note the name of the bucket and the `/*` which means everything in this bucket.

Go Ahead and save this file with the name `policy.json` in the current directory. Then open up your terminal or command line again and put this:

```bash
aws s3api put-bucket-policy --bucket myfantasticwebsite --policy file://policy.json
```

Note that we used `s3api` instead of `s3` here and didn't use the `s3://` prefix. We also had to use the `file://` prefix for the JSON file. Now once again, go ahead to your bucket, click on the html file, click on the Object URL link and bam! You can now see your HTML file in the browser. How cool is this?

As cool as this is, it's not still a website as we normally know it. True, you now have a URL to your `index.html` and can pass it on to everyone and they can in fact access it on the Internet. But it's not a website. For one, just try deleting the `/index.html` part of the URL, because no one accesses a website with a `/index.html` at the end, right? Now try to render the page. You'll, of course, get another Access Denied page. For another, try putting a non-existent address, like `<YOUR_DOMAIN_WITHOUT_INDEX.HTML_AT_THE_END>/somethingdummy` and reload the page. You'll again get the Access Denied page, where, normally, you would expect to see a 404 page! So, it feels like a website, but it's not. Let's fix this.


In order to fix this issue, we need another JSON file that represents very basic website characteristics: `index.html` and `404` pages. Let's create the JSON file:

```JSON
{
    "IndexDocument": {
        "Suffix": "index.html"
    },
    "ErrorDocument": {
        "Key": "404.html"
    }
}
```

This is a very basic configuration that says, "OK, my index is `index.html` and my error page is `404.html`". Remember to create a `404.html` page. For simplicity sake, let's create a very simple `404.html` page:

```HTML
<p>Page not found!</p>
```

Save the JSON file as `webconfig.json` in the current directory, then open your terminal again and put this:

```bash
aws s3api put-bucket-website --bucket myfantasticwebsite --website-configuration file://webconfig.json
```

Your website is ready now, but a new address has been created. Head back to S3 console, go to your bucket and from the top menu, select **Properties**. You'll see a box named **Static website hosting** and it should be checked since we've already configure it. Click on it and you'll see your configuration there: `index.html` as the Index document and `404.html` as the Error document. On the top, you'll see a link named Endpoint. Click on the link, and voila... You've got yourself a static website. And this is actually a website. It doesn't need the `/index.html` and if you try accessing a page that doesn't exist, you'll get your 404 page.

The address is not very appealing, I know. But if you have a domain name, you can change your DNS settings to point to this address. You can also get an SSL certificate for your static website hosted in a S3 bucket using another AWS service called CloudFront. It's kind of beyond the scope of this post--It's already very long. But I will create another post and show you how you can achieve these things.
