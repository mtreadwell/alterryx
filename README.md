alterryx
========

[![Travis-CI Build
Status](https://travis-ci.org/mtreadwell/alterryx.svg?branch=master)](https://travis-ci.org/mtreadwell/alterryx)

`alterryx` provides access to each of the Alteryx Gallery API endpoints.
With `alterryx` users can:

-   Retrieve information on Alteryx Gallery resouces like apps,
    workflows, and macros

-   Queue jobs for applications

-   Retrieve the status and output of jobs

In order to use this package, you will need to have [a private gallery
API key and
secret](https://community.alteryx.com/t5/Alteryx-Knowledge-Base/Private-Gallery-API-Key-and-Secret/ta-p/22009)

Running Jobs in the Gallery
===========================

Setup
-----

Once you have obtained your API key and secret set them as global
options. Though it is not necessary, it will save you typing later if
you also set your Alteryx Gallery URL as an option.

    alteryx_api_key <- "ALTERYX_API_KEY"
    alteryx_secret_key <- "ALTERYX_API_SECRET"
    alteryx_gallery <- "https://yourgallery.com/gallery"

    options(alteryx_api_key = alteryx_api_key)
    options(alteryx_secret_key = alteryx_secret_key)
    options(alteryx_gallery = alteryx_gallery)

Your Apps
---------

Access to Alteryx Gallery resources like workflows, applications, and
macros are managed through studios. Your account has a subscription id
which determines what you can access. For the purpose of this package,
when you see the term 'resource' that can refer to anything published to
the Alteryx Gallery like workflows, applications, and macros. When you
see 'application' or 'app' it specifically refers to files with the
extension *.yxwz* that are published to your Gallery.

The resources you can access are obtained using `get_app`.

### Search Apps

    subscription <- get_app()

You now have a `list` containing all of the resources you can access. If
you are a power user, this is probably going to be a long list. To pare
down the list use the `request_params` parameter.

If you wanted to see only the five most recently uploaded resources, you
can use the `limit` and `sortField` parameters.

    request_params <- list(
      limit = "5",
      sortField = "uploaddate"
    )

    subscription <- get_app(request_params)

### Non-applications

There is a reason to differentiate between 'resources' and 'apps'.
`get_app` will return all resources that you can access via your
subscription that match the search parameters. However, only 'apps' can
be used with the rest of the API functions. To make sure that `get_app`
only returns apps, use `packageType = "0"` as a request parameter.

    request_params <- list(
      packageType = "0",
      limit = "5",
      sortField = "uploaddate"
    )

    subscription <- get_app(request_params)

### Search Apps by Name

If you are looking for a specific app, it might be easiest to simply
search for it by name. Lets say we are looking for an application named
"api\_tester.yxwz".

    request_params <- list(
      packageType = "0",
      search = "api"
    )

    subscriptions <- get_app(request_params)
    app <- subscriptions[[1]]

In this case, the app I was looking for was the first result in the
list.

### Download a Specific App

If you would like to work with the application in Alteryx, you can use
`doanload_app` to download the application as a *.yxzp* file.

    download_app(app)

Queueing a Job
--------------

Now that I have the app I want, I want to queue a job for it. A 'job' is
one run, a single iteration of an app.

### App Questions

Most of the time, applications have questions that must be answered in
order for the app to run. For example, an app that performs trade area
analysis might ask you to specify a radius for the trade area. These
questions are set by the app author when they create the application in
Alteryx Designer. My app, "api\_tester.yxwz" has a single question: "How
long should this application run?"

If you don't have your app questions memorized, use `get_app_questions`.

    questions <- get_app_questions(app)

### Build the Answers

Each question has a *name* and a *value*. In my specific case, the
question name is "runtime" and the default value is "1". Because this
app "api\_tester.yxwz" was built specifically to test this API client,
the "runtime" question will simply determine how long the app will run.

I would like the app to run for "3" minutes. Use `build_answers` to
format the answers.

    name_values <- list(
      name = "runtime",
      value = "3"
    )
    answers <- build_answers(name_values)

If your application has multiple questions, send each as a `list` to
`build_answers`

    name_values1 <- list(
      name = "one",
      value = "1"
    )

    name_values2 <- list(
      name = "two",
      value = "2"
    )

    multiple_answers <- build_answers(name_values1,
                                      name_values2)

### Queue the Job

Once you have the answers to the app questions you can queue the job.

    job <- queue_job(app, answers)

If the server administrator has granted you access to set job priority,
you can add priority using the `priority` parameter. Valid priority
values are "low", "medium", "high", and "critical".

    job <- queue_job(app, answers, priority = "high")

The `job` will begin with status "Queued". Poll the job using `get_job`
to update the job status.

    job <- get_job(job)

Job Output
----------

Most Alteryx jobs contain an output tool that writes data once the job
is complete. Use `get_job_output` to retrieve the results as a
`data.frame`. The result will be a list with one element for each valid
output. 'Valid' in this case means an output from an Alteryx output tool
that can be converted into a `data.frame`.

A job needs to have a "Completed" status before outputs can be
retrieved.

    output <- get_job_output(job)

### Invalid Job Outputs

All outputs cannot be properly converted to a `data.frame`. If your job
contains outputs that cannot be converted, `get_job_output` will issue a
warning by default and skip the 'invalid' outputs.

In order to be properly converted, your output must be written in *csv*
or *yxdb* format from the Alteryx app published to Gallery.

Migrating Apps, Macros, and Workflows
=====================================

Setup
-----

For migration, you will need keys and access to the admin Gallery API.
Non-admin keys will not work for migration and admin keys will not work
for normal API operation.

Once you have obtained your admin API key and secret set them as global
options. Though it is not necessary, it will save you typing later if
you also set your Alteryx Gallery URL as an option.

    alteryx_api_key <- "ALTERYX_API_KEY"
    alteryx_secret_key <- "ALTERYX_API_SECRET"
    alteryx_gallery <- "https://yourgallery.com/gallery"

    options(alteryx_api_key = alteryx_api_key)
    options(alteryx_secret_key = alteryx_secret_key)
    options(alteryx_gallery = alteryx_gallery)

Migration involves movement from one environment to another, so you will
also need access to the admin API for the target environment.

    target_alteryx_api_key <- 'TARGET_ALTERYX_API_KEY'
    target_alteryx_secret_key <- 'TARGET_ALTERYX_API_SECRET'
    target_alteryx_gallery <- "https://your-other-gallery.com/gallery"

    options(target_alteryx_api_key = target_alteryx_api_key)
    options(target_alteryx_secret_key = alteryx_secret_key)
    options(target_alteryx_gallery = target_alteryx_gallery)

*NOTE* You can migrate from and to the same enviornment.

Migratable Apps
---------------

Users and admins can mark all resources on Alteryx Gallery as "ready for
migration." Once marked, admins can get a list of all resources that are
ready. The function returns all apps in the Gallery that are ready for
migration.

    migratable_apps <- get_migratable()

    # Searching for resources marked for migration... 
    # 2 resources found... 
    # 
    #    APP_1.yxwz 
    #    APP_2.yxwz

*Known Issue* `get_migratable` has an option to filter resources by a
list of specific subscriptions (private studios) but the functionality
is not working properly.

    get_migratable(
      subscription = list(
        subscriptionIds = "subscription1,subscription2"
      )
    )

Downloading and Migrating
-------------------------

Once you've obtained a list of apps to migrate, you can migrate them
using `publish()`. The function downloads the app to the given directory
and then publishes to the Gallery using the credentials and information
you set as options.

When publishing, you can provide a form which will determine certain
options for the app in the target environment. If you do not provide a
form, the form will be generated for you using the app information in
the source environment.

-   `name`: Give the app a new name
-   `owner`: Give the app a new owner (must exist in target system)
-   `validate` Perform a validation run prior to publishing?
-   `isPublic` Place the app in the Public Gallery?
-   `sourceId` See below
-   `workerTag` Assigned worker name
-   `canDownload` Can others download the app?

If `sourceId` is left blank, it will create a new copy of the published
app in the target enviornment. If you provide the app ID from the
*source* environment, it will overwrite the appropriate app in the
*target* environment.

    form <- list(
      name = "APP_1.yxwz",
      owner = "yourname@domain.com",
      validate = "false",
      isPublic = "false",
      sourceId = "",
      workerTag = "",
      canDownload = "false"
    )

    publish(
      migratable_app,
      migration_directory = "c:/temp",
      form = form
    )

    # APP_1.yxwz 
    # --------------- 
    #   Downloading workflow... 
    #   Saving workflow... 
    #   Publishing workflow to target environment... 
    #   Done. Publish in target environment as 2acd7c 

Toggle Migration Flag
---------------------

Once the app has been published to the target enviornment, you will
likely want to turn off the "Ready to Migrate" flag in the source
environment.

    toggle_migratable(migratable_app)

Automation
----------

I've provided a convenient wrapper to automate migrating workflows:
`migrate()`. This function identifies all resources ready for migration
and publishes them to the target enviornment. If you provide a form for
migration, it will use the information provided but change the app name
and ID for each item marked for migration.

    migrate(
      form = form,
      migration_directory = "c:/temp"
    )

    # Beginning Migration... 
    # 
    # Searching for resources marked for migration... 
    # 2 resources found... 
    # 
    #    APP_1.yxwz 
    #    APP_2.yxwz 
    # 
    # APP_1.yxwz 
    # --------------- 
    #   Downloading workflow... 
    #   Saving workflow... 
    #   Publishing workflow to target environment... 
    #   Done. Publish in target environment as 2acd7c 
    # 
    # 
    # APP_2.yxwz 
    # --------------- 
    #   Downloading workflow... 
    #   Saving workflow... 
    #   Publishing workflow to target environment... 
    #   Done. Publish in target environment as 2acd97 
    # 
    # Toggling Migrate Flag... 
    # Done
