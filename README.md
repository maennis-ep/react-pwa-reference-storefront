# Elastic Path's reference storefront experience

Reference storefront demonstrating usage of Cortex in a modern UI progressive web application.


Table of contents
=================

<!--ts-->
   * [Development Quick-Start](#gh-md-toc)
      * [Pre-req](#pre-req)
      * [Configuration](#configuration)
      * [Sample Data](#sample-data)
      * [Setup (Development)](#setup-development)
      * [Setup (Production)](#setup-production)
   * [Linting](#linting)
   * [Offline Mode](#offline-mode)
   * [Localization](#localization)
   * [Unit Tests](#unit-tests)
   * [Jenkins Pipeline](#jenkins-pipeline)
   * [Contribution Guide](#contribution-guide)
   * [License](#license)
<!--te-->

## Development Quick-Start
Built with REACT.js, Bootstrap 4, Babel, and Webpack.

### Pre-req:
If you haven’t already, you’ll need to install the following software:
* Install [Git](https://git-scm.com/downloads)
* Install [Node.js](https://nodejs.org/en/download/)
* Install [Visual Studio Code](https://code.visualstudio.com/) from Microsoft and the following extensions:<br/>
    * [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome)<br/>
    * [ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)<br/>
* Install [Java JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html) for executing unit tests.
* Install [Maven 3.5.2](https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/) for executing unit tests.
* Install [IntelliJ IDEA](https://www.jetbrains.com/idea/) optionally for creating/running unit tests.

### Configuration
##### `ep.config.json (options)`

 - `cortexApi.path`, **required**, *string*:
URL consisting of hostname and port which the storefront will use to reach Cortex. A web proxy is configured in this project's Webpack config for your convenience. Set this value to `/cortex` for local development to redirect Cortex calls to the local proxy.
 - `cortexApi.scope`, **required**, *string*:
Name of store to retreive data from Cortex.
 - `cortexApi.pathForProxy`, **required**, *string*:
The path the webpack proxy will route storefront Cortex calls to. URL consisting of hostname and port to actual running instance of Cortex. Leave this value empty to disable the proxy.
 - `skuImagesS3Url`, **required**, *string*:
 Path to catalog images hosted on S3 bucket. Set this to the full URL of your S3 images, replacing the sku/file-name with the string `%sku%`. This value will be populated during pageload with values retreived by Cortex.
 - `enableOfflineMode`, **optional**, *bool*:
  Option to enable offline mode. When in offline mode, requests are fetched from static data instead of Cortex. Check out [how it works](#offline-mode)
 - `gaTrackingId`, **optional**, *string*:
 Google Analytics tracking ID to integrate with Google Analytics Suite and track enhanced ecommerce activity as it happens on your site.

### Sample Data:
This project includes a set of sample data for your convenience, and assumes you have a valid Elastic Path development environment.<br/>
1. Extract and copy the sample catalog data contents from this project `ep-store/data` into your `ep-commerce/extensions/database/ext-data/src/main/resources/data` directory.
2. Update `ep-commerce/extensions/database/ext-data/src/main/resources/data/liquibase-changelog.xml`.<br/>
Add the following sample data to the file. Note that it should be the only sample data included within the sample data blocks: `<include file="ep-blueprint-data/liquibase-changelog.xml" relativeToChangelogFile="true" />`
3. Update `ep-commerce/extensions/database/ext-data/src/main/resources/environments/local/data-population.properties`. Set the liquibase.contexts property to `default,ep-blueprint`.
4. In the command line, navigate to the `extensions/database` module.
5. To run the Data Population tool, run the following command: `mvn clean install -Preset-db`
6. For further information, refer to Elastic Path's developer documentation [Populate the Database](https://developers.elasticpath.com/commerce/7.3/Core-Commerce-Development/Setting-up-your-Developer-Environment/Populate-the-Database#ConfigureDemoDataSets)

### Setup (Development):
1. Clone/pull this repo to a directory of your choosing
2. Run `cd ep-store`
3. Run `npm install` to install dependencies
4. Configure `./src/ep.config.json` as required for your environment: [here](#configuration)
5. Run `npm start` to start the server in development node
6. Navigate to `http://localhost:8080/` to see the running PWA

### Setup (Production):
1. Clone/pull this repo to a directory of your choosing
2. Run `cd ep-store`
3. Run `docker build -t ep-store -f ./docker/prod/Dockerfile .`
4. Push `ep-store` image to your docker repo.

##### On a host computer:

5. Pull `ep-store` from your docker repo.
6. Copy provided `docker-compose.yaml` and `nginx.conf` files from `docker/prod/` in this repo to a folder on the remote host.
7. Replace `$CORTEX_URL` in `nginx.conf` with a URL of your cortex server as well as `$DOCKER_REPO` in `docker-compose.yaml` with `ep-store`.
8. Run `docker-compose up -d`

## Linting
This project has been set up with ESLint as our linting utility.<br/>
If you'd like to learn a bit more about ESLint check out their [documentation](https://eslint.org/)<br/>
We're currently extending Airbnbs’ ESLint configuration because at this time the Airbnb Code Style and the according ESLint configuration are very popular and well accepted by developers.<br/>
You can check out the style guide on [github](https://github.com/airbnb/javascript)<br/>
* ESLint loader has been added to `webpack.config.dev.js` so it runs when you start the app in development mode.
* To run the linter from your commandline first go to the project root directory and run the following command:
    * `./node_modules/.bin/eslint --ext .js --ext .jsx [file|dir|glob]`
        * Run on entire project: `./node_modules/.bin/eslint --ext .js --ext .jsx .`
* With the ESLint extension for Visual Studio Code you'll get feedback while you're writing the code under the `Problems` View

If you plan to check in your code, make sure to fix all your linting errors first!

## Offline Mode
You can enable offline Mode in [`./src/ep.config.json`](#configuration)<br/>

**How it works**<br/>
The *mock magic* is contained in `./src/utils/Mock.js`<br/>
The *mock data files* are stored in `./src/offlineData`<br/>
At a high level, **Mock.js** uses a map of Requests to Responses to send the mock data, given a Request. Instead of doing a fetch call to a url, it does a lookup in the map to retrieve/return the mock data. If mock data cannot be found for a request, an Error is thrown.<br/>

**How to add/edit data**<br/>
If you're looking to create/modify mock data:<br/>
* Start *Online* and perform the *flow* that you want to mock.<br/>
* In your browser using Dev Tools, view the Network tab for requests made by your *flow*. Filter your requests using the **XHRF** or **XHR and Fetch** filter.<br/>
* Copy the response directly from your request into a *.json* file under the `./src/offlineData` directory.<br/>
* In `.src/utils/Mock.js` add a variable for your data: `const myData = require('../offlineData/myData.json')`<br/>
* Finally add your data into the map: `mockData.set(myData.self.uri, { status: myStatusCode, data: myData}`<br/>
    * In the case of a **followlocation** you'll want to create a new variable for the request uri, and use that instead of `myData.self.uri`. This is because the responses include the *followed* url instead of the *request* url.<br/>
     * In the case of a request that doesn't have a response you can add the request url with a status code, and empty data. (Check out forms in Mock.js as an example)

**Verifying your data**<br/>
Now that you've mocked up the data for your *flow*, it's time to go offline and verify! Enable [Offline Mode](#configuration) and restart your server.<br/>
Go through your *flow* and verify everything works the same as Online. If there was something missed, there will be an error thrown in your *browser console* which will include the request it could not find the mock data for. Mock that request and you'll be able to continue!

**Out of the box flows**<br/>
Out of the box you get some mock data for the following flows, give them a shot!<br/>
* You can search for "water" in the search bar, and view the products returned.
* You can browse to "Womens" category and view the products from that category.
* Under the "Womens" category, you can add "Women's CR550 Polo" to cart. *Note:* Currently you can only add Medium Grey as the options. Try mocking up the other skus!
* You can check out the cart, there's one product already in there for viewing. *Note:* You won't be able to modify the cart, as it's response is static. `./src/offlineData/cartData.json`
* From Cart, *Proceed To Checkout*, currently you can only **Continue without an account**
* On the Checkout page there is mock data for Billing and Shipping Address as well as a payment method, which allows you to complete the order!
    * After completing the order you're brought to the Order Review Page, you can complete the purchase to see the Purchase Receipt Page
    * Purchase Receipt Page has all the information for your order displayed!

## Localization
This store supports multiple languages and currencies.
Any front-end textual data should be added to the `localization/en-CA.json` as a resource string.
Project includes two locales out of the box: `en-CA` and `fr-FR`.
For development purpose run: `node tools/translate.js` which will run a pseudo translation from `en-CA` locale to `fr-FR`. In order to add a new locale add a new entry to `supportedLocales` array in `ep.config.json` and add an appropriate json file to `localization` folder. In addition you will have to configure language and currency for all products in Commerce Manager.

## Unit Tests
Test data can be found in `tests`

**Running Tests:**<br/>
Run all Tests: `mvn clean install -Dcucumber.options="--tags @smoketest"`<br/>
Run Sanity Tests: `@sanity`<br/>

*Maven Options:*
* `-Dcucumber.options="--tags @smoketest"` - You can replace the tag with your own tag.
* `-Dfailsafe.fork.count="1"` - This is the number of parallel tests run at the same time. Default is 1 and can be changed to other values depending on number of TestsIT classes.
* `-Premote -Dremote.web.driver.url="<REMOTE DRIVER IP>"` - The `remote` parameter triggers tests to be executed using a remote VM. The `remote.web.driver.url` specifies the URL of the remote VM. e.g. `http://<IP_ADDRESS>:4444/wd/hub`
    * Note: You have to have selenium grid setup in order to use this feature. Please refer to official documentation on Selenium Grid.

**Running subset of tests:**
* You can run a subset of tests in IntelliJ by right clicking and running any one of TestsIT classes under `/selenium/src/test/java/com/elasticpath/cucumber/`
* You can create your own local runner class to run your own tagged tests. E.g. RunLocalTestsIT.java which runs your own tagged tests @local
    * Do not commit the local runner class and tags as they are only for your local testing purpose.

*Updating Browser Driver Versions*
* You can download the latest browser driver from web. e.g. chromedriver.
* Update the RepositoryMap.xml for the driver version.
* Hash value can be found locally if you run following in bash command locally.
```
openssl sha1 <filename>
```
* Example: https://github.com/Ardesco/Selenium-Maven-Template/blob/master/src/test/resources/RepositoryMap.xml

## Jenkins Pipeline
This project includes a Jenkinsfile for a Scripted Pipeline which builds a store docker image from this project, deploys it to AWS and then runs the Unit Tests from this project. For using this pipeline you'll need to create an EC2 instance for the pipeline to deploy your store + cortex

**Configuring the Pipeline**<br/>
Create a new Jenkins Pipeline and configure it with the following:
* Give your pipeline a name
* Set this project to be parameterised and include the following parameters:
  * `DOCKER_REGISTRY_ADDRESS` - *Path to AWS ECR* ie: `${awsAccountID}.dkr.ecr.${region}.amazonaws.com`
  * `CORTEX_NAMESPACE` - *The namespace for your cortex images (activemq, batch, integration, search, cortex)*
  * `DOCKER_IMAGE_TAG` - *The tag used for your cortex images*
  * `STORE_NAMESPACE` - *The namespace for your store image and db image*
  * `STORE_IMAGE_TAG` - *The tag for your store images (store + db)*
  * `STORE_NAME` - *The name of the store in your data* ie: `vestri`
  * `EC2_INSTANCE_HOST` - *The ip of your EC2 host for deploying this pipeline*
  * `EC2_INSTANCE_USER` - *The user of your EC2 instance*
  * `EC2_INSTANCE_SSH_KEY` - *The path in Jenkins node to your ec2.pem file*
  * `SOLR_HOME_GIT_URL` - *The git url to your project containing your SOLR HOME Config*
  * `SOLR_HOME_BRANCH` - *The branch name for your SOLR HOME Config*
  * `SOLR_HOME_PATH` - *The path in your git project to your SOLR HOME Config*
* Set Build Triggers to **Poll SCM** and choose a schedule
* Set Pipeline with the following:
  * **Definition** to *Pipeline script from SCM*
  * **SCM** to *Git*
  * **Repository URL** to your Repository for this project
  * **Branches to build** specifier to *\*/master*
  * **Script Path** to *Jenkinsfile*
* **Save** your configuration<br/>

You now have a pipeline that triggers on commits to master based on your chosen schedule. Check out the next section for more on the logic to the pipeline

**How it works**<br/>
The Stages:
* `SETUP` - The setup stage pulls from this project and the project containing SOLR HOME Config
* `BUILD` - The build stage builds the store docker from the `docker/dev/Dockerfile` in this project and then pushes it to AWS
  * The dev docker that is created uses an `entrypoint.sh` to replace the path to CORTEX and the STORE name in `ep.config.json` and then starts the project in dev mode using `npm install`
* `UNDEPLOY_EXISTING` - The undeploy stage will clean up the working directory if it exists and then remove any docker containers and images
* `DEPLOY` - The deploy stage starts by creating the working directory, copying over files from this project, and copying over the SOLR HOME Config. Then exporting environment variables used in the compose file and then deploys the store and cortex with docker-compose
* `TEST` - The test stage sets environment variables for JAVA_HOME, and adds JAVA + MAVEN to path. These are pulled from your Jenkins tools. This stage will then use a script from [intoli](https://intoli.com/blog/installing-google-chrome-on-centos) for installing google chrome to use for headless tests. The script downloads `google-chrome-stable` so we rename it to `google-chrome` so that chromedriver will find it for the tests. Before running the tests we need to replace the `selenium.session.baseurl` in the `pom.xml`. The tests are then run in a try-finally block so that the cucumber reports are added even if one fails. This uses the Jenkins Cucumber Plugin for viewing reports in Jenkins UI


## Contribution Guide
* Contributions may be performed by developers at their choosing. Changes to this project must be reviewed and approved by an owner of this repository <br/>
See [CONTRIBUTING.md](https://github.com/shaunmaharaj/ep-store/blob/master/CONTRIBUTING.md)<br/>

## License
[GPLv3 License](https://github.com/shaunmaharaj/ep-store/blob/master/LICENSE)
