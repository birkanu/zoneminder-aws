# Running [Zoneminder](https://zoneminder.com/) on AWS

This project uses the AWS CDK to deploy a fully functioning Zoneminder installation on AWS.
Some bits are particular to my use case, but most of it can be tweaked and re-used by anyone wanting to run Zoneminder on AWS.

## AWS Account prerequisites

You'll need a couple of things from your AWS account before you can get started.

1.  Pick an AWS region where things will run (for me, `us-east-1`)
2.  An access key, secret key, and AWS account number (I highly recommend following the [security best practices](https://aws.amazon.com/blogs/security/getting-started-follow-security-best-practices-as-you-configure-your-aws-resources/)).
3.  An ssh keypair, in case you need to ssh into the EC2 instance for troubleshooting
4.  The public IP address where your security cameras live

Include those variables in a .env (see [.env-sample](./.env-sample)) file,
and source it `. .env` to set the environment variables in your shell.

Next, you'll need some things that I, personally, have set up in a [separate github project](https://github.com/matthewtgilbride/aws-infrastructure).

1.  A domain name and hosted zone
2.  A wildcard certificate e.g. `*.yourdomain.com`

Create two values in AWS Systems Manager Parameter store called `domainName` and `certificateArn`, respectively, and populate them with these values.

This project will look those values up, and setup zoneminder to run at `https://zoneminder.yourdomain.com/zm`

## Create the zoneminder stack

Run the following at the root of the project:

`yarn`

`yarn deploy`

You should end up with a stack created, and be able to access Zoneminder at `https://zoneminder.yourdomain.com/zm`
    
## Automated user, settings, monitor, and zone setup
    
I have a utility script that automates the process of configuration, monitor, and zone setup for me.  
It will first prompt you to manually update the user's password with the value from secrets manager.  
Then it will upload a python script that can back up video files to the created S3 bucket, update some configuration, and create two monitors and two zones.  

**Before you run it**:

*   Note that the config files it uses are quite specific to my setup, but you can use them as inspiration.
*   The script it runs is [after-user-setup.ts](scripts/after-user-setup.ts), and it reads values from [zm_reference_data/cameras](./zm_reference_data/cameras)
*   LOOK AT THESE FILES FIRST.

To run it: `yarn post:deploy`
    
## Manual Zoneminder settings

The script outlined above puts the python file `zm-s3-upload.js` into the `/usr/bin` directory of the EC2 instance.
You can use this to back up videos to S3 by setting up a filter in Zoneminder.

`<your-host>/zm/index.php??view=filter`

Create a new filter with the following values checked:

*   Execute command on all matches
*   In the text box for the command: `. /etc/profile.d/domain_name.sh && /usr/bin/zm-s3-upload.js '%ET%' '%MN%' '%ED%'`
*   Run filter in background

Save the filter.
