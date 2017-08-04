How does Spark's `wholeTextFiles` work?
==========================================

Spark's  `wholeTextFiles` is a pretty painless way to read many small files. It returns an RDD of pair values, where the key is the path of each file, the value is the content of each file. It works by relying on Hadoop's `CombineFileInputFormat`, which combines multiple small files into bigger chunks to get better performance. Here are the steps taken by this call:

 - List the all the relevant files in the given paths. This can be done in parallel if the file system supports. This is our original input. 
 - Group the orignal input into bigger chunks to be split later. You can control the number of splits you want. In the Spark API, this is done by passing a calling `wholeTextFiles()` with a specific `minPartitions` value. The default is 2, which is very low. As alway, there is a balance to be struck here: set this value too low and you'll be using fewer workers and your network performance will be low. Set it too high and the number of partitions can overwhelm the cluster. 
 
 The only catch here is that if you're using S3 as your source, then listing files in a bucket be single-threaded. This is because the S3 API for listing the keys in a bucket only returns keys by chunks of 1000 per call(that's the maximum amount of keys it can return in a single call). So if you have at millions of files, you're looking at at  thousands API calls to s3. As an example, it took about 15 minutes to list all the keys in a bucket that as about 7 million keys. During that operation, only the driver node was working. 
