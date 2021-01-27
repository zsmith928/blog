# Filling in the gaps

Around 2010 I was a part of a company that was going through a series of acquisitions, which meant that on any given day I was involved in a passive aggressive knife fight over what to do with our infrastructure. Public clouds were ramping up in popularity around then, and this directly intersected with our needs; challenging existing competencies and surfacing all the expected fears. The end result was a migration of our infrastructure to AWS - while their platform was significantly more advanced than what we were doing at the time, it didn't come without its feature gaps.

When `@jacobsmith` finally wore me down enough to write about some of the things we've done with Kubernetes, parts of our architecture built for multi-cluster management jumped out at me. I'm still struck by how many aspects of that first public-cloud adoption required "filling-in", and in many ways, not much has changed. New platforms are both fun and terrible. A mentor of mine used a phrase "knowledge hard-won" that sums up the myriad of dark debt and dangerous corners waiting for you when implementing a platform.

Before everyone dusts off their pitchforks or blows up my inbox with links to [kubefed](https://github.com/kubernetes-sigs/kubefed) or KubeSphere, I'd like to explain why such projects wouldn't work for our specific use cases. From day one, Equinix Metal made clear that we'd be aggressively expanding into new data centers. Moreover, establishing our own PXE stack, cluster bootstrapping, and continuous deployment tooling was the pre-requisite for Engineering and Ops to do their work: so I'd effectively be the first tenant - no niceties, there. (For more on this, check out Kelsey's blog on [turning the physical digital](https://metal.equinix.com/blog/turning-up-a-cloud/)). We need to both maintain a home for centrally run assets (e.g. our API / data stores) _and_ run at least one cluster per facility we're in - the icing being a degree of variability of what networking or not we may have.

The topic of multi-cluster management is broad and treacherous, but we're going to narrow in on three main areas. The main problems we set out to solve as we worked to build a cohesive internal engineering platform on Kubernetes with the described constraints revolved around:

- Cluster Classifications
- Configuration Management
- Service Provider Boundaries

Despite the often overexaggerated complexity of a well-rounded Kubernetes implementation, assessing how to approach Equinix's architecture was not materially different than analysis of any other distributed system. I was concerned initially with the overarching power structures - how resources would be allocated, how control is shared, how workloads would be divided, how to keep a semblance of order across systems in a perpetual state of failure. Grouping workload types with a dash of organizational structure for access controls led me to some generalized categories of classes - their exact names and boundaries aren't relevant here, but what was worthwhile was the attributes they shared and logical grouping which could be applied around them. These class definitions can drive:

- cluster tenants
- default cluster and/or component parameters
- structure for instantiating services
- cluster naming, domain naming, service naming
- network policies

This is just like anything in code: you want it to be something you can look at and quickly learn things about. Standardizing some basic conventions around cluster classes might read as pedestrian, but it can be a powerful tool to group behaviors downstream in a way that doesn't make it difficult to scale.

Let's take a look at an example of this in practice. I put together a [repo](https://github.com/matoszz/blog) with some reference material, which includes the use of:

- [Ansible](https://www.ansible.com/)
- [ArgoCD](https://argoproj.github.io/argo-cd/)
- [Cilium](https://cilium.io/)
- [Mkdocs Material](https://github.com/squidfunk/mkdocs-material)
- [External-DNS](https://github.com/kubernetes-sigs/external-dns)

Take a look at the [ansible inventory](https://github.com/matoszz/blog/blob/main/ansible/inventory/example.yml) - you'll see the usual suspects, but also the attribute `k8s_cluster_class`. Within the roles, some uses of this grouping:

- Adding the cluster to ArgoCD based [on class](https://github.com/matoszz/blog/blob/main/ansible/roles/argo/tasks/main.yml#L10)
- Generating a per-cluster [values file](https://github.com/matoszz/blog/blob/main/ansible/roles/metadata_collector/templates/values.yml.j2#L3) with an ArgoCD [app-of-apps](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/)
- Querying [Netbox](https://netbox.readthedocs.io/en/stable/) and outputting attributes you could use in a [Cilium network policy](https://docs.cilium.io/en/v1.9/gettingstarted/host-firewall/#enable-the-host-firewall-in-cilium)
- Documentation via an output of new cluster inventory and [automated updates to your mkdocs.yml](https://github.com/matoszz/blog/blob/main/ansible/roles/physical_inventory/tasks/main.yml)

While some templating is convenient, the relevance of this example is the values inheritance - the Ansible facts and values generation feeds the `/cluster-services/` values. If you review ArgoCD's [declarative setup](https://argoproj.github.io/argo-cd/operator-manual/application.yaml), you'll find you can pass values in not just with a file, but as a _block_ - this gives you the ability to inherit through apps and into the destination chart, such as the external-dns example in the repo. By creating and passing values in this way, consistent generation of cluster attributes and subsequently management of the cluster services becomes easier to scale.

The above example is representative of "where we started", and the next piece in this series surrounding Configuration Management will touch more on "where we're going" with a centralized API for cluster management, cluster-local controllers, and more.
