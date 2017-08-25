Setting and accessing enviroment variables in Spark jobs on EMR
=================================================================

Our use case is: we want to access a custom environment variable in our Spark job that we set previously.This use case seemed pretty simple, but I'm surprised how involved it got. In our case, our Spark job used this environment variable to access some paths protected by a secret key.

First, let's go through the options that didn't work for us:

- setting the environment variable at bootstrap time, ie `export SECRET_KEY=xxxx`. This looks like it should work, but doesn't probably of how hadoop/spark jobs are launched. 

- submit the Spark job with `spark.executorEnv.SECRET_KEY=xxx` and `spark.yarn.appMasterEnv.SECRET_KEY=xxxx`. These options are specified in the [documentation](https://spark.apache.org/docs/latest/configuration.html#runtime-environment), soit it looks like they should work. Unfortunately we struck out  here also. There's even a [Stack Overflow thread](https://stackoverflow.com/questions/36054871/spark-executorenv-doesnt-seem-to-take-any-effect) about it.


Here's what ended up working for us: at bootstrap time, set `SECRET_KEY=xxxx` in both `hadoop-env.sh` and `spark-env.sh`. You can do this using the [EMR API](http://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-configure-apps.html). The json request will look like this: 


     [
    {
    "Classification": "hadoop-env",
    "Properties": { 
    },
    "Configurations": [
      {
        "Classification": "export",
        "Properties": {
          "SECRET_KEY": "xxxx",
        },
        "Configurations": [
         ]
      }
    ]},
    
    {
    "Classification": "spark-env",
    "Properties": {
      
    },
    "Configurations": [
      {
        "Classification": "export",
        "Properties": {
          "SECRET_KEY": "xxxx",
        },
        "Configurations": [
          
        ]
      }
    ]}]
    

The obvious drawback with this approach is that to change the value of this variable, we'd have to log into each node and edit `spark-env.sh` and `hadoop-env.sh`.
 
