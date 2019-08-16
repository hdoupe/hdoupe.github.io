# COMP 0.12.0

[Release 0.12.0][1] brings a re-write of the inputs page and significant improvements in the compute architecture. The re-write scrapped the inputs form created from a combination of Python, HTML, Django templating, and JQuery in favor of a JavaScript based frontend written with the ReactJS UI framework. Writing the form completely in JavaScript put all of the form creation, validation, and submission logic in one place, saving time spent hunting down UI bugs in different corners of the code base. A JavaScript based frontend made it easy to perform basic range and type validation on the fly, giving users instantaneous feedback when they enter an invalid value. Most importantly, it makes it easy to perform asynchronous form submission.

The compute architecture needed to be updated to handle asynchronous inputs validation. Previously, the user submitted data for validation to the COMP web server, it did some initial form validation, and then the data was sent through the model's custom validation code in the compute cluster. All of this occurred during a single, blocking request while the user waited on the results from validation. If the number of inputs to be validated was large or if there was any extra delay at all, it was likely that the request would time out and the application would crash. 

COMP 0.12.0 made the validation step asynchronous. Now, the inputs are submitted for validation to the compute cluster, the job is kicked off to a Celery worker, and the task ID is returned to the web server. While the validation task is running, the client, either the React frontend or a REST API user, polls the inputs at until validation is complete. The compute cluster pushes the validation results back to the web server where the inputs object is updated and a new simulation is started in the compute cluster. The React client then redirects to the outputs page and waits on the simulation to complete.

After a day of running in production, COMP 0.12.0 has held up very well. The page load times are down, and there have not been any errors from request timeouts. That is not to say that there have been no errors at all. There have been some issues with connectivity to the worker cluster, but that is a separate issue. Not only did COMP 0.12.0 resolve a major error that affected many users on COMPmodels.org, the introduction of React on the inputs page opens up a path for real innovation in the COMP's UI.



[1]: https://github.com/comp-org/comp-ce/releases/tag/0.12.0

