FORMAT: 1A
HOST: https://api.gemnasium.com/

# Gemnasium

Gemnasium API lets you interract with your Gemnasium projects and more.

This API is still in development. Most of the API endpoints are already available but the endpoints having the **[NOT AVAILABLE]** tag will be released later on.

## API versioning

Gemnasium API is versioned by appending the version number at the end of the root URL (ie: https://api.gemnasium.com/v1).

The current stable version of the API is `v1`.

## Rate limiting

There is currently no rate limit on the Gemnasium API. We may apply a rate limit in the future.

# Group Authentication

Each API request should be authenticated using HTTP Basic authentication with 'X' as the login, and the Gemnasium user API key as the password.

The user API key can be found at the bottom of the [user settings page](https://gemnasium.com/settings/) or can be requested over the API.

## POST /login

Authenticate a user and return her/his user API key

+ Request (application/json)

            { "email": "email@example.com", "password": "password" }

+ Response 200 (application/json)

            { "api_token": "authenticated_user_api_token" }

# Group Projects

A Gemnasium project has dependency files and it keeps track of the dependencies.

Each project is identified by a unique slug. The slug can either be:

* the slug of a GitHub hosted projects, like `octokit/go-octokit` or `rails/rails`
* a random string when the project is not on GitHub

The project slug can be found on the project settings page.

## GET /projects

List the projects the authenticated user can access to

Projects are grouped by owner accounts. The projects owned by the authenticated appear as they belong to the "owned" account.

+ Response 200 (application/json)

        {
            "owned": [
                {
                    "slug": "randomly_generated_slug", "name": "project-1", "description" : "project-1 description", "origin": "gitlab", "private": true, "color": "green", "monitored": true, "unmonitored_reason": ""
                }, {
                    "slug": "gemnasium/project-2", "name": "project-2", "description" : "project-2 description", "origin" : "github", "private": false, "color": "yellow", "monitored": true, "unmonitored_reason": ""
                }
            ],
            "John Doe": [
                {
                    "slug": "randomly_generated_slug", "name": "project-1", "description" : "project-1 description", "origin": "gitlab", "private": true, "color": "green", "monitored": true, "unmonitored_reason": ""
                }, {
                    "slug": "gemnasium/project-2", "name": "project-2", "description" : "project-2 description", "origin" : "github", "private": false, "color": "yellow", "monitored": true, "unmonitored_reason": ""
                }
            ]
        }

## GET /projects/{slug}
Return the details of the a project.

+ Response 200 (application/json)

        {
            "slug": "gemnasium/API_project",
            "name: "API_project",
            "description": "This is a brief description of a project on Gemnasium"
            "origin": "github",
            "private": false,
            "color": "green",
            "monitored": true,
            "unmonitored_reason": ""
        }

## POST /projects

Create a project

+ Parameters
    + name (string) ... Name
    + description (optional, string) ... A short description
    + origin (optional, string) ... Where the project is hosted. It is possible to create `offline` or `gitlab` projects using the API. Other origins will be rejected. Default orgin is `offline`.

+ Request (application/json)

        { "name": "API_project", "description": "", origin: "gitlab" }

+ Response 201 (application/json)

        { "name": "API_project", "slug": "randomly_generated_slug", "remaining_slot_count": 100 }

## PUT /projects/{slug}
**[NOT AVAILABLE]**

Update a project

It is possible to update the name, the description or the monitored attribute. All parameters are optional.

+ Parameters
    + name (optional, string) ... Name
    + description (optional, string) ... A short description
    + monitored (optional, boolean) ... Whether the project is watched by the user. The dependency files are automatically synchronized if the project is monitored and hosted on GitHub.
    
+ Request (application/json)

        {
            "name": "New_project_name",
            "description": "New description",
            "monitored": false
        }

+ Response 200

        {
            "id": 1,
            "name: "New_project_name",
            "description": "New description"
            "origin": "offline",
            "color": "green",
            "monitored": false
        }

## DELETE /projects/{slug}
**[NOT AVAILABLE]**

Destroy a project

Warning! There is no way to restore a project once it has been destroyed!

+ Response 204

## POST /projects/{slug}/sync
**[NOT AVAILABLE]**

Start project synchronization

The project synchronization proceeds in two steps:

* it updates the dependency files when this is possible (GitHub hosted project)
* it updates the dependencies according to the dependency files

+ Response 204

# Group Dependency files

## GET /project/{slug}/dependency_files

List the dependency files of a project

+ Response 200 (application/json)

        [
            { "id": "1", "name": "Gemfile", "content": "Gemfile content" },
            { "id": "2", "name": "Gemfile.lock", "content": "Gemfile.lock content" }
        ]

## POST /project/{slug}/dependency_files
**[NOT AVAILABLE]**

Add dependency files to an existing project, or update existing ones

If a dependency file with the same name already exists, its content will be updated.

The request contains an array of dependency files where each file has a `name` and a `content`. The parsing of a dependency file depends on its name (ie. the `Gemfile` name implies this is a Ruby Bundler dependency file, etc.).

+ Request (application/json)

        [
            { "name": "Gemfile", "content": "Gemfile content" },
            { "name": "Gemfile.lock", "content": "Gemfile.lock content" },
        ]

+ Response 200 (application/json)

        {
            "added": ["Gemfile"],
            "updated": ["Gemfile.lock"],
            "rejected": ["picture.jpg"]
        }

## DELETE /projects/{slug}/dependency_files/{dependency_file-id}
**[NOT AVAILABLE]**

Delete a dependency file

+ Response 204

# Group Dependencies

## GET /projects/{slug}/dependencies

List the **first level** dependencies of the requested project.

+ Response 200 (application/json)

        [
            {
                "package": {
                    "name": "gemnasium-gem",
                    "slug": "gems/gemnasium-gem",
                    "type": "rubygem"
                },
                "requirement": ">=1.0.0",
                "locked": "2.0.0",
                "type": "development",
                "first_level": true,
                "color": "green"
            },
            {
                "package": {
                    "name": "rails",
                    "slug": "gems/rails",
                    "type": "rubygem"
                },
                "requirement": "=3.1.12",
                "locked": "3.1.12",
                "type": "runtime",
                "first_level": true,
                "color": "red"
            }
        ] 

# Group Dependency alerts
## GET /projects/{slug}/alerts

List the dependency alerts the given project is affected by

+ Response 200 (application/json)

        [
            {
                "id": 1,
                "advisory": {
                    "id": 1,
                    "title": "XSS vulnerability"
                },
                "created_at": "2014-05-07T09:59:53.738404Z",
                "status": "acknowledged"
            },
            {
                "id": 2,
                "advisory": {
                    "id": 2,
                    "title": "DOS vulnerability"
                },
                "created_at": "2014-05-07T09:59:53.738404Z",
                "status": "closed"
            }
        ]

## PUT /projects/{slug}/alerts/{alert-id}
**[NOT AVAILABLE]**

Update the alert status

The new status can either be:

* `open` if the issue still applies and the user wants to be notified about it
* `acknowledged` if the issue applies but the user does not want to be notified
* `closed` if the issue does not apply to the project or if it has been worked around

+ Request (application/json)

        {
            "status": "closed"
        }

+ Response 200

        {
            "id": 1,
            "status": "closed"
        }

# Group Live evaluation

**This endpoint is reserved for Gold members only**

The Gemnasium API is able to parse a set of dependency files without creating any project.

The dependency file evaluation is asynchronous and proceeds in two steps:

* post the dependency files and get a job id back
* poll the job till it has a result

The result of the job is similar to the description of a project (ie. it has colors, dependencies and alerts).

**The files content must be base64 encoded.**

## POST /evaluate

Process a set of dependency files and return a job

+ Request (application/json)

        {
            "dependency_files": [
                {
                    "name": "Gemfile",
                    "content": "base64 Gemfile content"
                },
                {
                    "name": "Gemfile.lock",
                    "content": "base64 Gemfile.lock content"
                }
            ]
        }

+ Response 200 (application/json)

        { "job_id": "123" }

## GET /evaluate/{job-id}

Get the status of a job, and possibly its result

The job status can either be:

* `queued` if the job is enqueued but not started yet
* `working` if the job is under progress
* `completed` when the result of the job are available
* `failed` if the job has failed
* `killed` if the job has been manually killed

When completed, the job has a result that includes:

* an overall color for each environment type (runtime and development)
* a list of dependencies; each dependency has a color, a package and a list of advisories

+ Response 200 (application/json)

        {
            "status": "completed",
            "result": {
                "runtime_status": "red",
                "development_status": "yellow",
                "dependencies": [
                    {
                        "package": {
                            "name": "actionmailer",
                            "type": "Rubygem",
                            "slug": "gems/actionmailer"
                        },
                        "requirement": "= 4.0.4",
                        "locked_version": "4.0.4",
                        "type": "runtime",
                        "first_level": false,
                        "color": "red",
                        "advisories": [ { "id": 1 }, { "id": 2 }]
                    },
                    {
                        "package": {
                            "name": "actionpack",
                            "type": "Rubygem",
                            "slug": "gems/actionpack"
                        },
                        "requirement": "= 4.0.4",
                        "locked_version": "4.0.4",
                        "type": "runtime",
                        "first_level": false,
                        "color": "yellow",
                        "advisories": []
                    },
                    {
                        "package": {
                            "name": "active_link_to",
                            "type": "Rubygem",
                            "slug": "gems/action_link_to"
                        },
                        "requirement": ">= 0",
                        "locked_version": "1.0.2",
                        "type": "development",
                        "first_level": true,
                        "color": "green",
                        "advisories": []
                    }
                ]
            }

# Group Packages

A package is a reusable library a project depends on.

Each package has:

* a `name` relative to the package type
* a `type` that relates to a package repository
* a unique `slug` that embeds the package type and the package name

Supported package types:

* rubygem
* packagist
* pypi
* npm

## GET /{package-type}
**[NOT AVAILABLE]**

List the packages of a give type

Supported values for package types are:

* `gems` for Rubygems hosted on [Rubygems.org](https://rubygems.org/)
* `npms` for Node.js packages registered on [npmjs.org](https://www.npmjs.org/)
* `pypi` for Python packages hosted on [Pypi](https://pypi.python.org/pypi)
* `packagist` for PHP Composer packages hosted on [Packagist.org](https://packagist.org/)

+ Parameters
    + package-type (string) ... Type of package

+ Response 200 (application/json)

        [
            { "name": "rails",  "type": "rubygem", "slug": "gems/rails", "description": "Rails description", "latest": "4.0.0" },
            { "name": "rspec",  "type": "rubygem", "slug": "gems/rspec", "description": "Rspec description", "latest": "2.14.1" }
        ]

## GET /{package-slug}
**[NOT AVAILABLE]**

Return the details of a package

+ Response 200 (application/json)

        {
            "name": "rails",
            "type": "rubygem",
            "slug": "gems/rails", 
            "description": "Ruby On Rails Framework",
            "versions": ["0.0.1", "1.0.0", "2.0.1"]
        }

## GET /{package-slug}/versions/{version-number}
**[NOT AVAILABLE]**

Get the detail of a package version

+ Parameters
    + package-slug (string) ... Slug of a package (cf: package-slug definition above)
    + version-number (string) ... Published version of a package

+ Response 200 (application/json)

        {
            "package": {
                "name": "rails",
                "type": "rubygem",
                "slug": "gems/rails", 
                "description": "Ruby On Rails Framework"
            },
            "version": "3.12.1", 
            "changelog": "Changelog of requested version",
            "dependencies": [
                { "name": "actionmailer", "requirement": "= 3.12.1", "color": "yellow" },
                { "name": "actionsupport", "requirement": "= 3.12.1", "color": "red" },
            ]
        }

# Group Security advisories

## GET /advisories/{advisory-id}
**[NOT AVAILABLE]**

Return the details of a security advisory

+ Response 200 (application/json)

        {
            "title": "XSS vulnerability",
            "identifier": "CVE-20130221",
            "description": "Advisory description",
            "solution": "Advisory solution",
            "affected_versions": "1.4.x",
            "package": {
                "name": "rails",
                "type": "rubygem",
                "slug": "gems/rails"
            },
            "cured_versions": "2.0.0",
            "credits": "",
            "links": ["http://google.com"]
        }