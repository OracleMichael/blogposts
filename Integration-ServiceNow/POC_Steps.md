# How to construct the POC

## Requirements

- OIC instance
- A Virtual Cloud Network (VCN)
- Access to OCI Events and OCI Functions (requires policies)
- Access to OCI Cloud Shell
- Access to a compartment
- ServiceNow developer account with admin access
- Ability to create a private/public PEM key pair

## Steps in brief

1. Create a VCN (Networking > Virtual Cloud Networks)
2. Create an OCI application (Developer Services > Functions)
3. Create an OCI repository (Developer Services > Container Registry)
4. Create an OCI function within the application that uses Python (Developer Services > Functions > [your app])
5. Upload code to OCI function using OCI Cloud Shell
6. Create an OCI event
7. Create PEM key files (locally) and an OCI API key (User Settings under your Profile)
8. Upload our integration file to integration instance
9. Add configuration variables to the function
10. Update the connection credentials for ServiceNow connection and REST
11. Modify the ServiceNow table cmdb_ci_vm_instance to have two extra variables "Image Details" and "Shape"
12. Test

## Things to keep track of

- Make sure everything is created in the same region
- Your OCI tenancy OCID
- Your OCI user OCID
- Your ServiceNow developer account credentials

## Steps in detail

_For most of these steps, you will need to log in to your OCI console. To do so, navigate to [cloud.oracle.com](cloud.oracle.com) and click **View Accounts** in the upper right corner, then **Sign in to Cloud**. On the next page, provide your **tenancy**, or **cloud account name**, and click **Next**. Finally, sign in using either SSO or local user login._

Before you continue, **make sure you are using the same region for the entire guide**. The default region depends on where you are in the world. For most North American users, the default region will be set to "US East (Ashburn)". You can change this region if you desire, but remember the region in which you are using to create all the resources to avoid running into cross-regional issues.

### Step 1. Create a VCN

_In this step, you will create a default VCN for use in steps 2 and 11. If you already know how to create a VCN or have one to use with an associated subnet, you may skip this step and move to step 2._

Navigate to **Networking > Virtual Cloud Networks** on your OCI console. On the left, select the compartment in which you wish to create the VCN. Then, click **Start VCN Wizard**. In the dialog box that appears, keep "VCN with Internet Connectivity" selected, then click **Start VCN Wizard**.

**Keep track of this compartment, you will use it in the future. Namely, keep track of the compartment name and compartment ID.** To find the compartment ID, you can search for your compartment name and view the OCID, which looks like this regular expression `ocid1\.compartment\.oc1\.\.[a-z0-9]{60}`.

In the wizard, give the VCN a name (for instance, "ExperianVCN") and click **Next**. Review the resources being created (most notably a public and private subnet), then click **Create**.

### Step 2. Create an OCI application

_In this step, you will configure an application to host an OCI function. If you are unable to complete this step, please contact your OCI admin to add you to a group with relevant policies (see [this link](https://docs.cloud.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm))._

Navigate to **Developer Services > Functions** on your OCI console. On the left, select the compartment in which the VCN resides, then click **Create Application**. On the dialog box that appears, give your application a name. Select your VCN on the VCN dropdown list, and then select one of the two subnets that appear. If you are unsure of which one to pick, select the public subnet. Then click **Create**.

### Step 3. Create an OCI repository

_In this step, you will create a repository that will host the code in which the OCI function will sit. If you are unable to complete this step, please contact your OCI admin to add you to a group with relevant policies (see [this link](https://docs.cloud.oracle.com/en-us/iaas/Content/Registry/Tasks/registrycreatingarepository.htm))._

Navigate to **Developer Services > Container Registry** on your OCI console. Click **Create Repository**. Give your repository a name, and select **public**, then create the repository.

### Step 4. Create an OCI function

_In this step, you will create a function that will capture an OCI event and call your integration with it. If you are unable to complete this step, please contact your OCI admin to add you to a group with relevant policies (see [this link](https://docs.cloud.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm))._

Navigate to **Developer Services > Functions > [your app]**. If you do not see your application, make sure you are in the correct compartment and region. Once in your application, on the left select the **Getting Started** option under the list of Resources if it has not been selected already. Follow the **Getting Started** steps up to step 7; skip steps 8-11 unless you are curious to see how a function is created and deployed.<!--Follow the **Cloud Shell Setup** instructions here, replacing the default variables (e.g. "Hello World") with pertinent ones as you decide. For your convenience those instructions are pasted here:-->
<!--
#### 4.1: Launch Cloud Shell

There is an icon for this that looks like `>_` between the region dropdown menu and the announcements icon (looks like a bell) at the top dark blue nav bar. **You can also launch this via the "Launch Cloud Shell" button in step 1 in the "Getting Started" section.**

#### 4.2: Apply context variables

**You can find these commands in steps 2-4 in the "Getting Started" section.**

Here, you will apply the following functions once cloud shell has loaded (it looks like a console on the webpage at the bottom of the page):
```
fn list context
fn use context us-phoenix-1
fn update context oracle.compartment-id ocid1\.compartment\.oc1\.\.[a-z0-9]{60}
fn update context registry [REGISTRY-ID]/[TENANCY-NAME]/[OCIR-REPO]
```
- Command 1 lists the contexts
- For command 2, you should replace `us-phoenix-1` with whichever region you are using. The contexts listed should include `default` and the region you are using.
- For command 3, replace the `ocid1\.compartment\.oc1\.\.[a-z0-9]{60}` regular expression with your compartment ID.
- For command 4, replace the last string `[REGISTRY-ID]/[TENANCY-NAME]/[OCIR-REPO]` with the one given by the steps, which might look something like `phx.ocir.io/mytenancy/Experian` for tenancy `mytenancy` and OCI repo `Experian`.

#### 4.3: Generate an Auth Token

You may find instructions on doing so [here](https://console.us-ashburn-1.oraclecloud.com/identity/users/ocid1.user.oc1..aaaaaaaa4pukeggfkqfo6vrhhtet6datuplgphjz7yrju4jebpivaedmxkkq/swift-credentials).

**It is imperative that after you create your auth token, you copy the auth token itself. It will not be shown again.** This auth token will be used in step 4.4, and also step 9.

#### 4.4: Log in to the registry using the Auth Token as your password

**You can find this command in step 6 in the "Getting Started" section.**

Run this command, replacing the last strings as needed:
```
docker login -u 'mytenancy/oracleidentitycloudservice/my.user@oracle.com' phx.ocir.io
```
for tenancy `mytenancy`, username `oracleidentitycloudservice/my.user@oracle.com`, and registry ID `phx.ocir.io`. You can find your username by clicking the profile icon in the upper right corner, then **User Settings**. Your full username will be displayed at the top; usually it is `oracleidentitycloudservice/` followed by the email you used to configure your user.

Once run, you will be prompted for your password. You may paste the auth token you received from step 4.3.

#### 4.5: (OPTIONAL) Create, Deploy, and Invoke your function

You may optionally run these commands to see how a function operates.
```
fn init --runtime java hello-java
cd hello-java
fn -v deploy --app [YOUR_APP_NAME]
fn invoke [YOUR_APP_NAME] hello-java
```

Here, you initialize a classic Hello World program written in Java (you may also choose Python). Deploying the app requires being in the same directory as the code, so you need to enter the `hello-java` folder. Once inside, you deploy the code to your application; this may take a couple of minutes as the compiler runs checks and pushes the Docker image to your app. Finally, you invoke the application, revealing the amazing message "Hello World", truly indicative of the overwhelming success these four lines of text accomplished.
-->
### Step 5. Upload code to OCI function using OCI Cloud Shell

_In this step, you will add code to the function. You may view the full comprehensive guide on creating a function [here](https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_functions_cloudshell_quickview/functions_quickview_top/functions_quickview/index.html#) to supplement any knowledge gaps this guide may have left out. If you are unable to complete this step, you know what to do (see [this link](https://docs.cloud.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsuploading.htm))._

If you just completed step 4, you are at the right place; otherwise, navigate to **Developer Services > Functions > [your app] > [your function]**. Open Cloud Shell. After the Cloud Shell loads, run this command to initialize your function: `fn init --runtime python [FUNCTION_NAME]`. (Remember to replace `[FUNCTION_NAME]` with an actual function name.) Once you have done this, enter the folder created for this function by running `cd [FUNCTION_NAME]`.

Finally, open the file in an editor of your choice. My choice is vim, so you would run `vim func.py`. Once inside, replace all of the code with the code provided to you. You can delete everything in a vim file by navigating the cursor to the very top and pressing `dG` (d, then shift+G while holding d). Then, enter `INSERT` mode, entered by typing one of (`a`,`A`,`i`,`I`), and then paste the code using cmd+V or ctrl+V. To exit `INSERT` mode (and re-enter `COMMAND` mode), press the escape key, and finally to save and quit from the file, type `:wq` while in `COMMAND` mode.

Here is the code below:

```
import io
import json
import requests
from fdk import response
 
def call_oic(event_json, oicbaseurl, oicusername, oicuserpwd):
    print(">>>> In call_oic")
    auth = (oicusername,oicuserpwd)
    headers = {"Content-Type": "application/json"}
    try:
        r = requests.post(oicbaseurl,auth=auth, headers=headers, data=event_json)
    except (Exception) as error:
        print('ERROR: In calling OIC', error, flush=True)
        raise
    print(">>>> Exiting call_oic")
    return 'STATUS: SUCCESS' if r.status_code == 202 else 'STATUS: ERROR - ' + str(r.status_code)
 
 
def handler(ctx, data: io.BytesIO=None):
    print(">>>> In handler")
    try:
        cfg = ctx.Config()
        oicbaseurl = cfg["oic_base_url"]
        oicusername = cfg["oic_username"]
        oicuserpwd = cfg["oic_userpwd"]
    except Exception:
        print('Missing function parameter(s)', flush=True)
        raise
    try:
        event_py = json.loads(data.getvalue())
        event_json = json.dumps(event_py)
    except (Exception) as ex:
        print('ERROR: Bad event payload', ex, flush=True)
        raise
    result = call_oic(event_json, oicbaseurl, oicusername, oicuserpwd)
    print(">>>> Exiting handler")
    return response.Response(
        ctx,
        response_data=json.dumps(result),
        headers={"Content-Type": "application/json"}
    )
```

So what's going on in the code? There are two methods: `handler` and `call_oic`. `handler` orchestrates the function, listening for an event payload and subsequently calling `call_oic` with the configuration parameters (you will configure these in step 8). `call_oic` forwards this payload to OIC; it discards any response from OIC. At a high level, this function serves as the connection between the OCI events service and the integration, which handles mapping this data to ServiceNow.

There are a few more changes to make. In `requirements.txt`, you must add `requests` as one of the requirements, as that is a package that is used by the python script not natively provided by OCI functions. Your requirements should contain `fdk` and `requests`. Once again, using vim, you can add the requisite requirement to `requirements.txt` as required, but alternatively since you are pasting a single line to `requirements.txt`, which should only contain `fdk` initially, you can instead just run this code: `echo "requests" >> requirements.txt`.

Finally, you will push the code. Make sure you are in the correct directory by running `pwd`; it should show something like `/home/[USERNAME_STRING]/[FUNCTION_NAME]` for your specific username and function name. If not, run `cd ~/[FUNCTION_NAME]`. Then, run this command: `fn -v deploy --app [YOUR_APP_NAME]`, once again, replacing the bracketed variable. As before, the command will take up to a couple of minutes to process.

### Step 6. Create an OCI event

_In this step, you will set up an event that captures the compute instance creation event and calls the function you configured._

Navigate to **Application Integration > Events Service** on your OCI console. On the left, select the compartment in which the function resides, then click **Create Rule**. On the dialog box that appears, give your rule a name and description. For the **Rule Conditions**, there will be one condition. The **Condition** should be "Event Type", the **Service Name** should be "Compute", and the **Event Type** should be "Instance - Launch End". You may click "Validate Rule" to look at the payload being transferred to the function, which is also the same exact payload sent to the integration (click **Close** to return to the rule configuration). For the **Actions**, there will be one action. The **Action Type** should be "Functions", **Function Compartment** should be the same compartment in which the function resides, and the **Function Application** and **Function** should be those you created above. Once these steps are completed, click **Create Rule**.

### Step 7. Create PEM key files (locally) and OCI API key (User Settings under your Profile)

_In this step, you will use a local bash shell to create a PEM key. Mac/Linux users will have an in-built terminal that allows them to execute shell commands, whereas Windows users will most likely need to download a shell such as Git Bash. Either way, you will be able to use Cloud Shell to create a PEM key. Once you have created the PEM key, you will use it to generate an OCI API key. The API key is used by Oracle applications to make authenticated calls to various OCI APIs, the most common being the OCI REST API._

Open up the shell of your choice. Once open, run these commands:
```
openssl genrsa -out test.pem -aes128 2048
chmod go-rwx test.pem
openssl rsa -pubout -in test.pem -out test_pub.pem
```
The first command creates a PEM key called `test.pem` that requires a passphrase to read (encrypted as aes128). The second command changes permissions on that key so that only you can read/write from it. Finally the last command creates a public key called `test_pub.pem` that will require you to enter the passphrase you previously set for the private key.

If you have created your keys on Cloud Shell, make sure to copy them to your computer for safekeeping. To do so, run `cat test.pem` and `cat test_pub.pem` so that the file contents are in the shell, allowing you to highlight and copy. NOTE: if you are using Windows or Linux, you will probably have to right-click on the highlighted text to copy, as ctrl+C in a bash shell is the interrupt command, not the copy command. On Mac, cmd+C works as intended. Finally, make sure you save the files as pem files (that is, the extension is pem and not pem.txt). For more specific details, visit [this link](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm).

Once you have created a set of PEM keys, you will now use them to create an OCI API key. In your Oracle Cloud home page, click on the profile icon (in the upper right hand corner), then click **User Settings**. On the left, there is a list of Resources; click **API Keys** if it is not selected already. Click **Add Public Key**, and then either upload or paste the **public key**. Once this is completed, the API key is ready to use, and its fingerprint (which is also the MD5 fingerprint of the PEM key) is its unique identifier. Save this fingerprint somewhere, as you will need it for step 9.

### Step 8. Upload our integration file to integration instance

_In this step, you will upload the integration file to your instance._

First, you will need to navigate to your integration instance. This usually comes in the form `https://[OIC_INSTANCE_NAME]-[TENANCY_NAME].integration.ocp.oraclecloud.com`. So if your integration instance is called "oictest1" and your tenancy is called "companyx" then the full URL would be `https://oictest1-companyx.integration.ocp.oraclecloud.com`.

Once logged in, click the hamburger menu in the upper left corner, then navigate to **Integrations > Integrations**. You will see an **Import** button in blue in the upper right area; click this button. When the window pops up, click **Choose File** and select the IAR file provided to you. This file contains the integration metadata and relevant connections. When imported, the IAR file will give you two connections and an integration. You will configure the connection properties for the integration in step 9.

### Step 9. Add configuration variables to the function

_In this step, you will add configuration variables to your function. These variables are ones you may have had prior to this step, but by the time you finish the previous step you will have proven that the values required for this step will work as configured._

Navigate to **Developer Services > Functions > [your app] > [your function]**. On the left, there should be two **Resources** options: Metrics and Configuration. Click **Configuration**. Here, you will be able to add configuration variables to this function. Add precisely these URLs in the table below:

<table>
	<tr>
		<th>Key</th>
		<th>Value Example</th>
	</tr>
	<tr>
		<td>oic_base_url</td>
		<td>https://[OIC_INSTANCE_NAME]-[TENANCY_NAME].integration.ocp.oraclecloud.com:443/ic/api/integration/v1/flows/rest/[INTEGRATION_NAME]/[VERSION]/[ENDPOINT_URI]</td>
	</tr>
	<tr>
		<td>oic_username</td>
		<td>[YOUR_USERNAME]</td>
	</tr>
	<tr>
		<td>oic_userpwd</td>
		<td>[YOUR_PASSWORD]</td>
	</tr>
</table>

- `oic_base_url`: As an example: if the name of the integration is `EXPERIAN_SERVICENOW`, so if your integration instance is named `OIC_INSTANCE` inside the tenancy `mytenancy`, then it is likely that the URL will be `https://oic_instance-mytenancy.integration.ocp.oraclecloud.com:443/ic/api/integration/v1/flows/rest/EXPERIAN_SERVICENOW/1.0/start`. You can extract this from the OIC instance after you complete step 7.
- `oic_username` and `oic_userpwd`: This is your username. This configuration is for demo purposes only so that you can see this working. There are more secure ways to configure authentication from OCI events to ping the integration using OCI vaults...but that's for another day.

### Step 10. Update the connection credentials for ServiceNow connection and REST

_In this step, you will modify the credentials for the connections that came in the IAR file. The connections are part of the integration that processes compute instance creation events set up in steps 4-5. Adding credentials allows the integration to access OCI resources and your ServiceNow instance so that it can collate VM resources to send to ServiceNow._

Navigate to your OIC instance. In the upper left hamburger menu, navigate to **Integrations > Connections**. Provided you did not import an integration or create more connections, both the **ServiceNow** and **REST** connections should be at the top of the connection list. They are called "Experian_SN" and "Compute_REST_API" respectively.

Click on "Experian_SN" to edit the connection. There are three items to edit: the **ServiceNow Instance Name**, **Username**, and **Password**. The **ServiceNow Instance Name** follows this pattern: `https://[SERVICENOW_INSTANCE_NAME].service-now.com/`. The default ServiceNow instance name follows the regular expression `dev\d{5}`, for instance `dev12345`. The **Username** and **Password** are those that you use to log in to this ServiceNow instance. The default instance username is probably "admin". Once you have filled in the relevant information, click the **Test** button in the upper right area. If the test does not complete successfully, double-check that the credentials you have can log you into that ServiceNow instance, then re-enter and retry. If you still run into issues please contact your Oracle Cloud representative. Otherwise, once the test completes successfully (you will see a 100% and green banner on the page), click **Save** and then the left chevron in the upper left area to return to the connections page.

Click on "Compute_REST_API" to edit the connection. You will edit the **Connection URL** and the security credentials which consist of five items: **Tenancy OCID**, **User OCID**, **Private Key**, **Fingerprint of API Key**, and **Passphrase**. The **Connection URL** may change from `https://iaas.us-phoenix-1.oraclecloud.com/` depending on what region you intend to capture compute instance creation events. For instance, if you wanted to service the ashburn region, you would instead use `https://iaas.us-ashburn-1.oraclecloud.com/`.
- You can find the **Tenancy OCID** by navigating back to your Oracle Cloud home page. In the upper left hamburger menu, scroll all the way to the bottom and select **Administration > Tenancy Details**. Click **Show** or **Copy** to get the **Tenancy OCID**.
- You can find your **User OCID** by navigating back to your Oracle Cloud home page. In the upper right corner, click on the profile icon, then click **User Settings**. Click **Show** or **Copy** to get your **User OCID**.
- Your **Private Key** should be uploaded as a file.
- The **Fingerprint** of your API key was configured in step 7. In case you don't have it handy, you can find it again by navigating back to your Oracle Cloud home page, then going to your **User Settings**, and finally selecting **API Keys** on the left. You **must** use the fingerprint that corresponds to the private key you just uploaded! Otherwise the authentication will fail.
- If your Private Key file has a **passphrase**, provide that here. Otherwise leave this field blank.

Once again, you must test and save your connection, making any changes as necessary.

### Step 11. Modify the ServiceNow table cmdb_ci_vm_instance to have two extra variables "Image Details" and "Shape"

<!-- MAY NEED SOME ADDITIONAL CONFIG ON THE SN SIDE TO GET IT TO WORK WITH OIC -->
<!-- Link: https://docs.oracle.com/en/cloud/paas/integration-cloud/servicenow-adapter/prerequisites-creating-connection.html -->

Log in to your developer instance with an admin account. On the left, search for "Table", then select the entry that just says "Tables". Search for these two tables: cmdb_ci_vm_instance, and cmdb_ci_linux_server. For both of these, add two new columns called **Image Details** and **Shape**. Both of these should be strings, and other settings can be kept as default.

Finally, you can modify the table view so that when you go to view details for a specific VM instance or linux server (via https://[YOUR_SERVICENOW_INSTANCE].service-now.com/cmdb_ci_server_list or https://[YOUR_SERVICENOW_INSTANCE].service-now.com/cmdb_ci_vm_instance_list, then selecting a record) you can see these two new variables. Also it would be a good idea to filter out some variables not used by the integration to avoid clutter.

### Step 12. Test

Testing is relatively simple. All you have to do is create a compute instance. The exact settings are not important.

The flow of the solution you have just created follows these steps:
- A compute instance is created.
- An OCI event is triggered on launch end
- The OCI event you configured captures this event and calls the OCI function you created
- The OCI function triggers the integration and passes the compute instance information over
- The integration creates a new record in the cmdb_ci_vm_instance and cmdb_ci_linux_server tables in ServiceNow

To verify that the process was successful, you can check a couple of locations:
- OCI function logs
- Track integration instances

The OCI function logs require setup in order for logs to actually be recorded, and the logs more or less only collect error messages. That is, on a successful call there should be no logs.

When an integration is triggered, the instance of such an integration run pops up on the **Track Instances** page. Here, you can view the progress of the integration while it is running, or view the path that the integration took after it has completed. If the entire integration completed without any errors, then the entire process was successful. Otherwise, you may have to contact your Oracle representative to help debug any failed runs, as there are multiple causes of integration failure. The most common errors are authentication-related, and these can be resolved by re-testing the connections, and also using Postman to make equivalent REST calls with the same authentication mechanism. For this guide specifically, you may consult https://www.ateam-oracle.com/oracle-cloud-infrastructure-oci-rest-call-walkthrough-with-curl to make REST calls to the Oracle IaaS API, for instance to [get information about all shapes](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/Shape/ListShapes) that Oracle offers (complete list [here](https://docs.cloud.oracle.com/en-us/iaas/api/#/en/iaas/20160918/))

# That's all!

This guide was prepared by michael.j.chen@oracle.com for Experian to help with their certification process of multiple cloud vendors in November 2020.
