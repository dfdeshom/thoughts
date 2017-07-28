Writing a map-reduce job to concatenate a millions of small documents
======================================================================

Hadoop and small files really don't mix well and when you add to that terabytes of small files, things don't bode well. Usually, you deal with the small files issue by using `distcp` but your files have to be concatenable to begin with, like json or similar. In our case our files are raw documents (e.g. pdf, doc files), and we want to run analysis on each of them individually. It would be nice to process larger chunks of them. So let's turn millions of small files into hundreds of bigger files.

The job
=========

First we need a map-reduce job. We basically need to do what `distcp`, which is concatenate files, does but in a way that we will be able to refer to individual documents. Here is something simple: 

```python
import  json
from mrjob.job import MRJob
from mrjob.protocol import TextValueProtocol
import os

class KittyJob(MRJob):
    """An M/R version of /bin/cat """
    OUTPUT_PROTOCOL = TextValueProtocol
    INPUT_PROTOCOL = TextValueProtocol
    HADOOP_INPUT_FORMAT = "org.apache.hadoop.mapred.lib.CombineTextInputFormat"

    def mapper(self, _, line):
        fname = os.environ['mapreduce_map_input_file']
        yield fname, line

    def reducer(self, fname, lines):
        text = "".join(lines)
        yield fname, json.dumps({"raw_content": text})
```

Here I'm using `mrjob` since it makes it easy to write one. You'll also notice that I'm using the `CombineTextInputFormat` which groups smaller files into bigger ones. I'm using it along with `mapreduce.input.fileinputformat.split.maxsize=800000000` which tells hadoop to group the small files into ones of apprximately chunks of 800 MB. With this simple job and some big machines, we can process about 0.5 terabyte of small files in about 3 hours

Production tips for huge loads
===============================

For cases where you have 500+ GB of small files, here are some useful tips:  

  - you will want to give the driver more memory, via the `HADOOP_HEAPSIZE` envirnment variable. It can often takes a good 10 minutes for the job to set up its input list in cases like this. 

  - The number of `map` tasks depends on the input split of your input so make sure that your input split is big enough (controlled in this case by `mapreduce.input.fileinputformat.split.maxsize`) so that you don't have to spawn thousands of mappers. This is especially important in the cases where you have tens of millions of small files.

  - In cases like this it's probably not necessary to set the number of `reduce` tasks. In this case, this number will be guaranteeed to be small enough. Small number of reduce tasks lead to bigger chunks of data being written, which is what we want anyway. 

  - Expect to tweak the amount of memory assigned to map and reduce tasks

