# ecr-cleaner
Deletes old images from ecr

This clean up a specific repository as well as all repos within an aws account.
This works perfectly with images which are tagged like 0.1.2-b12-g43gsdf, it's version-jenkinsBuildNumer-gitHash.

|Order example|
|-------------|
|0.1.1-b44-g3g9s7|
|0.1.2-b3-g89hjf|
|0.1.2-b10-g4fs7h|
|0.1.2-b12-g9j6ng|

### Algorithm
1. Retrieve repo from ecr
2. Get repo images
3. Add all images without tags to deletion
4. Sort the remaining images in alphanumeric order with respect to their integer parts
5. Add n oldest images to deletion
6. Delete images from the repository

### Installation
    go get github.com/WeltN24/ecr-cleaner

### Default values
    aws.region = eu-central-1
    dry-run = false
    amount-to-keep = 100

### Examples
clean up all repos

`ecr-cleaner -aws.region eu-west-1`

clean up my-awesome-repo

`ecr-cleaner -aws.region eu-west-1 -repository my-awesome-repo`

go for a dry run

`ecr-cleaner -aws.region eu-west-1 -repository my-awesome-repo -dry-run true`

leave n images in repo

`ecr-cleaner -aws.region eu-west-1 -repository my-awesome-repo -amount-to-keep 5`


### Deploying as lambda to aws
If you wish to clean up your repositories periodically you can to this with the help of terraform. 

in the root of the repo, this creates an archive which will be 

1. You have to fork the repo
2. execute `make package`
3. go to into terraform folder
4. set up the needed variables
    * `cron` expects a string in [aws cron syntaxt](http://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) (`0 3 1 * ? *` run lambda at 3am 1. of each month)
    * `aws_region` is the region in which you want to deploy the lambda
    * `repo_region` is the region in which you store your ec2 repositories
    * `repository` is the repo you want to process
    * `dry-run` (boolean) if you want to dry run
5. run terraform

If you want to persist the state it's the easiest way to create a shell script and write the remote state to s3. Here is an example:
    
    #!/bin/bash
    terraform get -update
    terraform remote config \
        -backend=s3\
        -backend-config="bucket=maintaince" \
        -backend-config="key=ecr_cleaner/terraform.tfstate" \
        -backend-config="region=eu-central-1"
        
Execute the script the get remote state from s3 or create one and execute terraform afterwards.

### Run as Docker container

Build:

	docker build -t ecr-cleaner .
	
Run:

	docker run -e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -it --rm ecr-cleaner -aws.region eu-west-1