# The Twelve-Factor App

# Objectives
- Explain how conventions make development with teams easier and more efficient.
- Understand the conventions of a 12-Factor App.

# Important Conventions
In the previous lessons, we have learned a few important programming conventions, such as:
- Separation of Concerns
- DRY code
- Clean Code
- Convention over Configuration

Today we are going to learn a more sophisticated, and a more important set of conventions.

# The Twelve-Factor App
The 12-Factor App is a collection of 12 conventions which most modern application follow. It doesn't matter which languages, frameworks, or libraries you are using, you can always build a 12-Factor App.

N.B. This section is just a summary of information available on the 12-Factor App Website: https://12factor.net/.

A 12-Factor App Follows these conventions:

I. Codebase

One codebase tracked in revision control, many deploys
A twelve-factor app is always tracked in a version control system, such as Git, Mercurial, or Subversion. A copy of the revision tracking database is known as a code repository, often shortened to code repo or just repo.

If there are multiple codebases, it’s not an app – it’s a distributed system. Each component in a distributed system is an app, and each can individually comply with twelve-factor.

Multiple apps sharing the same code is a violation of twelve-factor. The solution here is to factor shared code into libraries which can be included through the dependency manager.

II. Dependencies

Explicitly declare and isolate dependencies

A twelve-factor app never relies on implicit existence of system-wide packages. It declares all dependencies, completely and exactly, via a dependency declaration manifest. Furthermore, it uses a dependency isolation tool during execution to ensure that no implicit dependencies “leak in” from the surrounding system.

Twelve-factor apps also do not rely on the implicit existence of any system tools. If the app needs to shell out to a system tool, that tool should be vendor into the app.

III. Config

Store config in the environment

An app’s config is everything that is likely to vary between deploys (staging, production, developer environments, etc). This includes:
- Resource handles to the database, Memcached, and other backing services
- Credentials to external services such as Amazon S3 or Twitter
- Per-deploy values such as the canonical hostname for the deploy

Twelve-factor, which requires strict separation of config from code. Config varies substantially across deploys, code does not.

A litmus test for whether an app has all config correctly factored out of the code is whether the codebase could be made open source at any moment, without compromising any credentials.

Note that this definition of “config” does not include internal application config, such as config/routes.rb in Rails. This type of config does not vary between deploys, and so is best done in the code.
The twelve-factor app stores config in environment variables (often shortened to env vars or env).

IV. Backing services

Treat backing services as attached resources

A backing service is any service the app consumes over the network as part of its normal operation. The code for a twelve-factor app makes no distinction between local and third party services. A deploy of the twelve-factor app should be able to swap out a local MySQL database with one managed by a third party (such as Amazon RDS) without any changes to the app’s code. Only the resource handle in the config needs to change.

V. Build, release, run

Strictly separate build and run stages

A codebase is transformed into a (non-development) deploy through three stages:
- The build stage is a transform which converts a code repo into an executable bundle known as a build. Using a version of the code at a commit specified by the deployment process, the build stage fetches vendors dependencies and compiles binaries and assets.
- The release stage takes the build produced by the build stage and combines it with the deploy’s current config. The resulting release contains both the build and the config and is ready for immediate execution in the execution environment.
- The run stage (also known as “runtime”) runs the app in the execution environment, by launching some set of the app’s processes against a selected release.
The twelve-factor app uses strict separation between the build, release, and run stages. Any change must create a new release. Every release should always have a unique release ID, such as v100.

Builds are initiated by the app’s developers whenever new code is deployed. Runtime execution, by contrast, can happen automatically in cases such as a server reboot, or a crashed process being restarted by the process manager. Therefore, the run stage should be kept to as few moving parts as possible, since problems that prevent an app from running can cause it to break in the middle of the night when no developers are on hand.

VI. Processes

Execute the app as one or more stateless processes
The app is executed in the execution environment as one or more processes. In the simplest case, the code is a stand-alone script, the execution environment is a developer’s local laptop with an installed language runtime, and the process is launched via the command line (ruby app.rb or node index.js). On the other end of the spectrum, a production deploy of a sophisticated app may use many process types, instantiated into zero or more running processes.

Twelve-factor processes are stateless and share-nothing. Any data that needs to persist must be stored in a stateful backing service, typically a database.
The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job.

VII. Port binding

Export services via port binding
The twelve-factor app is completely self-contained and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app exports HTTP as a service by binding to a port, and listening to requests coming in on that port.

Note also that the port-binding approach means that one app can become the backing service for another app, by providing the URL to the backing app as a resource handle in the config for the consuming app.

VIII. Concurrency

Scale out via the process model

In the twelve-factor app, processes are a first class citizen. Processes in the twelve-factor app take strong cues from the unix process model for running service daemons. Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a process type. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

Twelve-factor app processes should never daemonize or write PID files. Instead, rely on the operating system’s process manager (such as Upstart, a distributed process manager on a cloud platform, or a tool like Foreman in development) to manage output streams, respond to crashed processes, and handle user-initiated restarts and shutdowns.

IX. Disposability

Maximize robustness with fast startup and graceful shutdown
The twelve-factor app’s processes are disposable, meaning they can be started or stopped at a moment’s notice. This facilitates fast elastic scaling, rapid deployment of code or config changes, and robustness of production deploys.
Processes should strive to minimize startup time.

Processes shut down gracefully when they receive a SIGTERM* signal from the process manager. For a web process, graceful shutdown is achieved by ceasing to listen on the service port (thereby refusing any new requests), allowing any current requests to finish, and then exiting.

Processes should also be robust against sudden death, in the case of a failure in the underlying hardware. A twelve-factor app is architected to handle unexpected, non-graceful terminations. Crash-only design takes this concept to its logical conclusion.

* The SIGTERM signal is sent to a process to request its termination.

X. Dev/prod parity

Keep development, staging, and production as similar as possible

The twelve-factor app is designed for continuous deployment by keeping the gap between development and production small. Looking at the three gaps described above:
- Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
- Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
- Make the tools gap small: keep development and production as similar as possible.

The twelve-factor developer resists the urge to use different backing services between development and production.

XI. Logs

Treat logs as event streams

A twelve-factor app never concerns itself with routing or storage of its output stream. It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to stdout. During local development, the developer will view this stream in the foreground of their terminal to observe the app’s behavior.

In staging or production deploys, each process’ stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment.

XII. Admin processes

Run admin/management tasks as one-off processes
One-off admin processes should be run in an identical environment as the regular long-running processes of the app. They run against a release, using the same codebase and config as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

Twelve-factor strongly favors languages which provide a REPL shell out of the box.
