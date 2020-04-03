---
layout: post
title:  "Introduction to AWS Identity and Access Management"
date:   2020-03-25
tags:
  - aws
  - iam
---
As companies continue to move parts of their infrastructure to the cloud, the focus of many IT security teams is slowly shifting from traditional perimeter-based security controls (firewalls, intrusion detection systems, etc.) towards more user-based and data-centric security models. From a security perspective, it is critical for companies to understand the [shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/) agreed upon with the cloud provider. However, one thing is certain, independently of the cloud provider or service model you choose:
> Identity and Access Management will always be your responsibility.

Unfortunately, understanding how to properly create identities and configure access control policies in the cloud is not as easy as it sounds. This post is intended to be a simple introduction to the **AWS Identity and Access Management** service. We will cover the main concepts through the lens of two use cases. First, we'll look at how users can get access to AWS services. Then, we will see how AWS services interact with each other.

## Use Case 1: IAM User to AWS Service
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/aws-iam-user-to-service.png)

Here we have a worker (i.e. somebody that works for your organization) that needs to get interactive access to an AWS service such as S3 or EC2. In a nutshell, two things need to happen before that person can access the service:
1. The person needs to authenticate against AWS. This is done by creating an **IAM User** along with a set of long term credentials (e.g. username and password). At the time of **authentication**, AWS will prompt the person to provide these credentials to ensure the user is who she/he claims to be.
2. AWS needs to verify that the IAM User can, in fact, access the AWS service or resource. This process is called **authorization**. During authorization, AWS will evaluate a pre-configured set of **IAM policies** to ensure that the IAM User can access the service.

Let's build our use case in AWS to understand this two-step process in more detail.

### Authentication
From the AWS console, select the IAM service, and click on "Users" to create a new IAM user:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-user.png)

As you can see, there are two types of credentials to choose from:
- **Username and Password** for interactive logins such as when using a browser to access S3.
- **Access Key ID and Access Key** for programmatic logins. This is useful in situations when the entity trying to authenticate is not a human but some kind of process or application. For example, the AWS CLI uses access keys for authentication.

Once the user is created, you should be able to login into the AWS console:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/aws-console.png)

*The sign-in link can be found at the top of your IAM dashboard page and should look like this:
{% highlight shell %}
https://<YOUR_AWS_ACCOUNT_ID>.signin.aws.amazon.com/console
{% endhighlight %}

### Authorization
Now that we are logged in, let's see what happens when we try to access an AWS service such as S3:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/s3-no-access.png)

Access is being denied because our user has not been authorized to access anything yet.

We need to create an IAM policy to allow our user access to S3. Luckily, AWS comes with several pre-configured polices out of the box. These built-in policies are called **AWS Managed Policies**. It is always a good idea to check if an AWS managed policy already exists for your use case. You can create your own policies though, if none of the existing ones meet your needs. Custom policies are also called **Customer Managed Policies**.

You can see the list of AWS managed policies by clicking on "Policies". As you can see, there are a couple S3 policies available:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/s3-managed-policies.png)

At a high level, policies consist of four components:
- **Service**: the AWS service that is protected by the policy. S3 in this case.
- **Access Level**: the actions that the users will be able to perform for the given service. AWS actions can get really granular and are service specific.
- **Resource**: the resources on which the actions can be performed. For example, in the case of S3, a resource can be a whole bucket or just certain objects within the bucket.
- **Request condition**: conditions that must be met before the user can perform an action on the resource. Conditions can be really powerful but we will ignore them for now for the sake of simplicity.

For example, the "AmazonS3ReadOnlyAccess" gives users "List" and "Read" permissions on all S3 buckets registered within your AWS account:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/s3-policy-summary.png)

Let's assign this policy to our user Bob by going to his profile and clicking on "Add permissions" and selecting "Attach existing policies directly":

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/attach-policy-to-user.png)

Bob should now be able to see and access all S3 buckets:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/s3-user-access.png)

Simple, right? Even though the process is simple to understand, it is not so simple to put into practice. What if I have to grant access to hundreds or even thousands of users?

### Groups
You can bundle users into **IAM Groups** and attach policies to groups instead of attaching them directly to your users. Groups are intended to ease management and are also considered a security best practice.

Creating groups is a three step process:
1. Choose a name for your group such as "ReadOnlyUsers":
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-group-name.png)
2. Attach policies to your group:
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-group-permissions.png)
3. Add users to the group:
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/add-users-to-group.png)

Now, every time a user is added to our group they will automatically be granted read-only access to S3.

## Use Case 2: AWS Service to Service
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/aws-iam-service-to-service.png)

The second use case that we'll cover today involves an AWS resource, such as an EC2 instance, trying to access another AWS resource, such as S3 an bucket.

The above diagram is very similar to the one from the previous use case. The main differences being:
- AWS services authenticate using short-lived credentials (i.e. an access token that expires after a certain amount of time).
- There are no IAM users involved in this case. Instead, AWS services can assume **IAM Roles** once they've been properly authenticated.
- IAM Policies are attached to IAM Roles, and roles can then be assigned to resources.

Let's walk through the scenario to solidify these concepts.

I already have an EC2 instance running inside my account:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/ec2-instance.png)

We can create an IAM Role for our EC2 instance following these simple steps:
1. Create a service role for EC2:
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-role-type.png)
2. Attach policies to your role:
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-role-attach-policy.png)
3. Choose a name and a description for your role:
![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/create-role-name.png)

Go back to your EC2 instance, navigate to "Instance Settings > Attach/Replace IAM Role" and select the role we just created:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/attach-role.png)

You should now be able to SSH into your EC2 instance and use the [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/s3/) to list and download objects from your S3 buckets:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/ec2-terminal.png)

### Resource-based Policies
Up until this point, we've only been using **Identity-based Policies**. Identity-based policies are attached to IAM users, groups, or roles. There is another type of policy called **Resource-based Policies**, which, as the name implies, are attached to resources.

To create an S3 resource policy you need to use the S3 console, locate you bucket, and select "Permissions > Bucket Policy":

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/s3-resource-policy.png)

S3 does not yet have a GUI for editing policies so you have to be comfortable working with JSON. As an alternative, you can use the [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html).

This policy should grant "List" access to the bucket as well as "Get" access to any object contained within it.

It is worth pointing out that resource-based policies are always defined inline. **Inline policies** are embedded in an AWS resource (an S3 bucket in this case). You can also create Identity-based inline policies (i.e. embedded in an IAM user or role) but they are difficult to manage and therefore not recommended.

### Bonus Points: EC2 Instance Data Endpoint
Where did our EC2 instance get the temporary credentials from to be able to access our S3 bucket? The answer isn't obvious at first glance since the AWS CLI did all the "magic" for us behind the scenes.

The answer relies on something called EC2 Instance Data. The Instance Data is an HTTP endpoint that AWS exposes on every EC2 instance under the following URL:
{% highlight shell %}
http://instance-data/latest
{% endhighlight %}

The endpoint is only accessible from your EC2 instance and you can query it using a tool like Curl:

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/ec2-instance-data.png)

If you dig into the endpoint you'll find information about the IAM roles assumed by the EC2 instance as well as the temporary credentials that the instance will use when authenticating to other services:
{% highlight shell %}
http://instance-data/latest/meta-data/iam/security-credentials/<RoleName>
{% endhighlight %}

![image]({{ site.baseurl }}/assets/images/intro-to-aws-iam/ec2-instance-data-creds.png)

## Key Concepts
In this post we covered some basic AWS IAM concepts such as users, groups, roles and policies. We defined AWS managed and customer managed policies and saw the difference between identity-based and resource-based policies.

In future posts, we will explore more advanced IAM topics.

## Additional resources
[AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)

[AWS Policy generator](https://awspolicygen.s3.amazonaws.com/policygen.html)
