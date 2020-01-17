# Compute Studio 1.0



Highlights:

- Simulations are documents that can be titled and annotated.
- Slimmed down outputs page.
- Streamlined authentication process.

## Features overview:

Compute Studio 1.0 transforms simulations into documents that you can title, annotate, and share. Simulation creators now have the ability to provide the necessary context to their simulations that is needed to make them shareable products. Like documents, simulation configurations can change over time and be modified by multiple authors, and the new history feature will make it easy to look back and see what has changed and who made those changes. Finally, simulations can be made public or private, giving their creators the power to choose when they are ready to share their simulation with the public. During this process, we streamlined the sign in and sign up flow by further integrating it into the simulation page, and we slimmed down our outputs page.  Those of us working on the Compute Studio project are beyond excited to add these features and to continue to build on top of them. 

If you have any questions or feedback about this release or C/S in general, please don't hesitate to send me an [email](mailto:hank@compute.studio) or [open an issue](https://github.com/compute-tooling/compute-studio/issues/new) in the C/S code repository.

Thanks to Matt Jensen for designing the new simulation page. Also, thanks to Peter Metz for giving feedback on this set of features at every step along the way, including the morning after they were shipped. Finally, thanks to the C/S publishers for updating their models to the new outputs API.

## Engineering process: 

In order to build the simulation document, we needed to transform our server-rendered frontend into a cutting edge client-side application that could quickly respond to users and manage user and simulation state. The first problem was that the C/S inputs and outputs pages are really heavy, with the Tax-Brain inputs page rendering over 400 input fields and the outputs page rendering dozens of detailed HTML tables and several interactive plots. Last July, I re-wrote the inputs form in JavaScript, using the [React UI](https://reactjs.org/) framework and the [Formik](https://jaredpalmer.com/formik/) form library. This resulted in an inputs page that loaded quickly, responded immediately to input, and easily handled asynchronous validation with the models in the compute cluster. This new inputs page went live in [C/S 0.12.0](https://hankdoupe.com/posts/COMP-0.12.0.html).

Next up, we combined the outputs and inputs pages, and in the process, we made the outputs render just as quickly as the inputs page by pre-rendering the results, taking screen-shots of them, and rendering those screen-shots instantly while the outputs loaded asynchronously. The results are rendered in a headless chromium browser using [Pyppeteer](https://github.com/miyakogi/pyppeteer), a Python port of [Puppeteer](https://github.com/puppeteer/puppeteer), to render each result and take a screen shot of the HTML node that the result is encompassed in. This approach was simple and worked surprisingly well, delivering tight screen-shots without much unused white space surrounding them. The hardest part of this step was that all of the existing interactive Bokeh plots on Compute Studio were in the wrong format.  We'd stored them in an HTML format that was incompatible with the JSON format required by Bokeh's JavaScript API. However, [thanks in large part to Bryan on the Bokeh team](https://discourse.bokeh.org/t/migrating-from-components-to-json-items/4104/4), I developed a series of regular expressions to convert our existing HTML data into the JSON format that we needed, and it *somehow* worked flawlessly:

```python
# don't try this at home
exp = r'{"roots":.+"version":"[0-9].[0-9].[0-9]"}'
res = re.search(exp, output["data"]["javascript"])
unescaped = (
    res.group(0)
    .replace("&gt;", ">")
    .replace("&lt;", "<")
    .replace("&amp;gt;", "&gt;")
    .replace("&amp;lt;", "&lt;")
    .replace("\\\\n", "\\n")
    .replace('\\\\"', '\\"')
)
parsed = json.loads(unescaped)
root_id = parsed["roots"]["root_ids"][0]
output["data"] = {"target_id": None, "root_id": root_id, "doc": parsed}
```



The final step was to turn our *single page simulation* into a *simulation document* by adding a title and an editor to write the description. We chose the [Slate editor framework](https://docs.slatejs.org/) so that we could have the ability to customize our editor to our needs such as potentially linking the editor to comments on parameter sections. I adapted the [Rich Text](https://www.slatejs.org/examples/richtext) and [Link](https://www.slatejs.org/examples/links) examples in their documentation to our needs, and now we have an easy-to-use editor. I initially looked into using a Markdown editor with a preview button but this wasn't a good fit for our use case since our users aren't likely to be familiar with Markdown and would be more familiar with a WYSIWYG experience from their editor.

This release streamlines Compute Studio's authentication so that users can sign up at any point in the process of viewing or creating a simulation without losing any of their data on the page. [C/S 0.14.0](https://hankdoupe.com/posts/Compute-Studio-0.13.0-0.14.0.html) added a sign-in/sign-up window to the run simulation dialog; however, we needed a solution for users who would want to write a title or some notes for their simulation before they clicked "Run". The solution was to use [React Portal](https://reactjs.org/docs/portals.html) to hook onto the existing "Sign in" / "Sign up" buttons and replace them with their React twins allowing the user to sign in or sign up through a dialog window. On completion, the application state is updated and either a new simulation object is created for them in the C/S database or they get write access to the simulation they are viewing if they are owners of it or the ability to "fork" it if they are not.

Compute Studio 1.0 is such a massive release that all of the new features can't be covered in a single blog post. I could go on for thousands and thousands of words about converting Compute Studio to TypeScript, a statically typed superset of JavaScript, which has vastly improved the frontend developer experience for Compute Studio and made it easy and fast to refactor the code for this release several times.

