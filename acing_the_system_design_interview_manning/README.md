
# Table of Contents

1.  [Part 1 Theory](#org0829574)
    1.  [Chapter 1 A walkthrough](#org638c7c5)

Study notes from : 
Acing the system design Interview by Zhiyong Tan
Manning Press 2024
ISBN : 9781633439108


<a id="org0829574"></a>

# Part 1 Theory


<a id="org638c7c5"></a>

## Chapter 1 A walkthrough

1.  A bagle shop

    <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
    
    
    <colgroup>
    <col  class="org-left" />
    
    <col  class="org-left" />
    </colgroup>
    <tbody>
    <tr>
    <td class="org-left">gzip</td>
    <td class="org-left">brotli</td>
    </tr>
    
    
    <tr>
    <td class="org-left">- faster in compression and decompression</td>
    <td class="org-left">- better compression ratios</td>
    </tr>
    
    
    <tr>
    <td class="org-left">- older and better used for binary files</td>
    <td class="org-left">- newer and better for text</td>
    </tr>
    
    
    <tr>
    <td class="org-left">&#xa0;</td>
    <td class="org-left">&#xa0;</td>
    </tr>
    </tbody>
    </table>
    
    1.  So we use brotli to compress the JS bundle the user downloads
        for our ReactJS webapp
    2.  we also have ios and android app
    
    \#+BEGIN\_ SRC dot :file system-architecture.png
    digraph {
    rankdir = LR;
    
    subgraph cluster<sub>client</sub> {
    label = "Client";
    labeljust = "l";
    
    Browser [shape=box];
    Android [shape=box];
    iOS [shape=box];
    }
    
    subgraph cluster<sub>server</sub> {
    label = "Server";
    labeljust = "l";
    
    Frontend [shape=box];
    Backend [shape=box];
    SQL [shape=cylinder];
    }
    
    Browser -> Frontend;
    Frontend -> Backend;
    Android -> Backend;
    iOS -> Backend;
    Backend -> SQL;
    }
    \#+END\_ SRC
    
    ![img](system-architecture.png)
    
    The service now has millions of customers and we had set up **stateless**
    service so we could handle them all in different locations and scale
    up the backend without any hiccups.
    
    ![img](geodns.png)
    
    1.  Adding a caching service
    
        We now add redis in memory database as the cache.
        ![img](./images/cashing_redis.png)
    
    2.  CDN
    
        We remove static files from source code repo to third party CDN
        service and serve them from CDN urls.
        
        ![img](./images/csn.png)
        examples : cloudflare, rackspace, aws cloudfront
    
    3.  Brief on horizontal scalibility, cluster management and CI CD
    
        Our backends are <a id="org78ef3ef">idempotent</a> meaning they respond with the
        same data when queried the same regardless of other factors.
        <a href="#org78ef3ef">Idempotent</a> systems are thus horizontally scalable.
        
        1.  CI CD and <a href="#orgc153f3d">IAC</a>
        
            <a id="orgdacfd99">Infrastructure as code</a>.<a id="orgc153f3d">IAC</a> is the process of managing
            and provisioning computer data centers through machine readable
            config files rather than physical hardware config, or
            interactive configuration.
            
            -   Docker or podman to containerize
            -   docker swarm or kubernetes for clustering and load balancing
            -   Ansible or Terraform for configuration management
                -   Terraform DSL (Domain specific language) is compatible with
                    multiple cloud providers and can provision infrastructure
                    without vendor lock in.
        
        2.  gradual rollout and rollback
        
            rollout changes slowly like 1% , 3%, 5% and so on to 100% if no
            problems are detected. Roll back the change if any of these
            occur:
            
            1.  Increased user churn because of performance or dislike of updates.
            2.  Crash
            3.  memory leaks
            4.  increased latency
            5.  increased resource usage : cpu, ram etc
        
        3.  Experimentation
        
            This deals with UX change and consumer reaction to those
            changes rather than performance that is associated with rollout
            and rollback. Common approaches:
            
            1.  A/B testing
            2.  Multivariate testing
                1.  Multi armed bandit
            
            These allow for short feedback cycles and faster iterations.

2.  <a href="#org301204d">Functional Partitioning</a> and centralization of cross cutting concerns

    <a id="org301204d">Functional Partitioning</a> : seperation of functions into
    different services or hosts.
    
    1.  Shared services
    
        -   We create a shared search service with elasticsearch
        -   On top of caching, CDN, horizontal scaling we add <a href="#org301204d">functional
            partitioning</a>
            
            ![img](./images/funcpartition.png)
        
        1.  Adding shared services
        
            -   added logging service of log based message broker
            -   <a id="orgdff6a21">Elastic Stack</a> : Elasticsearch, Logstach, Kibana, Beats
            -   use <a id="orga8d82d9">distributed logging</a> system like `zipkin` or `jaegar` to
                trace request traversal
            -   monitoring and alerting system for customer complaints and
                customer service teams
    
    2.  TODO Ways to centralize cross cutting concerns
    
        1.  <a id="org23777bd">API gateway</a>
            We expose our services to other companies and external
            developers. We implement
            -   Authorization and authentication
            -   rate limiting
            -   billing
            -   logging monitoring and alerting on request level
            -   analytics
                
                ![img](./images/apigateway.png)
        
        2.  <a id="org6e9dee8">Service mesh</a> / <a id="orgc249bfe">sidecar pattern</a>
            This solves the awkward architecture, latency, complexity of
            <a href="#org23777bd">API gateway</a>.
            -   Use <a href="#org6e9dee8">service mesh</a> framework like Istio.
                
                ![img](./images/servicemesh.png)
            
            -   Sidecarless Servicemesh
                In early stages of development and the cutting edge.
        
        3.  <a id="org37ca202">CQRS</a> : Command Query Responsibility Segregation
            -   microservices pattern where read and write are
                functionally partitioned into seperate services.
            -   low latency, scalable, easier to maintain and use
            -   seperate table or databases for reading from and writing
                into and they are synced periodically as required.
        
        4.  Decorator pattern
        
        5.  Aspect Oriented Programming
        
        Look at these two topics yourself
    
    3.  Batch Streaming ETL
    
        -   Use event streaming systems like Kafka or AWS Kinesis  and
            batch ETL tools like Airflow for batch jobs.
        -   Preprocess most requested things and cache them to serve
            multiple users rather than processing for each user
        -   delay non essential things like user statistics and logs and
            place them in a queue to be executed when resources are not
            under heavy load
        -   Apache Flink can be used of we want to to streaming rather
            than batch jobs.
    
    4.  Other common Services
    
        1.  Customer / external credential management
        2.  Storage services
        3.  Asynchronous processing
        4.  privacy and compliance teams
        5.  notebook and data analytics /  machine learning services
        6.  fraud detection
        7.  internal search and subproblems
        
        This author seems biased towards Cloud perhaps because of his pedigree
        and history with this companies. I disregard this advice and instead
        choose bare metal for data security, cost efficiency and independence
        from vendor lock in. Shared servers can easily simulate serverless
        systems now, there is no reason to bleed money to cloud providers.
        For more on this see [fn project](https://fnproject.io/), [cloudflare workered](https://blog.cloudflare.com/workerd-open-source-workers-runtime/), [knative](https://knative.dev/docs/). knative is CNCF incubation project

