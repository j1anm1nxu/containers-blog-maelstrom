# Manage your EKS Cluster with CDK
This repository holds the skeleton code where you would start the journey to *[Manage your EKS Cluster with CDK](http://demogo-multiregion-eks.s3-website.ap-northeast-2.amazonaws.com/ko/)* Hands-on Lab.

Please clone this repository and start [the workshop](http://demogo-multiregion-eks.s3-website.ap-northeast-2.amazonaws.com/ko/) to play with the lab. :)


## Related Repository
* [Skeleton Repository](https://github.com/yjw113080/aws-cdk-eks-multi-region-skeleton): You would clone this repository and build up the code as going through the steps in the lab.
* [Full-code Repository](https://github.com/yjw113080/aws-cdk-eks-multi-region): Once you complete the workshop, the code would look like this repository! You can also use this repository as a sample code to actually build CDK project for your own infrastructure and containers.
* [CI/CD for CDK](https://github.com/yjw113080/aws-cdk-multi-region-cicd): Fabulous CDK team is working on providing CI/CD natively, in the meantime, you can check out simple way to do it with AWS CodePipeline and CodeBuild.
* [Sample App for Multi-region Application Deployment](https://github.com/yjw113080/aws-cdk-multi-region-sample-app): In third lab of [this workshop](http://demogo-multiregion-eks.s3-website.ap-northeast-2.amazonaws.com/ko/), you will deploy your application in your developer's shoes. This repository holds the sample app to deploy. The sample simply says 'Hello World' with the information where it is hosted.

## Useful commands

* `npm run build`   compile typescript to js
* `npm run watch`   watch for changes and compile
* `npm run test`    perform the jest unit tests
* `cdk deploy`      deploy this stack to your default AWS account/region
* `cdk diff`        compare deployed stack with current state
* `cdk synth`       emits the synthesized CloudFormation template

* [Using CDK to perform continuous deployments in multi-region Kubernetes environments](https://aws.amazon.com/blogs/containers/using-cdk-to-perform-continuous-deployments-in-multi-region-kubernetes-environments/)

```bash
#
# This works with arn:aws:iam::177340731096:user/jimxu7 in both us-east-1 and us-west-2
#
ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
AWS_PRIMARY_REGION=us-east-1
AWS_SECONDARY_REGION=us-west-2
#
#
#
git clone https://github.com/aws-samples/containers-blog-maelstrom
cd containers-blog-maelstrom/aws-cdk-eks-multi-region-skeleton/
npm install
#
#
#
cdk bootstrap aws://$ACCOUNT_ID/$AWS_PRIMARY_REGION
cdk bootstrap aws://$ACCOUNT_ID/$AWS_SECONDARY_REGION
#
#
#
cdk list
cdk diff
#
#
#
cdk deploy "*"
#
#
#
mkdir ../../sample-app
cp -R ../aws-cdk-multi-region-sample-app ../../sample-app
cd ../../sample-app/aws-cdk-multi-region-sample-app
export EKS_CDK_CODECOMMIT_REPO=$(aws codecommit list-repositories --region $AWS_PRIMARY_REGION --query "repositories[?starts_with(repositoryName,'hello-py')].repositoryName" --output text)
export EKS_CDK_CODECOMMIT_REPO_URL=$(aws codecommit get-repository --region $AWS_PRIMARY_REGION  --repository-name $EKS_CDK_CODECOMMIT_REPO --query 'repositoryMetadata.cloneUrlHttp' --output text)
git init
git remote add codecommit $EKS_CDK_CODECOMMIT_REPO_URL
#
git add .
git commit -am "1.0.0 initial commit"
git push codecommit master
#
# Set kubectl current context to the Amazon EKS cluster in the primary region:
#
export CLUSTER1_KUBECONFIG_COMMAND=$(aws cloudformation describe-stacks \
  --stack-name "ClusterStack-$AWS_PRIMARY_REGION" \
  --region $AWS_PRIMARY_REGION \
  --query 'Stacks[0].Outputs[?starts_with(OutputKey,`demoeksclusterConfig`)].OutputValue' \
  --output text)
$(echo $CLUSTER1_KUBECONFIG_COMMAND)
#
kubectl get svc
#
curl $(kubectl get service hello-py -o jsonpath='{.status.loadBalancer.ingress[*].hostname}') && echo ""
#
# Set kubectl current context to the Amazon EKS cluster in the secondary region:
#
export CLUSTER2_KUBECONFIG_COMMAND=$(aws cloudformation describe-stacks \
  --stack-name "ClusterStack-$AWS_SECONDARY_REGION" \
  --region $AWS_SECONDARY_REGION \
  --query 'Stacks[0].Outputs[?starts_with(OutputKey,`demoeksclusterConfig`)].OutputValue' \
  --output text)
$(echo $CLUSTER2_KUBECONFIG_COMMAND)
#
kubectl get svc
#
# This CDK EKS project was successfully deployed and tested.
#
#
# Cleanup
#
$(echo $CLUSTER1_KUBECONFIG_COMMAND)
kubectl delete svc hello-py
#
$(echo $CLUSTER2_KUBECONFIG_COMMAND)
kubectl delete svc hello-py
#
cd ../../containers-blog-maelstrom/aws-cdk-eks-multi-region-skeleton
AWS_ECR_REPO=$(aws ecr describe-repositories --query "repositories[].[repositoryName]" --region $AWS_PRIMARY_REGION | grep 'cicdstack-ecrforhellopy' | sed -e 's/^[[:space:]]*//' | sed -e 's/^"//' -e 's/"$//')
AWS_ECR_IMAGES_TO_DELETE=$(aws ecr list-images --region $AWS_PRIMARY_REGION --repository-name $AWS_ECR_REPO  --query 'imageIds[*]' --output json )
aws ecr batch-delete-image --region $AWS_PRIMARY_REGION --repository-name $AWS_ECR_REPO --image-ids "$AWS_ECR_IMAGES_TO_DELETE" || true
#
cdk destroy "*" 
#
#
#
```
