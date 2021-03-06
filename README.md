# Jupyter_Notebook_on_aws_Spark_Cluster

# Spinning up Spark cluster on aws and running Jupyter Notebook


##1. Starting up the AWS Elastic MapReduce (EMR) Cluster
Log into AWS with your account.

Under ‘Analytics’, choose ‘EMR - Managed Hadoop Framework’.

Click on ‘Create Cluster’

Select the cluster options. See below for recommended configuration. Note that your EC2 key pair will be named differently (whatever you created saved it as).

	launch mdoe : cluster
	Vendor: Amazon
	Release : emr-4.1.0
	select spark


Wait for cluster provisioning. This can take 15-20 minutes depending on availability and configuration chosen. If you have not yet downloaded the Sparkov code or the sample data set, this would be a good time to do so. You may also want to compress the customer / transaction data set into a .zip / tar.gz file for faster transfer to AWS, as we will need to put the files on the cluster HDFS.

Obtain your “Master Public DNS” name for the name node. To do this, click on “SSH” on the “Cluster Details” page (Master Public DNS: ec2-xx-xx-xxx-xx.whatever.amazonaws.con SSH) 

Choose Mac/Linux (or Windows, if you’re using Windows). 

Copy the ‘ssh -i’ line from the prompt and paste into a terminal shell to connect to your cluster. Save this line in a text editor, as we’ll refer to it frequently.

ssh -i ~/xxx.pem hadoop@ec2-52-91-xx-xxx.compute-1.amazonaws.com


##2. get the name node ips:

hadoop dfsadmin -report | grep ^Name | cut -f2 -d: | cut -f2 -d' '

DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

172.31.xx.xx
172.31.xx.xx

##3. exit

##4. We need to our AWS key pair / .pem file to our local SSH agent.

ssh-add ~/.ssh/xxx.pem

##5. Now we’ll re-connect to AWS (again, replace the below connection information with your AWS DNS information).

ssh -A hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com


##6. Python Upgrade Steps

Please execute the following steps to upgrade Python on the name node. You can re-use the same instructions for each data node, though the output may vary slightly depending on the data node configurations.


sudo yum install make automake gcc gcc-c++ kernel-devel git-core -y 
sudo yum install python27-devel -y 
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python2.7 /usr/bin/python 
sudo cp /usr/bin/yum /usr/bin/_yum_before_27 
sudo sed -i s/python/python2.6/g /usr/bin/yum
sudo sed -i s/python2.6/python2.6/g /usr/bin/yum

At this point we should make sure python is reporting the correct versions.

python -V

It should output a version number higher than 2.7.0, which means our upgrade was successful.

Python Upgrade on the Data Node(s)

We need to repeat the python upgrade steps for each data node in our cluster. The steps remain the same as above, but in order to connect to our data nodes we can execute the following command (be sure to replace the IP address listed below with the IP addresses we obtained via the hadoop dfsadmin command in the section labeled ‘Upgrading Python on each node in the cluster’).

ssh hadoop@172.xx.xx.xx

This should log you in automatically to the data node. Repeat the above steps to ugprade python. Once complete, connect to each subsequent data node IP address and upgrade python until all data nodes are upgraded. You should be able to paste the entire block of text, rather than each line individually. Remember to close each SSH connection (use the exit command) after upgrading and return to the name node before connecting to the next data node.



##7. on master: sudo pip install nose "ipython[notebook]"

##8.
PYSPARK_DRIVER_PYTHON=ipython PYSPARK_DRIVER_PYTHON_OPTS="notebook --no-browser --port=7777" pyspark --packages com.databricks:spark-csv_2.10:1.1.0 --master spark://spark_master_hostname:7077 --executor-memory 6400M --driver-memory 6400M

##9.
ssh -N -f -L localhost:7776:localhost:7777 hadoop@ec2-xx-xx-xx-xxx.compute-1.amazonaws.com

##10.
http://localhost:7776
