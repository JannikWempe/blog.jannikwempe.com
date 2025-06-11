---
title: "Adjusting Shopify App Pricing: Impact on New vs. Existing Users"
seoTitle: "Adjusting Shopify App Pricing: Impact on New vs. Existing Users"
seoDescription: "What happens to existing customers if you change or delete a pricing plan of your Shopify app?"
datePublished: Wed Jun 11 2025 12:17:53 GMT+0000 (Coordinated Universal Time)
cuid: cmbrx0xe1001302jr1fh47yjl
slug: change-the-pricing-of-your-shopify-app
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1749644252446/2fe362b8-46fb-4c0a-8346-5a6a916a4d0a.png
tags: shopify

---

[Sandro](https://sandrovolpicella.com/) and I have considered changing the pricing of our Shopify app [FraudFalcon](https://fraudfalcon.app/). We want to focus more on Enterprise customers and offer free trials instead of a free plan. We asked ourselves how to change the pricing of our app. What would happen to existing customers if we change or delete pricing plans in our app listing? We couldn't find good information, and even Shopify staff were not very helpful. If you are in a similar situation, this post is for you.

<div data-node-type="callout">
<div data-node-type="callout-emoji">⚠</div>
<div data-node-type="callout-text">Please make sure to test pricing changes on a non-production app first. Things may change, or you might handle the webhooks differently.</div>
</div>

## Managed Pricing

First of all, we are using [Shopify Managed App Pricing](https://shopify.dev/docs/apps/launch/billing/managed-pricing). We don't have to build a pricing page or a UI for plan changes ourselves. We simply link to the Managed Pricing page, and Shopify takes care of the rest.

<div data-node-type="callout">
<div data-node-type="callout-emoji">ℹ</div>
<div data-node-type="callout-text">There are some <a target="_self" rel="noopener noreferrer nofollow" href="https://shopify.dev/docs/apps/launch/billing/managed-pricing#limitations" style="pointer-events: none">limitations to Managed Pricing</a>. Even if you don't use the Managed Pricing feature, the insights still apply to your app.</div>
</div>

On the Shopify Managed Pricing page, it also shows which plan the customer is subscribed to:

![Screenshot showing the Shopify Managed Pricing page with the plan the customer is subscribed to](https://wempe.dev/_astro/shopify-plan-subscribed.B-qVfW00_HwpSg.webp align="left")

So what happens to the subscription of existing customers if you change or delete a pricing plan? And how is it shown on the Shopify Managed Pricing page?

## Changing an Existing Pricing Plan

In the previous screenshot, you can see that the customer is subscribed to the "Professional" plan. Let's change the price of that plan to $55,99 / month and have a look at the Shopify Managed Pricing page:

![Screenshot showing the Shopify Managed Pricing page with the plan the customer is subscribed to](https://wempe.dev/_astro/shopify-plan-price-changed.Q5K9WXv2_M5mj8.webp align="left")

As you can see, the customer still sees that they are subscribed to the "Professional" plan. However, the price has changed to $55.99 / month.

**But what happens to the actual subscription of the customer? Nothing.** The pricing plans in your listing are only relevant for new customers. Changes to them are not reflected in the subscriptions of existing customers. If you want to change the price for an existing plan, you have to do it yourself. That is what we wanted, so I won't go into detail on how to do that.

## Deleting a Pricing Plan

What happens if you delete a pricing plan? Nothing. The plan simply disappears from the Shopify Managed Pricing page.

![Screenshot showing the Shopify Managed Pricing page with current plan being deleted](https://wempe.dev/_astro/shopify-plan-deleted.DocXr_UE_i9gHJ.webp align="left")

Again, nothing happens to the subscriptions of existing customers.

If you recreate the plan with the exact same name and handle, the new plan will still not be treated as the current plan. Here I have recreated the "Professional" plan with the same name and handle:

![Screenshot showing the Shopify Managed Pricing page with current plan being deleted](https://wempe.dev/_astro/shopify-plan-recreated.D7SJdDC5_Kj9gE.webp align="left")

## Conclusion

We wanted to adjust the pricing of our plans while keeping their names. However, we didn't want the user to see the plan with the new price as their "current" plan on the Shopify Managed Pricing page because we thought that might be confusing and could prompt them to cancel their subscription, mistakenly believing that they'd be charged more (which isn't the case). So we deleted the existing plans and recreated them with the new price, but with the same name and handle.

In essence, the pricing plans you set up are only relevant at the moment of subscription. Once a customer has subscribed, any modifications to the pricing plans—whether altering or deleting them—do not affect their current subscription. This means that if you want existing subscribers to experience a change in their pricing, you must manage that transition separately. Understanding this behavior is key to making informed decisions when updating your app's pricing structure.