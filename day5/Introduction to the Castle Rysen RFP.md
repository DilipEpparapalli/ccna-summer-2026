
## What This Is
Castle Rysen Coffee is a fictional post-apocalyptic coffee empire 
used as the running project scenario for the NetworkChuck CCNA 
course. Instead of learning networking concepts in isolation, every 
topic gets applied to a real business problem — designing and 
building the network infrastructure for Castle Rysen's locations.

This is how real networking works. A client hands you an RFP 
(Request for Proposal), you read their requirements, and you design 
a solution to meet them. NetworkChuck is forcing that mindset from 
Day 1.

## The Business Structure
Castle Rysen operates three tiers of locations:

| Location | Users | Scale |
|---|---|---|
| Central Office | ~200 | Supports 30 Fallout Shelters |
| Fallout Shelter | ~50 | Supports 50 District Shops |
| District Shop | ~15 | Customer-facing cafe |

One Central Office region can support up to **1,500 District Shops** 
at full scale. That number matters — it's why enterprise networking 
concepts like routing protocols, VLANs, and redundancy exist.

## Goals 
The RFP lays out 8 primary goals for the project. At a high level they cover: building an adaptable and resilient infrastructure across all three location types, providing full documentation, establishing redundant internet connectivity, supporting local Plex video streaming and surveillance cameras at every District Shop, enforcing security segmentation between user types, optimizing loop prevention and performance, and maintaining vigilant network monitoring across all locations. These goals essentially map directly to the CCNA syllabus — every major topic this course covers exists because the RFP requires it.

## What the Network Needs to Do
The RFP defines 6 phases of work:
- **Phase 1** — Requirements gathering and network design
- **Phase 2** — Device configuration, VLANs, routing, wireless
- **Phase 3** — Network services and security (NAT, DHCP, ACLs)
- **Phase 4** — Automation and programmability
- **Phase 5** — Troubleshooting and optimization
- **Phase 6** — Documentation and closeout

![[RFP.pdf]]