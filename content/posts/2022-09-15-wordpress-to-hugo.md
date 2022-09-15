---
title: WordPress to Hugo
tags:
- aws

date: 2022-09-15

---




# Migrating Wordpress to Hugo

After my first full month hosting this blog on an EC2 instance (link here) I was billed roughly $20. 
Coincidently the same day my bill arrived I found a post on Reddit suggesting to someone to move away from Wordpress and to utilise a static webpage generated via Hugo

## What is Hugo 
Hugo is a static site generator written in Go which produces incredibly fast and efficient HTML pages from Markdown files
More here (link here)

## Installing Hugo
Installing Hugo was pretty straightforward. I already had Chocolatey installed so went the easy approach and simply ran 

‘’’powershell
choco install hugo -confirm
‘’’

There are also options to install via scope or  if you have go installed, compile via GitHub 

I then used the QuickStart guide to setup a example website so I could get an idea of what I needed


‘’’powershell
hugo new site quickstart
‘’’

Next was time to choose a theme. I went over to themes.gohugo.io and had a look around, finally choosing Hello Friend (a little bit because it reminded me of Mr. Robot)

Installation of the theme was very straightforward, simply submodule the git location to your site folder and that’s it. 

## Migrating Wordpress 
I needed to export my existing Wordpress posts to markdown files in order to import them into my new Hugo site
There were a lot of options available, but I eventually ended up using this  https://github.com/SchumacherFM/wordpress-to-hugo-exporter
custom Wordpress plugin to convert the posts to markdown. 

Once I had the files I needed to do a bit of cleanup such as resizing and relinking my images and cleaning up some leftover html code, it didn’t take me all that long to complete luckily.

Hugo allows you to deploy a local web server so you may launch your site from your local machine. Once I was happy with how it looked I was able to package up my site using the command Hugo -D

This created a folder named Public inside my site folder. The contents of this Public folder is what I needed to deploy to my S3 bucket that will become the host point for this blog

I created an S3 bucket with the same name as this blogs url, turned on public access and static webpage hosting and set the default page to index.html

Next we need to upload the contents from your Hugo/Public folder to your new S3 bucket. Once that’s done you’ll have a fully functional Hugo website running on AWS!

## Domain
I wanted this site to work with my purchased domain name and to also work on HTTPS.
I utilised AWS ACM console to create a certificate for my domain that I could use to secure my blog. The process for that is rather straightforward following this article from AWS https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html
I chose the DNS validation option. This certificate is free to secure services you deploy on AWS.

With certificate sorted I moved onto creating a cloud front distribution. I pointed cloud front to the s3 bucket I uploaded my Hugo files to earlier and secured the domain with the certificate I’d just created.

And that’s it! You know it’s worked because that’s how you are reading this blog. 
I’ll created a new post soon detailing my code pipeline work to deploy posts on this blog which I am very proud of.


