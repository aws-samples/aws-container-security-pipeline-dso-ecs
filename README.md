## aws-container-security-pipeline-dso-ecs

1.	Clone the repository.
git clone https://github.com/aws-samples/aws-container-security-pipeline-dso-ecs.git

2.	The demo is using nested AWS CloudFormation templates and some configuration files that need to be uploaded to an Amazon Simple Storage Service (Amazon S3) bucket. Create a bucket in the same Region that you’ll create the demonstration in, the bucket doesn’t need to be public facing.
aws s3 mb s3://<bucketname> --region <region name>

3.	Security Hub is used in this demonstration and has an integration with Trivy. To enable this, go to Security Hub in the AWS Management Console and select Integrations from the menu on the left. Filter for Aqua Security and select Accept findings.
If you do not have Security Hub setup in your account, please follow the instructions here.

4.	Upload files in the environment directory directly to the new bucket. This can be done via the Console or via the CLI as shown.
aws s3 sync environment s3://<bucketname> —region <region name>

5.	There are 2 zipfiles required which need to be created in the following way.
		```cd initial-commit
 
		   zip -rj ../devsecops/initial-commit.zip ./*

		   cd ../trivy-build 

		   zip -rj ../devsecops/trivy-build.zip ./* 
		```

6.	Upload the entire devsecops folder to the bucket.

aws s3 sync ../devsecops s3://<bucketname>/devsecops —region <region name>

7.	Create a stack using the container-devsecops-demo.yml template and use the bucket name of the bucket that you just created as the DevSecOpsResourcesBucket Parameter.
Once all the nested stacks have completed building, you’ll have a CICD (Continuous Integration, Continuous Delivery) pipeline with built in configurable security tools, that deploys new images to an Amazon ECS Fargate cluster.

Now it’s time to clone the application repository to your local machine or to an AWS Cloud9 instance, if that’s what you’re running. You need the AWS Command Line Interface (AWS CLI) and Git commands available to update the application.

1.	Clone the CodeCommit repository container-security-demo-app to your local machine or AWS Cloud9, depending on what you want to use.

git clone https://git-codecommit.eu-west-1.amazonaws.com/v1/repos/container-security-demo-app
      
2.	We also want an application to use with the pipeline. We will clone the popular application 2048 from Github to use.
git clone https://github.com/gabrielecirulli/2048.git

NB: the 2048 application comes under the MIT license.

3.	Switch into the new Git repository that you cloned from CodeCommit.

cd container-security-demo-app

4.	Switch to the development branch.

git checkout development

5.	Now, it’s time to fully populate the application as only a Dockerfile and some additional required nginx settings are committed initially. 

cp -r ../../2048/* .
git add .
git commit“-m "nice commit message goes here"
git push origin development


6.	At this stage, you have committed your changes to the repo but not instigated the pipeline as it’s waiting for a pull request. Create the pull request:

aws --region <region> codecommit create-pull-request \
--title "Uploading new application" \
--description "Please Review" \
--targets repositoryName=container-security-demo-app,sourceReference=development,destinationReference=main

7.	Now if you go to look at the pipeline by selecting CodePipeline from the AWS Console and select the container-security-demo-pipeline, you can see the change being submitted through the pipeline.

The image is tested for vulnerabilities, secrets, and linting errors. If all tests pass, then the container image is built, tagged, and added to the Amazon ECR private repository.

If there are any vulnerabilities at the specified failure level or above, then the pipeline stops and the image won’t be added to the Amazon ECR repository.

At the time of testing the following image was used in the Dockerfile to replace the default image “public.ecr.aws/nginx/nginx:1-alpine-perl” and the ADD commands in the Dockerfile were replaced by the COPY command. With these 2 substitutions the pipeline completed successfully. 

8.	If there are vulnerabilities only at the audit level and below, those are logged directly with Security Hub so that there is an audit trail for the future. To see these, go to Security Hub and add a filter on findings (Product Name is Aqua Security).
Changing the image for a new version is easy by editing the Dockerfile.

Now you have a pipeline with some open-source tooling that enables you to add security checks for your container images as they are built, and prevents images with CRITICAL or HIGH (depending on the threshold you set) vulnerabilities from being used as part of a task.

If you want to update the config you can clone the container-security-demo-config repo and commit changes to master branch directly.

Cleanup

To clean up the stack please complete the following actions:

•	Delete all objects from the Amazon S3 bucket that was created for CodePipeline, which is found in the AWS CloudFormation stack for DSO-Initial-Pipeline under Resources as the Pipeline Bucket (i.e., container-security-demo-<accountid>-<region>-artifacts).
•	Delete all images from the Amazon ECR Private repositories container-demo-security-sample and container-security-demo-trivy.
•	Now go to the AWS Cloudformation console and delete the stack that you created from the AWS CloudFormation yaml.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
