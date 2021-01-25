# Filling in the gaps

Around 2010-ish I was a part of a company that was executing acquisitions at an aggressive clip, which meant that at any given time I was involved in a passive aggressive knife fight over what to do with our infrastructure. Because public clouds were ramping up in popularity around this time, these were especially unique because it challenged existing competencies and surfaced all the expected fears. The end result was a migration of our infrastructure to AWS - while their platform was significantly more advanced than what we were doing at the time, it didn't come without it's own gaps.

When `@jacobsmith` finally wore me down enough to write about some of the things we've done with Kubernetes, some of the tooling / automation we made for multi-cluster management jumped out at me. One of the things that's stuck with me over the years is how many different aspects of my first adoption of a public cloud required "filling-in" - aspects of the billing, or IP planning, were things we needed to figure out on our own. Before everyone dusts off their pitchforks or blows up my inbox with links to [kubefed](https://github.com/kubernetes-sigs/kubefed) or similar, let me talk a bit lower-level as to why functionally or technically many of these projects would not work for our specific use cases.

Walking in the door day 1 at Equinix Metal, the expectation was set that we'd be aggressively expanding into new data centers, and that I'd be effectively the first tenant as establishing our cluster and deployment tooling was the pre-requisite for Engineering and Ops to do their work (check out Kelsey's blog on [turning the physical digital](https://metal.equinix.com/blog/turning-up-a-cloud/)). I could write an entire piece on the how we PXE and initialize a cluster in a new facility with no existing assets, but for now just trust that no amount of rubbing cluster-api or kubefed on the problem changes that paradigm. On top of running on bare metal, needing to break ground in new facilities, we have to run at least one cluster per-facility we're in and also need a home for more centrally run assets (e.g. our API / data stores). Take a look at Equinix Metal's [locations](https://metal.equinix.com/product/locations/) and planned location and you'll get an immediate sense we're talking at least dozens of clusters day 1, maybe upwards of 100+ over the coming years.

## Circling the problem

New platforms are both fun and terrible. A mentor of mine used a phrase "knowledge hard-won" that sums up the myriad of dark debt and dangerous corners waiting for you when implementing a platform. The topic of multi-cluster management is broad and treacherous, but we're going to narrow in on three main areas - if you were hoping for an article on gitops'ing your SRE's service-mesh CI/CD pipeline, apologies for disappointing.  The substantial problems (and areas of discussion) we set out to solve with the target outcome of providing a cohesive internal engineering platform on Kubernetes revolved around:

- Cluster Classifications
- Configuration Management
- Service Provider Boundaries

Despite the often over exaggerated complexity of a well rounded Kubernetes implementation, assessing how to approach Equinix's implementation was not materially different than analysis of any distributed system. I was concerned initially with the overarching power structures - how resources would be allocated, how control is shared, how workloads would be divided, how to keep a semblance of order across systems in a perpetual state of failure. Basic analysis of our topology led me quickly to some generalized categories of classes - it's largely irrelevant the exact names and purposes of these classes, but what was worthwhile was the logical grouping. These class definitions can drive:

- cluster tenants
- default cluster and/or component parameters
- structure for instantiating services
- cluster naming, domain naming, service naming
- network policies

This is just like anything in code: you want it to be something you can look at and quickly learn things about. Standardizing some basic conventions around cluster classes might read as pedestrian, but it can be a powerful tool to group behaviors down stream in a way that doesn't make it difficult to scale. Let's take a look at a real-world example of this in practice. There is a [repo](https://github.com/matoszz/blog) with the below info. This example will use the following tools:

- [Ansible](https://www.ansible.com/)
- [ArgoCD](https://argoproj.github.io/argo-cd/)
- [Cilium](https://cilium.io/)
- [Mkdocs Material](https://github.com/squidfunk/mkdocs-material)
- [External-DNS]()

WIP ->

## Our Stack
