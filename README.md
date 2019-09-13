# Dojo static website hosting on S3

## Prerequisite

 - npm [[installation using nvm](https://github.com/nvm-sh/nvm#installation-and-update)]
 - serverless ([installation instructions](https://serverless.com/framework/docs/getting-started/))
 - aws cli ([installation instructions](https://docs.aws.amazon.com/fr_fr/cli/latest/userguide/cli-chap-install.html))


 ## Créer un projet serverless

 - Generate boilerplate

 ```
 serverless
 # chose python env
 # name your project
 ```

- Remove unused parts
    - delete file `handler.py`
    - remove all the lines in the `serverless.yml` except:

```yml
service: quentin-static-website # put a name of yours here

provider:
  name: aws
  runtime: python3.6 # or 2.7 it doesn't matter for now
```

 ## Créer a bucket with public read access

- chose a unique bucket name and export it as an environment variable:
`export STATIC_WEBSITE_BUCKET=<YOUR_NAME_HERE>-static-website-bucket-dojo`

 - in the `serverless.yml`, add the cloudformation template for a bucket s3:
Add the following lines at the end of the `serverless.yml`

```yml
resources: # Below this is a cloudformation template
  Resources: # Describes the AWS resources
    StaticSiteBucket:
      Type: AWS::S3::Bucket # Type of resource
      Properties: # Properties
        BucketName: ${env:STATIC_WEBSITE_BUCKET}
        AccessControl: PublicRead # Bucket ACL
        WebsiteConfiguration:
          IndexDocument: index.html # Entry point of the website
    Outputs: # Describes what to output once the stack is built
    WebsiteURL:
        Value: !GetAtt [StaticSiteBucket, WebsiteURL]  # Let's print the website url
        Description: URL for website hosted on S3
```

- Create the stack with `sls deploy -vv`


## Build a frontend app:

 - You can chose a example project [here](https://reactjs.org/community/examples.html)
 - clone it in your project directory (I chose bitcoin-price-index):

 `git clone --depth=1 --branch=master https://github.com/MarkFChavez/bitcoin-price-index.git && rm -rf bitcoin-price-index/.git`

 - install and build the app:

 ```shell
 cd bitcoin-price-index # Or the folder of the app you chose
 npm install
 npm run-script build
 ```


## Upload your static files on the bucket

- Use the aws-cli to upload the files by running the following command (REPLACE the name of your bucket):

 `aws s3 cp --recursive --acl public-read build s3://$STATIC_WEBSITE_BUCKET`

 (Note that each file you upload gets assign a `public-read` access control: without this, you'll get a 403 Forbidden :())


Browse your website:
- You can get your website URL with:
`echo http://$STATIC_WEBSITE_BUCKET.s3-website-$(sls info | grep region |awk '{print $NF}').amazonaws.com/ `

 ## Delete your website:

- Empty the bucket:
 `aws s3 rm --recursive s3://$STATIC_WEBSITE_BUCKET`

- Delete the stack and the bucket:
 `sls remove`
