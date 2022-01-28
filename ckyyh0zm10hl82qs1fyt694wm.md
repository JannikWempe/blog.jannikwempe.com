## The Power of AWS CDK Aspects

## What are CDK Aspects?

CDK Aspects are a powerful tool provided by AWS CDK. With great power comes great responsibility. In this article, we’ll talk about what CDK Aspects are, and some of its use cases.

This is the very first sentence in the [CDK Developer Guide about Aspects](https://docs.aws.amazon.com/cdk/latest/guide/aspects.html):

> CDK Aspects are a way to **apply an operation to every construct in a given scope**. 

CDK Aspects can be applied to everything that implements `IConstruct` (for example `App`, `Stack` and `Construct`). CDK Aspects use the **Visitor Pattern**. Here is the definition for the Visitor Pattern from the Design Patterns Book (aka Gang of Four Book):

> Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.
 
It's similar to the quote from the CDK Aspects Guide, but there's one important additional aspect (*pun intended*): Apply operations **without changing the classes**.

This is how the `IAspect` interface looks like:

```typescript
interface IAspect {
   visit(node: IConstruct): void;
}
```

*Notice the method name: visit. That name should make sense by now.*

This is how you apply Aspect to an `IConstruct` (`App` in this case):

```typescript
const app = new cdk.App();

Aspects.of(app).add(new TagChecker());
```

This implementation gives you access to every single `IConstruct` in the scope of the `app`. It includes all `CfnResource`s (i.e. the actual CFN Resource), which implements the `visit` method (in class `TagChecker` in this case). You can do whatever you want with it. We'll have a look at some use cases in the next section.

One additional piece of information is quite important: Aspects are **applied on the early phase** of a CDK app, which is even before synthesizing the CloudFormation template:

![CDK app lifecycles](https://cdn.hashnode.com/res/hashnode/image/upload/v1642622909709/M0HvdhMxq.png)
(Source: [CDK Developer Guide about Aspects](https://docs.aws.amazon.com/cdk/latest/guide/aspects.html))

Therefore, you do not have to wait for deployment, it provides fast feedback.

## Use Cases for CDK Aspects

Okay, now that we know what CDK Aspects are, let's have a look into some of its use cases.

### Assist With Compliance

Probably the most common use case is to help create code that meets compliance rules. Do you have to add certain tags to (some) resources? You can write an Aspect that helps you remember that. You must enable versioning for S3 buckets? You can write an Aspect for that. Do you have to exclusively use encrypted databases? I think you know the answer.

Whether you throw an error, or silently change the config for non-compliant resources is up to you. But, keep in mind that it can be very frustrating for devs if you overdo it with creating Aspects. This is because it may cause errors all the time or (probably worse) they have a hard time debugging due to configs are silently changed.

Since compliance rules are relevant for multiple applications in an organization, they are most likely shared via an (internal) package manager.

Check out [cdklabs/cdk-nag](https://github.com/cdklabs/cdk-nag), it contains a lot of  Aspects that assist with meeting compliance rules.

*Please note that I intentionally did not use the word "enforce" here. This is because it does not enforce deploying only compliant resources. A developer could just remove the Aspect and deploy the code. [AWS Config](https://aws.amazon.com/config/) can really "enforce" rules.*

### Modyfing 3rd Party Constructs

Another great use case for CDK Aspects is 3rd party constructs. Imagine you are using a higher-level construct that is provided by someone else. It uses an S3 bucket, but you can't configure it as you like, and you don't have direct access to it.

You can suggest making it configurable and/or make a contribution, but that could take some time or it might not even be possible for whatever reason. Well, you can apply an Aspect to the provided construct and modify the bucket that way. You gain control over something that otherwise would be not under your control. Awesome!

### Help With Refactoring

*This use case is mentioned in [The CDK Book](https://thecdkbook.com/) (which I highly recommend, it is worth every penny).*

Imagine that you'd like (or are forced to) do some refactoring and move stuff around. If you move a resource to a different construct it will get a new logical id. That will lead to the original resource being deleted and a new one being created. That is no big deal for stateless resources like Lambda, but it certainly is for stateful resources like S3.

You can provide a map of logical ids referencing an override for the logical id. The Aspect will apply the override, keep the logical id stable, and ensure that the resource doesn't get deleted.

### Simplify Integration Tests

For integration tests, you will most likely deploy to a temporary environment, do your tests, and delete the stack afterwards. Most likely, you have stateful resources (like databases) with the removal policy `RemovalPolicy.RETAIN`. These resources will not get cleaned up after the tests.

You could make the constructs with these resources configurable, but it is not a good idea to change your code to make something configurable just for tests. And the good news is: you don't have to. You can apply an Aspect just to the test stacks with `RemovalPolicy.DESTROY`. That way, the resources will be properly deleted.

*This idea originates from a discussion about Julian Michels talk about AWS CDK integration tests with [projen](https://github.com/projen/projen).*

## Example

This post isn't about how to implement Aspects (will be covered in a later post in detail, so stay tuned), here’s a brief example:

```typescript
class TagChecker implements IAspect {

  constructor(private readonly requiredTags: string[]) {
  }

  public visit(node: IConstruct): void {
    if (!(node instanceof Stack)) return;

    this.requiredTags.forEach((tag) => {
      if (!Object.keys(node.tags.tagValues()).includes(tag)) {
        Annotations.of(node).addError(`Missing required tag "${tag}" on stack with id "${node.stackName}".`);
      }
    });
  }
}
```

This will be the error if a `Stack` is missing a required tag:

```
[Error at /my-stack-id] Missing required tag "environment" on stack with id "my-stack-id".
Found errors
```

*TIL that [aws-cdk itself uses Aspects for tags](https://github.com/aws/aws-cdk/blob/master/packages/%40aws-cdk/core/lib/tag-aspect.ts).*

## Conclusion

With CDK Aspects, you can modify constructs without changing their implementation. However, you don't have to modify constructs with Aspects. You might as well just throw an error and let the developer fix it. CDK aspects are really powerful, and perhaps the various use cases have shown that there are no limits to your imagination (at least in some ways). Use that power wisely and don't annoy developers by making their lives harder. Instead, Aspect should make our lives easier and CDK more enjoyable.

## Ressources

[AWS CDK Developer Guide - Aspects](https://docs.aws.amazon.com/cdk/latest/guide/aspects.html)

[AWS DevOps Blog - Align with best practices while creating infrastructure using CDK Aspects](https://aws.amazon.com/blogs/devops/align-with-best-practices-while-creating-infrastructure-using-cdk-aspects/)

[AWS Summit DC 2021: Improve the developer experience with AWS CDK](https://youtu.be/R3AEIIw98j4?t=1431)

[The CDK Book](https://thecdkbook.com/)