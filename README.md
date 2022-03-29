# AWS Service Catalog Versioned Product

This project is to design an AWS Shared Service Catalog proof of concept where a central Portfolio (DevTools) account has sets of portfolios and products. Those portfolios should be shared with an AWS Organization OU that contains DevOps accounts for projects.

These projects will have two or more product environment accounts where products are deployed, such as VPC, network routes and subnets, ECS and others. When you deploy an application service such as an ECS Service/Task, it should also deploy the pipeline for versioning those workloads.


## Versioned products in Service Catalog

This repository keeps to Service Catalog templates, one for a sample **Portfolio** and another for a **versioned product**. All products belong to one or more Portfolios and can be versioned for the evolution of the products.

Every time there is an update in a product, such as an S3 bucket, an updated **CloudFormation** template is pushed to the product repository and the **CodePipeline** will be notified and process the product update in the Service Catalog. The updates also contain a config.json file that describes the new product version and description. 

2022-03-29 > There is no provision yet in this project to mark a version as deprecated so no-one can use to launch a specific version.

## Using the templates

There is two CloudFormation stacks to create first a Portfolio and last a Product.

2022-03-29 > There is no provision yet to create a CodeCommit repository with a sample template and config.json file. For now this needs to be added manually as instructed. 
2022-03-29 > The artifacts bucket needs to be created first and it the ACL needs to be enabled. If you have created a CodePipeline before, the standard bucket in the region will be already setup. On task of this project is to be able to create an artifacts bucket with Bucket policy instead of enabled ACL and uploading a sample CloudFormation product template as a Hello World.

### Portfolio

First, create the Portfolio using the sample **portfolio-template.yaml** template. The name of the Portfolio and wait it completion. Write down the stack name to reuse when creating products for this portfolio.

Ideally, a portfolio contains a group of related products and components. For example, we could have a **Foundation** portfolio that has **VPC** and **NAT Gateway** products. A team can use them to build VPC with or without Nat Gateway support.

Another portfolio that build over the foundational one is a **DevOps** portfolios with ECS Cluster, repositories and multiple **Services**, each one being a separated Product. The idea is that users can consume and build workloads by setting up AWS products themselves as they like.

2022-03-29 > The Portfolio does not have permissions added to it at this time, so once it is create, go ahead and add an IAM user, group or roles that are allowed to see and launch the products on it. This is an improvement we need to do.

### Product

Now we create one or more products using the template **Product1-ServiceCatalog-template.yaml** template in CloudFormation.

**Important**: Since we don't have a bucket yet to hold the artifacts, please create one for this product. Buckets can be shared to more than one product as well, we just need different product names. However, it is possible or desirable to build an S3 Bucket product via automation with proper bucket policies that allow only to the product owner team (or portfolio team) to protect unauthorised changes or access. Once we have this bucket created, go ahead and enable ACL in the Permissions tab. This is not the production grade security level we would like to use, but it is enough to learn the process. We will try to fix this in the future.

There is no support to automatically upload a sample product template to S3 at this time, so go ahead and upload the **product.yaml** file to bucket. Ideally this will be updated to take from the Git repository instead.

When launching the Product stack, there are a lot of parameters to fill in about the product owner, support contacts and links. Most are free to use whichever makes sense in the project but make sure the Portfolio stack name matches the stack name used in the Portfolio step before so the product can be attached to it. The key here is that one or more products are attached the Portfolio.

Currently, this does not support attaching to more than one Portfolio but the template can be adapted to have more than one portfolio parameter. Alternatively, one can deploy a generic product > portfolio attachment stack multiple times for that.

The product template will create the following elements in AWS:

- Policies for the CodePipeline execution: Policies grant permission to buckets, CloudFormation and Service Catalog actions. This can be improved with fine tuned policies to the same bucket this product would create later on, the same for the CF stack of the product and other resources.
- CodeCommit repository: The stack will make a blank repository. At this point, please upload the **product.yaml** and **config.json** files to it. No other files are needed.
- CodePipeline: A new pipeline with two states will be created, where the first stage will get source updates from the repository and the second deploy to a Service Catalog action.

Once the pipeline is ran you should have an updated product version. The sample product is a Bucket S3 with some tags. To test the product, go the Service Catalog products view, find and launch the first version of it. An S3 Bucket should be created in about a minute. Go check the bucket properties and notice the Tags matches the template.

To test an update, go ahead and update both the Tag parameters in the S3 product template and the version name and description on the config.json file. Push the changes to the remote CodeCommit repository and the CodePipeline will detect and start running the update.

Go back to the Service Catalog product list and you should see the new version listed now. You can try going back to the launched product list and request an updated version. The new template should be installed and you should see the new tag value on the bucket properties.


<img width="1324" alt="Screenshot 2022-03-29 at 18 50 34" src="https://user-images.githubusercontent.com/63541433/160713150-7b863bcb-3ae7-48be-a0e1-edf2fac26e76.png">
