If you want to build a scalable software-as-a-service product, that's easy to maintain, you need to set it up in a very specific way. Doing that is really important, because if you don't, you're going to run into a lot of problems, your software won't work well, your customers are going be unhappy, and ultimately your product is going to fail.

Today I want to go through the 12 factors that are crucial to developing a modern software-as-a-service product. This video's not Python-specific by the way, you can use any programming language or environment you like. I'm also not going to talk much about software design today, this is going to be a DevOps video. But the concepts are not completely independent. Thinking carefully about software design is going to help you choose the right technologies that will best serve your product. I've written a guide to help you with designing software from scratch in 7 steps. You can get it for free at arjancodes.com/designguide. It's based on my own experience developing at least three different software-as-a-service products over the last five years. You know, why not benefit from my experience to avoid some of the mistakes that I made in the past? If you go to the link below and enter your email address, you'll get it in your inbox right away.

Back in the day, if you bought software, you got a bunch of floppies, a CD-ROM, or a DVD. I remember buying WordPerfect 5.1 on 12 floppies. And then when you're installing it and you're at floppy 10 out 12, of course then that disk is damaged. Ah, the memories. When the Internet took off, companies switched from sending you physical disks to allowing to download the software instead. And now that cloud technology is maturing, we're no longer directly downloading software, but software is delivered as a service instead. You register on a website, and use the software directly as a web application.

The twelve-factor app is a methodology for building software-as-a-service apps in a particular way. Most of the cloud technologies you see today (AWS, Azure, Google Cloud) are all built on the principles of the twelve-factor app.

The overall idea is that you create the app in such as way that you automate setup and deployment, that you're compatible with modern cloud platforms so you don't need your own servers anymore, that development and production environments are as similar as possible, and that you can scale up your app without having to make too many changes. So what are these twelve factors then? Let's dive in.

# I. Codebase: One codebase tracked in revision control, many deploys

The first factor states that you should store the code in a version control system like Git. There should be a one-to-one correlation between your codebase and the app. So if your app is a website, the code that renders that website should be in a single repository. It's possible that your app consists of multiple services: your website (or frontend) server, a backend server, a database server, services for specific things like handling payments, performing analytics, etc. Each of these should be defined in a separate repository, at least the ones you're developing yourself. More about that later.

If your apps and services need to share code (like certain data structures or interfaces), extract these into libraries that you can then include using a dependency manager. If you're building a service with JavaScript or Typescript and Node.js, you can use npm. In Python, you can use Pip, or more advanced tools for managing dependencies like Poetry.

There is one codebase, but there can be multiple deploys. A deploy is a running instance of your app. The production version of your app is one of the deploys, but there can be other versions running as well: a version locally on your machine, a version used for testing, a staging version, and so on.

# II. Dependencies: Explicitly declare and isolate dependencies

In most programming languages, you have a mechanism for handling the dependencies that your application needs. If you're doing Typescript or JavaScript development, then you'll probably use NPM or Yarn. Rust has the Cargo package manager. In Python, that's Pip. A 12 factor app doesn't rely on system-wide installed packages, but it declares all its dependencies using some kind of manifest file. In Typescript and JavaScript, this is the package.json file that contains all the dependencies. If you're using Pip in Python, this is the requirements text file. Because there's only a single codebase that's deployed one or more times to different environments, the specification of these dependencies is also the same for both production and development environments. Though of course, these can be attached to different branches in your Git repository.

A second thing that's important for dealing with dependencies is that there's also a way to isolate dependencies so to ensure that you're not accidentally using dependencies from the surrounding system. In Python, you can use something like virtualenv to isolate your dependencies. If you want to take this a step further, there are overarching dependency management and isolation systems like Poetry (I'll put a link in the description).

But even then, there's another layer of isolation that you can benefit from, and that's the system-wide specification of dependencies and providing an isolated execution environment. The tool for this that everyone uses is obviously Docker. This is a virtualization layer that allows you to specify exactly which operating system and version you'll be using, and install any extra dependencies that you need as well. The Dockerfile is the manifest specifying the execution context and can install dependencies that you need. Docker containers are the virtual machines that provide the isolation.

# III. Config: Store config in the environment

Depending on the deploy of your app, you're going to have many configuration settings. These settings will be different for the different deploys. For example, you production web application will be on a different domain that your staging or development version of the app. And there are other things that might be different:

- Connecting to the development database in stead of the production database
- Credentials for storage locations like Amazon S3 or Google Cloud storage
- Credentials of other services like email clients or other third-party integrations
- You may even have globally enabled or disabled areas of your application depending on the deploy. For example, an admin area might not be visible by default in production, you could have features that you're already building but they're hidden behind a global feature flag, a global maintenance mode that is on or off, logging settings, and so on.

These kinds of settings shouldn't be part of your codebase. If you're for example storing these things as constants in the code, this is a violation of the 12-factor app. A good test to verify that you're setting up configuration correctly is whether your code could be released as open source without giving public access to any credentials. Another test is to verify that if you give access to your repository to let's say an intern, that you can control what the intern has access to. This is especially important because everyone at a software company always makes jokes about interns. If you piss them off too much, they might be out for revenge and you don't want them to have access to the production database at that point, do you?

The most common place to store these settings is by using environment variables. They're language and OS agnostic. Most cloud providers offer the possibility to define environment variables as part of your virtual server. For example, if you launch a Google Cloud function, you can add environment variables which are then available to the app at startup.

You can also use a tool like dotenv to group your variables into a single dot env file. This works well for local development. Whenever a new developer wants to start working on the code, you can provide that developer a dot env file containing all the necessary credentials. But you have to make sure that you don't accidentally commit the .env file to your repository.

For some configuration settings it might make sense to make them part of the code. For example the names of the routes in a web app, file size limits for uploads, default prefixes for database ids, etc. These things are probably not going to be different for different deploys, so you can keep them in the code itself. As soon as they change depending on the deploy though, move them out into environment variables.

# IV. Backing services: Treat backing services as attached resources

Unless you're developing a simple standalone desktop application, your app will have a variety of backend services that it uses: a database, an email sending service, a payment provider, logging service, storage services (like Amazon S3), other APIs, your own API that forms a layer on top of these things, and so on.

The whole idea is that your code in principle shouldn't make a distinction between local and third-party services. Services communicate with each other via URLs or other locators (like a database connection string). Furthermore, you should be able to swap out services with other services without having to change to code. You may have to change configuration settings though but they shouldn't be stored in the code.

I have to admit though, this is not the reality in general. We had to switch between storage providers a couple of years ago - we went from Amazon S3 to Google Cloud Storage. We had to change quite a bit in the code, even if we had built a layer between the low-level storage access and the rest of our backend.

# V. Strictly separate build and run stages

Deployment of your code should happen in three independent stages: build, release and run. In the build stage you convert your code into an executable bundle. It contains any dependencies and other assets that the code needs. If you use Docker, this would be the Docker image.

You then have a release stage that combines any configuration settings with the build output. If you're using something like Kubernetes this is the definition of a deployment that links a Docker image with other settings like environment variables, information about the number of replicas and so on.

Finally, there's the run stage that actually loads the Docker image and starts the service according to the release configuration.

Every release should have a unique id: can be a timestamp, or a version number. Once you create a release, it can't be mutated. Any change must create a new release.

Separating build, release and run stages means for example that you shouldn't change code in a service that's currently running. You should change it in your code base, then build, release and run a new version.

You can initiate a build and release when you're changing code. For example, we have coupled this with pushing our code to a develop, staging and production branch which automatically launches a build. You can do this really easily with GitHub Actions nowadays. If you're not using GitHub but for example BitBucket, they also have a similar feature called Pipelines.

Kubernetes is a good example of a system that manages the run stage. If a service crashes, it's automatically restarted. Or if a service starts to be overloaded by requests, Kubernetes can scale it up automatically.

# VI. Processes: Execute the app as one or more stateless processes

It's really important to make a distinction between what parts of your system maintain state and which parts don't. Examples of things that maintain state are your database, a cache, file storage, etc. Anything else shouldn't maintain state. So for example if you develop an API service that acts as a layer on top of your database, it shouldn't have any state. You might be tempted to do things locally, like maintain a log file, or store user activity. Don't. Because if you ever decide to scale up and maintain multiple services in parallel, this can result in a big mess.

Twelve-factor processes are stateless and share-nothing. Any data that needs to persist must be stored in a stateful backing service, typically a database.

You may use space temporarily if that's part of the transaction, but basically any request that an API handles should be isolated and not rely on data obtained in a previous request. For example, at my company we have a server that runs code on demand based on a set of files from a git repository. So when we handle each code running request, we retrieve the files from the Git repository and store them in a temporary folder, run the code and then delete the files again.

Basically, you should assume that at any moment, your stateless service could be restarted and any local data will be lost.

# VII. Port binding: Export services via port binding

The next few points are kind of specific and really technical. Which is a bit strange in my opinion as the other factors are more high-level.

The seventh factor is that your services communicate over a specific port.

Web apps are sometimes executed inside a webserver container. For example, PHP apps might run as a module inside Apache HTTPD, or Java apps might run inside Tomcat.

The twelve-factor app is completely self-contained and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app exports HTTP as a service by binding to a port, and listening to requests coming in on that port.

In a local development environment, the developer visits a service URL like http://localhost:5000/ to access the service exported by their app. In deployment, a routing layer handles routing requests from a public-facing hostname to the port-bound web processes.

This is typically implemented by using dependency declaration to add a webserver library to the app, such as Tornado for Python, Thin for Ruby, or Jetty for Java and other JVM-based languages. This happens entirely in user space, that is, within the app’s code. The contract with the execution environment is binding to a port to serve requests.

HTTP is not the only service that can be exported by port binding. Nearly any kind of server software can be run via a process binding to a port and awaiting incoming requests. Examples include ejabberd (speaking XMPP), and Redis (speaking the Redis protocol).

Note also that the port-binding approach means that one app can become the backing service for another app, by providing the URL to the backing app as a resource handle in the config for the consuming app.

# VIII. Concurrency: Scale out via the process model

Any computer program, once run, is represented by one or more processes. Web apps have taken a variety of process-execution forms. For example, PHP processes run as child processes of Apache, started on demand as needed by request volume. Java processes take the opposite approach, with the JVM providing one massive uberprocess that reserves a large block of system resources (CPU and memory) on startup, with concurrency managed internally via threads. In both cases, the running process(es) are only minimally visible to the developers of the app.

In the twelve-factor app, processes are a first class citizen. Processes in the twelve-factor app take strong cues from the unix process model for running service daemons. Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a process type. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

This does not exclude individual processes from handling their own internal multiplexing, via threads inside the runtime VM, or the async/evented model found in tools such as EventMachine, Twisted, or Node.js. But an individual VM can only grow so large (vertical scale), so the application must also be able to span multiple processes running on multiple physical machines.

The process model truly shines when it comes time to scale out. The share-nothing, horizontally partitionable nature of twelve-factor app processes means that adding more concurrency is a simple and reliable operation. The array of process types and number of processes of each type is known as the process formation.

Twelve-factor app processes should never daemonize or write PID files. Instead, rely on the operating system’s process manager (such as systemd, a distributed process manager on a cloud platform, or a tool like Foreman in development) to manage output streams, respond to crashed processes, and handle user-initiated restarts and shutdowns.

# IX. Disposability: Maximize robustness with fast startup and graceful shutdown

The twelve-factor app’s processes are disposable, meaning they can be started or stopped at a moment’s notice. This facilitates fast elastic scaling, rapid deployment of code or config changes, and robustness of production deploys.

Processes should strive to minimize startup time. Ideally, a process takes a few seconds from the time the launch command is executed until the process is up and ready to receive requests or jobs. Short startup time provides more agility for the release process and scaling up; and it aids robustness, because the process manager can more easily move processes to new physical machines when warranted.

Processes shut down gracefully when they receive a SIGTERM signal from the process manager. For a web process, graceful shutdown is achieved by ceasing to listen on the service port (thereby refusing any new requests), allowing any current requests to finish, and then exiting. Implicit in this model is that HTTP requests are short (no more than a few seconds), or in the case of long polling, the client should seamlessly attempt to reconnect when the connection is lost.

For a worker process, graceful shutdown is achieved by returning the current job to the work queue. For example, on RabbitMQ the worker can send a NACK; on Beanstalkd, the job is returned to the queue automatically whenever a worker disconnects. Lock-based systems such as Delayed Job need to be sure to release their lock on the job record. Implicit in this model is that all jobs are reentrant, which typically is achieved by wrapping the results in a transaction, or making the operation idempotent.

Processes should also be robust against sudden death, in the case of a failure in the underlying hardware. While this is a much less common occurrence than a graceful shutdown with SIGTERM, it can still happen. A recommended approach is use of a robust queueing backend, such as Beanstalkd, that returns jobs to the queue when clients disconnect or time out. Either way, a twelve-factor app is architected to handle unexpected, non-graceful terminations. Crash-only design takes this concept to its logical conclusion.

# X. Dev/prod parity: Keep development, staging, and production as similar as possible

Historically, there have been substantial gaps between development (a developer making live edits to a local deploy of the app) and production (a running deploy of the app accessed by end users). These gaps manifest in three areas:

The time gap: A developer may work on code that takes days, weeks, or even months to go into production.
The personnel gap: Developers write code, ops engineers deploy it.
The tools gap: Developers may be using a stack like Nginx, SQLite, and OS X, while the production deploy uses Apache, MySQL, and Linux.
The twelve-factor app is designed for continuous deployment by keeping the gap between development and production small. Looking at the three gaps described above:

Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
Make the tools gap small: keep development and production as similar as possible.
Summarizing the above into a table:

Traditional app Twelve-factor app
Time between deploys Weeks Hours
Code authors vs code deployers Different people Same people
Dev vs production environments Divergent As similar as possible
Backing services, such as the app’s database, queueing system, or cache, is one area where dev/prod parity is important. Many languages offer libraries which simplify access to the backing service, including adapters to different types of services. Some examples are in the table below.

Type Language Library Adapters
Database Ruby/Rails ActiveRecord MySQL, PostgreSQL, SQLite
Queue Python/Django Celery RabbitMQ, Beanstalkd, Redis
Cache Ruby/Rails ActiveSupport::Cache Memory, filesystem, Memcached
Developers sometimes find great appeal in using a lightweight backing service in their local environments, while a more serious and robust backing service will be used in production. For example, using SQLite locally and PostgreSQL in production; or local process memory for caching in development and Memcached in production.

The twelve-factor developer resists the urge to use different backing services between development and production, even when adapters theoretically abstract away any differences in backing services. Differences between backing services mean that tiny incompatibilities crop up, causing code that worked and passed tests in development or staging to fail in production. These types of errors create friction that disincentivizes continuous deployment. The cost of this friction and the subsequent dampening of continuous deployment is extremely high when considered in aggregate over the lifetime of an application.

Lightweight local services are less compelling than they once were. Modern backing services such as Memcached, PostgreSQL, and RabbitMQ are not difficult to install and run thanks to modern packaging systems, such as Homebrew and apt-get. Alternatively, declarative provisioning tools such as Chef and Puppet combined with light-weight virtual environments such as Docker and Vagrant allow developers to run local environments which closely approximate production environments. The cost of installing and using these systems is low compared to the benefit of dev/prod parity and continuous deployment.

Adapters to different backing services are still useful, because they make porting to new backing services relatively painless. But all deploys of the app (developer environments, staging, production) should be using the same type and version of each of the backing services.

# XI. Logs: Treat logs as event streams

Logs provide visibility into the behavior of a running app. In server-based environments they are commonly written to a file on disk (a “logfile”); but this is only an output format.

Logs are the stream of aggregated, time-ordered events collected from the output streams of all running processes and backing services. Logs in their raw form are typically a text format with one event per line (though backtraces from exceptions may span multiple lines). Logs have no fixed beginning or end, but flow continuously as long as the app is operating.

A twelve-factor app never concerns itself with routing or storage of its output stream. It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to stdout. During local development, the developer will view this stream in the foreground of their terminal to observe the app’s behavior.

In staging or production deploys, each process’ stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment. Open-source log routers (such as Logplex and Fluentd) are available for this purpose.

The event stream for an app can be routed to a file, or watched via realtime tail in a terminal. Most significantly, the stream can be sent to a log indexing and analysis system such as Splunk, or a general-purpose data warehousing system such as Hadoop/Hive. These systems allow for great power and flexibility for introspecting an app’s behavior over time, including:

Finding specific events in the past.
Large-scale graphing of trends (such as requests per minute).
Active alerting according to user-defined heuristics (such as an alert when the quantity of errors per minute exceeds a certain threshold).

# XII. Admin processes: Run admin/management tasks as one-off processes

The process formation is the array of processes that are used to do the app’s regular business (such as handling web requests) as it runs. Separately, developers will often wish to do one-off administrative or maintenance tasks for the app, such as:

Running database migrations (e.g. manage.py migrate in Django, rake db:migrate in Rails).
Running a console (also known as a REPL shell) to run arbitrary code or inspect the app’s models against the live database. Most languages provide a REPL by running the interpreter without any arguments (e.g. python or perl) or in some cases have a separate command (e.g. irb for Ruby, rails console for Rails).
Running one-time scripts committed into the app’s repo (e.g. php scripts/fix_bad_records.php).
One-off admin processes should be run in an identical environment as the regular long-running processes of the app. They run against a release, using the same codebase and config as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

The same dependency isolation techniques should be used on all process types. For example, if the Ruby web process uses the command bundle exec thin start, then a database migration should use bundle exec rake db:migrate. Likewise, a Python program using Virtualenv should use the vendored bin/python for running both the Tornado webserver and any manage.py admin processes.

Twelve-factor strongly favors languages which provide a REPL shell out of the box, and which make it easy to run one-off scripts. In a local deploy, developers invoke one-off admin processes by a direct shell command inside the app’s checkout directory. In a production deploy, developers can use ssh or other remote command execution mechanism provided by that deploy’s execution environment to run such a process.

Link: https://12factor.net