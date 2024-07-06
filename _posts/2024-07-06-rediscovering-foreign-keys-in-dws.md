---
layout: post
title: Rediscovering Foreign Keys in DWs
---

Foreign keys are standard in transactional databases but rarely used in data warehouses. This isn't just about database design - it's a missed chance to improve data discovery and use.

In data warehouses, foreign keys matter for metadata, not constraints. Unlike transactional databases where it's typical for a single team to manage the schema, data warehouses mix data from many sources with different structures. The connections between these datasets exist, but they're not explicit. They're just in the minds of data engineers.

This presents two challenges. First, finding data connections takes too long. Second, you can't automate anything based on these hidden relationships. Take joining Stripe data with internal data. There's probably a shared customer ID column, but you have to know it exists. I've spent hours searching table schemas for the right join key, sometimes finding that the ID I thought was right (`customer_id`) only works sometimes, and we actually join on something else (`subscription_id`).

With proper foreign keys defined, you could see these connections quickly. I'd love to see a network graph of a data warehouse with tables as nodes and foreign keys as edges. It could make for a great user experience when joining tables.

But how do we add foreign keys to existing data warehouses? The system's there - most support foreign keys in the schema. But how do we find the actual relationships? Should we manually encode them, like refs in dbt? Should we discover them based on metadata, then have humans check and approve?

Where should this information live? In an existing data catalog? As a standalone tool? Should dbt get involved? Could a tool like Fivetran propagate foreign keys as part of ETL? Probably not, as we need a whole-warehouse approach, not a source by source one. After all, the whole point is to link disparate sources with no knowledge of one another.

I think there's room for an open-source tool for curating foreign keys. It could run as a service, regularly check for schema changes, suggest relationships for humans to approve, and write them back to the data warehouse.

There's also potential to use these encoded foreign keys to help write SQL, but that feels like an entirely separate tool.

The bigger question remains: how do we seed foreign keys in all the data warehouses that don't have them now? It's a challenge, but solving it could greatly improve how we work with data warehouses.

_I've had these thoughts about foreign keys in data warehouses for some years now, but getting them into a coherent written form always felt daunting. With the help of LLMs, I was finally able to put this post together (in a way that felt easy enough). This piece was co-written by an AI assistant, Claude. It's a blend of my ideas and experiences with Claude's ability to organize and articulate._
