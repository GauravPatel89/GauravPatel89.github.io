## How to Speed up AWS Lambda deployment on Serverless Framework by leveraging Lambda Layers.

Artificial Intelligence has emerged as a buzz word in the recent times. Most people involved in technology industry have been drawn toward this AI wave. For anyone starting out in the field Deep learning(DL)/ Machine learning(ML) natural starting point is wide variety popular courses available on the internet eg. Andrew NG's [deeplearning.ai](https://www.coursera.org/specializations/deep-learning) course or Jeremy Howard's [fast.ai](https://course.fast.ai) course. A common theme among these courses is they start with basics of Neural networks and deep learning and go upto defining custom models and their training on frameworks like [PyTorch]() or [TensorFLow](). When we look at it from production point of view this is just half the picture. Equally important task is to deploy our model and make it accesible so that someone can use it to calculate, measure or see something. 

Recently I came across this beautiful course [Extensive Vision AI Program](https://theschoolof.ai/#programs) by [The School of AI](https://theschoolof.ai/). This course not only covers basics of Deep Learning but after every session students are suppose do an assignment. These are assignments are not just model creation and training but also its deployment on AWS Lambda and making it accessible through simple static website deployed on AWS S3. 

As part of coursework we were supposed to our deep learning models to AWS Lambda. We did this using [Serverless](https://www.serverless.com/) framework. This blog explains how we speeded up this deployment process by leveraging AWS Lambda Layers. 

### AWS Lambda and Important Limits

[AWS Lambda](https://aws.amazon.com/lambda/) is one of the most famous serverless service available today. It offers low cost and simple platform for deployment of solutions written in different languages like Python, Node.js, Java, C# and many more. For getting started with AWS Lambda have a look at this [link](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html).

Although very useful and low cost there are some important limits while deploying on Lambda that one needs to keep in mind.

- The deployment package has size limit of 262144000 bytes or 262 MB(official page mentions 250MB). So the unzipped size of your package — including the layers — cannot be greater than this number.

- /tmp provided for running Lambda functions has size limit of 512 MB

- Function memory (RAM) has limit of 3008 MB

- Max execution time is limited to 900 sec or 15 mins

For complete list of limits see this [link](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html). 

### Serverless framework for AWS Lambda Deployment  

There are number of options for function deployment on AWS Lambda. It can be done using AWS Lambda [GUI interface](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html), [AWS Serverless Application Model(SAM)](https://lumigo.io/aws-serverless-ecosystem/aws-serverless-application-model/),[Serverless](https://www.serverless.com/), [Terraform](https://lumigo.io/aws-lambda-deployment/aws-lambda-terraform/) and many more. Check this [list](https://lumigo.io/aws-lambda-deployment/) for more information.

We used Serverless framework for Lambda Deployment. Basic procedure 








