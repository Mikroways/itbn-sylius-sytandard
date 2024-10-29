# Power on/off EC2 Instance using tags

This example uses lambda functions to power on and off the instances with the
tag "PowerOnOff = true".


### Installing dependencies
This repository uses NodeJs 16.x. Once you are using said version, install the NodeJs dependencies by running:

```
npm install
```

And to install the Pyhton dependencies run:
```
npm run sls:requirements:install
```

## Deployment

First, you need to configure your AWS credentials. After that, run:
```
npm run sls:deploy -- --stage <stage>
```
The serverless deploy command deploys your entire service via CloudFormation.

## Local development

We are using the [serverless-offline](https://github.com/dherault/serverless-offline) plugin for local development. This Serverless plugin emulates AWS λ and API Gateway on your local machine. To start the local server run:
```
npm run sls:offline -- --stage <stage>
```

This project uses the [.env.local](.env.local) file to configure the local environment.
