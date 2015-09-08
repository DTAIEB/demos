# Tutorial: Build a custom library for Spark and deploy it to a Jupyter Notebook in IBM Analytics for Apache Spark on Bluemix

**Introduction**  
In this tutorial, you will learn how to build a simple custom library for Spark (written in [scala](http://scala-lang.org/)) and how to deploy it on IBM Analytics for Apache Spark on Bluemix. The motivation is to encapsulate your business logic into reusable modules that can be deployed on [Bluemix Apache Spark](https://console.ng.bluemix.net/catalog/apache-spark-starter/) and then easily reused by Notebook users (data scientists, data engineers, etc...)  
We will go over the following steps:  
	1. Create a new Scala project using sbt and package it as a deployable jar  
	2. Optional: Import, test and debug your project into Scala IDE for Eclipse  
	3. Deploy the jar into a Jupyter Notebook on Bluemix  
	4. Call the helper functions from a Notebook cell  

**Prerequisites**  
* Sign up for an Apache spark account on Bluemix [here](https://console.ng.bluemix.net/catalog/apache-spark-starter/)  
* Download scala sbt (simple build tool) from [here](http://www.scala-sbt.org/download.html)  
* Install sbteclipse (sbt plugin for Eclipse) from [here](https://github.com/typesafehub/sbteclipse/wiki/Installing-sbteclipse)   
* Be familiar with the [scala language](http://scala-lang.org/)  
* Download the Scala IDE for Eclipse [here](http://scala-ide.org/download/current.html). (Note that you can alternatively use the Intellij scala IDE [here](https://www.jetbrains.com/idea/features/scala.html) but it would be easier to follow this tutorial with Scala IDE for Eclipse)   
* Download scala runtime 2.10.4 from [here](http://www.scala-lang.org/download/2.10.4.html)

**Create a Scala project for Eclipse using sbt**  
There are multiple build frameworks that can be used to build Spark projects and [Maven](https://maven.apache.org/) is widely used by enterprise build engineer. However, SBT which is an alternative to Maven that will be used in this tutorial, provides you with a end to end faster time to working.  
The following steps let you create a new Spark project. You can also directly download the code from this [location on Github](https://github.com/ibm-cds-labs/spark.samples): 
 
1. From a terminal, cd to the directory that contains your development project and create a directory named "helloSpark":  
 
   ```  
	mkdir helloSpark && cd helloSpark
   ```  	 

2. Create the recommended directory layout for projects builts by Maven or SBT:  

   ```  
	mkdir -p src/main/scala
	mkdir -p src/main/java
	mkdir -p src/main/resources
  ```  

3. In src/main/scala directory, create a subdirectory that correspond to the package of your choice. e.g. `mkdir -p com.ibm.cds.spark.samples`. Then create a new file called HelloSpark.scala and with your favorite editor add the following contents:  

   ```scala
	package com.ibm.cds.spark.samples

	import org.apache.spark._

	object HelloSpark {
	  	//main method invoked when running as a standalone Spark Application
  		def main(args: Array[String]) {
    		val conf = new SparkConf().setAppName("Hello Spark")
    		val spark = new SparkContext(conf)

    		println("Hello Spark Demo. Compute the mean and variance of a collection")
    		val stats = computeStatsForCollection(spark);
    		println(">>> Results: ")
    		println(">>>>>>>Mean: " + stats._1 );
    		println(">>>>>>>Variance: " + stats._2);
    		spark.stop()
  		}
  
  		//Library method that can be invoked from Jupyter Notebook
  		def computeStatsForCollection( spark: SparkContext, countPerPartitions: Int = 100000, partitions: Int=5): (Double, Double) = {    
    		val totalNumber = math.min( countPerPartitions * partitions, Long.MaxValue).toInt;
    		val rdd = spark.parallelize( 1 until totalNumber,partitions);
    		(rdd.mean(), rdd.variance())
  		}
	}
   ```

4. The next step is to create your sbt build definition. Note: You can you find detailed documentation on sbt build definition [here](http://www.scala-sbt.org/0.12.4/docs/Getting-Started/Hello.html). In your project root directory, create a file called build.sbt and add the following contents:  
   
   ```scala
	name := "helloSpark"
	
	version := "1.0"

	scalaVersion := "2.10.4"

	libraryDependencies ++= {
  		val sparkVersion =  "1.3.1"
	  	Seq(
	    	"org.apache.spark" %% "spark-core" % sparkVersion,
	    	"org.apache.spark" %% "spark-sql" % sparkVersion,
	    	"org.apache.spark" %% "spark-repl" % sparkVersion 
	  	)
	}
   ```  
The libraryDependencies line is telling sbt to download the specified spark components. In this example, we specify dependencies to spark-core, spark-sql and spark-repl, but you may add more spark components dependencies by following the same pattern e.g. spark-mllib, spark-graphx, etc... 
5. From the root directory of your project, run the following command: `sbt update`. Using apache ivy, this command to compute all the dependencies and download them in your local machine at <home>/.ivy2/cache directory
6. You are now ready to compile your source code using the following command: `sbt compile`
7. After you've succesfully compiled the code, the next step is to package it as a jar using the following command: `sbt package`. If all goes well, you should see a file name hellospark_2.10-1.0.jar in the root directory of your project.  
Note the naming convention for the jar file is as follow: \[projectName\]\_\[scala_version\]\_\[projectVersion\]

**Optional: Import, test and debug your project in Scala IDE for eclipse**  
This is an optional step that lets you import, test and debug your project in a local deployment of Spark. It is assumed to you have already installed Scala IDE for Eclipse as well as downloaded and installed Scala runtime 2.10.4.  

1. Configure Scala IDE to run with Scala 2.10.4: Select Preferences/Scala/Installations, then click on the Add button. Navigate to your scala 2.10.4 installation root directory and select the lib directory. Click ok and give a name to your installation. Click OK multiple time to complete this step.  
	![Scala installation](https://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-tutorial-Configure-Scala-2.10.4-installation.png)  
	
2. Generate the eclipse artifacts necessary to import the project into Scala IDE for eclipse. This is done using the sbt plugin for eclipse that you downloaded from [here](https://github.com/typesafehub/sbteclipse/wiki/Installing-sbteclipse). From the root directory use the following command: `sbt eclipse`. Once done, verify that .project and .classpath have been succesfully created  

3. From Scala IDE, use File Import/General/Existing Projects into Workspace menu. Click next.  

4. In the next dialog page, use Browse button to navigate to the root directory of your project, then click finish: ![Import Scala project](http://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-tutorial-Import-Scala-Project-1024x560.png)  

5. If you have project/build automatically turn on, the project will automatically be compiled. You will notice error in the problems view. This is because you need to configure the scala installation for your project. To do this, right click on the project and select Scala/Set the Scala installation, then select 2.10.4 in the dialog. Click ok and wait until the project has been recompiled. You should see no more error in the problems view: ![Set Scala Installation Menu](http://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-tutorial-set-Scala-Intallation-1-1024x997.png)  ![Select Scala Installation](http://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-tutorial-set-Scala-installation-2.png)

6. The next step is to create a launch configuration that will start a spark-shell. Select Run/Run Configurations menu  

7. Right click on Scala Application from the left handside and select New  
8. In the main tab, under Main Class input box, type `org.apache.spark.deploy.SparkSubmit`  
9. In the argument tab, type the following in the Program Arguments box: `--class org.apache.spark.repl.Main spark-shell`  
10. In the argument tab, type the following in VM Argument box: `-Dscala.usejavacp=true
-Xms128m -Xmx800m -XX:MaxPermSize=350m -XX:ReservedCodeCacheSize=64m`.  ![Launch Configuration](http://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-tutorial-Launch-configuration-1024x677.png)
11. Click Run.  
12. Open the console view and wait until you get the scala prompt.


One thing that's important to note is that the previous steps showed how to run/debug a spark-shell from within your development environment that includes your project in the classpath, which can then be called from the shell interpreter.  
Another use case is to build a self-contained [Spark Application](http://spark.apache.org/docs/latest/quick-start.html#self-contained-applications) that can be run manually using spark-submit or scheduled. This sample code is designed to run both as a Spark Application and a reusable library. If you want to run/debug the Spark application from within the Scala IDE, then you can follow the same steps as above, but in Step 9, replace the call in the Program Arguments box with the fully qualified name of your main class.  

**Deploy your custom library jar into a Jupyter Notebook**  
By now, you've been able to build, test, debug and package your custom library. In this section, we'll look into how to deploy it on Bluemix into a Jupyter Notebook.
Note: it is assumed that you already have an accound on IBM Analytics for Apache Spark, if not please take the time to sign up for a free trial [here](https://console.ng.bluemix.net/catalog/apache-spark-starter).  

1. The first step is to upload your deployable jar into a publicly available url. One possibility is to upload the jar into a github repository. The download url will be used later to deploy the jar into Spark as a Service. For convenience, the sample code has been pre-built and posted at the following [url on github](https://github.com/ibm-cds-labs/spark.samples/raw/master/dist/hellospark_2.10-1.0.jar)
2. If you haven't done so, create a new instance of Apache Spark by using the Apache Spark Starter Boilerplate (From the Bluemix dashboard, use Create App tile, then select Web and Browse Boilerplates)  
3. Open the Spark application and launch the Notebook.  
4. Next, we'll use a special command call AddJar to upload the jar from the url created in Step 1 e.g. `%AddJar https://github.com/ibm-cds-labs/spark.samples/raw/master/dist/hellospark_2.10-1.0.jar -f`   
Notice the % before the AddJar which is part of the special command and the -f to force the download even the file is already in the cache.  
5. Now that the jar has been deployed, you can call APIs from within the Notebook.

**Call the helper functions from a Notebook cell**  
You are now able to call the code from the helloSpark sample library. In a new cell, add the following code:  

	```  
	val countPerPartitions = 500000
	var partitions = 10
	val stats = com.ibm.cds.spark.samples.HelloSpark.computeStatsForCollection(
   			 sc, countPerPartitions, partitions)
	println("Mean: " + stats._1)
	println("Variance: " + stats._2)
	```  

The following screen shot shows the final results on a Jupyter Notebook: ![Hello Spark Jupyter Demo](http://developer.ibm.com/clouddataservices/wp-content/uploads/sites/47/2015/09/Spark-Tutorial-HelloSpark-Jupyter-1024x509.png)


