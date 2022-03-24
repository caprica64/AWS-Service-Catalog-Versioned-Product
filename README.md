# sharedservicecatalog
AWS Shared Service catalog

This project is to design an AWS Shared Service Catalog proof of concept where a central Portfolio (DevTools) account has sets of portfolios and products. Those portfolios should be shared with an AWS Organization OU that contains DevOps accounts for projects.

These projects will have two or more product environment accounts where products are deployed, such as VPC, network routes and subnets, ECS and others. When you deploy an application service such as an ECS Service/Task, it should also deploy the pipeline for versioning those workloads.

