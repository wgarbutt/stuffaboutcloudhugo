---
title: Cloud Resume Challenge – Part 1
series: ["Cloud Resume Challenge"]
series_order: 1
date: 2022-07-14
tags:
- cloudresume
- aws

---

I'm the kind of person who is never done with learning and trying to align my skillset with industry demand. I've done a lot of VMWare training and certification over the years and decided it was time to branch out to one of the popular public cloud offerings.

While researching Amazon Web Services courses and certifications I stumbled across the Cloud Resume Challenge. The challenge is a 16 step project designed to show off the skills typically required of a cloud engineer, specific to the flavour of cloud provider you are interested in (AWS, Azure, or Google).

Here are the steps I took as part of the challenge, they are referenced from the Cloud Resume Challenge website

<https://cloudresumechallenge.dev/docs/the-challenge/aws/>

### Step 1: Certification

For this I had to sit and pass the AWS Cloud Practitioner certification. (More details on this certification [here][1])  
I used Stephane Maarek's Ultimate AWS Certified Cloud Practitioner&nbsp;&#8211; 2022 Udemy course ([Link][2]) as well as his 6 practice exams ([Link][3]).  
I found Stephanes course to be tightly packed and didn't drag on too much, if you didn't quite understand a particular topic you could always dive into it more using AWS's available resources.  
The practice exams gave very detailed explanations of the available answers and highlighted areas where I needed to study more.  
I sat the exam from home and was told to check in half an hour before my exam time. I really needed that extra half an hour as the identity and room setup verification took nearly that whole time.  
I passed the exam with 803/1000  




### Step 2: HTML Resume

When I first left school I signed up to do a Network engineering course. It had a very small module on web design and coding, that was the last time I ever wrote in HTML. A poster on Reddit suggested finding some inspiration on a site called [Codepen.io][4]. There I found a clean simple template that I could use as a starting point.  
I saved my code to my GitHub [here.][5] (Spoiler alert, you can see the finished project [here][6])

### Step 3: CSS

My resume needed to be styled with CSS. I would be the worst choice for a game of Pictionary and my design imagination is non-existent. I scoped out [Codepen.io][4] again and luckily found a suitable template for my css file. I made a number of adjustments and learnt quite a bit about css while I was at it.  
I link to this file on my GitHub [here][7]

### Step 4: Static Website

My HTML files are stored on an AWS S3 bucket configured to be a static website. I found this to be pretty straight forward to setup. Here is the steps I needed to do to configure a AWS S3 static website.

  * Deploy your empty bucket. You'll need to name the bucket the same name as your domain. AWS uses the host header in the web request to route traffic to the appropriate bucket
![](/Images/CloudResumeChallenge/image1.png)

 
  * Configure the bucket for static web hosting. On the properties of your bucket you should see an option &#8220;Static website hosting&#8221;, Select the option &#8220;Use this bucket to host a website&#8221;, enter &#8220;index.html&#8221; as the index document

  * Your bucket is now configured for static website hosting and you should have an S3 website URL listed at the bottom of the bucket property page

### Step 5/6: DNS and HTTPS

I purchased myself a domain name [will-garbutt.me][8] for around $50. The challenge suggests using AWS Route 53 to host the DNS for your domain but since I am also doing some Terraform learning and playing with deploying resources using code I found that AWS considers a domain that was stood up, deleted, and stood up again to be two domains and charging me $10 each time I try. 

For this I used Cloudflare as it has a well documented Terraform provider module and is free no matter how many times I have deployed and destroyed resources. In my Terrafrom GitHub Repository ([link][9]) I have both AWS and Cloudflare code to setup DNS.

Cloudflare also has a simple way to enable HTTPS with just a click of a button. 

  


Part 2 will be coming soon!

 [1]: https://aws.amazon.com/certification/certified-cloud-practitioner/
 [2]: https://www.udemy.com/share/103a093@Xyy5BUVKwUTNgK4T9fYuttnh2s3pKp7w8ogebGWcUkt4f_lZ2kycxHVTRAWWeMC-wQ==/
 [3]: https://www.udemy.com/share/103e7s3@nproIormrwH46Z8rw68C34SfBvUi69yf-VseCS0INp1bETewtEUfUJFqxveq6m9CCg==/
 [4]: http://codepen.io
 [5]: https://github.com/wgarbutt/CloudResume/blob/main/index.html
 [6]: https://will-garbutt.me
 [7]: https://github.com/wgarbutt/CloudResume/blob/main/style.css
 [8]: http://will-garbutt.me
 [9]: https://github.com/wgarbutt/Terraform