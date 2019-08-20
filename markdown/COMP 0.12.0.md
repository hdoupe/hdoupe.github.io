# COMP 0.12.0

Highlights:

- Improved COMP developer experience
- Immediate basic inputs validation
- Async comprehensive validation, fixing the largest source of simulation failures
- Path to UI innovation and customization

[Release 0.12.0][1] includes a re-write of the inputs page and significant improvements in the compute architecture. The re-write scraps the old inputs form created from a combination of Python, HTML, Django templating, and JQuery in favor of a JavaScript based frontend written with the [ReactJS UI][2] framework. This puts all of the logic for displaying the model’s inputs page and validating inputs in one place, vastly improving the COMP developer experience. A JavaScript based frontend also makes it easy to perform basic range and type validation on the fly, giving users immediate feedback when they enter invalid values. The compute architecture also needed to be updated to handle comprehensive inputs validation asynchronously and put an end to a significant source of simulation failures. Finally, the introduction of ReactJS on the inputs page opens up a path for real innovation in COMP's UI. 

Immediate basic input validation is run after each input is entered by the user, and an error message is shown under the input if it is invalid. This immediate validation is a first draft at a JavaScript implementation of the [ParamTools][3] validation spec and is tentatively named ParamTools.js. Currently, ParamTools.js only implements validation to determine whether the value is of the correct type and is in the correct range. In the future, it will be able to do more complex validation like determining whether the value of a parameter is greater than the value of another parameter. 

This release makes the comprehensive validation step asynchronous. Previously, when the user submitted a simulation, the inputs were also sent for validation to the COMP web server, it did some initial form validation, and then the data was sent through the model's custom validation code in the compute cluster for comprehensive inputs validation. All of this occurred during a single, blocking request while the user waited on the results from validation. If the number of inputs to be validated was large or if there was any extra delay at all, it was likely that the request would time out and the application would crash. This was our largest source of failed simulations by far. Now, COMP submits the inputs to the compute cluster for validation, the job is kicked off to a Celery worker, and the task ID is returned to the web server. While the validation task is running, the client, either the React frontend or a REST API user, polls the inputs  until validation is complete. The compute cluster pushes the validation results back to the web server where the inputs object is updated and a new simulation is started in the compute cluster. The ReactJS client then redirects to the outputs page and waits on the simulation to complete.

ReactJS is a powerful UI framework that we can leverage to build more creative input elements, empower users to create more complex simulations, and provide a more intuitive experience for new users. COMP aims to provide a standard process for adding new types of input elements, which will allow community members to contribute their own custom inputs to COMP’s library of input elements. COMP’s experience with outsourcing the creation of outputs to the model publishers has shown that motivated publishers are capable of producing far more interesting and creative ways for users to interact with their models than we could ever hope to do. I’m excited at the prospect of opening the inputs page to this same type of innovation.

After a few days of running in production, COMP 0.12.0 has held up very well. The page load times are down, and there have not been any errors from request timeouts. That is not to say that there have been no errors at all. There have been some issues with connectivity to the worker cluster, but that is a separate issue that we are resolving rapidly. In addition, due to a misunderstanding with the Google Cloud Platform, the compute cluster was shut down for a few hours on Monday, August 19th. However, everything is now operating as expected.

Thanks to Matt Jensen and Peter Metz for your contributions to this release!


[1]: https://github.com/comp-org/comp-ce/releases/tag/0.12.0
[2]: https://reactjs.org/
[3]: https://github.com/pslmodels/paramtools
