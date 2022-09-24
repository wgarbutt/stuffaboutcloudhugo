---
title: Hugo Publishing Pipeline
tags:
- aws
description: "Using AWS CodePipeline and Lambda to fully automate deployment from Commit to Publish"
date: 2022-09-15
image: https://stuffabout.cloud/posts/hugo-pipeline/cover.png

---

Now that I have this blog fully migrated over from Wordpress to Hugo pages I wanted to share with you the deployment pipeline that I created that takes care of publishing new posts.

## Overview


![](/Images/HugoPipeline/pipeline-flow.png)
The AWS Code Pipeline is comprised of four stages that automatically take source code, compile it from markdown pages to HTML, stores it, and refreshes the CDN cache.


#### Source
The code Pipeline is triggered when a commit is pushed to the Github repository that stores the source code for this blog ([here][1] if you are interested).
I setup a GitHub (version 2) connection provider and followed the prompts to allow Code Pipeline to monitor a specified repository and branch for any changes.


#### Build
The next step will trigger a CodeBuild Docker Container to be created. This small container allows for Hugo to be downloaded, installed, and run against the downloaded source files. 

The container is a prebuilt Amazon Linux 2 install and takes around a minute to stand up.

Once the container is fully initialized Codebuild will look for and run a user defined script called buildspec.yml

``` yaml
version: 0.2
phases:
  install:
    commands:
      - apt-get update
      - echo Installing hugo
      - curl -L -o hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.102.3/hugo_0.102.3_Linux-64bit.deb
      - dpkg -i hugo.deb
  pre_build:
    commands:
      - echo In pre_build phase..
      - echo Current directory is $CODEBUILD_SRC_DIR
      - ls -la
  build:
    commands:
      - hugo -v
artifacts:
  files:
    - '**/*'
  base-directory: public

  ```
This very simple script is split up into stages, Install, Pre-Build, Build and Artifacts

Install Phase:
* Updates the package lists from the Linux update repository
* Downloads and installs Hugo

Pre_build:
* Copy over source artifact (the source files downloaded from GitHub)

Build:
* Runs the Hugo build command to convert Markdown source files into nicely deployable HTML pages

Artifacts:
* Makes the converted HTML pages available to CodePipeline ready for the next stage 


#### Deploy
This stage takes the artifact from the build stage and writes the files to the S3 hosting bucket that runs this blog



#### Refresh
The issue with having this blog hosting via a CloudFront distribution is that the cache takes some to refresh. I found a couple of posts on Reddit and other blogs suggesting that a simple Lambda function could be used to force an invalidation of the CloudFront cache.

``` python
****def lambda_handler(event, context):
    job_id = event["CodePipeline.job"]["id"]
    try:
        user_params = json.loads(
            event["CodePipeline.job"]
                ["data"]
                ["actionConfiguration"]
                ["configuration"]
                ["UserParameters"]
        )
        cloud_front.create_invalidation(
            DistributionId=user_params["distributionId"],
            InvalidationBatch={
                "Paths": {
                    "Quantity": len(user_params["objectPaths"]),
                    "Items": user_params["objectPaths"],
                },
                "CallerReference": event["CodePipeline.job"]["id"],
            },
        )
    except Exception as e:
        code_pipeline.put_job_failure_result(
            jobId=job_id,
            failureDetails={
                "type": "JobFailed",
                "message": str(e),
            },
        )
    else:
        code_pipeline.put_job_success_result(
            jobId=job_id,
        )

```
When the stage is triggered, it passes the pre-defined `distrubutionID` and `objectPaths` parameters to the Lambda function. The function uses [Boto3][2] SDK for Python to call AWS CLI commands, in this case `cloud_front.create_invalidation`

#### Deploy!
And with that, all I have to do now is save and commit this `.md` file and push my changes to GitHub and within a few minutes this post will be live!






[1]: https://github.com/wgarbutt/stuffaboutcloudhugo
[2]: https://docs.aws.amazon.com/pythonsdk/?id=docs_gateway