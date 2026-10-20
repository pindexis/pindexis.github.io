---
layout: post
title: "The hard part of bringing AI into business workflows"
date: 2026-10-19
---

"Which shipments are going to be late?" is a useful question. But in a supply chain application, there is probably already a screen that answers it.

The work starts with the next question: "What should I do about them?"

To answer that, someone might check inventory at the affected stores, look at expected demand, review outstanding orders, and consult the retailer's procedures. A delay might be harmless for one store and urgent for another.

There is value in making information easier to retrieve through an AI agent. The larger opportunity is understanding what the user needs that information for and carrying out more of the investigation on their behalf.

I work for a supply chain visibility vendor, so I'll use supply chain and retail context to explore what makes this difficult. Three challenges stand out: giving the agent access to the information it needs, making that information meaningful, and establishing enough trust to act on its recommendations.

## The workflow often lives with the user

A product serving several retailers has to accommodate different organizations, roles, and ways of working. Even people with the same job title may follow different procedures. Those procedures also change frequently over time.

We can implement a particular workflow in code. We do that all the time. The difficulty is the number of variations across customers and roles, and the effort required to maintain them as business needs evolve. Even when each workflow is understood, building dedicated functionality for every variation may be impractical.

Products commonly address this by providing shared capabilities: screens, filters, reports, and actions. Users connect them. They know which screen to open next, what to compare, and when to take action. The software supports the operations, while the person supplies much of the process.

An LLM creates an interesting possibility here. Given a goal, business guidance, and access to the necessary information and tools, it could help assemble and carry out that investigation. This could make it practical to support more variations without developing a separate product feature for each one.

Making that dependable requires more than a capable model.

## Can the agent reach the information it needs?

Consider that delayed shipment again. A visibility provider might know where it is and when it is expected to arrive. The retailer might hold inventory and sales data in its own systems.

Each source contains part of the answer.

Giving an agent access across those sources is often primarily a business challenge. A vendor may prefer to make its product the center of the AI experience, while a retailer may be unwilling to expose certain internal information to that vendor. The parties need to agree on what can be accessed and by whom.

MCP servers or other interfaces can give the agent a way to reach the necessary information without moving every source into one central database. But the availability of a technical connection does not resolve those business decisions.

## Can the agent interpret and combine the data?

Once the assistant can retrieve information, it needs to understand what that information represents.

Does an absent record mean zero stock or missing information? Do two systems use the same definition of an order? These details can change a recommendation completely.

Traditional applications were often built around a database, application logic, and a user interface. The interface presents information in a particular context, while the application handles rules and relationships behind it. Exposing the underlying data to an agent does not automatically expose that understanding.

Preparing systems for agents can therefore require substantial work: replacing unexplained internal fields with clear business concepts, documenting definitions, exposing business rules, and providing usable relationships between datasets. Some platforms can add a suitable access and interpretation layer. Others may need deeper restructuring.

In our own work, we consolidated the data sources available within our platform into Databricks and incorporated business rules into what we call dataset models. Each model is generally organized around a workflow and brings together the information we hold to support it. We also have agents that help create these models.

This preparation reduces the need for the agent to work out how to join underlying sources during every investigation, an approach we have found brittle in practice. The relationships and business logic can instead be established and reviewed as part of preparing the dataset.

Some datasets are expensive to compute, so we execute them on a schedule and let the LLM query the resulting snapshots. This moves part of the preparation out of the agent's investigation and makes routine querying more practical.

Alongside that structure, data annotations and explicit contracts need to explain what the information represents.

The problem becomes more complex when a workflow spans several data vendors. Each provider may describe its own data well without explaining how it relates to another provider's information.

Vendor A might report inventory by warehouse and SKU, while vendor B forecasts demand by store and product group. Connecting them requires mappings and business rules that neither vendor's documentation necessarily provides. An intelligent agent may identify that gap, but answering without resolving it requires assumptions.

Who is responsible for defining those relationships? The retailer, the vendors, or the team building the agent harness?

## What makes the result trustworthy?

Understanding the available data does not establish that it is sufficient for a particular decision.

Even if we assume the agent reasons correctly and combines the sources appropriately, a recommendation may still be unsuitable because a relevant business rule or exception was never supplied. This is particularly difficult when the missing information is not obvious to either the agent or the person reviewing its answer.

In a low-stakes conversation, an incorrect answer may cause little more than inconvenience. A business recommendation can affect inventory, deliveries, costs, and customers. Trust requires evidence that the workflow has the context it needs and produces appropriate results.

How do we build that evidence without first solving the open-ended problem of preparing an agent for every possible business question?

## Start with bounded workflows

In our work, we chose to start with defined, recurring workflows. That gave us a concrete scope within which to prepare the agent and validate its behavior.

In retail, users often perform the same kinds of tasks each day: investigate delayed shipments, identify potential stock shortages, or review orders that need attention. These recurring tasks within a particular role and organization give us a manageable starting point.

Assuming the necessary data can be made accessible, we can establish the definitions and relationships required for a specific task, then validate the results against its intended outcome. Narrowing the scope helps identify missing sources.

This is where a specification and planning step becomes valuable. The user and the agent investigate the workflow together, clarifying the goal, the information required, and the rules that guide the recommendation.

The result should be a workflow guide that identifies the required sources, explains how they should be joined, records the applicable business rules, and makes assumptions and unresolved questions visible. It should also establish what the workflow is expected to produce, when missing information should prevent a recommendation, and when user intervention is necessary.

This is the idea behind the planning layer we added: take a standard operating procedure or a complex task, investigate the available information and tools, and produce suggestions and an execution plan for stakeholder review.

Preparing the dataset model is also part of this process. In our implementation, this involves generating a SQL query that joins the relevant tables in Databricks and applies the required business logic. The result is then materialized as a snapshot and exposed as a single dataset for the workflow to query.

Review can involve the end user, a business consultant, and a technical reviewer. Each contributes a different perspective: whether the workflow reflects how the work is done, whether the business rules are appropriate, and whether the data preparation and execution are correct.

That review should include testing the proposed workflow against representative sample data and expected outcomes, including exceptions and missing information. This provides a way to check its behavior before relying on it routinely.

One objection is that this still resembles traditional development: understand the customer's needs, implement a workflow, and review the result. Is the LLM simply making that process faster?

Yes, and that speed can change what is practical.

When the required data and tools are available, the LLM can take on much of the investigation and implementation. Reviewers can focus on validating the approach, clarifying exceptions, and confirming how sources should be joined. The gain depends on making the generated workflow easier to validate than to build from scratch.

Reducing that effort could make it worthwhile to support many more workflows, including specialized tasks that would not justify a conventional development project. It could also make deeper, more complex investigations practical within the time available.

User corrections and business knowledge can also carry forward through updated workflow guides, examples, tests, or code. This gives the system a way to learn from experience.

As that knowledge accumulates, familiar situations may require less clarification, and related workflows may become easier to develop and review. Those gains need to be demonstrated through use. Validated results provide evidence of where the system can be trusted and where further investigation is still needed.

## Building trust one workflow at a time

Putting an LLM to useful work in a business requires confidence in the decisions its output supports.

Bounded workflows offer a practical way to build that confidence. We can investigate their requirements, make assumptions explicit, test their behavior, and learn from corrections. As the system demonstrates reliable performance, we can expand its scope.

This depends on a usable data foundation: access to the information required for the task, clear business definitions, documented relationships between sources, and retrieval that is fast and economical enough for routine use.

I don't see bounded workflows as a complete solution or a limit on how AI can eventually be used in a business, but they offer a starting point where value can be demonstrated and trust can grow.
