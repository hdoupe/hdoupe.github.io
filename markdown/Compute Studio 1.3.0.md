​
# Compute Studio 1.3.0

Highlights:

- Add coathors to your simulation.
- Share your public simulations for free with other users on Compute Studio.
- [Upgrade](https://compute.studio/billing/upgrade/) to share your private simulations on Compute Studio and support Compute Studio’s development.

## Features Overview

Compute Studio 1.3.0 adds new collaboration features for our Compute Studio  users! Now you can easily share your public and private simulations with other users on Compute Studio, and you can now recognize collaborators  as coauthors. While there are no limits to collaboration on *public* simulations, you will need to upgrade to Compute Studio Plus or Pro to have access to these collaboration features on *private* simulations. Upgrading is a great way to share your work privately with colleagues or clients.

This is just the start of the collaboration suite that we are building for  Compute Studio. Please let us know how these features suit your work,  and what else would help your collaboration with colleagues, clients,  friends, and followers.  This is an exciting release for the team, and  finally offers the collaboration features that we’ve been planning for  months. I’m very eager to hear your feedback.

Finally, if you’re a C/S user, I’d like to encourage you to check out [our new subscriptions](https://compute.studio/billing/upgrade/). Buying a subscription unlocks private collaboration for you *and* funds Compute Studio’s developers (like me) and the servers on which it runs.

![img](https://user-images.githubusercontent.com/9206065/77544377-4f62d300-6e7f-11ea-8c18-8fbe93db3e24.gif)

## Engineering Process:

Thanks to the excellent tooling in the open-source ecosystem and [Stripe’s easy-to-use API](https://stripe.com), the biggest challenge of creating collaboration features and multi-tier subscriptions was getting the design right. Like many other design  decisions that we’ve had to make for Compute Studio, we looked to GitHub to see how they designed access levels for GitHub repositories.  Essentially, GitHub lets you assign roles like “Read”, “Write”, and  “Admin” for your repositories. The rules are:

- Users can only have one of these roles at a time.
- Each role is granted a set of permissions.
- Each role up the hierarchy inherits permissions from roles that are lower in the hierarchy.

If this was good enough for GitHub, then we decided it was good enough for us. Thanks to [`django-guardian`](https://github.com/django-guardian/django-guardian) the implementation process was easy once we decided on this role-based framework. `django-guardian` provides object-based (row-level) permissions for Django database models. Now, the owner of each simulation is assigned the `admin` role. The `admin` role allows you to edit the title, description, or inputs and assign  roles to other users. For example, a simulation owner can assign the `read` role to another user on a private simulation, making it easy for to get feedback before sharing the simulation more broadly. A simulation `admin` can also invite coauthors to their simulation. Note that coauthors are automatically granted the `read` role.

During the development of these features a simple recipe emerged for adding  role-based access to different aspects of a simulation:

1. Isolate an action  that can be taken on a simulation, such as being able to view it, and  assign a permission for this action to a role. For example, assign  “viewing a simulation” to the `read` role.
2. Before letting a user perform this action, check if their role permits it. Do they at least have the `read` role? If not, throw an error.
3. Write a test that  ensures that users who are supposed to have the permission have it and  can perform the action like being able to view a simulation.
4. Write another test that ensures that users who are **not** supposed to have the permission don’t have it and cannot perform the action.

While Compute Studio is an open-source project and offers its core services for free, we have also adopted a market-based approach to sustaining and  advancing our work. Like many other services such as GitHub, our new  paid subscription plans on Compute Studio give you access to different  levels of private collaboration. Our first offering provides four plans: C/S Free, C/S Plus, C/S Pro, and a forthcoming C/S Teams. C/S Free lets you do everything that you’ve always been able to do like run  simulations, make them private or public, and now, add an unlimited  number of collaborators on *public* simulations. However, by upgrading to C/S Plus, you can add one additional collaborator on *private* simulations, and by upgrading to C/S Pro, you can add an unlimited number of collaborators on *private* simulations. C/S Teams will make it easier for organizations to  administer teams on Compute Studio. If you want to support Compute  Studio’s development, [upgrading](https://compute.studio/billing/upgrade/) to a paid plans, sending us your feedback, or sharing Compute Studio with others are the absolute best ways to do it.

On the implementation side, every time a user tries to add a collaborator on a *private* simulation the code checks if your plan allows you to add an additional collaborator. If the plan does not, then we gently nudge you to upgrade your subscription.

For everyone who has made it this far, I’d love to hear your feedback. What do you think of Compute Studio’s role-based permissions? Are you a C/S  user or are you writing your own role-based permissions for another  project? Let me know what you think at hank@compute.studio.

 