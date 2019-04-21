# hadoopSort
hadoop利用MapReduce进行排序
hadoop实现最基本的数字排序，并且是多文件的总排序。
配置：
系统：**ubuntu 16.04**

java :  **1.8.0_191**

hadoop: **1.2.1**

实现的前提是配置好hadoop环境变量并启动。
终端输入：
```linux
jps
```
如果出先下列进程则说明**hadoop启动成功**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421143830768.png)
# 一、MapReduce 执行过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421134443844.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZUdlNjY2,size_16,color_FFFFFF,t_70)
# 二、排序算法讲解

```java
import java.io.IOException;

import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;

import org.apache.hadoop.fs.Path;

import org.apache.hadoop.io.IntWritable;

import org.apache.hadoop.io.Text;

import org.apache.hadoop.mapreduce.Job;

import org.apache.hadoop.mapreduce.Mapper;

import org.apache.hadoop.mapreduce.Reducer;

import org.apache.hadoop.mapreduce.Partitioner;

import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import org.apache.hadoop.util.GenericOptionsParser;

public class Sort {

	public static class Map extends
			Mapper<Object, Text, IntWritable, IntWritable> {

		private static IntWritable data = new IntWritable();

		public void map(Object key, Text value, Context context)
				throws IOException, InterruptedException {
			String line = value.toString();

			data.set(Integer.parseInt(line));

			context.write(data, new IntWritable(1));

		}

	}

	public static class Reduce extends
			Reducer<IntWritable, IntWritable, IntWritable, IntWritable> {

		private static IntWritable linenum = new IntWritable(1);

		public void reduce(IntWritable key, Iterable<IntWritable> values,
				Context context) throws IOException, InterruptedException {

			for (IntWritable val : values) {

				context.write(linenum, key);

				linenum = new IntWritable(linenum.get() + 1);
			}

		}
	}

	public static class Partition extends Partitioner<IntWritable, IntWritable> {

		@Override
		public int getPartition(IntWritable key, IntWritable value,
				int numPartitions) {
			int MaxNumber = 65223;
			int bound = MaxNumber / numPartitions + 1;
			int keynumber = key.get();
			for (int i = 0; i < numPartitions; i++) {
				if (keynumber < bound * i && keynumber >= bound * (i - 1))
					return i - 1;
			}
			return 0;
		}
	}

	/**
	 * @param args
	 */

	public static void main(String[] args) throws Exception {
		// TODO Auto-generated method stub
		Configuration conf = new Configuration();
		String[] otherArgs = new GenericOptionsParser(conf, args)
				.getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("Usage WordCount <int> <out>");
			System.exit(2);
		}
		Job job = new Job(conf, "Sort");
		job.setJarByClass(Sort.class);
		job.setMapperClass(Map.class);
		job.setPartitionerClass(Partition.class);
		job.setReducerClass(Reduce.class);
		job.setOutputKeyClass(IntWritable.class);
		job.setOutputValueClass(IntWritable.class);
		FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}

```
# 三、具体操作
## 1、创建工程目录
```linux
cd /usr/projects/hadoopExamples
mkdir numbersort
```
## 2、写入Sort.java文件并编译(javac)
将上面的java代码写入复制入Sort.java并保存；
```linux
vi Sort.java
```
创建目录number_sort_class 存放编译好的.class文件；
```linux
mkdir number_sort_class
```
编译Sort.java文件并将编译好的.class文件存放到number_sort_class文件夹下；
/opt/hadoop-1.2.1为我的hadoop安装目录。
```linux
javac -classpath /opt/hadoop-1.2.1/hadoop-core-1.2.1.jar:/opt/hadoop-1.2.1/lib/commons-cli-1.2.jar -d number_sort_class/ Sort.java
```
## 3、将编译好的.class文件打包
*代表所有的.class文件，
将打包的好的jar命名为numberSort.jar；
```linux
cd number_sort_class
jar -cvf numberSort.jar *.class
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421141853715.png)
## 4、创建输入文件
回到上级目录，创建input目录，并创建3个文件，里面每个都存放多个数字。
例如：![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421141244202.png)
```linux
cd ..
mkdir input
cd input
vi number1
vi number2
vi number3
cd ..
```
## 5、上传输入文件到hadoop中
必须先创建文件夹才能上传，
将input下所有的文件都上传到hadoop下的input_numersort中；
```linux
hadoop fs -mkdir input_numbersort
hadoop fs -put input/* input_numersort/
```
查看上传的文件；

```linux
hadoop fs  -ls
```
结果:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421142034711.png)
查看具体文件目录：

```linux
hadoop fs -ls input_numbersort
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421142223520.png)
正是我们input文件夹中的三个number文件!

查看文件具体内容：

```linux
hadoop fs -cat input_numbersort/number2
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019042114234786.png)
## 6、将运行jar包并合成输出结果
jar包的路劲为number_sort_class/numberSort.jar，
Sort为java文件的名称 ，
input_numbersort 为输入文件夹，
out_numbersort 为输出文件（自动创建）；
```linux
hadoop jar number_sort_class/numberSort.jar Sort input_numbersort out_numbersort
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421142842832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZUdlNjY2,size_16,color_FFFFFF,t_70)
可以看到先是map到达100%后，reduce才开始执行。

在查看下hadoop文件：

```linux
hadoop fs -ls
```
结构，发现多了一个out_numbersort，就是刚刚输出出来的，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421143305168.png)
查看out_numbersort目录：

```linux
hadoop fs -ls out_numbersort
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421143501343.png)
结果就在part-r-00000中。

## 7、查看结果
查看排序结果。
```linux
hadoop fs -cat out_numbersort/part-r-00000
```
结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421143627403.png)
到这里hadoop利用MapReduce进行排序就已经完成了，对以后学习更复杂的hadoop应用很有帮助。
