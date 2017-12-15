This is somewhat the inverse of reading of millions of tiny files on S3. Both tasks share the same main characteristic: this is a network-bound job so you the more executors you have, the better. And since these are small files, executor memory is likely not an issue.

The main idea is that you want to minimize the amount of time you instantiate a `S3` connection that writes the files. Creating and then destroying connections for an RDD with millions of entries will take hours to complete. In these situations, you use `mapPartitions`. This will allow you to create one connection per partition (vs per RDD item) and re-use that connection to process items in the partition. Here's a simple example in a gist here : https://gist.github.com/dfdeshom/0496d4b4f659fdab48a33f7ad9c4f3de

Using something similar to this script,  I was able to write millions of files to an s3 bucket in a couple of hours. 
