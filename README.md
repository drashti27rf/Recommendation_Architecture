# Recommendation_Architecture

Data and ML flow diagram shows us the how the data will process and flow through model which will later deploy on server and serve recommendation to user based on request received, like if user comes with cookie id, it’ll serve recommendation using Collaborative filtering algorithm, if he’s new user it will come with location id and device id and then it’ll serve recommendation using Generic recommendations and if user is on specific product page or cart page, then it’ll serve recommendation using Computer Vision Algorithm. While user is surfing it’ll send live data to db which will process and train on model again and serve recommendation dynamically as user continue his journey and it’ll also save users logs, all activities and served recommendation in one db. Now we’re going to develop architecture on AWS which is public cloud and deploy it on it.
1) Data preparation: Here data will get clean, label and pre-process and will be ready for model to ingest. We’ll have 4 db where 3 will store records of each model and one will store all the logs activity and recommendation served to user.
2) Training/Tuning: Here all the models will get train, then parameter will be tuned as per result and the best model will be saved and then will be ready for deployment.
3) Deployment: Here our model will get deploy on server and will take input from user and serve recommendation to user based on request received. Also we’ll keep check performance here using some metrics which will be discussed later.
4) User: Here our user will interact with UI which will send some request to model and receive recommendation in return. User live data and logs will also be send to get recommendation as user continues his journey.

Basically our recommendation system processed the data and provide recommendation to user based on
request, and also it’ll send data to which will be used for recommendation as journey continues. At first we have
3 AWS DyanmoDB which is NoSQL database and can scale more than 10 trillion request per day, where each of
them will have pre-process data of user, image, location and product, which will be connected with AWS Glue.
AWS Glue is a fully managed, pay-as-you-go, extract, transform, and load (ETL) service that automates the timeconsuming
steps of data preparation for analytics. We’ll create AWS Glue ETL jobs from DynamoDB to connect
with service like S3. Each dataset consists of one or multiple CSV files, which can be uniquely identified by an
Amazon S3 prefix.
AWS Glue Crawlers scan the data in the data sources, extract the schema information from it, and store the
metadata as tables in the AWS Glue Data Catalog. This is the preferred approach for defining tables. The
crawlers use AWS Glue Connections to connect to the data sources. Each connection contains the properties
that are required to connect to a particular data store. The connections will be also used later by the ETL jobs to
fetch the data from the data sources.
Now, once we have data in S3 we’ll use AWS Lambda which is basically an event-driven, serverless computing platform provided by Amazon as a part of Amazon Web Services. It is a computing service that runs code in response to events and automatically manages the computing resources required by that code. AWS lambda will get trigger and notify Amazon SageMaker which is basically used for training and deployment of ML model. Now Amazon Sagemaker will start training all the three models using respective data from S3 bucket.
Now when our model is ready we’ll Docker Image for our training container, then we’ll push the Docker Image Elastic Container Registry, then we’ll finally deploy our model using Sagemaker as Inference Endpoint. Inference Endpoint is connected to AWS Load Balancing which automatically distributes incoming application traffic across multiple Amazon EC2 instances. EC2 Instance is a web service that provides secure, resizable compute capacity in the cloud.
Now our application is ready and when user comes to the home-page it interact with API Gateway which is connected to EC2 Instance, and if he’s old user he’ll send cookie id which is basically User ID to Inference Endpoint and when will get recommendation using Collaborative Filtering Model. If he’s new user he’ll send location ID and device ID to endpoint and in return will get recommendation using Generic Recommendation. Now when user goes to any specific product page, it’ll send product id and Image to endpoint and in return will get recommendation using Computer Vision Algorithm. This was our whole recommendation lifecycle.
Now when user gets in touch with UI it’ll also send live data, logs, activities and recommendation to S3 bucket, from there live data will pass through and trigger Lambda function which will notify Sagemaker to start pre-processing on data and then it’ll send to respective database. So new entry gets recorded in DB. As soon as any new entry comes in DyanamoDB our AWS EventBridge will record new event and it’ll notify Lambda which will trigger Sagemaker to start training again and serve recommendation accordingly. Amazon EventBridge is a serverless event bus that makes it easier to build event-driven applications at scale using events generated from your applications, integrated Software-as-a-Service (SaaS) applications, and AWS services. This is how user will get dynamic recommendation throughout his journey.
Performance: For performance tracking we have used AWS CloudWatch, which collects monitoring and operational data in the form of logs, metrics, and events, and visualizes it using automated dashboards so you can get a unified view of your AWS resources, applications, and services that run in AWS and on-premises. We have setup CloudWatch alarm that sends an Amazon Simple Notification Service (Amazon SNS) message when a particular metric goes beyond a specified threshold for a specified number of period or when our EC2 instance gets scale up.
Scalability: AWS Load Balancing has been used to make scalable application. We have setup a system where if CPU utilization goes above 50% we scale out to 100 EC2 Instances from 50 Instances.
Role of MLOps:
1) Data Scientist has made sure model is ready for deployment before giving to deployment team.
2) All the key phases like data collecting, pre-processing, training and deployment are automated in order to faster release.
3) Each Microservice is running separately, so failure of one service won’t affect other service or if main service breakdown, it’ll notify the admin.
Assumptions:
1) Initially we have data ready in form of NoSQL data for training the data in 3 databases.
2) Initially our application is running on 50 Server.
3) We have all trained model ready for deployment.
