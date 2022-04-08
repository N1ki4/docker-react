### To replace Travis with Github Actions you can do the following: 

1. Delete your .travis.yml file from the local project.

2. Navigate to your Github repository.

3. Click Settings

4. Click Secrets

5. Click New repository secret

6. Create key/value pair secrets for AWS_ACCESS_KEY, AWS_SECRET_KEY, DOCKER_USERNAME, DOCKER_PASSWORD.

7. In your local development environment, create a .github directory at the root of your project

8. Create a workflows directory inside the new .github directory

9. In the workflows directory create a deploy.yaml file which should contain the following code (name does not matter):

Important - remember to change your application_name, environment_name, existing_bucket_name and region to the values used by your AWS EBS environment:



name: Deploy Frontend
on:
  push:
    branches:
      - main
 
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
      - run: docker build -t cygnetops/react-test -f Dockerfile.dev .
      - run: docker run -e CI=true cygnetops/react-test npm test
 
      - name: Generate deployment package
        run: zip -r deploy.zip . -x '*.git*'
 
      - name: Deploy to EB
        uses: einaregilsson/beanstalk-deploy@v18
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
          aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
          application_name: docker-gh
          environment_name: Dockergh-env
          existing_bucket_name: elasticbeanstalk-us-east-1-923445559289
          region: us-east-1
          version_label: ${{ github.sha }}
          deployment_package: deploy.zip
 


10. Run the typical git add, commit and push commands

11. Click Actions in the Github repository dashboard to view each step of the workflow.

Note - This code is using a well-supported marketplace action, more info can be found here:

https://github.com/einaregilsson/beanstalk-deploy

### ----------------------------------------------------------------------------
### Required Updates for Amazon Linux 2 Platform - DO NOT SKIP
updated 8-12-2021

When creating our Elastic Beanstalk environment in the next lecture, we need to select Docker running on 64bit Amazon Linux 2 and make a few changes to our project:


This new AWS platform will conflict with the project we have built since it will look for a docker.compose.yml file to build from by default instead of a Dockerfile.

To resolve this, please do the following:

1. Rename the development Compose config file

Rename the docker-compose.yml file to docker-compose-dev.yml. Going forward you will need to pass a flag to specify which compose file you want to build and run from:
docker-compose -f docker-compose-dev.yml up
docker-compose -f docker-compose-dev.yml up --build
docker-compose -f docker-compose-dev.yml down

2. Create a production Compose config file

Create a docker-compose.yml file in the root of the project and paste the following:

version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '80:80'
AWS EBS will see a file named docker-compose.yml and use it to build the single container application.