Execution Steps
---------------
1) Clone repos under the same top-level folder:

Project: api-platform-samples 
-  Repo URL: git@github.com:apigee/api-platform-samples.git
-  Branch: **docs-functest-demo**
-  Check out to a sub-directory: **api-platform-samples**

Project: apitestframework 
-  Repo URL: git@revision.aeip.apigee.net:utils/apitestframework.git 
-  Branch: **hdas_TESTTOOLS-1417**
-  Check out to a sub-directory: **apitestframework**


2) Set the variable 'WORKSPACE' to the top-level folder. Many of the commands below rely on 'WORKSPACE' being set.

```export WORKSPACE=<your local working directory>```


3) Install pre-requisites: node, mvn, java.

  1. ```yum install -y nodejs``` <br />  
  1. ```npm -g install npm@latest  ``` <br /> 
  1. ```npm install -g n``` <br /> 

see:

- install node and npm:https://coolestguidesontheplanet.com/installing-node-js-osx-10-9-mavericks/
- install mvn: https://maven.apache.org/install.html

4) To add tests.json in api-platform-samples please follow the below instructions: 
- Add "**name**_tests.json" at every individual proxy folder level, where **name** is the proxy name. The test framework looks for tests with the pattern "*_tests.json" Eg: helloworld_tests.json 
- Retain the same json structure as in the apigee-healthcheck sample - see healthcheck_tests.json for examples of tests.
- Give a unique TestID in name_tests.json for each test. TestID can be of the form TC-**proxy**-**function**.  Eg: TC-helloworld-getIP

5) Cleanup output folder and other old data.
  1. ```rm -rvf ${WORKSPACE}/apitestframework/APITestFramework/test-output/ ```
  1. ```rm -vf ${WORKSPACE}/apitestframework/APITestFramework/src/resources/customers/apigee-docs/modules/proxysamples/data/docsTests.xlsx ```
  1. ```rm -vf ${WORKSPACE}/apitestframework/APITestFramework/src/resources/customers/apigee-docs/modules/proxysamples/data/docsTests.json ```

6) **This step is optional if you are running the tests aganst apigee-docs on https://edge-beta.e2e.apigee.net.** Change parameters under api-platform-samples/setup/setenv.sh to deploy all sample proxies. The tests are designed to run on the api-docs org on E2E at https://edge-beta.e2e.apigee.net. So make your settings below for acess to that org. These instruction use sed, but you canjust edit setenv.sh by hand. 
  1. ```cd ${WORKSPACE}/api-platform-samples/setup ```
  1. ```sed -i.bu "s/org=.*$/org=${Organization}/g" setenv.sh ```
  1. ``` sed -i.bu "s/username=.*$/username=${OrgAdminUsername}/g" setenv.sh ```
  1. ```sed -i.bu "s/password=.*$/password=${OrgAdminPassword}/g" setenv.sh ```
  1. ```sed -i.bu "s/url=.*$/url=${ManagementHost}/g" setenv.sh ```
  1. ```sed -i.bu "s/env=.*$/env=${proxyEnv}/g" setenv.sh ```
  1. ```sed -i.bu "s/api_domain=.*$/api_domain=${apiDomain}/g" setenv.sh ```
  1. ```sed -i.bu "s/provision=.*$/provision=${provisionApiProducts}/g" setenv.sh ```

7) **This step is optional if you are running the tests aganst apigee-docs on https://edge-beta.e2e.apigee.net.**  Deploy all the proxies to the Organization mentioned in setenv.sh. This should already be done so if you are running locally you can skip this step. But check apigee-docs on https://edge-beta.e2e.apigee.net to be sure the proxies are deployed.
  1. ```cd ${WORKSPACE}/api-platform-samples/setup ```
  1. ```sh deploy_all.sh ```

8) Parse api-platform-samples folder for all *tests.json and create a master.json and generateJson.sh, and copy the API ZIP files to the necessary location.  

  1. Create master.json. master.json contains all the tests for all the samples so it might be a large JSON file.
    - ```cd ${WORKSPACE}```
    - ```rm master.json; echo '[' > master.json; find api-platform-samples -name '*tests.json' -print0 | while IFS= read -r -d '' file; do sed '1d; $d' $file >> master.json; echo $file; echo ',' >> master.json; done; sed -i.bu '$d' master.json; echo ']' >> master.json```

  1. Find proxy zip files and copy them to the respources dir (pattern *.zip)
    - ```find api-platform-samples -name '*.zip' -print0 | while IFS= read -r -d '' file; do cp $file apitestframework/APITestFramework/src/resources/customers/apigee-docs/resources/bundles; echo $file; done```

  1. Find POST body files and copy them to resources. (pattern *_body.* Eg: developer_body.json; healthchk-apiproduct_body.xml). These files are used if a test is required to pass a request body.
    - ```find api-platform-samples -name '*_body.*' -print0 | while IFS= read -r -d '' file; do cp $file apitestframework/APITestFramework/src/resources/customers/apigee-docs/resources; echo $file >> generateJson.sh; done```

  1. This step is optional and not sure what it even does:
    - ```rm -f master.json.bu ```
    - ```cat generateJson.sh ```
    - ```sh generateJson.sh ```

9) Copy master.json to apitestframework:
- ```yes | cp -rvf master.json ${WORKSPACE}/apitestframework/APITestFramework/src/resources/customers/apigee-docs/modules/proxysamples/data/docsTests.json```

10) Execute json2xls from apitestframework to generate docTests.xlsx file. 

The tests are actually run based on the contents of docTests.xlsx. See the generated file at ${WORKSPACE}/apitestframework/APITestFramework/src/resources/customers/apigee-docs/modules/proxysamples/data/docsTests.xlsx.

You only have to do the install the first time you run this. 
  1. ```cd ${WORKSPACE}/apitestframework/APITestFramework/utils/json2xls ```
  1. ```npm install json2xls ```
  1. ```node index.js ```

11) Clean build APITestFramework to execute tests from the above generated .xlsx. I usually only run the mvn step once.
  1. ```cd ${WORKSPACE}/apitestframework/APITestFramework ```
  1. ```mvn clean compile assembly:single ```
  1. ```java -cp .:target/APITestFramework-4.2.5-jar-with-dependencies.jar:src/resources com.apigee.eng.api.test.Driver -config resources/customers/apigee-docs/properties/config.properties -env apigee-docs -module PROXYSAMPLES ```

12) **If running against apigee-docs on https://edge-beta.e2e.apigee.net skip this step. Otherwise you will undeploy all the proxies.** To UnDeploy all the proxies from the Org & Cleanup keys provisioned for Oauth. 
  1. ```cd ${WORKSPACE}/api-platform-samples/setup/provisioning ```
  1. ```sh cleanup.sh ```
