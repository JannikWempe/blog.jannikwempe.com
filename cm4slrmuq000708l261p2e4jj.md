---
title: "Why and How to Use Snapshot Tests in AWS CDK"
seoTitle: "AWS CDK Snapshot Tests: A Practical Guide for Better Infrastructure"
seoDescription: "Learn how to leverage snapshot tests in AWS CDK to catch infrastructure issues early. Discover why they're essential for your IaC testing strategy and how t"
datePublished: Tue Dec 17 2024 15:12:33 GMT+0000 (Coordinated Universal Time)
cuid: cm4slrmuq000708l261p2e4jj
slug: aws-cdk-snapshot-tests
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734448327694/a94e4307-52a1-4595-a792-998e478a5a97.png
tags: aws, testing, jest, cdk, aws-cdk

---

When I first encountered snapshot tests, I was skeptical. The concept seemed strange, and their benefits weren't immediately obvious. But after experiencing their value firsthand, I've become a convert – and here's why I think you should give them a chance.

## What are Snapshot Tests (in AWS CDK)?

Snapshot tests **capture the output of a system** and save it as a reference "snapshot" file. During subsequent test runs, new snapshots are compared against this stored reference – **any differences trigger a test failure**. Think of it as taking a "photograph" of your system's output and comparing future changes against it.

![Flowchart illustrating two processes: The first run involves creating a snapshot using CDK code and CloudFormation; the snapshot is then created. Subsequent runs involve CDK code and CloudFormation to compare with the snapshot, resulting in either a pass or fail outcome.](https://cdn.hashnode.com/res/hashnode/image/upload/v1734444906658/b009a943-42f2-4eb8-8133-a92756ba9482.png align="center")

In AWS CDK, snapshot tests **verify the CloudFormation templates** that CDK generates from your infrastructure code. Whether you write CDK in TypeScript, Python, or other supported languages, your code ultimately **synthesizes into CloudFormation templates**. Snapshot testing ensures these templates remain consistent with your intentions.

Consider defining an S3 bucket in CDK – the snapshot test captures all generated CloudFormation properties, from basic bucket configurations to complex access policies. If a future code change modifies any of these properties unexpectedly, **the snapshot test fails.**

**Snapshot tests are versatile** – you can write them not only for individual constructs but also for entire CDK stacks, allowing you to verify your complete infrastructure definition in a single test.

## Why Use Snapshot Tests in AWS CDK?

While snapshot tests might initially seem like a curious approach to testing infrastructure, they offer several compelling benefits that make them invaluable in AWS CDK development.

### **Low Development Effort**

Writing snapshot tests requires minimal code – often just a few lines. Unlike traditional tests where you must write explicit assertions for each property, snapshot tests automatically capture and verify all aspects of your infrastructure. This makes them an excellent return on investment, providing broad coverage with minimal development overhead.

This is a snapshot test that I have written for my `WebhooksStack`.

```typescript
import { App } from 'aws-cdk-lib'
import { Template } from 'aws-cdk-lib/assertions'
import { config } from '../../config'
import { EventBusStack } from '../event-bus/event-bus-stack'
import { WebhooksStack } from './webhooks-stack'

describe('WebhooksStack', () => {
	it('should match the snapshot', () => {
		// ARRANGE
		const app = new App()

		const eventBusStack = new EventBusStack(app, 'EventBus', config)
		const webhooksStack = new WebhooksStack(app, 'Webhooks', {
			...config,
			eventBus: eventBusStack.eventBus,
		})

		// ACT
		const template = Template.fromStack(webhooksStack)

		// ASSERT
		expect(template.toJSON()).toMatchSnapshot()
	})
})
```

It just creates a CDK `App`, instantiates the `WebhooksStack` (and its dependencies), generates the CloudFormation template as JSON and asserts the snapshot to match the existing snapshot. It is all just a few lines of code.

### **Visual Infrastructure Changes**

One of the most powerful features of snapshot tests is their integration with version control systems. When you modify your infrastructure code, the changes are reflected in the snapshot diff, making them clearly visible in pull requests. This visual representation makes code reviews more effective – your team can easily spot and discuss infrastructure modifications before they reach production.

This is a real-world screenshot I have taken from a simple AWS Lambda timeout adjustment, but I think it is sufficient to get the idea:

![Screenshot from a GitHub pull request that shows a diff in an updated snapshot test file.](https://cdn.hashnode.com/res/hashnode/image/upload/v1734445468360/ca9e6cee-b9cd-446d-94dc-f56a1486a240.png align="center")

### **Guard Rails for Infrastructure Changes**

Snapshot tests act as a safety net, ensuring that all infrastructure changes are intentional. If you're updating an S3 bucket policy but accidentally modify the bucket's encryption settings, the snapshot test will catch this unintended change. This protection is particularly valuable in large teams where multiple developers work on shared infrastructure code.

### **Refactoring with Confidence**

When refactoring CDK constructs or reorganizing your infrastructure code, snapshot tests shine. They verify that your refactoring efforts maintain the same CloudFormation output, even if the CDK code structure changes significantly. This allows you to confidently modernize your infrastructure code without fear of accidental modifications to the deployed resources.

### **Additional Benefits**

* **Documentation**: Snapshots serve as living documentation of your infrastructure, showing exactly what CloudFormation resources your CDK code generates
    
* **Cross-Stack Validation**: When testing entire stacks, snapshots help identify unintended changes in resource dependencies and cross-stack references
    
* **Learning Tool**: For developers new to CDK, examining snapshot diffs helps them understand how their code changes translate to CloudFormation changes
    
* **Compliance Verification**: Snapshots can help ensure that infrastructure changes comply with organizational standards by making all modifications visible and reviewable
    

## How do you write snapshot tests in AWS CDK?

While writing snapshot tests is straightforward, there's an important caveat when testing infrastructure with Lambda functions. The generated CloudFormation template includes an `S3Key` property that contains a hash of your Lambda function's code. This means that **every time you modify your Lambda code**, even without changing the infrastructure, the snapshot test will fail because the `S3Key` hash changes. This isn't ideal when you only want to validate infrastructure changes rather than code changes.

Fortunately, **snapshot serializers** provide a solution to this problem. Testing frameworks like [Jest](https://jestjs.io/docs/expect#expectaddsnapshotserializerserializer) and [Vitest](https://vitest.dev/guide/snapshot#custom-serializer) support custom snapshot serializers that can modify how the snapshots are generated. By implementing a serializer, you can **strip out code-dependent elements** like the `S3Key` (which contains a hash based on the code) from the CloudFormation template before it's saved as a snapshot. This ensures your tests only fail when there are actual infrastructure changes, not just Lambda code updates.

```typescript
// ./setup-after-env.ts

// include this in `setupFilesAfterEnv` in the Jest config.

import 'aws-sdk-client-mock-jest'

import { config } from '../../src/config'

const bucketMatch = new RegExp(
	`cdk-[0-9a-z]{9}-assets-${config.env.account}-${config.env.region}`,
)
const assetMatch = /[0-9a-f]{64}\.zip/

/**
 * This is a custom snapshot serializer for the CDK.
 * It substitutes the bucket and asset zip parts with [ASSET BUCKET] and [ASSET ZIP] respectively.
 *
 * This ensures that the snapshot stays the same on asset changes.
 *
 * @see https://blog.bigbandsinger.dev/robust-cdk-snapshot-testing-with-snapshot-serializers#heading-making-snapshots-less-fragile-ftw
 */
expect.addSnapshotSerializer({
	test: (val) =>
		typeof val === 'string' && (val.match(bucketMatch) != null || val.match(assetMatch) != null),
	print: (val) => {
		// Substitute both the bucket part and the asset zip part
		let sval = `${val}`
		sval = sval.replace(bucketMatch, '[ASSET BUCKET]')
		sval = sval.replace(assetMatch, '[ASSET ZIP]')
		return `"${sval}"`
	},
})
```

I should mention that I borrowed this serializer code from another blog post – shout out to whoever wrote it! Sadly, the original link isn't working anymore as I write this, but hey, credit where credit is due.

### What Does My AWS CDK and Snapshot Tests Workflow Look Like?

After initially creating your snapshot files (by running `jest --testMatch '**/*.snapshot.test.ts'` you will have to run `jest --testMatch '**/*.snapshot.test.ts -u'` (notice the `-u` for “update“) to update them on subsequent changes to the infrastructure.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">I put my snapshot tests in files with the suffix <code>.snapshot.test.ts</code>which allows me to only run or update the snapshot tests by targeting them with the <code>testMatch</code> argument.</div>
</div>

Here's how I use snapshot tests in my workflow: When I modify infrastructure code, I first run the tests locally to generate updated snapshots. I review these changes carefully, and if they match my intentions, I commit both the code and updated snapshots together. Then I push to GitHub and create a PR. The CI pipeline runs the snapshot tests again, verifying that everything matches – this catches any unintended changes before they make it to production.

## Conclusion

Snapshot tests have become an essential part of my CDK development workflow. While they might seem unusual at first, their ability to catch unintended infrastructure changes with minimal effort makes them incredibly valuable. Combined with the custom serializer to handle Lambda code changes, they provide just the right level of protection without getting in the way.

If you haven't tried snapshot testing in your CDK projects yet, I encourage you to give it a shot. The small upfront investment in setting them up will pay off in confident infrastructure changes and more effective code reviews.