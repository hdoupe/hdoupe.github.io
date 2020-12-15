# Compute Studio 1.8.0

**Highlights**

- Simplified and streamlined app publishing process.
- Private apps that are only available to you and your team.
- Simulations are public by default.
- Infrastructure updates producing a snappier site, silky-smooth releases, and support for running your own C/S compute cluster.
- Revamped C/S Pro subscription for unlimited private simulations and private apps at $9/month.

Compute Studio 1.8.0 brings something for everyone in the community. For all of our existing users, you have been upgraded to C/S Pro for a free three month trial. You have the option to [opt-in to continuing the subscription](https://compute.studio/billing/upgrade/monthly/aftertrial/) anytime. Users who are interested in viewing public content on Compute Studio will be pleased to hear that simulations are now public by default. We have big plans for a C/S public activity log, but the bare-bones version of it is available at [compute.studio/log/](https://compute.studio/log/). App developers can now publish private apps. This allows app developers to not only create apps with sensitive data sources but also have sensitive data in their outputs. The app publishing flow has also been streamlined significantly, making it much easier to publish new apps. Bokeh and Dash apps are now easy to publish on C/S, with new straightforward docs, available from the [C/S publish page](https://docs.compute.studio/publish/data-viz/guide.html).

Of course, I snuck in some infrastructure features to make my life as a C/S developer better (and in turn build a better website for you). Updates to the site are now deployed automatically. This makes it easy to release new features and bug fixes. C/S has out-grown Heroku and is now hosted with Kubernetes, a deployment platform, on Google Cloud. Finally, I made significant progress on my pet project of letting anyone who’s interested run their own compute clusters. This is a cool feature if you have sensitive data that you *must* keep on your own servers. Or, if you are like me, you just have a raspberry pi laying around that you want to put to good use.

If you enjoy using Compute Studio and you want to contribute to its development, please check out our [C/S Pro subscription](https://compute.studio/billing/upgrade/monthly/). It’s $9/month and goes towards paying for servers and my rent.

Thanks for checking out the release notes and please let me know what you think!