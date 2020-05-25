# Server Deployment, Containerization and Testing

Here is a simple guid to success this project cotaining tips to avoid issues and fix them.

> Note if you are using windows os you should use git bash instead of cmd to make things more easier.

## 1. Run the API locally using the Flask server (no containerization)

The following steps describe how to run the Flask API locally with the standard Flask server, so that you can test endpoints before you containerize the app:

1- Install python dependencies from requirements.txt: 

```sh
 pip install -r requirements.txt
```

2- Set up the environment. You do not need to create an env_file to run locally but you do need the following two variables available in your terminal environment. The following environment variable is required:

**JWT_SECRET** - The secret used to make the JWT, for the purpose of this course the secret can be any string.

**LOG_LEVEL**(optional) - The level of logging. This will default to 'INFO', but when debugging an app locally, you may want to set it to 'DEBUG'. To add these to your terminal environment, run the 2 lines below.

```sh
 export JWT_SECRET='myjwtsecret'
 export LOG_LEVEL=DEBUG
```

> Ps You can create a bash file to run those both command if you want.
>
> First create a `setup.sh` whcih contains the 02 commands above.
>
> Then run it in the same folder you create by runing `. setup.sh`
>
>Check if the variable was really created by runing `echo $JWT_SECRET` you should see `myjwtsecret`if you did not meas the variable is not created.
>
> It is alwyas a good practive to check if your vars

3- Run the app using the Flask server, from the top directory, run:

```sh
python main.py
```

4- Try now the API 03 endpoints using **postamn** it will be more easier. You can use your own version or take my postman collection here and import it to you postamn. Modify the host var from the collection by clicking on edit go to variable tab and in the value of hos change my value to your whihc in this case `localhost:5000` or whatever is you host.
And check the 03 end points :).

### Concept Checklist

- [x]  I have run the project Flask API locally.
- [x] I have tested the endpoints of the running Flask application.

## 2. Containerizing and Running Locally

## 3. CD Pipeline

The next step of the project will be:

- Create an EKS cluster
- Set a secret using AWS parameter store
- Create a pipeline watching for commits to your Github repository
- Build and deploy your image using CodeBuild

### 3.1 Create an EKS Cluster and IAM Role

> We will start by creating IAM role first. We will create the clusters only once we need them to avoid extra billing.

#### 3.1.1 Set Up an IAM Role for the Cluster

The next steps are provided to quickly set up an IAM role for your cluster.

2. Create an IAM role that CodeBuild can use to interact with EKS

- Set an environment variable ACCOUNT_ID to the value of your AWS account id. You can do this with awscli:
  
```sh
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

- Create a role policy document that allows the actions "eks:Describe*" and "ssm:GetParameters". You can do this by setting an environment variable with the role policy:
  
```sh
 TRUST="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"
 ```

> Ps: Check if the var was created

```sh
 echo $TRUST
```

- Create a role named 'UdacityFlaskDeployCBKubectlRole' using the role policy document

```sh
EKS_DESCRIBE="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Action\": [ \"eks:Describe*\", \"ssm:GetParameters*\" ], \"Resource\": \"*\" } ] }"
```

> Check if EKS_DESCRIBE was really created by running this command `echo $EKS_DESCRIBE`

```sh
$ echo $EKS_DESCRIBE
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "eks:Describe*", "ssm:GetParameters*" ], "Resource": "*" } ] }
```

- Attach the policy to the 'UdacityFlaskDeployCBKubectlRole'

```sh
aws iam put-role-policy --role-name UdacityFlaskDeployCBKubectlRole --policy-name eks-describe --policy-document "$EKS_DESCRIBE"
```

=> You have now created a role named 'UdacityFlaskDeployCBKubectlRole'
