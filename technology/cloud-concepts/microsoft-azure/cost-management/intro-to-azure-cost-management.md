---
description: Because money doesn't grow on trees
---

# Intro to Azure Cost Management

## How does Azure pricing work?

### Consumption Based Pricing

Consumption based pricing is not unique to Azure, and most cloud services use it. On a basic level, you only pay for what you use. Whether that be hours a compute resource is running, the number of executions of a function, or total GB of storage used, for example. This is typically referred to as a "Pay-As-You-Go" plan. This will be the plan for the bulk of what we discuss here.

### Fixed Cost Pricing

Fixed cost pricing is what you would most likely be familiar with if you're new to cloud billing. You pay an agreed upon rate, and you get a resource for a set amount of time. This cost does not flex depending on usage like Consumption Based pricing does. In Azure, these are commonly referred to as "Reserved Instances". Essentially, you agree to pay for 1-3 years of using a VM, and are then given a "Reserved" rate that can be pre-paid up front, or paid monthly. Microsoft gets assurance of the resource being provisioned and generating revenue for a set length of time, and the customer gets a discount in exchange for this agreement. This pricing model is often utilized for Virtual Machines that will be required to run 24/7 anyways, thus making a fixed cost model more enticing, among other scenarios.

### Examples

#### VM - Consumption Based

We have a requirement for a virtual machine that is used to run an employee timecard software. This is a fairly lightweight application, and only needs to run on weekdays from 8AM to 6PM as hourly employees are not scheduled outside of this time range. In this instance, a consumption based VM would likely be our best option. Across four weeks in a month, with the VM being active for 10 hours 5 days per week, that's 200 hours per month. On a $0.188/hr consumption based VM, that's about **$22/mo**.

Compare that to a pre-paid reserved instance. For one-year reserved VM of the same size, due to it provisioned 24/7, the cost would be about $41/mo. Even with 3 years reserved, that still comes out to $36/mo. In this instance, consumption-based pricing is the most cost-effective method.

#### VM - Fixed Cost Reserved Instance

Let's take the above example and say now we need to have a VM that hosts a public database of their products. Our customer has specified that this resource must always be available, 24/7, as their customers cannot see what SKUs they have availble without it. In this case, we'd want to consider a reserved VM. At the same size that costs $0.188/hr, to run this for 730 hours (730 hours \* 12 months = 8760 hours in a year. 8760 hours / 24 hours = 365 days in a year) per month, that balloons our cost to **$70/mo.**

Let's now look at a reserved instance. For 1 year, we'd pay $495.55 upfront, or $41.33/mo. If we were confident in this resource being required for a 3-year term, that would further take our upfront cost to $958.96, further reducing the monthly cost to just **$26.64/mo.** In this instance, fixed cost reserved based pricing is the most cost-effective method.



## Controlling Costs

### Creating a Budget and Cost Alerts

Assigning a Budget to an Azure Subscription or Resource Group helps plan for costs and allows you to review expected vs actual spending. Additionally, you can use budgets to create spending alerts and kick-off automations that help control costs.

Start out in the Azure portal, in the scope you'd like the budget assigned to. For example, we want to place a budget on our Lab resource group, so we'll navigate there and select Budgets in the left navigation bar:

![](<../../../../.gitbook/assets/image (1) (1) (1).png>)

In Budgets, we'll select the Add button. We can also change our scope to the parent Subscription here, if we'd like

![](<../../../../.gitbook/assets/image (8) (1) (1).png>)

We'll then be shown the budget creation screen. Here we can set the name of the budget, the reset period, and any start and end dates to be defined. By default, budgets will be valid for 2 years. Helpfully, a total scope cost is shown in the right side of this page to help us define what the budget limit should be. As this is a lab group costs can fluctuate depending on what is being tested, so I've set this to a $100.

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption><p>Azure Budget Creation</p></figcaption></figure>

We'll now set our alert conditions on the next page. I've configured this to send me an email once this resource group gets to 80% of actual costs on the budget, or $80. Don't worry about setting an action group for now. I've then entered my email and saved the new budget and alerts.

<figure><img src="../../../../.gitbook/assets/image (20) (1).png" alt="" width="484"><figcaption></figcaption></figure>

### Automating Cost Saving Actions

With Azure Automation Runbooks, we can take this a step further. [Check out this article on automating cost management actions to give yourself an extra set of eyes when controlling spending!](automating-azure-cost-management.md)
