---
layout: post
title: Reading Dataset in Tensorflow by TFRecords
tags: [Tensorflow,MNIST]
comments: True
---

<div class="message">
  This post shows how to encode and decode MNIST dataset in Tensorfow by Tfrecords.
</div>

### Why Tfrecords?
Tfrecords are binary files defined by Tensorflow. By using Tfrecords to read and save datasets, we can save lot of time and memory. Most of all, it works efficiently with Queue, which allow the training thread and reading data thread to be independent.  

Let's see how to encode and decode data step-by-step. In this post, we use MNIST dataset as example.

### Create Tfrecords

- Read MNIST Dataset. As tensorflow already provide us the function of reading the MNIST dataset: *mnist.read_data_sets*, we shall just use it to get data. 

{% highlight js %}
data_sets = mnist.read_data_sets("data_dir",dtype=tf.uint8, reshape=False,validation_size=100)
{% endhighlight %}

- Encode the data to binary format. To do so we use [tf.train.Example](https://www.tensorflow.org/api_docs/python/tf/train/Example) Protobuf objects. The protocol buffer contains Features, which is a protocol to describe the data. There are three types of features: bytes, float and int64. In our case, the feature of a data is its image and label.

- Write binary data into tfrecords. We use [TFRecordWriter](https://www.tensorflow.org/api_docs/python/tf/python_io/TFRecordWriter) to write data.

{% highlight js %}

def _int64_feature(value):
  return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))

def _bytes_feature(value):
  return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))

with tf.python_io.TFRecordWriter(filename) as tf_writer:
        for i in range(num):
            image_raw = images[i].tostring()
            example = tf.train.Example(features=tf.train.Features(feature={
                'label':_int64_feature(int(labels[i])),
                'image_raw':_bytes_feature(image_raw)
            }))
            tf_writer.write(example.SerializeToString())
        tf_writer.close()
{% endhighlight %}


### Reading tfrecords
Reading tfrecords usually works with Queues and Coordinators, which allow us to handle the decoding process in multiple threads.
- Thread coordination. Tensorflow use [tf.train.Coordinator](https://www.tensorflow.org/api_docs/python/tf/train/Coordinator) to control the threads.
{% highlight js %}
    coord = tf.train.Coordinator()//create a coordinator
	threads = //create threads
	while not coord.should_stop():
		//do the works
		if work is done:
			coord.request_stop()//request that the thread should stop
	coord.join(threads)//wait until the specified thread have stopped
{% endhighlight %}
- Queue and QueueRunner. Queue is like a container for the files. There are couples of different types of Queues. In our case, we use the FIFO (first in first out) Queue.  QueueRunner creates a number of threads that repeatly fun an enqueue op. 
  
*tf.train.string_input_producer* will return a queue and add a QUEUE_RUNNER to the current graph.
{% highlight js %}
    filename_queue = tf.train.string_input_producer([tf_file], num_epochs=epoch)
{% endhighlight %}

- Read data from tfrecords and decode the data.
{% highlight js %} 
def read_and_decode(file_q):
    reader = tf.TFRecordReader()
    _,serialized_example = reader.read(file_q)
    features = tf.parse_single_example(serialized_example,
                                       features={
                                           'row': tf.FixedLenFeature([], tf.int64),
                                           'col': tf.FixedLenFeature([], tf.int64),
                                           'depth': tf.FixedLenFeature([], tf.int64),
                                           'label': tf.FixedLenFeature([], tf.int64),
                                           'image_raw':tf.FixedLenFeature([],tf.string)
                                       })
    //Convert label from a scalar uint8 tensor to an int32 scalar.
    label = tf.cast(features['label'], tf.int32)
    //Convert row from a scalar uint8 tensor to an int32 scalar.
    row = tf.cast(features['row'], tf.int32)
    //Convert col from a scalar uint8 tensor to an int32 scalar.
    col = tf.cast(features['col'], tf.int32)
    //Convert depth from a scalar uint8 tensor to an int32 scalar.
    depth = tf.cast(features['depth'], tf.int32)
    image = tf.decode_raw(features['image_raw'], tf.uint8)
    image.set_shape([mnist.IMAGE_PIXELS])


    return image,label,row,col,depth
{% endhighlight %}

- Data batch. We use a shuffling queue to create batches.
{% highlight js %}
//return a list or dictionary. adds 1) a shuffling queue into which tensors are enqueued; 2) a dequeue_many operation to create batches from the queue 3) a queue runner to QUEUE_RUNNER collection , to enqueue the tensors.
data, labels,H,W,D = tf.train.shuffle_batch([image, label,row,col,depth], batch_size=batch_size, num_threads=2,                                                     capacity=100 + 3 * batch_size, min_after_dequeue=1)
{% endhighlight %}


The general steps are as follows:
{% highlight js %}
- Define a queue.
- Define operation nodes to enqueue data.
- Defind an operation node to dequeue data.
- Create a QueueRunner to start multiple enqueue.
- Create a session and a coordinator.
-- Let the QueueRunner to lauch multiple enqueue node threads
-- Run the dequeue operation
-- Wait until the coordinator to request stop.
{% endhighlight %}