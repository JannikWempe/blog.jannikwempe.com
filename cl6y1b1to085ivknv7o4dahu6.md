---
title: "AWS Beginner: AWS is different. What are AWS Accounts, IAM Users and Root User?"
seoTitle: "Differences Between AWS Account, IAM User, and Root User"
datePublished: Wed Aug 17 2022 19:56:25 GMT+0000 (Coordinated Universal Time)
cuid: cl6y1b1to085ivknv7o4dahu6
slug: aws-accounts-iam-users-root-user
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1660766052768/9vKwygz5X.png
tags: cloud, aws, cloud-computing, beginners, amazon-web-services

---

Things are confusing when you are just starting to use AWS. You don't just create an account and get started as you do with other services. You come across terms like "Account", "User" and "Root User" and maybe ask yourself how they differ. This post should help understand these terms.

## When You Sign Up for an AWS Account

If you [sign up for an AWS account](https://portal.aws.amazon.com/billing/signup) you will be asked to enter a "Root user email address" and an "AWS account name":

![AWS sign up screen](https://cdn.hashnode.com/res/hashnode/image/upload/v1660759462081/pqv6ArCLL.png align="center")

After you finish the signup process, you will end up with an account (identified by a 12-digit account ID and an account name, also known as account alias) and some credentials (email and password of your root user) to enter that account.

This is what the [AWS docs say about the root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html) in a red warning callout:

> We strongly recommend that you do **not use the root user for your everyday tasks**, even the administrative ones. Instead, adhere to the best practice of using the root user only to **create your first IAM user**.

Okay, so you should create an IAM user (often just referred to as user; this is [how you can create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)) but what is the difference between an IAM user and the root user? And usually, if you sign up for a service you create an account with credentials and end up with one thing: an account. Why do you have (at least) three account-related things in AWS? How are they different? Before we will get to the definitions we will have a look at how to sign in using either the root user or an IAM user.

## Sign In to your account

This is how the sign-in screen looks like when you click on the "Sign In" button on [https://aws.amazon.com/console/](https://aws.amazon.com/console/):

![AWS sign in screen](https://cdn.hashnode.com/res/hashnode/image/upload/v1660763912747/hqdSRaXss.png align="center")

You can either go ahead and sign in as a root user by just providing the root user email address and the password or you can sign in as an IAM user by providing the account ID or account alias first and the IAM user name and password in the second step:

![AWS sign in screen IAM user](https://cdn.hashnode.com/res/hashnode/image/upload/v1660763779171/uhl7ljSG5.png align="center")

Note how you don't have to provide the account ID for the root user as that user has to be unique globally.

Also, you can use your account name in a URL like [https://myaccountname.signin.aws.amazon.com/console](https://myaccountname.signin.aws.amazon.com/console) to populate the account id input with `myaccountname` (that account alias must exist, otherwise you will land on a 404 page).

## Definitions

Now let's get to the definitions.

### Account

Think of an account as just a container. A container containing your resources, users and account settings.

You can create multiple of these containers. In fact, it is best practice to separate your different workloads (for example test and production) into different accounts.

### Root User

The root user is the master of your account. It is the central key (this is why you should activate Multi-Factor Authentication (MFA) for it) and provides unrestricted access to the account. There is always exactly one root user per account.

There are some [tasks that only a root user can do](https://docs.aws.amazon.com/accounts/latest/reference/root-user-tasks.html), e.g. changing the account settings (including the account name) and billing information.

Again: use MFA for your root user and only use that user if you have to, don't use it for everyday work.

### IAM User

An IAM User is an entity representing a person or an application (it is not necessarily a person, this can be confusing because the word "user" implies a person). You create the IAM users after the initial creation of your root user. An account can have several users (think of people or applications being able to access the container of resources). 

There are various ways how you can grant IAM users permissions in order to be able to do something. One of them is attaching policies to the IAM user ([learn more about IAM permissions](permissions for an IAM user)). An IAM user with the `AdministratorAccess` policy attached is not the same as the root user.

## Conclusion
The concept of accounts and users in AWS is different from what we are used to (Google, Spotify, etc.). This can lead to confusion. Hopefully this post has cleared up that confusion for you.

If you need help getting going, feel free to ask questions.