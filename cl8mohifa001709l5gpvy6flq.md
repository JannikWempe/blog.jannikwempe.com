---
title: "Mastering AWS CDK Aspects"
datePublished: Thu Sep 29 2022 06:31:28 GMT+0000 (Coordinated Universal Time)
cuid: cl8mohifa001709l5gpvy6flq
slug: mastering-aws-cdk-aspects
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1664433023392/hqulxhQZs.png
tags: aws, aws-cdk

---

## CDK Aspects Introduction

CDK Aspects are a powerful tool provided by the AWS Cloud Development Kit (CDK). They are utilizing the [Visitor Pattern](https://refactoring.guru/design-patterns/visitor). By applying a CDK Aspect to a specific `scope`, you get access to every child node within it. You can inspect them or alter them. That `scope` can be any `IConstruct` which includes `App` and `Stack` as they extend `Construct`. Therefore an Aspect can be applied at various levels.

> CDK Aspects are a way to apply an operation to every construct in a given scope.

This should be enough on a high level about what CDK Aspects are for this post. I have written a post on the [Learn AWS hashnode blog](https://aws.hashnode.com/) about [The Power of AWS CDK Aspects](https://aws.hashnode.com/the-power-of-aws-cdk-aspects). Read that post if you are interested in some more theory.

In this post, I will show you a few possible implementations of custom CDK Aspects on how you can leverage third-party Aspects. We will create Aspects that alter your infrastructure, that show errors, that take arguments and that don't take arguments.

The goal is that you should be able to write your own Aspects and have a better idea of possible use cases for CDK Aspects.

You can find all the code in a CDK app that you can play around with in my GitHub repository [JannikWempe/cdk-aspects-examples](https://github.com/JannikWempe/cdk-aspects-examples).

## Implementing Custom CDK Aspects

Aspects have a small interface. This is what we have to implement creating our own custom Aspects:

```typescript
interface IAspect {
    visit(node: IConstruct): void;
}

```

This terms `visit` (based on Visitor Pattern), `node`, and `IConstruct` should already be familiar to you after the introduction. The `visit` method will be called for every `node` within the `scope` you are applying the Aspect to.

### Example 1: Applying Tags

The [standard way of applying tags in a CDK](https://docs.aws.amazon.com/cdk/v2/guide/tagging.html) app is by using `Tags.of(scope).add(key, value)`. You know what? [That is already using an Aspect](https://github.com/aws/aws-cdk/blob/f834a4537643b32131076111be0693c6f8f96b24/packages/@aws-cdk/core/lib/tag-aspect.ts#L152-L154).

But maybe you want something more custom and make certain tags required.

This is how a custom `ApplyTags` Aspect could look like:

```typescript
type Tags = { [key: string]: string } & {
  stage: 'dev' | 'staging' | 'prod';
  project: string;
  owner: string;
}; 

class ApplyTags implements IAspect {
  #tags: Tags;
  
  constructor(tags: Tags) {
    this.#tags = tags;
  }

  visit(node: IConstruct) {
    if (TagManager.isTaggable(node)) {
      Object.entries(this.#tags).forEach(([key, value]) => {
        this.applyTag(node, key, value);
      });
    }
  }

  applyTag(resource: ITaggable, key: string, value: string) {
    resource.tags.setTag(
      key,
      value
    );
  }
}
```

In `visit` we narrow down the `node` to a construct that is taggable by checking if `TagManager.isTaggable(node)` is truthy. After that, we go ahead and add the desired tags.

TypeScript helps here because you won't be able to just call `setTag` on any `IConstruct`. You have to filter out the resources that are not taggable. This is a common pattern for Aspects: You narrow down all the nodes to the resources you want to act upon.

This is how you apply `ApplyTags` on the `App`-level:

```typescript
const app = new cdk.App();
const myStack = new MyStack(app, 'MyStack');

const appAspects = Aspects.of(app);

appAspects.add(new ApplyTags({
  stage: 'dev',
  project: 'CDK Aspects',
  owner: 'Jannik Wempe'
}));
```

You first call `Aspects.of(scope)` to get access to the `Aspects` within `scope` and add your own Aspect to it.

This will add three tags to all taggable resources that are within your `app`.

### Example 2: Enabling S3 Bucket Versioning

This example will show you how to alter constructs within the scope. Use this power responsively as you will end up in a mess if you overuse this.

You want to enable bucket versioning for all of your buckets? You could create your own `VersionedBucket` by extending `Bucket`. But that is probably not the best idea. You would end up maintaining your own L2 construct and the requirements for that bucket will increase. You will end up with a custom bucket construct that would have tons of props in order to be suitable for all sorts of scenarios. Using an Aspect for that is more maintainable – and it requires fewer lines of code.

```typescript
class EnableBucketVersioning implements IAspect {
  
  visit(node: IConstruct) {
    if (node instanceof CfnBucket) {
      node.versioningConfiguration = {
        status: 'Enabled'
      };
    }
  }
}
```

That's it. All of your `CfnBucket`s within `scope` will be versioned.

Note that all `Bucket`s (L2 construct) are a `CfnBucket` (L1 construct) but `CfnBucket`s are not necessarily a `Bucket`. Double check that you are narrowing down to the desired ones by adding some `console.log`s – as a real developer does 😅

### Example 3: Enforcing Minimum Lambda Node Runtime Version

As mentioned in the previous example, altering all kinds of constructs through Aspects can end up in a mess that is hard to debug. Displaying an error and letting the developer intentionally change the code might be the better option. This is what we will do in this example.

Let us create an Aspect that checks all Lambda functions runtime versions and enforces a minimum NodeJS version being used. We pass a `minimumNodeRuntimeVersion: Runtime` into the `constructor` of our `EnforceMinimumLambdaNodeRuntimeVersion` Aspect *(Yes, I know, it is a creative name 😅)*. The Aspect would check every Lambda functions runtime and adds an error to the functions `Annotations` if the check fails:

```typescript
class EnforceMinimumLambdaNodeRuntimeVersion implements IAspect {
  #minimumNodeRuntimeVersion: Runtime;

  constructor(minimumNodeRuntimeVersion: Runtime) {
    if (minimumNodeRuntimeVersion.family !== RuntimeFamily.NODEJS) {
      throw new Error('Minimum NodeJS runtime version must be a NodeJS runtime');
    }
    this.#minimumNodeRuntimeVersion = minimumNodeRuntimeVersion;
  }

  visit(node: IConstruct) {
    if (node instanceof CfnFunction) {

      // runtime is optional for functions not being deployed from a package
      if (!node.runtime) {
        throw new Error(`Runtime not specified for ${node.node.path}`);
      }

      if (!node.runtime.includes('nodejs')) return;

      const actualNodeJsRuntimeVersion = this.parseNodeRuntimeVersion(node.runtime);
      const minimumNodeJsRuntimeVersion = this.parseNodeRuntimeVersion(this.#minimumNodeRuntimeVersion.name);

      if (actualNodeJsRuntimeVersion < minimumNodeJsRuntimeVersion) {
        Annotations
          .of(node)
          .addError(`Node.js runtime version ${node.runtime} is less than the minimum version ${this.#minimumNodeRuntimeVersion.name}.`);
      }
    }
}

  private parseNodeRuntimeVersion(runtimeName: string): number {
    const runtimeVersion = runtimeName.replace('nodejs', '').split('.')[0];
    return +runtimeVersion;
  }
}
```

This one is slightly more complex but it follows the same principles as the previous example. The main difference is the `Annotations.of(node).addError()` part. We don't throw an error that would abort the synthesis and show an ugly, unhelpful stack tract, but we are adding an error annotation to the construct itself. This is what error annotations would look like:

```
[Error at /MyStack/MyLambda1/Resource] Node.js runtime version nodejs12.x is less than the minimum version nodejs16.x.
[Error at /MyStack/MyLambda2/Resource] Node.js runtime version nodejs12.x is less than the minimum version nodejs16.x.
```

Note that it shows two errors. The synthesis doesn't stop after the first one.

### Example 4: Configure Lambda Log Groups

This one is also different from the previous ones. We now want to add an actual resource. Lambda functions are creating their CloudWatch log groups implicitly. You won't be able to see the log groups in the synthesized CloudFormation template but they will be created for you. That is why we can't just narrow the `node` in `visit` down to `CfnLogGroup`. The log group won't be there.

In order to customize a log group of a Lambda we have to explicitly create it with the `logGroupName` being `/aws/lambda/[REPLACE_WITH_LAMBDA_FN_NAME]`. We can pass configurations to that explicitly created log group.

This is what it looks like:

```typescript
class LambdaLogGroupConfig implements IAspect {
  #logGroupProps?: Omit<LogGroupProps, "logGroupName">;

  constructor(logGroupProps?: Omit<LogGroupProps, "logGroupName">) {
    this.#logGroupProps = logGroupProps;
  }

  visit(construct: IConstruct) {
    if (construct instanceof CfnFunction) {
      this.createLambdaLogGroup(construct);
    }
  }

  private createLambdaLogGroup(lambda: CfnFunction) {
    new LogGroup(lambda, "LogGroup", {
      ...this.#logGroupProps,
      logGroupName: `/aws/lambda/${lambda.ref}`,
    });
  }
}
```

To be honest, this one was hard for me as it has some caveats. Thanks, Glib Shpychka for helping me out on the [cdk.dev Slack channel](https://cdk.dev/).

We can pass all `LogGroupProps` to the Aspect excluding the `logGroupName` as this is the prop that connects the `LogGroup` to the `CfnFunction`. We again narrow down the node to the construct we are interested in: `CfnFunction`. Now we create a `LogGroup` and are using the `CfnFunction` construct itself as the `scope` for that `LogGroup`. This helps us to avoid conflicts in the CDK-generated logical IDs as the scope will be used to generate a prefix.

Now you might wonder what the `lambda.ref` is about and why it's not just `lambda.functionName`. This is the tricky part.

It is [best practice to not assign specific resource names to your constructs but rather let CDK generate them for you](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html#best-practices-constructs#best-practices-apps-names). If you log `lambda.functionName` to the console you would see something like this: `${Token[TOKEN.246]}`. It is a `Token`. What is a `Token`?

> Tokens represent values that can only be resolved at a later time in the lifecycle of an app.

*(from [CDK docs - Tokens](https://docs.aws.amazon.com/cdk/v2/guide/tokens.html)*)

I tried different kinds of things to resolve the `Token` (e.g. checking `Token.isUnresolved`) but nothing was working. The solution was to use `CfnRefElement.ref` which lets you access the CloudFormation `{ Ref }` element – the physical name. 

With  `LambdaLogGroupConfig` you could configure all Lambda functions log groups based on the environment like this:

```typescript
appAspects.add(new LambdaLogGroupConfig({
  retention: config.stage.name === "prod" ? RetentionDays.ONE_MONTH : RetentionDays.ONE_WEEK
}));

// OR

const myProdStackAspects = Aspects.of(myProdStack);
myProdStackAspects.add(new LambdaLogGroupConfig({
  retention: RetentionDays.ONE_MONTH
}));

const myDevStackAspects = Aspects.of(myDevStack);
myDevStackAspects.add(new LambdaLogGroupConfig({
  retention: RetentionDays.ONE_WEEK
}));
```

## Using 3rd-Party CDK Aspects

There are already useful CDK Aspects out there that are ready to be used in your CDK application. In this section I want to give you an example for one of them: One of them is [cdk-nag](https://github.com/cdklabs/cdk-nag).

`cdk-nag` contains several Aspects to check your applications for best practices. It is especially useful if you need to be HIPAA-compliant or have other compliance requirements. It is inspired by [cfn_nag](https://github.com/stelligent/cfn_nag) which is a a tool checking for patterns in your CloudFormation templates.

After installing `cdk-nag` with your favorite package manager (e.g. `npm install cdk-nag`) you can use one of the Aspects just like the others above:

```typescript
const app = new cdk.App();
new MyStack(app, 'MyStack');

const appAspects = Aspects.of(app);
appAspects.add(new AwsSolutionsChecks());
```

The `AwsSolutionChecks` [includes a lot of rules](https://github.com/cdklabs/cdk-nag/blob/main/RULES.md#awssolutions) which is why you initial output after trying to synthesize your app could look something like this:

```
[Error at /MyStack/CdkAssertionsQueue/Resource] AwsSolutions-SQS2: The SQS Queue does not have server-side encryption enabled.
[Error at /MyStack/CdkAssertionsQueue/Resource] AwsSolutions-SQS3: The SQS queue does not have a dead-letter queue (DLQ) enabled or have a cdk-nag rule suppression indicating it is a DLQ.
[Error at /MyStack/CdkAssertionsQueue/Resource] AwsSolutions-SQS4: The SQS queue does not require requests to use SSL.
[Error at /MyStack/MyBucket/Resource] AwsSolutions-S1: The S3 Bucket has server access logs disabled.
[Error at /MyStack/MyBucket/Resource] AwsSolutions-S2: The S3 Bucket does not have public access restricted and blocked.
[Error at /MyStack/MyBucket/Resource] AwsSolutions-S10: The S3 Bucket or bucket policy does not require requests to use SSL.
```

Each rule has an identifier like `AwsSolutions-S2`. You can [turn off rules individually](https://github.com/cdklabs/cdk-nag/blob/main/RULES.md#awssolutions).

## Conclusion

We had a look at different possibilities you have when creating your own CDK Aspects and have used a 3rd-party Aspect as well. By now you should have a good idea of how to work with CDK Aspects. Maybe you even have some use cases in mind in which Aspects could be helpful.

There is also one other useful use case that I haven't explicitly shown above: You can alter constructs that you don't have direct access to. Imagine using a 3rd party L3 construct that does not expose ways to customize one of the underlying resources. You can create a CDK Aspect and apply it to that 3rd party construct.

All the code is on GitHub [JannikWempe/cdk-aspects-examples](https://github.com/JannikWempe/cdk-aspects-examples).

Thanks for reading this article ✌🏼