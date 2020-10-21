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

We used Serverless framework for Lambda Deployment. Serverless framework installation and an example deployment of a PyTorch model is explained in detail in this [blog](https://towardsdatascience.com/scaling-machine-learning-from-zero-to-hero-d63796442526).

#### Procedure in Short

1. Create an empty serverless package.  
Create an empty serverless package with aws-python3 as a template. This will create a 'example-lambda-function-name' directory with configuration file 'serverless.yaml', and python file handler.py


        serverless create --template aws-python3 --path example-lambda-function-name
        
2. Install serverless-python-requirements plugin in your package.  
cd into your package directory and execute command given below to install serverless-python-requirements plugin in your package. This will add few more configuration files to your package.

        serverless plugin install -n serverless-python-requirements

3. Add requirements to requirements.txt  
Add a file named 'requirements.txt' in your package directory. In this file, list out python packages required to run your deployed function.

4. Edit serverless.yml  
Edit serverless.yml file for necessary AWS S3 configurations. Add configuration settings for 'serverless-python-requirements' to reduce deployment package size. Please refer this [blog](https://towardsdatascience.com/scaling-machine-learning-from-zero-to-hero-d63796442526) for detailed procedure.

5. Deploy package  
Deploy your package using following. 

        serverless deploy
      
On executing this command serverless will take a AWS Lambda compatible docker image, 'pip' install python packages mentioned in 'requirements.txt' and compress these into a package **.requirements.zip** then again compress this as part of another anothe zip file containing other necessary package files like handler.py. It also adds a file in your deployment package **unzip_requirements.py**. This file is executed at the beginning of your Lambda function to unzip the compressed python packages in **.requirements.zip** into user directory and adds its path into system path. Finally serverless will deploy your package onto AWS lambda and show its url. 

### The Problem

Although serverless framework made deployment task very easy for us but we faced one irritating issue.  Most of our assignment deployments were on PyTorch framework, so in all of our deployments we were using PyTorch and torchvision packages. Now with just these two packages (and their dependencies) as part of lambda deployment, the deployment package size was coming out to be ~140 MB. Irritating part was each time there is any change in any of the file and we redeployed the package, this entire ~140 MB artifact will get uploaded. This was time consuming and irritating but seemed solvable.

****** Include image with zip size 142MB *****

### The Solution
AWS Lambda has something called [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) for precisely this kind of scenarios wherein you can deploy common static dependencies as Lambda Layers and include upto 5 layers in your Lambda function. So when deploying a Lambda function if you link Lambda layers with it, whenever Lambda function is triggered contents of Lambda layers are copied into **/opt** directory of lambda function. This way you can keep static parts of your deployments (in our case PyTorch and torchvision) in lambda layer and  each time deploy just the application code. 

Lets try this!!!

#### Create a Lambda Layer for Static dependencies.

We already have **.requirements.zip** file prepared for us by serverless. This file contains all the necessary python packages required in our Lambda function.

*Note: You can generate just the deployment package in serverless using command **serverless package**. Package thus generated will be available under .serverless directory. **.requirements.zip** package containing python dependencies will be available in your package directory itself. (You may have to enable viewing hidden file to see these files under linux)*

Easiest way to create basic Lambda Layer is through GUI as shown below.

*****XXXXX  Include picture showing lambda layer creation Page***** 

Layer deployment through this requires us to upload layer contents as zip file. But as shown, for zip files greater than 10MB uploading through S3 is preferred. So first create a bucket on AWS S3 and upload .requirements.zip file

*****XXXXX  Include picture showing AWS S3 bucket and zip file***** 

 Now we can feed the AWS S3 url of uploaded file for Lambda Layer creation
 
 *****XXXXX  Include picture showing lambda layer creation***** 
 
 Select supported runtimes
 Click create.
 
 *****XXX Picture about failure to create lambda layer******
 
 **What??!! that failed**  
 
 It says package size should be less than **262144000 bytes** but our .requirements.zip was ~140MB. What happened??
 
 Catch here is when we create Lambda Layer, it unzips the zip file provided by us. So when we provided S3 url of .requirements.zip file, it downloaded and unzipped it and rightly so unzipped size is far greater than **262144000 bytes**.
 
 
 ******XXX Picture about actual size of .requirements.zip when unzipped***.
 
 Hey!! but then how come **serverless** is able to do it? For Lambda functions also we have size limit of **262144000 bytes**. 
 
 
 If we look carefully serverless smartly tackles this issue as shown below.
 
 *****XX graphics .requirements->package.zip->[Lambda(unzip package.zip->unzip .requirements.zip**
 
 It puts .requirement.zip under another deployment zip file and when lambda function is triggerred this **.requirements.zip** file is unzipped and its path is added to the system path. 
 
 
 So why not replicate Serverless's solution. 
 
 We already have python packages inside **.requirements.zip**. lets put this under another zip file named say **package.zip**. 
 
 Now try Lambda Layer creation through GUI.
 
 *******XXX Image about successful lambda layer creation********
 
 So our layer is now created. Not down the layer ARN.
 
 Now lets try and use this layer. 
 
 Create simple serverless package
 
               serverless create --template aws-python3 --path example-use-lambda-layer
               
Edit **serverless.yml**               
 
 *******XXXX edited serverless.yml image
 
 As shown above we have mentioned ARN of our deployed layer to be used with our example lambda function. 
 
 Now we know whenever our Lambda function is triggered **.requirements.zip** from our layer will be copied into /opt directory of lambda function. We must unzip it and add its path to the system path so that we can make use of these python packages. 
 
 Edit **handler.py**
 
 ****Image of handler.py with unzip code***
 
 to check whether PyTorch and torchvision packages are available we simply include following lines.
 
                import torch
                import torchvision
 








