This post has been inspired by
http://blog.maartenballiauw.be/post/2015/08/04/Replaying-IIS-request-logs-using-Apache-JMeter.aspx

What is Locust?
----------------
[Locust](http://locust.io/) is a  load-testing framework written in python. It is similar
to JMeter, but allow much more flexibility since you can use a proper
programing language to define your tests. The framework makes it easy
to write tests. Here is an example:

    from locust import HttpLocust, TaskSet, task

    class WebsiteTasks(TaskSet):
        @task
        def index(self):
            self.client.get("/")

        @task
        def about(self):
            self.client.get("/about/")

    class WebsiteUser(HttpLocust):
        task_set = WebsiteTasks


Using Locust to replay log files
---------------------------------
Our interest in replaying log files is for the following use case: we
are migrating our API to a new backend. We want to make sure that both
versions of the API work the same (even down to the same bugs!) and
that we obtain the similar results on both API versions. We could
painstakingly  write locust tasks that test each endpoint but:

 - we might overlook a combination of requests parameters that some
   API clients use frequently on each endpoint.

 - we want to be able to get a rough idea of the performance of both
   APIs, ideally the new backend should perform better than the old
   one (since backend migrations are almost always about improving
   performance)

Feeding locust
---------------
First we need to parse the log file to extract relative urls out of
it. Each line we parse and extract will be a url to be fed to locust. 

Next comes the problem of feeding each url to locust to execute.
Every `TaskSet` has a `tasks` attribute, which is a list of tasks you
can pass it that will be executed. Each task implicitly takes a
`Taskset` instance as its first argument. The syntax is as follows:

    class MyTaskSet(TaskSet):
        tasks = [mytask1, ]

Which means that if we pass each url to be retrieved as a task, we
will accomplish our goal.

    import functools
    def gen_tasks(taskset):
        def func(url):
            taskset.client.get(url)

        urls = urls_from_log_file(log_file)
        data = [functools.partial(func,url) for url in urls]


Now we just have to tell our `TaskSet` to use our list of generated
tasks:

    class MyTaskSet(TaskSet):
        def __init__(self, parent):
            super(MyTaskSet, self).__init__(parent)
            self.tasks = gen_tasks(self)

Distributing Locust tasks and their data
-----------------------------------------
Here is our Locust works in distributed mode: there is one master node
and several slave nodes. The master node will take care of determining
how many locust task it can start on each node. Each locust started
will have access to the same tasks and thus same data, if that data
resides in a file.

Having access to the same data and replaying that is fine for most
cases, but in the case of million of tasks, we may also want to find a
way to distribute the total number of tasks among several slave nodes,
by giving them separate chunks.

Idea: save all urls in a redis hash. Eache slave that is started will
take a chunk of this redis hash (the chunks can be overlapping for
simplicity) and populate its `tasks` attribute with that. Each node
can also delete items in the hash so that other nodes don't have to
get the same urls. The `HDEL <keys...>` is a fairly efficient command
in this respect.

So we simply need to modify our `urls_from_log_file` to store urls
into redis. We then access this list of urls by putting a lock on read
and delete operations so that other slave clients don't get overlaping
or stale data.
