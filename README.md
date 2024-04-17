# Pulumi AWS S3 Tutorial

## Install Pulumi
```
# Windows
choco install pulumi

# Linux
curl -fsSL https://get.pulumi.com | sh
```

## Setup AWS Credentials
```
export AWS_ACCESS_KEY_ID="<YOUR_ACCESS_KEY_ID>"
export AWS_SECRET_ACCESS_KEY="<YOUR_SECRET_ACCESS_KEY>"

export AWS_PROFILE="<YOUR_PROFILE_NAME>"
```

## Create a Project
```
mkdir quickstart && cd quickstart
pulumi new aws-typescript
```

## Deploy Stack
```
pulumi up

pulumi stack output bucketName
```

## Modify 
```
echo '<html>
    <body>
        <h1>Hello, Pulumi!</h1>
    </body>
</html>' > index.html
```

Update index.ts to include bucketObject
```
// Create an S3 Bucket object
const bucketObject = new aws.s3.BucketObject("index.html", {
    bucket: bucket.id,
    source: new pulumi.asset.FileAsset("./index.html")
});
```

## Deploy Changes
```
pulumi up

aws s3 ls $(pulumi stack output bucketName)
```

## Modify to Serve as a Website
Modify index.ts as follows
```
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

// Create an AWS resource (S3 Bucket)
const bucket = new aws.s3.Bucket("my-bucket", {
    website: {
        indexDocument: "index.html",
    },
});

const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
    bucket: bucket.id,
    rule: {
        objectOwnership: "ObjectWriter"
    }
});

const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
    bucket: bucket.id,
    blockPublicAcls: false,
});

const bucketObject = new aws.s3.BucketObject("index.html", {
    bucket: bucket.id,
    source: new pulumi.asset.FileAsset("./index.html"),
    contentType: "text/html",
    acl: "public-read",
}, { dependsOn: [publicAccessBlock,ownershipControls] });

// Export the name of the bucket
export const bucketName = bucket.id;

export const bucketEndpoint = pulumi.interpolate`http://${bucket.websiteEndpoint}`;

```

## Deploy Changes
```
pulumi up

curl $(pulumi stack output bucketEndpoint)
```

## Destroy Stack
```
pulumi destroy
```