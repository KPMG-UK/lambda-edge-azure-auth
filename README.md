# lambda-edge-azure-auth

[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg?style=flat-square)](https://github.com/semantic-release/semantic-release)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg?style=flat-square)](http://commitizen.github.io/cz-cli/)

[Microsoft Azure AD][azure] authentication for [CloudFront] using [Lambda@Edge].

>This project is based on [Widen/cloudfront-auth][cloudfront-auth], but has diverged in the following ways:
>
> * Stripped down to focus on Microsoft Azure __Authentication__ and __Authorization__ only.
> * Webpack config added to bundle the handler and dependencies in to a single file.
> * A __zip__ of the bundled lambda (sans `config.json`) [is released][releases] via a GitHub Action for use in downstream [IaC] projects like [terraform-aws-lambda-edge-azure-auth].
> * Simple-Url handling for default `index.html` and trailing slash redirects.
> * Downstream terraform module for deployment ([terraform-aws-lambda-edge-azure-auth]).

## Description

Upon successful authentication, a cookie (named `TOKEN`) with the value of a signed JWT is set
and the user redirected back to the originally requested path. Upon each request, Lambda@Edge
checks the JWT for validity (signature, expiration date, audience and matching hosted domain) and
will redirect the user to configured provider's login when their session has timed out.

## Usage

If your CloudFront distribution is pointed at a S3 bucket, [configure origin access
identity][oai] so S3 objects can be stored with private permissions. (Origin access identity
requires the S3 ACL owner be the account owner. Use our
[s3-object-owner-monitor](https://github.com/Widen/s3-object-owner-monitor) Lambda function if
writing objects across multiple accounts.)

Enable SSL/HTTPS on your CloudFront distribution; AWS Certificate Manager can be used to
provision a no-cost certificate.

Session duration is defined as the number of hours that the JWT is valid for. After session
expiration, cloudfront-auth will redirect the user to the configured provider to re-authenticate.
RSA keys are used to sign and validate the JWT. If the files `id_rsa` and `id_rsa.pub` do not
exist they will be automatically generated by the build. To disable all issued JWTs upload a new
ZIP using the Lambda Console after deleting the `id_rsa` and `id_rsa.pub` files (a new key will
be automatically generated).

## Microsoft Azure Guide

1. Clone or download this repo
1. In your Azure portal, go to Azure Active Directory and select **App registrations**
    1. Create a new application registration with an application type of **Web app / api**
    1. Once created, go to your application `Settings -> Certificates & Secrets` and make a new **client secret** with your desired duration. Click save and copy the value. This will be your `client_secret`
    1. Click on **Overview**, go to `Redirect URIs` and enter your Cloudfront hostname with your
    preferred path value for the authorization callback.
        >Example: `https://my-cloudfront-site.example.com/_callback`
1. Execute `./build.sh` in the downloaded directory. NPM will run to download dependencies and a RSA key will be generated.
1. Choose `Microsoft` as the authorization method and enter the values for [Tenant][azure-tenant], Client ID (**Application ID**), Client Secret (**previously created key**), Redirect URI and Session Duration
1. Select the preferred authentication method
    1. Azure AD Membership (default)
    1. JSON Username Lookup
        1. Enter your JSON Username Lookup URL (example below) that consists of a single JSON array of usernames to search through
1. Upload the resulting `zip` file found in your distribution folder using the AWS Lambda console and jump to the [configuration step](#configure-lambda-and-cloudfront)

## Configure Lambda and CloudFront

[Manual Deployment](https://github.com/Widen/cloudfront-auth/wiki/Manual-Deployment) __*or*__ [AWS SAM Deployment](https://github.com/Widen/cloudfront-auth/wiki/AWS-SAM-Deployment)

## Testing

Detailed instructions on testing your function can be found [in the Wiki](https://github.com/Widen/cloudfront-auth/wiki/Debug-&-Test).

## Build Requirements

* [npm] ^5.6.0
* [node] ^10.0
* [openssl]

## Contributing

See [CONTRIBUTING.md](Contributing.md).

[azure]:https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code
[azure-tenant]:https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-howto-tenant
[cloudfront]:https://aws.amazon.com/cloudfront/
[cloudfront-auth]:https://github.com/Widen/cloudfront-auth
[IaC]:https://en.wikipedia.org/wiki/Infrastructure_as_code
[lambda@edge]:https://aws.amazon.com/lambda/edge/
[node]:https://nodejs.org/en/
[npm]:https://www.npmjs.com/
[oai]:http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html#private-content-creating-oai-console
[openssl]:https://www.openssl.org
[terraform-aws-lambda-edge-azure-auth]:https://registry.terraform.io/modules/nickshine/lambda-edge-azure-auth/aws/
[releases]:https://github.com/nickshine/lambda-edge-azure-auth/releases
