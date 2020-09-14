# Hosting Dash/Plotly and Bokeh Apps on Compute Studio


Over the past month, I've worked on adding the capability for [Compute Studio (C/S)](https://compute.studio) to host plots made with [Dash/Plotly](https://plotly.com/dash/) and [Bokeh](https://bokeh.org/). This work also paves the way for supporting [Shiny](https://shiny.rstudio.com/) and other plotting libraries. 

For quite some time, I have been interested in adding the capability for C/S to host apps that are built with a wider variety of plotting technologies. I've admired how easy and fast it is to create valuable tools for interacting with data using Bokeh or Dash. This is also a way for C/S to meet develepers where they are by supporting the tools that they use every day. In turn, C/S offers them the capability to share their work with colleagues or the public at large.

As always, publishing apps on C/S is free, but either the app developer, app sponsor, or app user must pay for the compute costs. One of the requirements when building this out was to make sure that the apps are only running when they are being used. This resulted in a very cheap way for developers to deploy and share their Bokeh and Dash/Plotly creations.

### Step 1: Run a Dash app in Kubernetes

One of the things that I've learned in the short time that I've been working with Kubernetes is that you should start simple and check your progress often. So before I started trying to automate anything with the Kubernetes Python API, I created a couple basic YAML files describing the `Deployment`, `Service`, and `Ingress` objects needed to serve my example project.

Once I got this to work, I knew that I was going in the right direction and could more confidently write the Python code required to automatically create the necessary Kubernetes objects.

### Step 2: Automate publishing a Dash app in Kubernetes

Next, I translated the YAML files into a new [`Server`](https://github.com/compute-tooling/compute-studio/blob/master/workers/cs_workers/models/clients/server.py) class in the `cs_workers` package to be used to automatically create, delete, and check the status of visualization deployments and services. 

```python
from cs_workers.models.clients import Server

viz = server.Server(
    owner="hdoupe",
    title="myviz",
    tag="v1",
    callable_name="start_dash_server",
    incluster=False,
)

# Create a visualization
viz.create()

# Check visualization status
viz.ready_stats()

# Delete visualization
viz.delete()

```


### Step 3: Embed the Viz!

After starting at the low-level of *just* getting the visualization server running and viewing it in my browser. I needed to see it embedded on a C/S webpage. Beyond just being excited, this was a helpful sanity check to make sure that embedding it as an IFrame was the right approach.

![It works!](https://user-images.githubusercontent.com/9206065/93091821-9f6f6500-f66c-11ea-99ab-882b2abeef72.png)


Next, it was time to share some early progress. I deployed it to the C/S development site and got a big message from my browser saying that I had a problem, specifically an HTTPS problem. For good reason, you can't embed a link on a site with HTTPS turned on if the embedded link does not support HTTPS. So, I set about learning how to add TLS to a service in a Kubernetes cluster.


### Step 4: Enter Traefik

[Traefik](https://docs.traefik.io/) is a routing service that can be deployed several ways including with Docker and Kubernetes. Traefik routes traffic from your domain to your services based on simple rules such as:

```
Host(`viz.compute.studio`) && PathPrefix(`/hdoupe/ccc-widget/dash/`)
```

which corresponds to this URL: `viz.compute.studio/hdoupe/ccc-widget/dash/`.

Further, it's straightforward to integrate with your DNS provider and [Let's Encrypt](https://letsencrypt.org/) to make your routes secure with TLS and HTTPS.

One of the cool things about open-source software is that you can learn a lot just by following a project and seeing how they approach problems. For example, I learned a lot about Traefik by reading about how JupyterHub and Dask-Gateway use it. This was an enormous help when setting Traefik up for this project.

I'll spare the technical details for this section, but the take-away is that the viz was live at `https://viz.compute.studio/hdoupe/ccc-widget/dash/` and rendering correctly as an IFrame on the development server.

### Step 5: Bringing it all together

Now that the cluster is more-or-less ready to start serving visualizations, it's time to bring in the brains of the operation, aka the frontend of C/S. A key design decision for C/S is that the compute cluster is simple but reliable. The frontend is where most data is stored and where higher-level cluster automation logic lives. 

There are three data sources that are important for this project:

- **App data**: This needed to be extended to store how the app should be served: as a viz or as a model that runs simulations?
- **Deployment data**: This stores all data on past, present, and future app deployments. This is the place to go to figure out whether an app has a viz deployment running, how long it has been running, and whether it needs to be taken down.
- **Embed data**: The visualizations are going to be embedded on different sites around the internet. We need to generate a secure endpoint that can be used for these deployments that *only* allows it to be embedded on a specific website. This is done by modifying the [`frame-ancestors`](https:/https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) CSP to match the website where the viz will be embedded.


### Step 6: Spin them up, spin them down

We now have the data we need to answer some key questions, and we have the operations needed to act on these answers. The general lifecycle of a visualization is as follows:

1. **A user goes to the visualization page.**

    *Is there a deployment running?* If not, let's start one up. A new deployment object is created in the front-end's database, and a new visualization deployment is created in the compute cluster. While the deployment is spinning up, a JavaScript function is pinging a REST API endpoint checking the status of the deployment. Once it is ready, the snippet embeds the Iframe in the HTML page. The user can now interact with it. 
    
    Another Javascript function is triggered that ping's the REST API endpoint letting it know the deployment is still in use. The snippet only "phones home" while [`document.hasFocus()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/hasFocus) is True. If the user navigates to another tab, opens another window, or their computer falls asleep, the document loses focus and the page no longer phones home.
    
2. **Another user comes to the visualization page.**

    *Is there a deployment running?* Yes! The timestamp of the deployment's most recent full page load is updated, and the visualization page continues to phone home as long as the user's webpage has focus.
    
    
3. **It's been some time since we've heard from the viz.**

    *Is anyone still using this deployment?* Every 15 minutes a process runs that asks this for every live deployment. If there hasn't been a full page load or ping in over 30 minutes, the deployment is spun down. The frontend does this by doing a `DELETE` request to a REST API endpoint on the compute cluster.
    
These rules will need some tweaking. Bots caused problems early on because their constant pinging of the viz page meant that the deployment was never spun down. This was corrected to some extent by adding the viz pages to the `robots.txt` file. However, there are more measures that can be taken in the future to mitigate this.

The check for deleting deployments could also be tweaked by setting a different stale-after time limit for the most recent *load* time and most recent *ping* time.
    
    
### Step 7: Security measures

Now that some of the Kubernetes resources are controlled by a REST API endpoint, I needed to make sure that not just anyone can manipulate the compute cluster. After some research, it seemed like there are two approaches that could be used to authenticate requests to the compute cluster. One is [OAuth 2.0](https://oauth.net/2/), and the other is to use [JSON Web Tokens (JWT)](https://tools.ietf.org/html/rfc7519). I noticed that [JupyterHub uses OAuth](https://github.com/jupyterhub/jupyterhub/issues/3155), but for our situation, adding OAuth seemed like it would add a great deal of complexity to the architecture of C/S since it required an external authentication service. In the end, the simplest way forward was to use a JWT approach where the frontend and compute cluster use a securely stored secret to encode and decode JWT's. Then, when the cluster receives requests, it ensures that it can decode the JWT token in the request header with the shared JWT secret.


### Step 8: Put it in production

Time to deploy! The capability to host Bokeh and Dash/Plotly apps has been live for about 10 days now. Deployments are being spun up and down quite smoothly. This isn't to say that there haven't been any bugs. However, the primary components of serving apps to users and spinning them up and down quickly have worked as expected. 

Here are the numbers for the first Dash app published on Compute Studio: It's been live for 10 days, has been deployed 54 times and has been running for about 25% of the time during this period. On average, each deployment is live for about an hour.


## Wrapping up

Over the past few months, I have spent a most of my time improving Compute Studio's infrastructure to be more scalable, more maintainable, and cheaper. Each of these projects has pushed me to learn a different part of the Kubernetes API. By the end of each project, I feel like I have gained at least twice as much Kubernetes and general infrastructure knowledge as I had at the beginning. It's been an immensely rewarding (and at times frustrating) experience.

This project had two major pain points and I hope that others learn from them:
1. Before trying Traefik, I tried to use [cert-manager](https://cert-manager.io/docs/) and could not get it to work. I spent two days on it, debugging countless errors and bugging my friend who does this stuff full-time. Finally, I swapped over to Traefik, and within a couple hours, I had it working. Further, the routing support wound up being extremely useful for routing requests to the right viz deployment.
2. The Kubernetes Python client is great if you know what you are looking for. I needed to manage Traefik's [`IngressRoute`](https://docs.traefik.io/providers/kubernetes-crd/) object which is a [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/). You are looking for the [`CustomObjectsApi`](https://github.com/kubernetes-client/python/blob/master/kubernetes/docs/CustomObjectsApi.md), and here's an example for how to use it with Traefik's `IngressRoute`: [`ingressroute.py`](https://github.com/compute-tooling/compute-studio/blob/1.7.0/workers/cs_workers/ingressroute.py).

Thanks for reading! As always, I'd love to hear your feedback either about the visualizations or the engineering behind them. Please feel welcome to email me at hank at compute.studio, join our [community chat](https://app.element.io/#/room/#compute-studio:matrix.org), or open an issue at the [compute-studio repo](https://github.com/compute-tooling/compute-studio/issues). 

