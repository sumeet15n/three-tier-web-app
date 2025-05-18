# Build a Three-Tier Web App

## Introducing this Project!

In this project, I set up a three-tier web app.

### Services Used

- S3
- CloudFront
- Lambda
- API Gateway
- DynamoDB
- IAM

### Concepts Learnt

- Setting up a CloudFront distribution
- Origin access control (OAC)
- S3 bucket policies
- Creating and deploying a Lambda function
- Understanding and writing Lambda function code
- Creating and deploying APIs
- Setting up API resources
- Setting up API methods
- Creating a DynamoDB table and adding table items
- Creating a Lambda function, implementing the logic, and writing a test
- Granting Lambda the access to read from DynamoDB
- Integrating the presentation, logic, and data tiers
- Troubleshooting errors from the browser's developer tool
- Cross-origin resource sharing (CORS)

### Architecture Diagram

![Image](https://github.com/sumeet15n/three-tier-web-app/blob/master/Screenshots/SS0.png)

---

## Project Guide

### Step 1: Set up the Presentation Tier

Please refer to [my earlier project](https://github.com/sumeet15n/website-delivery-with-CloudFront) for this step.

### Step 2: Set up the Logic Tier

Please refer to [my earlier project](https://github.com/sumeet15n/Lambda-and-API-Gateway) for this step.

### Step 3: Set up the Data Tier

Please refer to [my earlier project](https://github.com/sumeet15n/fetch-from-DynamoDB-with-Lambda) for this step.

### Step 4: Integrate the Tiers

The logic and data tiers were integrated, which I verified by appending '/users?userId=1' to my API's invoke URL and obtaining the output as shown in the below screenshot.

![Image](https://github.com/sumeet15n/three-tier-web-app/blob/master/Screenshots/SS1.png)

I then integrated the presentation and logic tiers as described below.

I navigated to my CloudFront distribution and visited the URL provided under 'Distribution domain name'. I entered '1' in the userID field and clicked on 'Get User Data', but no data was returned. This is because the presentation and logic tiers were not connected.

I opened my browser's developer tool and navigated to the 'Console' tab, where I noticed an error as shown in the below screenshot. 

![Image](https://github.com/sumeet15n/three-tier-web-app/blob/master/Screenshots/SS2.png)

Upon expanding the details, I noticed that the URL 'https://[YOUR-PROD-API-URL]/users?userId=1' was being referenced, which was on line number 9 of the script.js file.

Next, I opened my script.js file and replaced the placeholder '[YOUR-PROD-API-URL]' with my actual prod stage API's invoke URL (I have reverted this now in my repo file for security).

After updating the script.js file, I reuploaded it to my S3 bucket.

### Step 5: Validate the Fully Functioning Web App

I revisited my CloudFront distributed site, entered '1' in the userID field, and clicked on 'Get User Data', but no data was returned again.

I opened my browser's developer tool and navigated to the 'Console' tab again, where I noticed that the error this time was because of CORS as my API Gateway was not configured to allow requests from my CloudFront distributed site.

Note: CORS (cross-origin resource sharing) decides whether a frontend (like a CloudFront-distributed site) is allowed to talk to a backend (like an API Gateway). By default, browsers do not let one website access resources from another domain unless the backend explicitly says it's okay. In this case, because CORS was not configured in my API Gateway, the browser blocked the request from CloudFront, causing the CORS error.

To resolve this error, I navigated to my 'users' API resource and enabled CORS. While enabling CORS, I selected both GET and OPTIONS under Access-Control-Allow-Methods and entered my CloudFront distribution domain name as the Access-Control-Allow-Origin. After enabling CORS, for the changes to take effect, I deployed my API again by selecting prod as the stage.

Since I had enabled Lambda Proxy Integration in Step 2, my API Gateway simply forwards requests to my Lambda function and expects it to return the full HTTP response including CORS headers. Therefore, I updated my Lambda function code as shown below to include the 'Access-Control-Allow-Origin' header in the HTTP response. I ensured that the region is correct and that * was replaced with my CloudFront domain name. Once done, I deployed my updated Lambda function.

```
// Import individual components from the DynamoDB client package
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand } from "@aws-sdk/lib-dynamodb";

const ddbClient = new DynamoDBClient({ region: 'ap-south-1' }); // Replace with your region
const ddb = DynamoDBDocumentClient.from(ddbClient);

async function handler(event) {
    const userId = event.queryStringParameters.userId;
    const params = {
        TableName: 'UserData',
        Key: { userId }
    };

    try {
        const command = new GetCommand(params);
        const { Item } = await ddb.send(command);

        if (Item) {
            return {
                statusCode: 200,
                headers: {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*' // Replace '*' with specific domain in production
                },
                body: JSON.stringify(Item)
            };
        } else {
            return {
                statusCode: 404,
                headers: {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*'
                },
                body: JSON.stringify({ message: "No user data found" })
            };
        }
    } catch (err) {
        console.error("Unable to retrieve data:", err);
        return {
            statusCode: 500,
            headers: {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            body: JSON.stringify({ message: "Failed to retrieve user data" })
        };
    }
}

export { handler };
```
After deploying my updated Lambda function, I refreshed my CloudFront-distributed site, entered 1 in userID field, and clicked on Get User Data. I noticed that the user data was successfully fetched.

A screenshot of the successfully working three-tier web app is shown below.

![Image](https://github.com/sumeet15n/three-tier-web-app/blob/master/Screenshots/SS3.png)

### Step 6: Delete Resources

Finally, I deleted all the created resources after completing this project to avoid any further charges on AWS.

---

## Project Reflection

This project took me approximately 3.5 hours to complete. The most challenging part was to ensure that the presentation and logic layers are connected and the logic and data layers are connected, and the most rewarding part was to see the three-tier app successfully.

Big thanks to NextWork (https://www.nextwork.org/) for this project! I highly recommend this platform to anyone who wants to learn DevOps concepts and complete more projects like this one. Happy learning!